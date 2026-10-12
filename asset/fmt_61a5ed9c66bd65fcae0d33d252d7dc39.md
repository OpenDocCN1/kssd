
# 菜鸟学习《加密与解密》BlowFish 笔记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-376.htm`](https://www.pediy.com/kssd/pediy07/pediy7-376.htm)

**【破文作者】** `rdsnow[BCG][PYG][D.4s]`

**【作者主页】** `http://rdsnow.ys168.com`

**【 E-mail 】** `rdsnow@163.com`

**【 作者 QQ 】** `83757177`

**【文章题目】** 菜鸟学习《加密与解密》BlowFish 笔记

**【破解平台】** `Win9x/NT/2000/XP/XP SP2`

\----------------------------------------------------------------------------------------------

## 【文章简介】

这两天跟踪某个软件，其中使用了变了形的 `BlowFish`，虽然可以找到注册码的保存地址做出内存注册机，但是写不出算法注册机，心里很不平衡。以前从来没有接触过密码学知识，就从这个对称算法开始吧！

\----------------------------------------------------------------------------------------------

## 【破解过程】

`BlowFish` 是个 64 位分组的算法。输入 64 信息，运算后输出 64 位密文。

刚开始当然要找个简单的。找来找去，最后拿《加密与解密》光盘上的夜月的 `BlowFish's crackme keygen` 分析了。这个 `keygen` 对 `ID` (待加密数据) 进行了两次 `BlowFish` 的计算。

通过跟踪发现，对某个消息加密的过程，分两步进行：

### 1、密钥预处理过程：

```
6A 12           PUSH 12                             ; /Arg2 = 00000012    压入密钥长度 12
68 00304000     PUSH BFKEYGEN.00403000              ; |Arg1 = 00403000    压入密钥地址
E8 50FEFFFF     CALL BFKEYGEN.004010C8              ; \\BFKEYGEN.004010C8  密钥预处理
```

### 2、对信息加密过程：

```
68 CD404000     PUSH BFKEYGEN.004040CD              ; /Arg2 = 004040CD    被加密信息的地址 1
68 D1404000     PUSH BFKEYGEN.004040D1              ; |Arg1 = 004040D1    被加密信息的地址 2
E8 E5FDFFFF     CALL BFKEYGEN.0040106C              ; \\BFKEYGEN.0040106C  BlowFish 的核心函数 BF_EN 加密函数
```

先来认识一下 `BlowFish` 的两个盒，所谓盒子，我想就是两个数组：

`PBOX` 盒：`unsigned long PBOX[18]`，有 18 个元素的一维数组
`unsigned long PBOX[4,256]`，总共有 1024 个元素的二维数组

这两个盒子里已经保存了一些固定的数值：

```
PBOX    dd 0243f6a88h, 085a308d3h, 013198a2eh, 003707344h, 0a4093822h, 0299f31d0h
        dd 0082efa98h, 0ec4e6c89h, 0452821e6h, 038d01377h, 0be5466cfh, 034e90c6ch
        dd 0c0ac29b7h, 0c97c50ddh, 03f84d5b5h, 0b5470917h, 09216d5d9h, 08979fb1bh
SBOX1   dd 0d1310ba6h, 098dfb5ach, 02ffd72dbh, 0d01adfb7h, 0b8e1afedh, 06a267e96h
        dd 0ba7c9045h, 0f12c7f99h, 024a19947h, 0b3916cf7h, 00801f2e2h, 0858efc16h
        dd 0636920d8h, 071574e69h, 0a458fea3h, …………(因为很多所以就省略了)
SBOX2   ………………
SBOX3   ………………
SBOX4   ………………
```

`MD5` 编码过程中只要用到 4 个常数，而 `BlowFish` 却要用到这么多常数。现在很多的算法 search 工具，就是通过搜索 `SBOX` 来判断程序使用了什么算法的。

#### ★★★★★★★★★★★★ 过程 1、密钥的预处理 ★★★★★★★★★★★★

