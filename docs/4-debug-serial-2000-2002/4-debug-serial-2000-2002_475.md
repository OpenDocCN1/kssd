
# Snappy Fax 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum588.htm`](https://www.pediy.com/kssd/pediy03/forum588.htm)

软件名称：`Snappy Fax 2000`

整理日期：`2001.1.19`

最新版本：`2.11.2.1`

文件大小：`2075KB`

软件授权：共享软件

使用平台：`Win95/98/NT/2000`

发布公司：`Home Page`

软件简介：

运用您的扫描器、打印机和 `Fax Modem` 将您的计算机变成一台传真机。

破解工具：`W32dasm`，`HIEW`，这两个当然少不了。

下载地址：`http://www.newhua.com/SNAPPYFAX.HTM`

近几天，在论坛里好像看到很多网友在玩暴力破解，所以今天也来热闹热闹，我不喜欢玩暴力破解，原因只有二个，因为玩这个东西的人有二种，（在没有注册码输入的软件除处）一种是高高手，另一种就是小菜类，我当然是后者了，说句老实话，程序又不是自己写的，又怎么知道它如何走的呢？一般的暴力破解只是改一两个地方就成为注册版了，多没趣，当然前提条件是作者没有装陷井的的前提下了，大家在玩暴力破解的时候可能都会遇到这个问题，就是改了自己认为的地方，但后来的结果发现错误，都会怀着运气+瞎猜的方法去改到正确，我不提倡初学者去玩暴力破解，用 `ICE` 去看看程序的流程不是还要好吗？以上内容只是我的个人意见，不认同的，可以当我没有说过（不是的话又有人骂我了！）

一般用暴力破解的入手都是看看软件中的提示，也就是破解的入口点了，就 `Snappy Fax 2000` 这个软件来说，在你填上注册名和错误的注册号后，它就会跳出一个错误的窗口，说 `THE REGISTRATION KEY IS NOT VALID` 这个就是我们破解它的入口点，用 `W32dasm` 来反汇编它后找这些提示就行了。

我就用 `W32dasm` 将 `Snappy Fax 2000 EXE` 文件反汇编后找到 `THE REGISTRATION KEY IS NOT VALID` 如下：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0054A7B8(C)--------------------------------------------找这个地址就行了

\* Possible StringData Ref from Code Obj ->"alert.wav"

:0054A884 B80CA95400                 mov eax, 0054A90C
:0054A889 E8724E1000                 call 0064F700
:0054A88E 6A00                       push 00000000
:0054A890 668B0D18A95400             mov cx, word ptr [0054A918]
:0054A897 B201                       mov dl, 01

\* Possible StringData Ref from Code Obj ->"The Registration Key is not Valid."—————————这就是破解的入口点

:0054A899 B858A95400                 mov eax, 0054A958
:0054A89E E8A16BF2FF                 call 00471444
:0054A7AA 8B800C030000               mov eax, dword ptr [eax+0000030C]
:0054A7B0 59                         pop ecx
:0054A7B1 E896B1F5FF                 call 004A594C
:0054A7B6 3C01                       cmp al, 01
:0054A7B8 0F85C6000000               jne 0054A884-----------------------这个就是鬼了，用 `HIEW` 改 `JNE` 为 `JE`，即是将 `85` 改为 `84`。按 `F9` 保存后运行它试试，在输入的地方填上你喜欢的名字和注册码就成为注册版了。因为这个是决定你的注册码是否正解后跳转的地方！
:0054A7BE A15CFD6600                 mov eax, dword ptr [0066FD5C]
:0054A7C3 8B00                       mov eax, dword ptr [eax]
:0054A7C5 8B800C030000               mov eax, dword ptr [eax+0000030C]

是否觉得很没趣，再一次说明，以上内容，只是我的观点，不认同者............... 4dd466d0e1e0cc123afbe504da2462ca
