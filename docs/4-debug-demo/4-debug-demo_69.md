

# BlowFish 2000 v2.3(The Works) 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50044.htm`](https://www.pediy.com/kssd/pediy05/pediy50044.htm)

The Works 全攻略－破文第二篇（初学者适用）   
工具：TRW2000   
目标：Software by Design-The Works(URL http://www.gregorybraun.com/ 或者 URL http://gt.onlinedown.net/down/THEWORKS.ZIP)   
    The Works 中的工具虽然很小，但却是我们经常需要用到的，很有用。如：MS Windows 错误信息我们就很熟悉。还有其它的，比如：工具栏工具，系统托盘快捷工具，小巧实用的记事本工具、文本编辑工具、十六进制文件编辑工具，图标工具、上网填表工具、密码保存工具（还可生成随机密码，相当于 Advanced Password Generater），日历工具、放大镜工具（比 M$的好用）、联系人卡片工具等等 20 几个。   
    由于 The Works 各个软件的 pj 方法大同小异，所以在此仅以 BlowFish 为例。   
    这是我的破文第二篇，多谢大家对我第一篇的关注，谢谢。首先声明我还只是个初学者，如果文中有误，请指出，多谢。   
    如需转载请保持原文的完整。   
    首先运行 TRW，再运行 BlowFish->Help->Register...，输入名字、组织名和注册码，huboo，huboo.Inc，58585858，然后按 Ctrl+N，激活 TRW，下断点 bpx hmemcpy，按 F5 回到程序，按 OK，被拦住了，然后一直按 F12 直到出现错误提示框，一共是按了 12 次（其中有一个程序按 11 次就出现了提示框，到底是什么我现在不记得了，自己试试吧。）然后二拦，按 11 次（即比刚才少按一次）后就出现了如下代码：   
00408A44  |. E8 B77B0000    CALL BLOWFISH.00410600   
00408A49  |. 83C4 38        ADD ESP,38   
00408A4C  |. 3D 92A71901    CMP EAX,119A792   
00408A51  |. 75 18          JNZ SHORT BLOWFISH.00408A6B   
00408A53  |. 8B1D 38A14100  MOV EBX,DWORD PTR DS:\[<&KERNEL32.lstrcpy>   
00408A59  |. 68 48DD4100    PUSH BLOWFISH.0041DD48   
00408A5E  |. 56            PUSH ESI   
00408A5F  |. FFD3          CALL EBX   
00408A61  |. 68 38DD4100    PUSH BLOWFISH.0041DD38                
00408A66  |. 57            PUSH EDI   
00408A67  |. FFD3          CALL EBX   
00408A69  |. EB 07          JMP SHORT BLOWFISH.00408A72   
00408A6B  |> 3D 3CCE5F0D    CMP EAX,D5FCE3C   
00408A70  |. 75 0C          JNZ SHORT BLOWFISH.00408A7E   
00408A72  |> 57            PUSH EDI   
00408A73  |. 56            PUSH ESI   
00408A74  |. E8 B7750000    CALL BLOWFISH.00410030   
00408A79  |. 83C4 08        ADD ESP,8   
00408A7C  |. 8BD8          MOV EBX,EAX   
00408A7E  |> 57            PUSH EDI   
00408A7F  |. 56            PUSH ESI   
00408A80  |. E8 AB750000    CALL BLOWFISH.00410030   
00408A85  |. 83C4 08        ADD ESP,8   
00408A88  |. 3BD8          CMP EBX,EAX              ——————>关键比较注册码处，看 TRW 窗口上方的寄存器窗口，记下 EAX=\*\*\*\*\*\*\*\*（把这个数用计算器工具转换成十进制即为你的注册码）   
00408A8A  |. 5F            POP EDI   
00408A8B    74 1D          JZ SHORT BLOWFISH.00408AAA —————>如果这里跳的话，就注册成功。所以你想暴破的话就改这里，但不幸的是即使你改了，下一次启动还是未注册版本，所以还是找他的注册码。   
00408A8D  |. 68 CFEA0000    PUSH EACF   
00408A92  |. 68 88130000    PUSH 1388   
00408A97  |. 55            PUSH EBP   
00408A98  |. E8 53E0FFFF    CALL BLOWFISH.00406AF0   
    好了，如果你已经记下了上面 EAX 的值的话，那么用 Windows 里的计算器工具转换他，使用科学型，选十六进制和双字，然后输入 EAX 寄存器中的值，然后选十进制，OK，复制你所看到的数字到 BlowFish 的注册码框，点 OK，现在你应该完成了注册了。   
    其它 Software by Design 的几款软件和这里大同小异，你只需注意观察 TRW 窗口，只要出现“CMP EBX,EAX”该命令，就记下寄存器 EAX 中的值，然后在计算器转换就行了。   
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝   
附：KeyMaker 的注册机   
选另类注册机   
程序名称：BlowFish.exe   
中断地址：408A88   
中断次数：1   
第一字节：3B   
指令长度：4   
注册码为寄存器方式：EAX，十进制   
然后生成即可。   
其它几款的注册机也照上面的即可，只需把程序名称和中断地址改一下即可。（我在看雪的论坛里的谈天说地里已放了我做得注册机，大家可以下载使用。［文件是一个 zip 类型，里面有一个可执行文件，是用 Winimp 制作的］）   
    再次感谢那些作出贡献的人。   
huboo 于 3/31/2002  4dd466d0e1e0cc123afbe504da2462ca