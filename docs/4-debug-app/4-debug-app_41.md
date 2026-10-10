

# 一串字符的解密

> 原文：[`www.pediy.com/kssd/pediy12/143671.html`](https://www.pediy.com/kssd/pediy12/143671.html)

无聊，逛博客(http://hi.baidu.com/wordexp/blog/item/f33479433a1bad1873f05d99.html)的时候发现一串有技巧的字符串:  
   

代码:

```
000102030405060708090A0B0C0D0E0F
00:526172211A0700CF907300000D000000
10:0000000077B77420902B00BA00000038
20:010000024FDBC8236865393C1D330600
30:2000000043432E45584500F010A96708
40:150D10BE0D66F3DA083C2BF0286D0DC9
50:C0DAD1D42D4A87588829C86BF07D437B
60:5234BF0A88FC3B5221CD8FC4363F09B5
70:EA1D375068762F51D4BF078484C0CC99
80:09F479EA3C7AAC33D266424AE7C063CB
90:B82A07A1F75AD73AE402E209FC1323C0
A0:C588C60E76B3611CEE160D58C7829C0A
B0:6443DFB6411E8EA14B24BA9B1E65D9FB
C0:43A17725D3AA512F512C4319C74F903F
D0:824F15DA6DBF6E0397FBE7377A4D9D3B
E0:E110E155C8AB8B847ED52B853B11DB22
F0:7836B5F57F44C44F28C43D7B00400700

```

然后看了下面的 x 牛回复的汇编我很迷茫，介个怎么会变成那个呢？  
我很好奇，接着我就开始挣扎了。中间问了好多人。最后终于眉目来了。当然知道了一些细节后，实际过程也就明朗了。  
这里只是个技巧，老鸟请飘过。。。。。。  
  
1.首先把这串数据除了第一行和那些偏移值外全部粘贴到 WinHex 中，得到  
代码:

```
52 61 72 21 1A 07 00 CF    90 73 00 00 0D 00 00 00  Rar!...s......
00 00 00 00 77 B7 74 20     90 2B 00 BA 00 00 00 38  ....wt +....8
01 00 00 02 4F DB C8 23    68 65 39 3C 1D 33 06 00  ....O#he9<.3..
20 00 00 00 43 43 2E 45     58 45 00 F0 10 A9 67 08   ...CC.EXE..g.
15 0D 10 BE 0D 66 F3 DA   08 3C 2B F0 28 6D 0D C9  ....fó.<+(m.
C0 DA D1 D4 2D 4A 87 58   88 29 C8 6B F0 7D 43 7B  -JX)k}C{
52 34 BF 0A 88 FC 3B 52    21 CD 8F C4 36 3F 09 B5  R4.ü;R!6?.
EA 1D 37 50 68 76 2F 51     D4 BF 07 84 84 C0 CC 99  ê.7Phv/Q.
09 F4 79 EA 3C 7A AC 33    D2 66 42 4A E7 C0 63 CB  .yê<z3fBJc
B8 2A 07 A1 F7 5A D7 3A    E4 02 E2 09 FC 13 23 C0  \*.÷Z×:..ü.#
C5 88 C6 0E 76 B3 61 1C    EE 16 0D 58 C7 82 9C 0A  .va...X.
64 43 DF B6 41 1E 8E A1    4B 24 BA 9B 1E 65 D9 FB  dCA.K$.e
43 A1 77 25 D3 AA 51 2F    51 2C 43 19 C7 4F 90 3F  Cw%Q/Q,C.O?
82 4F 15 DA 6D BF 6E 03    97 FB E7 37 7A 4D 9D 3B  O.mn.7zM;
E1 10 E1 55 C8 AB 8B 84    7E D5 2B 85 3B 11 DB 22  á.áU~+…;."
78 36 B5 F5 7F 44 C4 4F     28 C4 3D 7B 00 40 07 00  x6DO(={.@..

```

可以发现是一个开头有一个 Rar 标志，中间寻找可以发现 CC.EXE。把它保存为.rar 格式文件。  
  
2.解压该 rar 文件，得到 CC.EXE  
  
3.使用 LordPE 查看该 CC.EXE 的 PE 信息。可以发现程序入口点是 000000F8.考虑到快对齐与文件对齐相等，直接在 WinHex 打开 CC.EXE 并定位到 000000F8 处，把 000000F8 处到结束的字符都拷贝下来，拷贝到 WinHex 中。命名为 hehe.bin  
  
4.习惯有时候跑到 Linux 下去玩玩，用 hexdump 看一下。  
代码:

```
\[root@localhost ~\]# hexdump -C hehe.bin
00000000  60 e8 0e 00 00 00 8b 44  24 0c 05 b8 00 00 00 ff  |\`......D$.......|
00000010  00 33 c0 c3 5e 64 a1 30  00 00 00 05 00 08 00 00  |.3..^d.0........|
00000020  8b f8 a5 a5 a5 a5 50 33  c0 64 ff 30 64 89 20 cc  |......P3.d.0d. .|
00000030  58 64 a3 00 00 00 00 83  c4 04 61 c3 00 00 00 00  |Xd........a.....|

```

5.使用 ndisasm 对这段字符进行操作。  
代码:

```
\[root@localhost ~\]# ndisasm -u hehe.bin > hehe.txt

```

6.最终得到如下情景，与博客中下面 x 的那段汇编代码基本一致。  
代码:

```
\[root@localhost ~\]# cat hehe.txt | more
00000000  60                 pushad
00000001  E80E000000        call dword 0x14
00000006  8B44240C          mov eax,\[esp+0xc\]
0000000A  05B8000000        add eax,0xb8
0000000F  FF00              inc dword \[eax\]
00000011  33C0              xor eax,eax
00000013  C3                ret
00000014  5E                pop esi
00000015  64A130000000      mov eax,\[fs:0x30\]
0000001B  0500080000        add eax,0x800
00000020  8BF8              mov edi,eax
00000022  A5                movsd
00000023  A5                movsd
00000024  A5                movsd
00000025  A5                movsd
00000026  50                push eax
00000027  33C0              xor eax,eax
00000029  64FF30            push dword \[fs:eax\]
0000002C  648920            mov \[fs:eax\],esp
0000002F  CC                int3
00000030  58                pop eax
00000031  64A300000000      mov \[fs:0x0\],eax
00000037  83C404            add esp,byte +0x4
0000003A  61                popad
0000003B  C3                ret

```

因为见识不够，不知道是不是这种方式来的，就当作参考哦，时间有限，就到此咯，谢谢。