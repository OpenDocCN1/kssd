
# 细说So动态库的加载流程

## dlopen之内存装载

![](upload/attach/201911/540885_BRR58U5CP6HMAEG.png)

![](upload/attach/201911/540885_EASVK75UCJT8BGK.png)

可以看到，这里又是在调用`DexPathList`类下的`findLibrary`。关注`splitLibraryPath`方法，它返回了需要加载的动态库所在目录。

![](upload/attach/201911/540885_99TXECYNKZ8Z4XQ.png)

![](upload/attach/201911/540885_DVXGGERRB4DV4CT.png)

这里简单说下`splitLibraryPath`方法的作用，它是根据传进来的`libraryPath`和system property中`"java.library.path"`的属性值即`/vendor/lib:/system/lib`来构造出要加载的动态库所在目录列表。

``` Java
/**
 * Splits the given library directory path string into elements
 * using the path separator ({@code File.pathSeparator}, which
 * defaults to {@code ":"} on Android, appending on the elements
 * from the system library path, and pruning out any elements that
 * do not refer to existing and readable directories.
 */
private static File[] splitLibraryPath(String path) {
    // Native libraries may exist in both the system and
    // application library paths, and we use this search order:
    //
    // 1. this class loader's library path for application libraries
    // 2. the VM's library path from the system property for system libraries
    //
    // This order was reversed prior to Gingerbread; see http://b/2933456.
    ArrayList<File> result = splitPaths(path, System.getProperty("java.library.path"), true);
    return result.toArray(new File[result.size()]);
}

/**
 * Splits the given path strings into file elements using the path
 * separator, combining the results and filtering out elements
 * that don't exist, aren't readable, or aren't either a regular
 * file or a directory (as specified). Either string may be empty
 * or {@code null}, in which case it is ignored. If both strings
 * are empty or {@code null}, or all elements get pruned out, then
 * this returns a zero-element list.
 */
private static ArrayList<File> splitPaths(String path1, String path2,
                                          boolean wantDirectories) {
    ArrayList<File> result = new ArrayList<File>();
    splitAndAdd(path1, wantDirectories, result);
    splitAndAdd(path2, wantDirectories, result);
    return result;
}

/**
 * Helper for {@link #splitPaths}, which does the actual splitting
 * and filtering and adding to a result.
 */
private static void splitAndAdd(String searchPath, boolean directoriesOnly,
                                ArrayList<File> resultList) {
    if (searchPath == null) {
        return;
    }
    for (String path : searchPath.split(":")) {
        try {
            StructStat sb = Libcore.os.stat(path);
            if (!directoriesOnly || S_ISDIR(sb.st_mode)) {
                resultList.add(new File(path));
            }
        } catch (ErrnoException ignored) {
        }
    }
}
```

现在可以对动态链接库的加载路径做个总结了，系统默认的目录为`/vendor/lib`和`/system/lib`。当使用`System.loadLibrary`或`System.load`来加载一个共享库的时候，会将VM栈中的`ClassLoader`传入。之后调用`findLibrary`方法，在两个目录中去寻找指定的so文件：一个是构造`ClassLoader`时，传进来的那个`libraryPath`；另一个则是system property中`"java.library.path"`的属性值。也就是说，实际上是会在如下的3个目录中进行查找：

1. `/vendor/lib`
2. `/system/lib`
3. `/data/app-lib/包名-n`

对于`/data/app-lib/包名-n`这个路径，大家可能会比较陌生，但应该都知道`/data/data/包名/lib`目录，这里就简单讲解下apk安装过程中的一点细节，以说明二者之间的关系（在Android源码中的路径为`/frameworks/native/cmds/installd/commands.c`）

``` C
int install(const char *pkgname, uid_t uid, gid_t gid, const char *seinfo)
{
    char pkgdir[PKG_PATH_MAX];
    char libsymlink[PKG_PATH_MAX];
    char applibdir[PKG_PATH_MAX];
    struct stat libStat;

    if ((uid < AID_SYSTEM) || (gid < AID_SYSTEM)) {
        ALOGE("invalid uid/gid: %d %d\n", uid, gid);
        return -1;
    }

    if (create_pkg_path(pkgdir, pkgname, PKG_DIR_POSTFIX, 0)) { //创建包路径，"/data/data/包名"
        ALOGE("cannot create package path\n");
        return -1;
    }

    if (create_pkg_path(libsymlink, pkgname, PKG_LIB_POSTFIX, 0)) { //创建库路径，"/data/data/包名/lib"
        ALOGE("cannot create package lib symlink origin path\n");
        return -1;
    }

    if (create_pkg_path_in_dir(applibdir, &android_app_lib_dir, pkgname, PKG_DIR_POSTFIX)) { //创建"/data/app-lib/包名"目录
        ALOGE("cannot create package lib symlink dest path\n");
        return -1;
    }

    if (mkdir(pkgdir, 0751) < 0) {
        ALOGE("cannot create dir '%s': %s\n", pkgdir, strerror(errno));
        return -1;
    }

    if (chmod(pkgdir, 0751) < 0) {
        ALOGE("cannot chmod dir '%s': %s\n", pkgdir, strerror(errno));
        unlink(pkgdir);
        return -1;
    }

    if (lstat(libsymlink, &libStat) < 0) {
        if (errno != ENOENT) {
            ALOGE("couldn't stat lib dir: %s\n", strerror(errno));
            return -1;
        }
    } else {
        if (S_ISDIR(libStat.st_mode)) {
            if (delete_dir_contents(libsymlink, 1, 0) < 0) {
                ALOGE("couldn't delete lib directory during install for: %s", libsymlink);
                return -1;
            }
        } else if (S_ISLNK(libStat.st_mode)) {
            if (unlink(libsymlink) < 0) {
                ALOGE("couldn't unlink lib directory during install for: %s", libsymlink);
                return -1;
            }
        }
    }

    if (symlink(applibdir, libsymlink) < 0) {
        ALOGE("couldn't symlink directory '%s' -> '%s': %s\n", libsymlink, applibdir,
              strerror(errno));
        unlink(pkgdir);
        return -1;
    }

    if (selinux_android_setfilecon2(pkgdir, pkgname, seinfo, uid) < 0) {
        ALOGE("cannot setfilecon dir '%s': %s\n", pkgdir, strerror(errno));
        unlink(libsymlink);
        unlink(pkgdir);
        return -errno;
    }

    if (chown(pkgdir, uid, gid) < 0) {
        ALOGE("cannot chown dir '%s': %s\n", pkgdir, strerror(errno));
        unlink(libsymlink);
        unlink(pkgdir);
        return -1;
    }

    return 0;
}
```

