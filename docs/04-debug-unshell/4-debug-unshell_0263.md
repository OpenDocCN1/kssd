
# Armadillo 3.78 带 Key 双进程手动脱壳

> 原文：[`www.pediy.com/kssd/pediy11/121451.html`](https://www.pediy.com/kssd/pediy11/121451.html)

Armadillo 3.78 带 Key 双进程手动脱壳

Passion

2010-10-01

拿到一 HTTPS 的暴力破解软件，带一个和机器有关的合法注册码，只能在别的某一台机器码为 `9534-57ED` 的 PC 上使用。`PEID` 查壳为 Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks，找了几个自动脱壳机如 `dilloDIE` 等，均无效，卡在那儿没动作。于是尝试手动脱壳。

以前未研究过穿山甲壳，搜了许多资料，费了几天时间，按部就班地脱成功了，写下来记录一下。

工具：`PEID`、`OD`、`ArmaAFP`、`LordPE`、`ImportRec`

## 一、绕过 Key

参考资料：

`http://hi.baidu.com/csh0w/blog/item/ca8a8f83a9e462b46d811979.html`

绕过 Key 的原理在于拦截程序获得机器码的部分，将其改为我们已知的机器码，这样和已知的注册码配合就能通过而运行。

首先，用 `ArmaAFP` 看到 Armadillo 的保护选项：

```
!- Protected Armadillo

Protection system (Professional)

!- <Protection Options>

Debug-Blocker

!- <Backup Key Options>

Fixed Backup Keys

!- <Compression Options>

Better/Slower Compression

!- <Other Options>

Use Digital River Edition Keys

Version 3.78 22Sep2004
```

可以看到是 Armadillo 3.78 版，里头有反调试器与 Key 选项，其实还有双进程保护，但没写出来。

程序名为 `https.exe`，运行后有两个 `https.exe` 进程。原理大概是互相通过一个 Mutex 来通讯检测对方是否存在。`OD` 中可以通过手动汇编的方式进行双变单，但我直接用了 `ArmaAFP` 的双变单功能，打印如下：

```
!- Child detach

Child process ID: 00001524

Entry point: 005A3243

Original bytes: 558B
```

它启动了一个进程号是 `1524` 的子进程，并通过将入口代码俩字节 `558B` 改成“JMP 自身”的方式停在入口 `5A3243`。双已经变单了。

下面启动 `OD` 附加 `1524` 这个进程。因为它带调试器检测，首先得隐藏 `OD`：忽略所有异常，并在异常忽略范围内加入 `C000001D..C000001E`。如果不干这个，程序一跑就会提示有调试器并退出。

设置绕过调试器检测并附加进程后，首先停在入口 `005A3243`，将 `ArmaAFP` 改的死循环代码改回 `558B` 后这样：

```
005A3243    55            push    ebp
```

开始拦截机器码。下断点 `he GetDlgItem`，`Shitf+F9` 运行，中断再运行等出需要注册的提示框，点 OK 后中断，单步进行。

```
00CC4689    E8 253CFEFF     call    00CA82B3 在此 F7 进入
00CA82C2    E8 24D70000     call    00CB59EB 再 F7 进入
00CB59FF    C2 0800         retn    8
```

到这儿返回时，`eax` 已经是根据我的机器生成的机器码 `6837-9958` 了。这儿手工将 `EAX` 改成 `953457ED`，运行。出现注册对话框，里头的机器码果然变成了需要的 `9534-57ED`，于是拿已知的 KEY 注册，注册成功，能跑了。

## 二、绕过 IAT 加密，寻找 OEP

参考资料：看雪论坛精华中 Armadillo 相关文章。

绕过机器 Key 才是第一步，我们的目的是脱壳，于是开始寻找 OEP。寻找 OEP 本身不难，但 Armadillo 会对 IAT 进行加密处理，导致后面 dump 出的内容无效，因此寻找 OEP 之前，需要绕过 IAT 的加密处理，以后 dump 后才能方便地用 `ImpRec` 修复。

重复上述 `ArmaAFP` 加 `OD` 加载 `https.exe` 的步骤，将首俩字节改回 `558B` 后，下断 `he GetModuleHandleA` 并反复 `Shift+F9` 运行（大概 8 次），直到 `OD` 堆栈窗口出现 `VirtualAlloc` 和 `VirtualFree` 后，寄存器窗口出现两个 `kernel32.dll` 字符串并且堆栈里也有个 `kernel32.dll` 时，`Alt+F9` 返回。如下：

```
EAX 001293DC ASCII "kernel32.dll"
ECX 001293E8
EDX 001293DC ASCII "kernel32.dll"
EBX 00000000

0012928C   00CB5CE1  /CALL 到 GetModuleHandleA 来自 00CB5CDB
00129290   001293DC  \pModule = "kernel32.dll"
00129294   00000000
```

`Alt+F9` 返回，出现 Magic Jump，可以在此绕过 IAT 加密：

```
00CB5CDB    FF15 B860CD00   call    dword ptr [CD60B8]           ; kernel32.GetModuleHandleA
00CB5CE1    8B0D AC40CE00   mov     ecx, dword ptr [CE40AC]      <====== 返回到这儿
00CB5CE7    89040E          mov     dword ptr [esi+ecx], eax
00CB5CEA    A1 AC40CE00     mov     eax, dword ptr [CE40AC]
00CB5CEF    391C06          cmp     dword ptr [esi+eax], ebx
00CB5CF2    75 16           jnz     short 00CB5D0A
00CB5CF4    8D85 B4FEFFFF   lea     eax, dword ptr [ebp-14C]
00CB5CFA    50             push    eax
00CB5CFB    FF15 BC62CD00   call    dword ptr [CD62BC]           ; kernel32.LoadLibraryA
00CB5D01    8B0D AC40CE00   mov     ecx, dword ptr [CE40AC]
00CB5D07    89040E          mov     dword ptr [esi+ecx], eax
00CB5D0A    A1 AC40CE00     mov     eax, dword ptr [CE40AC]
00CB5D0F    391C06          cmp     dword ptr [esi+eax], ebx
00CB5D12    0F84 2F010000   je      00CB5E47                     <====== 这儿是绕过 IAT 处理的 Magic Jump
00CB5D18    33C9            xor     ecx, ecx
```

在 `00CB5D12` 处手工汇编改为 `jmp 00CB5E47`，然后往下滚，找到：

```
00CB5E50    83C6 04         add     esi, 4
00CB5E53    395F FC         cmp     dword ptr [edi-4], ebx
00CB5E56  ^ 0F85 49FeFFFF   jnz     00CB5CA5
00CB5E5C    EB 03           jmp     short 00CB5E61               <====== 在这儿下断
00CB5E5E    D6             salc
00CB5E5F    D6             salc
00CB5E60    8F             ???                                  ; 未知命令
```

`00CB5E5C` 处的 `jmp` 与 `salc` 处是 IAT 加密结束标志，在 `jmp` 处下断，`Shift+F9` 运行，断在这儿。回到上面，把 `jmp` 的改动恢复成 `je`。至此 IAT 加密被绕过了。

下面寻找 OEP，清除 `GetModuleHandleA` 断点后下断 `he CreateThread`，`Shift+F9` 运行一次，断后 `Alt+F9` 返回。

```
00CBC23B    57             push    edi
00CBC23C    FF15 5C61CD00  call    dword ptr [CD615C]           ; kernel32.CreateThread
00CBC242    50             push    eax                           <======= 返回到这儿
00CBC243    FF15 4C62CD00  call    dword ptr [CD624C]           ; kernel32.CloseHandle
00CBC249    5F             pop     edi
00CBC24A    5E             pop     esi
00CBC24B    C9             leave
00CBC24C    C3             retn
```

再 `F8` 从 `retn` 返回，到上层：

```
00CCF5F9    E8 95CBFEFF    call    00CBC193
00CCF5FE    59             pop     ecx                           <======= 返回到这儿
00CCF5FF    BE 98FACD00    mov     esi, 0CDFA98
00CCF604    8BCE           mov     ecx, esi
00CCF606    E8 3393FDFF    call    00CA893E
00CCF60B    84C0           test    al, al
00CCF60D    75 09          jnz     short 00CCF618
```

单步 `F8` 下去几十步，一直到一个 `CALL ECX` 的地方：

```
00CCF709    8B90 90000000  mov     edx, dword ptr [eax+90]
00CCF70F    3350 40        xor     edx, dword ptr [eax+40]
00CCF712    3350 04        xor     edx, dword ptr [eax+4]
00CCF715    2BCA           sub     ecx, edx
00CCF717    FFD1           call    ecx                            ; https.00401000 这儿！！！F7 进去！！！
00CCF719    8945 E4        mov     dword ptr [ebp-1C], eax
00CCF71C    8B45 E4        mov     eax, dword ptr [ebp-1C]
```

`00CCF717` 进去，里头就是 OEP，`401000`。

```
00401000   /EB 10          jmp     short 00401012                <======= 程序入口点
00401002   |66:623A        bound   di, dword ptr [edx]           <======= 这几句不是代码，是字符串 C++HOOK
00401005   |43             inc     ebx
00401006   |2B2B           sub     ebp, dword ptr [ebx]
00401008   |48             dec     eax
00401009   |4F             dec     edi
0040100A   |4F             dec     edi
0040100B   |4B             dec     ebx
0040100C   |90             nop
0040100D  -|E9 78545000    jmp     0090648A
00401012   \\A1 6B545000    mov     eax, dword ptr [50546B]
```

后来分析出这是 C++Builder 写的，入口点是一个 `JMP`，后面紧跟 ASCII 字符串 `C++HOOK`，这是可以用来辨认 C++Builder 程序的入口点的特征。

## 三、Dump 以及 Fix IAT

现在可以 DUMP 了，用 `LoadPE` 找到 `1524` 号进程，Full dump 之，并在 PE Editor 中修改其入口点为 `1000`，也就是 `401000 - ImageBase 400000 = 1000`。

照理现在可以直接用 `ImpRec` 来修复输入表了，不过启动 `ImpRec`，载入 `1524` 号进程后，`GetImport` 不完全。修复出来的没法用。于是继续在 `OD` 中手工找 IAT 的开始和结束地址。

有篇教程说在这种情况下可以往下 `Ctrl+B` 查找汇编代码 `FF25`，也就是 `JMP DWORD PTR[]`，这种代码通常是 API 的调用。我们只要找到一个 API 调用，看它的地址在哪儿，就能找到 IAT 的所在。掌握这种思想就行。

刚上面断在 OEP 处，往下翻一两页，看到个 API 的调用：

```
0040109B    6A 08          push    8
0040109D    E8 CE2B1000    call    00503C70
004010A2    50             push    eax
004010A3    E8 522C1000    call    00503CFA          ; jmp 到 ntdll.RtlAllocateHeap
004010A8    0BC0           or      eax, eax
004010AA    75 0A          jnz     short 004010B6
```

从 `004010A3` 处有个 `CALL`，`Ctrl+G` 跑到 `00503CFA`，一看是一堆 API 的 `JMP DWORD PTR` 调用。

```
00503CE8  - FF25 0CC45300   jmp     dword ptr [53C40C]    ; kernel32.GlobalReAlloc
00503CEE  - FF25 10C45300   jmp     dword ptr [53C410]    ; kernel32.GlobalSize
00503CF4  - FF25 14C45300   jmp     dword ptr [53C414]
00503CFA  - FF25 18C45300   jmp     dword ptr [53C418]    ; ntdll.RtlAllocateHeap
00503D00  - FF25 1CC45300   jmp     dword ptr [53C41C]    ; ntdll.RtlFreeHeap
00503D06  - FF25 20C45300   jmp     dword ptr [53C420]    ; kernel32.InitializeCriticalSection
```

后面的 `53C414` 等就是 IAT 表的内容！

按长型，地址在数据窗口显示内容如下：

```
0053C410  7C834DD1  kernel32.GlobalSize
0053C414  00CBA413
0053C418  7C9300C4  ntdll.RtlAllocateHeap
0053C41C  7C92FF2D  ntdll.RtlFreeHeap
0053C420  7C809F91  kernel32.InitializeCriticalSection
```



往上下翻，可以得到 `IAT` 表的起始地址和大小。起始是 `53C100`，结束是 `53CED4`。长度一减得到 `DD4`。

现在启动 `ImpRec`，选这个进程，输入其 `OEP` `1000`，`IAT` 起始地址 `13C100`（已经减去了 `ImageBase` `400000`），长度 `DD4`，点 `GetImports`，得到一堆 `Valid NO` 的。

`Show Invalid` 后把十几个无效指针挨个 `CUT` 掉，再 `Fix dump` 刚才 `dump` 出的文件。保存后双击，正常运行。脱壳成功。
