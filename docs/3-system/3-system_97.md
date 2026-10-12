
# 翻译 TLS 处理

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-658.htm`](https://www.pediy.com/kssd/pediy07/pediy7-658.htm)

英语又退步了唉，不准就不准吧凑或看。

支持 TLS

原文：Writing Your Own Packer - by BigBoote

TLS（Thread Local Storage，线程局部存储）是一种便利的编程机制。我们通常不使用，因此并不太关心。但是要压缩的原程序可能会用到它。事实上，Delphi 总是使用它，如果我们打算支持 Delphi 程序，最好兼容它。

TLS 基本上是通过 API 实现。大致过程是，你分配一个“ Index（索引）”并存储在一个全局变量中。通过这个 Index 获得针对每个线程的一个双字值。通常使用这个值保存一个为每个线程分配好的内存块的指针。人们认为这样很乏味，一个特殊机制的出现使得实现它更容易些。因此，你可以这样写代码：

```
__declspec ( thread ) int tls_int_value = 0;
```

每个线程可以通过名称访问它独特的实例，就像访问其他变量一样。我不知道这种 TLS 形式是否有官方名称，所以我叫它“简化 TLS”。这种机制与操作系统兼容，并且 PE 文件中有对应的结构。这些结构包含在数据目录的一个块中：

```
origdirinfo[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress
```

问题是， 处理这信息发生在由 OS 在每个线程的创建时刻，在执行到线程开始地址之前。（这句翻译的拗口，原文 The problem is that the processing of this information happens by the OS on the creation of every thread prior to execution being passed to the thread start address. ）这通常不牵涉到我们，除了至少一个线程在我们解压数据之前被执行：我们的线程！我们必须设置一个伪 TLS 处理区段捕获 OS 在我们开始之前做的事情，然后在最后一步把这个信息传递给原程序。

为此，我在外部加壳器外部全局结构添加了 2 个项目：

```
GlobalExternVars
{
 //(other stuff we previously described)
 IMAGE_TLS_DIRECTORY tls_original;
 IMAGE_TLS_DIRECTORY tls_proxy;
 };
```

加壳器将会在运行期复制原始数据到 `tls_orginal` 为我们所用。`tls_proxy` 几乎是一个精确的副本，除了 2 个项目将会被修改：

```
tls_proxy.AddressOfIndex
tls_proxy.AddressOfCallBacks
```

在这个块中我们将要初始化 `AddressOfIndex` 指向一个正常的全局双字变量，并且我们将初始化 `AddressOfCallBacks` 指向一个函数指针数组。它是一个线程创建时将会被调用的函数指针列表。用户使用它定义 TLS 对象的初始化。唉，我没见过一个编译器使用它。此外，在 9x 下，这些函数不会被调用。尽管如此，我们还是要支持它以防万一哪天它会被使用。我们令 `AddressOfCallbacks` 指向一个 2 个成员的数组，一个试我们将要执行的函数指针，另一个是 NULL 作为列表结束符。

设置一个全局双字存储 TLS slot（槽？）

```
DWORD TLS_slot_index;
```

TLS 回调函数必须是这种原型：

```
extern "C" void NTAPI TLS_callback ( PVOID DllHandle, DWORD Reason, PVOID Reserved );
```

当然还要添加两个逻辑标志表示是否可以安全地调用原来的回调函数，和是否延期调用。这样初始化它们：

```
bool safe_to_callback_tls = false;
bool delayed_tls_callback = false;
```

再提供一些变量保存延迟调用的数据：

```
PVOID TLS_dll_handle = NULL;
DWORD TLS_reason = 0;
PVOID TLS_reserved = NULL;
```

编写回调函数：

```
extern "C" void NTAPI TLS_callback ( PVOID DllHandle, DWORD Reason, PVOID Reserved )
{
        if ( safe_to_callback_tls )
        {
                PIMAGE_TLS_CALLBACK* ppfn = g_pkrdat.m_tlsdirOrig.AddressOfCallBacks;
                if ( ppfn )
                {
                        while ( *ppfn )
                        {
                        (*ppfn) ( DllHandle, Reason, Reserved );
                        ++ppfn;
                        }
                }

        }
        else
        {
                delayed_tls_callback = true;
                TLS_dll_handle = DllHandle;
                TLS_reason = Reason;
                TLS_reserved = Reserved;
        }
}
```

这样会为 OS 提供一个存储 slot 信息的地方，我们稍候恢复它，并且如果调用了我们的回调函数我们将捕获参数，在解压缩之后调用原来的回调函数。否则会出错因为 0S 会在我们有机会解压缩之前做这件事情。解压缩之后，我们把参数传递给原来的回调函数。

最后一步是这样的：

```
void FinalizeTLSStuff()
{
        if ( origdirinfo[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress != 0 )
        {
                *gev.tls_original.AddressOfIndex = TLS_slot_index;
                void* TLS_data;

                __asm
                {
                mov ecx, DWORD PTR TLS_slot_index;
                mov edx, DWORD PTR fs:[02ch]
                mov ecx, DWORD PTR [edx+ecx*4]
                mov pvTLSData, ecx
                }

                int size = gev.tls_original.EndAddressOfRawData - gev.tls_original.StartAddressOfRawData;

                memcpy ( pvTLSData, (void*) gev.tls_original.StartAddressOfRawData, size );
                memset ( (void*) gev.tls_original.EndAddressOfRawData, 0,
                gev.tls_original.SizeOfZeroFill );
        }

        safe_to_callback_tls = true;
        if ( delayed_tls_callback )
        {
                TLSCallbackThunk ( TLS_dll_handle TLS_reason TLS_reserved );
        }
}
```

 4dd466d0e1e0cc123afbe504da2462ca
