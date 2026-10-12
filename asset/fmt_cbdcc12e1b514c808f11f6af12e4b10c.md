
# AspApp golden3.0(中文名称:asp 加密锁 3.0 企业版)加密分析

> 原文：[`www.pediy.com/kssd/pediy10/62544.html`](https://www.pediy.com/kssd/pediy10/62544.html)

**软件名称：**`AspApp golden3.0`(中文名称:asp 加密锁 3.0 企业版)（想传附件可是权限不够，软件可以自己下载）

**保护方式：**加壳+反跟踪（可能还有别的保护方式）

**加壳方式：**`ASPack 2.12`

**软件简介：**将 `IIS` + `ASP`(Active Server Page)实现的网站进行加密、压缩，加密后的应用仍然可以在 `IIS` 下使用，但是他人已经无法直接阅读 `ASP` 脚本内容，彻底保护您的辛苦劳动成果和知识产权。

**加密后的代码样式如下：**

```asp
<%@ Language="VBSCRIPT" %>

<% 'AspApp Golden - Enterprise (V3.0.0.0)
  On Error Resume Next
  Set MyAspAppG = Server.CreateObject("AspAppGX.AspDecoder")
  If Not IsObject(MyAspAppG) Then
    Response.Write "请先运行 Setup\_Ent.exe 并注册组件！"   (setup\_ent.exe 等同于 regsvr32.exe aspappG.dll) 
    Response.End
  End If
  MyAspAppG.Tag = "ZY3BywXyuYlp=|X1o"
  MyAspAppG.Crc64 = "415r1a|mED;\_\*AH1HO|@Ppp"
  MyAspAppG.Crc32 = "S8ZnhxVfWCV9/B/Y7N4KFClA3fHjAk2dJMBIzwcyHzk="
  MyAspAppG.AddCode "jvPFkUtIhuV11sFGCHGKm/+TdNnAqPe3hlINvF3XVuR9TgKHmBxZTFt/JwJJA6gdzXLtgjfj1m4tUoil"
  MyAspAppG.AddCode "L/zmqixl9I5+Yo/J61C18UEoyBPKLxlFxtYqqvfPQ7c16oe8Gh5ENl6gF5Snz1IvK72zN4/K8EJyrYsn"
  MyAspAppG.AddCode "LQZ+Y0omo3aWUxeqII3+17mycWbajZLKaejTRQO+u9GBnI2emn9UkWCh9nP8G/roDtwsIcbsFLfFKahC"
  MyAspAppG.AddCode "fesyREg2R+cXYfaDwzFZNcjkwIU1gASGTmEk4fyMVzMdlyhZ8/jFWgEfpvwbdcrknMtezxX7zTfGiqtM"
  MyAspAppG.AddCode "JCiaWalJcLn4Of6Dudkn21gWedShsQXkh4AS/Mau9VLsU48ZYWcnjBgbWDlPGpOEfdHR466Qs6BbN3Vx"
  MyAspAppG.AddCode "0s8r8RJX7+yznlqCaJpvFglgNHedXLW82mFgcfNQ/7lUb2D1V4POgd8xwvdnIy8qx2/EwWj3LdCm5LST"
  MyAspAppG.AddCode "G/LiX8VfJrihCAZkyV0JbEgE8Y5qdlZK36wPZglLKIZaVOh19U5XnQeSCGKKnMnP/dK5EE8oOOgxD5U5"
  MyAspAppG.AddCode "1/ka+h4Q207ED3mx9vYQzr1eTSRa1uSStKxXt0crMdJ6Rksj8VrBO7WAiU5eGN6wK/9Bqg=="
  MyAspAppG.Execute
  Set MyAspAppG = nothing
%>
```

经过测试：同一 `asp` 源代码，加密后 `MyAspAppG.Crc32` 后面的数据是一样的，其它三项：`MyAspAppG.Tag`、`MyAspAppG.Crc64`、`MyAspAppG.RegCode` 数据每次都不一样。

加密后的代码要在 `IIS` 下使用，必须注册一个组件：`aspappG.dll`（`setup\_ent.exe` 等同于 `regsvr32.exe aspappG.dll`）。

**目的：**找出其加密算法（据软件主页说是采用高强度算法进行加密的）。

**声明：**我是一只菜得不能再菜的菜鸟，没有汇编基础，编程语言只学过 `QBASIC` 皮毛。在分析过程中，找不到高手请教，全凭对软件加解密的一腔热情，以下是我一个月以来对这个软件分析过程写的笔记，如有不对之处请高手指教，也许 `aspappG.dll` 里有解密的代码，但是对 `DLL` 文件不知道要怎么分析，只好做罢。

![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

## 分析过程

带壳情况下用 `OD` 载入，跟进就退出，说明软件有反动态跟踪，可以用 `OD` 的一个插件轻松对付（插件叫什么名字，忘记了）。

首先想到对 `AspAppEncoder.exe` 进行脱壳，用 `PEID` 查是用 `ASPack 2.12` -> `Alexey Solodovnikov` 加的壳，脱壳后一运行就退出，认为是文件有 `CRC` 校验，用 `OD` 载入脱壳后的文件，按 `F8` 键一步一步的跟进到 `005016B5` 处，如下：

```asm
005016B5  |.  E8 3292F0FF    call    0040A8EC               ;  取文件大小
005016BA  |.  81FE D8FC0800 cmp     esi, 8FCD8               ;  此时 ESI 值班为 001B1600 即十进制的：1775104 刚好是文件的大小，和 8FCD8 比较，
005016C0  |.  7E 05         jle     short 005016C7           ;  因为脱壳后的文件比较大，所以这个跳转不会实现
005016C2      E8 3D34F0FF   call    00404B04。
```

退出程序。

将上面的 `8FCD8` 的值改大一点，这里就会可以过去。

再跟进到 `00501702` 处：

```asm
00501702  |.  84C0          test    al, al                   ;这里的 al=00，然后跳转不会实现，下一步和上面一样：CALL 到 00404B04 处运行，退出程序，我和加壳的源程序对比了一下，加壳的源程序运行到此处 al=01，（在信息窗口可以看到）我不知道这里怎么才能让 al=01,
00501704  |.  75 05         jnz     short 0050170B           ;只能改这里，将 jnz 改成 jmp,无条件转移。
00501706  |.  E8 F933F0FF   call    00404B04
```

改后，继续跟进，到此处。

```asm
00501733  |.  E8 6C3CF6FF   call    004653A4                 ;这个 call 就会打开那个 NAG 窗口，不知道为什么，继续跟进到终止运行，窗口里面的字啊什么的全部都没有，我认为：根据壳的原理，程序完全运行后，代码是还原的真的代码，因为我的目的是待程序完全载入后，点开始加密找出加密的方式，所有就没有对脱壳进行下去。(脱壳失败)
```

以下分析全部是用 `OD` 带壳分析的过程。

用 `OD` 载入源程序，按 `F9` 直接让程序运行，程序完全载入后，代码停在如下位置：

```asm
0050164B    56             push    esi
0050164C    57             push    edi
0050164D    33C0           xor     eax, eax
0050164F    8945 E8        mov     dword ptr [ebp-18], eax
00501652    8945 EC        mov     dword ptr [ebp-14], eax
00501655    B8 94115000    mov     eax, 00501194
0050165A    E8 C15AF0FF    call    00407120
0050165F    8B3D B0C15100  mov     edi, dword ptr [51C1B0]     ; AspAppEn.0051FE58
00501665    33C0           xor     eax, eax
00501667    55             push    ebp
00501668    68 53175000    push    00501753
0050166D    64:FF30        push    dword ptr fs:[eax]
00501670    64:8920        mov     dword ptr fs:[eax], esp
00501673    8B07           mov     eax, dword ptr [edi]
00501675    E8 923CF6FF    call    0046530C
0050167A    E8 5DF7FFFF    call    00500DDC
0050167F    84C0           test    al, al
00501681    74 07          je      short 0050168A
00501683    8B07           mov     eax, dword ptr [edi]
00501685    E8 063EF6FF    call    00465490
0050168A    8D55 EC        lea     edx, dword ptr [ebp-14]
0050168D    8B07           mov     eax, dword ptr [edi]
0050168F    E8 E842F6FF    call    0046597C
00501694    8B45 EC        mov     eax, dword ptr [ebp-14]
00501697    BA 40000000    mov     edx, 40
0050169C    E8 2791F0FF    call    0040A7C8
005016A1    8BD8           mov     ebx, eax
005016A3    B9 02000000    mov     ecx, 2
```

在 `OD` 中右键查找所有参考文件字符，找到：

地址=`004FEBC5`，反汇编=`mov eax, 004FEFC8`，文本字串=`ASCII "hh:nn:ss"" 开始加密..."""`

双击到反汇编窗口，如下：

```asm
004FEBC5    B8 C8EF4F00     mov     eax, 004FEFC8                    ; ASCII "hh:nn:ss"" 开始加密..."""
004FEBCA    E8 0DE4F0FF     call    0040CFDC
004FEBCF    8B4D D8         mov     ecx, dword ptr [ebp-28]
004FEBD2    B2 01          mov      dl, 1
004FEBD4    8B45 F0         mov     eax, dword ptr [ebp-10]
004FEBD7    E8 84C9FFFF     call    004FB560
004FEBDC    A1 B0C15100     mov     eax, dword ptr [51C1B0]
004FEBE1    8B00           mov      eax, dword ptr [eax]
004FEBE3    E8 1066F6FF    call    004651F8
```

在 `004FEBC5` 行按 `F2` 下断点，然后切换到 `aspapp` 程序（以下称源程序）窗口，点击开始加密，自动跳回 `OD` 窗口，并且停在我下断点的位置，按 `F8` 跟进，在不停重新载入程序，下断点，删除断点的过程中，最后一次我在 `004FD469` 处下断点，程序到了如下位置：

```asm
004FD469    E8 EEA1F8FF     call    0048765C                          ; 得到 CRC32 每次一样
004FD46E    8B95 94FDFFFF   mov     edx, dword ptr [ebp-26C]
004FD474    8B45 F4         mov     eax, dword ptr [ebp-C]
004FD477    E8 7078F0FF     call    00404CEC
004FD47C    33C0            xor     eax, eax
004FD47E    5A              pop     edx
004FD47F    59              pop     ecx
004FD480    59              pop     ecx
004FD481    64:8910         mov     dword ptr fs:[eax], edx
```

按一下 `F8` 键到：

```asm
004FD46E    8B95 94FDFFFF   mov     edx, dword ptr [ebp-26C]
```

此时，信息窗口显示如下：

堆栈 `ss:[0012F804]=00FD4668`, `(ASCII "S8ZnhxVfWCV9/B/Y7N4KFClA3fHjAk2dJMBIzwcyHzk=")` 记住这个 `SS=00FD4668` 这个值。

`edx=00000000`

`S8ZnhxVfWCV9/B/Y7N4KFClA3fHjAk2dJMBIzwcyHzk=` 就是密文里面 `MyAspAppG.Crc32 =` 后面的数据。

再往下跟进（`F8`）到这里：

```asm
004FD49E    E8 9160F0FF     call    00403534
004FD4A3    8BF0            mov     esi, eax
004FD4A5    8D8D DCFDFFFF   lea     ecx, dword ptr [ebp-224]
004FD4AB    8B45 14         mov     eax, dword ptr [ebp+14]
004FD4AE    8B40 F0         mov     eax, dword ptr [eax-10]
004FD4B1    BA 10000000     mov     edx, 10
```

按两下 `F8` 跟进到：

```asm
004FD4A5    8D8D DCFDFFFF   lea     ecx, dword ptr [ebp-224]
```

此时，信息窗口显示如下：

堆栈地址=`0012F84C`, `(ASCII 14,"wdqT/c7B0p(B?0DecEs6")` 这里是 `MyAspAppG.Crc64`， 但是不全。
