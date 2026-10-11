
# SkinEngine 1.0 控件采用 CC 方式对 NAG 的 100％破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-458.htm`](https://www.pediy.com/kssd/pediy07/pediy7-458.htm)

SkinEngine 1.0 控件采用 CC 方式对 NAG 的 100％破解  

【破解作者】 aCaFeeL  
【我的主页】 [`cztool.ys168.com/`](http://cztool.ys168.com/)  
【使用工具】 `ODbyDKY` 1.10(感谢 Dyk158 打造的这把利器）  
【破解平台】 Win98SE  
【软件名称】 SkinEngine v1.0  
【下载地址】 [`www.ksdev.com/`](http://www.ksdev.com/)  
【软件简介】 这是一个（2005 年 11 月 15 日）新出的界面控件，支持 Visual Basic，Visual C++，Borland Delphi, C++ Builder，Visual Basic.NET， Visual C#.NET，Visual J++.NET，Bolrand Delphi.NET， C# Builder 等众多编译程序。未注册版本编译后的程序，在程序开始和结束时都会出现 nag 提示窗口。  
【破解声明】 是§看雪§领我进了破解的这道门，而来了§看雪§这么久，通过前辈们的许多帖子，感觉自己也不断的有所收获！因此，出于对破解的爱好，自己斗胆也发一帖，将一点心得和体会拿出来与大家分享！不足之处还请多包含！  
【破解内容】  

安装完 skinengine 控件后，通过分析它的说明文件和 `unins000.dat` 文件，知道了在它的"redist"目录以及在系统的"Windows\\System"或"Windows\\System32"目录下都安装了 `skinengine.dll` 文件，而这就是 SkinEngine 控件的核心部分。分析 `skinengine.dll` 文件，发现它是被 UPX 压缩了的，故将其解压。

用 `ODbyDYK` 打开用了 SkinEngine 控件编译后的 Exe 文件，在 CPU 窗口中，点击鼠标右键，选择‘搜索-》所有参考文本串’，在新弹出的文本字符串参考窗口中，却没有找到任何与 NAG 窗口有关的信息 :- ( 。

于是按（Alt+E）键打开执行模块窗口，怪了！！窗口中居然也没有显示 `skinengine.dll` 的信息，而作为控件的核心，这个 dll 是一定要被导入的呀！嘿嘿，看来控件作者对自己产品的加密还是用了心的，没将它写入被编译程序的导入表中！那运行一下试试？按（F9）键运行，这时出现了 NAG 窗口，不要慌着按 NAG 窗口的“Close”按钮，而是在 `ODbyDKY` 中按（F12）键暂停，这时发现 模块窗口 中 `skinengine.dll` 却出现了，鼠标双击这一行后，便来到了 `skinengine.dll` 的领空，再搜索“所有参考文本串”，还是没有任何有价值的信息可供参考，看来只有从 `skinengine.dll` 的入口点开始一步一步的找了，这就是破解所需要的耐心了！

而通过上面的分析，知道了控件是在运行中才调用 `skinengine.dll` 的，而要从 `skinengine.dll` 的入口点开始分析，就要在 `skinengine.dll` 的入口点周围找地方插入一个 Int3（即机器码 CC），以便程序一运行到 `skinengine.dll` 的入口点周围就被中断下来，这样才能对 `skinengine.dll` 进行详细的分析。

在 `skinengine.dll` 的领空下，按（Ctrl＋N）键打开“显示当前模块名称（标签）”的窗口，点击“类型”后，找到导出类型中的`<ModuleEntryPoint>`这一行，鼠标双击后来到 `skinengine.dll` 的入口点，看到如下代码：

。。。。。。

```
00DEA230  >  55             PUSH EBP                       //来到这里，skinengine.dll 的入口点
00DEA231    8BEC           MOV EBP,ESP
00DEA233    83C4 C4        ADD ESP,-3C
00DEA236    B8 789EDE00    MOV EAX,SKINENGI.00DE9E78     //发现周围无法修改任何一个命令为 Int3
00DEA23B    E8 B0C6F3FF    CALL SKINENGI.00D268F0        //因此决定进入该 Call 看看
00DEA240    E8 9BA0F3FF    CALL SKINENGI.00D242E0
00DEA245    8D40 00        LEA EAX,DWORD PTR DS:[EAX]
00DEA248    0000           ADD BYTE PTR DS:[EAX],AL
00DEA24A    0000           ADD BYTE PTR DS:[EAX],AL
00DEA24C    0000           ADD BYTE PTR DS:[EAX],AL
00DEA24E    0000           ADD BYTE PTR DS:[EAX],AL
00DEA250    0000           ADD BYTE PTR DS:[EAX],AL
00DEA252    0000           ADD BYTE PTR DS:[EAX],AL
00DEA254    0000           ADD BYTE PTR DS:[EAX],AL
00DEA256    0000           ADD BYTE PTR DS:[EAX],AL
。。。。。。
```

于是，来到了虚拟地址： 00D268F0 处，看到如下的代码：

。。。。。。

```
00D268E9    E8 F6F6FFFF      CALL SKINENGI.00D25FE4
00D268EE    C3              RETN                             //上一个子程序的结束
00D268EF    90              NOP                              //**呵呵**重要** 可将此处改为: Int3
00D268F0    BA A8B0DE00     MOV EDX,SKINENGI.00DEB0A8   //这便是 00DEA23B 处的 Call 所调用的开始处了
00D268F5    837D 0C 01      CMP DWORD PTR SS:[EBP+C],1
00D268F9    75 2A           JNZ SHORT SKINENGI.00D26925
00D268FB    50              PUSH EAX
00D268FC    52              PUSH EDX
00D268FD    C605 64E6DE00 0>MOV BYTE PTR DS:[DEE664],1
00D26904    8B4D 08         MOV ECX,DWORD PTR SS:[EBP+8]
00D26907    890D 68E6DE00   MOV DWORD PTR DS:[DEE668],ECX
00D2690D    894A 04         MOV DWORD PTR DS:[EDX+4],ECX
00D26910    C742 08 0000000>MOV DWORD PTR DS:[EDX+8],0
00D26917    C742 0C 0000000>MOV DWORD PTR DS:[EDX+C],0
00D2691E    E8 C1FFFFFF      CALL SKINENGI.00D268E4
00D26923    5A              POP EDX
00D26924    58              POP EAX
00D26925    FF35 6CE6DE00   PUSH DWORD PTR DS:[DEE66C]
00D2692B    B9 C0B0DE00     MOV ECX,SKINENGI.00DEB0C0
00D26930    E8 5BD7FFFF      CALL SKINENGI.00D24090
00D26935    C3              RETN                               //00DEA23B 处的 Call 的结束
。。。。。。
```

发现虚拟地址：00D268EF 处居然有 NOP 命令，呵呵，如果猜得不错，这是控件作者自己留下的！

因此将虚拟地址：00D268EF 处的 NOP  命令  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;改为 INT3 命令，相应的，

也要将虚拟地址：00DEA23B 处的 CALL SKINENGI.00D268F0 命令  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;改为 CALL SKINENGI.00D268EF 命令，以便能够调用到 00D268EF 处的 INT3；

然后点击鼠标右键，选择‘复制到可执行文件-》所有修改’，在新弹出的窗口中，再次点击鼠标右键，选择‘保存文件’，将修改后的 `skinengine.dll` 文件保存。

在 `ODbyDYK` 中按（Alt＋F2）键结束调试，然后将被保存的 `skinengine.dll` 文件拷贝到 Windows\\system 目录中覆盖掉原来的 `skinengine.dll` 文件，再在 `ODbyDYK` 中按（Ctrl＋F2）键重新装载被调试程序，按（F9）键运行，这时程序中断在了 `skinengine.dll` 领空中的 00D268F0 处，按（F8）键一步一步的运行，到如下代码：

。。。。。。

```
00D268EF    CC              INT3                              //修改后所插入的 CC
00D268F0    BA A8B0DE00     MOV EDX,SKINENGI.00DEB0A8   //重新装载程序后，便中断在了此处
00D268F5    837D 0C 01      CMP DWORD PTR SS:[EBP+C],1
00D268F9    75 2A           JNZ SHORT SKINENGI.00D26925
00D268FB    50              PUSH EAX
00D268FC    52              PUSH EDX
00D268FD    C605 64E6DE00 0>MOV BYTE PTR DS:[DEE664],1
00D26904    8B4D 08         MOV ECX,DWORD PTR SS:[EBP+8]
00D26907    890D 68E6DE00   MOV DWORD PTR DS:[DEE668],ECX
00D2690D    894A 04         MOV DWORD PTR DS:[EDX+4],ECX
00D26910    C742 08 0000000>MOV DWORD PTR DS:[EDX+8],0
00D26917    C742 0C 0000000>MOV DWORD PTR DS:[EDX+C],0
00D2691E    E8 C1FFFFFF      CALL SKINENGI.00D268E4
00D26923    5A              POP EDX
00D26924    58              POP EAX
00D26925    FF35 6CE6DE00   PUSH DWORD PTR DS:[DEE66C]
00D2692B    B9 C0B0DE00     MOV ECX,SKINENGI.00DEB0C0
00D26930    E8 5BD7FFFF      CALL SKINENGI.00D24090          //F8 键运行此 Call 之后弹出 NAG，因此，在之处需按 F7 键，进入此 Call 中
00D26935    C3              RETN
。。。。。。
```

进入 CALL SKINENGI.00D24090 后，来到如下的代码：

。。。。。。

```
00D24090    51              PUSH ECX
00D24091    56              PUSH ESI
00D24092    57              PUSH EDI
00D24093    BE 38E6DE00     MOV ESI,SKINENGI.00DEE638
00D24098    8D7D C4         LEA EDI,DWORD PTR SS:[EBP-3C]
00D2409B    B9 0B000000     MOV ECX,0B
00D240A0    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI>
00D240A2    8F05 58E6DE00   POP DWORD PTR DS:[DEE658]
00D240A8    8F05 54E6DE00   POP DWORD PTR DS:[DEE654]                 ; SKINENGI.00D20000
00D240AE    892D 4CE6DE00   MOV DWORD PTR DS:[DEE64C],EBP
00D240B4    891D 50E6DE00   MOV DWORD PTR DS:[DEE650],EBX
00D240BA    A3 40E6DE00     MOV DWORD PTR DS:[DEE640],EAX
00D240BF    8915 48E6DE00   MOV DWORD PTR DS:[DEE648],EDX
00D240C5    8D4D C4         LEA ECX,DWORD PTR SS:[EBP-3C]
00D240C8    890D 38E6DE00   MOV DWORD PTR DS:[DEE638],ECX
00D240CE    31c9            XOR ECX,ECX
00D240D0    837D 0C 00      CMP DWORD PTR SS:[EBP+C],0
00D240D4    75 02           JNZ SHORT SKINENGI.00D240D8
00D240D6    8B08            MOV ECX,DWORD PTR DS:[EAX]
00D240D8    890D 44E6DE00   MOV DWORD PTR DS:[DEE644],ECX
00D240DE    B8 6013D200     MOV EAX,<JMP.&KERNEL32.RaiseException>
00D240E3    A3 14E0DE00     MOV DWORD PTR DS:[DEE014],EAX
00D240E8    B8 6813D200     MOV EAX,<JMP.&KERNEL32.RtlUnwind>
00D240ED    A3 18E0DE00     MOV DWORD PTR DS:[DEE018],EAX
00D240F2    E8 91FeFFFF     CALL SKINENGI.00D23F88
00D240F7    8B45 0C         MOV EAX,DWORD PTR SS:[EBP+C]
00D240FA    40              INC EAX
00D240FB    A2 60E6DE00     MOV BYTE PTR DS:[DEE660],AL
00D24100    48              DEC EAX
00D24101    59              POP ECX
00D24102    8B11            MOV EDX,DWORD PTR DS:[ECX]
00D24104    8915 5CE6DE00   MOV DWORD PTR DS:[DEE65C],EDX
00D2410A    74 07           JE SHORT SKINENGI.00D24113
00D2410C    3C 03           CMP AL,3
00D2410E    7D 03           JGE SHORT SKINENGI.00D24113
00D24110    FF1481          CALL DWORD PTR DS:[ECX+EAX*4]
00D24113    51              PUSH ECX
00D24114    8B4C24 08       MOV ECX,DWORD PTR SS:[ESP+8]
00D24118    85C9            TEST ECX,ECX
```
