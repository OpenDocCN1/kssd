
# 例说 Exe 程序作为 DLL 进行加载

> 原文：[`www.pediy.com/kssd/pediy10/68730.html`](https://www.pediy.com/kssd/pediy10/68730.html)

## 1. 背景与方法

调用第三方 `exe` 程序里面的函数，一直是大家所向往并已经讨论过不少的问题，其方法大体有三类：

1. 让第三方 `exe` 启动，然后自己程序注入进去调用之；
2. 让第三方 `exe` 启动，然后远程读入其内容；
3. 把第三方 `exe`，当作 `DLL` 进行加载，并调用里面的函数。

前 2 个方法容易实现，但无法摆脱让 `exe` 运行的缺点。今天我们讨论第三条思路，把 `exe` 向 `dll` 一样加载，然后调用里面的函数。

就此，主要面临以下两个问题：

1. 导入表修复；
2. 重定位 `dll` 数据，也就是 `exe` 数据。

其实这 2 个问题归结起来都是重定位问题，这里就不再解释重定位原理和原因了。有些人认为 `PE` 文件重定位，就要搞重定位表，`exe` 要输出函数，就要搞导出表，这些其实是不必要的。只要理解了他们的原理，自己实现反而更方便。当然，另外一点原因是我现在基本忘却 `PE` 结构了。以下方法不涉及给 `exe` 增加导出表或者重定位表。

### 问题示例

举例来说碰到的问题和解决思路，`exe` 里面有如下代码：

```
push    425570                    ; /kernel32.dll
call    dword ptr [425280]        ; GetModuleHandleA
```

这个很正常，也很简单。`0x425570` 指向一个字符串，`[425280]` 里面是 `GetModuleHandleA` 函数指针，也就是导入表内容。但如果把这个 `exe` 作为 `DLL` 进行 `LoadLibrary`，你会发现这 2 行代码仍然如此，一点也没变，但却无法执行了。因为这时候涉及到的这 2 个指针，指向的内容已经不是我们预想的了，需要把它们重定位，才能让他们指向预期目的地。重定位方法也很简单：

1. `0x425570`，这个地址，用当前自身模块基地址加上 `0x25570`，就是新的地址；
2. `0x425280`，这个地址，用当前自身模块基地址加上 `0x25280`，就是新的地址。

理解了这一点，就知道我们需要作什么了。

## 2. 实例说明

口说无凭，动手为真，下面我们举例来进行说明。就采用壳狼最近写的 `antidebugger` 测试程序吧，当然没经过他的同意 :)

**目的**：加载 `AntiDebug.exe`，调用它的第一个标签里面的 `Find Debugger` 功能，里面有 20 多个选项，我们争取把它调用完！

**调用函数**：

简单跟踪以下，他这些反调试手段都在一个函数体内，调用方式如下：

```
00404FCC   .  50        push    eax
00404FCD   .  8BCB      mov     ecx, ebx        //这句无所谓，可以 nop
00404FCF   .  E8 ECF1FFFF   call    004041C0
```

实现方式也很简单：

```cpp
hMod = LoadLibrary(ExePath);
DWORD FindDebugerCall = (DWORD)hMod + 0x41C0;

_asm{
    push 0xFFFFFFFF        //这里是调用标记，如果为这个参数，表示所有的反调试功能都选了
    mov  eax, FindDebugerCall
    call eax;
}
```

最终我们的目的就是要让上面这个函数正常运行。为此，我们要作以下大量工作。

### 一、修复输入表

思路很简单，首先根据被加载 `dll` 的模块基地址，找到其导入表，然后根据其函数名，自己获取导入函数地址，重新填入到正常位置。

在此感谢鸡蛋壳，让我在茫茫网海搜到了他的一些代码并加以改之。

