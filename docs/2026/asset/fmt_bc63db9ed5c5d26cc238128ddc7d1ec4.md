

```c
Java_jdk_internal_loader_NativeLibraries_load(JNIEnv *env, jobject this, jobject lib, jstring name, jboolean isBuiltin, jboolean isJNI) {
    const char *cname;
    jint jniVersion;
    jthrowable cause;
    void *handle;
    jboolean loaded = JNI_FALSE;

    if (!initIDs(env))
        return JNI_FALSE;

    cname = JNU_GetStringPlatformChars(env, name, 0);
    if (cname == 0)
        return JNI_FALSE;

    handle = isBuiltin ? procHandle : JVM_LoadLibrary(cname);

    if (isJNI) {
        if (handle) {
            JNI_OnLoad_t JNI_OnLoad;
            JNI_OnLoad = (JNI_OnLoad_t)findJniFunction(
                env,
                handle,
                isBuiltin ? cname : NULL,
                JNI_TRUE
            );

            if (JNI_OnLoad) {
                JavaVM *jvm;
                (*env)->GetJavaVM(env, &jvm);
                jniVersion = (*JNI_OnLoad)(jvm, NULL);
                // 应该是执行 JNI_Onload 的地方
            } else {
                jniVersion = 0x00010001;
            }

            cause = (*env)->ExceptionOccurred(env);
            if (cause) {
                (*env)->ExceptionClear(env);
                (*env)->Throw(env, cause);

                if (!isBuiltin) {
                    JVM_UnloadLibrary(handle);
                }

                goto done;
            }

            if (!JVM_IsSupportedJNIVersion(jniVersion) ||
                (isBuiltin && jniVersion < JNI_VERSION_1_8)) {
                char msg[256];

                jio_snprintf(
                    msg,
                    sizeof(msg),
                    "unsupported JNI version 0x%08X required by %s",
                    jniVersion,
                    cname
                );

                JNU_ThrowByName(
                    env,
                    "java/lang/UnsatisfiedLinkError",
                    msg
                );

                if (!isBuiltin) {
                    JVM_UnloadLibrary(handle);
                }

                goto done;
            }

            (*env)->SetIntField(env, lib, jniVersionID, jniVersion);
        } else {
            cause = (*env)->ExceptionOccurred(env);

            if (cause) {
                (*env)->ExceptionClear(env);
                (*env)->SetLongField(env, lib, handleID, (jlong)0);
                (*env)->Throw(env, cause);
            }

            goto done;
        }
    }

    (*env)->SetLongField(env, lib, handleID, ptr_to_jlong(handle));
    loaded = JNI_TRUE;

done:
    JNU_ReleaseStringPlatformChars(env, name, cname);
    return loaded;
}
```

## `findJniFunction`

```c
static void *findJniFunction(
    JNIEnv *env,
    void *handle,
    const char *cname,
    jboolean isLoad
) {
    const char *onLoadSymbols[] = JNI_ONLOAD_SYMBOLS;
    const char *onUnloadSymbols[] = JNI_ONUNLOAD_SYMBOLS;
    const char **syms;
    int symsLen;
    void *entryName = NULL;
    char *jniFunctionName;
    int i;
    size_t len;

    // Check for JNI_On(Un)Load<_libname> function
    if (isLoad) {
        syms = onLoadSymbols;
        symsLen = sizeof(onLoadSymbols) / sizeof(char *);
    } else {
        syms = onUnloadSymbols;
        symsLen = sizeof(onUnloadSymbols) / sizeof(char *);
    }

    for (i = 0; i < symsLen; i++) {
        // cname + sym + '_' + '\0'
        if ((len = (cname != NULL ? strlen(cname) : 0) +
                   strlen(syms[i]) + 2) > FILENAME_MAX) {
            goto done;
        }

        jniFunctionName = malloc(len);

        if (jniFunctionName == NULL) {
            JNU_ThrowOutOfMemoryError(env, NULL);
            goto done;
        }

        buildJniFunctionName(syms[i], cname, jniFunctionName);
        // jniEntryName = _JNI_OnLoad

        entryName = JVM_FindLibraryEntry(handle, jniFunctionName);
        free(jniFunctionName);

        if (entryName) {
            break;
        }
    }

done:
    return entryName;
}
```

## `buildJniFunctionName`

```c
void buildJniFunctionName(
    const char *sym,
    const char *cname,
    char *jniEntryName
) {
    if (cname != NULL) {
        char *p = strrchr(sym, '@');

        if (p != NULL && p != sym) {
            // sym == _JNI_OnLoad@8
            strncpy(jniEntryName, sym, (p - sym));
            jniEntryName[(p - sym)] = '\0';

            // jniEntryName == _JNI_OnLoad
            strcat(jniEntryName, "_");
            strcat(jniEntryName, cname);
            strcat(jniEntryName, p);

            // jniEntryName == _JNI_OnLoad_cname@8
        } else {
            strcpy(jniEntryName, sym);
            strcat(jniEntryName, "_");
            strcat(jniEntryName, cname);
        }
    } else {
        strcpy(jniEntryName, sym);
    }

    return;
}
```

## `JVM_FindLibraryEntry` 中调用了 `dlsym()`

```c
JNIEXPORT void *JNICALL JVM_FindLibraryEntry(
    void *handle,
    const char *name
) {
    ...
    else {
        return dlsym(handle, name);
    }
    ...
}
```

然后在 `Onload` 里面写 `RegisterNatives` 方法就可以进行方法注册。

`RegisterNatives` 的第二个参数为 `JNINativeMethod` 类型，这个类型的定义为：

```c
typedef struct {
    const char *name;      // Java 层 native 函数名
    const char *signature;
    void *fnPtr;           // so 层对应的方法指针
} JNINativeMethod;
```

最后对 `native` 方法对应的方法对象进行赋值，和静态注册类似：

```c
nativeFunc = dvmCallJNIMethod;
insns = 找到的 so 中的方法地址;
```

最后可以总结一些：在静态注册的时候，感觉和 Windows 的 IAT 加载有点类似。一个是通过 `GetProAddress()` 查找 DLL 的 EAT 来实现，一个是通过 `dlsym` 寻找动态符号表。

动态注册的时候，是事先对结构体中的函数指针进行赋值，通过 `RegisterNatives` 进行注册。当注册好了之后，`native` 方法对应的方法对象的成员 `nativeFunc == dvmCallJNIMethod`，就不会再设置为 `dvmResolveNativeMethod`，而是直接执行 `vmCallJNIMethod`，调用 `insns` 的方法。

[[招生]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-9-28 20:48

被 margina1 编辑

原因：

[#其他](https://bbs.kanxue.com/forum-161-1-129.htm)