以上代码会先构造几个目录名：`pkgdir`为`/data/data/包名`，`libsymlink`为`/data/data/包名/lib`，`applibdir`为`/data/app-lib/包名`。然后创建相应目录，并赋权限。之后，建立`/data/data/包名/lib`指向`/data/app-lib/包名`的符号链接。

现在再回过头来说明下`/data/app-lib/包名-n`、`/data/data/包名/lib`这二者之间的关系。在`/data/data/包名/`目录下执行`ls -l`命令，就会发现`lib`是一个链接，So其实是放在`/data/app-lib/包名-n`路径下的。

![](upload/attach/201911/540885_4Y4Y3SKJ7X6E7XS.png)

### Native 层的加载实现

`doLoad`实际上是调用本地的`nativeLoad`方法，`nativeLoad`会先更新`LD_LIBRARY_PATH`，然后执行`dvmLoadNativeCode`函数，真正实现so文件的加载。

``` C++
/*
 * static String nativeLoad(String filename, ClassLoader loader, String ldLibraryPath)
 *
 * Load the specified full path as a dynamic library filled with
 * JNI-compatible methods. Returns null on success, or a failure
 * message on failure.
 */
static void Dalvik_java_lang_Runtime_nativeLoad(const u4* args,
                                                JValue* pResult)
{
    StringObject* fileNameObj = (StringObject*) args[0];
    Object* classLoader = (Object*) args[1];
    StringObject* ldLibraryPathObj = (StringObject*) args[2];

    assert(fileNameObj != NULL);

    char* fileName = dvmCreateCstrFromString(fileNameObj);

    if (ldLibraryPathObj != NULL) {
        char* ldLibraryPath = dvmCreateCstrFromString(ldLibraryPathObj);
        void* sym = dlsym(RTLD_DEFAULT, "android_update_LD_LIBRARY_PATH");
        if (sym != NULL) {
            typedef void (*Fn)(const char*);
            Fn android_update_LD_LIBRARY_PATH = reinterpret_cast<Fn>(sym);
            (*android_update_LD_LIBRARY_PATH)(ldLibraryPath);
        } else {
            ALOGE("android_update_LD_LIBRARY_PATH not found; .so dependencies will not work!");
        }
        free(ldLibraryPath);
    }

    StringObject* result = NULL;
    char* reason = NULL;
    bool success = dvmLoadNativeCode(fileName, classLoader, &reason);
    if (!success) {
        const char* msg = (reason != NULL) ? reason : "unknown failure";
        result = dvmCreateStringFromCstr(msg);
        dvmReleaseTrackedAlloc((Object*) result, NULL);
    }

    free(reason);
    free(fileName);
    RETURN_PTR(result);
}
```

`dvmLoadNativeCode`定义在Android源码中的路径为`/dalvik/vm/Native.cpp`，它的主要功能如下：

1. 调用`findSharedLibEntry`方法，遍历查找已加载的lib。具体来说，就是先用待加载的lib路径名计算出一个32位hash值，然后遍历`gDvm`中的`nativeLibs`（其结构为`HashTable`用来保存加载的本地库），如果找到则返回一个`SharedLib`结构。这里如果LIB已被加载，则会对其加载的`ClassLoader`进行比较，JNI只允许同一个LIB只被一个`ClassLoader`加载；

![](upload/attach/201911/540885_N8EBF9UVW88P75N.png)

![](upload/attach/201911/540885_V2GVUWUGFT3KG59.png)

![](upload/attach/201911/540885_AM2ZJVPC7JRABWZ.png)

2. 调用`dlopen`打开一个so；

![](upload/attach/201911/540885_TV9QQ4WZWPGBNQA.png)

3. 将新加载的LIB插入到`gDvm`保存的链表中，执行`JNI_OnLoad`的调用。

![](upload/attach/201911/540885_NZSENMZPPV3QMXT.png)

![](upload/attach/201911/540885_5TDF922AY6A4WRH.png)

## 总结

在了解So在内存中的加载原理后，可以得知以下几点：

1. So的加载路径为：`/vendor/lib`、`/system/lib`、`/data/app-lib/包名-n`；
2. So的入口为`init_array`、`init_func`这些初始化函数。这部分在`dlopen`的过程中就会执行，再之后的是`JNI_OnLoad`方法的调用。这里面可以注册一些本地方法，也可以继续做些变量的初始化等操作；
3. 在So的加载流程中，最终会被存放到`SharedLib`这个结构体中，并添加到`nativeLibs`这个hash表下。

博客：00aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6A6M7X3g2Y4K9r3!0K6N6q4)9J5k6h3y4F1
