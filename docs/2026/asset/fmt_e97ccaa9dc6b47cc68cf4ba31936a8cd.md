
## NativeLibrary 加载流程

```private NativeLibrary loadLibrary(Class<?> fromClass, String name, boolean isBuiltin){
...
try {
if (!lib.open()) {
return null; // fail to open the native library
}
...
}
```
```boolean open() {
if (handle != 0) {
throw new InternalError("Native library " + name + " has been loaded");
}
return load(this, name, isBuiltin, isJNI);
}
```

然后发现居然最后调用了一个 native :

```private static native boolean load(NativeLibraryImpl impl, String name, boolean isBuiltin, boolean isJNI);
```

### NativeLibraries.c

```Java\_jdk\_internal\_loader\_NativeLibraries\_load
(JNIEnv \*env, jobject this, jobject lib, jstring name, jboolean isBuiltin, jboolean isJNI)
{
const char \*cname;
jint jniVersion;
jthrowable cause;
void \* handle;
jboolean loaded = JNI\_FALSE;
if (!initIDs(env))
return JNI\_FALSE;
cname = JNU\_GetStringPlatformChars(env, name, 0);
if (cname == 0)
return JNI\_FALSE;
handle = isBuiltin ? procHandle : JVM\_LoadLibrary(cname);
if (isJNI) {
if (handle) {
JNI\_OnLoad\_t JNI\_OnLoad;
JNI\_OnLoad = (JNI\_OnLoad\_t)findJniFunction(env, handle,
isBuiltin ? cname : NULL,
JNI\_TRUE);
if (JNI\_OnLoad) {
JavaVM \*jvm;
(\*env)->GetJavaVM(env, &jvm);
jniVersion = (\*JNI\_OnLoad)(jvm, NULL); //应该是执行JNI\_Onload的地方
} else {
jniVersion = 0x00010001;
}
cause = (\*env)->ExceptionOccurred(env);
if (cause) {
(\*env)->ExceptionClear(env);
(\*env)->Throw(env, cause);
if (!isBuiltin) {
JVM\_UnloadLibrary(handle);
}
goto done;
}
if (!JVM\_IsSupportedJNIVersion(jniVersion) ||
(isBuiltin && jniVersion < JNI\_VERSION\_1\_8)) {
char msg[256];
jio\_snprintf(msg, sizeof(msg),
"unsupported JNI version 0x%08X required by %s",
jniVersion, cname);
JNU\_ThrowByName(env, "java/lang/UnsatisfiedLinkError", msg);
if (!isBuiltin) {
JVM\_UnloadLibrary(handle);
}
goto done;
}
(\*env)->SetIntField(env, lib, jniVersionID, jniVersion);
} else {
cause = (\*env)->ExceptionOccurred(env);
if (cause) {
(\*env)->ExceptionClear(env);
(\*env)->SetLongField(env, lib, handleID, (jlong)0);
(\*env)->Throw(env, cause);
}
goto done;
}
}
(\*env)->SetLongField(env, lib, handleID, ptr\_to\_jlong(handle));
loaded = JNI\_TRUE;
done:
JNU\_ReleaseStringPlatformChars(env, name, cname);
return loaded;
}
```

### findJniFunction

```static void \*findJniFunction(JNIEnv \*env, void \*handle,
const char \*cname, jboolean isLoad) {
const char \*onLoadSymbols[] = JNI\_ONLOAD\_SYMBOLS;
const char \*onUnloadSymbols[] = JNI\_ONUNLOAD\_SYMBOLS;
const char \*\*syms;
int symsLen;
void \*entryName = NULL;
char \*jniFunctionName;
int i;
size\_t len;
// Check for JNI\_On(Un)Load<\_libname> function
if (isLoad) {
syms = onLoadSymbols;
symsLen = sizeof(onLoadSymbols) / sizeof(char \*);
} else {
syms = onUnloadSymbols;
symsLen = sizeof(onUnloadSymbols) / sizeof(char \*);
}
for (i = 0; i < symsLen; i++) {
// cname + sym + '\_' + '\0'
if ((len = (cname != NULL ? strlen(cname) : 0) + strlen(syms[i]) + 2) >
FILENAME\_MAX) {
goto done;
}
jniFunctionName = malloc(len);
if (jniFunctionName == NULL) {
JNU\_ThrowOutOfMemoryError(env, NULL);
goto done;
}
buildJniFunctionName(syms[i], cname, jniFunctionName); //jniEntryName = \_JNI\_OnLoad
entryName = JVM\_FindLibraryEntry(handle, jniFunctionName);
free(jniFunctionName);
if(entryName) {
break;
}
}
done:
return entryName;
}
```

### buildJniFunctionName

```void buildJniFunctionName(const char \*sym, const char \*cname,
char \*jniEntryName) {
if (cname != NULL) {
char \*p = strrchr(sym, '@');
if (p != NULL && p != sym) {
// sym == \_JNI\_OnLoad@8
strncpy(jniEntryName, sym, (p - sym));
jniEntryName[(p-sym)] = '\0';
// jniEntryName == \_JNI\_OnLoad
strcat(jniEntryName, "\_");
strcat(jniEntryName, cname);
strcat(jniEntryName, p);
//jniEntryName == \_JNI\_OnLoad\_cname@8
} else {
strcpy(jniEntryName, sym);
strcat(jniEntryName, "\_");
strcat(jniEntryName, cname);
}
} else {
strcpy(jniEntryName, sym);
}
return;
}
```