```cpp
pDosHeader = (PIMAGE_DOS_HEADER)hMod;
pNTHeaders = (PIMAGE_NT_HEADERS)((BYTE *)hMod + pDosHeader->e_lfanew);
pOptHeader = (PIMAGE_OPTIONAL_HEADER)&(pNTHeaders->OptionalHeader);
pImportDescriptor = (PIMAGE_IMPORT_DESCRIPTOR)((BYTE *)hMod + pOptHeader->DataDirectory[1].VirtualAddress);

while(pImportDescriptor->FirstThunk)
{
    //获取 dll 名称
    char * dllname = (char *)((BYTE *)hMod + pImportDescriptor->Name);
    pThunkData = (PIMAGE_THUNK_DATA)((BYTE *)hMod + pImportDescriptor->OriginalFirstThunk);
    int no = 1;

    while(pThunkData->u1.Function)
    {
        if ((pThunkData->u1.Ordinal &  IMAGE_ORDINAL_FLAG) != IMAGE_ORDINAL_FLAG)
        {
            //获取函数名称
            char *funname = (char *)((BYTE *)hMod + (DWORD)pThunkData->u1.AddressOfData + 2);
            myaddr = (int*)GetProcAddress(GetModuleHandle(dllname), funname);
        }

        PDWORD lpAddr = (DWORD *)((BYTE *)hMod + (DWORD)pImportDescriptor->FirstThunk) +(no-1);
        MEMORY_BASIC_INFORMATION  mbi;
        VirtualQuery(lpAddr,&mbi,sizeof(mbi));
        VirtualProtect(lpAddr,sizeof(DWORD),PAGE_READWRITE,&dwOLD);
        WriteProcessMemory(GetCurrentProcess(),
            lpAddr, &myaddr, sizeof(DWORD), NULL);
        VirtualProtect(lpAddr,sizeof(DWORD),dwOLD,0);
        no++;
        pThunkData++;
    }
    pImportDescriptor++;
}
```

这个东西搞完，`DLL`（也就是加载的 `exe`，后面都叫 `dll`）里面的 `API` 和其他导入函数可以正常使用了。

### 二、重定位代码段指针

重定位意义比较广泛，大体有三类数据需要重定位。

1.  **代码段指针**。比如我上面举例的 `push 425570`，这个 `0x425570` 是个字符串指针。再比如 `mov edi, dword ptr [425400]`，这里面的 `0x425400`，也是指针，也需要重定位；
2.  **数据段指针**。比如这个 `exe` 里面，用到了一些 `SEH` 结构，`SEH` 结构里面含有指向函数体的指针，比如：

    `SEH` 异常处理结构：

    ```
    0042BA94  E0 1C 40 00 E6 1C 40 00 00 00 00 00 FE FF FF FF  ?@.?@.....?
    0042BAA4  00 00 00 00 D8 FF FF FF 00 00 00 00 FE FF FF FF  ....?....?
    0042BAB4  28 20 40 00 2E 20 40 00 00 00 00 00 FE FF FF FF  ( @.. @.....?
    0042BAC4  00 00 00 00 D4 FF FF FF 00 00 00 00 FE FF FF FF  ....?....?
    0042BAD4  95 24 40 00 9B 24 40 00 00 00 00 00 FE FF FF FF  ?@.?@.....?
    0042BAE4  00 00 00 00 D8 FF FF FF 00 00 00 00 FE FF FF FF  ....?....?
    0042BAF4  C8 28 40 00 CE 28 40 00 00 00 00 00 E4 FF FF FF  ?@.?@.....?
    0042BB04  00 00 00 00 B8 FF FF FF 00 00 00 00 FE FF FF FF  ....?....?
    ```

    这段数据，在数据段，里面有诸如 `0x00401CE0` 之类的指针，指向函数体，也需要重定位。
3.  **其他一些野指针**。这个就需要自己去调试找了。

#### 如何查找需要重定位的数据

如何找上面三类数据，这是个问题，首先看一些需要重定位的例子：

```
00407D5F   B9 402A4300      mov     ecx, 00432A40
00408137   8B3D 90534200    mov     edi, dword ptr [425390]
00D88131   FF15 78544200    call    dword ptr [425478]
```

