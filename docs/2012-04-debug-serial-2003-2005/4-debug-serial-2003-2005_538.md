
# 卸载精灵 v1.22

> 原文：[`www.pediy.com/kssd/pediy05/pediy50156.htm`](https://www.pediy.com/kssd/pediy05/pediy50156.htm)

## 算法分析

＜献给初学者＞ 之二

**作者**：`goodbao[P.J.CHINA]`

### 软件信息

- **软件名称**：卸载精灵 v1.22
- **下载地址**：`http://www.softreg.com/download.asp?id={C3347316-B547-4DDC-80C3-884F2FBF3BA8}`

### 软件简介

已经厌倦了 WINDOWS 控制面板那个功能很弱的“添加/删除程序”了吗？你是否发现你的电脑里面有些软件无法卸载？＊＊＊＊的智能卸载功能可以干净彻底地卸载软件，包括那些无法正常卸载的程序它都可以帮您卸载。

### 破解难度

非明码，较难。（此软件非算法不可解。当然，除爆破！）

### 算法难度

较难

请随我来一步步看看它的算法吧！！`LET'S GO`！！！！！！

下断点 `BPX HMEMCPY`，然后填写注册名与注册码（注册名任意，注册码必须 8 位，不能多也不能少。为什么？请看下边嘛！）。当然，在不知道的情况下，可以任意填写注册码。然后确定，然后中断，然后清断，然后 19 次 `F12`（为什么 19 次？不会吧，这个问题不用回答了吧？），会到达下边：

```
:00403BF0 E8ED580100 call 004194E2

:00403BF5 8D4C2414 lea ecx, dword ptr [esp+14] //我们到达这里

:00403BF9 8DBE98000000 lea edi, dword ptr [esi+00000098]

:00403BFF 51 push ecx

:00403C00 8BCF mov ecx, edi

:00403C02 E8DB580100 call 004194E2

:00403C07 8B542414 mov edx, dword ptr [esp+14]

* Possible Reference to Dialog:

:00403C0B 6874134300 push 00431374

:00403C10 52 push edx

:00403C11 E8AB830000 call 0040BFC1

:00403C16 83C408 add esp, 00000008

:00403C19 85C0 test eax, eax

:00403C1B 0F8458010000 je 00403D79

:00403C21 8B442410 mov eax, dword ptr [esp+10]

* Possible Reference to Dialog:

:00403C25 6874134300 push 00431374

:00403C2A 50 push eax

:00403C2B E891830000 call 0040BFC1

:00403C30 83C408 add esp, 00000008

:00403C33 85C0 test eax, eax

:00403C35 0F843E010000 je 00403D79

:00403C3B 51 push ecx

:00403C3C 8D542414 lea edx, dword ptr [esp+14]

:00403C40 8BCC mov ecx, esp

:00403C42 89642420 mov dword ptr [esp+20], esp

:00403C46 52 push edx

:00403C47 E89F7D0100 call 0041B9EB

:00403C4C 8B0D70134300 mov ecx, dword ptr [00431370]

:00403C52 E8D9FCFFFF call 00403930 //F8 跟入

:00403C57 3BC3 cmp eax, ebx

:00403C59 0F84DC000000 je 00403D3B //跳则死

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝进入雷区喽＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

:00403930 64A100000000 mov eax, dword ptr fs:00000000

:00403936 6AFF push FFFFFFFF

:00403938 6848394200 push 00423948

:0040393D 50 push eax

:0040393E 64892500000000 mov dword ptr fs:00000000], esp

:00403945 56 push esi

:00403946 8B442414 mov eax, dword ptr [esp+14]

:0040394A 8B48F8 mov ecx, dword ptr [eax-08]

:0040394D 83F908 cmp ecx, 00000008 //注册码与 8 比较

:00403950 7425 je 00403977 //不等就不跳，不跳就死喽！

:00403952 8D4C2414 lea ecx, dword ptr [esp+14]

:00403956 C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:0040395E E813830100 call 0041BC76

:00403963 33C0 xor eax, eax

:00403965 8B4C2404 mov ecx, dword ptr [esp+04]

:00403969 64890D00000000 mov dword ptr fs:00000000], ecx

:00403970 5E pop esi

:00403971 83C40C add esp, 0000000C

:00403974 C20400 ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00403977 8A10 mov dl, byte ptr [eax] //取注册码第一位字母的 16 进制值放入 DL

:00403979 0FBE4802 movsx ecx, byte ptr [eax+02] //取注册码第三位字母的 16 进制值，赋给 ECX

:0040397D 0FBEF2 movsx esi, dl //第一位值赋给 ESI

:00403980 8D4C31A0 lea ecx, dword ptr // 计算结果保存到 ECX

:00403984 83F907 cmp ecx, 00000007 //比较刚才计算的结果是否等于 7

:00403987 7425 je 004039AE //不等就不跳，那就死喽！

:00403989 8D4C2414 lea ecx, dword ptr [esp+14]

:0040398D C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:00403995 E8DC820100 call 0041BC76

:0040399A 33C0 xor eax, eax

:0040399C 8B4C2404 mov ecx, dword ptr [esp+04]

:004039A0 64890D00000000 mov dword ptr fs:00000000], ecx

:004039A7 5E pop esi

:004039A8 83C40C add esp, 0000000C

:004039AB C20400 ret 0004 //不跳就从这里退出此 CALL 了，

那就 GAME OVER 了！(下同)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004039AE 8A4801 mov cl, byte ptr [eax+01] //取第二位字母的 16 进制 值放入 CL

:004039B1 0FBE7003 movsx esi, byte ptr [eax+03] //取第四位字母的 16 进制 值，赋给 ESI

:004039B5 0FBEC9 movsx ecx, cl //CL 值送 ECX

:004039B8 8D4C0EA0 lea ecx, dword ptr [esi+ecx-60]//计算结果存入 ECX

:004039BC 83F908 cmp ecx, 00000008 //结果与 8 比较

:004039BF 7425 je 004039E6 //不等就死喽！！！

:004039C1 8D4C2414 lea ecx, dword ptr [esp+14]

:004039C5 C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:004039CD E8A4820100 call 0041BC76

:004039D2 33C0 xor eax, eax

:004039D4 8B4C2404 mov ecx, dword ptr [esp+04]

:004039D8 64890D00000000 mov dword ptr fs:00000000], ecx

:004039DF 5E pop esi

:004039E0 83C40C add esp, 0000000C

:004039E3 C20400 ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004039E6 8A4804 mov cl, byte ptr [eax+04] //取第五位字母值入 CL

:004039E9 0FBE7006 movsx esi, byte ptr [eax+06] //取第七位字母值入 ESI

:004039ED 0FBEC9 movsx ecx, cl //CL 送 ECX

:004039F0 8D4C0EA0 lea ecx, dword ptr [esi+ecx-60]//同上计算，结果入 ECX

:004039F4 83F909 cmp ecx, 00000009 //结果与 9 比较

:004039F7 7425 je 00403A1E //不等就死喽！！！

:004039F9 8D4C2414 lea ecx, dword ptr [esp+14]

:004039FD C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:00403A05 E86C820100 call 0041BC76

:00403A0A 33C0 xor eax, eax

:00403A0C 8B4C2404 mov ecx, dword ptr [esp+04]

:00403A10 64890D00000000 mov dword ptr fs:00000000], ecx

:00403A17 5E pop esi

:00403A18 83C40C add esp, 0000000C

:00403A1B C20400 ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00403A1E 8A4805 mov cl, byte ptr [eax+05] //取第六位字母值入 CL

:00403A21 8A4007 mov al, byte ptr [eax+07] //取第八位字母值入 AL

:00403A24 0FBEF0 movsx esi, al //AL 送 ESI

:00403A27 0FBEC9 movsx ecx, cl //CL 送 ECX

:00403A2A 8D4C0EA0 lea ecx, dword ptr [esi+ecx-60]//同上计算，结果入 ECX

:00403A2E 83F90A cmp ecx, 0000000A //结果与 10 比较

:00403A31 7425 je 00403A58 //不等就死喽！

:00403A33 8D4C2414 lea ecx, dword ptr [esp+14]

:00403A37 C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:00403A3F E832820100 call 0041BC76

:00403A44 33C0 xor eax, eax

:00403A46 8B4C2404 mov ecx, dword ptr [esp+04]

:00403A4A 64890D00000000 mov dword ptr fs:00000000], ecx

:00403A51 5E pop esi

:00403A52 83C40C add esp, 0000000C

:00403A55 C20400 ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00403A58 0FBEC0 movsx eax, al //第八位值送 EAX

:00403A5B 0FBECA movsx ecx, dl //第一位值送 ECX

:00403A5E C744240CFFFFFFFF mov [esp+0C], FFFFFFFF

:00403A66 8D5408A0 lea edx, dword ptr [eax+ecx-60]//计算，结果入 EDX

:00403A6A 8D4C2414 lea ecx, dword ptr [esp+14]

:00403A6E 83FA08 cmp edx, 00000008 //结果等于 8 吗？

:00403A71 7419 je 00403A8C //不等就死喽！！！！

:00403A73 E8FE810100 call 0041BC76

:00403A78 33C0 xor eax, eax

:00403A7A 8B4C2404 mov ecx, dword ptr [esp+04]

:00403A7E 64890D00000000 mov dword ptr fs:00000000], ecx

:00403A85 5E pop esi

:00403A86 83C40C add esp, 0000000C

:00403A89 C20400 ret 0004
```

## 算法总结

从上边的分析可以看出，此软件的算法是这样的：

1. 输入的注册码必须是 8 位，且注册码与注册名无关。（但是，不能让注册名空）
2. 涉及的计算公式是统一的，`x` 先后取值为 7、8、9、10、8。
3. 具体计算过程：
    ① 注册码的第一位 16 进制值 ＋ 第三位 16 进制值 － 60 ＝ 7
    ② 注册码的第二位 16 进制值 ＋ 第四位 16 进制值 － 60 ＝ 8
    ③ 注册码的第五位 16 进制值 ＋ 第七位 16 进制值 － 60 ＝ 9
    ④ 注册码的第六位 16 进制值 ＋ 第八位 16 进制值 － 60 ＝ 10
    ⑤ 注册码的第一位 16 进制值 ＋ 第八位 16 进制值 － 60 ＝ 8

所以，只要满足以上条件，就可以任意注册喽！！！！！！！

那就随便编一个注册码吧：`12666337`。当然，`50283763` 也行啊！！呵呵，简直太多啦！！！

呵呵，看来这个软件拿来编写注册机就合适不过了。呵呵。。。。。。。。

--------------------------------------------------------------------------------

神龙宝宝

`goodbao[P.J.CHINA][BCG][CNCG]` `4dd466d0e1e0cc123afbe504da2462ca`
