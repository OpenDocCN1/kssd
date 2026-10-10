

# 利用播放器提取 LTC 课件中的音频

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-221.htm`](https://www.pediy.com/kssd/pediy09/pediy09-221.htm)

> 【文章标题】: 利用播放器提取 LTC 课件中的音频  
> 【文章作者】: filly  
> 【作者邮箱】: vaov@163.com  
> 【作者 QQ 号】: 22758718  
> 【下载地址】: 自己搜索下载  
> 【使用工具】: OllyICE1.10&IDA5.0  
> 【作者声明】: 写出来想和大家讨论。失误之处敬请诸位大侠赐教!感谢诸位看雪人，是你们的不懈努力，教会了我许多！  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   利用播放器提取 LTC 课件中的音频  
>      偶 MM 想考法硕，下载了一堆资料。可是大部分都是 LTC 格式，没办法放在 mp4 里边听。于是，决定写一个 LTC 转 MP3 的程序。上网查资料，结果发现 LTC 格式,是 LT 公司自己研发的课件生成系统,不提供解压缩源代码.郁闷了.这么多资料不能浪费阿,怎么办?!只好开始从播放器入手,探索一下压缩原理了.  
>      查壳!没有壳阿!心情轻松很多!真讨厌脱壳,感觉那就是一种耐力的考验!  
>   掏出我们的法宝 OllyICE1.10 上战场!  
>      通过几次试验,知道了程序是 mfc 直接编译的.由于解压缩要读包,在所有"KERNEL32.ReadFile"下断.同时发现播放器还进行写操作,于是又在所  
>   有"MFC42.#6385\_CFile::Write"上下了断.  
>      忽略所有异常,在命令行参数写上课件包的地址,运行!开始几个 ReadFile 断点,是读取皮肤文件,略过!再往后读包了,首先读了 20 个字节  
>     
>   00421472  |.  FF15 A0A14400 call    dword ptr \[<&KERNEL32.ReadFile>\] ; \\ReadFile  
>   堆栈:  
>   000FD448   000002AC  |hFile = 000002AC  
>   000FD44C   000FD49C  |Buffer = 000FD49C  
>   000FD450   00000014  |BytesToRead = 14 (20.)  
>   000FD454   000FD4B0  |pBytesRead = 000FD4B0  
>   000FD458   00000000  \\pOverlapped = NULL  
>     
>     
>   接着又读 130 个字节,  
>   00419D80  |.  FF15 A0A14400 call    dword ptr \[<&KERNEL32.ReadFile>\] ; \\ReadFile  
>   堆栈:  
>   000FD56C   000002AC  |hFile = 000002AC  
>   000FD570   00E3CFE0  |Buffer = 00E3CFE0  
>   000FD574   00000082  |BytesToRead = 82 (130.)  
>   000FD578   000FD5D8  |pBytesRead = 000FD5D8  
>   000FD57C   00000000  \\pOverlapped = NULL  
>     
>   然后就要写?!写得什么呢?打开一瞧,一堆"乱码".可是,通过经验观察,知道这可能是包的全局向量表.整个包的结构应该都在这里边,也就是说 LTC 文件就象个大箱子,里面装着各种打成包了的声音、图片之类的东西,然后通过一个全局向量表来告诉程序需要读的包的位置。后来验证果然是这样.  
>        再往后,就开始读数据了.首先读了一段出来,赶紧瞧瞧读的内容:  
>    50 4B 03 04 14 00 02 00....   
>   这是什么?怎么很眼熟??又想了想刚才用 IDA5.0 查看 sting 表时有这么一段: deflate 1.1.3 Copyright 1995-1998 Jean-loup Gailly 还有   
>   inflate 1.1.3 Copyright 1995-1998 Mark Adler 怎么那么熟悉.上网查查,唉,这就是 ZIP 包嘛,好多年前,破解 zip 密码时见过的阿,我都忘光了![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心") !!  
>        WinHex 打开 LTC 文件,发现到处都是这种包.于是设想,LTC 文件是按时间顺序,把图像和声音打成 ZIP 包,然后逐个解压!  
>        立即编程,搜索 ltc 文件里的所有 zip 包,逐个解压!忙了整整大半天,程序终于出炉了.运行,结果解压出来了一堆图片,和一些类似超链接的符号,  
>   其他的没了?!!!声音呢?我要的声音在哪里?看来,判断错误了![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") .声音不再 zip 包里边。  
>        声音到底在哪里？茫然的打开 OllyICE,突然发现，程序使用了 dsound。难道这是发声的部位？不管那么多先下断.运行，果然在 dsound 断下了  
>   ,并且播放器还没有声音.老办法,顺藤摸瓜，找到了建立 dsound 过程,得位置.  
>     
>   00403180  /$  83EC 10       sub     esp, 10  
>   00403183  |.  56            push    esi  
>   00403184  |.  8BF1          mov     esi, ecx  
>   00403186  |.  57            push    edi  
>   00403187  |.  33FF          xor     edi, edi  
>   00403189  |.  8B46 04       mov     eax, dword ptr \[esi+4\]  
>   0040318C  |.  3BC7          cmp     eax, edi  
>   0040318E  |.  0F84 B6000000 je      0040324A  
>   00403194  |.  8B08          mov     ecx, dword ptr \[eax\]  
>   00403196  |.  57            push    edi  
>   00403197  |.  50            push    eax  
>   00403198  |.  FF51 34       call    dword ptr \[ecx+34\]  
>   0040319B  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]  
>   0040319F  |.  57            push    edi  
>   004031A0  |.  51            push    ecx  
>   004031A1  |.  8D4C24 1C     lea     ecx, dword ptr \[esp+1C\]  
>   004031A5  |.  8B46 04       mov     eax, dword ptr \[esi+4\]  
>   004031A8  |.  51            push    ecx  
>   004031A9  |.  8D4C24 14     lea     ecx, dword ptr \[esp+14\]  
>   004031AD  |.  8B10          mov     edx, dword ptr \[eax\]  
>   004031AF  |.  51            push    ecx  
>   004031B0  |.  8D4C24 1C     lea     ecx, dword ptr \[esp+1C\]  
>   004031B4  |.  51            push    ecx  
>   004031B5  |.  8B4E 0C       mov     ecx, dword ptr \[esi+C\]  
>   004031B8  |.  51            push    ecx  
>   004031B9  |.  57            push    edi  
>   004031BA  |.  50            push    eax  
>   004031BB  |.  FF52 2C       call    dword ptr \[edx+2C\] -------->此处是锁定 dsound 的 buff!  
>   004031BE  |.  8B5424 08     mov     edx, dword ptr \[esp+8\]  
>   004031C2  |.  8B4424 0C     mov     eax, dword ptr \[esp+C\]  
>   004031C6  |.  8B4E 38       mov     ecx, dword ptr \[esi+38\]  
>   004031C9  |.  52            push    edx  
>   004031CA  |.  50            push    eax  
>   004031CB  |.  51            push    ecx  
>   004031CC  |.  FF56 34       call    dword ptr \[esi+34\] -------->那么这里就是我们要找的，读取声音地方！！  
>   004031CF  |.  8B4C24 1C     mov     ecx, dword ptr \[esp+1C\]  
>   004031D3  |.  83C4 0C       add     esp, 0C  
>   004031D6  |.  8B46 04       mov     eax, dword ptr \[esi+4\]  
>   004031D9  |.  51            push    ecx  
>   004031DA  |.  8B4C24 18     mov     ecx, dword ptr \[esp+18\]  
>   004031DE  |.  8B10          mov     edx, dword ptr \[eax\]  
>   004031E0  |.  51            push    ecx  
>   004031E1  |.  8B4C24 10     mov     ecx, dword ptr \[esp+10\]  
>   004031E5  |.  51            push    ecx  
>   004031E6  |.  8B4C24 18     mov     ecx, dword ptr \[esp+18\]  
>   004031EA  |.  51            push    ecx  
>   004031EB  |.  50            push    eax  
>   004031EC  |.  FF52 4C       call    dword ptr \[edx+4C\] -------->此处是解锁 dsound 的 buff!  
>   004031EF  |.  8B46 04       mov     eax, dword ptr \[esi+4\]  
>   004031F2  |.  6A 01         push    1  
>   004031F4  |.  897E 1C       mov     dword ptr \[esi+1C\], edi  
>   004031F7  |.  897E 18       mov     dword ptr \[esi+18\], edi  
>   004031FA  |.  897E 14       mov     dword ptr \[esi+14\], edi  
>   004031FD  |.  897E 10       mov     dword ptr \[esi+10\], edi  
>   00403200  |.  897E 3C       mov     dword ptr \[esi+3C\], edi  
>   00403203  |.  8B10          mov     edx, dword ptr \[eax\]  
>   00403205  |.  57            push    edi  
>   00403206  |.  57            push    edi  
>   00403207  |.  50            push    eax  
>   00403208  |.  FF52 30       call    dword ptr \[edx+30\] --------> 声音发自于此!  
>   0040320B  |.  8B3D D4AA4400 mov     edi, dword ptr \[<&WINMM.timeSetE>;  WINMM.timeSetEvent  
>   00403211  |.  6A 01         push    1  
>   00403213  |.  56            push    esi  
>   00403214  |.  68 80304000   push    00403080  
>   00403219  |.  6A 02         push    2  
>   0040321B  |.  68 E8030000   push    3E8  
>   00403220  |.  FFD7          call    edi                              ;  <&WINMM.timeSetEvent>  
>   00403222  |.  6A 32         push    32  
>   00403224  |.  8946 28       mov     dword ptr \[esi+28\], eax  
>   00403227  |.  FF15 D0AA4400 call    dword ptr \[<&WINMM.timeBeginPeri>;  WINMM.timeBeginPeriod  
>   0040322D  |.  85C0          test    eax, eax  
>   0040322F  |.  74 08         je      short 00403239  
>   00403231  |.  5F            pop     edi  
>   00403232  |.  33C0          xor     eax, eax  
>   00403234  |.  5E            pop     esi  
>   00403235  |.  83C4 10       add     esp, 10  
>   00403238  |.  C3            retn  
>     
>     
>        这是多么标准的一个 dsound 处理过程哦！赶紧进去瞧瞧怎么把声音装进 buff 里边的吧！直奔主题得了，来到这里：  
>     
>      为了节省读者时间,告诉大家从 Call 401AA0 以后都是对文件向量表的对照操作!读者可以略过!  
>     
>   00401800  /$  B8 90BB0000   mov     eax, 0BB90  
>   00401805  |.  E8 76430400   call    00445B80  
>   0040180A  |.  53            push    ebx  
>   0040180B  |.  8BD9          mov     ebx, ecx  
>   0040180D  |.  33C0          xor     eax, eax  
>   0040180F  |.  55            push    ebp  
>   00401810  |.  8B4B 30       mov     ecx, dword ptr \[ebx+30\]  
>   00401813  |.  8B53 38       mov     edx, dword ptr \[ebx+38\]  
>   00401816  |.  49            dec     ecx  
>   00401817  |.  56            push    esi  
>   00401818  |.  3BD1          cmp     edx, ecx  
>   0040181A  |.  57            push    edi  
>   0040181B  |.  894424 14     mov     dword ptr \[esp+14\], eax  
>   0040181F  |.  894424 18     mov     dword ptr \[esp+18\], eax  
>   00401823  |.  894424 10     mov     dword ptr \[esp+10\], eax  
>   00401827  |.  76 0D         jbe     short 00401836  
>   00401829  |.  5F            pop     edi  
>   0040182A  |.  5E            pop     esi  
>   0040182B  |.  5D            pop     ebp  
>   0040182C  |.  5B            pop     ebx  
>   0040182D  |.  81C4 90BB0000 add     esp, 0BB90  
>   00401833  |.  C2 0800       retn    8  
>   00401836  |>  8BAC24 A8BB00>mov     ebp, dword ptr \[esp+BBA8\]  ;旅行从这里开始  
>   0040183D  |.  3BE8          cmp     ebp, eax                   ;准备就绪吗？  
>   0040183F  |.  0F8E A0010000 jle     004019E5                   ;准备好了就开始！  
>   00401845  |>  396B 60       /cmp     dword ptr \[ebx+60\], ebp   ;和向量表比较，要播放下一内容需要读多少个包  
>   00401848  |.  0F87 A9010000 |ja      004019F7  
>   0040184E  |.  8B53 38       |mov     edx, dword ptr \[ebx+38\]   ;ebx 为读取向量表类的指针  
>   00401851  |.  8B43 2C       |mov     eax, dword ptr \[ebx+2C\]   ;定位向量表  
>   00401854  |.  8B3490        |mov     esi, dword ptr \[eax+edx\*4\];确定读取文件的偏移量!  
>   00401857  |.  F7C6 00000080 |test    esi, 80000000  
>   0040185D  |.  0F84 F7010000 |je      00401A5A  
>   00401863  |.  8D7B 40       |lea     edi, dword ptr \[ebx+40\]  
>   00401866  |.  57            |push    edi                             ; /pCriticalSection  
>   00401867  |.  FF15 A4A14400 |call    dword ptr \[<&KERNEL32.EnterCrit>; \\EnterCriticalSection  
>   0040186D  |.  8B4B 24       |mov     ecx, dword ptr \[ebx+24\]  
>   00401870  |.  81F6 00000080 |xor     esi, 80000000  
>   00401876  |.  6A 00         |push    0                               ; /Origin = FILE\_BEGIN  
>   00401878  |.  6A 00         |push    0                               ; |pOffsetHi = NULL  
>   0040187A  |.  56            |push    esi                             ; |OffsetLo  
>   0040187B  |.  51            |push    ecx                             ; |hFile  
>   0040187C  |.  FF15 9CA14400 |call    dword ptr \[<&KERNEL32.SetFilePo>; \\SetFilePointer 找到读文件的位置  
>   00401882  |.  8B53 38       |mov     edx, dword ptr \[ebx+38\]  
>   00401885  |.  8B43 2C       |mov     eax, dword ptr \[ebx+2C\]  
>   00401888  |.  8B4490 04     |mov     eax, dword ptr \[eax+edx\*4+4\]  
>   0040188C  |.  A9 00000080   |test    eax, 80000000  
>   00401891  |.  74 05         |je      short 00401898  
>   00401893  |.  35 00000080   |xor     eax, 80000000  
>   00401898  |>  2BC6          |sub     eax, esi  
>   0040189A  |.  8D4C24 1C     |lea     ecx, dword ptr \[esp+1C\]  
>   0040189E  |.  8BF0          |mov     esi, eax  
>   004018A0  |.  8B43 24       |mov     eax, dword ptr \[ebx+24\]  
>   004018A3  |.  6A 00         |push    0                               ; /pOverlapped = NULL  
>   004018A5  |.  51            |push    ecx                             ; |pBytesRead  
>   004018A6  |.  8D5424 28     |lea     edx, dword ptr \[esp+28\]         ; |  
>   004018AA  |.  56            |push    esi                             ; |BytesToRead  
>   004018AB  |.  52            |push    edx                             ; |Buffer  
>   004018AC  |.  50            |push    eax                             ; |hFile  
>   004018AD  |.  C74424 30 000>|mov     dword ptr \[esp+30\], 0           ; |  
>   004018B5  |.  FF15 A0A14400 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile 读文件就不说啦  
>   004018BB  |.  85C0          |test    eax, eax  
>   004018BD  |.  57            |push    edi                             ; /pCriticalSection  
>   004018BE  |.  74 2D         |je      short 004018ED                  ; |  
>   004018C0  |.  FF15 B0A14400 |call    dword ptr \[<&KERNEL32.LeaveCrit>; \\LeaveCriticalSection  
>   004018C6  |.  8B43 34       |mov     eax, dword ptr \[ebx+34\]  
>   004018C9  |.  8D4C24 18     |lea     ecx, dword ptr \[esp+18\]  
>   004018CD  |.  8D5424 14     |lea     edx, dword ptr \[esp+14\]  
>   004018D1  |.  51            |push    ecx                             ; /Arg4  
>   004018D2  |.  85C0          |test    eax, eax                        ; |  
>   004018D4  |.  52            |push    edx                             ; |Arg3  
>   004018D5  |.  8D4424 28     |lea     eax, dword ptr \[esp+28\]         ; |  
>   004018D9  |.  56            |push    esi                             ; |Arg2  
>   004018DA  |.  50            |push    eax                             ; |Arg1  
>   004018DB  |.  8BCB          |mov     ecx, ebx                        ; |  
>   004018DD  |.  74 07         |je      short 004018E6                  ; |  
>   004018DF  |.  E8 BC010000   |call    00401AA0                        ; \\player.00401AA0  
>   004018E4  |.  EB 0D         |jmp     short 004018F3                  ;上边就是我梦寐以求的地方!!解压缩代码!!  
>   004018E6  |>  E8 05030000   |call    00401BF0  
>   004018EB  |.  EB 06         |jmp     short 004018F3  
>   004018ED  |>  FF15 B0A14400 |call    dword ptr \[<&KERNEL32.LeaveCrit>; \\LeaveCriticalSection  
>   004018F3  |>  8B4424 18     |mov     eax, dword ptr \[esp+18\]  
>   004018F7  |.  3BE8          |cmp     ebp, eax  
>   004018F9  |.  7D 02         |jge     short 004018FD  
>     
>     
>        代码总结:开头就不说了,从 Call 401AA0 以后都是对文件向量表的对照操作!读者可以略过，而 Call 401AA0 里边就是完整的读包，解包代码了。  
>        从开头可以看出程序的运作意图非常明显,可是令我想不到的是,这个软件的作者太天才了,他...他...他...竟然把读取的包放到堆栈里!哭了,我不会,诸位谁会的教我一下,因为这样做对于读写解压代码太方便了!  
>        下边就是我千辛万苦想得到的解压缩代码:  
>        太长了,就给个读取包头的代码:  
>     
>   00401AA0  /$  81EC 24020000 sub     esp, 224  
>   00401AA6  |.  55            push    ebp  
>   00401AA7  |.  56            push    esi  
>   00401AA8  |.  57            push    edi  
>   00401AA9  |.  33F6          xor     esi, esi  
>   00401AAB  |.  894C24 18     mov     dword ptr \[esp+18\], ecx  
>   00401AAF  |.  68 80BB0000   push    0BB80  
>   00401AB4  |.  897424 14     mov     dword ptr \[esp+14\], esi  
>   00401AB8  |.  E8 0D390400   call    <jmp.&MFC42.#823\_operator new>  
>   00401ABD  |.  8B8C24 400200>mov     ecx, dword ptr \[esp+240\]  
>   00401AC4  |.  8BAC24 380200>mov     ebp, dword ptr \[esp+238\]  
>   00401ACB  |.  8D7C24 24     lea     edi, dword ptr \[esp+24\]  
>   00401ACF  |.  83C4 04       add     esp, 4  
>   00401AD2  |.  8901          mov     dword ptr \[ecx\], eax  
>   00401AD4  |.  B9 0C000000   mov     ecx, 0C  
>   00401AD9  |.  33C0          xor     eax, eax  
>   00401ADB  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
>   00401ADD  |.  39B424 380200>cmp     dword ptr \[esp+238\], esi  
>   00401AE4  |.  0F8E E6000000 jle     00401BD0  
>   00401AEA  |.  53            push    ebx  
>   00401AEB  |>  8B45 00       /mov     eax, dword ptr \[ebp\]  
>   00401AEE  |.  8B8C24 3C0200>|mov     ecx, dword ptr \[esp+23C\]  
>   00401AF5  |.  8B5424 14     |mov     edx, dword ptr \[esp+14\]  
>   00401AF9  |.  8B5D 04       |mov     ebx, dword ptr \[ebp+4\]  
>   00401AFC  |.  83E9 08       |sub     ecx, 8  
>   00401AFF  |.  83C5 08       |add     ebp, 8  
>   00401B02  |.  03D0          |add     edx, eax  
>   00401B04  |.  898C24 3C0200>|mov     dword ptr \[esp+23C\], ecx  
>   00401B0B  |.  85C0          |test    eax, eax  
>   00401B0D  |.  895424 14     |mov     dword ptr \[esp+14\], edx  
>   00401B11  |.  0F86 8B000000 |jbe     00401BA2  
>   00401B17  |.  8D0C76        |lea     ecx, dword ptr \[esi+esi\*2\]  
>   00401B1A  |.  894424 18     |mov     dword ptr \[esp+18\], eax  
>   00401B1E  |.  8D0C89        |lea     ecx, dword ptr \[ecx+ecx\*4\]  
>   00401B21  |.  C1E1 05       |shl     ecx, 5  
>   00401B24  |.  03F0          |add     esi, eax  
>   00401B26  |.  894C24 10     |mov     dword ptr \[esp+10\], ecx  
>   00401B2A  |.  897424 20     |mov     dword ptr \[esp+20\], esi  
>   00401B2E  |>  8BCB          |/mov     ecx, ebx  
>   00401B30  |.  8BF5          ||mov     esi, ebp  
>   00401B32  |.  8BD1          ||mov     edx, ecx  
>   00401B34  |.  8D7C24 24     ||lea     edi, dword ptr \[esp+24\]  
>   00401B38  |.  C1E9 02       ||shr     ecx, 2  
>   00401B3B  |.  F3:A5         ||rep     movs dword ptr es:\[edi\], dword>  
>   00401B3D  |.  8BCA          ||mov     ecx, edx  
>   00401B3F  |.  8B5424 1C     ||mov     edx, dword ptr \[esp+1C\]  
>   00401B43  |.  83E1 03       ||and     ecx, 3  
>   00401B46  |.  8D4424 24     ||lea     eax, dword ptr \[esp+24\]  
>   00401B4A  |.  F3:A4         ||rep     movs byte ptr es:\[edi\], byte p>  
>   00401B4C  |.  50            ||push    eax  
>   00401B4D  |.  8B42 1C       ||mov     eax, dword ptr \[edx+1C\] ---->这里是我终生难忘的地方 ebx 竟然是主函数的 this 指针！  
>   00401B50  |.  8D4C24 58     ||lea     ecx, dword ptr \[esp+58\]  
>   00401B54  |.  51            ||push    ecx  
>   00401B55  |.  50            ||push    eax  
>   00401B56  |.  E8 C5140400   ||call    00443020  
>   00401B5B  |.  8B8C24 4C0200>||mov     ecx, dword ptr \[esp+24C\]  
>   00401B62  |.  8B4424 1C     ||mov     eax, dword ptr \[esp+1C\]  
>   00401B66  |.  8B9424 480200>||mov     edx, dword ptr \[esp+248\]  
>   00401B6D  |.  83C4 0C       ||add     esp, 0C  
>   00401B70  |.  8B39          ||mov     edi, dword ptr \[ecx\]  
>   00401B72  |.  2BD3          ||sub     edx, ebx  
>   00401B74  |.  03F8          ||add     edi, eax  
>   00401B76  |.  05 E0010000   ||add     eax, 1E0  
>   00401B7B  |.  894424 10     ||mov     dword ptr \[esp+10\], eax  
>   00401B7F  |.  8B4424 18     ||mov     eax, dword ptr \[esp+18\]  
>   00401B83  |.  03EB          ||add     ebp, ebx  
>   00401B85  |.  B9 78000000   ||mov     ecx, 78  
>   00401B8A  |.  8D7424 54     ||lea     esi, dword ptr \[esp+54\]  
>   00401B8E  |.  48            ||dec     eax  
>   00401B8F  |.  F3:A5         ||rep     movs dword ptr es:\[edi\], dword>  
>   00401B91  |.  899424 3C0200>||mov     dword ptr \[esp+23C\], edx  
>   00401B98  |.  894424 18     ||mov     dword ptr \[esp+18\], eax  
>   00401B9C  |.^ 75 90         |\\jnz     short 00401B2E  
>   00401B9E  |.  8B7424 20     |mov     esi, dword ptr \[esp+20\]  
>   00401BA2  |>  8B8424 3C0200>|mov     eax, dword ptr \[esp+23C\]  
>   00401BA9  |.  85C0          |test    eax, eax  
>   00401BAB  |.^ 0F8F 3AFFFFFF \\jg      00401AEB  
>   00401BB1  |.  8B9424 440200>mov     edx, dword ptr \[esp+244\]  
>   00401BB8  |.  5B            pop     ebx  
>   00401BB9  |.  5F            pop     edi  
>   00401BBA  |.  5E            pop     esi  
>   00401BBB  |.  C702 80BB0000 mov     dword ptr \[edx\], 0BB80  
>   00401BC1  |.  B8 01000000   mov     eax, 1  
>   00401BC6  |.  5D            pop     ebp  
>   00401BC7  |.  81C4 24020000 add     esp, 224  
>   00401BCD  |.  C2 1000       retn    10  
>   00401BD0  |>  8B8424 400200>mov     eax, dword ptr \[esp+240\]  
>   00401BD7  |.  5F            pop     edi  
>   00401BD8  |.  5E            pop     esi  
>   00401BD9  |.  5D            pop     ebp  
>   00401BDA  |.  C700 80BB0000 mov     dword ptr \[eax\], 0BB80  
>   00401BE0  |.  B8 01000000   mov     eax, 1  
>   00401BE5  |.  81C4 24020000 add     esp, 224  
>   00401BEB  \\.  C2 1000       retn    10  
>     
>         作者天才,代码没有废话,佩服之极!  
>         感到长了么,我一句一句反复的看了几十遍了.正因为如此,我的第二次失败到来了!我以为 Call 401AA0 是一段独立的解压缩代码,这样我可以  
>   编程,自己读文件然后直接硬 Call 进去,我是这样实现的:  
>   //声明  
>   int (\*func)( BYTE \*dest, unsigned long destLen, const BYTE \*source, unsigned long sourceLen);  
>     
>   void Mydeflate()  
>   {  
>    FILE \*FileIn = fopen("xxxxxx", "rb");  
>    FILE \*FileOut = fopen(outFile, "wb");  
>   //读入的文件缓冲  
>    fseek(FileIn, 0, SEEK\_END);  
>    unsigned long FileInSize = ftell(FileIn);  
>    void \*InDataBuff = malloc(FileInSize);  
>     
>   //读文件  
>    fseek(FileIn, 0, SEEK\_SET);  
>    fread(InDataBuff,FileInSize,1,FileIn);  
>     
>   //建立输出缓冲  
>    void \*OutDataBuff = NULL;  
>    unsigned long OutBuffSize;  
>     
>   //把播放器程序当成 DLL,然后 loadlibrary  
>    HINSTANCE mod;  
>    int offset=6816;  
>    int myvalue;  
>    mod = LoadLibrary( "player.exe" );  
>     
>   //计算函数偏移量,然后硬 CALL  
>    void \*fun = (void \*)((char\*)mod + offset);  
>    \_\_asm {  
>     mov eax,fun  
>     mov \[func\],eax    
>    }  
>    myvalue = func((BYTE\*)InDataBuff,FileInSize,(const BYTE\*)OutDataBuff,OutBuffSize);  
>     
>   //输出  
>   //fwrite(OutDataBuff,OutBuffSize,1,FileOut);  
>   }  
>     
>         虽然,这段程序执行没有问题.可是,当我调试后,晕了.......  
>         程序作者,太天才了,是不是早就料到有一天有人会像我这样苯,敢于硬 Call 他的宝贝代码阿!  
>         他偷偷的带进去了主程序的 this 指针,并且把一个很小的临时缓冲区地址通过 this 指针带进去了!我晕倒在地,服了作者了,解压缩个文件,至于动用主函数的 this 指针嘛！！  
>         最后，不得不使出我们破解人的杀手锏了，直接修改它的代码得了！  
>         老步骤：增加新节，修改函数 401AA0 最后的 retn,使它跳到我的代码，把声音缓冲区写到一个文件里保存。然后，修改查包向量表代码，使其连续查表、读、，解压缩。这样终于搞到了没有压缩的音频，懒于再写转 MP3 的代码，直接用个叫 Cool edit 的软件，压制成 MP3!完成任务！  
>   下面是写文件的代码：  
>   004BB000    55              push    ebp                     ;保护现场  
>   004BB001    56              push    esi                                              
>   004BB002    52              push    edx  
>   004BB003    53              push    ebx                       ;保护现场  
>   004BB004    8BEC            mov     ebp, esp  
>   004BB006    81EC 74020000   sub     esp, 274  
>   004BB00C    68 10B14B00     push    004BB110     ; ASCII "ab"  
>   004BB011    68 00B14B00     push    004BB100     ; SCII "c:\\temp\\ltc.pcm"  
>   004BB016    FF15 48A94400   call    dword ptr \[<&MSVCRT.fopen>\]      ; msvcrt.fopen  
>   004BB01C    85C0            test    eax, eax  
>   004BB01E  ^ 74 32           je      short 004BB083  
>   004BB020    8BF0            mov     esi, eax  
>   004BB022    6A 02           push    2  
>   004BB024    6A 00           push    0  
>   004BB026    56              push    esi  
>   004BB027    FF15 44A94400   call    dword ptr \[<&MSVCRT.fseek>\]      ; msvcrt.fseek  
>   004BB02D    85C0            test    eax, eax  
>   004BB02F  ^ 75 25           jnz     short 004BB083  
>   004BB031    8B9C24 C0020000 mov     ebx, dword ptr \[esp+2C0\]  
>   004BB038    8B9424 C4020000 mov     edx, dword ptr \[esp+2C4\]  
>   004BB03F    81FA 80BB0000   cmp     edx, 0BB80  
>   004BB045    76 1D           jbe     short 004BB064  
>   004BB047    56              push    esi  
>   004BB048    6A 01           push    1  
>   004BB04A    68 80BB0000     push    0BB80  
>   004BB04F    53              push    ebx  
>   004BB050    FF15 4CA94400   call    dword ptr \[<&MSVCRT.fwrite>\]     ; msvcrt.fwrite  
>   004BB056    81C3 80BB0000   add     ebx, 0BB80  
>   004BB05C    81EA 80BB0000   sub     edx, 0BB80  
>   004BB062  ^ EB DB           jmp     short 004BB03F  
>   004BB064    56              push    esi  
>   004BB065    6A 01           push    1  
>   004BB067    52              push    edx  
>   004BB068    53              push    ebx  
>   004BB069    FF15 4CA94400   call    dword ptr \[<&MSVCRT.fwrite>\]     ; msvcrt.fwrite  
>   004BB06F    56              push    esi  
>   004BB070    FF15 CCA84400   call    dword ptr \[<&MSVCRT.fclose>\]     ; msvcrt.fclose  
>   004BB076    33C9            xor     ecx, ecx  
>   004BB078    B8 01000000     mov     eax, 1  
>   004BB07D    8BE5            mov     esp, ebp  
>   004BB07F    5B              pop     ebx  
>   004BB080    5A              pop     edx  
>   004BB081    5E              pop     esi  
>   004BB082    5D              pop     ebp  
>   004BB083    C2 1000         retn    10  
>     
>         写这小段代码时候，有几个心得  
>        1.在 Mfc 中 call 外部函数要通过\_IMG\_。\_IMG\_ 里是函数地址，所以要用 FF15 的 Call,就是 Call \[xxxx\];  
>        2.fwrite 一次最大能写 48000(BB80h)个字节,一定要判断要写内容的大小是否超过 48000  
>     
> \--------------------------------------------------------------------------------  
> 【经验总结】  
>   用到的都是普通的技术。在使用硬 Call 时，也可以在执行文件里构建输出表，导出要 call 得函数。不过，觉得很麻烦，也不知道比我的方法稳定多少？请赐教。  
>   还有就是，怎么样才能把文件直接读到堆栈里？虽然反复看了上边的代码可还是不清楚这是怎么实现的。我的想法是他在主函数那声明了一个大的全局数组，作为缓冲空间。而这个变量声明后的地址恰恰在堆栈里，所以往这个数组里读数据当然读到堆栈里了。不知道这个想法对不对？  
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 08 月 05 日 10:26:31 

> 01 年就来看雪了.那时,我才读高二,之前自学了 c++后来 vc++.很喜欢编程,也很喜欢看别人的代码.那时侯,是 64k 猫上网,下个 winice4.05 要一个钟头![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心").为了读别人的代码,我学习破解,所以来到了看雪.发现这里的高手多多,而自己经验少,所以弄出来东西了,感觉也不好,也就没写什么留下.后来,上大学,专业与计算机全无关系.所以破解了东西,也没时间写.现在毕业了,终于有时见了,写点,也能更好的提高自己水平! 

> To filly,  
>   
>  有个地方不明白：  
>   .ACM 不是已经都是压缩格式了吗？难道程序把.ACM 音频存储到 LTC 文件里真的又压缩了一次？看上面的压缩代码也看不明白程序又用什么压缩方式把 ACM 文件压缩了一次。  
>   
>    我的意思是：  
>      要是程序在 LTC 文件里没有压缩.ACM 音频，提取就简单些了，也不用去修改主程序了；  
>   
>     要是程序在 LTC 文件里压缩.了 ACM 音频，那如果我们知道压缩算法，提取也不会很难 － 毕竟你已经跟到了“从 Call 401AA0 以后都是对文件向量表的对照操作!”。  
>   
>   
>   filly 这篇文章的思路与啊 CR 解电子书的思路相似，都需要 DIY 原程序 － 但是 DIY 的时候增加段要写汇编代码，这个就有些难了 － 俺不会写汇编。  
>   
>   如果要实现 LTC 音频文件的提取，  
>   
>   第一种方法：  
> －－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
>     laomms 的做法可能会是（我是说 laomms 的思路）：  
>       直接提取程序里的汇编码，然后在新程序里直接使用。  
>   
>     好处：  
>       不修改人家的原程序。  
>       可以直接利用原程序的解压代码或解密代码 － 可以取巧。  
>     技术要求：太难 －因为原始 CALL 可能不是就一个简单得函数，如果带了很多函数，工作量很大 － 虽然可以取巧地利用原程序的解压代码，但如果解压代码非常复杂，则难度也会变大。  要添加汇编码。  
>   
>   第二种方法：  
> －－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
>     filly 和啊 CR 的做法：  
>       DIY 人家的原程序，增加段，添加自己的或利用人家的输出代码。  
>       
>     坏处：必须修改人家的原程序 － 又得改跳转，又得添加文件输出代码。  
>     好处：自己代码量很少，几乎完全利用程序原来得代码；实在原程序里没有文件输出代码，那么添加得汇编代码总量也就是文件输出代码。  
>   
>   
>   第三种方法：  
> －－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
>    我希望的提取方式：  
>      了解 LTC 文件格式 － 文件头、OFFSET、长度；  
>      根据 LTC 文件格式分离相应的 ZIP 文件；  
>      了解 ACM 音频文件在 LTC 文件里是否压缩，如果压缩了，用的什么压缩算法；  
>      根据 LTC 文件格式提取内部的音频文件。  
>   
>    好处：  
>      真正的逆向！  
>      不修改原程序的任何一个地方 － 原程序只用来被跟踪而已。  
>   
>   看了 filly 的这篇文章，和啊 CR 的 DIY 电子书系列，个人感觉他们两人离熟悉文件格式只差最终一步，但是他们的选择都是直接“取巧” － 就是添加文件输出代码，改变程序走向，迫使程序自己输出自己想要的东西。  
>   不过这样做的一个坏处是：必须修改人家的程序！！我个人觉得这对于“逆向”工作来说不是太好，不知道两位及大家怎么看？？  
>    我们来看 filly 的这篇文章， 既然已经知道“从 Call 401AA0 以后都是对文件向量表的对照操作!”，那么应该就可以知道音频文件的 OFFSET 及长度是怎样计算出来的，但是 filly 一到这里，采取的方式就是“略...”，绕过去了，我觉得很遗憾，因为很明显 filly 会写 C 代码及汇编码，仔细跟 Call 401AA0 肯定也就有可能了解了 LTC 的文件格式，如果了解了 LTC 的文件编排格式，那我们就不必去修改人家的原程序了。  
>   大家要是看了啊 CR 的 DIY 电子书的那篇精华，也会发现啊 CR 已经跟踪到了遍历电子书内部文件的地方，但是啊 CR 也没有继续逆向电子书的内部结构，也是直接取巧 － 加个文件输出函数，改变程序的内部跳转，迫使原程序输出全部文件。  
>   
>   我也参照啊 CR 的方法跟了一下，但是感觉自己汇编码读起来实在吃力。  
>   
>   不过，我在这里写的观点不知道有人同意吗？  
>   我希望使用第三种方法来完成最后的提取，理由就是：只逆向原程序，但绝不修改人家的程序－因为这会带来很多麻烦，特别是 DIY 的同时顺便修改了人家的版权信息，这个俺更反对。  
>   
>   
>   有人愿意就此讨论吗？  
>     
>   filly，有个请求，可不可以上传一个小的 LTC 文件，和一个提取出来的音频文件？还有你跟踪的程序具体是哪个版本？我也想参照你的思路跟踪一下，先谢谢了。 

> 引用: *最初由 **qwert123**发布* ![查看帖子](img/showthread.php?p=342658#post342658) To filly,  
>   
>  有个地方不明白：  
>   .ACM 不是已经都是压缩格式了吗？难道程序把.ACM 音频存储到 LTC 文件里真的又压缩了?...  
> 
>        非常感谢你的讨论,特别是你对这 3 种方法的总结.  
>        首先,从 dsound 缓冲区读出的数据本身就是没有压缩的音频原文,文件名应该写成\*.pcm.当时  
>   
> 我记错了写成.acm 了.不好意思,让你误会了.  
>        LTC 文件里包含了许多东西，就好像是个大箱子，里面装的东西很多，声音图片什么的都放进  
>   
> 去了，然后靠包向量表来告诉程序需要的东西在那个位置。图片和链接符号是 zip 压缩的，声音是 LT  
>   
> 公司自己的压缩方式。CALL 401AA0 里边就是完整的解压代码。  
>        对于压缩解压,有很多实现方式.每个公司都有一套自己的绝活.特别是 LT 公司的这个压缩方式  
>   
> ,用了大量的浮点运算,很先进的,90 多兆的音频硬是压缩到了 4 兆多.我对压缩算法了解很少,所以几次企图逆向都放弃了.  
>        对于第一种方法,可以用我介绍的硬 CALL 的方法.可是,如果 CALL 的函数不独立(比如前面说的  
>   
> 含有主函数的 this 指针)或者需要访问大量外部资源.那么,我们就很难创造条件来实现了.  
>        对于添加新节的问题，就是看雪老大反复强调的，要学好 PE 文件格式了。其实也有专门的工  
>   
> 具，来添加。代码可以用高级语言写，然后编译，再把编译后得结果复制粘贴到新节里就行了。这  
> 里主要的难点在于对修改的目标要有全面地了解，知道在哪里跳最合适，和保护好现场。  
>   
>        还是很关心这个程序的压缩代码,希望高手能给予指点!  
> 程序下载地址: http://www.kaoyansky.cn/thread-135485-1-1.html 

 4dd466d0e1e0cc123afbe504da2462ca