上面的三条，开头是代码地址，中间是 opcode，后面是指令。我们需要把 `00432A40` `425390` `425478` 这 3 个地址重新转换成新的地址达到重定位的目的。他们的特点，是都是 `0x004xxxxx`，也就是大小在 `exe` 的代码段范围内，也就是 `exe` 的基地址加上模块大小。

因此我们可以查找每条 `mov`、`call` 指令，判断一下立即数，发现有在这个范围内的，就认为该条需要重定位。但所需要处理的汇编指令和类型是在太多了，其他还有 `cmp` `je` `add` ....

所以我的思路，就是把程序里面每条指令都搜索以下，发现有类似常量，就把它揪出来供重定位。

不可否认这个思路很挫，但实际也很有效。为此我找到了裸葱裸大虾的 `OD` 插件 `ustrref` 的代码，把这个很挫的思路应用在他的代码上。事实正明任何很挫的方法，只要跟裸大虾联系起来，就不仅不挫反而很有效了。

对其 `StrFinder.cpp` 里面的代码进行修改，大体如下：

```cpp
    if (    //这里只列举这几条指令情况，不全面
    (0 != memicmp(da.result, "mov", 3)) &&
    (0 != memicmp(da.result, "cmp", 3)) &&
    (0 != memicmp(da.result, "call", 4)) &&
    (0 != memicmp(da.result, "push", 4))
    )
    continue;

    ip1 = 0;
    ip2 = 0;
    for (int j = 0; j < MAXCMDSIZE; j++)
    {
        Readmemory(&pConst, da.ip + j, 4, MM_RESTORE | MM_SILENT);
        if ((pConst> 0x400000) && (pConst < 0x441000))
        {
            if (ip1 == 0)
            {
                ip1 = da.ip + j;
                sprintf((char*)pszStr, "0x%X  ,", ip1);
                pCallBack(nStrIndex++, dwBase, dwOffset, dwSize, StrType, (char *)pszStr);
                continue;
            }
            if (ip2 ==0)
            {
                ip2 = da.ip + j;
                sprintf((char*)pszStr, "0x%X  ,", ip2);
                pCallBack(nStrIndex++, dwBase, dwOffset, dwSize, StrType, (char *)pszStr);
                break;
            }
        }
    }
```

修改后的代码很挫，因此得到的结果也很挫，大体如下：

```
Address    Disassembly                                                  Text String
00401006   mov     eax, dword ptr [42FB98]                              0x401007  ,
00401021   push    esi                                                  0x40102A  ,
00401022   push    eax                                                  0x40102A  ,
00401023   call    00412A80                                             0x40102A  ,
00401028   mov     edi, dword ptr [<&KERNEL32.GetVersionExA>]   0x40102A  ,
00401120   push    00425570                                             0x401121  ,
00401120   push    00425570                                             0x401127  ,
00401125   call    dword ptr [<&KERNEL32.GetModuleHandleA>]     0x401127  ,
00401125   call    dword ptr [<&KERNEL32.GetModuleHandleA>]     0x40112C  ,
0040112B   push    0042555C                                             0x40112C  ,
0040112B   push    0042555C                                             0x401133  ,
            .......................................
```

总共 9476 条，里面还有很多重复的，过滤一下剩下 2803 条。后面的 `Text String`，是需要重定位的数据地址。以第一条为例：

```
00401006   mov     eax, dword ptr [42FB98]                              0x401007  ,
```

表示 `0x401007` 这个内存处的数据 `0x42FB98` 需要重新定位以下。

当然这么多数据，里面有很多不能随便修改的，譬如：

```
0040439A  F7C3 00400000  test    ebx, 4000
```

按照我上面的思路，把这个地方也搜索出来了，因为里面有 `0x04307C`，但实际上这个地方不能修改，所以我们手工挑出来删除就可以了。

另外还有比如：

```cpp
//这个地方是短跳转，也不能修改
```



三、重定位数据段数据

