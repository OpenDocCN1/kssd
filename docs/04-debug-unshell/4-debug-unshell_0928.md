
# Petite 脱壳详解

> 原文：[`www.pediy.com/kssd/pediy10/97055.html`](https://www.pediy.com/kssd/pediy10/97055.html)

【文章标题】: `Petite 脱壳详解`  

【文章作者】: `index09`  

【作者邮箱】: `cradiator@gmail.com`  

【作者主页】: `http://hi.baidu.com/index09`  

【软件名称】: `PeTite2.3` + `OD` + `Stud_PE`  

--------------------------------------------------------------------------------

【详细过程】  

已经分析了两个压缩壳了，这些天分析了第一款加密壳，果然比 `UPX` 之类的复杂很多。  

`PeTite` 是一款不错的加密压缩外壳，压缩分为 9 级，我用最强的级别加密了 `windows` 自带的计算器程序。  

居然压了 10 多分钟 = =||| 压缩完了和 `UPX` 的比率差不多。闲话少说开始分析。  

习惯用 `Stud_PE` 先看一下节表，不过事实证明这次没什么有用的信息。  

No | Name | VSize | VOffset | RSize | ROffset | Charact. |
|:---|:---|:---|:---|:---|:---|:---|
| 01 | `.petite` | `00015000` | `00001000` | `00006800` | `00000800` | `E0000060` |
| 02 | `.rsrc` | `00009000` | `00016000` | `0000712D` | `00007000` | `C0000040` |
| 03 | | `000003CA` | `0001F000` | `00000400` | `00000400` | `E2000060` |

`OD` 载入后，首先运行程序，出现异常。`SHIFT+F9` 两次以后出现了计算器的界面。  

看来加密壳果然还是在解压时动了一些手脚了。  

开始分析，`OD` 载入后代码如下

```assembly
  0101F046  >  B8 00F00101     MOV EAX,calc.0101F000                     ; eax = sec code(no name)
  0101F04B    68 004A0101     PUSH calc.01014A00                        ; 这里安装了 SEH  handler = 01014A00
  0101F050    64:FF35 0000000>PUSH DWORD PTR FS:[0]                     ; 这时 handler 里面还没有代码
  0101F057    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
```

已经非常清楚了，这里安装了异常处理函数。哈，这也太明显了。记下来函数地址 `01014A00`。

```assembly
  0101F05E    66:9C           PUSHFW
  0101F060    60              PUSHAD
  0101F061    50              PUSH EAX
  0101F062    8BD8            MOV EBX,EAX                               ; ebx = sec no name
  0101F064    0300            ADD EAX,DWORD PTR DS:[EAX]                ; eax = 函数指针数组
  0101F066    68 C85F0000     PUSH 5FC8
  0101F06B    6A 00           PUSH 0                                      ; GMEM_FIXED
  0101F06D    FF50 1C         CALL DWORD PTR DS:[EAX+1C]                ; GlobalAlloc
  0101F070    8943 08         MOV DWORD PTR DS:[EBX+8],EAX              ; [ebx+8] = 空间地址
```

继续往下，程序申请了一段缓冲区，并且存入了 03 号段 + 8 这个偏移之中。  

这段缓冲在以后的解压过程我们会看到。  

后来程序用 `VirtualProtect` 将自己 `IAT` 的一部分复制到 `01000780`

```assembly
  0101F091    56              PUSH ESI
  0101F092    6A 02         PUSH 2
  0101F094    50              PUSH EAX
  0101F095    57              PUSH EDI
  0101F096    6A 12         PUSH 12
  0101F098    6A 0A         PUSH 0A
  0101F09A    56              PUSH ESI                                  ; oldProtect
  0101F09B    6A 04         PUSH 4                                      ; PAGE_READWRITE
  0101F09D    50              PUSH EAX                                  ; size = 880
  0101F09E    57              PUSH EDI                                  ; address = 1000780
  0101F09F    FFD3            CALL EBX                                  ; VirtualProtect
  0101F0A1    83EE 08       SUB ESI,8
  0101F0A4    59              POP ECX                                   ; ecx = 0x0A0
  0101F0A5    F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>   ; 这里重新建立了一个函数数组 地址 01000780
  0101F0A7    59              POP ECX                                   ; ecx = 0x12
  0101F0A8    66:83C7 58    ADD DI,58
  0101F0AC    81C6 E0000000 ADD ESI,0E0                                 ; esi 指向 dll 数组 地址 01000780
  0101F0B2    F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
  0101F0B4    FFD3            CALL EBX                                  ; VirtualProtect  恢复 PAGE_READONLY
```

数据如下

```text
  01000780  7E4507EA  USER32.MessageBoxA
  01000784  7E41A8AD  USER32.wsprintfA
  01000788  00000002
  0100078C  7C81CAFA  kernel32.ExitProcess
  01000790  7C801D7B  kernel32.LoadLibraryA
  01000794  7C80AE30  kernel32.GetProcAddress
  01000798  7C801AD4  kernel32.VirtualProtect
  0100079C  7C80FDBD  kernel32.GlobalAlloc
  010007A0  7C80FCBF  kernel32.GlobalFree
  010007A4  7C80B731  kernel32.GetModuleHandleA
  010007A8  00000000
```

继续往下这里出现了一个很重要的数据结构

```assembly
  0101F0B7    8D90 A0010000   LEA EDX,DWORD PTR DS:[EAX+1A0]           ; ///////////////将元素 1 指向的数据复制到缓冲，大小为元素 5/////////////////////////
  0101F0BD    8B0A            MOV ECX,DWORD PTR DS:[EDX]               ; ecx = 元素 1
  0101F0BF    83C2 14         ADD EDX,14                               ; 0x14 字节的结构
  0101F0C2    8B5A F0         MOV EBX,DWORD PTR DS:[EDX-10]            ; ebx = 元素 2
  0101F0C5    85DB            TEST EBX,EBX
  0101F0C7  ^ 74 F4           JE SHORT calc.0101F0BD
  0101F0C9    8B0424          MOV EAX,DWORD PTR SS:[ESP]               ; eax = 一个偏移
  0101F0CC    8D3401          LEA ESI,DWORD PTR DS:[ECX+EAX]           ; esi = 不知什么
  0101F0CF    8B6C24 04       MOV EBP,DWORD PTR SS:[ESP+4]             ; ebp = sec no name
  0101F0D3    8B6D 08         MOV EBP,DWORD PTR SS:[EBP+8]             ; ebp = 刚才申请的空间
  0101F0D6    8B4A FC         MOV ECX,DWORD PTR DS:[EDX-4]             ; ecx = 大小
  0101F0D9    8BFD            MOV EDI,EBP
  0101F0DB    52              PUSH EDX
  0101F0DC    F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>   ; 复制到缓冲区里
  0101F0DE    8BF5            MOV ESI,EBP                               ; esi = 那个缓冲
  0101F0E0    8B7A F4         MOV EDI,DWORD PTR DS:[EDX-C]              ; edi = 元素 3
  0101F0E3    03F8            ADD EDI,EAX
  0101F0E5    EB 28           JMP SHORT calc.0101F10F                   ; .........................................................................
```

这里的 `EDX` 指向一个 `0x14` 字节的数据结构，源程序中有几个节就有几个这个结构，可见它和程序的解压过程密切相关。  

因为过程十分复杂，并没有完全知道这个结构的具体意思。  

大致是这样的。

```c
struct{
    PBYTE ptData1;      //保存了一些数据
    DWORD dwMagic;      //控制了解压过程的一个数
    DWORD unKnown;
    PBYTE ptData2;      //与 ptData1 共同进行解压过程
    DWORD dwSize;       //ptData1 的数据大小
}
```

说实话解压过程没怎么看明白，各数据结构与各段的对应关系也每太搞清楚。  

不过好在这并不影响我们脱壳。  

在解压过程中由于程序只有三个刚才提到得 `struct`，而程序却一直解压，到第四个的时候终于如愿的抛出了一场。  

一场出现在这一句。

```assembly
  0101F137    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
```

因为刚才安装了 `SEH`，所以我们在那上面下断点。`bp 01014A00` 然后 `shift + f9`

来到了这里

```assembly
  01014A54    33C0            XOR EAX,EAX
  01014A56    5E              POP ESI                                   ; esi = 刚才的返回地址
  01014A57    64:8B18         MOV EBX,DWORD PTR FS:[EAX]
  01014A5A    8B1B            MOV EBX,DWORD PTR DS:[EBX]
  01014A5C    8D63 D6         LEA ESP,DWORD PTR DS:[EBX-2A]
  01014A5F    5D              POP EBP
  01014A60    8D8E CB020000   LEA ECX,DWORD PTR DS:[ESI+2CB]
  01014A66    894B 04         MOV DWORD PTR DS:[EBX+4],ECX
  01014A69    64:891D 0000000>MOV DWORD PTR FS:[0],EBX                  ; SEH handler = 01014CD0
  01014A70    8B3C24          MOV EDI,DWORD PTR SS:[ESP]                ; edi = sec no name
  01014A73    FF77 08         PUSH DWORD PTR DS:[EDI+8]
  01014A76    FF95 A0070000   CALL DWORD PTR SS:[EBP+7A0]               ; 释放了缓冲区
```

这段代码释放了刚才申请的缓冲。  

而且又看到了暗桩。又是个 `SEH`，指向 `01014CD0`

往下看不远处 `JMP EAX`

到这里时 `EAX` 的值为 0，于是又是异常。在 `01014CD0` 下断，忽略异常。来到这里

```assembly
  01014CD0    33C0            XOR EAX,EAX                               ; SEH2
  01014CD2    64:8B18         MOV EBX,DWORD PTR FS:[EAX]
  01014CD5    8B1B            MOV EBX,DWORD PTR DS:[EBX]                ; ebx = 下一个 SEH
  01014CD7    8D63 AE         LEA ESP,DWORD PTR DS:[EBX-52]
  01014CDA    61              POPAD
  01014CDB    833E 00         CMP DWORD PTR DS:[ESI],0                   ; esi 是 0x14 字节的结构
  01014CDE  ^ 0F84 B6FDFFFF   JE calc.01014A9A
  01014CE4    8B7E 08         MOV EDI,DWORD PTR DS:[ESI+8]              ; edi = 元素 3(offset)
  01014CE7    03FD            ADD EDI,EBP
  01014CE9    8B4E 0C         MOV ECX,DWORD PTR DS:[ESI+C]              ; ecx = 元素 4
  01014CEC    D1F9            SAR ECX,1
  01014CEE    51              PUSH ECX
```



```
  01014CEF    72 15           JB SHORT calc.01014D06  
  01014CF1    037E 04         ADD EDI,DWORD PTR DS:[ESI+4]            ; edi + magicnum ???  
  01014CF4    C1F9 02         SAR ECX,2  

  这段代码我没有仔细读。  

  不过通过对 `GetProcAddress` 下断，知道它包含了修复 `IAT` 的部分。  

  往后有一个  

  01014D51    E8 2FE87C07     CALL 087E3585  

  进入以后  

  0101F03D    5F              POP EDI                                 ; calc.01014D56  
  0101F03E    F3:AA           REP STOS BYTE PTR ES:[EDI]  
  0101F040    61              POPAD  
  0101F041    66:9D           POPFW  
  0101F043    83C4 08         ADD ESP,8  
  0101F046 -> E9 2A34FFFF     JMP calc.01012475                       ; eax = sec code(no name)  
```

最后一个 `JMP` 跳转到 `OEP`。

可以发现这个 `OEP` 也是我们程序本身的入口地址，看来壳中还包含了 `SMC`。具体流在以后分析吧。

`JMP` 后便可以 `OllyDump` 了，可以正常运行。

--------------------------------------------------------------------------------

【经验总结】

这个壳调的不是很清楚，有许多地方还莫名奇妙。

不过这是我第一次遇到真正的 `SEH`，而且这里面有自修改代码，`OD F2` 下断时，要修改内存指令。所以如果下早了，断点就会被 `SMC` 覆盖掉，使断点失效甚至使代码解密过程出错。所以在产生异常时再下断会更加保险。

--------------------------------------------------------------------------------

2009 年 09 月 03 日 11:30:18
