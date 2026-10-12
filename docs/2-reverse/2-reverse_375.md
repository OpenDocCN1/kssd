
# IDA Pro 简介

> 原文：[`www.pediy.com/kssd/tutorial/chap4-3.htm`](https://www.pediy.com/kssd/tutorial/chap4-3.htm)

**IDA Pro 简介**

程式猎人翻译

（译者注：欢迎访问程式猎人的主页：[`programhunter.126.com`](http://programhunter.126.com)）

对象：`IDA Pro` 反汇编软件
目标: N/A
URL: [`www.datarescue.com`](http://www.datarescue.com)
作者： BlackB
日期： 2000-06-05
使用工具: Your brain
难度等级(级 1-5): 1

## 开始前

IDA Pro Advanced 是一个极好的反汇编工具，它大的胜过了 `w32dasm`。近来它的盗版已经在网上传播开了，你可以很轻松得到它。对于这方面的观点有如下几种：一些人认为这是好的，是自由的；而另一些人认为是可耻的，那些认为是可耻的人通常是这个软件的老用户，事实上他们买了那个程序，如果你花了$500 买了那个程序的话，你就可以理解他们的反应了。对于读者来说，你们做什么不是我的问题及我所关心的。而我的观点是如果认为非法的 `winzip` 软件同 `IDA Pro` 4.0x 一样好或坏。如果你使用非法的 `winzip`，你就不应当使用非法的 `IDA Pro`。

## I 介绍

几天前我烈军属了一篇关于解密程序的教程，而这篇也是一个对于高效的，极好的 `IDA Pro` 4.0x 教程。通常菜鸟不喜欢，而高手又崇拜它。不喜欢的原因是因为 `IDA` 相对于 `w32dasm` 来说有更多的附加功能和作用，有更大的复杂性。

在下面的教程中由于你过去可能喜欢使用 `w32dasm`，我将尽力给出一个简单和清楚的解释来介绍如何使用 `IDA`。也将试差回答"什么是最好的反汇编？"这里其实没有一个简单的回答。对于更多的高手来说可以读这个：向菜鸟 100％解释 `IDA` 的使用过程。

## II. `IDA`: 一个高效的反汇编工具

当你运行 `IDA Pro` 时，你所最先注意到的是它的界面比 `w32dasm` 更加专业，这里比 `w32dasm` 有更多的选项或更先进的地方。它的优点是可以更好的反汇编和更有深层分析。而缺点是使用 `IDA` 更困难。

实际上 `IDA` 同 `w32dasm` 有很多相同的功能：可以快速到达指定的代码位置；可以看到跳到指定的位置的 `jmp` 的命令位置；可以看参考字符串；可以保存静态汇编等。

现在让我们运行和反汇编一个小程序（我使用 `starclean.exe`）一个窗口弹出来，那里有很多选项，不要动它，仅仅点击 `OK`。让 `IDA` 来分析，当它结束后在 `name` 的选项中同 `w32dasm` 中参考字符串一样，查找"a"字符。在 `name` 窗口中搜索栏中按 `a`，然后开始查找，一旦找到，按 `enter` 或双击它，它就将到达正确的位置。按 `ctrl+enter` 将到达下一个位置。参考字符串就在光标附近。`IDA` 就可以到达相应的位置了。

另一个同 `w32dasm` 相同的：`jmp` 和 `call` 参考在 `IDA` 也是有效的。在 `w32dasm` 反汇编的代码同 `IDA` 中相同的。

**代码开始(w32dasm)**

```
\* 
Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00403B50(C) 
| :00403B5F 81FAFC000000 cmp edx, 000000FC :00403B65 0F8478010000 je 00403CE3 
:00403B6B 8D8424B4000000 lea eax, dword ptr \[esp+000000B4\] :00403B72 6804010000 
push 00000104 :00403B77 50 push eax \* Reference To: KERNEL32.GetModuleFileNameA, 
Ord:00E9h | :00403B78 8B2D88924000 mov ebp, dword ptr \[00409288\] :00403B7E 6A00 
push 00000000 :00403B80 FFD5 call ebp :00403B82 85C0 test eax, eax :00403B84 7516 
jne 00403B9C \* Possible StringData Ref from Data Obj ->"
" 
| :00403B86 BE206A4000 mov esi, 00406A20 :00403B8B 8DBC24B4000000 lea edi, dword 
ptr \[esp+000000B4\] :00403B92 B905000000 mov ecx, 00000005 :00403B97 F3 repz :00403B98 
A5 movsd :00403B99 66A5 movsw :00403B9B A4 movsb
```

**代码开始(IDA Pro 4.0x)**

```
00403B5F 
loc\_403B5F: ; CODE XREF: sub\_403B00+50j 00403B5F cmp edx, 0FCh 00403B65 jz loc\_403CE3 
00403B6B lea eax, \[esp+1B8h+var\_104\] 00403B72 push 104h 00403B77 push eax 00403B78 
mov ebp, ds:GetModuleFileNameA 00403B7E push 0 00403B80 call ebp ; GetModuleFileNameA 
00403B82 test eax, eax 00403B84 jnz short loc\_403B9C 00403B86 mov esi, offset 
aProgramNameUnk ; "" 00403B8B lea edi, \[esp+1B8h+var\_104\] 
00403B92 mov ecx, 5 00403B97 repe movsd 00403B99 movsw 00403B9B movsb 
```

你在这里看到，`IDA` 提供了更多的细节信息，它是更加清楚，用一个参考作为例子。

在 `w32dasm` 中如下：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403B50(C)

在 `IDA Pro` 中如下：

CODE XREF: `sub_403B00+50j`

`IDA` provides 使用更短的行来提供更多的信息。

1.  用一个 `jump` (`j`)
2.  `jump` 到代码的位置位于 `00403B50` (=403B00+50)
3.  函数/`call` 本身在 `403B00` (403B00+50)
4.  `jump` 在 `.code` section

如果参考的是一个 `call` 那么它将如下：

CODE XREF: `sub_403B00+50p`

另一个大的优点是 `IDA` 可以转换变量。你可以双击变量以找到它们保存的地址处: `lea edi, [esp+1B8h+var_104]`。

功能中最有用的之一是使用近似查找。很容易解释：`IDA` 尽力记住寄存器所关联的库函数。如下：

**代码开始 (IDA Pro)**

```
004012D2 
loc\_4012D2: ; CODE XREF: sub\_401280+5Fj 004012D2 movsx ecx, byte ptr \[eax\] 004012D5 
push eax 004012D6 lea ebp, \[ebp+ecx\*2+0\] 004012DA call edi ; CharNextA ; Return 
a pointer to the next character in a
```

**代码开始 (w32dasm)**

```
\* 
Referenced by a (U)nconditional or (C)onditional Jump at Address: |:004012DF(C) 
| :004012D2 0FBE08 movsx ecx, byte ptr \[eax\] :004012D5 50 push eax :004012D6 8D6C4D00 
lea ebp, dword ptr \[ebp+2\*ecx\] :004012DA FFD7 call edi :004012DC 803800 cmp byte 
ptr \[eax\], 00 :004012DF 75F1 jne 004012D2
```

你注意到它们的不同了吗？`IDA` 将向解释每一件事情。

在 `IDA` 中另一个同 `w32dasm` 近似的是十分简单的，如："Goto code location" 等同于在 `IDA` "Jump - Jump to adress" . 或者 "Goto entrypoint" 等同于在 `IDA` "Jump - Jump to entry point" .

那些就是在你了解 `IDA` 开始使用它前最为重要的东西。有时 ，`IDA` 显示大量的 "db"，参考字符串，仅有代码。仅仅可以设好光标，按 "c" 键，`IDA` 转换代码如下

**开始 (在按'C'之前)**

```
00401228 
dd offset unk\_406130 0040122C db 8Bh ; ?0040122D db 35h ; 5 0040122E dd offset 
RegSetValueExA 00401232 db 6Ah ; j 00401233 db 1 ; 00401234 db 6Ah ; j 00401235 
db 0 ; 00401236 db 68h ; h 00401237 dd offset aName ; "Name" 0040123B db 51h ; 
Q 0040123C db 0FFh ; 0040123D db 0D6h ; ?0040123E db 8Bh ; ?0040123F db 4Ch ; 
L 00401240 db 24h ; $ 00401241 db 8 ; 00401242 db 68h ; h
```

**开始 (在按'C'之后)**

```
0040122C 
mov esi, ds:RegSetValueExA 00401232 ; --------------------------------------------------------------------------- 
00401232 push 1 00401234 push 0 00401236 push offset aName ; "Name" 0040123B ; 
\--------------------------------------------------------------------------- 0040123B 
push ecx 0040123C call esi ; RegSetValueExA 0040123E mov ecx, \[esp+8\] 00401242 
push 100h 00401247 push offset unk\_406030 0040124C ; --------------------------------------------------------------------------- 
0040124C push 1 0040124E push 0 00401250 push offset aCode ; "Code" 00401255 ; 
\--------------------------------------------------------------------------- 00401255 
push ecx 00401256 call esi ; RegSetValueExA 00401258 mov eax, \[esp+8\] 0040125C 
push eax 0040125D call ds:RegCloseKey
```

`IDA` 另一个优点（快速相加）:

1.  能够对 `w32dasm` 无法反汇编的最难的软件进行反汇编（如加壳程序）
2.  能够以 `.asm` `.sym` 和甚至是 `.exe` 及其它文件形式保存
3.  压缩的静态汇编，可以节省大量的磁盘空间。
4.  可以重命名函数
5.  能够分析巨大的程序

等等.... ;-)

## III. `w32dasm`? ?

你看完了后你现在可以自问"如果 `IDA` 这么好看，为什么我还要使用 `w32dasm`？"下面就是对于这个的解释：

首先是速度，对于 `IDA` 来分析一个小程序是非常非常的快，但是如果分析一个大的 `exe` 它就要花费 3 个小时或更多的时间来全面分析 `exe`。

其次，如果你对于一个指定的搜索，`IDA` 将很慢的，而在 `w32dasm` 中在文本方面中搜索是很快的。

第三：当分析一个仅有简单的保护的程序时，`w32dasm` 就为首选。因为你就不需要在 `IDA` 那些附加的功能了。

上面的三个因素可以说明一切了 :-) 如果你知道其它的使用 `w32dasm` 原因的话，请告诉我。

## IV. 结束

又一个教程结束了。享受 `IDA` and `w32dasm` 以及所有.....享受研究和保持学习和继续研究和破解吧。不是为了解密，仅是文章，写下你的文章，传播你的知识。对于解密这是重要的来说。不要用于解密吧。

感谢大家

**The Blackbird**

**Endnote:**

The Blackbird 创作于 1999-2000

此文章可以自由分发，出版，打印……等尽可能长的修饰。

4dd466d0e1e0cc123afbe504da2462ca
