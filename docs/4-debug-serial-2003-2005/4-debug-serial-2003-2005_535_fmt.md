
# 小护士 V1.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50519.htm`](https://www.pediy.com/kssd/pediy05/pediy50519.htm)

下载页面： [`www.skycn.com/soft/11505.html`](http://www.skycn.com/soft/11505.html)

软件大小： 772 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 开关定时

应用平台： `WinNT/2000/XP`

加入时间： 2003-03-28 10:09:04

下载次数： 227

推荐等级： \*\*\*

开发商： [`yhh365.3322.net/`](http://yhh365.3322.net/)

**【软件简介】：** 本软件主要为那些一坐在电脑前就不要命的人设计。主要功能为每半小时就关闭屏幕、锁定键盘 3 分钟，促使你休息。关闭前 10－120 秒提醒你，可提醒两次。工作期间有 5 分钟不用电脑视为已休息，重新计时。注册用户可以调整工作和休息时间长短，并且以后自动按你的设定运行。目前，在 `WINDOWSNT/2000/XP` 上稳定运行，在 `WINDOWS9X` 上可提醒你休息，没锁定键盘。

**【软件限制】：** 功能限制

**【作者声明】：** 初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】：** `TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`Guw32`、`AspackDie`、`W32Dasm 10` 修改版

---

### 【过 程】

`Nurse.exe` 无壳。`Visual C++ 6.0` 编写。

用户号：`559898233`

试炼码：`13572468`

点“注册”后无提示，自动退出。用 `TRW` 下万能断点，返回程序领空后用 `F12` 走过几个 `RET` 就可看到核心了。

---

```
* Reference To: MFC42.Ordinal:18BE, Ord:18BEh
                        |
:004030B2 E8791A0000             Call 00404B30
:004030B7 8B5660                 mov edx, dword ptr [esi+60]
                        ====>EDX=215F5E79（H）=559898233（D）

:004030BA 8D4C240C               lea ecx, dword ptr [esp+0C]
:004030BE 81F23412AB78           xor edx, 78AB1234
                        ====>EDX=215F5E79 ^ 78AB1234=59F44C4D
                        1509182541

:004030C4 895660                 mov dword ptr [esi+60], edx

* Reference To: MFC42.Ordinal:021C, Ord:021Ch
                        |
:004030C7 E8461A0000             Call 00404B12
:004030CC 8B4664                 mov eax, dword ptr [esi+64]
:004030CF 8B4E60                 mov ecx, dword ptr [esi+60]
:004030D2 33DB                   xor ebx, ebx
:004030D4 3BC1                   cmp eax, ecx
                        ====>EAX=00CF1974（H）=13572468  （D）试炼码
                        ====>ECX=59F44C4D（H）=1509182541（D）注册码

:004030D6 899C2420010000         mov dword ptr [esp+00000120], ebx
:004030DD 7511                   jne 004030F0
                        ====>不跳则 OVER！

:004030DF 68F84FB527             push 27B54FF8
                        ====>EAX=666193912   成功的标志！

:004030E4 8D4C2410               lea ecx, dword ptr [esp+10]

* Possible StringData Ref from Data Obj ->"%d"
                        |
:004030E8 68E8934000             push 004093E8
:004030ED 51                     push ecx
:004030EE EB0F                   jmp 004030FF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004030DD(C)
|
:004030F0 6838C54C36             push 364CC538
                        ====>EAX=911000888   失败的标志！

:004030F5 8D542410               lea edx, dword ptr [esp+10]

* Possible StringData Ref from Data Obj ->"%d"
                        |
:004030F9 68E8934000             push 004093E8
:004030FE 52                     push edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004030EE(U)
|

* Reference To: MFC42.Ordinal:0B02, Ord:0B02h
                        |
:004030FF E8F81A0000             Call 00404BFC
:00403104 8B442418               mov eax, dword ptr [esp+18]
                        ====>置 标志！

:00403108 83C40C                 add esp, 0000000C
:0040310B 8D4C2418               lea ecx, dword ptr [esp+18]
:0040310F 50                     push eax

* Possible StringData Ref from Data Obj ->"%s"
                        |
:00403110 68D0944000             push 004094D0
:00403115 51                     push ecx

* Reference To: MSVCRT.sprintf, Ord:02B2h
                        |
:00403116 FF15C8634000           Call dword ptr [004063C8]
:0040311C 83C40C                 add esp, 0000000C

* Reference To: MFC42.Ordinal:0486, Ord:0486h
                        |
:0040311F E8741B0000             Call 00404C98
:00403124 8B10                   mov edx, dword ptr [eax]
:00403126 8D442418               lea eax, dword ptr [esp+18]
:0040312A 50                     push eax
:0040312B 8D4C2414               lea ecx, dword ptr [esp+14]
:0040312F 89542414               mov dword ptr [esp+14], edx

* Reference To: MFC42.Ordinal:035C, Ord:035Ch
                        |
:00403133 E8D4190000             Call 00404B0C
:00403138 8B4C2410               mov ecx, dword ptr [esp+10]
:0040313C 8D542414               lea edx, dword ptr [esp+14]
:00403140 52                     push edx
:00403141 6806000200             push 00020006
:00403146 8B41F8                 mov eax, dword ptr [ecx-08]
:00403149 53                     push ebx

* Possible StringData Ref from Data Obj ->"Software\\Microsoft\\MS Setup (ACME)\\User "
                  ->"Info"
                        |
:0040314A 68B8934000             push 004093B8
:0040314F 6801000080             push 80000001
:00403154 C684243401000001       mov byte ptr [esp+00000134], 01
:0040315C 8D7801                 lea edi, dword ptr [eax+01]

* Reference To: ADVAPI32.RegOpenKeyExA, Ord:0172h
                        |
:0040315F FF1514604000           Call dword ptr [00406014]
                        ====>写入注册表！
```

---

### 【算 法 总 结】

呵呵，`very easy`！

`系列号 = 用户号 XOR 78AB1234` 的十进制值！

---

### 【C++ KeyGen】

呵呵，刚看了几天的 `C++`，偶然碰到了这个超级简单的算法。 ^-^ ^-^

就用我这“超级蹩脚”的 `C++` 做 `fly` 的第一个算法注册机吧！诸位老师见笑了！

```cpp
#include<iostream.h>

#include<math.h>

void main()
{
    int m;
    cout<<"\n★★★★小护士 V1.2 KeyGen{2th}★★★★\n\n\n\n";
    cout<<"请输入您的用户号:";
    cin>>m;
    m^=0X78AB1234;
    cout<<"\n 呵呵，软件系列号:"<<m<<endl;
    cout<<"\n\n\nCracked By 巢水工作坊——fly【OCN】 03-3-31  8:33 编译";
    cout<<"\n\n\n              * * * 按回车退出！* * *";
    cin.get();
    cin.get();
}
```

---

### 【KeyMake 之{49th}内存注册机】

中断地址：`4030D4`

中断次数：`1`

第一字节：`3B`

指令长度：`12`

寄存器方式：`ECX`

十进制

---

### 【注册信息保存】

```
REGEDIT4

[HKEY_CURRENT_USER\Software\Microsoft\MS Setup (ACME)\User Info]
"DefName"="666193912"
```

---

### 【整 理】

用户号：`559898233`

系列码：`1509182541`

---

Cracked By 巢水工作坊——`fly`【`OCN`】

03-3-31 8:32 `4dd466d0e1e0cc123afbe504da2462ca`
