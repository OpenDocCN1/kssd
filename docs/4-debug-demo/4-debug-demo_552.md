

# 常见自校检分析实例

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-512.htm`](https://www.pediy.com/kssd/pediy08/pediy8-512.htm)

自校检是许多软件的保护手段之一，对软件加个简单的壳再增加自校检在一定程序上可以抵挡住一大部分新手，不过，对许多人来说，这个保护已经很弱了。可是，搜索论坛，居然没有一篇系统的文章。不知道是大家太忙了还是因为要守住点秘密。其实大部分技术应该拿出来交流才好，只有交流才有进步。就象老王的 EPE，netsowell 一次一次的脱壳文章，才促使老王一次一次的升级，保护强度也越来越强。废话一堆，不讲了。下面讲几种常见的解决自校检方法，写的粗略，希望大家补充。

附件：CheckMe.rar  
  
1、通用对比法。就是将已经触发自校检的程序与原来正常的程序进行关键跳转对比，这种方法比较通用，大部分的自校检可以通过此方法解决（如果软件有防多开窗口的限制，需  
要先解决这个问题。）附件中的 sample1.EXE 是一个加了 ASPACK 的自校检程序，修改任何代码或大小等都会触发校检提示软件被修改。dumped.EXE 是脱壳后的文件，由于触发校检运行后出现“文件被非法修改”的提示，现在我们来解决脱壳后文件的自校检问题。打开脱壳后的程序 dumped.EXE，下断 BP CreateFileA，F9 两次后出现出错提示。CTRL+F2 重新载入 dumped.EXE，下断 BP reateFileA，F9 一次。这时另开一个 OD 打开原来的程序 sample1.EXE，用脚本到达 OEP，命令中也下断 BP CreateFileA，F9 一次，这时两个 OD 停在同一个地方，然后在两个 OD 中逐步单步跟踪，碰到 JE、JNE、JBE...之类的关键跳转要对比一下两者的区别。  
7C801A24 >  8BFF            MOV EDI,EDI                              ; BP CreateFileA 断在这里，ALT+F9 返回  
7C801A26    55              PUSH EBP  
\=================  
0040111C  |.  3BF4          CMP ESI,ESP                              ;  都停在这里，逐步 F8，进行关键跳转的对比  
0040111E  |.  E8 0D030000   CALL crcdumpe.00401430  
00401123  |.  8985 E0FEFFFF MOV DWORD PTR SS:\[EBP-120\],EAX  
00401129  |.  83BD E0FEFFFF>CMP DWORD PTR SS:\[EBP-120\],-1  
00401130  |.  75 07         JNZ SHORT crcdumpe.00401139  
  
逐步 F8，跟到下面的代码时，发现两个跳转不一样：  
0040120C   /75 07           JNZ SHORT crc.00401215                   ; 原版这个地方信息窗口提示：跳转没有实现  
0040120E   |B8 01000000     MOV EAX,1  
00401213   |EB 02           JMP SHORT crc.00401217  
00401215   \\33C0            XOR EAX,EAX  
\=======================  
0040120C  |. /75 07         JNZ SHORT crcdumpe.00401215              ;  脱壳版这个地方信息窗口提示：跳转已经实现，NOP 掉  
0040120E  |. |B8 01000000   MOV EAX,1  
00401213  |. |EB 02         JMP SHORT crcdumpe.00401217  
00401215  |> \\33C0          XOR EAX,EAX  
  
这时将脱壳版 0040120C 处代码 NOP 掉后另存为 dumpedFIX.EXE。试着运行一下，如果正常完事，还不行的话继续跟踪下去。该例只改这一处。  
  
2、跟踪退出函数。附件中 sample2 也是个自校检程序，修改任何一处软件会自动退出。我们试着用 UltraEdit 将 sample2.EXE 的最后一个字节改为 01 后另存为 sample2-change.EXE，这时运行 sample2-change 就会自动退出，我们就是要从退出函数入手。软件退出一般都是调用 ExitProcess、PostQuitMessage 之类的，我们用 OD 载入 sample2-change.EXE，从输入表中我们可以看出软件是调用 ExitProcess 退出的。于是在 OD 中下断 BP ExitProcess，F9 运行，断下后看堆栈信息：  
0012FEB8   004015B5  /CALL 到 ExitProcess 来自 sample2-.004015AF     //从这里我们可以看出 ExitProcess 的调用地方是在 004015AF  
0012FEBC   00000000  \\ExitCode = 0  
0012FEC0   20DFA6E6  
在 OD 中 CTRL+G，输入 004015AF：  
004015AF  |.  FF15 AC514200 CALL DWORD PTR DS:\[<&KERNEL32.ExitProces>; \\就在这里，向上找这个子 CALL 的首部  
004015B5  |>  8BE5          MOV ESP,EBP  
004015B7  |.  5D            POP EBP  
004015B8  \\.  C3            RETN  
\=======================================  
004014E0  /$  55            PUSH EBP                                 ;  找到这里，注意信息栏的内容  
004014E1  |.  8BEC          MOV EBP,ESP  
004014E3  |.  51            PUSH ECX  
004014E4  |.  833D F8354200>CMP DWORD PTR DS:\[4235F8\],1  
004014EB  |.  75 11         JNZ SHORT sample2-.004014FE  
信息栏的内容：  
Local Calls from 0040146B, 0040148B, 004014A9, 004014C9  
也就是说有四个地方调用 ExitProcess 退出，因为程序的退出按钮和关闭的叉号也是调用 ExitProcess 函数的，一般都会在前面几个，我们在内容上右击，“前往 CALL 来自 0040146B”  
0040146B  |.  E8 70000000   CALL sample2-.004014E0                   ; \\到这里，同样查找首部  
00401470  |.  83C4 0C       ADD ESP,0C  
00401473  |.  5D            POP EBP  
00401474  \\.  C3            RETN  
\============  
00401460  /$  55            PUSH EBP                                 ;  在这里，信息栏提示：Local Calls from 00401072, <ModuleEntryPoint>+11A  
00401461  |.  8BEC          MOV EBP,ESP  
00401463  |.  6A 00         PUSH 0                                   ; /Arg3 = 00000000  
00401465  |.  6A 00         PUSH 0                                   ; |Arg2 = 00000000  
在 Local Calls from 00401072 上右击，前往 CALL 来自 00401072：  
00401048  |.  E8 BDFFFFFF   CALL sample2-.0040100A  
0040104D  |.  85C0          TEST EAX,EAX  
0040104F  |.  74 1F         JE SHORT sample2-.00401070               ;  是从这里跳过去的，NOP 掉  
00401051  |.  8BF4          MOV ESI,ESP  
00401053  |.  6A 30         PUSH 30                                  ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
00401055  |.  68 28004200   PUSH sample2-.00420028                   ; |Title = "提示"  
0040105A  |.  68 1C004200   PUSH sample2-.0042001C                   ; |Text = "正常运行!"  
0040105F  |.  6A 00         PUSH 0                                   ; |hOwner = NULL  
00401061  |.  FF15 B4524200 CALL DWORD PTR DS:\<&USER32.MessageBoxA>>; \\MessageBoxA  
00401067  |.  3BF4          CMP ESI,ESP  
00401069  |.  E8 82050000   CALL sample2-.004015F0  
0040106E  |.  EB 07         JMP SHORT sample2-.00401077  
00401070  |>  6A 00         PUSH 0  
00401072  |.  E8 E9030000   CALL sample2-.00401460                   ;  就是这里了，最终会调用 ExitProcess，向上看是从哪里跳过来  
00401077  |>  33C0          XOR EAX,EAX  
  
可以看出，00401072 处的 CALL 最终呼叫 ExitProcess 退出，所以只要使 0040104F 处的跳转失效即可，将 0040104F 的跳转 NOP 掉后保存，测试运行正常。  
  
三、利用第三方软件辅助查找关键的地方。很多软件利用 CRC 或者 MD5 实现磁盘文件校验或者内存映像校验等，对此类软件我们可以利用算法识别工具找到密码学算法和核心，然后层层向上，找到最初的调用地方更改其流程方向。还是附件中的 sample1.EXE，脱壳的文件为 dumped.EXE，我们来解决这个软件的自校检。先用 PEID 的插件 kanal 分析 dumped.EXE 所采的密码学算法，如图，  
![  
可以看出，软件采用了 CRC 算法，算法特征码出现在 0040131C，就从这里入手，OD 载入 dumped.EXE，CTRL+G 搜索 0040131C：  
0040131C  |?  2083 B8ED898D AND BYTE PTR DS:\[EBX+8D89EDB8\],AL        ;  找到的地方在这里，上拉找到这个 CALL 的首部  
00401322  |?  FC            CLD  
00401323  |?  FB            STI  
00401324  |?  FFFF          ???                                      ;  未知命令  
00401326  |.  EB 0E         ||JMP SHORT crcdumpe.00401336  
00401328  |>  8B95 FCFBFFFF ||MOV EDX,DWORD PTR SS:\[EBP-404\]  
0040132E  |.  D1EA          ||SHR EDX,1  
00401330  |.  8995 FCFBFFFF ||MOV DWORD PTR SS:\[EBP-404\],EDX  
00401336  |>^ EB B5         |\\JMP SHORT crcdumpe.004012ED  
00401338  |>  8B85 F8FBFFFF |MOV EAX,DWORD PTR SS:\[EBP-408\]  
0040133E  |.  8B8D FCFBFFFF |MOV ECX,DWORD PTR SS:\[EBP-404\]  
00401344  |.  898C85 00FCFF>|MOV DWORD PTR SS:\[EBP+EAX\*4-400\],ECX  
0040134B  |.^ E9 6AFFFFFF   \\JMP crcdumpe.004012BA  
\============================  
00401290  /> \\55            PUSH EBP                                 ;  到这里，看 OD 的提示栏：Jump from 00401005，从 00401005 跳转来的  
00401291  |.  8BEC          MOV EBP,ESP  
00401293  |.  81EC 50040000 SUB ESP,450  
00401299  |.  53            PUSH EBX  
0040129A  |.  56            PUSH ESI  
  
在 Jump from 00401005 上右击，“前往 JMP 来自 00401005”  
00401005   $ /E9 86020000   JMP crcdumpe.00401290                    ;  到这里，看信息栏内容：Local Call from 00401201，00401201 处的 CALL 调用这里  
0040100A   $ |E9 B1000000   JMP crcdumpe.004010C0  
在 Local Call from 00401201 上右击，“前往 CALL 来自 00401201”  
00401201  |.  E8 FFFDFFFF   CALL crcdumpe.00401005                   ;  到这里，再向上找到这个 CALL 的顶部  
00401206  |.  83C4 08       ADD ESP,8  
\==================  
004010C0  /> \\55            PUSH EBP                                 ;  到这里，看任务栏信息：Jump from 0040100A  
004010C1  |.  8BEC          MOV EBP,ESP  
004010C3  |.  81EC 64010000 SUB ESP,164  
在 Jump from 0040100A 上右击，“前往 JMP 来自 0040100A”  
0040100A   $ /E9 B1000000   JMP crcdumpe.004010C0                    ;  到这里，继续根据任务栏信息向上找：Local Call from 00401048  
0040100F   $ |E9 1C000000   JMP crcdumpe.00401030  
在 Local Call from 00401048 上右击，“前往 CALL 来自 00401048”  
00401048  |.  E8 BDFFFFFF   CALL crcdumpe.0040100A                   ;  就是这里了，计算 CRC 并进行对比的 CALL  
0040104D  |.  85C0          TEST EAX,EAX  
0040104F  |.  74 1F         JE SHORT crcdumpe.00401070               ;  这里就是关键跳转了，NOP 掉  
00401051  |.  8BF4          MOV ESI,ESP  
00401053  |.  6A 30         PUSH 30                                  ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
00401055  |.  68 40004200   PUSH crcdumpe.00420040                   ; |Title = "提示"  
0040105A  |.  68 34004200   PUSH crcdumpe.00420034                   ; |Text = "正常运行!"  
0040105F  |.  6A 00         PUSH 0                                   ; |hOwner = NULL  
00401061  |.  FF15 B4524200 CALL DWORD PTR DS:\[<&user32.MessageBoxA>>; \\MessageBoxA  
更改 0040104F 处的跳转后保存为 dumpedFIX，运行正常。  
  
四、对付校检自身大小的软件的一般方法。也有许多软件，利用 CHECKSUM、FILELEN 等在代码中对本身的大小做了标记，如果发现大小变了就自动退出。附件中的 FILELEN 就是这样的，FILELEN-UNPACK.EXE 是脱壳后的软件，由于脱壳后软件大小发生了变化，所以 FILELEN-UNPACK 运行后就自动退出。对付这种校检，我们有个简单的方法，就是将脱壳后的软件中检测自身大小的部分改为脱壳后文件的大小。我们看一下 FILELEN 的大小为：10752 字节转 16 进制为 2A00，再看看脱壳后的文件 FILELEN-UNPACK.EXE 的大小 30208 字节，即 7600，关键是如何找到对自身大小进行对比的语句，我们知道一般的形式都是 CMP EAX，2A00，但是到底是哪个寄存器呢？EAX，EBX、ECX....，寄存器种类比较多，我们不可能每个去找，这时 W32Dasm 派上用场。用 W32Dasm 载入 FILELEN-UNPACK，我们只要搜索 00002A00 即可，前面的部分不管它，找到在这里：  
:00401B8F 81FE002A0000            cmp esi, 00002A00  
右击-“HEX”，更改代码 002A00 为脱壳后的大小 007600 即可，再搜索，发现该例只有一个对比的地方，保存后运行正常。  
对于 VB 检测自身大小的软件我们还可以跟踪 FileLen 函数，因为 VB 一般都用 FileLen 检测自身的大小，用 OD 载入 FILELEN-UNPACK.EXE，下断 BP rtcFileLen，F9 后断下，ALT+F9 返回：  
00401B5E   .  8BF0          MOV ESI,EAX                              ;  这里 EAX 的值就是 FILELEN 取得的自身的大小，EAX=00007600  
00401B60   .  8D55 D4       LEA EDX,DWORD PTR SS:\[EBP-2C\]  
00401B63   .  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  
00401B66   .  52            PUSH EDX  
00401B67   .  8D4D E0       LEA ECX,DWORD PTR SS:\[EBP-20\]  
00401B6A   .  50            PUSH EAX  
00401B6B   .  8D55 DC       LEA EDX,DWORD PTR SS:\[EBP-24\]  
00401B6E   .  51            PUSH ECX  
00401B6F   .  8D45 E4       LEA EAX,DWORD PTR SS:\[EBP-1C\]  
00401B72   .  52            PUSH EDX  
00401B73   .  50            PUSH EAX  
00401B74   .  6A 05         PUSH 5  
00401B76   .  FF15 70104000 CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStrList  
00401B7C   .  8D4D CC       LEA ECX,DWORD PTR SS:\[EBP-34\]  
00401B7F   .  8D55 D0       LEA EDX,DWORD PTR SS:\[EBP-30\]  
00401B82   .  51            PUSH ECX  
00401B83   .  52            PUSH EDX  
00401B84   .  6A 02         PUSH 2  
00401B86   .  FF15 14104000 CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeO>;  msvbvm60.\_\_vbaFreeObjList  
00401B8C   .  83C4 24       ADD ESP,24  
00401B8F   .  81FE 002A0000 CMP ESI,2A00                             ;  这里就是利用 FILELEN 取得的大小与原来做的标记对比，可以改这里的 2A00 为 7600 或者更改下面的跳转  
00401B95   .  75 6E         JNZ SHORT FILELEN-.00401C05  
00401B97   .  B9 04000280   MOV ECX,80020004  
  
  
好像常见的自校检就这些，我不知道还有没有其它的，如果有希望大家能够补充完善。 

 4dd466d0e1e0cc123afbe504da2462ca