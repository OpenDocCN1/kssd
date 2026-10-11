
# 软件注册码查找与 Ollydbg 调试实践

## 第一部分：某国产软件分析

`\* Possible StringData Ref from Code Obj ->"感谢您支持国产软件，祝您好运！"`

|   
:00488DF7 68E08E4800 | push 00488EE0 | <---双击串式参考便跳到此行，我们向上找第一个跳转处就是关键跳转，关键跳转上面就是关键比较;   
:00488DFC 8B45FC | mov eax, dword ptr [ebp-04] |   
:00488DFF E81CD2FBFF | call 00446020 |   
:00488E04 50 | push eax |

...省略代码若干...

向上看，`00488DEA`处有一跳转,不相等便跳到`00488E90`处，跳必挂!还记的`00488E97`处的出错对话框吧! 罪魁祸首啊!

在向上一行，看`00488DE7`处:`cmp eax, dword ptr [ebp-08]`,此为关键比较。可用`? EAX`查看软件正确的注册码。

### 整理

打开该软件，在注册码处输入`19870219`,打开`TRW2000`,下断点`bpx 00488DE7`，点注册被拦。输入`? EAX`得到软件正确的注册码。

*   机器码:`533226313`
*   注册码:`25061473`

用注册机编写器`keymake`编写该软件的注册机:

1.  点其它-另类注册机(`F8`),软件名称输入`ePaper.exe`,注册码选寄存器方式`EAX`十进制。
2.  添加断点，中断地址:`00488DE7`,中断次数:`1`,第一字节:`3B`,指令长度:`3`。
3.  生成注册机后完工，万事`OK`!

嘿嘿，现在是不是觉的找软件的注册码越来越像小时候玩的躲猫猫了？ `![  img/2323557208ff00a82c471843ba84a675.jpg  ]`可惜偶小时候没有青梅竹马那种类型的伙伴...

## 第二部分：WinZIP 8.1 分析

好的，我们这次讲个有点儿名气的软件，`WinZIP8.1`，这个软件相信大家都用过吧，反正偶是喜欢用`RAR`，不过也多少用过几天这玩意儿...

如果你没听说过，那看介绍好了 `![  img/2323557208ff00a82c471843ba84a675.jpg  ]`

