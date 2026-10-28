
重点在接口定义，后面几步都是 API 调用，对于一些有文档记录的接口一般都有对应的头文件或 IDL，直接用就行，但是对于其他 COM 接口，调用之前先要定义一个包含方法虚表的结构体/接口，这个虚表的内存偏移、方法顺序需要保证和接口实现一致，后面拿到接口指针才能正确调用对应的方法，C++ 里的接口定义示例：

```cpp
const IID IID_IRundown = { 0x00000134, 0x0000, 0x0000, {0xC0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x46}};

MIDL_INTERFACE("00000134-0000-0000-C000-000000000046")
IRundown : public IUnknown {
    STDMETHOD(RemQueryInterface)         ( REFIPID         ripid,
        ULONG           cRefs,
        USHORT          cIids,
        IID *iids,
        REMQIRESULT **ppQIResults);

    STDMETHOD(RemAddRef)                 ( unsigned short  cInterfaceRefs,
        REMINTERFACEREF InterfaceRefs[],
        HRESULT *pResults);

    STDMETHOD(RemRelease)                ( USHORT          cInterfaceRefs,
        REMINTERFACEREF InterfaceRefs[]);
};
```

所有 COM 接口的祖先就是 `IUnknown`，大部分接口直接继承自 `IUnknown`，还有部分通过继承 `IDispatch` 或其他接口间接的继承自 `IUnknown`。继承在内存布局上实际上就是在父类的内存结构基础上进行新增，所以不继承直接将 `IUnknown` 中的方法搬过来也行。

由于 Rust 里面接口、类全部都以 `struct` 的形式表达，并且和 C++ 中的 `struct` 内存布局是有区别的，所以在定义接口虚表时，全部需要加上 `#[repr(C)]`，代表该结构体内存布局和 C 完全一致。C 里面有 `IUnknown`，Rust 里也不需要我们从 `IUnknown` 开始实现，实际上在 `windows-rs` 和 `winapi` 这两个 crate 中都有实现，但是实现方式上有所不同。主要体现在对 “接口指针” 的定义上，下面是使用 C、`winapi`、`windows-rs` 各自如何声明一个接口指针变量：

|  | 声明方式 |
| --- | --- |
| C | `IUnknown *p = NULL;` |
| `winapi` | `let p: *mut IUnknown = ptr::null_mut();` |
| `windows-rs` | `let p: IUnknown;` |

可以看出来 `winapi` 的接口定义方式更符合 C 的接口调用风格，而 `windows-rs` 从声明上则看不出来是一个指针，指针被隐藏在了内部：

```rust
#[repr(transparent)]
pub struct IUnknown(std::ptr::NonNull<std::ffi::c_void>);
```

`transparent` 可以理解为透传，相当于：`pub type IUnknown = std::ptr::NonNull<std::ffi::c_void>`，所以 `let p: IUnknown` 等价于 `let p: std::ptr::NonNull<std::ffi::c_void>`，这样才能看出来是个指针了。

对于这块暂时解释到这里，想更进一步理解具体怎么用 Rust 定义一个接口的话，可以借鉴我代码里对 `IRundown` 接口的实现方式。

接下来理解 COM 接口方法的调用过程，COM 实际上可以理解为 RPC 的一种上层实现，所以还是 RPC，调用接口的程序称为客户端，真正处理执行调用请求的称为服务端。之前列出的调用过程步骤中的第 3 步，使用 `CoGetObject`、`CoCreateInstance`、`CoGetObjectContext` 这些 API 获取接口指针，如果获取成功就相当于和服务端连接成功，当通过指针调用方法后，相当于发起一个请求到服务端了。

所以回到该技术中，该技术使用了一个名为 `IRundown` 的接口，此接口中包含一个可以执行回调的方法 `DoCallback`，定义如下：

```rust
pub DoCallback: unsafe extern "system" fn(this: *mut ::core::ffi::c_void, pParam: *mut XAptCallback) -> windows::core::HRESULT,
```

`XAptCallback` 参数设置回调地址和参数地址：

```rust
#[repr(C)]
#[derive(Clone, Copy)]
pub struct tagXAptCallback {
    pub pfnCallback: PTRMEM, // what to execute. e.g. LoadLibraryA, EtwpCreateEtwThread
    pub pParam: PTRMEM, // parameter to callback.
    pub pServerCtx: PTRMEM, // combase!g_pMTAEmptyCtx
    pub pUnk: PTRMEM, // Not required
    pub iid: windows::core::GUID, // Not required
    pub iMethod: i32, // Not required
    pub guidProcessSecret: windows::core::GUID // combase!CProcessSecret::s_guidOle32Secret
}
pub type XAptCallback = tagXAptCallback;
```

`pfnCallback` 为回调函数指针，`pParam` 为参数指针。加上之前说的 C/S 架构，接口调用请求实际上是在服务端处理的，所以当服务端进程接收到执行回调的请求后，触发回调执行完成代码注入。

大致的技术利用原理就这些，其他的都是一些细节问题，比如如何获取到该接口指针、如何注入到任意进程中去，这两个实际上是一个问题，前面说过成功获取接口指针即是连接到目标进程，所以对于此类问题的根本是 “哪些进程属于这个接口的服务进程”。

好像目前唯一好用的查看 COM 进程信息的工具就是 [OleViewDotNet](https://bbs.kanxue.com/elink@b97K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1P5i4u0S2L8X3W2V1i4K6u0r3L8$3I4W2N6X3W2W2N6$3c8G2N6r3&6W2N6l9`.`.) 了，需要提前使用 `windbg` 或者其他调试器把 `combase.dll` 的符号下载到本地，然后配置到 OleViewDotNet 里，否则是查不到任何结果的：

![x](https://github.com/0xlane/com-process-inject/blob/master/assets/Snipaste_2023-01-17_15-14-29.png?raw=true)

然后在 Processes -> All Proccess -> By Name 打开 COM 进程列表，搜索 `IRundown`：

![x](https://github.com/0xlane/com-process-inject/blob/master/assets/Snipaste_2023-01-17_15-11-08.png?raw=true)

相当于执行 `com-inject.exe list -v`。

这些进程中存在 `IRundown` 接口指针，由于 `IRundown` 接口的实现者是 `combase.dll`，所以加载 `combase.dll` 的进程都有可能。

`windbg` 里面可以直接按下面的方式找 `IRundown` 接口虚表：

|  |  |
| --- | --- |
