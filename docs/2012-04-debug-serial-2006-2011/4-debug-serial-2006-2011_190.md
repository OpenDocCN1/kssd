
# 美萍菜单安全卫士 8.87

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-25.htm`](https://www.pediy.com/kssd/pediy08/pediy8-25.htm)

题目：注册“美萍安全卫士 ”

软件名称：美萍菜单安全卫士 8.87 标准版

文件大小：584 KB

使用平台：`Win9x/NT/2000/XP`

软件来源：电脑报 2002 合订本光盘

软件简介：

美萍电脑安全卫士是最实用的网吧，电脑屋，学校机房安全保护，计费管理软件，它利用许多先进的 `windows` 内核技术，全真虚拟 `win9x` 桌面，实现了硬盘文件保护远程控制，会员远程登陆，限时，定时运行计算机，应用软件选择运行,网站记录，黄色网站限制等多项功能。

工具：`SOFTICE`，`CASPER`，`PEID`，`OD`，`UltraEDIT`

引子：以前只是听说过这个软件，根本没有用过，今天尝试安装了一下，结果刚启动安装程序“咣当”一声，电脑关机没商量:(，据说是对 `SOFTICE` 和 `TRW` 等设防了，我每次 `SOFTICE` 都是自动启动，所以关机是必然的了。我先用 `PEID` 查看是否加壳，结果是被 `Aspack2.0` 加壳了，我尝试用 `AspackDIE` 脱壳，运行脱壳后的程序不成功！我用 `Prodump` 也不成功，最后只好用 `Casper` 脱壳，运行一下，成功。但是用 `W32DASM` 反汇编不成功，用 `OD` 反汇编成功。

首先干掉关机的函数，用 `OD` 打开软件，搜索函数名，查看哪个函数关机，发现 `ExitWindowEx`。再查看有几个地方调用它，看到 3 个地方 `45AC32`,`4A1B01`,`4A1F3D`，再看看是否上面能否跳过这个调用。下面挨个来解决吧。

```assembly
0045AC18  /$ 53             PUSH EBX
*略去十来行
0045AC31  |. 57             PUSH EDI
0045AC32  |. E8 89B7FAFF    CALL <JMP.&user32.ExitWindowsEx> //就是这个函数关机。
0045AC37  |. 85C0           TEST EAX,EAX
0045AC39  |. 75 02          JNZ SHORT SMENU1.0045AC3D
*略去十来行
0045AC4F  |. 5B             POP EBX
0045AC50  \. C3             RETN
```

（1）.先看第一个地址所在的函数如下，从 `45AC18` 开始到 `45AC50` 结束，右击第一行指令，然后选择“搜索选定指令的引用”命令，发现有 6 个地方对此函数调用，看来软件作者非要你死机不可啦！！这六个地方是 `49683B`，`496854`,`496893`,`49FC7F`,`49FC94`,`49FCC5`.看来陷阱重重。下面看这六处如何解决：

（1）`49683B`，`496854` 和 `496893` 所在的函数上下文

```assembly
004967DA  |. 8B83 E4010000  MOV EAX,DWORD PTR DS:[EBX+1E4]
004967E0  |. 80B8 1D010000  >CMP BYTE PTR DS:[EAX+11D],0
004967E7  |. 74 70          JE SHORT SMENU1.00496859  //此处 JE 改为 JMP 可跳过关机 CALL，即 75->eb    
（*）
004967E9  |. 8B15 58BE4A00  MOV EDX,DWORD PTR DS:[4ABE58]
004967EF  |. 8B12           MOV EDX,DWORD PTR DS:[EDX]
*略去 10 来行
0049681F  |. A1 E8BF4A00    MOV EAX,DWORD PTR DS:[4ABFE8]
00496824  |. 8338 01        CMP DWORD PTR DS:[EAX],1
00496827  |. 75 19          JNZ SHORT SMENU1.00496842 //如果（*）跳转 NOP 掉，此处改为跳到 496898，
即 EB 6F （**）
00496829  |. A1 30C04A00    MOV EAX,DWORD PTR DS:[4AC030]
0049682E  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
00496830  |. 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
00496836  |. BA 09000000    MOV EDX,9
0049683B  |. E8 D843FCFF    CALL SMENU1.0045AC18     //调用关机函数
00496840  |. EB 17          JMP SHORT SMENU1.00496859
00496842  |> A1 30C04A00    MOV EAX,DWORD PTR DS:[4AC030]
00496847  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
00496849  |. 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
0049684F  |. BA 05000000    MOV EDX,5
00496854  |. E8 BF43FCFF    CALL SMENU1.0045AC18      //调用关机函数
00496859  |> 8B83 E8010000  MOV EAX,DWORD PTR DS:[EBX+1E8]  
0049685F  |. 80B8 1D010000  >CMP BYTE PTR DS:[EAX+11D],0
00496866   . 74 30          JE SHORT SMENU2.00496898  //如果（*）改为 JMP，则这个也改为 JMP，通往
光明。否则从（**）直接跳到目的地。
00496868   . 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
0049686B  |. A1 ACB84A00    MOV EAX,DWORD PTR DS:[4AB8AC]
00496870  |. E8 73CEFDFF    CALL SMENU1.004736E8
00496875  |. A1 30C04A00    MOV EAX,DWORD PTR DS:[4AC030]
0049687A  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
0049687C  |. E8 0794F9FF    CALL SMENU1.0042FC88
00496881  |. A1 30C04A00    MOV EAX,DWORD PTR DS:[4AC030]
00496886  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
00496888  |. 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
0049688E  |. BA 06000000    MOV EDX,6
00496893  |. E8 8043FCFF    CALL SMENU1.0045AC18      //调用关机函数
00496898  |> 33C0           XOR EAX,EAX
0049689A  |. 5A             POP EDX
0049689B  |. 59             POP ECX
0049689C  |. 59             POP ECX
\========================================================
```

（2）`49FC7F`,`49FC94`,`49FCC5` 所在的上下文

```assembly
0049FC68   . 833D 50BA4A00  >CMP DWORD PTR DS:[4ABA50],1
0049FC6F   . 75 15          JNZ SHORT SMENU1.0049FC86
0049FC71   . 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0049FC74   . 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
0049FC7A   . BA 09000000    MOV EDX,9
0049FC7F   . E8 94AFFBFF    CALL SMENU1.0045AC18     //调用关机函数，NOP 掉
0049FC84   . EB 13          JMP SHORT SMENU1.0049FC99
0049FC86   > 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0049FC89   . 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
0049FC8F   . BA 05000000    MOV EDX,5
0049FC94   . E8 7FAFFBFF    CALL SMENU1.0045AC18     //调用关机函数，NOP 掉
0049FC99   > 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0049FC9C   . 8078 03 72     CMP BYTE PTR DS:[EAX+3],72
0049FCA0   . 75 28          JNZ SHORT SMENU1.0049FCCA
0049FCA2   . 833D 7CBA4A00 >CMP DWORD PTR DS:[4ABA7C],1
0049FCA9   . 75 0C          JNZ SHORT SMENU1.0049FCB7
0049FCAB   . A1 A4BD4A00    MOV EAX,DWORD PTR DS:[4ABDA4]
0049FCB0   . 8B00           MOV EAX,DWORD PTR DS:[EAX]
0049FCB2   . E8 D1FFF8FF    CALL SMENU1.0042FC88
0049FCB7   > 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0049FCBA   . 8B80 14020000  MOV EAX,DWORD PTR DS:[EAX+214]
0049FCC0   . BA 06000000    MOV EDX,6
0049FCC5   . E8 4EAFFBFF    CALL SMENU1.0045AC18     //调用关机函数，NOP 掉
0049FCCA   > 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0049FCCD   . 8078 03 6D     CMP BYTE PTR DS:[EAX+3],6D
0049FCD1   . 0F85 8B000000  JNZ SMENU1.0049FD62
\========================================================
```

（3）另外两处调用关机关机函数的地方 `4A1B01`,`4A1F3D`

```assembly
004A1AF4   > E8 7FFEFCFF    CALL SMENU1.00471978
004A1AF9   . 84C0           TEST AL,AL
004A1AFB     74 09          JE SHORT SMENU1.004A1B06     //JE 改为 JMP，即 74->EB
004A1AFD   . 6A 00          PUSH 0
004A1AFF   . 6A 05          PUSH 5
004A1B01   . E8 BA48F6FF    CALL <JMP.&user32.ExitWindowsEx>   //调用关机函数
004A1B06   > 8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
004A1B09   . BA E4104D00    MOV EDX,SMENU1.004D10E4
004A1B0E   . B9 00010000    MOV ECX,100
004A1B13   . E8 6421F6FF    CALL SMENU1.00403C7C
\==================