`JVM_FindLibraryEntry` 中调用了 `dlsym()`

```JNIEXPORT void\* JNICALL JVM\_FindLibraryEntry(void\* handle, const char\* name) {
...
else {
return dlsym(handle, name);
}
...
}
```

然后在 `Onload` 里面写 `RegisterNatives` 方法就可以进行方法注册:

`RegisterNatives` 的第二个参数为 `JNINativeMethod` 类型, 这个类型的定义为:

```typedef struct {
const char\* name; //Java层native函数名
const char\* signature;
void\* fnPtr; //so层对应的方法指针
} JNINativeMethod;
```

最后对 native 方法对应的方法对象进行赋值, 和静态注册类似

`nativeFunc = dvmCallJNIMethod`, `insns =` 找到的 so 中的方法地址.

最后可以总结一些, 在静态注册的时候, 感觉和 windows 的 iat 加载有点类似, 一个是通过 `GetProAddress()` 查找 DLL 的 EAT 来实现, 一个是通过通过 `dlsym` 寻找动态符号表. 动态注册的时候, 是事先对结构体中的函数指针进行赋值, 通过 `RegisterNatives` 进行注册. 当注册好了之后, native 方法对应的方法对象的成员 `nativeFunc == dvmCallJNIMethod`, 就不会在再设置为 `dvmResolveNativeMethod`, 而是直接执行 `vmCallJNIMethod`, 调用 `insns` 的方法.

# android JNI静态注册和动态注册

这几天在摸 android 的题目, 在 XCTF 里面做了一个 ONLOAD 来动态注册 native 的题目, 但是本身 ida 的 export 能够找到对应 native 函数名. 想找找 ONLOAD 注册方法和直接调用的区别, 就仔细去翻了一些相关的帖子.

这个 OnLoad 注册了一个名为 `CheckFlag` 的方法.

![](https://bbs.kanxue.com/upload/attach/202109/931103_5ZGAJTX4Q6H6BDU.png)

native 方法分为静态注册和动态注册两种. (我的环境是 java16)

因为注册的过程不太好调, 可能会有问题, 如果有错误请师傅们提出来, 我好及时改进.

## 静态注册

java 的每个方法都会有一个 `Method` 对象, 而这个 `Method` 对象有两个成员: `nativeFunc` 和 `insns`.

当 java 构建这个 native 方法时, 会将该方法对应的 `Method` 对象的 `nativeFunc` 成员设置为 `dvmResolveNativeMethod`, `dvmResolveNativeMethod` 方法会按照方法的名称去查找对应的 C 方法.

|  |
| --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58 |
```
void dvmResolveNativeMethod(const u4* args, JValue* pResult,
const Method* method, Thread* self)
{
ClassObject* clazz = method->clazz;
/*
* If this is a static method, it could be called before the
* class
* has been initialized.
*/
if (dvmIsStaticMethod(method)) {
if (!dvmIsClassInitialized(clazz) && !dvmInitClass(clazz)) {
assert(dvmCheckException(dvmThreadSelf()));
return;
}
}
else {
assert(dvmIsClassInitialized(clazz) ||
dvmIsClassInitializing(clazz));
}

/*
start with our internal-native methods
*/
DalvikNativeFunc infunc = dvmLookupInternalNativeMethod(method);
if (infunc != NULL) {
/*
resolution always gets the same answer, so no race here
*/
IF_LOGVV() {
char* desc = dexProtoCopyMethodDescriptor(&method->prototype);
LOGVV("+++ resolved native %s.%s %s, invoking",
clazz->descriptor, method->name, desc);
free(desc);
}
if (dvmIsSynchronizedMethod(method)) {
ALOGE("ERROR: internal-native can't be declared 'synchronized'");
ALOGE("Failing on %s.%s", method->clazz->descriptor, method->name);
dvmAbort();
/ / harsh, but this is VM-internal problem
}
DalvikBridgeFunc dfunc = (DalvikBridgeFunc) infunc;
dvmSetNativeFunc((Method*) method, dfunc, NULL);
dfunc(args, pResult, method, self);
return;
}

/*
now scan any DLLs we have loaded for JNI signatures
*/
void* func = lookupSharedLibMethod(method);
if (func != NULL) {
/*
found it, point it at the JNI bridge
and then call it
*/
dvmUseJNIBridge((Method*) method, func);
(*method->nativeFunc)(args, pResult, method, self);
return;
}

IF_ALOGW() {
char* desc = dexProtoCopyMethodDescriptor(&method->prototype);
ALOGW("No implementation found for native %s.%s:%s",
clazz->descriptor, method->name, desc);
free(desc);
}

dvmThrowUnsatisfiedLinkError("Native method not found", method);
}
```

`lookupSharedLibMethod` 函数用来查找 SO 层里面的方法, 再看下 `lookupSharedLibMethod` 代码

|  |
| --- |
| 1  2  3  4  5  6  7  8  9 |
```
static void* lookupSharedLibMethod(const Method* method)
{
if (gDvm.nativeLibs == NULL) {
ALOGE("Unexpected init state: nativeLibs not ready");
dvmAbort();
}
return (void*) dvmHashForeach(gDvm.nativeLibs, findMethodInLib,
(void*) method);
}
```

`dvmHashForeach` 的注释是

|  |
| --- |