```
6A 12           PUSH 12                             ; /Arg2 = 00000012    压入密钥长度 12
68 00304000     PUSH BFKEYGEN.00403000              ; |Arg1 = 00403000    压入密钥地址
E8 50FEFFFF     CALL BFKEYGEN.004010C8              ; \\BFKEYGEN.004010C8  密钥预处理
```

跟进 `CALL BFKEYGEN.004010C8`

```
004010C8  /$  55               PUSH EBP
004010C9  |.  8BEC             MOV EBP,ESP
004010CB  |.  83C4 F4          ADD ESP,-0C
004010CE  |.  53               PUSH EBX
004010CF  |.  57               PUSH EDI
004010D0  |.  56               PUSH ESI
004010D1  |.  52               PUSH EDX
004010D2  |.  51               PUSH ECX
004010D3  |.  BE 3D304000      MOV ESI,BFKEYGEN.0040303D                 ;  KEY_PBOX 地址(PBOX 处理后放到这里)
004010D8  |.  B8 1D414000      MOV EAX,BFKEYGEN.0040411D                 ;  SBOX 地址
004010DD  |.  8D4E 48          LEA ECX,DWORD PTR DS:[ESI+48]            ;  KEY_SBOX 地址(SBOX 处理后放到这里)
004010E0  |>  BA 00010000      /MOV EDX,100                              ;  开始循环
004010E5  |>  8B38             |/MOV EDI,DWORD PTR DS:[EAX]             ;  取 SBOX 内容
004010E7  |.  83C0 04          ||ADD EAX,4
004010EA  |.  8939             ||MOV DWORD PTR DS:[ECX],EDI             ;  填充到 KEY_SBOX
004010EC  |.  83C1 04          ||ADD ECX,4
004010EF  |.  4A               ||DEC EDX
004010F0  |.^ 75 F3           |\JNZ SHORT BFKEYGEN.004010E5
004010F2  |.  3D 1D514000     |CMP EAX,BFKEYGEN.0040511D               ;  判断有没有填充完毕
004010F7  |.^ 7C E7            \\JL SHORT BFKEYGEN.004010E0             ;  循环出口
```

以上循环结束了，这个循环就是 `SBOX` 的 1024 个元素搬新家，搬到 `KEY_SBOX`

```
004010F9  |.  8B55 08          MOV EDX,DWORD PTR SS:[EBP+8]             ;  取用户密钥 KEY 的地址
004010FC  |.  BF D5404000      MOV EDI,BFKEYGEN.004040D5
00401101  |.  33C0             XOR EAX,EAX
00401103  |.  2BFE             SUB EDI,ESI
00401105  |.  C745 FC 1200000>MOV DWORD PTR SS:[EBP-4],12             ;  置循环次数为 18(PBOX 内有 18 个元素)
0040110C  |>  33C9             /XOR ECX,ECX                              ;  进入循环
0040110E  |.  C745 F8 0400000>|MOV DWORD PTR SS:[EBP-8],4
00401115  |>  33DB             |/XOR EBX,EBX
00401117  |.  8A1C02           ||MOV BL,BYTE PTR DS:[EDX+EAX]
0040111A  |.  C1E1 08          ||SHL ECX,8
0040111D  |.  0BCB             ||OR ECX,EBX
0040111F  |.  40               ||INC EAX
00401120  |.  3B45 0C          ||CMP EAX,DWORD PTR SS:[EBP+C]
00401123  |.  7C 02            ||JL SHORT BFKEYGEN.00401127
00401125  |.  33C0             ||XOR EAX,EAX
00401127  |>  8B5D F8          ||MOV EBX,DWORD PTR SS:[EBP-8]
0040112A  |.  4B               ||DEC EBX
0040112B  |.  895D F8          ||MOV DWORD PTR SS:[EBP-8],EBX
0040112E  |.^ 75 E5           |\JNZ SHORT BFKEYGEN.00401115           ;  这里嵌套了个小循环，作用是用 KEY 中依次取 4 个字符的 ASC,如果 KEY 取完，就从 KEY 的头个字母继续取
00401130  |.  8B1C3E          |MOV EBX,DWORD PTR DS:[ESI+EDI]
00401133  |.  83C6 04          |ADD ESI,4
00401136  |.  33D9             |XOR EBX,ECX                           ;  用取出的 4 个字符的 ASC XOR PBOX
00401138  |.  8B4D FC          |MOV ECX,DWORD PTR SS:[EBP-4]
0040113B  |.  895E FC          |MOV DWORD PTR DS:[ESI-4],EBX          ;  用异或结果填充 KEY_PBOX
0040113E  |.  49               |DEC ECX                                ;  计数器减 1
0040113F  |.  894D FC          |MOV DWORD PTR SS:[EBP-4],ECX
00401142  |.^ 75 C8           \\JNZ SHORT BFKEYGEN.0040110C          ;  循环出口
```