004A1F30   . E8 43FAFCFF    CALL SMENU2.00471978
004A1F35   . 84C0           TEST AL,AL
004A1F37     74 09          JE SHORT SMENU1.004A1F42   //JE 改为 JMP，即 74->EB
004A1F39   . 6A 00          PUSH 0
004A1F3B   . 6A 05          PUSH 5
004A1F3D   . E8 7E44F6FF    CALL SMENU2.004063C0          //调用关机函数
004A1F42   > B8 440E4D00    MOV EAX,SMENU2.004D0E44
```

还有一个地方就是 `4063C0`，看看如下：

```assembly
004063C0   $-FF25 74274D00  JMP DWORD PTR DS:[<&user32.ExitWindowsEx>]
```

这个 `JMP` 就是被上述各函数调用的跳转。现在已经没有函数调用它了。如果只把此处 `NOP` 是不可以的，道理很简单，因为程序已经检测到 `SOFTICE` 等工具的存在，然后调用这个函数，这个如果不存在了，那程序不会正常运行，被挂起来了。所以必须才根本上把调用这个函数的路线切断。

\========================================================

（4）下面解决注册问题

这个软件在输入注册码后不立即校验，在重新启动时校验，所以肯定写在注册表或者什么地方了。安装这个软件之前，用 `REGSNAP` 给系统留影，然后安装之后再次留影，比较两次结果，发现在注册表留下了踪迹：

`H.L.M\Software\Mpsoft\Smenu\Reg` 等 7 个键在 `REG` 下面可以看到输入的假码。程序每次重新启动后才读取注册表进行校验是否正确。所以我们下条件断点 `bpx regqueryvalueExa if *(esp->8)= ='RegN' do "d esp->14"`。

`F5` 退出 `SICE`，然后启动美萍，被断下，就停在这个系统函数这里，此刻不要用 `F10` 或者 `F12` 跟踪了，如果这样最后走到系统的消息循环那儿就走不出来了，我折腾了半天也没有回到主程序去，如果用 `TRW` 的 `pmodule` 也不成功，一下子就运行了，根本拦不住的。所以还是 `SICE` 好用，这里按一次 `F11` 键，然后用 `F10` 跟踪。以前我对这个键不明白，那时刚刚开始学习破解，现在终于明白其意义重大了。这个键的功能就是“返回到刚刚调用了某个子函数的主函数的后面一条指令位置。比如：

//下面是某个主函数体

```assembly
00400188      mov eax,1
00400192      call  00400288 //发出对某个函数的调用
00400198      test eax,eax
```

// 下面是某个子函数体

```assembly
00400288      push ebp       //在刚刚开始进入这个函数的位置，按 F11 键，立即执行这个函数并返回到主调
函数      0040028a      mov eax,ebx  //中发出调用的指令的后面一个指令处。比如返回到 00400198 处。
0040028c      call 400666
```

注意：在需要的时候，按一下该 `F11` 功能键就可以了，不要按多次。

美萍被断下后，按一下 `F11` 回到主程序，因为美萍调用了好几次 `regqueryvalueExa` 函数，所以注意观察在数据窗口内读出的注册表键值，按一次 `F5` 就可以发现在数据窗口显示了 `Reg` 的键值，就是你的假码的 16 进制数了。当然这个假码已经是解密了的，在注册表内是加密的。加密方法见下面。

```assembly
00477694  /$ 53             PUSH EBX
00477695  |. A1 38C04A00    MOV EAX,DWORD PTR DS:[4AC038]
0047769A  |. 8338 00        CMP DWORD PTR DS:[EAX],0
0047769D  |. 75 2A          JNZ SHORT SMENU2.004776C9
0047769F  |. A1 D4BC4A00    MOV EAX,DWORD PTR DS:[4ABCD4]
```



```
004776A4  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]

