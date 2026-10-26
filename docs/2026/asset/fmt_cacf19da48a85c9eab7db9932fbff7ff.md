
# [原创]全国******2012功能破解

●

## 题记

本来接到这个活，我是拒绝的，原因一是跟自己的工作实在是不相干，之二是编程基础差功能实现差。奈何实在是受不了人情，就答应下来把这个活干完，

加之本身工作忙，只能偷周末时间来完成，断断续续弄了几个月的时间来完成，也算是刷新了自己拖延症的记录。

文章之中的口水话太多，各位请轻拍。最近受打击比较大，还请大家饶恕！

●

## 测试环境&工具使用

操作系统：

Windows XP SP3 \_Cn 虚拟机

工具:

IDA 、dede、010editor、OD、Windbg、notepad++、PEID

●

## 查壳&脱壳

PEID查壳，发现是最简单的UPX壳，随便有点基础的就能够破解之。使用方法是堆栈ESP跳转法

![](https://bbs.kanxue.com/upload/attach/201507/162756_zsngb74nid2n0pv.png)

```
00701C60 >pushad

00701C61  mov esi,568881.00603000    ;执行到此步，寄存器窗口，esp指针数据窗口跟随，接着下dword硬件访问断点

00701C66  lea edi,dword ptr ds:[esi-0x202000]

00701C6C  push edi                                 ; 568881.005B7C80

00701C6D  mov ebp,esp

00701C6F  lea ebx,dword ptr ss:[esp-0x3E80]
```

断点之后F9运行，就会触发硬件断点，如下：

```
007027EC  lea eax,dword ptr ss:[esp-0x80]    ;断在此处

007027F0  push 0x0

007027F2  cmp esp,eax

007027F4  jnz short 568881.007027F0

007027F6  sub esp,-0x80

007027F9  jmp 568881.005E905C                   ；远跳即为OEP

007027FE  add byte ptr ds:[eax],al

00702800  sbb byte ptr ds:[eax],ch

00702802  jo short 568881.00702804

00702804  pop eax                                  ; kernel32.7C817067
```

跳转之后就可以使用OD自带的脱壳插件进行脱壳，脱壳之后发现使用的是delphi编写

```
Borland Delphi 6.0 - 7.0
```

完成程序脱壳，接下来就需要进行实际数据的获取

●

## 功能性限制破解

### 1. 会员试用过期破解

电脑第一次注册可以有30天的免费试用时间，过期之后，存在用户服务到期的提示：

```
---------------------------

警告

---------------------------

登录失败

用户服务到期.

---------------------------

确定   

---------------------------
```

用户到期猜测是在服务器端进行判断的，所以选择重新注册一个账户，点击注册时会有如下提示:

```
---------------------------

提示

---------------------------

此电脑已经注册过会员号，请与当地服务商联系!总部客服电话:(86)0755-83485277 83485279

---------------------------

确定   

---------------------------
```

已经注册过会员号，肯定是本地提交判断的，OD载入程序，跟踪下断点：

```
0045F894    FF93 10010000   call dword ptr ds:[ebx+0x110] ; 11.0058AB08 此函数中弹出警告框

0045F89A    5B              pop ebx                       ; 00CFBD00
```

F7跟入这个函数：

```
0058ABEB   je short 11.0058ABF2

0058ABED   sub eax,0x4

0058ABF0   mov eax,dword ptr ds:[eax]

0058ABF2   test eax,eax            ;判断是否注册过会员账号

0058ABF4   jle short 11.0058AC10   ;小于则跳转,必须使其跳转才能跳过验证，直接暴力改为 jne jle->jne

0058ABF6   lea edx,dword ptr ss:[ebp-0xC]

0058ABF9   mov eax,11.0058ACE4

0058ABFE   call 11.004993F8

0058AC03   mov eax,dword ptr ss:[ebp-0xC]

0058AC06   call 11.00496CE4

0058AC0B   jmp 11.0058ACB8

0058AC10   xor ecx,ecx

0058AC12   mov dl,0x1

0058AC14   mov eax,dword ptr ds:[0x58A02C]

0058AC19   call 11.004E2EA0
```

`0058ABF2`处代码修改完成之后，就能够点击注册，可以写入账号密码、昵称之类，点击注册，接着会有提示：

```
---------------------------

提示

---------------------------

对不起，系统检测到本电脑已有用户注册过！A47014B09DEC2C3C6FCCF840B5A89840

---------------------------

确定   

---------------------------
```

经过一番调试后发现 **A47014B09DEC2C3C6FCCF840B5A89840**是通过一个值计算出来的，具体计算过程未分析，约束条件是由一个值来决定：

```
005891F6  mov eax,dword ptr ss:[ebp-0xC4]

005891FC  lea edx,dword ptr ss:[ebp-0xC0]

00589202  call 11.0049CC98

00589207  mov ecx,dword ptr ss:[ebp-0xC0]

0058920D  lea eax,dword ptr ss:[ebp-0xBC]          ; CPU_ID

EAX 01277878 ASCII "00000000000000000001"

ECX 01277848

EDX 01277120

EBX 012A9CC0

ESP 0012EFA8

EBP 0012F108

ESI 00D24600

EDI 0012F204

EIP 005891FC 11.005891FC
```

`eax`的值为一个比较重要的计算因子，值不同，最后计算出来的`CUP_ID`也不同，程序依此与服务器上存储的值进行判断，是否重复注册！修改`eax`的值，为任意值如:`00000000000000000003`,即可完成注册：

```
---------------------------

提示

---------------------------

注册成功,请与当地服务商联系.

天津： 深圳：0755-83485277 83485279

---------------------------

确定   

---------------------------
```

### 2. 破解服务到期提示

新注册的一个账号会有30天的试用期，剩余几天的说话会有如下的提示：

![](https://bbs.kanxue.com/upload/attach/201507/162756_4ki021r9i3tnz5x.png)

0D中采用F12断点法很容易就定位到了弹框点：

```
0047C562  push ebp

0047C563  push 11.0047C5E4

0047C568  push dword ptr fs:[ecx]

0047C56B  mov dword ptr fs:[ecx],esp

0047C56E  push esi

0047C56F  mov eax,dword ptr ss:[ebp-0x8]           ; 11.00496D3C

0047C572  push eax                                 ; 11.00496D3C

0047C573  push edi

0047C574  push ebx

0047C575  call <jmp.&user32.MessageBoxA>    ;在此处弹出服务提示
```

堆栈回溯如下：

```
0012F8BC   001F019C  |hOwner = 001F019C ('全国物流信息网大件通2012(1128)',class='jHi1bbnVbbbbFRWcQxwaXj46ltV')

0012F8C0   00DBE778  |Text = "您的服务还有3天到期，请尽快续费！"

0012F8C4   00496D3C  |Title = "提示"

0012F8C8   00000040  \Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL

0012F8CC   0012F948  指向下一个 SEH 记录的指针

0012F8D0   0047C5E4  SE处理程序
```

找到对应的汇编代码，前后查找发现，无法绕过弹框提示，干脆暴力一点，直接暴力修改汇编代码，然后保存：

```
0047C56F  mov eax,dword ptr ss:[ebp-0x8]           ; 11.00496D3C

0047C572  xor eax,eax                              ; 11.00496D3C

0047C574  mov eax,0x1

0047C579  nop

0047C57A  mov dword ptr ss:[ebp-0xC],eax           ; 11.00496D3C

0047C57D  xor eax,eax
```

●

## 分析过程记录

### 1. 基本分析

首先考虑到了需求，目的是获取到货源信息，这个信息包括：货源信息、电话、货源信息的发布时间，

示例如下：

```
货源：

抚顺 天水 一台50装载机

电话：

189 3217 7126

时间：

10:55
```

都是能够通过页面来得到，在每条信息上双击能够查看详细信息，如图：

![](https://bbs.kanxue.com/upload/attach/201507/162756_3v5suk1ijtuhjaj.png)

### 2. 查找数据处理过程

分析过程中遇到了难点就是如何枚举获取到的数据，从程序的数据处理逻辑来说，所有数据都是从网络中获取并展示到页面中，

通过wireshark抓包，没有得到比较有用的数据，推断是数据是经过加密处理，后面分析也验证了推断。

使用Delphi的反编译工具dede对脱壳后的程序进行反编译操作，发现程序使用了web引擎来实现货源数据的获取和更新。

而web方面正是直接的弱项，实在是没有什么比较的方法，只能硬啃！大悲剧！！！

### 1. 查找货源信息的突破口

OD在数据搜索方面的确不如windbg好使，此后的分析过程全部使用的是windbg

既然是货源信息，就以一条信息作为一个突破口，进而得到全部信息。

以页面的显示的货源进行搜索，可以得到以下结果：

```
0:002> s -u 00000000 L7ffffff "家到衡阳220"

001b049c  5bb6 5230 8861 9633 0032 0032 0030 6316  .[0Ra.3.2.2.0..c

001b25b4  5bb6 5230 8861 9633 0032 0032 0030 6316  .[0Ra.3.2.2.0..c

001d52c4  5bb6 5230 8861 9633 0032 0032 0030 6316  .[0Ra.3.2.2.0..c
```

能够在内存中搜索到相应的数据，

下内存读写断点：

```
ba w1 001c6eac
```

继续运行，断点会断在：

```
770f4b7a 57              push    edi

770f4b7b 8bcb            mov     ecx,ebx

770f4b7d 8bd1            mov     edx,ecx

770f4b7f c1e902          shr     ecx,2

770f4b82 8bf8            mov     edi,eax

770f4b84 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]    ;断在此处，进行的是memcpy 操作

770f4b86 8bca            mov     ecx,edx

770f4b88 83e103          and     ecx,3

770f4b8b f3a4            rep movs byte ptr es:[edi],byte ptr [esi]

770f4b8d 5f              pop     edi

770f4b8e 6683240300      and     word ptr [ebx+eax],0

770f4b93 5b              pop     ebx

770f4b94 5e              pop     esi

770f4b95 5d              pop     eb
```

程序在进行

```
memcpy
```

操作，查找源数据如下：

```
0:000> du esi

0012e840  "衡阳220挖机马上装 绍兴到福州3吨挖机 昆明到郑州75挖机 黄"

0012e880  "山到福清2翻斗 上海到勐腊200挖机 包头到集宁50铲矒"
```

确认是在处理货源信息，就这样一点一点的向上追溯，最终能够跟踪到数据解密函数

```
DLS0:00488FB4 ; 数据包-货源的解密函数

DLS0:00488FB4

DLS0:00488FB4 sub_488FB4      proc near               ; CODE XREF: sub_489678+1CEp

DLS0:00488FB4

DLS0:00488FB4 var_18          = word ptr -18h

DLS0:00488FB4 var_16          = byte ptr -16h

DLS0:00488FB4 var_14          = dword ptr -14h

DLS0:00488FB4

DLS0:00488FB4                 push    ebx

DLS0:00488FB5                 push    esi

DLS0:00488FB6                 push    edi

DLS0:00488FB7                 push    ebp

DLS0:00488FB8                 add     esp, 0FFFFFFF8h

DLS0:00488FBB                 mov     ebp, edx

DLS0:00488FBD                 mov     eax, [ebp+0]

DLS0:00488FC0                 mov     [esp+18h+var_14], eax

DLS0:00488FC4                 mov     eax, [esp+18h+var_14]
```

解密完成的数据是以类似GB2312格式存放，直接使用windbg的`da`命令是看不到真实内容，保存到硬盘上之后才能看到，使用windbg命令：

```
.writemem c:\22.log 00ca29d8  L 100
```

之后通过MultiByteToWideCharAPI转换成unicode格式的数据，显示到程序的主页面中。

货源的信息找到了，接下来是电话和时间，这两个通过观察堆栈中数据指针，发现多数都会指向特定的数据，

其中就有感兴趣的数据。如下：
