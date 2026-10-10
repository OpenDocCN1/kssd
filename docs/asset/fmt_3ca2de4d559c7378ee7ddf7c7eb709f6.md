
# 我的第二轮比赛第一题解题思路(51bytes)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-258.htm`](https://www.pediy.com/kssd/pediy09/pediy09-258.htm)

> 看完题目要求，因为是修改弹出窗口显示内容，所以用 `ollydbg` 加载 `exlpoitme.exe`，下 `MessageBoxA` 断点。

> 点击按钮后断下来以后返回程序代码，得到如下程序段：

代码:

```
004002F6  /$  55            PUSH EBP
004002F7  |.  8BEC          MOV EBP,ESP
004002F9  |.  83EC 10       SUB ESP,10
004002FC  |.  56            PUSH ESI
004002FD  |.  57            PUSH EDI
004002FE  |.  33FF          XOR EDI,EDI
00400300  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
00400303  |.  33F6          XOR ESI,ESI
00400305  |.  897D FC       MOV DWORD PTR SS:[EBP-4],EDI
00400308  |.  897D F8       MOV DWORD PTR SS:[EBP-8],EDI
0040030B  |.  E8 60010000   CALL ExploitM.00400470
00400310  |.  68 6C024000   PUSH ExploitM.0040026C                           ;  ASCII "test.txt"
00400315  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
00400318  |.  E8 63010000   CALL ExploitM.00400480
0040031D  |.  85C0          TEST EAX,EAX
0040031F  |.  74 64         JE SHORT ExploitM.00400385
00400321  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00400324  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
00400327  |.  50            PUSH EAX
00400328  |.  E8 D3010000   CALL ExploitM.00400500
0040032D  |.  85C0          TEST EAX,EAX
0040032F  |.  74 54         JE SHORT ExploitM.00400385                       ; 判断文件是否打开
00400331  |.  8B4D FC       MOV ECX,DWORD PTR SS:[EBP-4]
00400334  |.  83F9 08       CMP ECX,8
00400337  |.  7E 4C         JLE SHORT ExploitM.00400385                      ; 判断文件大小是否>0
00400339  |.  B8 00100000   MOV EAX,1000
0040033E  |.  3BC8          CMP ECX,EAX
00400340  |.  7F 43         JG SHORT ExploitM.00400385                       ; 判断文件大小是否<1000H
00400342  |.  6A 40         PUSH 40                                          ; /Protect = PAGE_EXECUTE_READWRITE
00400344  |.  41            INC ECX                                          ; |
00400345  |.  50            PUSH EAX                                         ; |AllocationType => MEM_COMMIT
00400346  |.  51            PUSH ECX                                         ; |Size
00400347  |.  57            PUSH EDI                                         ; |Address
00400348  |.  FF15 24024000 CALL DWORD PTR DS:[<&KERNEL32.VirtualAlloc>]     ; \VirtualAlloc
0040034E  |.  8BF0          MOV ESI,EAX
00400350  |.  3BF7          CMP ESI,EDI
00400352  |.  74 31         JE SHORT ExploitM.00400385
00400354  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
00400357  |.  53            PUSH EBX
00400358  |.  50            PUSH EAX
00400359  |.  56            PUSH ESI
0040035A  |.  FF75 FC       PUSH DWORD PTR SS:[EBP-4]
0040035D  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
00400360  |.  E8 AB010000   CALL ExploitM.00400510
00400365  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
00400368  |.  8BD8          MOV EBX,EAX
0040036A  |.  E8 61010000   CALL ExploitM.004004D0
0040036F  |.  3BDF          CMP EBX,EDI
00400371  |.  5B            POP EBX
00400372  |.  74 11         JE SHORT ExploitM.00400385
00400374  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00400377  |.  3945 F8       CMP DWORD PTR SS:[EBP-8],EAX
0040037A  |.  75 09         JNZ SHORT ExploitM.00400385
0040037C  |.  50            PUSH EAX
0040037D  |.  56            PUSH ESI
0040037E  |.  E8 FDFEFFFF   CALL ExploitM.00400280
00400383  |.  59            POP ECX
00400384  |.  59            POP ECX
00400385  |>  57            PUSH EDI                                         ; /Style
00400386  |.  68 68024000   PUSH ExploitM.00400268                           ; |Title = "Try"
0040038B  |.  68 60024000   PUSH ExploitM.00400260                           ; |Text = "Failed!"
00400390  |.  57            PUSH EDI                                         ; |hOwner
00400391  |.  FF15 4C024000 CALL DWORD PTR DS:[<&USER32.MessageBoxA>]        ; \MessageBoxA
00400397  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
0040039A  |.  E8 31010000   CALL ExploitM.004004D0
0040039F  |.  3BF7          CMP ESI,EDI
004003A1  |.  74 0D         JE SHORT ExploitM.004003B0
004003A3  |.  68 00800000   PUSH 8000                                        ; /FreeType = MEM_RELEASE
004003A8  |.  57            PUSH EDI                                         ; |Size
004003A9  |.  56            PUSH ESI                                         ; |Address
004003AA  |.  FF15 20024000 CALL DWORD PTR DS:[<&KERNEL32.VirtualFree>]      ; \VirtualFree
004003B0  |>  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]
004003B3  |.  E8 38010000   CALL ExploitM.004004F0
004003B8  |.  6A 01         PUSH 1
004003BA  |.  58            POP EAX
004003BB  |.  5F            POP EDI
004003BC  |.  5E            POP ESI
004003BD  |.  C9            LEAVE
004003BE  |.  C3            RETN
```

根本就没有显示`"OK!"`的分支，所以显然是要修改 `400260` 和 `400268` 里面的输出内容了。

而且得到很重要的提示：`test.txt`

于是构造 `test.txt`，里面内容`"1234567890"`先分析。

打开 `test.txt` 后，判断了文件大小，需要 `8<文件大小<1000H`

`VirtualAlloc` 后，把文件内容读入，地址保存在 `ESI`

对 `ESI` 里的内容下内存断点，断下后返回，发现是从

代码:

```
0040037C  |.  50            PUSH EAX
0040037D  |.  56            PUSH ESI
0040037E  |.  E8 FDFEFFFF   CALL ExploitM.00400280
```

这里进去的，那就看看 `400280` 的代码内容是什么。

代码:
