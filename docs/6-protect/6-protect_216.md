

# 制作 Flexlm license 总结！

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-758.htm`](https://www.pediy.com/kssd/pediy07/pediy7-758.htm)

作者:  laoqian \[FCG\]  
时 间:2005-12-15   
软件名称: Zendenc （FLEXlm 7.1d）  
软件类别：Nolan Blender 提供的经典例子   
软件介绍： FLEXlm 7.1d 加密  
破解工具：ollydbg 1.10(FLY 修改) ，w32Dasm\_2002828\_pll621，UltraEdit10.0，Flexlm7.2 SDK,calcseed,lmkg7，lmcryptgui，flexlm9.2   
  
sdk 的源码等  
   
破解目的：制作无限制使用 license 文件  
  
通过这篇文章我们将向你介绍如何制作 FlexLm 的 license，并总结经验  
  
第一部分: 废话  
  
Zendenc 这个软件是 Nolan Blender 提供的经典例题，crackZ 或网上有下载和分析文章。玩了几个 Flexlm, 想总结一下， 就拿它来开刀方便！本  
  
文是关于 7.x 的，但有部分适用于 8.x 和 9.x,甚至 10.x。  
  
第二部分: 转入正题  
  
破解 FlexLm 最主要的是找到 4 样东西.  
  
1. 确定版本号  
2. 找 vendor  
3. 计算 ENCRYPTION\_SEED  
4. 找 FEATURE  
  
有了这几个,就可以使用 FlexLm SDK（如果你有）或工具做出注册机了.  
btw：FlexLm SDK 一般是通用兼容的的，比如 7.2 就可以制作 7.1 的，甚至有时 7.1 也可以制作 7.2 的，关键是参数设置。  
  
1. 确定 FlexLm 版本号 BEHAVIOR Version  
  
方法一：  
\* 用二进制编辑器,推荐使用 HexWorkshop（我用 UltraEdit10.0，呵呵）,它的查找功能很强,特别是 Find All Instances 更是我最最常用的.  
在查找对话框 Type 中选择类型 Text String,Value 中输入"@(#) FLEXlm v",查找,"@(#) FLEXlm v"后面的就是版本号.  
【以上是 tulipfan\[CCG\]大虾说的】  
方法二：  
用 Flexlm SDK 里的 lmtools（在\\flexlm\\v7.1\\i86\_n3 目录里），运行进入 Utilities，点击“Browse”查找你的 Flexlm 加密的程序主 exe 或 dll，  
  
然后“find version”，在下面就会显示你的 Flexlm 版本，用 v7.1 版甚至可以看到 8.3 版以后，是通用的。有个别程序可能此法不行！  
方法三：在反汇编以后，搜索“87654321”，在前面设断点  
在 Flexlm SDK 里 lm\_code.h 文件里你可以看到一下几句：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
>  \*      Vendor's private seeds, -- replace with 32-bit numbers that
>  \*                                 you make up.
>  \*/
> #define ENCRYPTION\_SEED1 0x87654321
> #define ENCRYPTION\_SEED2 0x12345678
> /\*
>  \*      FLEXlm vendor keys -- enter as received from Globetrotter.
>  \*      Changing these keys has NO impact on license files (unlike
>  \*      the ENCRYPTION\_SEEDs).
>  \*/
> /\*-
>  \*      Generate these keys with: lmvkey -v demo -d (+3 months) -p ALL -c DEMO
>  \*              (Use a date approx 3 months out)
>  \*/
> 
> * * *
> 
> 
> ```

  
这说明 demo 的 SEED1＝0x87654321，SEED2＝0x12345678  
对应 Zendenc 里是：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0043391D    E8 184B0400       call <jmp.&MSVCRT.memcpy>
> 00433922    83C4 0C           add esp,0C
> 00433925    8D8D 40FFFFFF     lea ecx,dword ptr ss:\[ebp-C0\]  ；我们可以在这里设断点！
> 0043392B    51                push ecx
> 0043392C    8B95 30FFFFFF     mov edx,dword ptr ss:\[ebp-D0\]  ；ebp-D0 地址是以后要用的！
> 00433932    81C2 4C010000     add edx,14C                    ；计算得到 VENDOR 地址
> 00433938    52                push edx                       ；VENDOR 入栈！(ASCII "zend")
> 00433939    8B85 30FFFFFF     mov eax,dword ptr ss:\[ebp-D0\]
> 0043393F    50                push eax
> 00433940    E8 95560000       call zendenc.00438FDA   ；这个 call 就是著名的所谓 7648B98E 标志 call！后面要进去的！！！
> 00433945    83C4 0C           add esp,0C
> 00433948    81BD 44FFFFFF 214>cmp dword ptr ss:\[ebp-BC\],87654321  ；此处比较是否是 demo 的 seed1，当然在此 seed1 已被加密！
> 00433952    74 0C             je short zendenc.00433960           ；此处比较是否是 demo 的 seed2，当然在此 seed2 已被加密！
> 00433954    81BD 48FFFFFF 785>cmp dword ptr ss:\[ebp-B8\],12345678
> 0043395E    75 5D             jnz short zendenc.004339BD
> 00433960    8B8D 30FFFFFF     mov ecx,dword ptr ss:\[ebp-D0\]
> 00433966    8379 14 00        cmp dword ptr ds:\[ecx+14\],0
> 0043396A    74 17             je short zendenc.00433983
> 
> * * *
> 
> 
> ```

  
我们在断点停下 d ebp-c0  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0012D404  00000004  ...
> 0012D408  E0AAA4A0  \_お?  此 seed1 已被加密
> 0012D40C  C0121579  y    此 seed2 已被加密！
> 0012D410  3F9F6A79  yj?    此 VENDOR\_KEY1 已被加密！
> 0012D414  25DC750E  u?    此 VENDOR\_KEY2 已被加密！
> 0012D418  B8B046C5  臚案
> 0012D41C  0B2EAC4E  N?
> ......向下拉看
> 0012D4A8  00000000  ....
> 0012D4AC  00010007  ..
> 0012D4B0  37300064  d.07   此处显示 7.1d
> 0012D4B4  0000312E  .1..   7.1
> 0012D4B8  00000000  ....
> 
> * * *
> 
> 
> ```

  
这之后，等我们第一次走过著名的所谓 7648B98E 标志 call，“00433940   call zendenc.00438FDA”以后，会发现被加密 seed 又改变了，其实  
  
