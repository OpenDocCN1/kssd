
# [原创]从Frida源码学习ArtHook

为了填[AppInspect](https://bbs.pediy.com/thread-269004-1.htm)的坑，开始研究ArtHook。

从Frida入手是因为的资料多，更新频繁。

坏处是Frida体系庞大，比其他单纯的Hook框架复杂。

Frida的native hook代码在`frida-gum`，虽然是C语言，但用了gobject框架，对没接触过的人需要一定时间习惯。

Frida的art hook代码在`frida-java-bridge`，纯js代码实现。

通过`enumerateLoadedClasses`的执行流程，作为入口开始代码走读。

第一步是初始化Runtime，这个Runtime是Art Runtime在js侧的一个代理，是Frida ArtHook的核心。

通过`getApi()`获取Art Runtime的Api接口。

怎么理解获取Api呢，

Android程序运行时，art运行时会以`libart.so`等动态库的形式加载在当前进程的地址空间中。那我们能不能像访问`libc.so`里的函数一样访问里面的函数呢？

答案是肯定的，

从CPU运行流程看，调用一个函数就是，准备好函数的参数，跳到函数地址执行。

这里只有两个问题要解决：

1. 如何知道函数的参数。
2. 如何知道函数的地址。

只是如果你在ndk的代码里调用c库的函数，头文件解决了参数问题，而函数地址，在编译时和运行时由链接器帮你填好。

但对于`libart.so`我们只能自己动手了，好在Linux有一套运行时加载so的机制。

```c
void* handle = dlopen("./hello.so", RTLD_LAZY);
typedef void (*hello_t)();
hello_t hello = (hello_t) dlsym(handle, "hello");
hello();
```

但在Android 7以后，限制了App直接调用dlopen打开`libart.so`等系统库，

```c
void* handle = dlopen("libart.so", RTLD_LAZY);
LOGD("dlopen %p",handle); //返回0
```

作为例外`libc.so`, `libandroid.so`等在白名单里的不受影响。白名单在`/system/etc/public.libraries.txt`

除了白名单，还有一个在代码里写死的灰名单，需在编译App时把`targetSdk`设定为23之前才能有效。

那么怎么绕过这个限制等呢：

有两种方法：

1. 我们要调用`dlopen`/`dlsym`的目的，并不是真的要加载so到内存，而是想获取函数在内存中的地址。Linux系统（包括Android）会把加载动态库的地址范围信息，通过`/proc/$pid/maps`文件暴露出来，只要读取这个文件就可以获取到so的基址，然后可通过解析elf头，获取到各个函数的地址信息。

2. 不直接调用libc的`dlopen`，而是使用android系统库`libdl.so`的`__loader_dlopen`，

```c
void* __loader_dlopen(const char* filename, int flags, const void* caller_addr) {
return dlopen_ext(filename, flags, nullptr, caller_addr);
}
```

跟踪调用，真正干活的是linker中的`do_dlopen`函数，它会根据`__loader_dlopen`传进来的`caller_addr`，检查调用者所属的模块，然后根据模块查找其对应命名空间，判断是否允许加载。

```c
soinfo* const caller = find_containing_library(caller_addr);
android_namespace_t* ns = get_caller_namespace(caller);
```

所以只要传一个系统库中的函数，就能加载其他在同一个命名空间里的系统模块。

如何找一个跟目标模块命名空间相同的地址呢,

方法1，调用`dl_iterate_data`

> The **dl_iterate_phdr**() function walks through the list of an application's shared objects and calls the function *callback* once for each object,

参考快手的KOOM

原理就是调用`dl_iterate_phdr`后，会在`callback`里返回App加载的所有模块信息，当找到目标模块时，把它基址保存下来，作为`__loader_dlopen`的`caller_addr`参数。

方法2，通过`/proc/self/maps`文件获取到目标模块的基址作为`caller_addr`。

另外，还有一个更简单的方法是传一个`libc`里的函数地址，但Android Q以后增加了新的命名空间，`libc`和`libart`并不在同一个命名空间里。

Rust版代码验证

```rust
type __loader_dlopen_t = unsafe extern "C" fn(*const c_char, i32, *const c_void) -> *const c_void;
unsafe fn test_dlopen() -> anyhow::Result<()> {
//获取libdl.so中的__loader_dlopen地址（libdl.so在白名单中不受限）
let libdl = libc::dlopen("libdl.so".to_c_string().as_ptr(), libc::RTLD_LAZY);
log::debug!("libdl: {:?}", libdl);
let __loader_dlopen_ptr = libc::dlsym(libdl, "__loader_dlopen".to_c_string().as_ptr());
let __loader_dlopen: __loader_dlopen_t = std::mem::transmute(__loader_dlopen_ptr);
//通过proc maps获取libart.so的基址
let module_info = util_rs::proc_map::get_module_info("libart.so", "/proc/self/maps")
.context("fail to parse map proc!")?;
let ptr = module_info.range_start as *const c_void;
log::debug!("find libart.so start ptr: {:p}", ptr);
//libart.so的基址作为caller_addr调用__loader_dlopen
let handle = __loader_dlopen("libart.so".to_c_string().as_ptr(), libc::RTLD_LAZY, ptr);
log::debug!("__loader_dlopen : {:?}", handle);
//直接使用dlsym查找符号表
let sym = libc::dlsym(
handle as *mut c_void,
"JNI_GetCreatedJavaVMs".to_c_string().as_ptr(),
);
log::debug!("sym: {:?}", sym);
Ok(())
}
```

看到这里，可能会有个疑问，为什么谷歌这么弱，弄个安全机制这么简单就被破解了，

看看官方对链接器命名空间的介绍,

链接器命名空间解决的问题是：可以隔离不同链接器命名空间中的共享库，以确保具有相同库名称和不同符号的库不会发生冲突。

所以它不是一种安全机制，只是为了安全的解析符号，前面的操作只是对这种限制的规避，不能算漏洞，谷歌也不会去修复，所以预估能够长期使用。

回到Frida，再看看Frida是怎么解决这个dlopen限制的

因为手头只有Android 10的设备，这里只关注Android29的版本

`frida-gum`在里查找`__dl___loader_dlopen`（地址就是`libdl.so`的`__loader_dlopen`），如果没有找到会通过搜索内存对比指令集特征的方式继续查找。但在Android 10上，这一步就找到了，之后的操作和前述`__loader_dlopen`方式一致。

`gum_linker_api_try_init`

理解了这些基本原理后，后面再回到`frida-java-bridge`，看看js端如何访问Android 运行时。

参考：

https://bbs.pediy.com/thread-257022.htm