004776A6  |. E8 BDF4FFFF    CALL SMENU2.00476B68

004776AB  |. 8BD8           MOV EBX,EAX                          ;

SMENU2.<ModuleEntryPoint>

004776AD  |. B8 54774700    MOV EAX,SMENU2.00477754              ;  ASCII "RegNum"

004776B2  |. E8 DDF6FFFF    CALL SMENU2.00476D94        //调用 regqueryvalueExa 函数

004776B7  |. 3BD8           CMP EBX,EAX                       //真假码进行对比，EBX 为解
密后的真码，EAX 解密后的假码

004776B9  |. 75 0A          JNZ SHORT SMENU2.004776C5  //不相同则 Fail。直接修改为 JZ 实行爆破，可
以输入任意注册码了。

004776BB  |. B8 01000000    MOV EAX,1

004776C0  |. E9 83000000    JMP SMENU2.00477748

004776C5  |> 33C0           XOR EAX,EAX                          ;

SMENU2.<ModuleEntryPoint>

004776C7  |. 5B             POP EBX                              ;  KERNEL32.BFF8B86C

004776C8  |. C3             RETN
```

========================================================

（5）跟踪注册码计算方法。只有在成功之前可以进行跟踪，如果注册成功了，那还跟踪干吗？？:) 输入任意码，比如 `78787878`，唤出 `SICE` 下断点 `bpx hmemcpy`，`F5` 退出 `SICE`，点击“注册”按钮，被拦住。`7` 次 `F12` 来到如下代码处，开始分析：

```
00477F37   E8 809EFAFF      CALL SMENU3.00421DBC

