
# PE LOADER，可运行 MS 自带的程序

> 原文：[`www.pediy.com/kssd/pediy12/141891.html`](https://www.pediy.com/kssd/pediy12/141891.html)

之前放着的代码，没啥用，干脆把以前的文档整理下发出来。

下面就是在实现一个 `PE LOADER` 过程中碰到的问题以及一些解决办法，更详细的请自己阅读代码，这个 `PE LOADER` 可以跑 `WINDOWS` 的大部分程序，在 `XP` 下几乎 `MS` 的程序都可以跑，包括 `NOTEPAD`, `REGEDIT`, 计算器等。

主要的问题如下

1）`FormatMessage`
2）`GetModuleFileName`
3）`msvcrt` 与参数
4）`TLS`
5）捕获输出
6）地址被占用的问题
7）进程退出
8）命令行
9）重定向

`Loader` 要做的事情

1）读取磁盘文件到内存
2）处理导入表，重定位表，资源表
3）`CALL` 入口函数

因为是加载 `EXE` 模块，`XP` 的话默认加载基地址就是 `PE` 头中的 `ImageBase`，这个地址空间一般状态是 `MEM_FREE`，

`XP`

```
0:000> !address 0x01000000
    003a3000 : 003a3000 - 5fc4d000
                Type     00000000
                Protect  00000001 PAGE_NOACCESS
                State    00010000 MEM_FREE
                Usage    RegionUsageFree
```

所以申请这地址开始的内存成功概率很大，这种情况下 `Loader` 会变得比较简单，只需要处理导入表就可以，而 `WIN 7` 因为采用了 地址随机分布 机制，`EXE` 加载的地址不一定是 `ImageBase`，`ImageBase` 开始的内存已经被占用了

`WIN 7`

```
Usage:               MemoryMappedFile
Allocation Base:     00870000
Base Address:        0096d000
End Address:         01470000
Region Size:         00b03000
Type:                00040000 MEM_MAPPED
State:               00002000 MEM_RESERVE
Protect:             00000000
Mapped file name:    PageFile
```

强制使用的后果是造成莫名的崩溃。

在 `XP` 为了让程序尽可能的在其默认加载的基地址运行，

需要减低 `ImageBase` 跟我们的 `EXE` 地址冲突，解决办法就是我们编译的模块选一个加载的基地址。

这里我选的是 `0x5FFF0000`，这个地址就可以避免跟要 `RUN` 的 `EXE` 模块的基地址冲突了，另外为了能在 `WIN 7` 跑，还需要 Disable Image Randomization (`/DYNAMICBASE:NO`)。为了避免 `DEP` 而导致出错，所以分配的内存的属性一律改为 `PAGE_EXECUTE_READWRITE`

## 1）`FormatMessage`

`XP` 很多程序的字符串都放在资源中，这样只要访问资源就可以了。但 `WIN 7` 中，很多字符串都是存在 `.MUI` 文件中而不是资源中。

因为微软的控制台程序几乎都会调用 `FormatMessage`，所以这个函数如果失败，后面的功能就得不到执行。在 `WIN 7` 中需要对这个 API 进行处理，因为它没有去访问资源而是 `.MUI` 文件。`MUI` 文件一般放在 `SYSTEM32\ZH-CN\` 目录下

，文件对应的 `.MUI` 文件可以通过 `GetFileMUIPath` 来获取

注:如果操作系统是 英文的或者其他语言的，那就不是 `ZH-CN` 了，具体可以通过 API

```
BOOL SetThreadPreferredUILanguages(
  DWORD  dwFlags,
  PCWSTR pwszLanguagesBuffer,
  PULONG pulNumLanguages
);
```

来获取。

`.MUI` 加载的时机

`.MUI` 加载的时机是比较早的，也就是在进入 `MAIN` 之前被 `LDR` 给加载了。以 `arp` 为例子，当运行 `arp.exe` 时，`LdrpRunInitializeRoutines` 就会加载 `arp.exe.mui` 这个文件，然后才会进入到 `MAIN` 函数里面。

问题来了，因为我们自己加载要运行的模块，并没有走 `LdrpRunInitializeRoutines`，自然我们进程空间就没有相应的 `.mui` 文件。解决办法就是需要自己 `LoadLibraryEx(RUN_IMAGE_FILE_PATH, NULL, NULL);` 一下。。最主要就是获取 `.MUI` 文件的全路径。

```
DWORD WINAPI FormatMessage(
  __in         DWORD dwFlags,
  __in         LPCVOID lpSource,
  __in         DWORD dwMessageId,
  __in         DWORD dwLanguageId,
  __out        LPTSTR lpBuffer,
  __in         DWORD nSize,
  __in         va_list* Arguments
);
```

对访问 `.MUI` 获取字符串来说，`dwFlags` 必须是要有 `FORMAT_MESSAGE_FROM_HMODULE`

，所以这个标志是我们判断的重要依据。`MS` 自己的 `lpSource` 传入为 `NULL`，这样它会自动访问 `.MUI` 文件。而如果 `.MUI` 是我们自己加载的，那传入 `NULL` 无疑会导致失败，没能找到资源。所以需要 `HOOK` 这个 API 把 `lpSource` 改成我们 `LOAD .MUI` 文件的内存地址。

## 2）`GetModuleFileName`

崩溃

```
7282f0ef ff766c          push    dword ptr [esi+6Ch]
7282f0f2 ff1568838372    call    dword ptr [MFC42u!_imp__GetModuleFileNameW (72838368)]
7282f0f8 8d8584010000    lea     eax,[ebp+184h]
7282f0fe 6a2e            push    2Eh
7282f100 50             push    eax
7282f101 ff1558878372    call    dword ptr [MFC42u!_imp__wcsrchr (72838758)]
7282f107 66832000        and     word ptr [eax],0         ds:0023:00000000=????
```

`eax = 0`，可见 `GetModuleFileNameW` 返回的是 `NULL`，而微软又没有对这个函数的返回值进行判断。所以导致崩溃。

为什么 `GetModuleFileNameW` 会崩溃呢？

`GetModuleFileName` 的定义

```
DWORD WINAPI GetModuleFileName(
  __in         HMODULE hModule,
  __out        LPTSTR lpFilename,
  __in         DWORD nSize
);
```

跟进去看堆栈

```
0:000> kvnf
 #    Memory  ChildEBP RetAddr  Args to Child
00     0012f650 7282f0f8 01000000 0012f874 00000104 kernel32!GetModuleFileNameW (FPO: [Non-Fpo])
```

传入的 `hModule` 的值是 `01000000`，注意这个值不是我们主程序的基地址，而是被 `RUN` 的 `EXE` 的基地址。

接着看

`GetModuleFileName` 内部实现：

```
.text:7C80B458 loc_7C80B458:                       ; CODE XREF: GetModuleFileNameW(x,x,x)+85j
.text:7C80B458                 mov     [ebp+var_38], esi
.text:7C80B45B                 mov     ecx, [ebp+var_24]
.text:7C80B45E                 cmp     ecx, [esi+18h]
.text:7C80B461                 jz      short loc_7C80B481
.text:7C80B463                 mov     esi, [esi]
.text:7C80B465
.text:7C80B465 loc_7C80B465:                       ; CODE XREF: GetModuleFileNameW(x,x,x)+54j
.text:7C80B465                 mov     [ebp+var_2C], esi
.text:7C80B468                 cmp     esi, eax
.text:7C80B46A                 jnz     short loc_7C80B458
..........
.text:7C80B413 loc_7C80B413:                       ; CODE XREF: GetModuleFileNameW(x,x,x)+1C9j
.text:7C80B413                         ; GetModuleFileNameW(x,x,x)+35979j
.text:7C80B413                 lea     eax, [ebp+var_1C]
.text:7C80B416                 push    eax
.text:7C80B417                 push    edi
.text:7C80B418                 push    1
.text:7C80B41A                 call    _LdrLockLoaderLock@12 ; LdrLockLoaderLock(x,x,x)
.text:7C80B41F                 mov     [ebp+ms_exc.disabled], edi
.text:7C80B422                 mov     eax, large fs:18h ; TEB
.text:7C80B428                 mov     [ebp+var_30], eax
.text:7C80B42B                 mov     eax, [eax+30h] ; PEB
.text:7C80B42E                 mov     eax, [eax+0Ch] ; _PEB_LDR_DATA
.text:7C80B431                 add     eax, 0Ch ; nLoadOrderModuleList : _LIST_ENTRY
.text:7C80B434                 mov     [ebp+var_34], eax
.text:7C80B437                 mov     esi, [eax]
.text:7C80B439                 jmp     short loc_7C80B465
```

`nLoadOrderModuleList` 链表中成员的结构是：

```
typedef struct _LDR_MODULE
{
    LIST_ENTRY          InLoadOrderModuleList;   +0x00
    LIST_ENTRY          InMemoryOrderModuleList; +0x08
    LIST_ENTRY          InInitializationOrderModuleList; +0x10
    void*               BaseAddress;  +0x18
    void*               EntryPoint;   +0x1c
    ULONG               SizeOfImage;
    UNICODE_STRING      FullDllName;
    UNICODE_STRING      BaseDllName;
    ULONG               Flags;
    SHORT               LoadCount;
    SHORT               TlsIndex;
    HANDLE              SectionHandle;
    ULONG               CheckSum;
    ULONG               TimeDateStamp;
} LDR_MODULE, *PLDR_MODULE;
```

可见 `GetModuleFileName` 是通过遍历 `LoadOrderModuleList`，然后看 `hModule` 跟 `LDR_MODULE::BaseAddress` 相等，相等的话就取出里面的 `FullDllName`。但由于我们传入的 `hModule` 是被处理过的，所以自然在链表里找不到导致返回 `NULL`，而返回之后 `MS` 又不做判断就直接引用返回值，导致崩溃。

## 3）`msvcrt` 与参数

`WIN 7 ARP.EXE`

```
.text:0100239E                push    dword_1005968
.text:010023A4                push    dword_1005960
.text:010023AA                call    _main
```

`msvcrt.dll` 第一次被加载的时候，会调用 `GetCommandLine` 来获取当前参数，然后进行处理填充 `msvcrt` 的内部变量

`msvcrt!__argc` 和 `msvcrt!__argv`。之后通过 `__getmainargs`, `__wgetmainargs` 就可以获取到相应参数，这时候并不会再走 `GetCommandLine`。`WIN 7` 的 `ARP.EXE` 获取命令行参数就是通过 `__getmainargs`。因为 `msvcrt.dll` 在进程运行的时候已经被加载，所以无法拦截到 `GetCommandLine`. 俩个参数在 `LOAD msvcrt.dll` 时被初始化。。`WIN 7` 每个进程一运行就会加载这个 DLL.所以需要自己 `HOOK __getmainargs`, `__wgetmainargs`

## 4）`TLS`

```
0100887a 648b0d2c000000  mov     ecx,dword ptr fs:[2Ch]
01008881 56              push    esi
01008882 8d3481          lea     esi,[ecx+eax*4]
01008885 57              push    edi
01008886 8b3e            mov     edi,dword ptr [esi]  ds:0023:00000000=????????
01008888 83bf0400000000  cmp     dword ptr [edi+4],0
```

因为是 `mov ecx,dword ptr fs:[2Ch]` 访问 `TEB` 的 `ThreadLocalStoragePointer`， 如果我们没模拟 `LOADER` 初始话得话就会失败。。。所以需要自己分析 `TLS` 目录获取数据，填到内存，然后修改 `ThreadLocalStoragePointer` 指针。其实只需要分配合适空间大小就可以。。里面的内容程序自己会添加，如果你的程序没使用 `TLS`，那你的线程的 `ThreadLocalStoragePointer` 就为 `NULL`，显然如果要跑的程序使用 `TLS`，那访问该地方就崩了