就是简单的 xor，可是这个 xor 我们不得而知！  
我们得到 BEHAVIOR Version  V7.1d  
btw：87654321 此处，在 6.x 版时，可以得到真 seed，  
参考文章“Ansoft Serenade v8.5 - Tutorial”  
  
http://www.woodmann.com/crackz/Tutorials/Serenade.htm  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> I also remembered that the checks for the default seed codes (shipped with the SDK) was a good place to fish the correct 
> 
> seeds, a simple disassembly search for '87654321' finds this code :-
> 
> :00429C4C CMP D, \[EBP-34\], 87654321 <-- Check for encryption\_seed1.
> :00429C53 JZ 00429C5E <-- Jump to Error.
> :00429C55 CMP D, \[EBP-30\], 12345678 <-- Check for encryption\_seed2.
> :00429C5C JNZ 00429C85 <-- Good jump.
> :00429C5E MOV EDI, FFFFFFA5 <-- Error Code (-91).
> 
> Routines inside Ser85.exe detect bpx type breakpoints and patching of key files, the checking code starts at 0040EFD3, here 
> 
> you'll see the names of the files that are checked and the rather obvious 'PUSH 7' instructions which produce a cryptic error 
> 
> message box asking you to call Ansoft for assistance. The files verified should give us a good idea where to look for other 
> 
> parts of the protection, Ansoft's developers evidently tried (as is good policy) to identify possible points of attack. Using 
> 
> bpmb style breakpoints we can quickly recover (what we think are valid) encryption\_seed1 (0x7CB2B081) & encryption\_seed2 
> 
> (0x2DFE22B6).
> .....
> 
> * * *
> 
> 
> ```

  
  
最后用方法一、二可以确定 Zendenc 版本号是 7.1d,这一步的目的有两个,一是在 lmkg.exe 中生成 Vendor 信息时用,二是在使用 FlexLm SDK 制作  
  
FlexLm 的 license 时使用。  
  
2. 先找 FEATURE  
  
方法一：  
不用 IDA，太麻烦了，既然有 SDK 和前人的经验，我偷个懒。我偷懒的办法，还是用安装 Flexlm SDK 里的 lmtools！打开“Server status”项，  
  
点击“Perform status Equiry”,你就会看到服务器端的正版 license.dat 内容！前提是你要配置好，而且有正版程序和 license，废话！  
比如显示如下：  
101: SERVER main 001234567890     （我的服务器网卡号，我隐去正确的，避免麻烦啊！）  
 80: VENDOR adskflex port=8080  
150: INCREMENT 41100ACD\_2002\_0F adskflex 1.000 permanent 100 VENDOR\_STRING=（中间有很多参数，省略避免麻烦）SIGN=787878787878  
  
（我隐去正确的，避免麻烦，为什么是 78　？）   
     关联信息: 省略  
170: (overall file checksum)  
  
看到了吗！  
  
VENDOR adskflex  
INCREMENT 41100ACD\_2002\_0F，这个 INCREMENT 其实就是 FEATURE，后面那个 1.000 就是 Version  
permanent 是用户数  
VENDOR\_STRING=　　这个也参与计算验证的  
中间有很多参数也参与计算验证的，省略避免麻烦  
SIGN=787878787878　我就不说了  
  
如果你没有像我的条件就只好去看别的资料学习，很简单的！  
  
btw：如果你的选项得当，现在你甚至就可以在客户端控制 license 服务器，你可以关闭 license 服务——慎用！！！，我曾在几次无意的操作中  
  
关闭了 license 服务，结果单位的所有机器 cad 罢工，管理员还不知道怎么回事，只好重启服务器，那几天可忙坏了她！这是 flexlm 的一个 bug 吧  
  
！  
方法二：  
  
用 IDA＋相应版本的 sig＋sdk，找到\_lc\_checkout 函数，设断点，你会找到 FEATURE。我采用的是 7.2 得 sig，显示是\_lp\_checkout  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00478043    68 00400000       push 4000
> 00478048    51                push ecx
> 00478049    6A 00             push 0
> 0047804B    6A 01             push 1
> 0047804D    68 ECDB4900       push zendenc.0049DBEC       ;FEATURE 版本 ASCII "1.0"
> 00478052    56                push esi                    ;FEATURE id， ASCII "Zend\_Encoder"
> 00478053    52                push edx
> 00478054    E8 57DFFBFF       call zendenc.00435FB0       ; lp\_checkout
> 00478059    83C4 4C           add esp,4C
> 0047805C    85C0              test eax,eax                ; 0，jmp
> 0047805E    74 3F             je short zendenc.0047809F
> 00478060    8A8424 18090000   mov al,byte ptr ss:\[esp+918\]
> 00478067    84C0              test al,al
> 00478069    75 12             jnz short zendenc.0047807D
> 0047806B    8B4424 10         mov eax,dword ptr ss:\[esp+10\]
> 0047806F    68 DCDB4900       push zendenc.0049DBDC        ; ASCII "Checkout failed"一般搜索这个字符，上面的就是 checkout 了！
> 00478074    50                push eax
> 
> * * *
> 
> 
> ```

  
在 00478054 中断，你会看到堆栈窗口里出现了 FEATURE id 和版本，或者 d esi。  
  
  
  
