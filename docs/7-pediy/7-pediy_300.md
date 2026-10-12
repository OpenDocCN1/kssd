
# DIY PPS 退出之前终结 PPSAP 进程,还我内存,还我网络

> 原文：[`www.pediy.com/kssd/pediy12/144382.html`](https://www.pediy.com/kssd/pediy12/144382.html)

**【文章标题】**: DIY PPS 退出之前终结 PPSAP 进程

**【软件名称】**: PPS

**【加壳方式】**: 无壳

**【使用工具】**: OD+VC

**【操作平台】**: WIN-XP

**【软件介绍】**: PPS

**【作者声明】**: 只是感兴趣，没有其他目的。请各位多多指教

无聊的时候看电影,一般用 `pps` 的人不少,但是对于只有 2G 内存的本本来讲,任何程序占用多余的内存,都是属于奢侈的事情,所以这次对 `pps` 的进程分析了一下,启动 `pps` 只有一个主进程`PPStream.exe`,在看电影的过程中,会多启动一个进程`PPSAP.exe`,最受不了的就是,`pps` 退出后,`PPSAP` 进程一直霸占着内存,不释放,人家不释放,咱得动点邪恶的念头.

**描述：**

*   `ppstream` 每次启动都会另开一个进程“网络加速器”(`ppsap.exe`) ，

    (1) 每次开机都会自动运行(可手动禁止）,

    (2) 退出 `ppstream` 之后会继续存在，

    (3) 可手动在任务管理器里结束，

    (4) 自动访问网络。也就是说，即使你退出 `ppstream` 了，你的电脑还在为其他观看者提供网络流量。占用一定带宽 。

以上描述来源百度百科,可以用网络流量监控软件,试试,退出后该进程是否还会上传数据,

1. 占内存
2. 占带宽

这次 diy 的目的是,在退出 `PPStream.exe` 的时候,也把 `ppsap.exe` 进程干掉,

为了简写代码,用 `PostMessage` 来关闭该进程.

### `PostMessage` 函数说明:

*   **函数功能**：该函数将一个消息放入（寄送）到与指定窗口创建的线程相联系消息队列里，不等待线程处理消息就返回，是异步消息模式。消息队列里的消息通过调用 `GetMessage` 和 `PeekMessage` 取得。
*   **函数原型**：`BOOL PostMessage(HWND hWnd, UINT Msg, WPARAM wParam, LPARAM lParam)`
*   **参数**
    *   `hWnd`：其窗口程序接收消息的窗口的句柄。可取有特定含义的两个值：
        *   `HWND_BROADCAST`：消息被寄送到系统的所有顶层窗口，包括无效或不可见的非自身拥有的窗口、被覆盖的窗口和弹出式窗口。消息不被寄送到子窗口。
        *   `NULL`：此函数的操作和调用参数 `dwThread` 设置为当前线程的标识符 `PostThreadMessage` 函数一样。
    *   `Msg`：指定被寄送的消息。
    *   `wParam`：指定附加的消息特定的信息。
    *   `lParam`：指定附加的消息特定的信息。
*   **返回值**：如果函数调用成功，返回非零值：如果函数调用失败，返回值是零。若想获得更多的错误信息，请调用 `GetLastError` 函数。

为了关闭该进程,先要获取到 `hWnd`,

### 使用 `FindWindow`

```c
HWND FindWindow(
  LPCSTR lpClassName,
  LPCSTR lpWindowName
);
```

*   **`lpClassName`**
    指向一个以 null 结尾的、用来指定类名的字符串或一个可以确定类名字符串的原子。如果这个参数是一个原子，那么它必须是一个在调用此函数前已经通过 `GlobalAddAtom` 函数创建好的全局原子。这个原子（一个 16bit 的值），必须被放置在 `lpClassName` 的低位字节中，`lpClassName` 的高位字节置零。

    如果该参数为 null 时，将会寻找任何与 `lpWindowName` 参数匹配的窗口。

*   **`lpWindowName`**
    指向一个以 null 结尾的、用来指定窗口名（即窗口标题）的字符串。如果此参数为 NULL，则匹配所有窗口名。

简单介绍了一下这 2 个 API,开始实战了,先用 `FindWindow` 的 `lpWindowName`,使用 VS 自带的工具,`spy++`,`Ctrl+P` 快捷键,找到 `ppsap.exe` 进程,发现标题为`PPStream 网络加速器`

用 API 大概可以这样写

```cpp
::PostMessage(::FindWindow(NULL, "PPStream 网络加速器") ,WM_CLOSE,0,0);
```

现在最关键的是将这句话翻译为汇编,并加入到 `PPStream.exe` 里面,且在退出的时候走一下刚加的代码就完成了.

第一,翻译为汇编,用 VC 编译一下.用 od 载入,把汇编的二进制抠出来就行了

如果熟悉汇编,也可以手动写

以下用内联汇编写,调试环境 VC6.0

```cpp
CString tmp;
tmp = "PPStream 网络加速器";

HMODULE hMod = LoadLibrary("user32.dll");
FARPROC Find = GetProcAddress(hMod,"FindWindowA");
FARPROC Post = GetProcAddress(hMod,"PostMessageA");

__asm
{
  mov ebx,tmp
  push ebx
  push 0
  call Find  ;调用 FindWindowA

  push 0
  push 0
  push 16
  push eax   ;eax 是 FindWindow 返回的结果
  call Post  ;调用 PostMessageA
}

FreeLibrary(hMod);
```

用 od 载入 `PPStream.exe`,找一段代码,把对应的汇编写进去,

最关键的问题就是怎么定位 `FindWindowA` `PostMessageA` 的地址.

使用 od 的 `bpx hmemcpy` 命令,把调用的 api 都找出来,看看有没有 `FindWindowA` `PostMessageA` 的地址.

如果存在,就不用再找了,幸运的是里面有这 2 个 API 地址.

接下来就是,找到退出的地方,一般程序退出使用的 `ExitProcess`

下面是本机上的

```asm
00631610    /E9 F0701100   jmp     00748709                    ;  ExitProcess //把这里改成上面新加汇编代码地址,执行完 PostMessage 再跳回 00631615
00631615  . |FF75 08       push    dword ptr [ebp+8]
00631618  . |E8 C8FFFFFF   call    006315E5
0063161D  . |59            pop     ecx
0063161E  . |FF75 08       push    dword ptr [ebp+8]            ; /ExitCode
00631621  . \FF15 68937400 call    dword ptr [<&KERNEL32.ExitProces>; \ExitProcess
```

以下代码是找了一段空白的地址,加入的关闭 `ppsap` 进程功能,注意 `007486EA`

```asm
00748709     6A 00         push    0
0074870B     6A 00         push    0
0074870D     6A 10         push    10
0074870F     68 EA867400   push    007486EA            //这个值就是存放"PPStream 网络加速器",也就是 FindWindow 的参数
00748714     6A 00         push    0
00748716     FF15 7C987400 call    dword ptr [<&USER32.FindWindowA>>;  USER32.FindWindowA
0074871C     50           push    eax
0074871D     FF15 049A7400 call    dword ptr [<&USER32.PostMessageA>;  USER32.PostMessageA
00748723     90           nop
00748724     90           nop
00748725     90           nop
00748726     90           nop
00748727     8BFF         mov     edi, edi    //修复一下上面 jmp 占用的几个字节.
00748729     55           push    ebp
0074872A     8BEC         mov     ebp, esp
0074872C     90           nop
0074872D     90           nop
0074872E     90           nop
0074872F     90           nop
00748730     90           nop
00748731     90           nop
00748732     90           nop
00748733     90           nop
00748734     90           nop
00748735     90           nop
00748736     90           nop
00748737    ^ E9 D98EEEFF   jmp     00631615            //跳回原始 ExitProcess 的空间....
```

基本能达到目的了.

在退出 `pps` 窗口之后,`ppsap` 进程都会被干掉了.

如有什么问题或错误,欢迎各位拍砖,谢谢