上面说了对付代码段内容，但对付数据段，就没这么简单了。比如我上面提到的 `SEH` 结构，这个地方的内存数据是 `exe` 运行起来才加载处理的，没有好的办法找出来，只有靠自己去调试。

拿上面 `SEH` 例子来说，编译后运行，会出错，错误地点定位在：

```
004043A2  |.  E8 F9F0FFFF   call    004034A0    //这个函数里面出错  
004043A7  |.  84C0          test    al, al  
004043A9  |.  74 0E         je      short 004043B9  
004043AB  |.  6A 00         push    0  
004043AD  |.  6A 10         push    10  
004043AF  |.  68 24664200   push    00426624                      ;  debugger is found by fd_find_debugger_window!  
004043B4  |.  E8 9A900000   call    0040D453
```

上面得 `call 004034A0`，功能是“`found by fd_find_debugger_window!`”，跟踪以下，他主要是强行关闭一个句柄，然后跳到一个异常处理的地方用到了某些 `SEH` 结构。跟踪，下面这段代码可能是在设置某个异常处理结构：

```
0040E289  /$  8BC1          mov     eax, ecx  
0040E28B  |.  33D2          xor     edx, edx  
0040E28D  |.  33C9          xor     ecx, ecx  
0040E28F  |.  C700 14884200 mov     dword ptr [eax], 00428814  //这里 0x00428814 指向一个结构，里面含有函数地址指针  
0040E295  |.  8950 34       mov     dword ptr [eax+34], edx  
0040E298  |.  8950 54       mov     dword ptr [eax+54], edx  
0040E29B  |.  8948 4C       mov     dword ptr [eax+4C], ecx  
0040E29E  |.  8950 50       mov     dword ptr [eax+50], edx  
0040E2A1  \.  C3            retn
```

`0x00428814` 指向内容：

```
00428814  83 E3 40 00 D8 B1 42 00 C8 E3 40 00 24 B2 42 00  @.乇 B.茹@.$.  
00428824  E6 E4 40 00 70 B2 42 00 25 E7 40 00 BC B2 42 00  驿@.p.%.疾 B.  
00428834  25 E7 40 00                                 %.c
```

我们把 `0x00428814` 这个地方所有看起来貌似地址一样的数据统统重定位就可以了。

其他还有很多地方，有的是被错误重定位了，有的是没有重定位，这些地方都需要靠人工调试去找出来。由于我们的目标程序本身就是反调试，这无疑让我调试起来有些吃力，最终仍然有 2 条功能没有实现：

- `found by fd_parent_process!`
- `found by fd_find_debugger_window!`

这 2 功能的实现，不仅仅是重定位问题，还涉及到了程序的初始化问题。因为我们直接把 `exe` 给 `LoadLibrary` 起来他并没有执行本身的初始化部分，导致一些内存数据没有被初始化。

原则上这个问题也很容易解决，我们自己调用函数 `OEP` 地方的代码就可以了，当然这肯定要导致我们所谓的 `dll` 会像 `exe` 一样启动起来出来自己的界面。所以可以在显示界面的地方搞点 `SMC`，让他停留在那个地方不出现就可以了。

话虽这么说，但这个启动过程是如此漫长，我修复了十几条重定位数据，仍然没有正常跑起来，我就放弃了。

我附件里面的程序，首先启动 `LoadExe.exe`，然后上面三个按钮分别用来加载 `exe`、修复位导入表、重定位数据。

下面的 3 个函数用来实现相关功能，都是调用的 `check.exe` 里面的函数。`check.exe` 就是壳狼同学写的，我对被调用函数其稍微处理了一下，发现调试器以后直接返回，不弹出他自己的对话框。

如果程序在你系统上执行产生非法、退出，那很正常，可以尝试打开调试器再跑我这个程序，如果运气好的话，能看到点东西，不至于白下载。

最后一并感谢海风月影、16 的友情测试，以及 `www.luocong.com`

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | `LoadExe.rar` |
| :--- | :--- |
