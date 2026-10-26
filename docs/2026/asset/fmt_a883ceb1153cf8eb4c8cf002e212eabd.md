
# 用Arm3.75加壳的cc 版+iat乱序主程序的脱壳 （1）

用Arm3.75加壳的ASProtect V1.31 build 06.14主程序 （1）

【破解作者】 wangli\_com  
【使用工具】 WinXP、Ollydbg、PEiD、LordPE、ImportREC 1.6F  
【官方主页】 <http://www.888cp.com/bbs/>  
【破解平台】 WinXP  
【软件名称】 asprotect(v1.31 build 6.14) armadillo cc 版  
【下载地址】 <http://bbs.pediy.com/showthread.php?s=&threadid=2453>  
【软件大小】 1.14 MB  
【加壳方式】 armadillo cc 版+iat乱序+抽取代码  
【破解声明】 我是一菜青虫，偶得一点心得，愿与大家分享  ! ! !前面这几行模仿小虾，这种格式看起来比较舒服! ! !

## 1. 前言

初学脱壳，看着大虾们的文章，照猫画虎的破了几个，无意间一天被armadillo的一个单进程壳给难住了，就是在dfcg论坛上的“赤月3（40515）脱壳笔记！|Armadillo 3.00a-3.61”，按照步骤一不不差，脱完一运行，初始化错误，代码00005，为什么？重复了n遍，未果。于是激起了我学习armadillo的尽头，我到处查资料，看所有关于armadillo的文章，于是了解到双进程，cc 版+iat乱序+抽取代码,一块一块的啃，一个多月时间，下了班就研究，好累啊！借鉴了很多大侠的文章，终于有了点收获，写出来与大家分享，也许方法比较笨拙，还望各位大侠指教。

## 2. 脱壳过程（分以下几步进行）

### （1）查找OEP

用OD中载入程序，下`he WaitForDebugEvent`，F9运行，出现一个出错对话框，shift+f9运行，中断如下：

```
77E93A07 k> 55 push ebp         <====中断在此处，清除断点
77E93A08 8BEC mov ebp,esp
77E93A0A 83EC 68 sub esp,68
77E93A0D 56 push esi
77E93A0E FF75 0C push dword ptr ss:[ebp+C]
```

查看堆栈窗口：

```
0012BC88    005140D5  /CALL 到 WaitForDebugEvent 来自 Aspr.005140CF
0012BC8C    0012CD60  |pDebugEvent = 0012CD60   <====注意这里
0012BC90    000003E8  \Timeout = 1000. ms
```

在`0012CD60`所在的行上点击右键选“转存中跟随”。

然后再下`he WriteProcessMemory`，F9运行

```
77E41A90 k>  55                          push ebp<====中断在此处
77E41A91     8BEC                        mov ebp,esp
77E41A93     51                          push ecx
77E41A94     51                          push ecx
77E41A95     8B45 0C                     mov eax,dword ptr ss:[ebp+C]
```

看数据转存窗口：

```
0012CD60  01 00 00 00 BC 08 00 00  ...?..
0012CD68  20 09 00 00 01 00 00 80   .....
0012CD70  00 00 00 00 00 00 00 00  ........
0012CD78  4C 87 4D 00 02 00 00 00  LM....
0012CD80  00 00 00 00 4C 87 4D 00  ....LM.
0012CD88  4C 87 4D 00 00 00 00 00  LM.....
```

上面的`004d874c`就是`OEP`。

### （2）dump文件

现在我们重新用OD中载入程序，下硬件断点，`he WaitForDebugEvent`，F9运行，中断如下：

```
77E93A07 k> 55 push ebp   <====中断在此处，删除断点
77E93A08 8BEC mov ebp,esp
77E93A0A 83EC 68 sub esp,68
77E93A0D 56 push esi
77E93A0E FF75 0C push dword ptr ss:[ebp+C]
```

Alt+F9返回，搜索常数`FFFFFFF8`，来到（`0051467c`处）：

