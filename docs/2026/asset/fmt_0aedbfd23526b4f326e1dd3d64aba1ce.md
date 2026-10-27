
# [原创]神算超人V3.0算法分析

【软件大小】：339 KB
【软件语言】：简体中文
【软件类别】：国产软件 / 共享版 / 理科工具
【应用平台】：Win9x/NT/2000/XP
【界面预览】：无
【加入时间】：2005-01-24 10:50:14
【下载地址】：天空软件站
【推荐等级】：★★★
【在线注册】：点击这里进入注册中心==>

【软件简介】：《神算超人Ⅲ》是第三代“神算超人”口算练习软件。适合小学到成人各不同年龄层次的人使用，练习形式生动活泼，内容有加法、减法、乘法、除法、乘方、开方、阶乘、解一元二次方程、排列组合。可自由选择练习，自设题量，做题时间限制、不同难度等级、错题加罚强化练习等功能。也可由电脑进程式控制提高练习效率，强化练习等。所有功能都可以自己定义。练习形式较为灵活，可以自选练习，也可以由电脑随机练习，练习过程中每一小题都采用倒计时，人机对话形式，轻松而又不乏刺激的环境里提高你的口算速度和心算能力，这将是炼就你一生的本领，预祝你成为新一代神算超人。
《神算超人Ⅲ》在前一版基础上对练习界面做了调整，增设了“暂停”功能，以便可以让你在练习中不影响其它事，也不必终止练习。取消了上一版本中途退出拦截设置，你可以随时终止练习，或重新选择练习。

【作者声明】：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!

【破解工具】：`Ollydbg1.10`

这个程序有点意思,调试一次后再调试会出错(`Run-time error'62'`,`Input past end of file`),必须重新安装后方可正常调试,好像是对安装文件夹下的文件大小进行比较什么的(`regedit`和`cus`文件),不大清楚,希望大侠指点!

这个程序中用了大量的浮点运算,由于浮点知识溃乏,以下分析错误在所难免,欢迎大家指正,也请哪位大侠有浮点知识的文章或链接给我一份!

程序为VB编写,定位VB程序最好是用`_vbastrcmp`(`或者其他VB函数,如果你对VB熟的话,我是VB白痴,只会一点汇编与TC`),这篇文章并未能全部把算法分析清楚,因为其中有一个函数`MSVBVM60.__vbaPowerR8`不是很明白,而且VB的参数传递基本在堆栈中,感觉与汇编中有所不同,希望大家多多指正!

如上所说,用`_vbastrcmp`定位到如下代码(我是第一次接触VB,把反汇编的程序从头粗略看到尾才定位到如下代码)