00477F3C   837D FC 00       CMP DWORD PTR SS:[EBP-4],0  //在这里停住

00477F40   74 65            JE SHORT SMENU3.00477FA7

00477F42   8D55 FC          LEA EDX,DWORD PTR SS:[EBP-4]

00477F45   8B83 F4010000    MOV EAX,DWORD PTR DS:[EBX+1F4]

00477F4B   E8 6C9EFAFF      CALL SMENU3.00421DBC

00477F50   8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]

00477F53   E8 A4F9F8FF      CALL SMENU3.004078FC         //这个 CALL 关键，F8 跟入

00477F58   8BD0             MOV EDX,EAX                          ;

SMENU3.<ModuleEntryPoint>

00477F5A   B8 D07F4700      MOV EAX,SMENU3.00477FD0              ;  ASCII "RegNum"

00477F5F   E8 F4EEFFFF      CALL SMENU3.00476E58         //这个 CALL 也关键，F8 跟入
```

先看第一个 `477F53` 处的 `CALL`：

*略去多行*

```
00407916   64:8920          MOV DWORD PTR FS:[EAX],ESP

00407919   8D55 FC          LEA EDX,DWORD PTR SS:[EBP-4]

0040791C   8BC3             MOV EAX,EBX

0040791E   E8 E5B2FFFF      CALL SMENU3.00402C08   //这个 CALL 关键，F8 跟入

00407923   8BF0             MOV ESI,EAX                          ;

SMENU3.<ModuleEntryPoint>

00407925   837D FC 00       CMP DWORD PTR SS:[EBP-4],0

00407929   74 23            JE SHORT SMENU3.0040794E
```

下面是跟入的代码：

```
00402C08   53               PUSH EBX

*略去多行*

00402C16   BF CCCCCC0C      MOV EDI,0CCCCCCC   //EDI 赋值一个常量

00402C1B   8A1E             /MOV BL,BYTE PTR DS:[ESI]  // 这个小循环判断每位注册码是否空格.

00402C1D   46               |INC ESI

00402C1E   80FB 20          |CMP BL,20

00402C21  ⁷⁴ F8            \JE SHORT SMENU3.00402C1B

00402C23   B5 00            MOV CH,0

00402C25   80FB 2D          CMP BL,2D          //判断第一位是否是短杠"-"，是则错。

00402C28   74 45            JE SHORT SMENU3.00402C6F

00402C2A   80FB 2B          CMP BL,2B          //判断第一位是否是加号"+"，是则错。

00402C2D   74 42            JE SHORT SMENU3.00402C71

00402C2F   80FB 24          CMP BL,24          //判断第一位是否是"$"，是则错。

00402C32   74 42            JE SHORT SMENU3.00402C76

00402C34   84DB             TEST BL,BL

00402C36   74 32            JE SHORT SMENU3.00402C6A