3. 找 vendor 以及计算 seed 的关键数据  
  
方法一同前，不说了。  
  
方法二：  
下面引用自 tulipfan\[CCG\]：  
\*\*\*\*\*\*\*\*  
“要找 vendor\_id 就需要找到 l\_sg 这个函数了(至于为什么要找这个函数建议看一些更基础的文章),在这儿我主要是介绍一下如何定位 l\_sg 函数,  
  
这个是 FlexLm 的一个内部使用的函数. lc\_init,lc\_checkout 都会调用它下面是它们的调用关系,括号里面的数字是调用的次数。  
  
a. lc\_init -> l\_init  
b. l\_init -> l\_sg(1)  
  
a. lc\_checkout -> l\_checkout  
b. l\_checkout -> lm\_start\_real(2)  
c. lm\_start\_real -> l\_good\_lic\_key(3)  
d. l\_good\_lic\_key -> l\_sg(2)  
  
从上面可以看到它们的调用关系.   
  
我们可以通过 l\_sg 确定 vendor\_id 和 ENCRYPTION\_SEED,通过 lc\_checkout 可以确定 FEATURE  
  
IDA 通过 FlexLm 的 sig 文件可以确定上面的大部分函数,再通过他们的调用关系,很容易找到其他函数.  
  
制作 FlexLm 的批处理文件和工具包可去 CrackZ 去下载.  
  
最后定位到 l\_sg 函数”  
  
\*\*\*\*\*\*\*\*\*\*\*\*  
  
我们有 9.2 的 source，我们看到 lm\_ckout.c 中有下面一段,那几个数就是确定 l\_sg 函数上很有用的，针对  
0x7648b98e;                   v7.x \*/  
0x6f7330b8;                 /\*- v8.x，9.x\*/  
下面给出参考：  
V7  
glseed = 0x788F71D2  
seedval = 0x7648B98E  
  
V8  
glseed = 0x3CDE3EBF  
seedval = 0x6F73330B8  
  
V9  
glseed = 0x72346B53  
seedval = 0x6F7330B8  
  
v10  
  
glseed = 0x5332322F  
seedval = 0x6F7330B8  
  
