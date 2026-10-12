
# 利用播放器提取 LTC 课件中的音频

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-221.htm`](https://www.pediy.com/kssd/pediy09/pediy09-221.htm)

> 【文章标题】: 利用播放器提取 LTC 课件中的音频

> 【文章作者】: filly

> 【作者邮箱】: vaov@163.com

> 【作者 QQ 号】: 22758718

> 【下载地址】: 自己搜索下载

> 【使用工具】: `OllyICE1.10` & `IDA5.0`

> 【作者声明】: 写出来想和大家讨论。失误之处敬请诸位大侠赐教!感谢诸位看雪人，是你们的不懈努力，教会了我许多！

> --------------------------------------------------------------------------------

> 【详细过程】

> 利用播放器提取 LTC 课件中的音频

> 偶 MM 想考法硕，下载了一堆资料。可是大部分都是 LTC 格式，没办法放在 `mp4` 里边听。于是，决定写一个 LTC 转 MP3 的程序。上网查资料，结果发现 LTC 格式，是 LT 公司自己研发的课件生成系统，不提供解压缩源代码。郁闷了。这么多资料不能浪费阿，怎么办?! 只好开始从播放器入手，探索一下压缩原理了。

> 查壳!没有壳阿!心情轻松很多!真讨厌脱壳，感觉那就是一种耐力的考验!

> 掏出我们的法宝 `OllyICE1.10` 上战场!

> 通过几次试验，知道了程序是 mfc 直接编译的。由于解压缩要读包，在所有"KERNEL32.ReadFile"下断。同时发现播放器还进行写操作，于是又在所有"MFC42.#6385_CFile::Write"上下了断。

> 忽略所有异常，在命令行参数写上课件包的地址，运行!开始几个 `ReadFile` 断点，是读取皮肤文件，略过!再往后读包了，首先读了 20 个字节。

```
00421472  |.  FF15 A0A14400 call    dword ptr [&KERNEL32.ReadFile] ; \ReadFile
堆栈:
000FD448   000002AC  |hFile = 000002AC
000FD44C   000FD49C  |Buffer = 000FD49C
000FD450   00000014  |BytesToRead = 14 (20.)
000FD454   000FD4B0  |pBytesRead = 000FD4B0
000FD458   00000000  \pOverlapped = NULL
```

> 接着又读 130 个字节，

```
00419D80  |.  FF15 A0A14400 call    dword ptr [&KERNEL32.ReadFile] ; \ReadFile
堆栈:
000FD56C   000002AC  |hFile = 000002AC
000FD570   00E3CFE0  |Buffer = 00E3CFE0
000FD574   00000082  |BytesToRead = 82 (130.)
000FD578   000FD5D8  |pBytesRead = 000FD5D8
000FD57C   00000000  \pOverlapped = NULL
```

> 然后就要写?!写得什么呢?打开一瞧，一堆"乱码"。可是，通过经验观察，知道这可能是包的全局向量表。整个包的结构应该都在这里边，也就是说 LTC 文件就象个大箱子，里面装着各种打成包了的声音、图片之类的东西，然后通过一个全局向量表来告诉程序需要读的包的位置。后来验证果然是这样。

> 再往后，就开始读数据了。首先读了一段出来，赶紧瞧瞧读的内容：

> `50 4B 03 04 14 00 02 00....`

> 这是什么？怎么很眼熟??又想了想刚才用 `IDA5.0` 查看 sting 表时有这么一段: `deflate 1.1.3 Copyright 1995-1998 Jean-loup Gailly` 还有 `inflate 1.1.3 Copyright 1995-1998 Mark Adler` 怎么那么熟悉。上网查查，唉，这就是 ZIP 包嘛，好多年前，破解 zip 密码时见过的阿，我都忘光了![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心") !!

> `WinHex` 打开 LTC 文件，发现到处都是这种包。于是设想，LTC 文件是按时间顺序，把图像和声音打成 ZIP 包，然后逐个解压!

> 立即编程，搜索 ltc 文件里的所有 zip 包，逐个解压!忙了整整大半天，程序终于出炉了。运行，结果解压出来了一堆图片，和一些类似超链接的符号，其他的没了?!!!声音呢?我要的声音在哪里?看来，判断错误了![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") 。声音不再 zip 包里边。

> 声音到底在哪里？茫然的打开 `OllyICE`，突然发现，程序使用了 dsound。难道这是发声的部位？不管那么多先下断。运行，果然在 dsound 断下了，并且播放器还没有声音。老办法，顺藤摸瓜，找到了建立 dsound 过程，得位置。

```
00403180  /$  83EC 10      sub     esp, 10
00403183  |.  56         push    esi
00403184  |.  8BF1       mov     esi, ecx
00403186  |.  57         push    edi
00403187  |.  33FF       xor     edi, edi
00403189  |.  8B46 04    mov     eax, dword ptr [esi+4]
0040318C  |.  3BC7       cmp     eax, edi
0040318E  |.  0F84 B6000000 je  0040324A
00403194  |.  8B08       mov     ecx, dword ptr [eax]
00403196  |.  57         push    edi
00403197  |.  50         push    eax
00403198  |.  FF51 34    call    dword ptr [ecx+34]
0040319B  |.  8D4C24 10  lea     ecx, dword ptr [esp+10]
0040319F  |.  57         push    edi
004031A0  |.  51         push    ecx
004031A1  |.  8D4C24 1C  lea     ecx, dword ptr [esp+1C]
004031A5  |.  8B46 04    mov     eax, dword ptr [esi+4]
004031A8  |.  51         push    ecx
004031A9  |.  8D4C24 14  lea     ecx, dword ptr [esp+14]
004031AD  |.  8B10       mov     edx, dword ptr [eax]
004031AF  |.  51         push    ecx
004031B0  |.  8D4C24 1C  lea     ecx, dword ptr [esp+1C]
004031B4  |.  51         push    ecx
004031B5  |.  8B4E 0C    mov     ecx, dword ptr [esi+C]
004031B8  |.  51         push    ecx
004031B9  |.  57         push    edi
004031BA  |.  50         push    eax
004031BB  |.  FF52 2C    call    dword ptr [edx+2C] -------->此处是锁定 dsound 的 buff!
004031BE  |.  8B5424 08  mov     edx, dword ptr [esp+8]
004031C2  |.  8B4424 0C  mov     eax, dword ptr [esp+C]
004031C6  |.  8B4E 38    mov     ecx, dword ptr [esi+38]
004031C9  |.  52         push    edx
004031CA  |.  50         push    eax
004031CB  |.  51         push    ecx
004031CC  |.  FF56 34    call    dword ptr [esi+34] -------->那么这里就是我们要找的，读取声音地方！！
004031CF  |.  8B4C24 1C  mov     ecx, dword ptr [esp+1C]
004031D3  |.  83C4 0C    add     esp, 0C
004031D6  |.  8B46 04    mov     eax, dword ptr [esi+4]
004031D9  |.  51         push    ecx
004031DA  |.  8B4C24 18  mov     ecx, dword ptr [esp+18]
004031DE  |.  8B10       mov     edx, dword ptr [eax]
004031E0  |.  51         push    ecx
004031E1  |.  8B4C24 10  mov     ecx, dword ptr [esp+10]
004031E5  |.  51         push    ecx
004031E6  |.  8B4C24 18  mov     ecx, dword ptr [esp+18]
004031EA  |.  51         push    ecx
004031EB  |.  50         push    eax
004031EC  |.  FF52 4C    call    dword ptr [edx+4C] -------->此处是解锁 dsound 的 buff!
004031EF  |.  8B46 04    mov     eax, dword ptr [esi+4]
004031F2  |.  6A 01      push    1
004031F4  |.  897E 1C    mov     dword ptr [esi+1C], edi
004031F7  |.  897E 18    mov     dword ptr [esi+18], edi
004031FA  |.  897E 14    mov     dword ptr [esi+14], edi
004031FD  |.  897E 10    mov     dword ptr [esi+10], edi
00403200  |.  897E 3C    mov     dword ptr [esi+3C], edi
00403203  |.  8B10       mov     edx, dword ptr [eax]
00403205  |.  57         push    edi
00403206  |.  57         push    edi
00403207  |.  50         push    eax
00403208  |.  FF52 30    call    dword ptr [edx+30] --------> 声音发自于此!
0040320B  |.  8B3D D4AA4400 mov edi, dword ptr [<&WINMM.timeSetE>; WINMM.timeSetEvent
00403211  |.  6A 01      push    1
00403213  |.  56         push    esi
00403214  |.  68 80304000 push  00403080
00403219  |.  6A 02      push    2
0040321B  |.  68 E8030000 push  3E8
00403220  |.  FFD7       call    edi                       ;  <&WINMM.timeSetEvent>
00403222  |.  6A 32      push    32
00403224  |.  8946 28    mov     dword ptr [esi+28], eax
00403227  |.  FF15 D0AA4400 call dword ptr [<&WINMM.timeBeginPeri>; WINMM.timeBeginPeriod
0040322D  |.  85C0       test    eax, eax
0040322F  |.  74 08      je      short 00403239
00403231  |.  5F         pop     edi
00403232  |.  33C0       xor     eax, eax
00403234  |.  5E         pop     esi
00403235  |.  83C4 10    add     esp, 10
00403238  |.  C3         retn
```

> 这是多么标准的一个 dsound 处理过程哦！赶紧进去瞧瞧怎么把声音装进 buff 里边的吧！直奔主题得了，来到这里：

> 为了节省读者时间，告诉大家从 `Call 401AA0` 以后都是对文件向量表的对照操作!读者可以略过!

```
00401800  /$  B8 90BB0000 mov eax, 0BB90
00401805  |.  E8 76430400 call  00445B80
0040180A  |.  53         push  ebx
0040180B  |.  8BD9       mov   ebx, ecx
0040180D  |.  33C0       xor   eax, eax
0040180F  |.  55         push  ebp
00401810  |.  8B4B 30    mov   ecx, dword ptr [ebx+30]
00401813  |.  8B53 38    mov   edx, dword ptr [ebx+38]
00401816  |.  49         dec   ecx
00401817  |.  56         push  esi
00401818  |.  3BD1       cmp   edx, ecx
0040181A  |.  57         push  edi
```
