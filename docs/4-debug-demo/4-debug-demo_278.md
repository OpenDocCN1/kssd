

# Total Control PC version 2.0

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-15.htm`](https://www.pediy.com/kssd/pediy08/pediy8-15.htm)

题目：去除 Total Control PC version 2.0 小组件的功能限制  
运行平台： Windows 95/98/98SE/ME.   
软件功能：这是一款美国人写的小程序，让你调整/清理 Windows 95/98/98SE/ME，加速运行。提供了一个简单的界面，包含 10 个设置面板，定制你自己  
  
的系统行为，添加/隐藏菜单，定制浏览器，修改系统或者驱动器图标，配置内存及关键设置，提高系统速度。另外还包括 5 个小组件：Sysclean 系统垃  
  
圾清理器；Shfix：删除无效快捷方式，Descol 设置桌面颜色； Backsys 文件和系统备份程序；Foldicons 改变文件夹的图标。  
使用工具：trw, ultraedit,W32DASM ,Peid  
破解目的：去除 Sysclean 系统垃圾清理器功能限制  
  
引子：今天又装了这个小软件，看看是否有点文章做，结果真没有太大戏。这个软件是 VB 写的，没有加壳。主程序没有地方输入注册码。而且是在线注  
  
册，但我不知从哪里下手，因为根本没有输入注册码的地方。我们来看看提供的小程序 Sysclean 系统垃圾清理器，实验了一下，查找垃圾很快，可是如  
  
果想删除，立即弹出对话框告诉你，需要注册才可以使用这个功能。立即调出 SOFTICE，下命令 hwnd,找到句柄，比如 xxxx，然后下 bpx xxxx wm\_destroy  
  
，F5 退出，结果死机，看来这个程序对 SICE 有防范，另外接着用 OD 加载，也是死机。用 W32DASM 加载 OK。只好请来 TRW 了，按照上面所说，找对话框句柄  
  
，然后下消息断点，F5 退出，点击 OK，被拦住。一次 F11 即可来到主程序空间，代码如下：  
\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h  
                                  |  
:00424905 FF157C104000            Call dword ptr \[0040107C\]  
:0042490B 8D4DA4                  lea ecx, dword ptr \[ebp-5C\] //我们停在这里。查看前面是否有跳转指令可以跳过这个地方。  
下面是前面的代码：  
:0042484C 0FBF053CD04200          movsx eax, word ptr \[0042D03C\]  
:00424853 85C0                    test eax, eax  
:00424855 0F84E5000000            je 00424940     //我们可以找到这个地方，修改 84->85。  
:0042485B C745FC03000000          mov \[ebp-04\], 00000003  
:00424862 C7854CFFFFFF04000280    mov dword ptr \[ebp+FFFFFF4C\], 80020004  
:0042486C C78544FFFFFF0A000000    mov dword ptr \[ebp+FFFFFF44\], 0000000A  
:00424876 C7855CFFFFFF04000280    mov dword ptr \[ebp+FFFFFF5C\], 80020004  
:00424880 C78554FFFFFF0A000000    mov dword ptr \[ebp+FFFFFF54\], 0000000A  
:0042488A C7853CFFFFFFE8964000    mov dword ptr \[ebp+FFFFFF3C\], 004096E8  
:00424894 C78534FFFFFF08000000    mov dword ptr \[ebp+FFFFFF34\], 00000008  
:0042489E 8D9534FFFFFF            lea edx, dword ptr \[ebp+FFFFFF34\]  
:004248A4 8D8D64FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF64\]  
  
\* Reference To: MSVBVM60.\_\_vbaVarDup, Ord:0000h  
                                  |  
:004248AA FF15E4114000            Call dword ptr \[004011E4\]  
:004248B0 68DC954000              push 004095DC  
:004248B5 680C884000              push 0040880C  
  
\* Reference To: MSVBVM60.\_\_vbaStrCat, Ord:0000h  
                                  |  
:004248BA FF1550104000            Call dword ptr \[00401050\]  
:004248C0 8BD0                    mov edx, eax  
:004248C2 8D4DA4                  lea ecx, dword ptr \[ebp-5C\]  
  
\* Reference To: MSVBVM60.\_\_vbaStrMove, Ord:0000h  
                                  |  
:004248C5 FF1510124000            Call dword ptr \[00401210\]  
:004248CB 50                      push eax  
:004248CC 6870964000              push 00409670  
  
\* Reference To: MSVBVM60.\_\_vbaStrCat, Ord:0000h  
                                  |  
:004248D1 FF1550104000            Call dword ptr \[00401050\]  
:004248D7 89857CFFFFFF            mov dword ptr \[ebp+FFFFFF7C\], eax  
:004248DD C78574FFFFFF08000000    mov dword ptr \[ebp+FFFFFF74\], 00000008  
:004248E7 8D8D44FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF44\]  
:004248ED 51                      push ecx  
:004248EE 8D9554FFFFFF            lea edx, dword ptr \[ebp+FFFFFF54\]  
:004248F4 52                      push edx  
:004248F5 8D8564FFFFFF            lea eax, dword ptr \[ebp+FFFFFF64\]  
:004248FB 50                      push eax  
:004248FC 6A30                    push 00000030  
:004248FE 8D8D74FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF74\]  
:00424904 51                      push ecx  
\========================================================  
小结：这个程序太简单，功能也比较简单，去除限制也简单多了。修改这个地方后，设置的选项参数可以保存，可以轻松删除找到的垃圾文件。不再有  
  
限制了。  
  
  
qduwg  
   
2006 年 1 月 14 日完成。  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
题目：去除 WinRamTurbo 时间限制  
软件功能：优化内存等。  
  
工具：TRW，w32dasm, PEID,UltraEdit  
  
引子：今天这是实验的又一个程序。结果是 30 天限制。没有注册码输入的地方，好象直接从其网站下载注册版的。拿 PEID 检查是  
  
UPX0.89~1.02/1.05~1.24 加壳，用 UPX -d 脱壳成功。是用 BC++写的。这个软件对 SOFTICE 敏感，我只好用 TRW 了。把时间调快一个月，结果启动时先出现  
  
一个 SPLASH 小窗口，显示版权信息，按 OK 后，接着出现第二个窗口提示过期，点 OK 则立即显示第三个大窗口，点 OK 结束。我们首先干掉第二个过期提示  
  
框，用 hwnd 查看这个提示框的句柄，下断点 bpx xxxx wm\_destroy，按一次 F11 即可回到主程序内，来到这里。  
1.先去掉第 2 个 NAG。  
004083F8  |. 74 73          JE SHORT WINRAMTU.0040846D      //这个可以跳过下面的 CALL，74 改为 75.  
004083FA  |. 33D2           XOR EDX,EDX  
004083FC  |. 8B45 D0        MOV EAX,DWORD PTR SS:\[EBP-30\]  
004083FF  |. 8B80 F8020000  MOV EAX,DWORD PTR DS:\[EAX+2F8\]  
00408405  |. E8 32F60400    CALL WINRAMTU.00457A3C  
0040840A  |. 68 30024E00    PUSH WINRAMTU.004E0230  
0040840F  |. 8D95 04FFFFFF  LEA EDX,DWORD PTR SS:\[EBP-FC\]  
00408415  |. 52             PUSH EDX  
00408416  |. E8 E9300B00    CALL WINRAMTU.004BB504  
0040841B  |. 83C4 08        ADD ESP,8  
0040841E  |. 6A 00          PUSH 0  
00408420  |. FF35 84D24C00  PUSH DWORD PTR DS:\[4CD284\]  
00408426  |. 8D8D 04FFFFFF  LEA ECX,DWORD PTR SS:\[EBP-FC\]  
0040842C  |. 51             PUSH ECX  
0040842D  |. 8B45 D0        MOV EAX,DWORD PTR SS:\[EBP-30\]  
00408430  |. E8 07BA0500    CALL WINRAMTU.00463E3C  
00408435  |. 50             PUSH EAX  
00408436  |. E8 0DC30B00    CALL <JMP.&USER32.MessageBoxA>  //这个 CALL 出现过期提示。  
0040843B  |. 8B45 D0        MOV EAX,DWORD PTR SS:\[EBP-30\]      //我们停在这里。往前查看是否有地方跳过这里。  
\==========================================  
2.重新启动，结果主界面出现了，但接着出现前面所说的第二个过期提示框。如果点 OK，则退出。下 bpx messageboxa 断点，重新启动程序，被拦住，按  
  
F10 来到这里。  
00404F99  |. E8 2E1D0100    CALL WINRAMTU.00416CCC  
00404F9E  |. 85C0           TEST EAX,EAX  
00404FA0  |. 74 63          JE SHORT WINRAMTU.00405005     //这个可以跳过下面的 CALL。74 改为 75。  
00404FA2  |. 68 30024E00    PUSH WINRAMTU.004E0230  
00404FA7  |. 8D95 C8FEFFFF  LEA EDX,DWORD PTR SS:\[EBP-138\]  
00404FAD  |. 52             PUSH EDX  
00404FAE  |. E8 51650B00    CALL WINRAMTU.004BB504  
00404FB3  |. 83C4 08        ADD ESP,8  
00404FB6  |. 6A 00          PUSH 0  
00404FB8  |. FF35 84D24C00  PUSH DWORD PTR DS:\[4CD284\]  
00404FBE  |. 8D8D C8FEFFFF  LEA ECX,DWORD PTR SS:\[EBP-138\]  
00404FC4  |. 51             PUSH ECX  
00404FC5  |. 8B45 D0        MOV EAX,DWORD PTR SS:\[EBP-30\]  
00404FC8  |. E8 6FEE0500    CALL WINRAMTU.00463E3C  
00404FCD  |. 50             PUSH EAX  
00404FCE  |. E8 75F70B00    CALL <JMP.&USER32.MessageBoxA>  
00404FD3  |. 8B45 D0        MOV EAX,DWORD PTR SS:\[EBP-30\]     //我们返回到此处，看前面哪个地方可以跳过这里。  
\==========================================  
3.在退出程序的时候也有一个大 NAG，而且软件作者故意让这个窗口出现在主程序窗口的后面，没有得到焦点，需要点击一下任务栏上的其他按钮，然后  
  
再回来点击这个程序在任务栏上的按钮，才可以显示这个 NAG，然后点击 OK，退出程序。比较烦人的。现在我们就来把它拿掉好了。我也走了几个弯路，  
  
最后才灵机一动，修改成功了。  
  
首先是下断点的问题，用 OD 打开符号引用窗口，发现这个软件使用了 MessageboxA 和 MessageBeep，我用这两个函数下断点不成功，换一个方法，点击退  
  
出按钮，在这个 NAG 出现后用消息断点，我查看了句柄，使用了几个比较高级别的句柄下断点 bpx xxxx wm\_destroy，还是不成功，拦不住，接着来第三  
  
个方法，用句柄下消息断点 bpx xxxx wm\_command，我以为点击窗口上的退出按钮时，可以被拦住，结果没有一点动静，更加激发了我非要搞掉它的信心  
  
。继续琢磨了一会儿，突然灵机一动，我发现在这个软件内有一个函数 EnableWindow，就用它下断点 bpx EnableWindow，退出 SOFTICE，启动程序，点击  
  
退出按钮，OK！！被成功拦截到。随后我们按 F10，一直来到如下代码处：  
  
0044B557  |. 53                         PUSH EBX  
0044B558  |. E8 0B900700                CALL <JMP.&USER32.EnableWindow>  
0044B55D  |> 83C8 FF                    OR EAX,FFFFFFFF       //我们来到这里，上面这个 CALL 就是显示窗口的函数。  
0044B560  |. 5B                         POP EBX  
0044B561  |. 5D                         POP EBP  
0044B562  \\. C2 0800                    RETN 8     
\*\*\*\*\*\*\*\*\*\*\*\*  
我们 F10 跟踪一直来到下面代码空间，关键地方就在下面了。  
004512D3  |. E8 90A2FFFF                CALL WINRAMTU.0044B568  
004512D8  |. 8945 F4                    MOV DWORD PTR SS:\[EBP-C\],EAX  //从前面代码可以逐步来到这里！  
004512DB  |. 33D2                       XOR EDX,EDX  
004512DD  |. 55                         PUSH EBP  
004512DE  |. 68 4C144500                PUSH WINRAMTU.0045144C  
004512E3  |. 64:FF32                    PUSH DWORD PTR FS:\[EDX\]  
004512E6  |. 64:8922                    MOV DWORD PTR FS:\[EDX\],ESP  
004512E9  |. 8B45 FC                    MOV EAX,DWORD PTR SS:\[EBP-4\]  
004512EC  |. E8 3BFEFFFF                CALL WINRAMTU.0045112C        //这个 CALL 开始显示 NAG 的基本框架  
004512F1  |. 33D2                       XOR EDX,EDX  
004512F3  |. 55                         PUSH EBP  
004512F4  |. 68 AB134500                PUSH WINRAMTU.004513AB  
004512F9  |. 64:FF32                    PUSH DWORD PTR FS:\[EDX\]  
004512FC  |. 64:8922                    MOV DWORD PTR FS:\[EDX\],ESP  
004512FF  |. 6A 00                      PUSH 0  
00451301  |. 6A 00                      PUSH 0  
00451303  |. 68 00B00000                PUSH 0B000  
00451308  |. 8B45 FC                    MOV EAX,DWORD PTR SS:\[EBP-4\]  
0045130B  |. E8 2C2B0100                CALL WINRAMTU.00463E3C  
00451310  |. 50                         PUSH EAX  
00451311  |. E8 B6340700                CALL <JMP.&USER32.SendMessageA>  
00451316  |. 8B45 FC                    MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451319  |. 33D2                       XOR EDX,EDX  
0045131B  |. 8990 2C020000              MOV DWORD PTR DS:\[EAX+22C\],EDX  
00451321  |> 8B03                       /MOV EAX,DWORD PTR DS:\[EBX\]   //循环体开始，制造 NAG。  
00451323  |. E8 382D0000                |CALL WINRAMTU.00454060  
00451328  |. 8B03                       |MOV EAX,DWORD PTR DS:\[EBX\]  
0045132A 80B88400000000          cmp byte ptr \[eax+00000084\], 00      //这行和下面这行可以做点文章的地方了。  
00451331 740F                    je 00451342                          //让这里跳到循环体后面即可。具体修改见后面。  
00451333  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451336  |. C780 2C020000 02000000     |MOV DWORD PTR DS:\[EAX+22C\],2  
00451340  |. EB 14                      |JMP SHORT WINRAMTU.00451356  
00451342  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451345  |. 83B8 2C020000 00           |CMP DWORD PTR DS:\[EAX+22C\],0  
0045134C  |. 74 08                      |JE SHORT WINRAMTU.00451356  
0045134E  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451351  |. E8 26FDFFFF                |CALL WINRAMTU.0045107C  
00451356  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451359  |. 8B80 2C020000              |MOV EAX,DWORD PTR DS:\[EAX+22C\]  
0045135F  |. 85C0                       |TEST EAX,EAX  
00451361  |.⁷⁴ BE                      \\JE SHORT WINRAMTU.00451321    //循环体结束。在这个循环体内一点一点地取窗口内容显示到前面的那  
  
个 NAG 窗体内。现在就是要把这个循环体跳过去。  
00451363  |. 8945 F8                    MOV DWORD PTR SS:\[EBP-8\],EAX   //上面循环体显示窗口结束，来到这里。等你按按钮。如果按 OK 按钮，  
  
则 EAX=1，则下面代码就是退出程序。还有一个 purchase（购买 ）按钮，则 EAX=2。  
00451366  |. 6A 00                      PUSH 0  
00451368  |. 6A 00                      PUSH 0  
0045136A  |. 68 01B00000                PUSH 0B001  
0045136F  |. 8B45 FC                    MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451372  |. E8 C52A0100                CALL WINRAMTU.00463E3C  
我在跟踪的时候注意到循环结束后，NAG 出现，如果你点击 OK，则重新被断下，来到 00451363，EAX 的值为 1。所以，我们修改前面的跳转的时候，一定  
  
记得给 EAX 赋值为 1。因为后面的函数需要这个参数。所以前面的那个跳转指令修改成下面这个样子了：  
0045132A     B8 01000000                MOV EAX,1  
0045132F     90                         NOP  
00451330     90                         NOP  
00451331     EB 30                    JMP SHORT WINRAMTU.00451363  
原来的程序代码指令空间刚好可以修改成新指令，绰绰有余啊！非常棒！立即用 ULTRAEDIT 修改吧！然后另行改名保存文件，以防你修改失败哟！！现在  
  
启动程序运行，然后关闭，OK！没有 NAG 了，非常干脆利落的关掉了。  
附一个失败方案：本来想从 4512D8 处把指令直接修改为 JMP 451359，同时 451359 和 45135A 处修改为 MOV EAX,1 ，结果程序无法启动了，而且导致死机了  
  
。我知道在 4512D8 到 00451321 之间的代码是必须的，不可以掠过去的。所以修改程序无法预见具体结果，必须实验再实验。  
\==========================================  
4.现在已经启动已经没有问题了，不过在启动时还有一个小 NAG，随着上面对最后一个大 NAG 的去除，我发现启动的时候第一个 SPLASH 版权窗口自动一闪  
  
而过，不再停在那里等你按 OK 了。好生奇怪啊！这是修改的副产品！捎带把第一个也消除了。实属必然！！破解不是目的，知道如何破解才是重要的。  
  
我觉得还是研究一下如何把它去掉。  
用 SOFTICE LOADER 加载程序，F10 跟踪到下面 004BFA87 处出现版权信息窗口，下断点，然后再次加载程序，F5 退出 SOFTICE，被拦住在 004BFA87 处，F8  
  
跟入，部分跟踪代码在后面。  
004BFA81  |. E8 3A450000    CALL <JMP.&KERNEL32.GetModuleHandleA>  
004BFA86  |. 50             PUSH EAX  
004BFA87  |. FF56 18        CALL DWORD PTR DS:\[ESI+18\]   //这个 CALL 出那个 SPLASH。:\[ESI+18\]的值为 4012C0。  
004BFA8A  |. 83C4 10        ADD ESP,10  
004BFA8D  |. 50             PUSH EAX  
004BFA8E  |. E8 BDF4FFFF    CALL WINRAMTU.004BEF50  
004BFA93  |. 59             POP ECX  
004BFA94  |. EB 21          JMP SHORT WINRAMTU.004BFAB7  
004BFA96  |> 8B15 E8894E00  MOV EDX,DWORD PTR DS:\[4E89E8\]  
004BFA9C  |. 52             PUSH EDX  
004BFA9D  |. 8B0D E4894E00  MOV ECX,DWORD PTR DS:\[4E89E4\]  
004BFAA3  |. 51             PUSH ECX  
004BFAA4  |. A1 E0894E00    MOV EAX,DWORD PTR DS:\[4E89E0\]  
004BFAA9  |. 50             PUSH EAX  
004BFAAA  |. FF56 18        CALL DWORD PTR DS:\[ESI+18\]   //这个 CALL 出那个 SPLASH。  
004BFAAD  |. 83C4 0C        ADD ESP,0C  
我们 F8 跟入前面那个 CALL，用 F10 沿着下面这个路径跟踪：  
00401554   . 8B0D 94E44D00              MOV ECX,DWORD PTR DS:\[4DE494\]  
0040155A   . 8B01                       MOV EAX,DWORD PTR DS:\[ECX\]  
0040155C   . 8B0D 58E44D00              MOV ECX,DWORD PTR DS:\[4DE458\]  
00401562   . 8B15 749D4C00              MOV EDX,DWORD PTR DS:\[4C9D74\]  
00401568   . E8 072C0500                CALL WINRAMTU.00454174      //这个 CALL 用 F8 切入跟踪，F10 则出现窗口。  
0040156D   . A1 94E44D00                MOV EAX,DWORD PTR DS:\[4DE494\]  
00401572   . 8B00                       MOV EAX,DWORD PTR DS:\[EAX\]  
00401574   . E8 7B2C0500                CALL WINRAMTU.004541F4  
00401579   . 66:C745 C4 0000            MOV WORD PTR SS:\[EBP-3C\],0  
0040157F   . EB 1B                      JMP SHORT WINRAMTU.0040159C  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
顺其自然，我们来到下面代码处：  
0040FF13  |. 8B10                       MOV EDX,DWORD PTR DS:\[EAX\]  
0040FF15  |. A1 78E44D00                MOV EAX,DWORD PTR DS:\[4DE478\]  
0040FF1A  |. 8B08                       MOV ECX,DWORD PTR DS:\[EAX\]  
0040FF1C  |. 8B81 D8020000              MOV EAX,DWORD PTR DS:\[ECX+2D8\]  
0040FF22  |. E8 B9E10400                CALL WINRAMTU.0045E0E0  
0040FF27  |. FF4D EC                    DEC DWORD PTR SS:\[EBP-14\]  
0040FF2A  |. 8D45 FC                    LEA EAX,DWORD PTR SS:\[EBP-4\]  
0040FF2D  |. BA 02000000                MOV EDX,2  
0040FF32  |. E8 39390B00                CALL WINRAMTU.004C3870  
0040FF37  |. 8B0D 78E44D00              MOV ECX,DWORD PTR DS:\[4DE478\]  
0040FF3D  |. 8B01                       MOV EAX,DWORD PTR DS:\[ECX\]  
0040FF3F  |. 8B10                       MOV EDX,DWORD PTR DS:\[EAX\]  
0040FF41  |. FF92 CC000000              CALL DWORD PTR DS:\[EDX+CC\] //这个 CALL 出现版权窗口，F8 跟入。:\[EDX+CC\]的值为 4511DC。这段代码其  
  
实就是前面第三部分所述的代码段了。说明 2 个窗口都调用同样的一个函数啦！部分节选如下：  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
004511DC  /. 55                         PUSH EBP  
004511DD  |. 8BEC                       MOV EBP,ESP  
004511DF  |. 83C4 E0                    ADD ESP,-20  
004511E2  |. 53                         PUSH EBX  
004511E3  |. 56                         PUSH ESI  
004511E4  |. 33D2                       XOR EDX,EDX  
004511E6  |. 8955 E0                    MOV DWORD PTR SS:\[EBP-20\],EDX  
004511E9  |. 8945 FC                    MOV DWORD PTR SS:\[EBP-4\],EAX  
\*省去 260 多行代码  
004512D3  |. E8 90A2FFFF                CALL WINRAMTU.0044B568  
004512D8  |. 8945 F4                    MOV DWORD PTR SS:\[EBP-C\],EAX  
004512DB  |. 33D2                       XOR EDX,EDX  
004512DD  |. 55                         PUSH EBP  
004512DE  |. 68 4C144500                PUSH WINRAMTU.0045144C  
004512E3  |. 64:FF32                    PUSH DWORD PTR FS:\[EDX\]  
004512E6  |. 64:8922                    MOV DWORD PTR FS:\[EDX\],ESP  
004512E9  |. 8B45 FC                    MOV EAX,DWORD PTR SS:\[EBP-4\]  
004512EC  |. E8 3BFEFFFF                CALL WINRAMTU.0045112C     //这个 CALL 出现版权窗口。同样关闭程序时的那个 BIG NAG 也调用这个函数  
  
显示主框架，如前所述。  
下面我给出另外一个简单的解决办法，就是把下面这个消息循环切断，不让它循环即可，最后两个 NAG 统统一闪即逝，根本影响不了什么了。：）  
00451321  |> 8B03                       /MOV EAX,DWORD PTR DS:\[EBX\]  
00451323  |. E8 382D0000                |CALL WINRAMTU.00454060  
00451328  |. 8B03                       |MOV EAX,DWORD PTR DS:\[EBX\]  
0045132A     80B8 84000000 00           CMP BYTE PTR DS:\[EAX+84\],0  
00451331     74 0F                      JE SHORT WINRAMTU.00451342  
00451333  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451336  |. C780 2C020000 02000000     |MOV DWORD PTR DS:\[EAX+22C\],2  
00451340  |. EB 14                      |JMP SHORT WINRAMTU.00451356  
00451342  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451345  |. 83B8 2C020000 00           |CMP DWORD PTR DS:\[EAX+22C\],0  
0045134C  |. 74 08                      |JE SHORT WINRAMTU.00451356  
0045134E  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451351  |. E8 26FDFFFF                |CALL WINRAMTU.0045107C  
00451356  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:\[EBP-4\]  
00451359  |. 8B80 2C020000              |MOV EAX,DWORD PTR DS:\[EAX+22C\]  
0045135F  |. 85C0                       |TEST EAX,EAX  
00451361  |.⁷⁴ BE                      \\JE SHORT WINRAMTU.00451321    //只改动半个字节即可，74->75，OK。循环链条被切断，程序继续往下  
  
走，看它还显示 NAG 骚扰我们。嘿嘿！：）收工！休息！  
\==========================================  
后记：我忙里偷闲终于把这个东西搞定了，现在运行起来真是清爽怡人啊！赶紧写出破文，分享我的快乐吧！当然能够带给你小小的经验和启发，那真  
  
是求之不得的事情了！最近看到许多热心大侠关注我的教程，也许他们从中收获了点什么，也是给我最大的安慰和鼓励，自己的辛苦算不了什么了！激  
  
励我写完一个还想下一个！再次感谢各位 CRACKER 道路上的盟友们！！同时也感谢看雪论坛各位高手的文章！你们的文章就是我们进步的阶梯！！  
我也是在向高手层次冲击的一个“敢死队员”。希望我们的目标早日实现！！  
  
这个软件功能一般，但是搞了那么多的 NAG，烦煞人也！不除掉不为快！！：）不过有点小遗憾，因为版权 NAG 和退出时的 NAG 及主窗口都共享同一段代码  
  
，通过消息传递实现窗口显示，所以牵一发而动全身。上面的修改虽然去掉了 NAG，但是同时也使得 2 个按钮功能失效，幸亏这 2 个按钮的功能不重要，去  
  
掉也就去掉也。：）本来想搞个注册码算法分析，结果没有收获，却收获了上面这些东西。也算不错啦！！进步也需要时间和经验！慢慢来！高手不是  
  
一天铸就的，长城不是一天垒起的！  
  
自己高兴之余，作诗一首，谨送给看雪论坛的各位大侠和菜鸟坛友，以兹共勉！  
  
书山有路勤为径，  
学海无涯苦做舟。  
共谋破解之大计，  
关键时处显身手！  
  
看雪论坛常做客，  
以文会友乐趣多。  
熟读精华三千篇，  
不会注册也会破。  
  
菜鸟如何变高手，  
天天练习不可缺。  
先来爆破后分析，  
摸清敌情易行动。  
  
解密软件费时间，  
还有运气在里边。  
破解不了不气馁，  
成功分析莫自满！  
  
破解如同练武功，  
开始心手都有剑。  
再是手有心却无，  
最后心有手中空。  
  
山外青山楼外楼，  
大侠外边有高手。  
破解路上莫独行，  
还望关照菜鸟虫。  
  
好风凭借力，破解逍遥行。  
美文如甘露，坛友似弟兄。  
  
破解不分长幼，只有经验不同。  
闻道自有先后，术业自有专攻。  
他山之石攻玉，取长补己之短。  
今日还是菜鸟，明天就是高手。  
  
  
QduWg  
qduwg@163.com  
  
2006 年 1 月 16 日完稿 

 4dd466d0e1e0cc123afbe504da2462ca