上面循环作用是：依次取 `KEY` 的字符的 `ASC` 异或 `PBOX`，`KEY` 可以循环使用，结果保存在 `KEY_PBOX` 中。

接下来要使用 `BlowFish` 的核心函数 `BF_EN` 来处理 `KEY_PBOX` 了，这个 `BF_EN` 函数就是后来用来加密输入信息的。被加密的 64 位信息被分成 32 位的两部分，`XL` 和 `XR`

```
PUSH BFKEYGEN.004040CD              ; /Arg2 = 004040CD    XR 的地址
PUSH BFKEYGEN.004040D1              ; |Arg1 = 004040D1    XL 的地址
CALL BFKEYGEN.0040106C              ; \\BFKEYGEN.0040106C  BlowFish 的核心函数 BF_EN 加密函数
```

输出信息保存在 `[4040C5]` 和 `[4040D1]` 两个地址中。

```
00401144  |.  BB 3D304000      MOV EBX,BFKEYGEN.0040303D                 ;  KEY_PBOX 地址
00401149  |.  33C0             XOR EAX,EAX
0040114B  |.  A3 C5404000      MOV DWORD PTR DS:[4040C5],EAX           ;  [4040C5]保存了 XR，先置 0
00401150  |.  A3 C9404000      MOV DWORD PTR DS:[4040C9],EAX           ;  [4040C9]保存了 XL，先置 0
00401155  |.  8BF3             MOV ESI,EBX
00401157  |.  BF 09000000      MOV EDI,9                               ;  循环次数置 9
0040115C  |>  8D05 C5404000   /LEA EAX,DWORD PTR DS:[4040C5]          ;  进入循环
00401162  |.  8D0D C9404000   |LEA ECX,DWORD PTR DS:[4040C9]
00401168  |.  50               |PUSH EAX                                ; /Arg2 => 004040C5
00401169  |.  51               |PUSH ECX                                ; |Arg1 => 004040C9
0040116A  |.  E8 FDFEFFFF      |CALL BFKEYGEN.0040106C                ; \\核心加密函数:BF_EN
0040116F  |.  A1 C9404000      |MOV EAX,DWORD PTR DS:[4040C9]
00401174  |.  8B0D C5404000    |MOV ECX,DWORD PTR DS:[4040C5]           ;  取出 BF_EN 结果
0040117A  |.  8906             |MOV DWORD PTR DS:[ESI],EAX
0040117C  |.  894E 04          |MOV DWORD PTR DS:[ESI+4],ECX            ;  替换 KEY_PBOX 的数据
0040117F  |.  83C6 08          |ADD ESI,8
00401182  |.  4F               |DEC EDI
00401183  |.^ 75 D7           \\JNZ SHORT BFKEYGEN.0040115C           ;  循环出口
```

这个循环大致是这样的：

```
BF_EN (0x0000000000000000)    结果填入 KEY_BOX[1]KEY_BOX[2]
BF_EN(KEY_BOX[1]KEY_BOX[2])   结果填入 KEY_BOX[3]KEY_BOX[4]
BF_EN(KEY_BOX[3]KEY_BOX[4])   结果填入 KEY_BOX[5]KEY_BOX[6]
……………………
BF_EN(KEY_BOX[15]KEY_BOX[16]) 结果填入 KEY_BOX[17]KEY_BOX[18]
```