```
0058DFFC   push eax
0058DFFD   push ecx
0058DFFE   call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  从字符串特定位置上获取其值
0058E004   push eax
0058E005   call dword ptr ds:[<&MSVBVM60.#696>]          ;  MSVBVM60.rtcCharValueBstr
0058E00B   movsx edx,ax                                  ;  上面那个Call看函数名猜想是按位取字串的ASCII值
0058E00E   mov dword ptr ss:[ebp-11C],edx                ;  用户名第1位放到ss:[ebp-11c]中
0058E014   fild dword ptr ss:[ebp-11C]                   ;  浮点指令,装入整数到st0=115("s"的Hex值)
0058E01A   fstp qword ptr ss:[ebp-124]                   ;  浮点指令,再把st0弹出到ss:[ebp-124]
0058E020   mov eax,dword ptr ss:[ebp-120]
0058E026   mov ecx,dword ptr ss:[ebp-124]
0058E02C   push eax
0058E02D   push ecx
0058E02E   call dword ptr ds:[<&MSVBVM60.__vbaPowerR8>]  ;  这个函数是求平方?似曾相识,往下看
0058E034   fmul qword ptr ds:[4011C8]                    ;  此时st0=13225=115*115,印证上面的猜想,不知对不对,这个指令为st0=st0*ds:[4011c8]=13225*1010323
0058E03A   mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrR8>] ;  字符串转换成双精度浮点数函数
0058E040   sub esp,8
0058E043   fstsw ax                                      ;  只查到fst dest是保存st0到dest,而fstsw是什么呢?
0058E045   test al,0D                                    ;  这个判断有点不明白,0D是回车符的Hex
0058E047   jnz 神算超人.0058E6AC
0058E04D   fstp qword ptr ss:[esp]                       ;  st0存入ss:[esp],再执行一次出栈操作
0058E050   call ebx                                      ;  还记得上面吗,字串转换成双精度数;<&MSVBVM60.__vbaStrR8>
0058E052   mov edx,eax                                   ;  edx=eax="13361521675",呵呵,返回值在eax中
0058E054   lea ecx,dword ptr ss:[ebp-78]
0058E057   call esi
0058E059   push eax                                      ;  大家记住一点,VB程序是利用堆栈传递参数的
0058E05A   call dword ptr ds:[<&MSVBVM60.#581>]          ;  MSVBVM60.rtcR8ValFromBstr
0058E060   mov edi,dword ptr ds:[<&MSVBVM60.__vbaVarMove>;  MSVBVM60.__vbaVarMove
0058E066   lea edx,dword ptr ss:[ebp-DC]
0058E06C   fstp qword ptr ss:[ebp-D4]                    ;  不注释了,此时st0中的值仍为13361521675
0058E072   lea ecx,dword ptr ss:[ebp-70]
0058E075   mov dword ptr ss:[ebp-DC],5
0058E07F   call edi                                      ;  <&MSVBVM60.__vbaVarMove>
0058E081   lea edx,dword ptr ss:[ebp-78]
0058E084   lea eax,dword ptr ss:[ebp-74]
0058E087   push edx
0058E088   push eax
0058E089   push 2                                        ;  以下都是释放内存空间
0058E08B   call dword ptr ds:[<&MSVBVM60.__vbaFreeStrLis>;  MSVBVM60.__vbaFreeStrList
0058E091   add esp,0C
0058E094   lea ecx,dword ptr ss:[ebp-8C]
0058E09A   call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]  ;  MSVBVM60.__vbaFreeVar
0058E0A0   lea edx,dword ptr ss:[ebp-CC]
0058E0A6   push 1
0058E0A8   lea eax,dword ptr ss:[ebp-8C]
0058E0AE   lea ecx,dword ptr ss:[ebp-38]
0058E0B1   push edx
0058E0B2   push eax
0058E0B3   mov dword ptr ss:[ebp-C4],ecx
0058E0B9   mov dword ptr ss:[ebp-CC],4008
0058E0C3   call dword ptr ds:[<&MSVBVM60.#619>]          ;  MSVBVM60.rtcRightCharVar
0058E0C9   push 40000000                                 ;  上面那个函数从字符串右边取相应字符
0058E0CE   lea ecx,dword ptr ss:[ebp-8C]
0058E0D4   push 0
0058E0D6   lea edx,dword ptr ss:[ebp-74]
0058E0D9   push ecx
0058E0DA   push edx
0058E0DB   call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  MSVBVM60.__vbaStrVarVal
0058E0E1   push eax
0058E0E2   call dword ptr ds:[<&MSVBVM60.#696>]          ;  MSVBVM60.rtcCharValueBstr
0058E0E8   movsx eax,ax                                  ;  又一轮循环开始,下面不多注释了,取用户名第2个字符"h"
0058E0EB   mov dword ptr ss:[ebp-128],eax
0058E0F1   fild dword ptr ss:[ebp-128]
0058E0F7   fstp qword ptr ss:[ebp-130]
0058E0FD   mov ecx,dword ptr ss:[ebp-12C]
0058E103   mov edx,dword ptr ss:[ebp-130]
0058E109   push ecx
0058E10A   push edx
0058E10B   call dword ptr ds:[<&MSVBVM60.__vbaPowerR8>]  ;  MSVBVM60.__vbaPowerR8
0058E111   fmul qword ptr ds:[4011C8]                    ;  在这里ds:[4011c8]中的值还为1010323,莫非为固定值
0058E117   sub esp,8                                     ;  程序在ds:[4011c8]内存段中存放了4个固定值,后面会还将继续看到
0058E11A   fstsw ax
0058E11C   test al,0D
0058E11E   jnz 神算超人.0058E6AC
0058E124   fstp qword ptr ss:[esp]
0058E127   call ebx
0058E129   mov edx,eax                                   ;  edx=eax="13130157708"
0058E12B   lea ecx,dword ptr ss:[ebp-78]
0058E12E   call esi                                      ;  MSVBVM60.__vbaStrMove
0058E130   push eax
0058E131   call dword ptr ds:[<&MSVBVM60.#581>]          ;  MSVBVM60.rtcR8ValFromBstr
0058E137   fstp qword ptr ss:[ebp-D4]                    ;  上面那个Call将字符串转化成双精度数=13130157708
0058E13D   lea edx,dword ptr ss:[ebp-DC]
0058E143   lea ecx,dword ptr ss:[ebp-24]
0058E146   mov dword ptr ss:[ebp-DC],5
0058E150   call edi                                      ;  MSVBVM60.__vbaVarMove
0058E152   lea eax,dword ptr ss:[ebp-78]
0058E155   lea ecx,dword ptr ss:[ebp-74]
0058E158   push eax
0058E159   push ecx
0058E15A   push 2
0058E15C   call dword ptr ds:[<&MSVBVM60.__vbaFreeStrLis>;  MSVBVM60.__vbaFreeStrList
0058E162   add esp,0C
0058E165   lea ecx,dword ptr ss:[ebp-8C]
0058E16B   call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]  ;  MSVBVM60.__vbaFreeVar
0058E171   mov edx,dword ptr ss:[ebp-38]                 ;  取用户名到edx中
0058E174   push edx
0058E175   call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr
0058E17B   push eax                                      ;  上面函数不用说了吧,eax=8,用户名长度
0058E17C   call dword ptr ds:[<&MSVBVM60.__vbaStrI4>]    ;  MSVBVM60.__vbaStrI4
0058E182   mov edx,eax                                   ;  上面的Call将一个长整型数转换成字符串,eax中返回的为地址
0058E184   lea ecx,dword ptr ss:[ebp-74]
0058E187   call esi                                      ;  MSVBVM60._vbaStrMove
0058E189   push eax
0058E18A   call dword ptr ds:[<&MSVBVM60.#581>]          ;  MSVBVM60.rtcR8ValFromBstr
0058E190   fstp qword ptr ss:[ebp-C4]                    ;  此时st0中为8,用户名的长度,上面的Call按字面理解为双精度数到字符串
0058E196   lea edx,dword ptr ss:[ebp-CC]
0058E19C   lea ecx,dword ptr ss:[ebp-34]
0058E19F   mov dword ptr ss:[ebp-CC],5
0058E1A9   call edi                                      ;  MSVBVM60._vbaVarMove
0058E1AB   lea ecx,dword ptr ss:[ebp-74]
0058E1AE   call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr
0058E1B4   lea eax,dword ptr ss:[ebp-70]
0058E1B7   lea ecx,dword ptr ss:[ebp-24]
0058E1BA   push eax
0058E1BB   lea edx,dword ptr ss:[ebp-8C]
0058E1C1   push ecx
0058E1C2   push edx
0058E1C3   mov dword ptr ss:[ebp-C4],1
0058E1CD   mov dword ptr ss:[ebp-CC],2
0058E1D7   call dword ptr ds:[<&MSVBVM60.__vbaVarAdd>]   ;  MSVBVM60.__vbaVarAdd
0058E1DD   push eax                                      ;  上面的Call将13130157708+13361521675=26491679383
0058E1DE   lea eax,dword ptr ss:[ebp-34]
0058E1E1   lea ecx,dword ptr ss:[ebp-CC]
0058E1E7   push eax
0058E1E8   lea edx,dword ptr ss:[ebp-34]
```
