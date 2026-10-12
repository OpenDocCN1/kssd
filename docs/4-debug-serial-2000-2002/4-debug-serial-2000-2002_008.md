
# AmazimgMIDI 1.60

> 原文：[`www.pediy.com/kssd/pediy01/forum047.htm`](https://www.pediy.com/kssd/pediy01/forum047.htm)

软件名称：`AmazimgMIDI 1.60`

软件大小：839KB

软件类型：共享

适用环境：Win95/98

软件介绍：

可以将您现有的 `WAV` 文件转换成为 `MIDI` 文件，`WAV` 可能会有人声或者其它非乐器声音的部分，这个部分可就无能为力了，`AmazingMIDI` 会将这一部分的声音资料转换成为类比的讯号，不过仍然还是以 `MIDI` 的格式储存，您可以在稍后与 `MIDI` 编辑软件中一起编辑使用。虽然 `AmazingMIDI` 在转文件方面的功能已经相当强大了，不过它还是有所限制的，目前版本的 `AmazingMIDI` 并无法直接在众多的乐器声音中只对某一种取样，相信这方面的功能限制应该很快有所改善的。如果对 `WAV` 音乐素材直接取样使用的话，`AmazingMIDI` 是个绝佳的转换工具。

软件来源:<<软件世界>>第 5 期

保护类型:固定注册码

所用工具:`S-ICE`, `W32Dasm`

破解过程:

在 `help` 菜单中选择 `registration`, 任意输入 `12345678`, `CTRL+D` 转到 `S-ICE`, 设断点 `bpx hmemcpy`, 返回软件画面, 点 `OK`. 程序被中断后, 按 `F12` 42 次(第 43 次出错误画面)后, 按 `F10` 若干次, 找到下面程序

```
|:00405DEA(U)
|
:00405D5D 8D4C2410 lea ecx, dword ptr [esp+10]
:00405D61 E8C8780100 call 0041D62E
:00405D66 83F801 cmp eax, 00000001
:00405D69 0F85B6000000 jne 00405E25
:00405D6F 51 push ecx
:00405D70 8D442470 lea eax, dword ptr [esp+70]
:00405D74 8BCC mov ecx, esp
:00405D76 89642410 mov dword ptr [esp+10], esp
:00405D7A 50 push eax
:00405D7B E887B50100 call 00421307
:00405D80 8D4C240C lea ecx, dword ptr [esp+0C]
:00405D84 C644247C00 mov [esp+7C], 00
:00405D89 51 push ecx
:00405D8A E861010000 call 00405EF0
:00405D8F 8BC8 mov ecx, eax
:00405D91 E8AA180000 call 00407640 <----将输入码变形
:00405D96 8B00 mov eax, dword ptr [eax]
* Possible StringData Ref from Data Obj ->"<<@::5<720"<-----(变形后的)注册码
|
:00405D98 BE58054400 mov esi, 00440558
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DBF(C)
|
:00405D9D 8A10 mov dl, byte ptr [eax]<-----取出变形后的输入码
:00405D9F 8A1E mov bl, byte ptr [esi]<-----取出变形后的注册码
:00405DA1 8ACA mov cl, dl
:00405DA3 3AD3 cmp dl, bl
:00405DA5 751E jne 00405DC5<-----错误信息
:00405DA7 84C9 test cl, cl
:00405DA9 7416 je 00405DC1
:00405DAB 8A5001 mov dl, byte ptr [eax+01]
:00405DAE 8A5E01 mov bl, byte ptr [esi+01]
:00405DB1 8ACA mov cl, dl
:00405DB3 3AD3 cmp dl, bl
:00405DB5 750E jne 00405DC5<-----错误信息
:00405DB7 83C002 add eax, 00000002
:00405DBA 83C602 add esi, 00000002
:00405DBD 84C9 test cl, cl
:00405DBF 75DC jne 00405D9D<------未完,继续比较
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DA9(C)
|
:00405DC1 33C0 xor eax, eax<-----比较正确 eax 清零
:00405DC3 EB05 jmp 00405DCA
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405DA5(C), :00405DB5(C)
|
:00405DC5 1BC0 sbb eax, eax
:00405DC7 83D8FF sbb eax, FFFFFFFF<-----比较错误 eax 置位
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DC3(U)
|
:00405DCA 85C0 test eax, eax
:00405DCC 8D4C2408 lea ecx, dword ptr [esp+08]
:00405DD0 0F94C3 sete bl <-----若 eax 为零,bl 置位
:00405DD3 E8BAB70100 call 00421592
:00405DD8 84DB test bl, bl
:00405DDA 7513 jne 00405DEF <-----bl 不为零,显示"Thank you for your registration"
:00405DDC 6AFF push FFFFFFFF
:00405DDE 6A00 push 00000000
* Possible Reference to String Resource ID=61212: "Incomplete or incorrect information."
|
:00405DE0 681CEF0000 push 0000EF1C
:00405DE5 E8DB180200 call 004276C5
:00405DEA E96EFFFFFF jmp 00405D5D
```

分析将输入码变形的 `CALL`, 便可知道程序是如何处理输入码的.

处理过程如下:

将输入码调头, 将其 `ASCII` 码与序号相加.

那么我们根据已知的变形后的注册码反推出注册码

| 序号 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 变形的注册码 | < | < | @ | : | : | 5 | < | 7 | 2 | 0 |
| ASCII 码 | 3C | 3C | 40 | 3A | 3A | 35 | 3C | 37 | 32 | 30 |
| 减序号后 | 33 | 34 | 39 | 34 | 35 | 31 | 39 | 35 | 31 | 30 |

注册码应为: `0159154943`
