
# [原创]蜘蛛纸牌也来辅助一下

作者：cyane  
时间：2009/11/18, 0:32  
连接：http://bbs.pediy.com/showthread.php?t=101435

由于工作需要，到了外蒙，到了驻地才发现这里没有网络，连为数不多的电视都是接收的卫星信号，我快崩溃了......

在工作之余，除了领略广袤的大草原的，还时不时的感受着不知从哪个方向刮来的大风，屋外虽算不上冰天雪地，但也是寒冷无比啊，在百无聊赖之际，就有了这篇东西(没什么技术，只是一时兴趣，希望大家不要笑话)。

记得曾在论坛里看到过有人把扫雷改成秒破版的，当时下载下来，挺有意思，想着自己有时间也改个玩玩。这下好了，到了外蒙，除了不缺时间，什么都缺，于是动手改了这个蜘蛛纸牌的游戏

在反编译游戏之前，我先玩了几局，感觉上不是很有意思 ：<

首先OD载入，打开后，来看看它调用的API函数吧，有很多，其中`BitBlt`比较明显，我们首先从它入手吧。可能有人会问为什么要首先在这个函数上设断，玩了这个游戏后你会发现它有很多位图的移动过程，所以免不了要复制图片从一点到另一点，所以我觉得这个函数比较重要，如果这里实在没什么收获，可以再选择嘛，呵呵。

`BP BitBlt`后

F9运行程序来看看

```
01002799  |.  8B46 0C       mov     eax, dword ptr [esi+C]
0100279C  |.  2B46 04       sub     eax, dword ptr [esi+4]
0100279F  |.  8955 E8       mov     dword ptr [ebp-18], edx
010027A2  |.  8B55 10       mov     edx, dword ptr [ebp+10]
010027A5  |.  3BD0            cmp     edx, eax
010027A7  |.  8955 18       mov     dword ptr [ebp+18], edx
010027AA  |.  0F8D BF000000 jge     0100286F
010027B0  |>  8B55 14       /mov     edx, dword ptr [ebp+14]
010027B3  |.  8B5D 18       |mov     ebx, dword ptr [ebp+18]
010027B6  |.  3B5A 0C       |cmp     ebx, dword ptr [edx+C]
010027B9  |.  0F8D B0000000 |jge     0100286F
010027BF  |.  8B55 EC       |mov     edx, dword ptr [ebp-14]
010027C2  |.  8B5D 10       |mov     ebx, dword ptr [ebp+10]
010027C5  |.  03D3            |add     edx, ebx
010027C7  |.  3955 18       |cmp     dword ptr [ebp+18], edx
010027CA  |.  7D 10           |jge     short 010027DC
010027CC  |.  8B55 EC       |mov     edx, dword ptr [ebp-14]
010027CF  |.  6A 40           |push    40
010027D1  |.  58                |pop     eax
010027D2  |.  2BC2            |sub     eax, edx
010027D4  |.  8955 F8       |mov     dword ptr [ebp-8], edx
010027D7  |.  8945 F0       |mov     dword ptr [ebp-10], eax
010027DA  |.  EB 1B           |jmp     short 010027F7
010027DC  |>  8B55 E8      |mov     edx, dword ptr [ebp-18]
010027DF  |.  8365 F0 00   |and     dword ptr [ebp-10], 0
010027E3  |.  2BC2            |sub     eax, edx
010027E5  |.  2B45 10       |sub     eax, dword ptr [ebp+10]
010027E8  |.  8955 F8       |mov     dword ptr [ebp-8], edx
010027EB  |.  3945 18       |cmp     dword ptr [ebp+18], eax
010027EE  |.  74 07           |je      short 010027F7
010027F0  |.  C745 F8 40000>|mov     dword ptr [ebp-8], 40
010027F7  |>  8B5D 0C      |mov     ebx, dword ptr [ebp+C]
010027FA  |.  EB 5A            |jmp     short 01002856
010027FC  |>  8B45 14       |/mov     eax, dword ptr [ebp+14]
010027FF  |.  3B58 08        ||cmp     ebx, dword ptr [eax+8]
01002802  |.  7D 56           ||jge     short 0100285A
01002804  |.  8B55 0C       ||mov     edx, dword ptr [ebp+C]
01002807  |.  8D0417        ||lea     eax, dword ptr [edi+edx]
0100280A  |.  3BD8            ||cmp     ebx, eax
0100280C  |.  7D 0A           ||jge     short 01002818
0100280E  |.  6A 3F            ||push    3F
01002810  |.  58                 ||pop     eax
01002811  |.  897D FC        ||mov     dword ptr [ebp-4], edi
01002814  |.  2BC7             ||sub     eax, edi
01002816  |.  EB 17            ||jmp     short 0100282F
01002818  |>  8B45 E4       ||mov     eax, dword ptr [ebp-1C]
0100281B  |.  2BC8             ||sub     ecx, eax
0100281D  |.  2BCA             ||sub     ecx, edx
0100281F  |.  3BD9            ||cmp     ebx, ecx
01002821  |.  8945 FC       ||mov     dword ptr [ebp-4], eax
01002824  |.  74 07           ||je      short 0100282D
01002826  |.  C745 FC 3F000>||mov     dword ptr [ebp-4], 3F
0100282D  |>  33C0          ||xor     eax, eax
0100282F  |>  68 2000CC00   ||push    0CC0020                      ; /ROP = SRCCOPY
01002834  |.  FF75 F0       ||push    dword ptr [ebp-10]             ; |YSrc
01002837  |.  50               ||push    eax                                      ; |XSrc
01002838  |.  FF75 F4       ||push    dword ptr [ebp-C]               ; |hSrcDC
0100283B  |.  FF75 F8       ||push    dword ptr [ebp-8]               ; |Height
0100283E  |.  FF75 FC       ||push    dword ptr [ebp-4]               ; |Width
01002841  |.  FF75 18       ||push    dword ptr [ebp+18]             ; |YDest
01002844  |.  53                ||push    ebx                                      ; |XDest
01002845  |.  FF75 08       ||push    dword ptr [ebp+8]              ; |hDestDC
01002848  |.  FF15 7C100001 ||call    dword ptr [<&GDI32.BitBlt>]    ; \BitBlt
0100284E  |.  8B4E 08       ||mov     ecx, dword ptr [esi+8]
01002851  |.  035D FC       ||add     ebx, dword ptr [ebp-4]
01002854  |.  2B0E           ||sub     ecx, dword ptr [esi]
01002856  |>  3BD9           | cmp     ebx, ecx
01002858  |.^ 7C A2         |\jl      short 010027FC
0100285A  |>  8B45 F8       |mov     eax, dword ptr [ebp-8]
0100285D  |.  0145 18       |add     dword ptr [ebp+18], eax
01002860  |.  8B46 0C       |mov     eax, dword ptr [esi+C]
01002863  |.  2B46 04       |sub     eax, dword ptr [esi+4]
01002866  |.  3945 18       |cmp     dword ptr [ebp+18], eax
01002869  |.^ 0F8C 41FFFFFF \jl      010027B0
```