00402C38   80EB 30          /SUB BL,30         //下面的循环把输入注册码的每位变成 16 进制数，累加起
来。

00402C3B   80FB 09          |CMP BL,9

00402C3E   77 2A            |JA SHORT SMENU3.00402C6A

00402C40   39F8             |CMP EAX,EDI       //不超过那个 EDI 内的常数则 OK。

00402C42   77 26            |JA SHORT SMENU3.00402C6A

00402C44   8D0480           |LEA EAX,DWORD PTR DS:[EAX+EAX*4]  //EAX=5*EAX

00402C47   01C0             |ADD EAX,EAX       //EAX=2*EAX

00402C49   01D8             |ADD EAX,EBX       // //EAX=EAX+EBX(下一位)

00402C4B   8A1E             |MOV BL,BYTE PTR DS:[ESI]  //取下位注册码到 BL。

00402C4D   46               |INC ESI

00402C4E   84DB             |TEST BL,BL

00402C50  ⁷⁵ E6            \JNZ SHORT SMENU3.00402C38  //继续循环累加。

00402C52   FECD             DEC CH

00402C54   74 10            JE SHORT SMENU3.00402C66

00402C56   85C0             TEST EAX,EAX                       ;

SMENU3.<ModuleEntryPoint>

00402C58   7C 10            JL SHORT SMENU3.00402C6A
```

========================================================

再看第二个 `477F5F` 处的 `CALL`：

```
00476E58   55               PUSH EBP

00476E59   8BEC             MOV EBP,ESP

*略去多行*

00476E96   BA F06E4700      MOV EDX,SMENU3.00476EF0              ;  ASCII
"SOFTWARE\\Mpsoft\\Smenu\\Reg"

00476E9B   8BC7             MOV EAX,EDI

00476E9D   E8 22C0FBFF      CALL SMENU3.00432EC4   //往注册表写入键。

00476EA2   8BCE             MOV ECX,ESI

00476EA4   81F1 45160100    XOR ECX,11645          //注册码与 11645h 异或运算，就是它的加密算法。写
入注册表的值就是异或后的值。比如 78787878 变成十六进制后是 4B23526h，4B23526h xor 11645h=4B32363h。

00476EAA   8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]

*略去多行*
```

========================================================

最后，得到我们的注册码，在 `004776B7 CMP EBX,EAX` 那里看到的 `EBX` 内的值为解密后的真码 `8C2Eh`，`8C2Eh` 变为 10 进制数为 `35886`，这就是你的注册码了。但在注册表内的值 `19A6Bh` 是与 `1164h` 进行异或后得到的。比如 `8C2Eh xor 11645h =19A6Bh`。

### 后记：

在学习看雪精华 3 的是时候偶尔看到这个破文，不过那里都写的比较精练，我按照其中一个下注册表断点失败，而且有的用“天意”破解的，还有一篇是静态分析出来的，我还是结合动态和静态分析自己的吧，自己实践了一把，感谢看雪精华的文章作者，我也把破解过程和心得奉献出来，希望菜鸟受益是最终目的。

### 体会：

软件破解的过程中，如果使用条件断点效率就比较高了，当然需要对函数的参数使用比较明确，函数调用后，参数在堆栈的什么位置上是关键。比如下条件断点 `bpx regqueryvalueExa if *(esp->8)= ='RegN' do "d esp->14"`。其中 `esp->8` 是键名地址`*(esp->8)`表示该地址指向的键名串。 `esp->14` 则是读取的键值所在的地址。这里给菜鸟说明一下这个语法：这个形式`(esp->8)`跟`(esp+8)`是一样的，就是 `esp` 内容加 `8` 得到一个新堆栈地址，比如当前 `esp=402000`，`esp+8=402008`。如果前面带了`*`号，则表示间接寻址的意思，要拿出 `402008` 处的 `4` 个字节作为地址使用，去存取目标对象。说白了，`esp->8` 是指针变量，里面保存了一个地址，如果是 `*(esp->8)`，则表示指针变量（值）所指向的内容。不知菜鸟能否明白我的 罗嗦？？：）希望阅读愉快，学习进步！

感谢看雪对我们的成长与进步所起的巨大作用！

QduWg

qduwg@163.com

2006 年 1 月 9 日完稿

4dd466d0e1e0cc123afbe504da2462ca