关于这一段，zhanzixin 有一点小小的补充。  
0x7648b98e 这个数字，被称为 seedval 的，  
在版本 4、5、6 中是 0xa8f38730  
在版本 7.0-8.0c 中是 0x7648b98e  
在版本 8.0d 以后，一直没变，是 0x6f7330b8。  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> /\*-
>  \*  Also used by flexcrypt -- notify if API changes.
>  \*/
> void
> l\_sg(
>   LM\_HANDLE \*    job,
>   char \*      vendor\_id,
>   VENDORCODE \*  key) /\*- l\_sg means "signature vendor\_key5" \*/
> {
>   unsigned long  keys\[4\];
>   char      sig\[SIGSIZE\] = {'\\0'};
>   /\*- If you change this, you must change it also in utils/lmnewgen.c \*/
>   /\*- unsigned long x = 0xa8f38730;                   v3.1 \*/
>   /\*- unsigned long x = 0x7648b98e;                   v7.0 \*/
>   unsigned long  x = 0x6f7330b8;                   /\*- v8.x \*/
>   extern void    (\*L\_UNIQ\_KEY5\_FUNC)();
>   unsigned long  d0 = 0, d1 = 0;
>   int        i = SIGSIZE-1;
> 
>   if (( job->options->flags & LM\_OPTFLAG\_CUSTOM\_KEY5) && L\_UNIQ\_KEY5\_FUNC)
>   {
>     (\*L\_UNIQ\_KEY5\_FUNC)(job, vendor\_id, key);
>     return;
>   }
> 
> * * *
> 
> 
> ```

  
废话少说，我们直接反汇编 zendenc.exe，我们查找“7648B98E”，来到下面，好像就这么１，２处  
  
。前人的经验就是宝贵啊，我省了很多事！  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> .text:00438FDA ; =============== S U B R O U T I N E ?=====================================
> .text:00438FDA
> .text:00438FDA ; Attributes: bp-based frame
> .text:00438FDA
> .text:00438FDA sub\_438FDA      proc near               ; CODE XREF: sub\_432CC4+C7Cp
> .text:00438FDA                                         ; sub\_437621+B9p ...
> .text:00438FDA
> .text:00438FDA                 push    ebp               ====先在这个地方下断
> .text:00438FDB                 mov     ebp, esp
> .text:00438FDD                 sub     esp, 30h
> .text:00438FE0                 mov     \[ebp+var\_10\], 7648B98Eh    ====就是这个数在确定 l\_sg 函数上很有用
> .text:00438FE7                 mov     \[ebp+var\_14\], 3
> .text:00438FEE                 mov     eax, \[ebp+arg\_0\]
> .text:00438FF1                 mov     ecx, \[eax+6Ch\]
> .text:00438FF4                 mov     edx, \[ecx+1D4h\]
> .text:00438FFA                 and     edx, 8000h
> .text:00439000                 test    edx, edx
> .text:00439002                 jz      short loc\_439027    ====这个跳前面分析了，第一次肯定会跳的！第二次再来！
> .text:00439004                 cmp     dword\_49E5EC, 0
> .text:0043900B                 jz      short loc\_439027
> .text:0043900D                 mov     eax, \[ebp+arg\_8\]
> .text:00439010                 push    eax
> .text:00439011                 mov     ecx, \[ebp+arg\_4\]
> .text:00439014                 push    ecx
> .text:00439015                 mov     edx, \[ebp+arg\_0\]
> .text:00439018                 push    edx
> .text:00439019                 call    dword\_49E5EC          ====在这个地方下断，F8 之后就是我们要的！
> .text:0043901F                 add     esp, 0Ch
> .text:00439022                 jmp     loc\_43913A
> .text:00439027 ; ---------------------------------------------------------------------------
> .text:00439027 loc\_439027:                             ; CODE XREF: sub\_438FDA+28j
> .text:00439027                                         ; sub\_438FDA+31j
> .text:00439027                 push    4
> .text:00439029                 lea     eax, \[ebp+var\_28\]
> .text:0043902C                 push    eax
> .text:0043902D                 mov     ecx, \[ebp+arg\_8\]
> .text:00439030                 add     ecx, 0Ch
> .text:00439033                 push    ecx
> .text:00439034                 mov     edx, \[ebp+arg\_4\]
> .text:00439037                 push    edx
> .text:00439038                 call    sub\_451F26
> 
> * * *
> 
> 
> ```

  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
别急，先做一下准备工作：  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
前面我们在 00433938 处可以得到 VENDOR 为 "zend"，而 0043392C 处的 ebp-D0 是初始化 VENDORkey 的地方！  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0043391D    E8 184B0400       call <jmp.&MSVCRT.memcpy>
> 00433922    83C4 0C           add esp,0C
> 00433925    8D8D 40FFFFFF     lea ecx,dword ptr ss:\[ebp-C0\]  ；我们可以在这里设断点！
> 0043392B    51                push ecx
> 0043392C    8B95 30FFFFFF     mov edx,dword ptr ss:\[ebp-D0\]  ；ebp-D0 地址是以后要用的！
> 00433932    81C2 4C010000     add edx,14C                    ；计算得到 VENDOR 地址
> 00433938    52                push edx                       ；VENDOR 入栈！(ASCII "zend")
> 00433939    8B85 30FFFFFF     mov eax,dword ptr ss:\[ebp-D0\]
> 0043393F    50                push eax
> 00433940    E8 95560000       call zendenc.00438FDA   ；这个 call 就是著名的所谓 7648B98E 标志 call！后面要进去的！！！
> 00433945    83C4 0C           add esp,0C
> 00433948    81BD 44FFFFFF 214>cmp dword ptr ss:\[ebp-BC\],87654321  ；此处比较是否是 demo 的 seed1，当然在此 seed1 已被加密！
> 00433952    74 0C             je short zendenc.00433960           ；此处比较是否是 demo 的 seed2，当然在此 seed2 已被加密！
> 00433954    81BD 48FFFFFF 785>cmp dword ptr ss:\[ebp-B8\],12345678
> 0043395E    75 5D             jnz short zendenc.004339BD
> 00433960    8B8D 30FFFFFF     mov ecx,dword ptr ss:\[ebp-D0\]
> 00433966    8379 14 00        cmp dword ptr ds:\[ecx+14\],0
> 0043396A    74 17             je short zendenc.00433983
> 
> * * *
> 
> 
> ```

  
但是问题又来了，我们并没有 license.dat，这就要我们做一个假的，否则第二次不会来到 text:00438FDA，那就前功尽弃了！  
license.dat 格式我不说了看雪的书上以及很多资料说得很清楚了。  
我们根据我们上面得到的正版 license.dat 内容，做一个假的 license.dat，放在 zendenc 目录里下:  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
FEATURE Zend\_Encoder zend 1.0 permanent uncounted VENDOR\_STRING=www.FCGchina.com \\  
  HOSTID=ANY SIGN=787878787878  (怎么又是 78　!?)  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
因为断开网络，所以运行 zendenc.exe 会找不到 license，我们把它指向我们的假的 license.dat，当然是出错退出！经过仔细跟踪还会发现  
  
license 文件名必须是 Zend\_Encoder.dat！  
  
好，工作开始：  
下面我启动 ollydbg 1.10 调入 zendenc.exe，在入口停下，直接下 bp 00438FDA ,bp 00439019  
F9 执行，中断在 00438FDA。  
  
这时第一次来到 00439002  jz short loc\_439027 时它是要跳过去的，注意要跳就让他跳吧。  
若开始第一次时强制不跳，倒是能来到 00439019，结果跟出了错误的 seed1 和 seed2，而且多少次结果还都一样。  
再次 F9，我们又回来中断在 00438FDA，F8 单步执行，这次不跳了，直到过了 00439019，暂停！  
我们在命令栏下如下命令，可以看到：（可以纪录了多次的调试结果）  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 输入　d \[esp+4\]  在内存窗口看到，【长型——ASCII 转存】
> \*\*\*\*\*\*\*\*\*\*
> 003983AC  646E657A  zend
> 
> 输入　d \[esp+8\] 在内存窗口看到，【长型——ASCII 转存】
> \*\*\*\*1\*\*\*\*
> 
> 0012CF98  00000004  ...
> 0012CF9C  B68ACAC8  仁姸     data\[0\]
> 0012CFA0  96327B11  {2      data\[1\]
> 0012CFA4  534D01FB  ?MS
> 0012CFA8  2D607B98  �{\`-
> 0012CFAC  B069483D  =Hi
> 0012CFB0  67FCD8B6  敦黦
> 0012CFB4  00000000  ....
> \*\*\*\*\*\*\*\*1\*\*\*\*\*\*\*\*\*\*\*\*
> 
> 输入 d \[esp\] 【长型——ASCII 转存】
> \*\*\*\*\*\*2\*\*\*\*\*
> 00398260  00000066  f...
> 00398264  00F400E6  ??
> 00398268  B9DEDAA8  ㄚ薰   job+08
> 0039826C  EB6E4C33  3Ln    job+0c
> 00398270  BEF9AA16      job+10
> 
> \*\*\*\*\*\*\*\*2\*\*\*\*\*\*\*\*\*\*\*
> 
> * * *
> 
> 
> ```

  
这些数据就是我们计算 seed1 和 seed2 的关键！  
  
4. 计算 ENCRYPTION\_SEED  
  
使用工具 calcseed.exe,输入 data\[0\]，data\[1\]，job+08，job+0c，job+10，VENDORname 为 "zend"上述信息,计算  
  
ENCRYPTION\_SEED1 0xfa5410de  
ENCRYPTION\_SEED2 0xdaeca107  
  
多次结果是一样的，注意是第二次自然来到时！  
  
至此,分析已经基本完成,可以使用 SDK 写 license.dat  
  
第三部分: 制作 license.dat  
  
方法一：  
  
我们得到数据如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Feathure  Zend\_Encoder
> FeathureVersion  1.0
> ENCRYPTION\_SEED1 0xfa5410de
> ENCRYPTION\_SEED2 0xdaeca107
> VENDOR Zend
> BEHAVIOR Version  V7.1d
> 
> * * *
> 
> 
> ```

  
使用 lmkg.exe,选择 version 7.0,在 vendor\_id 输入"Zend"得到:  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> /\* Version 7 keys \*/
> #define VENDOR\_KEY1 0xdb5c129b
> #define VENDOR\_KEY2 0x7c9d919a
> #define VENDOR\_KEY3 0x1ee3b786
> #define VENDOR\_KEY4 0x01745090
> #define VENDOR\_KEY5 0x2c2dd7f7
> #define CRO\_KEY1 0xdf7c1093
> #define CRO\_KEY2 0x3abff31c
> 
> #define VENDOR\_NAME "Zend"
> 
> * * *
> 
> 
> ```

  
  
按照上面的信息,编辑\\flexlm\\v7.2\\machind\\lm\_code.h  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
> #ifndef LM\_CODE\_H
> #define LM\_CODE\_H
> #include "lm\_cro.h"
> 
> /\*
>  \*      Pick an LM\_STRENGTH.
>  \*
>  \*              If you're not using CRO public-key, then leave this as
>  \*              LM\_STRENGTH\_DEFAULT.
>  \*              If you're upgrading from pre-v7.1, and want no changes,
>  \*              set this to LM\_STRENGTH\_LICENSE\_KEY.
>  \*/
> 
> #define LM\_STRENGTH LM\_STRENGTH\_DEFAULT   /\*注意 12 位的 SIGN 要这个参数 \*/
> 
> /\*
>  \* LM\_STRENGTH Options are
>  \*      LM\_STRENGTH\_DEFAULT             Public key protection unused
>  \*                                      Use SIGN= attribute
>  \*                                      sign length = 12
>  \* Public key:
>  \*       LM\_STRENGTH\_113BIT, LOW        sign length = 58 chars
>  \*       LM\_STRENGTH\_163BIT, MEDIUM     sign length = 84 chars
>  \*       LM\_STRENGTH\_239BIT, HIGH       sign length = 120 chars
>  \*
>  \* Use pre-v7.1, non-CRO
>  \*       LM\_STRENGTH\_LICENSE\_KEY        Use pre-v7.1 license-keys.
>  \*                                      Doesn't use SIGN= attribute.
>  \*/
> 
> /\*
>  \*      Vendor's private seeds, -- replace with 32-bit numbers that
>  \*                                 you make up.
>  \*/
> 
> #define ENCRYPTION\_SEED1 0xfa5410de　　 /\*我们找到的 SEED1 \*/
> #define ENCRYPTION\_SEED2 0xdaeca107　　　/\*我们找到的 SEED2 \*/
> #define ENCRYPTION\_SEED3 0x22222222     /\*没用，不变 \*/
> #define ENCRYPTION\_SEED4 0x32323232　　　/\*没用，不变 \*/
> 
> /\*
>  \*      FLEXlm vendor keys -- enter as received from Globetrotter.
>  \*      Changing these keys has NO impact on license files (unlike
>  \*      the ENCRYPTION\_SEEDs).
>  \*/
> /\*-
>  \*      Generate these keys with: lmvkey -v demo -d (+3 months) -p ALL -c DEMO
>  \*              (Use a date approx 3 months out)
>  \*/
> #define VENDOR\_KEY1 0xdb5c12db  /\*我们算出的 \*/
> #define VENDOR\_KEY2 0x7c9d91ba
> #define VENDOR\_KEY3 0x1ee3b7c6
> #define VENDOR\_KEY4 0x017450b0
> #define VENDOR\_KEY5 0x0c2dd7f7  /\*我们算出的 \*/
> #define CRO\_KEY1 0xdf7c10d3        /\* Used to enable CRO -- turned off by default \*/
> #define CRO\_KEY2 0x3abff33c        /\* Be sure to reset LM\_STRENGTH above if
>                                CRO\_KEY is non-zero \*/
> 
> 
> /\*
>  \*      FLEXlm vendor name.  Leave as "demo" if evaluating FLEXlm.  Otherwise
>  \*                           set to your vendor daemon name.
>  \*/
> 
> #define VENDOR\_NAME "Zend"　　/\*我们找到的 VENDOR \*/
> 
> /\*
>  \*      Older customers with newer versions may want to set
>  \*      behavior defaults to previous version, though this is usually
>  \*      discouraged.  Behaviors can be changed individually using
>  \*      LM\_A\_xxx in the flexible API.  New customers should use the
>  \*      current default, as set below
>  \*
>  \*      Valid settings include:
>  \*              LM\_BEHAVIOR\_V2, \_V3, \_V4, \_V5, \_V5\_1, \_V6, \_V7, \_V7\_1
>  \*/
> 
> #define LM\_VER\_BEHAVIOR LM\_BEHAVIOR\_V7\_1
> #endif /\* LM\_CODE\_H \*/
> 
> * * *
> 
> 
> ```

  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
编辑好了，存盘，OK  
  
打开\\flexlm\\v7.2\\i86\_n3\\genlic.exe 开始我们的制作吧  
看附图：  
１．basic 窗口内填好 Feathure，选择 permanent 打勾，选择 run anywhere  
２．进入 advance 窗口，version 内填 FEATURE 的 Version1 就是.000，Vendor info 填上 Cracked by laoqian\[FCG\]  
３．其余不改了，回到 basic 窗口，点击 make license，然后 save 你的 license.dat 文件  
大功告成！  
你可以制作各种限制条件的 license.dat，随你便了，我做的是无限制！  
  
本  
  
方法二：  
用 lmcryptgui 可以制作成功以上的 license！  
我们得到数据如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> Feathure  Zend\_Encoder
> Feathure Version  1.0
> ENCRYPTION\_SEED1 0xfa5410de
> ENCRYPTION\_SEED2 0xdaeca107
> VENDOR Zend
> BEHAVIOR Version  V7.1d
> 
> * * *
> 
> 
> ```

  
1.手动找到 vendor,seed1，seed2，FEATURE 和版本 7.1，7.2....，8.x 等  
2.填写 vendor,seed1，seed2，和 BEHAVIOR Version 7 等，注意没有 0x  
3.执行,生成一个 exe 文件.  
4.制作一个假的 license.dat 文件,格式参考其他资料了,一定要正确的格式,只有 sign=是随便写一个!  
5.把这个假的 license.dat 文件往那个生成的 exe 文件上一拖放,及自动生成正确的 license.dat 文件,sign 已经是正确的了,前提是你前面得到的  
  
vendor,seed1，seed2，FEATURE 和版本等都是正确的!  
附 license.dat 的写法：  
  
FEATURE xxxxx VENDORxxx 1.000 permanent uncounted \\  
VENDOR\_STRING=WWW.FCGCHINA.COM HOSTID=ANY ISSUER="by \\  
laoqian\[FCG\] 献给自由的 FCG" ISSUED=11-NOV-2004 \\  
SIGN=123456789012  
  
其中“xxxxx ”FEATURE 这个需要你手动找，“VENDORxxx”版本 Feathure Version“1.000 ”也需要你手动找确定，不过 Feathure Version 可  
  
以随便呢，我试了几个都可以运行的。  
SIGN=123456789012 随便填写 12 位 16 进制的数。其余一样。  
  
license.dat 文件格式根据版本不一样，以上是 flexlm7.1 版本之后的格式。  
flexlm7.1 之前的格式，没有 SIGN=，而且位置在前面。  
  
又说明：VENDOR\_STRING，ISSUER，ISSUED 可以不要的 。  
  
btw：这个的 license 文件名必须是 Zend\_Encoder.dat，且必须放在 zendenc.exe 同目录里！zendenc.exe 程序在 DOS 窗口运行！  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> FEATURE Zend\_Encoder zend 1.0 permanent uncounted VENDOR\_STRING=blah \\
>   HOSTID=ANY SIGN=B6457F8A4618
> 
> * * *
> 
> 
> ```

  
第四部分: 原创发挥  
  
分析得知 flexlm9.22。sdk 的源码  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 我们在 flexlm9.22 sdk 的源码里看到 l\_privat.h
> \*-
>  \*  -------------------------------------------------------------
>  \*  license-key-length stuff
>  \*/
> #define LM\_OPTFLAG\_LKEY\_LONG        0x800 /\* default False \*/
> #define L\_SECLEN\_SHORT 0x66D8B337
> #define L\_SECLEN\_LONG 0x289BEB8A
> #define L\_SECLEN\_SET\_LONG job->options->flags |= LM\_OPTFLAG\_LKEY\_LONG; \\
>       job->options->sf = L\_SECLEN\_LONG;
> #define L\_SECLEN\_SET\_SHORT job->options->flags &= ~LM\_OPTFLAG\_LKEY\_LONG; \\
>       job->options->sf = L\_SECLEN\_SHORT;
> #define L\_SECLEN\_OK (((job->options->flags & LM\_OPTFLAG\_LKEY\_LONG) && \\
>       (job->options->sf == L\_SECLEN\_LONG)) || \\
>       (job->options->sf == L\_SECLEN\_SHORT))
> 
> * * *
> 
> 
> ```

  
  
0x66D8B337 是 license-key-length 的一个标志，估计是 12 位 sign！  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0043C545    25 FF000000                and eax,0FF
> 0043C54A    25 FF000000                and eax,0FF
> 0043C54F    A2 41E64900                mov byte ptr ds:\[49E641\],al
> 0043C554    C605 47E64900 00           mov byte ptr ds:\[49E647\],0
> 0043C55B    8A15 47E64900              mov dl,byte ptr ds:\[49E647\]
> 0043C561    8815 46E64900              mov byte ptr ds:\[49E646\],dl
> 0043C567    C785 74FEFFFF 08000000     mov dword ptr ss:\[ebp-18C\],8
> 0043C571    817D 18 37B3D866           cmp dword ptr ss:\[ebp+18\],66D8B337  ；就在这里断下！走到这看看！
> 0043C578    75 0F                      jnz short zendenc.0043C589
> 0043C57A    8B85 74FEFFFF              mov eax,dword ptr ss:\[ebp-18C\]
> 
> * * *
> 
> 
> ```

  
  
输入 d 49E641 【长型——ASCII 转存】可以看到如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0049E63D  B6000000  ...
> 0049E641  468A7F45  E奆
> 0049E645  EE000018  ..
> 
> * * *
> 
> 
> ```

  
  
这是看到我的 SIGN=B6457F8A4618，再看上面是什么 B6+“468A7F45”+18！其中“468A7F45”低位在前的原则，你发现原来明码在这里！  
  
怎么找到这里的 0043C571？  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0043C6D0    83BD 8CFEFFFF 00  cmp dword ptr ss:\[ebp-174\],0
> 0043C6D7    74 2A             je short zendenc.0043C703
> 0043C6D9    8B95 78FEFFFF     mov edx,dword ptr ss:\[ebp-188\]
> 0043C6DF    33C0              xor eax,eax
> 0043C6E1    8A82 40E64900     mov al,byte ptr ds:\[edx+49E640\]
> 0043C6E7    50                push eax
> 0043C6E8    8B8D 78FEFFFF     mov ecx,dword ptr ss:\[ebp-188\]
> 0043C6EE    51                push ecx
> 0043C6EF    8D95 30FEFFFF     lea edx,dword ptr ss:\[ebp-1D0\]
> 0043C6F5    52                push edx
> 0043C6F6    8B45 08           mov eax,dword ptr ss:\[ebp+8\]
> 0043C6F9    50                push eax
> 0043C6FA    FF95 8CFEFFFF     call dword ptr ss:\[ebp-174\]
> 0043C700    83C4 10           add esp,10
> 0043C703    8B8D 30FEFFFF     mov ecx,dword ptr ss:\[ebp-1D0\]
> 0043C709    81E1 FF000000     and ecx,0FF
> 0043C70F    8B95 78FEFFFF     mov edx,dword ptr ss:\[ebp-188\]
> 0043C715    33C0              xor eax,eax
> 0043C717    8A82 40E64900     mov al,byte ptr ds:\[edx+49E640\]   ；看到 49E640 地址了吗？我们去那里下内存断点吧
> 0043C71D    3BC8              cmp ecx,eax                       ； eax＝真 sign 的一位和 ecx＝假 sign 的一位比较
> 0043C71F    74 04             je short zendenc.0043C725     ；比较
> 0043C721    33C0              xor eax,eax                   ；不对就拜拜了！
> 0043C723    EB 26             jmp short zendenc.0043C74B
> 0043C725  ^ E9 6BFEFFFF       jmp zendenc.0043C595           ；再比较下一位！
> 0043C72A    8B4D 18           mov ecx,dword ptr ss:\[ebp+18\]
> 
> * * *
> 
> 
> ```

  
  
怎么找到这里的 0043C71D？ 因为我们要读假 license 文件，读出假 sign 要在内存存着，我们在那里下内存断点（硬件断点），一步一步跟到这  
  
里，呵呵，需要耐心！不过也不难找，第二次进入“7648B98E”的 call，之后慢慢跟就来到了，中间过程不再罗嗦了！  
  
  
现在，你只要查找 66D8B337，看到上面上面的类似代码，下断吧！此发适用于 8.x，9.x 可能也适用！因为 66D8B337 没有变！  
  
就是说我们可以找明码，至少在 0043C71D 处可以一位一位找到！前提，你做了一个正确的假 license 文件，而且路径正确，还要解决一下 anti（  
  
如果有的话）  
  
  
第五部分:一个问题：  
就是我们在跟踪某些 flexlm 加密的软件时，有时不会来到类似 00439019 call dword\_49E5EC 的地方，应该是 2 次来到这个地址所在的 call，一般  
  
第一次跳过 00439019 call dword\_49E5EC，第二次才真正进入这个 00439019，此时才能得到文中那些值；问题是很多第二次就不来了！直接出  
  
错退出，即使是假 licese 文件格式参数正确（除了 sign 以外），也不过来第二次，因此无法得到需要的值，强制跳来是不行的！遇到此问题，  
  
如何解决呢？一般此情况出现是，关键 call 断点在 dll 中！个人感觉是 license 文件名或路径不对，或者有 anti 和 crc 校验，再或者是 ecc 作怪！此问题需要  
  
有心之人来解决！  
  
第六部分: 感谢  
  
CrackZ  
Nolan Blender  
Oleh Yuschuk  
tulipfan\[CCG\]  
allenzhu\[CCG\]  
zhanzixin  
FCG 的同仁  
和所有看学论坛上的朋友们! 

 4dd466d0e1e0cc123afbe504da2462ca