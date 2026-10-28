
有两个点，`class_loader == NULL`，和非空的情况。不晓得这两种是什么情况。大神可以指教下。第一个过程直接走 `dlopen`。第二个过程，走 `android_dlopen_ext`，两个函数最终都会走到 `do_dlopen` 来打开 `so`。

```
void* OpenNativeLibrary(JNIEnv* env, int32_t target_sdk_version, const char* path, jobject class_loader, jstring library_path, bool* needs_native_bridge, std::string* error_msg) {
#if defined(__ANDROID__)
UNUSED(target_sdk_version);
if (class_loader == nullptr) {
*needs_native_bridge = false;
return dlopen(path, RTLD_NOW);
}
std::lock_guard<std::mutex> guard(g_namespaces_mutex);
NativeLoaderNamespace ns;

if (!g_namespaces->FindNamespaceByClassLoader(env, class_loader, &ns)) {
if (!g_namespaces->Create(env, target_sdk_version, class_loader, false /* is_shared */, false /* is_for_vendor */, library_path, nullptr, &ns, error_msg)) {
return nullptr;
}
}

if (ns.is_android_namespace()) {
android_dlextinfo extinfo;
extinfo.flags = ANDROID_DLEXT_USE_NAMESPACE;
extinfo.library_namespace = ns.get_android_ns();

void* handle = android_dlopen_ext(path, RTLD_NOW, &extinfo);
if (handle == nullptr) {
*error_msg = dlerror();
}
*needs_native_bridge = false;
return handle;
} else {
void* handle = NativeBridgeLoadLibraryExt(path, RTLD_NOW, ns.get_native_bridge_ns());
if (handle == nullptr) {
*error_msg = NativeBridgeGetError();
}
*needs_native_bridge = true;
return handle;
}
#else
UNUSED(env, target_sdk_version, class_loader, library_path);
*needs_native_bridge = false;
void* handle = dlopen(path, RTLD_NOW);
if (handle == nullptr) {
if (NativeBridgeIsSupported(path)) {
*needs_native_bridge = true;
handle = NativeBridgeLoadLibrary(path, RTLD_NOW);
if (handle == nullptr) {
*error_msg = NativeBridgeGetError();
}
} else {
*needs_native_bridge = false;
*error_msg = dlerror();
}
}
return handle;
#endif
}
```

```
void* do_dlopen(const char* name, int flags, const android_dlextinfo* extinfo, const void* caller_addr) {
  // 在此之前都是针对，flag 和extinfo flag的处理。
  const char* translated_name = name;
  if (g_is_asan && translated_name != nullptr && translated_name[0] == '/') {
    char original_path[PATH_MAX];
    if (realpath(name, original_path) != nullptr) {
      asan_name_holder = std::string(kAsanLibDirPrefix) + original_path;
      LD_LOG(kLogDlopen,"... pareto's here realpath =\"%s\"",name);
      if (file_exists(asan_name_holder.c_str())) {
        soinfo* si = nullptr;
        if (find_loaded_library_by_realpath(ns, original_path, true, &si)) {
          PRINT("linker_asan dlopen NOT translating \"%s\" -> \"%s\": library already loaded", name, asan_name_holder.c_str());
        } else {
          PRINT("linker_asan dlopen translating \"%s\" -> \"%s\"", name, translated_name);
          translated_name = asan_name_holder.c_str();
        }
      }
    }
  }

  ProtectedDataGuard guard;
  // 关注点 find_library
  soinfo* si = find_library(ns, translated_name, flags, extinfo, caller);
  loading_trace.End();

  if (si != nullptr) {
    void* handle = si->to_handle();
    // 调用init
    si->call_constructors();
    return handle;
  }

  return nullptr;
}
```

`find_library` 内部调用重载函数 `find_library`，内部的重载函数就是本次分析的重点。

```
static soinfo* find_library(android_namespace_t* ns, const char* name, int rtld_flags, const android_dlextinfo* extinfo, soinfo* needed_by) {
  soinfo* si;

  // readers_map is shared across recursive calls to find_libraries.
  // However, the map is not shared across different threads.
  std::unordered_map<const soinfo*, ElfReader> readers_map;
  if (name == nullptr) {
    si = solist_get_somain();
  } else if (!find_libraries(ns, needed_by, &name, 1, &si, nullptr, 0, rtld_flags, extinfo, false /* add_as_children */, true /* search_linked_namespaces */, readers_map)) {
    return nullptr;
  }
  // 增加so的索引计数
  si->increment_ref_count();

  return si;
}
```

关键函数到位了
