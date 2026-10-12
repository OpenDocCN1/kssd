
# Serv-U 3.0 beta build 13

> 原文：[`www.pediy.com/kssd/pediy03/forum860.htm`](https://www.pediy.com/kssd/pediy03/forum860.htm)

软件名称：`Serv-U 3.0 beta build 13`

从看雪上下了几部教程看了两天，有点手痒。正好我用的 FTP 服务器软件 `serv-u 3.0 beta build 13` 过期了。到网上转了一圈没找到破解版，得了，就拿它开刀吧。

`Serv-U` 有两个可执行文件，`ServUAdmin.exe` 和 `ServUDaemon.exe`，如果日期超过 2001.3.15，执行 `ServUAdmin.exe` 就会弹出个破框说过期了。通常的做法是记下出错信息，将 `ServUAdmin.exe` 用 `W32dasm8.93` 反汇编后用出错信息去定位。如果你这样做，你就陷入万劫不复的深渊了（just like me，极度郁闷），虽然可去掉破框，但当从客户端 FTP 时，却永远上不去，郁闷吧？

实际上，真正要下刀子的是 `ServUDaemon.exe`，它启动后，会产生一个后台服务进程。但它无论运行是否正常，都不产生任何信息。反汇编后，去字符串引用中看一看，会发现 `'This trial version of Serv-U'`，是它，一定是它。

```
:004714D1 895338                   mov dword ptr [ebx+38], edx
:004714D4 E9F1000000               jmp 004715CA
:004714D9 68A0000000               push 000000A0
:004714DE 8D8DE8FDFFFF             lea ecx, dword ptr [ebp+FFFFFDE8]
:004714E4 51                       push ecx
* Possible Reference to String Resource ID=02075: "This trial version of Serv-U is out-of-date! All domains are"
                                      |
:004714E5 681B080000               push 0000081B
:004714EA FF7320                   push [ebx+20]
```

哈，只要到 `004714D4` 就可跳过，向前看一下，没有 `cmp`、`jnz`、`jz` 等，用 `softice` 跟一下吧。分别设断点为 `004714D1` 和 `004714D4`，结果程序可以执行到 `004714D4`，却到不了 `004714D1`，说明前面有间接跳转到 `004714D1` 和 `004714D4` 之间某一处，向前 `PgUp` 几次，

```
:00471397 FF7338                   push [ebx+38]
:0047139A E875AAFEFF               call 0045BE14
:0047139F 59                       pop ecx
:004713A0 8BF0                     mov esi, eax
:004713A2 8BC6                     mov eax, esi
:004713A4 83C003                   add eax, 00000003
:004713A7 83F805                   cmp eax, 00000005
:004713AA 0F871A020000             ja 004715CA
:004713B0 FF2485B7134700           jmp dword ptr [4*eax+004713B7] <-----就是它了
:004713B7 D9144700                 DWORD 004714D9
:004713BB 1F154700                 DWORD 0047151F
:004713BF 8D154700                 DWORD 0047158D
:004713C3 CA154700                 DWORD 004715CA
:004713C7 D4134700                 DWORD 004713D4
:004713CB 96144700                 DWORD 00471496
```

将 `jmp dword ptr [4*eax+004713B7]` 改为：

```
jmp 00471D43
nop
nop
```

保存，搞定。
4dd466d0e1e0cc123afbe504da2462ca