首先是在这里被断下了，我们先来看看这个函数体，有没有什么信息，往上瞧瞧

```
0100271F  |.  53            push    ebx
01002720  |.  56            push    esi
01002721  |.  57            push    edi
01002722  |.  FF75 08       push    dword ptr [ebp+8]                ; /hDC
01002725  |.  FF15 74100001 call    dword ptr            /<CreateCompatibleDC>
0100272B  |.  68 BC120001   push    010012BC             ; /RsrcName = "FELT"
01002730  |.  FF35 00200101 push    dword ptr [1012000]   ; |hInst = 01000000
01002736  |.  8945 F4       mov     dword ptr [ebp-C], eax           ; |
01002739  |.  FF15 80120001 call    dword ptr [<&USER32.LoadBitmapW>>;
0100273F  |.  50            push    eax                              ; /hObject
01002740  |.  FF75 F4       push    dword ptr [ebp-C]                ; |hDC
01002743  |.  8945 DC       mov     dword ptr [ebp-24], eax          ; |
01002746  |.  FF15 78100001 call    dword ptr [<&GDI32.SelectObject>>;
0100274C  |.  8B75 18       mov     esi, dword ptr [ebp+18]
0100274F  |.  8B1E          mov     ebx, dword ptr [esi]
01002751  |.  8B4D 14       mov     ecx, dword ptr [ebp+14]
```

恩，有了新的发现，有个`LoadBitmap`函数，这是加载位图的函数，加载的位图名称已经显示出来了，就是那个`"FELT"`，用RESHACKER打开程序看看，在位图资源处应该会看到这个名为`"FELT"`的图片，只是用我的RESHACKER没有打开，换eXeScope试试，可以看到这个图片就是程序的背景图片，还有52张扑克图片，扑克的点数图片咱就不说了，`"CARDBACK"`是蜘蛛的扑克背景图片，标识号106的是关于对话框的图片，标识号108的是放置扑克的图片，这里只要一一对照知道是哪些相互对应就可以了。

可以看到`"FELT"`应该是程序的背景图片，这里可以大胆猜猜测一下，程序中当前循环的目的就是画出程序的背景，这里可以先放一下，把这里`BitBlt`的断点先取消，接着F9往下看看

```
010028A7  |.  8945 FC       mov     dword ptr [ebp-4], eax
010028AA  |.  8B45 14       mov     eax, dword ptr [ebp+14]
010028AD  |.  83F8 01       cmp     eax, 1
010028B0  |.  56                push    esi
010028B1  |.  8B75 08       mov     esi, dword ptr [ebp+8]
010028B4  |.  57                push    edi
010028B5  |.  8975 D0       mov     dword ptr [ebp-30], esi
//[ebp-30]=esi
010028B8  |.  7C 1D           jl      short 010028D7
010028BA  |.  83F8 34       cmp     eax, 34
010028BD  |.  7F 18           jg      short 010028D7
//如果EAX<[ebp-30]或EAX>0x34(也就是52)跳到10028D7
//如果上述不满足向下走
010028BF  |.  50                push    eax                              ; /<%d>
010028C0  |.  8D45 D4       lea     eax, dword ptr [ebp-2C]          ; |
010028C3  |.  68 DC120001   push    010012DC             ; |Format = "CARD%d"
010028C8  |.  50                push    eax                              ; |s
010028C9  |.  FF15 84120001 call    dword ptr [<&USER32.wsprintfW>]  ; \wsprintfW
//以Format函数格式输出，这里可以应该是 输出“CARD1”~“CARD52”
010028CF  |.  83C4 0C       add     esp, 0C
010028D2  |.  8D45 D4       lea     eax, dword ptr [ebp-2C]
010028D5  |.  EB 1B           jmp     short 010028F2
//上面不符合条件的跳转到这里
010028D7  |>  83F8 69      cmp     eax, 69
010028DA  |.  75 07           jnz     short 010028E3
010028DC  |.  68 BC120001   push    010012BC                  ;  UNICODE "FELT"
010028E1  |.  EB 10           jmp     short 010028F3
```