*   **【软件名称】**`WinZIP`
*   **【软件版本】**`8.1 Beta 2`
*   **【文件大小】**`1757KB`
*   **【适用平台】``Win9x/Me/NT/2000`
*   **【软件简介】**一个强大并且易用的压缩实用程序，支持`ZIP`、`CAB`、`TAR`、`GZIP`、`MIME`，以及更多格式的压缩文件。其特点是紧密地与`Windows`资源管理器拖放集成，不用离开资源管理器而进行压缩、解压缩。

不用我说了吧，出处仍旧是电脑报 2001 年合订本的配套光盘 `![  img/2323557208ff00a82c471843ba84a675.jpg  ]`

我之所以先择它，是因为觉得它的关键`CALL`没有前边那两个那样好找(其实也就那样了^\_^)极具代表性，而且通过它可以让你感受一下`Ollydbg`这个魅力比你家的荼几还大的调试器 `![  img/2323557208ff00a82c471843ba84a675.jpg  ]`

这里之所以提到`Ollydbg`，是觉的它真是一个非常非常棒的调试器...强烈建议你多玩几次...(MP3 好听吗？ ^\_^)

### 调试过程

我们来吧,首先当然还是要装上它(左闪术，右闪术)，然后用`Ollydbg`来载入，此时界面会被分成四个部分,左上方是软件反汇编后的代码，右上方是寄存器开会的地方，左下方是内存区，右下方显示的则是堆栈的信息。

我们来下断点，按`Alt+F4`，之后选`USER32`,然后再鼠标右键-->搜索-->当前模块中的名称，然后在那一大堆函数中找到`GetDlgItemTextA`，按`F2`来下断点，它会提示你错误，并说无法设置中断点，是不是很过瘾？(呜呜呜...大哥，我错了，再也不敢了...)

呵呵，这个我也不知道什么原因，明明是用了这个函数嘛，就是不让断，其实我对`Ollydbg`也不是太那个(关键是讨厌它的下断方式)看来还是用我们的万能断点吧，输入注册名`Suunb[CCG]`，输入注册码`19870219`，然后用`TRW2000`下断`bpx hmemcpy`，断到之后，`pmodule`返回领空后一次`F12`就会出错，看来所有的东东就在这里了...

我们用`TRW2000`再断一下，返回领空之后记着第一条指令的地址`0040bd5f`，呜呜呜...上条指令明明是调用`GetDlgItemTextA`，为什么在`Ollydbg`中不让下呢？

没关系，我们记下这个地址后仍旧用`Ollydbg`来加载程序，之后在反汇编窗口中找到`0040bd5f`处，然后按下`F2`来下断(会变为红色)，下断之后便按`F9`来运行程序，接着输入注册名`Suunb[CCG]`，注册码`19870219`后按确定，程序会被`Ollydbg`给断到:

```
0040BD5F  |. 57                 PUSH EDI
0040BD60  |. E8 F34A0500        CALL WINZIP32.00460858
0040BD65  |. 57                 PUSH EDI                              ; /Arg1
0040BD66  |. E8 164B0500        CALL WINZIP32.00460881                 ; \WINZIP32.00460881
0040BD6B  |. 59                 POP ECX
0040BD6C  |. BE 1CCA4C00        MOV ESI,WINZIP32.004CCA1C
0040BD71  |. 59                 POP ECX
0040BD72  |. 6A 0B              PUSH 0B                                 ; /Count = B (11.)
0040BD74  |. 56                 PUSH ESI                                ; |Buffer => WINZIP32.004CCA1C
0040BD75  |. 68 810C0000        PUSH 0C81                               ; |ControlID = C81 (3201.)
0040BD7A  |. 53                 PUSH EBX                                ; |hWnd
0040BD7B  |. FF15 F4C54A00      CALL DWORD PTR DS:[<&USER32.GetDlgItemTe>; \GetDlgItemTextA
0040BD81  |. 56                 PUSH ESI
0040BD82  |. E8 D14A0500        CALL WINZIP32.00460858
0040BD87  |. 56                 PUSH ESI
0040BD88  |. E8 F44A0500        CALL WINZIP32.00460881
0040BD8D  |. 803D F0C94C00 >CMP BYTE PTR DS:[4CC9F0],0
0040BD94  |. 59                 POP ECX
0040BD95  |. 59                 POP ECX
0040BD96  |. 74 5F              JE SHORT WINZIP32.0040BDF7
0040BD98  |. 803D 1CCA4C00 >CMP BYTE PTR DS:[4CCA1C],0
0040BD9F  |. 74 56              JE SHORT WINZIP32.0040BDF7
0040BDA1  |. E8 31F9FFFF        CALL WINZIP32.0040B6D7                 <--关键 CALL，等会儿进去玩玩
0040BDA6  |. 84C0               TEST AL,AL                             <--根据关键 CALL 中比较的结果来做相应的测试
0040BDA8  |. 74 4D              JE SHORT WINZIP32.0040BDF7             <--跳走就没戏!
0040BDAA  |. 57                 PUSH EDI
0040BDAB  |. 68 08DE4B00        PUSH WINZIP32.004BDE08                 ;  ASCII "Name"
0040BDB0  |. FF35 1CC74A00      PUSH DWORD PTR DS:[4AC71C]             ;  WINZIP32.004BDDEC
0040BDB6  |. E8 8AFA0400        CALL WINZIP32.0045B845
0040BDBB  |. 56                 PUSH ESI
0040BDBC  |. 68 C8EB4B00        PUSH WINZIP32.004BEBC8                 ;  ASCII "SN"
0040BDC1  |. FF35 1CC74A00      PUSH DWORD PTR DS:[4AC71C]             ;  WINZIP32.004BDDEC
0040BDC7  |. E8 79FA0400        CALL WINZIP32.0045B845
0040BDCC  |. FF35 18C74A00      PUSH DWORD PTR DS:[4AC718]             ; |Arg4 = 004BDDF4 ASCII "winzip32.ini"
0040BDD2  |. 6A 00              PUSH 0                                  ; |Arg3 = 00000000
0040BDD4  |. 6A 00              PUSH 0                                  ; |Arg2 = 00000000
0040BDD6  |. 68 14DE4B00        PUSH WINZIP32.004BDE14                 ; |Arg1 = 004BDE14 ASCII "rrs"
0040BDDB  |. E8 4CFA0400        CALL WINZIP32.0045B82C                 ; \WINZIP32.0045B82C
0040BDE0  |. A1 A8914C00        MOV EAX,DWORD PTR DS:[4C91A8]
0040BDE5  |. 83C4 28            ADD ESP,28
0040BDE8  |. 85C0               TEST EAX,EAX
0040BDEA  |. 74 07              JE SHORT WINZIP32.0040BDF3
0040BDEC  |. 50                 PUSH EAX                                ; /hObject => 000013F4 (font)
0040BDED  |. FF15 80C04A00      CALL DWORD PTR DS:<&GDI32.DeleteObject>; \DeleteObject
0040BDF3  |> 6A 01              PUSH 1
0040BDF5  |. EB 30              JMP SHORT WINZIP32.0040BE27
0040BDF7  |> E8 C3020000        CALL WINZIP32.0040C0BF
0040BDFC  |. 68 8E020000        PUSH 28E
0040BE01  |. E8 61470500        CALL WINZIP32.00460567
0040BE06  |. 50                 PUSH EAX                                ; |Arg3
0040BE07  |. 53                 PUSH EBX                                ; |Arg2
0040BE08  |. 6A 3D              PUSH 3D                                 ; |Arg1 = 0000003D
0040BE0A  |. E8 C8050400        CALL WINZIP32.0044C3D7                 ; \WINZIP32.0044C3D7
```

我们用`Ollydbg`断到之后，可以像在`TRW2000`中一样通过`F8`(这个调试器跟我一样，也不喜欢`F4`^\_^)来单步执行程序，我们按`32`下`F8`后程序就会出错，那我们在第二遍载入时按`F8`按到`20`多下时就仔细看看有没有可疑的地方，你一眼就可以看到`0040BDA1`处的这个关键`CALL`，我们只要追到这里时追进去就有可能看到软件正确的注册码 `![  那还等什么呢？我们就进去吧...  ]`

按`F7`跟进后你会看的眼花眼花缭乱，到处都是`PUSH`跟`POP`，到底哪个才是呢？现在知道我为什么让你用`Ollydbg`了吧(偶起初也是要用`TRW2000`的，但临时改变主意 ^\_^)用`Ollydbg`的一个最大好处就是可以真接看到寄存器中的值，特别是你通过`F8`来单步执行的时候，在反汇编代码的下边，会有一个小窗体，在那里可以显示相关指令中所使用的寄存器的值，爽！

我们按`76`下`F8`之后，在`0040B803`处就可以第一次看到正确的注册码了，呵呵，我这边儿是`71C20EDC`，然后你还会再陆续看到几次，爽？

另外我还发现一个有趣的事情，在`WinZIP8.1`中，一个注册名可以有两个注册码，呵呵，不知道是不是还有为特别用户准备的特别注册码以用来和普通的做区别 `![  img/2323557208ff00a82c471843ba84a675.jpg  ]`当程序通过比较，发现你输入的注册码不正确后竟然会再次算出另一个注册码来再比较一次，嘿嘿，我的第二个注册码是`25170288`

### 追入关键 CALL 里的代码

```
0040B6D7  /$ 55                 PUSH EBP
0040B6D8  |. 8BEC               MOV EBP,ESP
0040B6DA  |. 81EC 0C020000      SUB ESP,20C
0040B6E0  |. 8065 FF 00         AND BYTE PTR SS:[EBP-1],0
0040B6E4  |. 803D F0C94C00 >CMP BYTE PTR DS:[4CC9F0],0
0040B6EB  |. 53                 PUSH EBX
0040B6EC  |. 56                 PUSH ESI
0040B6ED  |. 57                 PUSH EDI
0040B6EE  |. 0F84 FB000000      JE WINZIP32.0040B7EF
0040B6F4  |. 8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]
0040B6F7  |. 50                 PUSH EAX
0040B6F8  |. 68 C0E84B00        PUSH WINZIP32.004BE8C0
0040B6FD  |. E8 DE61FFFF        CALL WINZIP32.004018E0
0040B702  |. 8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]
0040B705  |. 50                 PUSH EAX
0040B706  |. E8 F57C0800        CALL WINZIP32.00493400
0040B70B  |. 83C4 0C            ADD ESP,0C
0040B70E  |. 83F8 14            CMP EAX,14
0040B711  |. 72 11              JB SHORT WINZIP32.0040B724
0040B713  |. BF 20C74A00        MOV EDI,WINZIP32.004AC720             ;  ASCII "auth.c"
0040B718  |. 6A 21              PUSH 21
0040B71A  |. 57                 PUSH EDI
0040B71B  |. E8 86F60000        CALL WINZIP32.0041ADA6
0040B720  |. 59                 POP ECX
0040B721  |. 59                 POP ECX
0040B722  |. EB 05              JMP SHORT WINZIP32.0040B729
```
