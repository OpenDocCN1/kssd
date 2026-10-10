

# Winline 1.21

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-253.htm`](https://www.pediy.com/kssd/pediy08/pediy8-253.htm)

【文章标题】: 一次带有疑问的“pediy”  
【文章作者】: rdsnow\[BCG\]\[PYG\]\[D.4s\]  
【作者邮箱】: rdsnow@163.com  
【作者主页】: http://rdsnow.ys168.com  
【作者 QQ 号】: 83757177  
【下载地址】: Winline121.rar  
【使用工具】: OllyICE、stud\_pe、ResourceHacker、金山游侠  
【软件介绍】: Winline 1.21 一个五子连线的彩球游戏  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
【文章简介】  
  
人生在世，有三件大事，“上网、女人和玩游戏”，可惜我们办公室内几个人合用一台电脑。汗！  
  
上次谁在休闲区放了这个小游戏，我水平太差了，玩这个游戏就好像等死的病人一样，看着桌面的小球越来越多，更加可气的是办公室同事们不知怎的，迷上了这个小游戏，一个接一个的上，竟然挤占了不少上网冲浪的时间。  
  
把游戏删了，不！君子如何能干这事(其实删了，同事会重新下载)，看来得给游戏动些手术，得让他们感觉到游戏太简单，没有意思！呵呵！  
  
学习了 andy00 提供的记事本修改系列，轮到自己动手了，现准备为游戏添加这样几个秘笈：(做好以后，不要忘记把这些故意透露给同事)  
  
1、按 F6 ，分数就加 100  
2、按 F7 ，小球将不再向下掉  
3、按 F8 ，恢复每次向下掉的小球数为 3  
4、在菜单中加入清除记录文件的功能  
5、为程序添加老板键功能  
  
学习了 andy00 提供的记事本修改系列，轮到自己动手了，没想到给自己带来了不少疑问，把问题归结到文章的最后。希望高手帮忙解释一下。  
\--------------------------------------------------------------------------------  
【破解过程】  
  
第一步：找到分值保存的地址：  
  
这个是 N 年前的小把戏了，通过不断的缩小范围，游侠最后定位所得分数保存在 dword ptr ds:\[430144\] 中，赶快用 OD 将游戏附上，预防分数保存在动态地址中，然后在\[430144\]上下硬件写断点，移动球直到拿分，果然被断下：  
  
00411318   .  89C6          MOV ESI,EAX  
0041131A   .  8B1D 44014300 MOV EBX,DWORD PTR DS:\[430144\]  
00411320   .  01D3          ADD EBX,EDX  
00411322   .  891D 44014300 MOV DWORD PTR DS:\[430144\],EBX  
00411328   .  89D9          MOV ECX,EBX                            ;  停在这里  
0041132A   .  8B1D 4C014300 MOV EBX,DWORD PTR DS:\[43014C\]  
00411330   .  8B15 48014300 MOV EDX,DWORD PTR DS:\[430148\]  
00411336   .  E8 37FAFFFF   CALL Winline1.00410D72  
  
这几行代码清楚的告诉我们，分值保存在 \[430144\] 这个固定地址中。  
  
第二步：粗略跟下这些代码：  
  
00411D69   > \\31DB          XOR EBX,EBX  
00411D6B   .  8A1D 18004300 MOV BL,BYTE PTR DS:\[430018\]  
00411D71   .  31D2          XOR EDX,EDX  
00411D73   .  8A146D 940043>MOV DL,BYTE PTR DS:\[EBP\*2+430094\]  
00411D7A   .  31C0          XOR EAX,EAX  
00411D7C   .  8A046D 950043>MOV AL,BYTE PTR DS:\[EBP\*2+430095\]  
00411D83   .  E8 64EAFFFF   CALL Winline1.004107EC                 ;  判断有没有五子成线  
00411D88   .  89C5          MOV EBP,EAX  
00411D8A   .  85C0          TEST EAX,EAX  
00411D8C   .  74 24         JE SHORT Winline1.00411DB2             ;  没有成线，则跳去分发小球  
00411D8E   .  89F8          MOV EAX,EDI  
00411D90   .  E8 E1EBFFFF   CALL Winline1.00410976  
00411D95   .  89E8          MOV EAX,EBP  
00411D97   .  E8 2AEEFFFF   CALL Winline1.00410BC6                 ;  根据成线小球的个数计算应增加的分值  
00411D9C   .  89C2          MOV EDX,EAX                            ; EDX = 所要加的分值  
00411D9E   .  89F8          MOV EAX,EDI  
00411DA0   .  E8 6EF5FFFF   CALL Winline1.00411313                 ;  增加分值，并且动画调整小人的高度  
00411DA5   .^ E9 D8FCFFFF   JMP Winline1.00411A82  
00411DAA   >  89F8          MOV EAX,EDI                            ;  /  
00411DAC   .  E8 C0ECFFFF   CALL Winline1.00410A71                 ;  |重新分发小球  
00411DB1   .  45            INC EBP                                ;  |  
00411DB2   >  83FD 03       CMP EBP,3                              ;  |循环三次，即分发三个小球  
00411DB5   .^ 7C F3         JL SHORT Winline1.00411DAA             ;  \\  
00411DB7   .  E8 7EECFFFF   CALL Winline1.00410A3A                 ;  判断有没有空格  
00411DBC   .  85C0          TEST EAX,EAX  
00411DBE   .  75 2C         JNZ SHORT Winline1.00411DEC            ;  有空格继续，没有则跳下面对话框  
00411DC0   .  6A 30         PUSH 30                                ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
00411DC2   .  68 90004200   PUSH Winline1.00420090                 ; |Title = "Color linez"  
00411DC7   .  68 DA8C4200   PUSH Winline1.00428CDA                 ; |Text = " 玩  完  !"  
00411DCC   .  8BAC24 880000>MOV EBP,DWORD PTR SS:\[ESP+88\]          ; |  
00411DD3   .  55            PUSH EBP                               ; |hOwner  
00411DD4   .  E8 E6130000   CALL <JMP.&USER32.MessageBoxA>         ; \\MessageBoxA  
00411DD9   .  89E8          MOV EAX,EBP  
00411DDB   >  E8 E5F9FFFF   CALL Winline1.004117C5  
00411DE0   .  89F8          MOV EAX,EDI  
00411DE2   .  E8 3EF7FFFF   CALL Winline1.00411525  
00411DE7   .^ E9 96FCFFFF   JMP Winline1.00411A82  
  
刚才的硬件写断就断在 00411DA0 CALL Winline1.00411313 内，这个函数的功能将\[430144\]的分值增加，同时在窗口画出一个编辑框，然后：  
  
跟进 00411336 CALL Winline1.00410D72：  
00410E1A  |.  68 FFFFFF00   PUSH 0FFFFFF                           ; /Color = <WHITE>  
00410E1F  |.  56            PUSH ESI                               ; |hDC  
00410E20  |.  E8 88230000   CALL <JMP.&GDI32.SetTextColor>         ; \\SetTextColor  
00410E25  |.  6A 01         PUSH 1                                 ; /BkMode = TRANSPARENT  
00410E27  |.  56            PUSH ESI                               ; |hDC  
00410E28  |.  E8 7A230000   CALL <JMP.&GDI32.SetBkMode>            ; \\SetBkMode  
00410E2D  |.  8D5424 10     LEA EDX,DWORD PTR SS:\[ESP+10\]  
00410E31  |.  8B4424 30     MOV EAX,DWORD PTR SS:\[ESP+30\]  
00410E35  |.  E8 6EF2FFFF   CALL Winline1.004100A8  
00410E3A  |.  6A 02         PUSH 2                                 ; /Flags = DT\_RIGHT|DT\_TOP  
00410E3C  |.  8D4424 04     LEA EAX,DWORD PTR SS:\[ESP+4\]           ; |  
00410E40  |.  50            PUSH EAX                               ; |pRect  
00410E41  |.  6A FF         PUSH -1                                ; |Count = FFFFFFFF (-1.)  
00410E43  |.  8D4424 1C     LEA EAX,DWORD PTR SS:\[ESP+1C\]          ; |  
00410E47  |.  50            PUSH EAX                               ; |Text  
00410E48  |.  56            PUSH ESI                               ; |hDC  
00410E49  |.  836C24 1C 08  SUB DWORD PTR SS:\[ESP+1C\],8            ; |  
00410E4E  |.  E8 4E230000   CALL <JMP.&USER32.DrawTextA>           ; \\DrawTextA  
  
显示分值，最后根据分数计算小人的高度，并调整之。  
  
第三步：前期思考准备  
  
1、按 F6 ，分数就加 100：  
  
不要直接改 \[430144\] 的数值，那样虽然分数加了，但编辑框分值并不改变，小人高度也不随之变化。  
  
00411D9C   .  89C2          MOV EDX,EAX                            ; EDX = 所要加的分值  
00411D9E   .  89F8          MOV EAX,EDI  
00411DA0   .  E8 6EF5FFFF   CALL Winline1.00411313                 ;  增加分值，并且动画调整小人的高度  
  
这个 call 可以被我们利用下  
  
2、按 F7 ，小球将不再向下掉：  
  
00411DAA   >  89F8          MOV EAX,EDI                            ;  /  
00411DAC   .  E8 C0ECFFFF   CALL Winline1.00410A71                 ;  |重新分发小球  
00411DB1   .  45            INC EBP                                ;  |  
00411DB2   >  83FD 03       CMP EBP,3                              ;  |循环三次，即分发三个小球  
00411DB5   .^ 7C F3         JL SHORT Winline1.00411DAA             ;  \\  
  
只需将 CMP EBP,3 改为 CMP EBP,0 小球再也不会向下掉。  
  
3、按 F8 ，恢复每次向下掉的小球数为 3  
  
和上面相反，改回 CMP EBP,3 即可。  
  
4、在菜单中加入清除记录文件的功能  
  
记录文件保存在程序目录下的 winlines.res 中，用二进制编辑软件打开可以看到记录文件的结果，可以随意修改名字和分值。将这个文件删除即可以清除记录。需要给程序添加 kernel32.DeleteFileA 函数。  
  
5、为程序添加老板键功能  
  
分两步完成，增加 WM\_RBOTTODOWN 事件的处理功能，在其中添加 ShowWindow 的隐藏功能，(这个输入功能程序中已有，无需自行加)，同时给程序安装日志钩子，这是修改最失败的地方，原本是想在窗口初始化时安装钩子，后来考虑到使用全局钩子会拖慢系统的速度，老板不来，老板键是用不到的，所以在隐藏窗口的同时安装钩子。  
  
一旦主窗口被隐藏，窗口则处于非活动状态，不能接受到 WM\_KEYDOWN 消息，所以安装了钩子，在钩子回调功能中显示被隐藏了的窗口。  
  
通过上面两步实现老板键功能。  
  
什么是钩子：“A hook is a point in the Microsoft Windows message-handling mechanism  
 where an application can install a subroutine to monitor the message traffic in the   
system and process certain types of messages before they reach the target window procedure.”  
  
我英文比较烂，只好猜想：安装了钩子就好像在大街上装个监视器，然后从人流中找到适合自己的 MM，如果找到，就拉过来聊聊。  
  
第四步：添加相应资源  
  
用资源黑客打开以上程序，添加一下快捷键：  
  
VK\_F6, 780, VIRTKEY     ;ID=0x30C，对应 F6 键，用于启动秘笈 1  
VK\_F7, 781, VIRTKEY     ;ID=0x30D，对应 F7 键，用于启动秘笈 2  
VK\_F8, 782, VIRTKEY     ;ID=0x30E，对应 F8 键，用于启动秘笈 3  
  
上面的几个既然是秘笈，就不要添加菜单了。然后继续添加菜单：  
  
MENUITEM "删除记录文件(&H)",  783                        ; ID=0x30F，用于启动秘笈 4  
POPUP "查看老板键(&B)"  
{  
  MENUITEM "鼠标右键  隐藏窗口",  784,  GRAYED     ; 只是放了一个菜单，没有为之添加代码  
  MENUITEM SEPARATOR  
  MENUITEM "键盘 A 键 恢复窗口",  785,  GRAYED     ; 只是放了一个菜单，没有为之添加代码  
}  
  
不知道源程序何以能做到"帮助"和其他如"文件"、"选项"等菜单分开的，资源重新编译后这个菜单有靠拢一起了。  
  
第五步：使用 stud\_PE 添加相应新的区段和 API 输入函数  
  
区段改好了，如下：  
  
No  | 名称      | VSize      | VOffset    | RSize      | ROffset    | Charact.   |   
01  | .text     | 00003400   | 00010000   | 00003400   | 00000400   | E0000020   | 改为可写  
02  | .data     | 00008E00   | 00020000   | 00008E00   | 00003800   | D0000040   |   
03  | .bss      | 00001200   | 00030000   | 00001200   | 00000000   | C0000080   |   
04  | .idata    | 00000800   | 00040000   | 00000800   | 0000C600   | C0000040   |   
05  | .reloc    | 00000600   | 00050000   | 00000600   | 0000CE00   | 42000040   |   
06  | .rsrc     | 0000248D   | 00060000   | 00002600   | 0000D400   | D0000040   |   
07  | .hook     | 00001000   | 00070000   | 00001000   | 0000FA00   | F0000060   | 自己加的段  
08  | .newimp   | 00010000   | 00080000   | 00010000   | 00010A00   | C0000040   | 增加函数添加的段  
  
.hook 是我新建的一个区段，用于放自己的一些代码  
.newimp 是下面增加输入函数的时候 stud\_PE 添加的区段，我晕，这个区段的 RSize 竟然如此大。  
  
添加区段时记得要勾选用 NULL 填充空间，省得再用二进制编辑工具添加，另外因要用代码修改 .text 中的数据，所以不要忘记了把 .text 段属性改为可写。否则下面的修改会出错。  
  
直接用 stud\_PE 添加下列函数：  
  
user32.dll  
  SetWindowsHookExA  ord:0 rva: 00080090  
  UnhookWindowsHookEx  ord:0 rva: 00080094  
  CallNextHookEx  ord:0 rva: 00080098  
    
kernel32.dll  
  DeleteFileA  ord:0 rva: 000800A0  
  
另外，随意跟进一个 API 的调用，会来到很多 JMP 并排放在一起的代码，如下面：  
  
004130B7   $- FF25 A4004400 JMP DWORD PTR DS:\[<&GDI32.CreateDIB>;  GDI32.CreateDIBitmap  
004130BD   $- FF25 AC004400 JMP DWORD PTR DS:\[<&GDI32.CreatePal>;  GDI32.CreatePalette  
004130C3   $- FF25 78024400 JMP DWORD PTR DS:\[<&KERNEL32.Global>;  kernel32.GlobalFree  
…………………………  
还有很多  
  
在这些 jmp 的最后加上我们的：  
  
0041325B   .- FF25 90004800  JMP DWORD PTR DS:\[<&user32.SetWindowsHookExA>\] ;  USER32.SetWindowsHookExA  
00413261   .- FF25 94004800  JMP DWORD PTR DS:\[<&user32.UnhookWindowsHookEx>;  USER32.UnhookWindowsHookEx  
00413267   .- FF25 98004800  JMP DWORD PTR DS:\[<&user32.CallNextHookEx>\]    ;  USER32.CallNextHookEx  
0041326D   $- FF25 A0004800  JMP DWORD PTR DS:\[<&kernel32.DeleteFileA>\]     ;  kernel32.DeleteFileA  
  
以后，若要调用 DeleteFileA ，直接 Call 41326D 就行了。  
  
第六步：修改代码实现前四个秘笈：  
  
先要找到窗口函数，OD 载入程序后停在这里：  
  
00410000 >/$  6A 0A         PUSH 0A  
00410002  |.  E8 48320000   CALL <JMP.&KERNEL32.GetCommandLineA>   ; \[GetCommandLineA  
00410007  |.  50            PUSH EAX  
00410008  |.  6A 00         PUSH 0                                 ; /pModule = NULL  
0041000A  |.  E8 46320000   CALL <JMP.&KERNEL32.GetModuleHandleA>  ; \\GetModuleHandleA  
0041000F  |.  6A 00         PUSH 0  
00410011  |.  50            PUSH EAX  
00410012  |.  E8 61010000   CALL Winline1.00410178  
00410017  |.  50            PUSH EAX                               ; /ExitCode  
00410018  \\.  E8 2C320000   CALL <JMP.&KERNEL32.ExitProcess>       ; \\ExitProcess  
  
跟进 00410012 CALL Winline1.00410178  
  
00410178  /$  53            PUSH EBX  
00410179  |.  56            PUSH ESI  
0041017A  |.  57            PUSH EDI  
0041017B  |.  55            PUSH EBP  
0041017C  |.  83EC 44       SUB ESP,44  
0041017F  |.  8B7C24 58     MOV EDI,DWORD PTR SS:\[ESP+58\]  
00410183  |.  893D 28004300 MOV DWORD PTR DS:\[430028\],EDI          ;  save hInstance  
00410189  |.  897C24 10     MOV DWORD PTR SS:\[ESP+10\],EDI  
0041018D  |.  C74424 24 200>MOV DWORD PTR SS:\[ESP+24\],00420020     ;  ASCII "LinezWindow"  
00410195  |.  C74424 04 7E1>MOV DWORD PTR SS:\[ESP+4\],0041187E      ;  窗口函数地址  
0041019D  |.  C70424 230000>MOV DWORD PTR SS:\[ESP\],23  
004101A4  |.  68 B80B0000   PUSH 0BB8                              ; /RsrcName = 3000.  
004101A9  |.  57            PUSH EDI                               ; |hInst  
004101AA  |.  E8 94300000   CALL <JMP.&USER32.LoadIconA>           ; \\LoadIconA  
004101AF  |.  894424 14     MOV DWORD PTR SS:\[ESP+14\],EAX  
004101B3  |.  68 007F0000   PUSH 7F00                              ; /RsrcName = IDC\_ARROW  
004101B8  |.  6A 00         PUSH 0                                 ; |hInst = NULL  
004101BA  |.  E8 7E300000   CALL <JMP.&USER32.LoadCursorA>         ; \\LoadCursorA  
004101BF  |.  894424 18     MOV DWORD PTR SS:\[ESP+18\],EAX  
004101C3  |.  C74424 20 4C0>MOV DWORD PTR SS:\[ESP+20\],44C  
004101CB  |.  31ED          XOR EBP,EBP  
004101CD  |.  896C24 08     MOV DWORD PTR SS:\[ESP+8\],EBP  
004101D1  |.  896C24 0C     MOV DWORD PTR SS:\[ESP+C\],EBP  
004101D5  |.  6A 04         PUSH 4                                 ; /ObjType = BLACK\_BRUSH  
004101D7  |.  E8 5B300000   CALL <JMP.&GDI32.GetStockObject>       ; \\GetStockObject  
004101DC  |.  894424 1C     MOV DWORD PTR SS:\[ESP+1C\],EAX  
004101E0  |.  89E0          MOV EAX,ESP  
004101E2  |.  50            PUSH EAX                               ; /pWndClass  
004101E3  |.  E8 49300000   CALL <JMP.&USER32.RegisterClassA>      ; \\RegisterClassA  
004101E8  |.  66:85C0       TEST AX,AX  
004101EB  |.  75 07         JNZ SHORT Winline1.004101F4  
004101ED  |.  31C0          XOR EAX,EAX  
004101EF  |.  E9 FB000000   JMP Winline1.004102EF  
004101F4  |>  C705 38014300>MOV DWORD PTR DS:\[430138\],92  
004101FE  |.  C705 3C014300>MOV DWORD PTR DS:\[43013C\],44  
00410208  |.  55            PUSH EBP                               ; /Index  
00410209  |.  E8 1D300000   CALL <JMP.&USER32.GetSystemMetrics>    ; \\GetSystemMetrics  
0041020E  |.  A3 2C004300   MOV DWORD PTR DS:\[43002C\],EAX  
00410213  |.  6A 01         PUSH 1                                 ; /Index = SM\_CYSCREEN  
00410215  |.  E8 11300000   CALL <JMP.&USER32.GetSystemMetrics>    ; \\GetSystemMetrics  
0041021A  |.  A3 30004300   MOV DWORD PTR DS:\[430030\],EAX  
0041021F  |.  55            PUSH EBP                               ; /lParam  
00410220  |.  57            PUSH EDI                               ; |hInst  
00410221  |.  55            PUSH EBP                               ; |hMenu  
00410222  |.  55            PUSH EBP                               ; |hParent  
00410223  |.  68 A4010000   PUSH 1A4                               ; |Height = 1A4 (420.)  
00410228  |.  68 6C020000   PUSH 26C                               ; |Width = 26C (620.)  
0041022D  |.  8D90 5CFEFFFF LEA EDX,DWORD PTR DS:\[EAX-1A4\]         ; |  
00410233  |.  89D0          MOV EAX,EDX                            ; |  
00410235  |.  C1FA 1F       SAR EDX,1F                             ; |  
00410238  |.  2BC2          SUB EAX,EDX                            ; |  
0041023A  |.  D1F8          SAR EAX,1                              ; |  
0041023C  |.  50            PUSH EAX                               ; |Y  
0041023D  |.  8B15 2C004300 MOV EDX,DWORD PTR DS:\[43002C\]          ; |  
00410243  |.  81EA 6C020000 SUB EDX,26C                            ; |  
00410249  |.  89D0          MOV EAX,EDX                            ; |  
0041024B  |.  C1FA 1F       SAR EDX,1F                             ; |  
0041024E  |.  2BC2          SUB EAX,EDX                            ; |  
00410250  |.  D1F8          SAR EAX,1                              ; |  
00410252  |.  50            PUSH EAX                               ; |X  
00410253  |.  68 0000CF00   PUSH 0CF0000                           ; |Style = WS\_OVERLAPPED|WS\_MINIMIZEBOX|WS\_MAXIMIZEBOX|>  
00410258  |.  68 26334100   PUSH Winline1.00413326                 ; |WindowName = "Color linez 1.21 rdsnow\[BCG\]\[PYG\]\[D.4s\]"  
0041025D  |.  68 20004200   PUSH Winline1.00420020                 ; |Class = "LinezWindow"  
00410262  |.  55            PUSH EBP                               ; |ExtStyle  
00410263  |.  E8 BD2F0000   CALL <JMP.&USER32.CreateWindowExA>     ; \\CreateWindowExA  
00410268  |.  89C6          MOV ESI,EAX  
0041026A  |.  89C3          MOV EBX,EAX  
0041026C  |.  A3 3C024300   MOV DWORD PTR DS:\[43023C\],EAX          ;  save hWnd，记住保存 hWnd 的地址  
00410271  |.  68 4C040000   PUSH 44C                               ; /TableName = 44C  
00410276  |.  57            PUSH EDI                               ; |hInst  
00410277  |.  E8 A32F0000   CALL <JMP.&USER32.LoadAcceleratorsA>   ; \\LoadAcceleratorsA  
0041027C  |.  A3 34004300   MOV DWORD PTR DS:\[430034\],EAX  
00410281  |.  8B4C24 64     MOV ECX,DWORD PTR SS:\[ESP+64\]  
00410285  |.  51            PUSH ECX                               ; /ShowState  
00410286  |.  56            PUSH ESI                               ; |hWnd  
00410287  |.  E8 8D2F0000   CALL <JMP.&USER32.ShowWindow>          ; \\ShowWindow  
  
我们只要注意以上红色部分，我们可以得到：hInstance 保存在\[430028\]中，hWnd 保存在\[43023C\]中，而 0041187E 则是窗口函数地址：  
  
跟到窗口函数：  
  
0041187E   .  53            PUSH EBX  
0041187F   .  56            PUSH ESI  
00411880   .  57            PUSH EDI  
00411881   .  55            PUSH EBP  
00411882   .  83EC 68       SUB ESP,68  
00411885   .  8B9C24 800000>MOV EBX,DWORD PTR SS:\[ESP+80\]          ;  取得消息种类  
0041188C   .  8BB424 880000>MOV ESI,DWORD PTR SS:\[ESP+88\]  
  
在地址 41188C 上，下条件断点：bp 41188c if ebx==111，然后随便按下一个菜单，跟着找到：  
  
00411FA3   > /8B8424 840000>MOV EAX,DWORD PTR SS:\[ESP+84\]          ;  Case 111 of switch 004118B1  
00411FAA   . |66:3D 0002    CMP AX,200  
00411FAE   . |72 28         JB SHORT Winline1.00411FD8  
00411FB0   .^|0F86 F5FEFFFF JBE Winline1.00411EAB  
00411FB6   . |66:3D 0003    CMP AX,300  
00411FBA   . |72 0D         JB SHORT Winline1.00411FC9  
00411FBC   .^|76 A3         JBE SHORT Winline1.00411F61  
00411FBE   . |66:3D 0103    CMP AX,301  
00411FC2   .^|74 BE         JE SHORT Winline1.00411F82  
00411FC4   .^|E9 B9FAFFFF   JMP Winline1.00411A82  
00411FC9   > |66:3D 0102    CMP AX,201  
00411FCD   .^|0F84 13FFFFFF JE Winline1.00411EE6  
00411FD3   .^|E9 AAFAFFFF   JMP Winline1.00411A82  
00411FD8   > |66:3D 0101    CMP AX,101  
00411FDC   . |72 15         JB SHORT Winline1.00411FF3  
00411FDE   .^|0F86 92FEFFFF JBE Winline1.00411E76  
00411FE4   . |66:3D 0201    CMP AX,102  
00411FE8   .^|0F84 AEFEFFFF JE Winline1.00411E9C  
00411FEE   .^|E9 8FFAFFFF   JMP Winline1.00411A82  
00411FF3   > |66:3D 0001    CMP AX,100  
00411FF7   .^|0F84 68FEFFFF JE Winline1.00411E65  
  
OD 已经自动注释了 00411FA3 正是处理受到 WM\_COMMAND 的地方，AX 中放的正是各个菜单的 ID ，类似 CMP AX,XXX 的代码正是判断消息来自于哪个菜单  
  
00411FA3   > /8B8424 840000>MOV EAX,DWORD PTR SS:\[ESP+84\]          ;  Case 111 of switch 004118B1  
00411FAA   . |66:3D 0002    CMP AX,200  
00411FAE   . |72 28         JB SHORT Winline1.00411FD8  
  
改为：  
  
00411FA3   > /E9 B9130000   JMP Winline1.00413361                  ;  Case 111 of switch 004118B1  
00411FA8     |90            NOP                                    ;  上面的修改，为了跳向我们自己的代码  
00411FA9     |90            NOP  
00411FAA   > |66:3D 0002    CMP AX,200  
00411FAE   . |72 28         JB SHORT Winline1.00411FD8  
  
PeiD 告诉我们，.text:00413361 向后是全 0 区，但是空间不大，开始没有准备加老板键功能，所以一些代码放这儿了：  
  
00413361   > \\8B8424 840000>MOV EAX,DWORD PTR SS:\[ESP+84\]          ;  原来代码搬来到  
00413368   .  66:3D 0C03    CMP AX,30C                             ;  判断是不是按下了 F6  
0041336C   .  75 0E         JNZ SHORT Winline1.0041337C  
0041336E   .  60            PUSHAD  
0041336F   .  BA 64000000   MOV EDX,64                             ;  准备加上 100 分  
00413374   .  8BC7          MOV EAX,EDI  
00413376   .  E8 98DFFFFF   CALL Winline1.00411313                 ;  加分，并且调整小人的高度  
0041337B   .  61            POPAD  
0041337C   >  66:3D 0D03    CMP AX,30D                             ;  是不是按下了 F7  
00413380   .  75 1C         JNZ SHORT Winline1.0041339E  
00413382   .  C605 B41D4100>MOV BYTE PTR DS:\[411DB4\],0             ;  将每次分发小球数改为 0  
00413389   .  60            PUSHAD  
0041338A   .  6A 30         PUSH 30                                ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
0041338C   .  68 508D4200   PUSH Winline1.00428D50                 ; |Title = "秘笈提示"  
00413391   .  68 108D4200   PUSH Winline1.00428D10                 ; |Text = "晕，你想作弊！现在开始吧！"  
00413396   .  6A 00         PUSH 0                                 ; |hOwner = NULL  
00413398   .  E8 22FEFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \\MessageBoxA  
0041339D   .  61            POPAD  
0041339E   >  66:3D 0E03    CMP AX,30E                             ;  是不是按下了 F8  
004133A2   .  75 1C         JNZ SHORT Winline1.004133C0  
004133A4   .  C605 B41D4100>MOV BYTE PTR DS:\[411DB4\],3             ;  将每次分发小球数改为 3  
004133AB   .  60            PUSHAD  
004133AC   .  6A 30         PUSH 30                                ; /Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
004133AE   .  68 508D4200   PUSH Winline1.00428D50                 ; |Title = "秘笈提示"  
004133B3   .  68 308D4200   PUSH Winline1.00428D30                 ; |Text = "恢复正常，你得靠自己的实力哦！"  
004133B8   .  6A 00         PUSH 0                                 ; |hOwner = NULL  
004133BA   .  E8 00FEFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \\MessageBoxA  
004133BF   .  61            POPAD  
004133C0   >  66:3D 0F03    CMP AX,30F                             ;  是不是按下了"删除记录文件"菜单  
004133C4   .  75 24         JNZ SHORT Winline1.004133EA  
004133C6   .  60            PUSHAD  
004133C7   .  6A 24         PUSH 24                                ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
004133C9   .  68 548D4200   PUSH Winline1.00428D54                 ; |Title = "提示"  
004133CE   .  68 708D4200   PUSH Winline1.00428D70                 ; |Text = "重启程序分数会被清 0，你确认么？"  
004133D3   .  6A 00         PUSH 0                                 ; |hOwner = NULL  
004133D5   .  E8 E5FDFFFF   CALL <JMP.&USER32.MessageBoxA>         ; \\MessageBoxA  
004133DA   .  83F8 07       CMP EAX,7  
004133DD   .  74 0A         JE SHORT Winline1.004133E9  
004133DF   .  68 608D4200   PUSH Winline1.00428D60                 ; /FileName = "winlines.res"  
004133E4   .  E8 84FEFFFF   CALL <JMP.&kernel32.DeleteFileA>       ; \\DeleteFileA  
004133E9   >  61            POPAD  
004133EA   >^ E9 BBEBFFFF   JMP Winline1.00411FAA  
  
PeiD 查看下，text 段到 413400 前结束，只剩下 17 字节就到段尾了。汗！  
  
另外不要忘记了，在.data 段为上述对话框添加文本，即在下面内存中写入以下数据：  
  
00428D10  D4 CE A3 AC C4 E3 CF EB D7 F7 B1 D7 A3 A1 CF D6  晕，你想作弊！现  
00428D20  D4 DA BF AA CA BC B0 C9 A3 A1 00 00 00 00 00 00  在开始吧！......  
00428D30  BB D6 B8 B4 D5 FD B3 A3 A3 AC C4 E3 B5 C3 BF BF  恢复正常，你得靠  
00428D40  D7 D4 BC BA B5 C4 CA B5 C1 A6 C5 B6 A3 A1 00 00  自己的实力哦！..  
00428D50  C3 D8 F3 C5 CC E1 CA BE 00 00 00 00 00 00 00 00  秘笈提示........  
00428D60  77 69 6E 6C 69 6E 65 73 2E 72 65 73 00 00 00 00  winlines.res....  
00428D70  D6 D8 C6 F4 B3 CC D0 F2 B7 D6 CA FD BB E1 B1 BB  重启程序分数会被  
00428D80  C7 E5 30 A3 AC C4 E3 C8 B7 C8 CF C3 B4 A3 BF 00  清 0，你确认么？.  
  
第七步：为程序增加老板键功能  
  
说是老板键，其实没有用键盘，而用鼠标右键，我想当用左键点小球时顺手右键隐藏窗口比按键盘快些，可惜程序中没有针对 WM\_RBOTTONDOWN 处理的功能，自己添加。  
  
(1)、实现鼠标右键隐藏窗口：  
  
004118E8   .  81FB 01020000 CMP EBX,201                           ; 判断是否是 WM\_LBOTTONDOWN  
004118EE   .  0F86 1E020000 JBE Winline1.00411B12  
004118F4   .  81FB 02020000 CMP EBX,202                           ; 判断是否是 WM\_LBOTTONUP  
004118FA   .  0F84 48020000 JE Winline1.00411B48  
  
改为：  
  
004118E8   .  81FB 01020000 CMP EBX,201  
004118EE   .  0F86 1E020000 JBE Winline1.00411B12  
004118F4   .  81FB 02020000 CMP EBX,202  
004118FA   .- E9 01E70500   JMP Winline1.00470000                  ;  跳去我们的代码 .text 空间不够了，放到自己添加的段中吧  
004118FF      90            NOP  
  
00470000  - 0F84 421BFAFF   JE Winline1.00411B48                   ; 原来的代码搬来到  
00470006    81FB 04020000   CMP EBX,204                            ; 是不是 WM\_RBOTTONDOWN  
0047000C    75 31           JNZ SHORT Winline1.0047003F  
0047000E    60              PUSHAD                                 ; 保护现场  
0047000F    A1 3C024300     MOV EAX,DWORD PTR DS:\[43023C\]          ; hInstance  
00470014    6A 00           PUSH 0                                 ; /ShowState = SW\_HIDE  
00470016    50              PUSH EAX                               ; |hWnd = 00110174 ('Color linez 1.21 rdsnow\[BCG\]\[...',class='LinezWindow')  
00470017    E8 FD31FAFF     CALL <JMP.&USER32.ShowWindow>          ; \\隐藏窗口  
0047001C    A1 908D4200     MOV EAX,DWORD PTR DS:\[428D90\]          ; \[428D90\]保存了 hHook  
00470021    85C0            TEST EAX,EAX                           ; \[428D90\]!=0，说明已经安装了钩子  
00470023    75 19           JNZ SHORT Winline1.0047003E  
00470025    6A 00           PUSH 0                                 ; /ThreadID = 0  
00470027    FF35 28004300   PUSH DWORD PTR DS:\[430028\]             ; |hModule = 00400000 (Winline1)  
0047002D    68 4C004700     PUSH Winline1.0047004C                 ; |Hookproc = Winline1.0047004C  
00470032    6A 00           PUSH 0                                 ; |HookType = WH\_JOURNALRECORD  
00470034    E8 2232FAFF     CALL <JMP.&user32.SetWindowsHookExA>   ; \\如果还没有钩子，就安装一个钩子  
00470039    A3 908D4200     MOV DWORD PTR DS:\[428D90\],EAX          ; 保存 hHook，以备以后卸载钩子用  
0047003E    61              POPAD  
0047003F  - E9 BC18FAFF     JMP Winline1.00411900  
  
  
(2)、用钩子的回调函数，实现用 A 键呼出窗口  
  
呼出来隐藏窗口，如果用另一个进程只要两个代码就可以实现：  
  
HWND hWinlinz = FindWindow (NULL,"Color linez 1.21 rdsnow\[BCG\]\[PYG\]\[D.4s\]");  
if(hWinlinz!=0)  ShowWindow (hWinlinz,SW\_SHOW);  
  
我这里为了学习用钩子实现，回调函数如下：  
  
0047004C    55              PUSH EBP                               ; 钩子回调函数  
0047004D    8BEC            MOV EBP,ESP  
0047004F    81EC 00010000   SUB ESP,100  
00470055    FF75 10         PUSH DWORD PTR SS:\[EBP+10\]             ; /lParam  
00470058    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]              ; |wParam  
0047005B    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]              ; |HookCode  
0047005E    FF35 908D4200   PUSH DWORD PTR DS:\[428D90\]             ; |hHook  
00470064    E8 FE31FAFF     CALL <JMP.&user32.CallNextHookEx>      ; \\CallNextHookEx  
00470069    60              PUSHAD  
0047006A    837D 08 00      CMP DWORD PTR SS:\[EBP+8\],0             ; if \_dwCode != HC\_ACTION  
0047006E    75 1E           JNZ SHORT Winline1.0047008E            ; return  
00470070    8B5D 10         MOV EBX,DWORD PTR SS:\[EBP+10\]          ; mov ebx,\_lParam  
00470073    813B 00010000   CMP DWORD PTR DS:\[EBX\],100             ; if message != WM\_KEYDOWN  
00470079    75 13           JNZ SHORT Winline1.0047008E            ; return  
0047007B    807B 04 41      CMP BYTE PTR DS:\[EBX+4\],41             ; 判断是不是按下了 a 键  
0047007F    75 0D           JNZ SHORT Winline1.0047008E            ; return  
00470081    6A 01           PUSH 1                                 ; /ShowState = SW\_SHOWNORMAL  
00470083    FF35 3C024300   PUSH DWORD PTR DS:\[43023C\]             ; |hWnd = 001B0256 ('Color linez 1.21 rdsnow\[BCG\]\[...',class='LinezWindow')  
00470089    E8 8B31FAFF     CALL <JMP.&USER32.ShowWindow>          ; \\显示隐藏的窗口  
0047008E    61              POPAD  
0047008F    33C0            XOR EAX,EAX  
00470091    C9              LEAVE  
00470092    C2 0C00         RETN 0C  
  
第八步：在 WM\_CLOSE 中添加卸载钩子的代码  
  
我这里就懒得找了：  
  
00410017  |.  50            PUSH EAX                               ; /ExitCode  
00410018  \\.  E8 2C320000   CALL <JMP.&KERNEL32.ExitProcess>       ; \\ExitProcess  
  
直接改为：  
  
00410017   .- E9 7C000600   JMP Winline1.00470098  
0041001C      00            DB 00  
  
00470098    60              PUSHAD  
00470099    A1 908D4200     MOV EAX,DWORD PTR DS:\[428D90\]          ; 判断有没有加载钩子  
0047009E    85C0            TEST EAX,EAX  
004700A0    74 06           JE SHORT Winline1.004700A8  
004700A2    50              PUSH EAX  
004700A3    E8 B931FAFF     CALL <JMP.&user32.UnhookWindowsHookEx> ; 如有则卸载  
004700A8    61              POPAD  
004700A9    6A 00           PUSH 0  
004700AB    E8 9931FAFF     CALL <JMP.&KERNEL32.ExitProcess>  
  
运行，程序已经 OK 了。但是还有问题，考虑到钩子没有能及时卸载，计划在显示了隐藏的窗口后，窗口处于活动状态时就卸载钩子。但是用 UnHookWindowsHook 卸载时均返回了 NULL，实际  
最后在 ExitProcess 前的 UnHookWindowsHook 返回的也是 NULL，证明钩子没有被正常卸载，错误代码 ERROR\_INVALID\_HOOK\_HANDLE (0000057C)，无效的钩子句柄。但是句柄明明是正确的啊，我知道 UnHookWindowsHook 不能放在钩子函数体内，也有人说钩子函数必须放在共享段，但是这里没有使用 DLL 啊，我把.hook 改为共享段也无效。只好厚着脸皮等系统来回收了。  
\--------------------------------------------------------------------------------  
【问题积累】  
  
把修改过程中的遇到的问题摆上来，想大家讨教下：  
  
疑问 1：关于菜单资源：  
  
不知道源程序何以能做到"帮助"和其他如"文件"、"选项"等菜单分开的，资源重新编译后这个菜单有靠拢一起了。附件中我用空格增加相邻两菜单的距离，但是源程序中并没有空格。  
  
疑问 2：关于钩子：  
  
程序中两处：UnHookWindowsHook 返回 NULL，证明钩子没有被正常卸载，错误代码 ERROR\_INVALID\_HOOK\_HANDLE (0000057C)，不甘心遇到这样的问题，所以把附件放上来，大家帮忙解决下。  
  
疑问 3：关于 OD：  
  
实际操作时发现 OD 并不分析自己添加段.hook 中的代码，注释都是手动添加的，另外在这个段也不能用全部复制到可执行文件，只能使用选择代码复制，.hook 是个代码段，大家也是这样么？  
  
虽然修改的还不完美，处理钩子卸载问题外，其他均已完成，上传到办公室电脑桌面上，果然没几天，个个都觉得玩小球没意思了，我又来上网了。^\_^！  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 04 月 24 日 18:24:48 

引用:    
疑问 1：关于菜单资源  
........ 

  
用 Restorator 2006 来修改资源，可以保证帮助菜单在右边。顺便修改一下里面分隔菜单的制表符。修改后的菜单资源如下：  
  
1100 MENU  
{  
 POPUP "文件(&F)"  
 {  
  MENUITEM "新游戏(&N)\\tF4", 256  
  MENUITEM "显示排行榜(&H)", 257  
  MENUITEM "删除记录文件(&D)", 783  
  MENUITEM SEPARATOR  
  MENUITEM "退出(&X)\\tAlt-X", 258  
 }  
 POPUP "选项(&O)"  
 {  
  MENUITEM "下一组(&N)\\tF3", 512, CHECKED  
  MENUITEM "显示统计(&S)\\tF5", 513  
 }  
 POPUP "查看老板键(&B)"  
 {  
  MENUITEM "鼠标右键\\t 隐藏窗口", 784, GRAYED  
  MENUITEM SEPARATOR  
  MENUITEM "键盘 A 键\\t 恢复窗口", 785, GRAYED  
 }  
 POPUP "帮助(&H)",0,RIGHTJUSTIFY  
 {  
  MENUITEM "游戏规则(&R)\\tF1", 768  
  MENUITEM SEPARATOR  
  MENUITEM "关于(&A) ...", 769  
 }  
}  
  
文章太长了，我翻一下眼就花了![](img/9e41176b4900ac914e77ca870db1e3dd.jpg)

 4dd466d0e1e0cc123afbe504da2462ca