```
00514630     83BD D0F5FFFF 00            cmp dword ptr ss:[ebp-A30],0<====参考文章里提到的关键代码
00514637     0F8C A8020000               jl Aspr.005148E5
0051463D     8B8D D0F5FFFF               mov ecx,dword ptr ss:[ebp-A30]
00514643     3B0D E0075400               cmp ecx,dword ptr ds:[5407E0]
00514649     0F8D 96020000               jge Aspr.005148E5<====跳转到0005148E5处，因此在05148E5处下断点
0051464F     8B95 44F6FFFF               mov edx,dword ptr ss:[ebp-9BC]
00514655     81E2 FF000000               and edx,0FF
0051465B     85D2                        test edx,edx
0051465D     0F84 AD000000               je Aspr.00514710
00514663     6A 00                       push 0
00514665     8BB5 D0F5FFFF               mov esi,dword ptr ss:[ebp-A30]
0051466B     C1E6 04                     shl esi,4
0051466E     8B85 D0F5FFFF               mov eax,dword ptr ss:[ebp-A30]
00514674     25 07000080                 and eax,80000007
00514679     79 05                       jns short Aspr.00514680
0051467B     48                          dec eax
0051467C     83C8 F8                     or eax,FFFFFFF8<=====搜索来到这里，向上看在0042438A处
0051467F     40                          inc eax
00514680     33C9                        xor ecx,ecx
00514682     8A88 2CE85300               mov cl,byte ptr ds:[eax+53E82C]
00514688     8B95 D0F5FFFF               mov edx,dword ptr ss:[ebp-A30]
0051468E     81E2 07000080               and edx,80000007
00514694     79 05                       jns short Aspr.0051469B
00514696     4A                          dec edx
00514697     83CA F8                     or edx,FFFFFFF8
0051469A     42                          inc edx
0051469B     33C0                        xor eax,eax
0051469D     8A82 2DE85300               mov al,byte ptr ds:[edx+53E82D]
005146A3     8B3C8D B0A25300             mov edi,dword ptr ds:[ecx*4+53A2B0]
005146AA     333C85 B0A25300             xor edi,dword ptr ds:[eax*4+53A2B0]
005146B1     8B8D D0F5FFFF               mov ecx,dword ptr ss:[ebp-A30]
005146B7     81E1 07000080               and ecx,80000007
005146BD     79 05                       jns short Aspr.005146C4
005146BF     49                          dec ecx
005146C0     83C9 F8                     or ecx,FFFFFFF8
005146C3     41                          inc ecx
005146C4     33D2                        xor edx,edx
005146C6     8A91 2EE85300               mov dl,byte ptr ds:[ecx+53E82E]
005146CC     333C95 B0A25300             xor edi,dword ptr ds:[edx*4+53A2B0]
005146D3     8B85 D0F5FFFF               mov eax,dword ptr ss:[ebp-A30]
005146D9     99                          cdq
005146DA     B9 1C000000                 mov ecx,1C
005146DF     F7F9                        idiv ecx
005146E1     8BCA                        mov ecx,edx
005146E3     D3EF                        shr edi,cl
005146E5     83E7 0F                     and edi,0F
005146E8     03F7                        add esi,edi
005146EA     8B15 C4075400               mov edx,dword ptr ds:[5407C4]
005146F0     8D04B2                      lea eax,dword ptr ds:[edx+esi*4]
005146F3     50                          push eax
005146F4     8B8D D0F5FFFF               mov ecx,dword ptr ss:[ebp-A30]
005146FA     51                          push ecx
005146FB     E8 1C210000                 call Aspr.0051681C
00514700     83C4 0C                     add esp,0C
00514703     25 FF000000                 and eax,0FF<====从这里开始修改
00514708     85C0                        test eax,eax
0051470A     0F84 D5010000               je Aspr.005148E5
```

在`00514630`处下硬件执行断点，F9运行到`00514630`处，得到`[ebp-A30]=12cd4c`，把`12cd4c`的“d7”改为0，再修改上述的地方为：

```
00514703     FF05 4CCD1200               inc dword ptr ds:[12CD4C]
00514709     C705 E4075400 01000000      mov dword ptr ds:[5407E4],1
00514713   ^ E9 18FFFFFF                 jmp Aspr.00514630
```

去掉所有断点，并在`0005148E5`处下断，F9运行，断住。好了，所有代码都强制解压完成。

运行`LordPE`，有2个关于`Aspr.exe`的进程，选择第2个，主程序可完全dump出来了。再选择区域脱壳选择`10000000`上面的保护属性为`xrw`，大小为`20000`的代码段，用`LordPE`把这个区段装进脱出来的主文件中。

我的为`Region033A0000-033C0000.dmp`，把虚拟地址改为`033A0000-00400000=02fa0000`，这样偷走的代码被补了回来。每个人机子上都不一样，已自己机子为准。

用`lordpe`在主程序中加一个`section`，`VOffset`设置为`02fa0000`，大小设置为`20000`。用`winhex`把dump出来的`033A0000`处代码放回主程序。这样可以保证其中的跳转地址不会改变。

### （3）找cc

在OD中重开程序`Aspr.exe`，停在入口。下`he GetThreadContext`，运行，第二次停住，按alt＋f9回到用户空间。

```
005154C9     FF15 A4A05300               call dword ptr ds:[<&KERNEL32.GetThreadContext>]   ; kernel32.GetThreadContext
005154CF     50                          push eax//停在这，这条指令到00515511都是垃圾指令
005154D0     F7D0                        not eax------>垃圾指令开始
005154D2     0FC8                        bswap eax
005154D4     58                          pop eax
005154D5     73 00                       jnb short Aspr.005154D7
005154D7     9C                          pushfd
005154D8     60                          pushad
005154D9     EB 2B                       jmp short Aspr.00515506
005154DB     D270 0E                     sal byte ptr ds:[eax+E],cl
005154DE     EB 1D                       jmp short Aspr.005154FD
005154E0     33C9                        xor ecx,ecx
005154E2     74 00                       je short Aspr.005154E4
005154E4     EB 10                       jmp short Aspr.005154F6
005154E6     F9                          stc
005154E7     83F1 03                     xor ecx,3
005154EA   ^ 74 90                       je short Aspr.0051547C
005154EC     EB 05                       jmp short Aspr.005154F3
005154EE     B9 EB1074EF                 mov ecx,EF7410EB
005154F3   ^ 73 FA                       jnb short Aspr.005154EF
005154F5     F2:                         prefix repne:
005154F6     1ADB                        sbb bl,bl
005154F8     33DB                        xor ebx,ebx
005154FA   ^ EB EA                       jmp short Aspr.005154E6
005154FC   - E9 EBE1EBDD                 jmp DE3D36EC
00515501     F2:                         prefix repne:
```
