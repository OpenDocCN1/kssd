
# linker 中 `dlopen` 与 `soinfo::to_handle()` 分析

```cpp
void* soinfo::to_handle() {
  if (get_application_target_sdk_version() <= 23 || !has_min_version(3)) {
    return this;
  }
  return reinterpret_cast<void*>(get_handle());
}
```

`bionic/linker/linker.cpp`

```cpp
void* do_dlopen(const char* name, int flags, const android_dlextinfo* extinfo,
                void* caller_addr) {
  soinfo* const caller = find_containing_library(caller_addr);

  if ((flags & ~(RTLD_NOW|RTLD_LAZY|RTLD_LOCAL|RTLD_GLOBAL|RTLD_NODELETE|RTLD_NOLOAD)) != 0) {
    DL_ERR("invalid flags to dlopen: %x", flags);
    return nullptr;
  }

  android_namespace_t* ns = get_caller_namespace(caller);
  ...
  ...
  ...
  ProtectedDataGuard guard;
  soinfo* si = find_library(ns, translated_name, flags, extinfo, caller);
  if (si != nullptr) {
    si->call_constructors();
    return si->to_handle();
  }

  return nullptr;
}
```

`dlopen` 函数最终会调用到 linker 中的 `do_dlopen`，`do_dlopen` 的返回值为 `soinfo->to_handle()`。

`void* soinfo::to_handle()` 这个函数中明确了 target sdk <= 23 返回 `this`（即 `soinfo`），否则会通过 `get_handle` 返回一个随机值。

来看看 `soinfo` 结构体：

省略了很多，可以查看源码（`bionic/linker/linker.h`）

```cpp
struct soinfo {
  ...
  ...
  ...
 public:
  const ElfW(Phdr)* phdr;
  size_t phnum;
  ElfW(Addr) entry;
  ElfW(Addr) base;
  size_t size;

#if defined(__work_around_b_24465209__)
  uint32_t unused1; // DO NOT USE, maintained for compatibility.
#endif

  ElfW(Dyn)* dynamic;

#if defined(__work_around_b_24465209__)
  uint32_t unused2; // DO NOT USE, maintained for compatibility
  uint32_t unused3; // DO NOT USE, maintained for compatibility
#endif

  soinfo* next;
 private:
  uint32_t flags_;

  const char* strtab_;
  ElfW(Sym)* symtab_;
  ...
  ...
#if defined(USE_RELA)
  ElfW(Rela)* plt_rela_; // 重定位表（函数）
  size_t plt_rela_count_;

  ElfW(Rela)* rela_; // 重定位表（数据）
  size_t rela_count_;
#else
  ElfW(Rel)* plt_rel_;
  size_t plt_rel_count_;

  ElfW(Rel)* rel_;
  size_t rel_count_;
#endif
```

`soinfo` 中可以拿到 `phdr*`（程序头表）、`phnum`（程序头个数）、`base`（装载基址）、`size`（装载所需空间）等等。

## 移植 linker 源码，仿照重定位过程修复 GOT 表数据

linker 源码文件结构图，便于大家理解关键文件

![图片描述](https://bbs.kanxue.com/upload/attach/202105/855618_YMG3MYM6DETUSM3.png)

目前只简单实现了 arch64 的 GOT 表重定位 hook。下面是一部分 demo 源码

```cpp
static int __open_2_hook(const char* path, int flags) {
  LOGD("__open_2_hook %s", path);
  return open(path, flags);
}

static void testHookOpen(JNIEnv* env, jclass clazz, jstring jPath) {

  const char* cPath = env->GetStringUTFChars(jPath, JNI_FALSE);
  hook_info info = {
      "__open_2",
      (void*)__open_2_hook,
      nullptr
  };
  relocateHook(cPath, &info);
  env->ReleaseStringUTFChars(jPath, cPath);

}
```

```cpp
bool relocateHook(const char* libpath, hook_info* info) {
  ElfSoinfo* elfSoinfo = ElfSoinfo::create(libpath);
  elfSoinfo->relocate(info);
  return false;
}
```
