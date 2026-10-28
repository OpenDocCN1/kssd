
```c
/*

 * Load native code from the specified absolute pathname.  Per the spec,

 * if we've already loaded a library with the specified pathname, we

 * return without doing anything.

 *

 * TODO? for better results we should absolutify the pathname.  For fully

 * correct results we should stat to get the inode and compare that.  The

 * existing implementation is fine so long as everybody is using

 * System.loadLibrary.

 *

 * The library will be associated with the specified class loader.  The JNI

 * spec says we can't load the same library into more than one class loader.

 *

 * Returns "true" on success. On failure, sets *detail to a

 * human-readable description of the error or NULL if no detail is

 * available; ownership of the string is transferred to the caller.

 */

bool dvmLoadNativeCode(const char* pathName, Object* classLoader,

        char** detail)

{

    SharedLib* pEntry;

    void* handle;

    bool verbose;

    /* reduce noise by not chattering about system libraries */

    verbose = !!strncmp(pathName, "/system", sizeof("/system")-1);

    verbose = verbose && !!strncmp(pathName, "/vendor", sizeof("/vendor")-1);

    if (verbose)

        ALOGD("Trying to load lib %s %p", pathName, classLoader);

    *detail = NULL;

    /*

     * See if we've already loaded it.  If we have, and the class loader

     * matches, return successfully without doing anything.

     */

    pEntry = findSharedLibEntry(pathName);

    if (pEntry != NULL) {

        if (pEntry->classLoader != classLoader) {

            ALOGW("Shared lib '%s' already opened by CL %p; can't open in %p",

                pathName, pEntry->classLoader, classLoader);

            return false;

        }

        if (verbose) {

            ALOGD("Shared lib '%s' already loaded in same CL %p",

                pathName, classLoader);

        }

        if (!checkOnLoadResult(pEntry))

            return false;

        return true;

    }

    /*

     * Open the shared library.  Because we're using a full path, the system

     * doesn't have to search through LD_LIBRARY_PATH.  (It may do so to

     * resolve this library's dependencies though.)

     *

     * Failures here are expected when java.library.path has several entries

     * and we have to hunt for the lib.

     *

     * The current version of the dynamic linker prints detailed information

     * about dlopen() failures.  Some things to check if the message is

     * cryptic:

     *   - make sure the library exists on the device

     *   - verify that the right path is being opened (the debug log message

     *     above can help with that)

     *   - check to see if the library is valid (e.g. not zero bytes long)

     *   - check config/prelink-linux-arm.map to ensure that the library

     *     is listed and is not being overrun by the previous entry (if

     *     loading suddenly stops working on a prelinked library, this is

     *     a good one to check)

     *   - write a trivial app that calls sleep() then dlopen(), attach

     *     to it with "strace -p <pid>" while it sleeps, and watch for

     *     attempts to open nonexistent dependent shared libs

     *

     * This can execute slowly for a large library on a busy system, so we

     * want to switch from RUNNING to VMWAIT while it executes.  This allows

     * the GC to ignore us.

     */

    Thread* self = dvmThreadSelf();

    ThreadStatus oldStatus = dvmChangeStatus(self, THREAD_VMWAIT);

    handle = dlopen(pathName, RTLD_LAZY);

    dvmChangeStatus(self, oldStatus);

    if (handle == NULL) {

        *detail = strdup(dlerror());

        ALOGE("dlopen(\"%s\") failed: %s", pathName, *detail);

        return false;

    }

    /* create a new entry */

    SharedLib* pNewEntry;

    pNewEntry = (SharedLib*) calloc(1, sizeof(SharedLib));

    pNewEntry->pathName = strdup(pathName);

    pNewEntry->handle = handle;

    pNewEntry->classLoader = classLoader;

    dvmInitMutex(&pNewEntry->onLoadLock);

    pthread_cond_init(&pNewEntry->onLoadCond, NULL);

    pNewEntry->onLoadThreadId = self->threadId;

    /* try to add it to the list */

    SharedLib* pActualEntry = addSharedLibEntry(pNewEntry);

    if (pNewEntry != pActualEntry) {

        ALOGI("WOW: we lost a race to add a shared lib (%s CL=%p)",

            pathName, classLoader);

        freeSharedLibEntry(pNewEntry);

        return checkOnLoadResult(pActualEntry);

    } else {

        if (verbose)

            ALOGD("Added shared lib %s %p", pathName, classLoader);

        bool result = false;

        void* vonLoad;

        int version;

        vonLoad = dlsym(handle, "JNI_OnLoad");

        if (vonLoad == NULL) {

            ALOGD("No JNI_OnLoad found in %s %p, skipping init", pathName, classLoader);

            result = true;

        } else {

            /*

             * Call JNI_OnLoad.  We have to override the current class

             * loader, which will always be "null" since the stuff at the

             * top of the stack is around Runtime.loadLibrary().  (See

             * the comments in the JNI FindClass function.)

             */

            OnLoadFunc func = (OnLoadFunc)vonLoad;

            Object* prevOverride = self->classLoaderOverride;

            self->classLoaderOverride = classLoader;

            oldStatus = dvmChangeStatus(self, THREAD_NATIVE);

            if (gDvm.verboseJni) {

                ALOGI("[Calling JNI_OnLoad for \"%s\"]", pathName);

            }

            version = (*func)(gDvmJni.jniVm, NULL);

            dvmChangeStatus(self, oldStatus);

            self->classLoaderOverride = prevOverride;

            if (version == JNI_ERR) {

                *detail = strdup(StringPrintf("JNI_ERR returned from JNI_OnLoad in \"%s\"",

                                              pathName).c_str());

            } else if (dvmIsBadJniVersion(version)) {

                *detail = strdup(StringPrintf("Bad JNI version returned from JNI_OnLoad in \"%s\": %d",

                                              pathName, version).c_str());

                /*

                 * It's unwise to call dlclose() here, but we can mark it

                 * as bad and ensure that future load attempts will fail.

                 *

                 * We don't know how far JNI_OnLoad got, so there could

                 * be some partially-initialized stuff accessible through

                 * newly-registered native method calls.  We could try to

                 * unregister them, but that doesn't seem worthwhile.

                 */

            } else {

                result = true;

            }

            if (gDvm.verboseJni) {

                ALOGI("[Returned %s from JNI_OnLoad for \"%s\"]",

                      (result ? "successfully" : "failure"), pathName);

            }

        }

        if (result)

            pNewEntry->onLoadResult = kOnLoadOkay;

        else

            pNewEntry->onLoadResult = kOnLoadFailed;

        pNewEntry->onLoadThreadId = 0;

        /*

         * Broadcast a wakeup to anybody sleeping on the condition variable.

         */

        dvmLockMutex(&pNewEntry->onLoadLock);

        pthread_cond_broadcast(&pNewEntry->onLoadCond);

        dvmUnlockMutex(&pNewEntry->onLoadLock);

        return result;

    }

}
```

做了一些常规的检查，不赘述了，可以看到 `version = (*func)(gDvmJni.jniVm, NULL);` 这里调用了 `JNI_OnLoad`，上一行是 `ALOGI("[Calling JNI_OnLoad for \"%s\"]", pathName);`，记录一下方便逆向时确定位置。

根据逆向经验 `.init(_array)` 段定义的内容是在 `JNI_OnLoad` 之前执行的，而 `dlopen` 是加载 SO 的函数可能会在这里执行 `.init`，看一下 `dlopen` 函数，它的定义在 `linker/dlfcn.cpp` # 63 行。

```c
void* dlopen(const char* filename, int flags) {

  ScopedPthreadMutexLocker locker(&gDlMutex);

  soinfo* result = do_dlopen(filename, flags);

  if (result == NULL) {

    __bionic_format_dlerror("dlopen failed", linker_get_error_buffer());

    return NULL;

  }

  return result;

}
```
