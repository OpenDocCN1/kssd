

# 联创上网助手 2.310.0 不完全协议分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-163.htm`](https://www.pediy.com/kssd/pediy09/pediy09-163.htm)

> 今天突然看到这个文件,总觉得让它发霉太委屈了,如文中所说,共有三个重要的数据包,这里是以前整理到前面两个的,一直没有兴趣去把最后一个也整理一下,等什么时候再说吧,反正都是重复的东西![](img/75356aaf48bd2553b2b09887823c13ee.jpg "酷")  
>    
> ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
> 【文章标题】: 联创上网助手 2.310.0 不完全协议分析  
> 【文章作者】: the0crat  
> 【作者邮箱】: the0crat.cn\_at\_gmail.com  
> 【作者主页】: [`the0crat.spaces.live.com`](http://the0crat.spaces.live.com)  
> 【生产日期】: 20070408  
> 【软件名称】: 联创上网助手 2.310.0  
> 【作者声明】: 本文仅供研究学习，本人对因这篇文章而导致的一切后果，不承担任何法律责任。本文中的不足之处请各位多多指教  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
> \--------------------------------------------------------------------------------  
> (注:"不完全协议分析"指的是所需要构造的三个数据包中此文只写到前面两个,作者中间搁置了很久,重新拿起来的时候不想继续分析第三个了,至于第叁个数据包的结构都是重复的东西大家可以参照第二个数据包)  
> 这个程序是用来连接网络的,江苏的很多高校都用它来上网计费,相信很多人都不会陌生.它用的是 802.1x 协议验证,FS5208 交换机  
> 简要介绍一下这个软件,用 winpcap,通过检测 ip 地址的数量和 nat 等来实现防代理,一旦检测到立马断线,使用户不能通过一台电脑共享上网,太黑了.上次的那个老版本还限制网卡数量,大于 1 也断线.  
> 这样就使得上网费用比较高,每小时 0.6 元,我习惯了不管做什么都经常连着网,每个月基本上上网费用就在 80 左右,根本没有一点大学该有的样子,很多同学学校都是可以包月.所以很是不爽.  
> 多人共享上网是个好方案，那就首先要解除这个软件对多 ip 和 nat 的限制，或者分析协议重写一个。废话不说，进入正题。  
> 关于 802.1x 的认证流程，参考图 1，考虑到网上能够找到很多介绍 802.1x 的资料，而且这东西比较简单但是太长，所以本文跳过。更加详细的内容请各位自行参考 802.1x 的相关资料以明白认证流程。  
> 客户端在认证过程中一共有五个包，其中客户端发送的一共有 EAPoL-Start 和 EAP-Response/Identity 和 EAP-Response/Challenge 三个包,限于文章长度,本文分析的是前两个  
> 用 Ethernet 抓包可以看到 Start 包很清晰,并且 802.1x 的数据包在 Ethernet 里也很直观,能看到清晰的结构,没有什么多说的  
> 0000  ff ff ff ff ff ff xx xx  xx xx xx xx 88 8e 01 01   ........ ..C.....  
> 0010  00 00 6c 69 6e 6b 61 67  65 00 00 00 00 00         ..linkag e.....    
>   
> (本文中一些敏感的数据做了处理)  
>   
> \[Start 数据包结构\]  
> 目标 MAC 6 字节  
> 源 MAC 6 字节  
> 类型 2 字节 0x888e  
> 802.1x  
>     版本 1 字节 1  
>     类型 1 字节 1  
>     长度 2 字节 0  
> Trailer 12 字节  
>     标记字符 7 字节 必须是"linkage"  
>     空 5 字节  
>    
> 我们关心的是后面两个包的内容  
> 先看一下 Response/Identity  
> 0000   xx xx xx xx xx xx xx xx xx xx xx xx 88 8e 01 00  ..S.......C.....  
> 0010   00 1a 02 04 00 1a 01 xx xx xx xx xx xx xx xx xx  .......xxxxxxxxx  
> 0020   xx xx xx 40 69 6e 74 65 72 6e 65 74 6c 69 6e 6b  xxx@internetlink  
> 0030   61 67 65 00 f4 1d af e2 dd 9f 48 a1 00 00 00 00  age.......H.....  
>   
> \[Response/Identity 数据包结构\]  
> 目标 MAC 6 字节  
> 源 MAC 6 字节  
> 类型 2 字节 0x888e  
> 802.1x  
>     版本 1 字节 1  
>     类型 1 字节 0  
>     长度 2 字节 0x1a  
>     扩展信息  
>         Code 1 字节 2  
>         id 1 字节 必须和 Resquest/Identity 包中的一样  
>         长度 26 字节 0x1a  
>         类型 1 字节 1  
>         用户名 21 字节  
> Trailer 20 字节  
>     标记字符 8 字节 必须是"linkage"  
>     IdentityCode 8 字节  
>     空 4 字节  
>   
> 如果是上 edu,区别只在于用户名的长度和包的总长度分别相应减小 5 个长度  
>    
> 还有 Request/Identity 中一个很重要的是返回一个 Identity number(多次观察推测可能是个时间截吧~)  
> 从协议流程来看,其中 IdentityCode 的值应该是 Identity number 经过某些运算后的得出的.废话不说,逆向一下  
> 查壳,ASProtect 2.3 SKE build 04.26 Beta \[3\]  
> 脱壳可以写好几篇文章了,跑太远了,本文重点讲的是逆向的过程,所以直接飞过,不会手脱的话可以试试 VolX 的脚本 Aspr2.XX\_unpacker\_v1.0SC.  
> 脱了以后什么东西都光光的了,这软件也没有什么其他的保护  
>   
> bp wpcap\_sendpacket,点验证后第二次断下来,return 就是 Response/Identity 的处理过程,多 return 几个可以到这里  
>    
> 00408ED0  /$  55               push    ebp  
> 00408ED1  |.  8BEC             mov     ebp, esp  
> 00408ED3  |.  83E4 F8          and     esp, FFFFFFF8  
> 00408ED6  |.  81EC 48060000    sub     esp, 648  
> 00408EDC  |.  A1 A0984100      mov     eax, dword ptr \[4198A0\]  
> 00408EE1  |.  66:8B0D A4984100 mov     cx, word ptr \[4198A4\]  
> 00408EE8  |.  53               push    ebx  
> 00408EE9  |.  55               push    ebp  
> 00408EEA  |.  56               push    esi  
> 00408EEB  |.  57               push    edi  
> 00408EEC  |.  894424 78        mov     dword ptr \[esp+78\], eax  
> 00408EF0  |.  66:894C24 7C     mov     word ptr \[esp+7C\], cx  
> 00408EF5  |.  E8 E686FFFF      call    004015E0  
> 00408EFA  |.  8B10             mov     edx, dword ptr \[eax\]             ;  //////////////////////////////////////////  
> 00408EFC  |.  895424 7E        mov     dword ptr \[esp+7E\], edx          ;  往缓冲区里填包的数据  
> 00408F00  |.  66:8B40 04       mov     ax, word ptr \[eax+4\]  
> 00408F04  |.  66:898424 820000>mov     word ptr \[esp+82\], ax  
> 00408F0C  |.  C68424 84000000 >mov     byte ptr \[esp+84\], 88  
> 00408F14  |.  C68424 85000000 >mov     byte ptr \[esp+85\], 8E  
> 00408F1C  |.  C68424 87000000 >mov     byte ptr \[esp+87\], 0  
> 00408F24  |.  C68424 86000000 >mov     byte ptr \[esp+86\], 1  
> 00408F2C  |.  C68424 8A000000 >mov     byte ptr \[esp+8A\], 2  
> 00408F34  |.  E8 B783FFFF      call    004012F0                         ;  mov eax,PacketID  
> 00408F39  |.  888424 8B000000  mov     byte ptr \[esp+8B\], al  
> 00408F40  |.  C68424 8E000000 >mov     byte ptr \[esp+8E\], 1  
> 00408F48  |.  E8 F38CFFFF      call    00401C40                         ;  mov eax,用户名指针  
> 00408F4D  |.  8D50 01          lea     edx, dword ptr \[eax+1\]  
> 00408F50  |>  8A08             mov     cl, byte ptr \[eax\]  
> 00408F52  |.  40               inc     eax  
> 00408F53  |.  84C9             test    cl, cl  
> 00408F55  |.^ 75 F9            jnz     short 00408F50  
> 00408F57  |.  2BC2             sub     eax, edx  
> 00408F59  |.  8BE8             mov     ebp, eax  
> 00408F5B  |.  0FB7DD           movzx   ebx, bp  
> 00408F5E  |.  E8 DD8CFFFF      call    00401C40                         ;  mov eax,用户名指针  
> 00408F63  |.  8BCB             mov     ecx, ebx  
> 00408F65  |.  8BD1             mov     edx, ecx  
> 00408F67  |.  C1E9 02          shr     ecx, 2  
> 00408F6A  |.  8BF0             mov     esi, eax  
> 00408F6C  |.  8DBC24 8F000000  lea     edi, dword ptr \[esp+8F\]  
> 00408F73  |.  F3:A5            rep     movs dword ptr es:\[edi\], dword p>  
> 00408F75  |.  8BCA             mov     ecx, edx  
> 00408F77  |.  83E1 03          and     ecx, 3  
> 00408F7A  |.  83C5 05          add     ebp, 5  
> 00408F7D  |.  F3:A4            rep     movs byte ptr es:\[edi\], byte ptr>  
> 00408F7F  |.  55               push    ebp                              ; /NetShort  
> 00408F80  |.  8D73 17          lea     esi, dword ptr \[ebx+17\]          ; |  
> 00408F83  |.  E8 FA3D0000      call    0040CD82                         ; \\ntohs  
> 00408F88  |.  55               push    ebp                              ; /NetShort  
> 00408F89  |.  66:898424 8C0000>mov     word ptr \[esp+8C\], ax            ; |  
> 00408F91  |.  E8 EC3D0000      call    0040CD82                         ; \\ntohs  
> 00408F96  |.  66:898424 8C0000>mov     word ptr \[esp+8C\], ax  
> 00408F9E  |.  0FB68424 8B00000>movzx   eax, byte ptr \[esp+8B\]  
> 00408FA6  |.  50               push    eax  
> 00408FA7  |.  8D4C24 14        lea     ecx, dword ptr \[esp+14\]  
> 00408FAB  |.  68 B0634100      push    004163B0                         ;  ASCII "SEND : RSP/ID  ID(%d)",LF  
> 00408FB0  |.  51               push    ecx  
> 00408FB1  |.  E8 B73F0000      call    0040CF6D                         ;  将包的 id 填入"...ID(%d)"  
> 00408FB6  |.  8D5424 1C        lea     edx, dword ptr \[esp+1C\]  
> 00408FBA  |.  68 08000088      push    88000008  
> 00408FBF  |.  52               push    edx  
> 00408FC0  |.  E8 4B0B0000      call    00409B10  
> 00408FC5  |.  E8 16F5FFFF      call    004084E0  
> 00408FCA  |.  8D8434 8C000000  lea     eax, dword ptr \[esp+esi+8C\]  
> 00408FD1  |.  6A 00            push    0  
> 00408FD3  |.  50               push    eax  
> 00408FD4  |.  E8 07FDFFFF      call    00408CE0                         ;  填 trailer 都在这里了！  
> 00408FD9  |.  03F0             add     esi, eax  
> 00408FDB  |.  8D8C24 94000000  lea     ecx, dword ptr \[esp+94\]  
> 00408FE2  |.  56               push    esi  
> 00408FE3  |.  51               push    ecx  
> 00408FE4  |.  E8 E72F0000      call    0040BFD0                         ;  sendpacket()  
> 00408FE9  |.  A1 10B24100      mov     eax, dword ptr \[41B210\]  
> 00408FEE  |.  83C4 24          add     esp, 24  
> 00408FF1  |.  40               inc     eax  
> 00408FF2  |.  5F               pop     edi  
> 00408FF3  |.  A3 10B24100      mov     dword ptr \[41B210\], eax  
> 00408FF8  |.  B8 01000000      mov     eax, 1  
> 00408FFD  |.  5E               pop     esi  
> 00408FFE  |.  5D               pop     ebp  
> 00408FFF  |.  5B               pop     ebx  
> 00409000  |.  8BE5             mov     esp, ebp  
> 00409002  |.  5D               pop     ebp  
> 00409003  \\.  C3               retn  
>    
>    
> 跟入 00408FD4  |.  E8 07FDFFFF      call    00408CE0  
>    
>    
> 00408CE0   $  55            push    ebp  
> 00408CE1   .  8BEC          mov     ebp, esp  
> 00408CE3   .  81EC A4000000 sub     esp, 0A4  
> 00408CE9   .  8B0D E8624100 mov     ecx, dword ptr \[4162E8\]  
> 00408CEF   .  53            push    ebx  
> 00408CF0   .  56            push    esi  
> 00408CF1   .  8B75 08       mov     esi, dword ptr \[ebp+8\]  
> 00408CF4   .  8BC6          mov     eax, esi  
> 00408CF6   .  8908          mov     dword ptr \[eax\], ecx  
> 00408CF8   .  8B15 EC624100 mov     edx, dword ptr \[4162EC\]  
> 00408CFE   .  57            push    edi  
> 00408CFF   .  8950 04       mov     dword ptr \[eax+4\], edx  
> 00408D02   .  C745 EC F8624>mov     dword ptr \[ebp-14\], 004162F8          ;  ASCII "hello"  
> 00408D09   .  C745 08 D45C4>mov     dword ptr \[ebp+8\], 00415CD4           ;  ASCII "info"  
> 00408D10   .  74 03         je      short 00408D15  
> 00408D12   .  75 01         jnz     short 00408D15  
> 00408D14      E8            db      E8  
> 00408D15   >  8D45 EC       lea     eax, dword ptr \[ebp-14\]  
> 00408D18   .  8D5D 08       lea     ebx, dword ptr \[ebp+8\]  
> 00408D1B   .  8B45 08       mov     eax, dword ptr \[ebp+8\]  
> 00408D1E   .  8B4D EC       mov     ecx, dword ptr \[ebp-14\]  
> 00408D21   .  50            push    eax  
> 00408D22   .  51            push    ecx  
> 00408D23   .  68 F0624100   push    004162F0                              ;  ASCII "%s,%s"  
> 00408D28   .  E8 48480000   call    0040D575                              ;  格式化  
> 00408D2D   .  83C4 0C       add     esp, 0C  
> 00408D30   .  E8 CBF7FFFF   call    00408500                              ;  自身 hash  
>   
> hash 值影响 IdentityCode,跟入后一路进入到这里  
> hash 运算的函数过程  
>   
> 004084A0  /$  8B5424 08     mov     edx, dword ptr \[esp+8\]           ;  文件长度,nLen  
> 004084A4  |.  8B4C24 04     mov     ecx, dword ptr \[esp+4\]           ;  指向文件数据,pData  
> 004084A8  |.  83C8 FF       or      eax, FFFFFFFF  
> 004084AB  |.  85D2          test    edx, edx  
> 004084AD  |.  74 21         je      short 004084D0  
> 004084AF  |.  56            push    esi  
> 004084B0  |.  57            push    edi  
> 004084B1  |>  0FB639        /movzx   edi, byte ptr \[ecx\]  
> 004084B4  |.  8BF0          |mov     esi, eax  
> 004084B6      81E6 FF000000 and     esi, 0FF  
> 004084BC  |.  33F7          |xor     esi, edi  
> 004084BE  |.  8B3CB5 889441>|mov     edi, dword ptr \[esi\*4+419488\]  
> 004084C5  |.  C1E8 08       |shr     eax, 8  
> 004084C8  |.  33C7          |xor     eax, edi  
> 004084CA  |.  41            |inc     ecx  
> 004084CB  |.  4A            |dec     edx  
> 004084CC  |.^ 75 E3         \\jnz     short 004084B1  
> 004084CE  |.  5F            pop     edi  
> 004084CF  |.  5E            pop     esi  
> 004084D0  |>  F7D0          not     eax  
> 004084D2  \\.  C3            retn  
>   
> 也就是  
> int sum=0xFFFFFFFF;  
> int i;  
> for(i=0;i<nLen;i++)  
> {  
> sum=(sum>>=8)^((sum&0xFF)^\*pData)\*4+0x419488;  
> pData++;  
> }  
> hash 值是 0x34DEB678,脱壳以后要将整个函数改一下,比如改成  
> mov eax,34DEB678  
> retn  
>   
> 解决了自效验后返回到刚才那里继续  
>    
> 00408D35   .  8BF8          mov     edi, eax  
> 00408D37   .  A1 60B64100   mov     eax, dword ptr \[41B660\]  
> 00408D3C   .  33DB          xor     ebx, ebx  
> 00408D3E   .  3BC3          cmp     eax, ebx  
> 00408D40   .  75 06         jnz     short 00408D48  
> 00408D42   .  893D 60B64100 mov     dword ptr \[41B660\], edi  
> 00408D48   >  E8 93F7FFFF   call    004084E0  
> 00408D4D   .  57            push    edi                                 ; /HostLong  
> 00408D4E   .  E8 29400000   call    <jmp.&ws2\_32.htonl>                 ; \\htonl(自身 hash)  
> 00408D53   .  8946 08       mov     dword ptr \[esi+8\], eax              ;  压入包的"linkage."后面  
> 00408D56   .  E8 15F7FFFF   call    00408470  
> 00408D5B   .  E8 80F7FFFF   call    004084E0  
> 00408D60   .  E8 0BF7FFFF   call    00408470  
> 00408D65   .  90            nop  
> 00408D66   .  EB 00         jmp     short 00408D68  
> 00408D68  />  55            push    ebp  
> 00408D69  |.  8BEC          mov     ebp, esp  
> 00408D6B  |.  41            inc     ecx  
> 00408D6C  |.  52            push    edx  
> 00408D6D  |.  90            nop  
> 00408D6E  |.  5A            pop     edx  
> 00408D6F  |.  49            dec     ecx  
> 00408D70  |.  5D            pop     ebp  
> 00408D71  |.  41            inc     ecx  
> 00408D72  |.  68 E700E700   push    0E700E7                               ; /NetLong = E700E7  
> 00408D77  |.  E8 00400000   call    <jmp.&ws2\_32.htonl>                 ; \\htonl(0xE700E7);  
> 00408D7C  |.  8946 0C       mov     dword ptr \[esi+C\], eax              ;  这个也压入包后面  
> 00408D7F  |.  33C0          xor     eax, eax  
> 00408D81  |.  889D 5CFFFFFF mov     byte ptr \[ebp-A4\], bl  
> 00408D87  |.  B9 1F000000   mov     ecx, 1F  
> 00408D8C  |.  8DBD 5DFFFFFF lea     edi, dword ptr \[ebp-A3\]  
> 00408D92  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
> 00408D94  |.  66:AB         stos    word ptr es:\[edi\]  
> 00408D96  |.  AA            stos    byte ptr es:\[edi\]  
> 00408D97  |.  33D2          xor     edx, edx  
> 00408D99  |.  33C0          xor     eax, eax  
> 00408D9B  |.  8955 F1       mov     dword ptr \[ebp-F\], edx  
> 00408D9E  |.  8945 DD       mov     dword ptr \[ebp-23\], eax  
> 00408DA1  |.  8955 F5       mov     dword ptr \[ebp-B\], edx  
> 00408DA4  |.  8945 E1       mov     dword ptr \[ebp-1F\], eax  
> 00408DA7  |.  8955 F9       mov     dword ptr \[ebp-7\], edx  
> 00408DAA  |.  8945 E5       mov     dword ptr \[ebp-1B\], eax  
> 00408DAD  |.  66:8955 FD    mov     word ptr \[ebp-3\], dx  
> 00408DB1  |.  66:8945 E9    mov     word ptr \[ebp-17\], ax  
> 00408DB5  |.  885D F0       mov     byte ptr \[ebp-10\], bl  
> 00408DB8  |.  8855 FF       mov     byte ptr \[ebp-1\], dl  
> 00408DBB  |.  8845 EB       mov     byte ptr \[ebp-15\], al  
> 00408DBE  |.  8BFF          mov     edi, edi  
> 00408DC0  |>  8A88 F0984100 /mov     cl, byte ptr \[eax+4198F0\]  
> 00408DC6  |.  888C05 5CFFFF>|mov     byte ptr \[ebp+eax-A4\], cl  
> 00408DCD  |.  40            |inc     eax  
> 00408DCE  |.  3ACB          |cmp     cl, bl  
> 00408DD0  |.^ 75 EE         \\jnz     short 00408DC0  
> 00408DD2  |.  B8 F0984100   mov     eax, 004198F0                         ;  ASCII "0123456789012345678901234567890123456789"  
> 00408DD7  |.  8D50 01       lea     edx, dword ptr \[eax+1\]                ;  记这个字符串为 strMagic  
> 00408DDA  |.  8D9B 00000000 lea     ebx, dword ptr \[ebx\]  
> 00408DE0  |>  8A08          /mov     cl, byte ptr \[eax\]  
> 00408DE2  |.  40            |inc     eax  
> 00408DE3  |.  3ACB          |cmp     cl, bl  
> 00408DE5  |.^ 75 F9         \\jnz     short 00408DE0  
> 00408DE7  |.  2BC2          sub     eax, edx                              ;  strMagic 长度 n  
> 00408DE9  |.  8BF8          mov     edi, eax  
> 00408DEB  |.  55            push    ebp  
> 00408DEC  |.  8BEC          mov     ebp, esp  
> 00408DEE  |.  6A FF         push    -1  
> 00408DF0  |.  68 07B20100   push    1B207  
> 00408DF5  |.  68 0E640300   push    3640E                                 ;  SE handler installation  
> 00408DFA  |.  64:A1 0000000>mov     eax, dword ptr fs:\[0\]  
> 00408E00  |.  50            push    eax  
> 00408E01  |.  64:8925 00000>mov     dword ptr fs:\[0\], esp  
> 00408E08  |.  58            pop     eax  
> 00408E09  |.  64:A3 0000000>mov     dword ptr fs:\[0\], eax  
> 00408E0F  |.  58            pop     eax  
> 00408E10  |.  58            pop     eax  
> 00408E11  |.  58            pop     eax  
> 00408E12  |.  58            pop     eax  
> 00408E13  |.  8BE8          mov     ebp, eax  
> 00408E15  |.  E8 8683FFFF   call    004011A0                              ;  跟得太阳升起又落下了，终于又将服务器返回的 identity number 取出来了  
> 00408E1A  |.  50            push    eax                                   ; /NetLong  
> 00408E1B  |.  E8 5C3F0000   call    0040CD7C                              ; \\ntohl(identity number)  
> 00408E20  |.  8B55 E0       mov     edx, dword ptr \[ebp-20\]  
> 00408E23  |.  8D8C3D 5CFFFF>lea     ecx, dword ptr \[ebp+edi-A4\]  
> 00408E2A  |.  8901          mov     dword ptr \[ecx\], eax  
> 00408E2C  |.  8B45 E4       mov     eax, dword ptr \[ebp-1C\]  
> 00408E2F  |.  8951 04       mov     dword ptr \[ecx+4\], edx  
> 00408E32  |.  8B55 E8       mov     edx, dword ptr \[ebp-18\]  
> 00408E35  |.  8941 08       mov     dword ptr \[ecx+8\], eax  
> 00408E38  |.  8951 0C       mov     dword ptr \[ecx+C\], edx                ;  在 strMagic 后面附加 16 个字节的长度，用 ntohl(identity number)接到后面，其余填 0  
> 00408E3B  |.  E8 A0F6FFFF   call    004084E0  
> 00408E40  |.  83C7 10       add     edi, 10  
> 00408E43  |.  57            push    edi                                   ;  strMagic 现在的长度 n=40+16  
> 00408E44  |.  8D85 5CFFFFFF lea     eax, dword ptr \[ebp-A4\]  
> 00408E4A  |.  50            push    eax                                   ;  指向 strMagic 头的指针  
> 00408E4B  |.  8D4D F0       lea     ecx, dword ptr \[ebp-10\]  
> 00408E4E  |.  51            push    ecx                                   ;  指向输出的地址，字符长度 16 字节  
> 00408E4F  |.  E8 DC180000   call    0040A730                              ;  关键 HashFunction(&array\_hash,&strMagic,56);  
> 00408E54  |.  8A4E 09       mov     cl, byte ptr \[esi+9\]                  ;  /////////////////////  
> 00408E57  |.  8A45 F1       mov     al, byte ptr \[ebp-F\]                  ;  得出最终 trailer  
> 00408E5A  |.  8A5E 08       mov     bl, byte ptr \[esi+8\]  
> 00408E5D  |.  8A55 F0       mov     dl, byte ptr \[ebp-10\]  
> 00408E60  |.  32C8          xor     cl, al  
> 00408E62  |.  8A46 0A       mov     al, byte ptr \[esi+A\]  
> 00408E65  |.  884E 09       mov     byte ptr \[esi+9\], cl  
> 00408E68  |.  8A4D F2       mov     cl, byte ptr \[ebp-E\]  
> 00408E6B  |.  32C1          xor     al, cl  
> 00408E6D  |.  8A4E 0C       mov     cl, byte ptr \[esi+C\]  
> 00408E70  |.  32DA          xor     bl, dl  
> 00408E72  |.  8A55 F3       mov     dl, byte ptr \[ebp-D\]  
> 00408E75  |.  8846 0A       mov     byte ptr \[esi+A\], al  
> 00408E78  |.  8A45 F4       mov     al, byte ptr \[ebp-C\]  
> 00408E7B  |.  32C8          xor     cl, al  
> 00408E7D  |.  8A46 0D       mov     al, byte ptr \[esi+D\]  
> 00408E80  |.  885E 08       mov     byte ptr \[esi+8\], bl  
> 00408E83  |.  8A5E 0B       mov     bl, byte ptr \[esi+B\]  
> 00408E86  |.  884E 0C       mov     byte ptr \[esi+C\], cl  
> 00408E89  |.  8A4D F5       mov     cl, byte ptr \[ebp-B\]  
> 00408E8C  |.  32DA          xor     bl, dl  
> 00408E8E  |.  8A55 F6       mov     dl, byte ptr \[ebp-A\]  
> 00408E91  |.  32C1          xor     al, cl  
> 00408E93  |.  8A4E 0F       mov     cl, byte ptr \[esi+F\]  
> 00408E96  |.  885E 0B       mov     byte ptr \[esi+B\], bl  
> 00408E99  |.  8A5E 0E       mov     bl, byte ptr \[esi+E\]  
> 00408E9C  |.  8846 0D       mov     byte ptr \[esi+D\], al  
> 00408E9F  |.  8A45 F7       mov     al, byte ptr \[ebp-9\]  
> 00408EA2  |.  83C4 0C       add     esp, 0C  
> 00408EA5  |.  32DA          xor     bl, dl  
> 00408EA7  |.  32C8          xor     cl, al  
> 00408EA9  |.  884E 0F       mov     byte ptr \[esi+F\], cl  
> 00408EAC  |.  8B4D 0C       mov     ecx, dword ptr \[ebp+C\]  
> 00408EAF  |.  885E 0E       mov     byte ptr \[esi+E\], bl  
> 00408EB2  |.  5F            pop                                           ;  ediarray\_trailer\_num 的末 8 位是 identity number 和 0x00e700e7  
> 00408EB3  |.  894E 10       mov     dword ptr \[esi+10\], ecx               ;  array\_trailer\_new 的前 8 位分别为 array\_trailer 的对应位与 array\_hash 的对应位的 xor，后 8 位为 0  
> 00408EB6  |.  5E            pop     esi                                   ;  /////////////////////  
> 00408EB7  |.  B8 14000000   mov     eax, 14  
> 00408EBC  |.  5B            pop     ebx  
> 00408EBD  |.  8BE5          mov     esp, ebp  
> 00408EBF  |.  5D            pop     ebp  
> 00408EC0  \\.  C3            retn  
>    
> 跟进 call    0040A730  
>    
> 0040A730  /$  83EC 58       sub     esp, 58  
> 0040A733  |.  8B4C24 60     mov     ecx, dword ptr \[esp+60\]  
> 0040A737  |.  33C0          xor     eax, eax  
> 0040A739  |.  894424 14     mov     dword ptr \[esp+14\], eax  
> 0040A73D  |.  894424 10     mov     dword ptr \[esp+10\], eax  
> 0040A741  |.  8B4424 64     mov     eax, dword ptr \[esp+64\]  
> 0040A745  |.  50            push    eax  
> 0040A746  |.  51            push    ecx  
> 0040A747  |.  8D5424 08     lea     edx, dword ptr \[esp+8\]  
> 0040A74B  |.  52            push    edx  
> 0040A74C  |.  C74424 0C 012>mov     dword ptr \[esp+C\], 67452301  
> 0040A754  |.  C74424 10 89A>mov     dword ptr \[esp+10\], EFCDAB89  
> 0040A75C  |.  C74424 14 FED>mov     dword ptr \[esp+14\], 98BADCFE  
> 0040A764  |.  C74424 18 765>mov     dword ptr \[esp+18\], 10325476  
> 0040A76C  |.  E8 4FFEFFFF   call    0040A5C0                              ;  将 strMagic 复制到另一个位置  
>                                                                           ;  取一块内存初始化为 0123456...54321,末尾加上 38h\*8=1C0h,8 个字节（这个是最初用来 md5 的）  
> 0040A771  |.  8B4C24 68     mov     ecx, dword ptr \[esp+68\]  
> 0040A775  |.  8D4424 0C     lea     eax, dword ptr \[esp+C\]  
> 0040A779  |.  50            push    eax  
> 0040A77A  |.  51            push    ecx  
> 0040A77B  |.  E8 00FFFFFF   call    0040A680                              ;  跟进  
> 0040A780  |.  83C4 6C       add     esp, 6C  
> 0040A783  \\.  C3            retn  
>    
> 进来 call    0040A680  
>    
> 0040A680  /$  83EC 08       sub     esp, 8  
> 0040A683  |.  56            push    esi  
> 0040A684  |.  57            push    edi  
> 0040A685  |.  8B7C24 18     mov     edi, dword ptr \[esp+18\]  
> 0040A689  |.  33C9          xor     ecx, ecx  
> 0040A68B  |.  8D47 12       lea     eax, dword ptr \[edi+12\]  
> 0040A68E  |.  8BFF          mov     edi, edi  
> 0040A690  |>  8A50 FE       /mov     dl, byte ptr \[eax-2\]                 ;  /\*  
> 0040A693  |.  88540C 08     |mov     byte ptr \[esp+ecx+8\], dl  
> 0040A697  |.  8A50 FF       |mov     dl, byte ptr \[eax-1\]  
> 0040A69A  |.  88540C 09     |mov     byte ptr \[esp+ecx+9\], dl  
> 0040A69E  |.  8A10          |mov     dl, byte ptr \[eax\]  
> 0040A6A0  |.  88540C 0A     |mov     byte ptr \[esp+ecx+A\], dl  
> 0040A6A4  |.  8A50 01       |mov     dl, byte ptr \[eax+1\]  
> 0040A6A7  |.  88540C 0B     |mov     byte ptr \[esp+ecx+B\], dl  
> 0040A6AB  |.  83C1 04       |add     ecx, 4  
> 0040A6AE  |.  83C0 04       |add     eax, 4  
> 0040A6B1  |.  83F9 08       |cmp     ecx, 8  
> 0040A6B4  |.^ 72 DA         \\jb      short 0040A690                       ;  \*/38h\*8=1C0h，将其复制一份到别处，长度 8 个字节  
> 0040A6B6  |.  8B4F 10       mov     ecx, dword ptr \[edi+10\]               ;  ecx=此 1C0h  
> 0040A6B9  |.  C1E9 03       shr     ecx, 3                                ;  ecx>>=3  
> 0040A6BC  |.  83E1 3F       and     ecx, 3F                               ;  ecx&=3Fh  
> 0040A6BF  |.  83F9 38       cmp     ecx, 38  
> 0040A6C2  |.  B8 38000000   mov     eax, 38  
> 0040A6C7  |.  72 05         jb      short 0040A6CE  
> 0040A6C9  |.  B8 78000000   mov     eax, 78  
> 0040A6CE  |>  2BC1          sub     eax, ecx                              ;  ecx 小于 38h 则 eax=0（即错误）,否则 eax=78h-38h=40h。（缓冲区按标准的 4 个字节对齐，所以分配 40h 的长度）  
> 0040A6D0  |.  50            push    eax  
> 0040A6D1  |.  68 B0984100   push    004198B0  
> 0040A6D6  |.  57            push    edi  
> 0040A6D7  |.  E8 E4FEFFFF   call    0040A5C0                              ;  这里才是重要的 md5  
> 0040A6DC  |.  6A 08         push    8  
> 0040A6DE  |.  8D4424 18     lea     eax, dword ptr \[esp+18\]  
> 0040A6E2  |.  50            push    eax  
> 0040A6E3  |.  57            push    edi  
> 0040A6E4  |.  E8 D7FEFFFF   call    0040A5C0                              ;  再次 md5,得到 array\_hash  
> 0040A6E9  |.  8B4C24 2C     mov     ecx, dword ptr \[esp+2C\]               ;  指向准备复制到的地址  
> 0040A6ED  |.  83C4 18       add     esp, 18  
> 0040A6F0  |.  41            inc     ecx  
> 0040A6F1  |.  8D47 02       lea     eax, dword ptr \[edi+2\]  
> 0040A6F4  |.  BE 04000000   mov     esi, 4  
> 0040A6F9  |.  8DA424 000000>lea     esp, dword ptr \[esp\]  
> 0040A700  |>  8A50 FE       mov     dl, byte ptr \[eax-2\]                  ;  //////////////////  
> 0040A703  |.  8851 FF       mov     byte ptr \[ecx-1\], dl  
> 0040A706  |.  8A50 FF       mov     dl, byte ptr \[eax-1\]  
> 0040A709  |.  8811          mov     byte ptr \[ecx\], dl  
> 0040A70B  |.  8A10          mov     dl, byte ptr \[eax\]  
> 0040A70D  |.  8851 01       mov     byte ptr \[ecx+1\], dl  
> 0040A710  |.  8A50 01       mov     dl, byte ptr \[eax+1\]  
> 0040A713  |.  8851 02       mov     byte ptr \[ecx+2\], dl  
> 0040A716  |.  83C0 04       add     eax, 4  
> 0040A719  |.  83C1 04       add     ecx, 4  
> 0040A71C  |.  4E            dec     esi                                   ;  复制  
> 0040A71D  |.^ 75 E1         jnz     short 0040A700                        ;  //////////////////  
> 0040A71F  |.  B9 16000000   mov     ecx, 16  
> 0040A724  |.  33C0          xor     eax, eax  
> 0040A726  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
> 0040A728  |.  5F            pop     edi  
> 0040A729  |.  5E            pop     esi  
> 0040A72A  |.  83C4 08       add     esp, 8  
> 0040A72D  \\.  C3            retn  
>    
> \[现在内存中 Response/Identity 数据包的结构\]  
> 目标 MAC 6 字节  
> 源 MAC 6 字节  
> 类型 2 字节 0x888e  
> 802.1x  
>     版本 1 字节 1  
>     类型 1 字节 0  
>     长度 2 字节 "internet":0x1a "edu":0x15  
>     扩展信息  
>         Code 1 字节 2  
>         id 1 字节 必须和 Resquest/Identity 包中的一样  
>         长度 26 字节 "internet":0x1a "edu":0x15  
>         类型 1 字节 1  
>         用户名 "internet":21 字节 "edu":16 字节  
> Trailer 20 字节  
>     标记字符 8 字节 必须是"linkage"  
>     自效验码 4 字节 必须是 0x78B6DE34  
>     标量 4 字节 0xE700E7  
>     空 4 字节  
>    
> 流程总结  
> strMagic\[\]="0123456789012345678901234567890123456789";  
> strcat(strMagic,(char\*)ntohl(服务器返回的 identity number));  
> MagicNum=0x67452301EFCDAB8998BADCFE10325476;  
> //////////////关键 HashFunction(&array\_hash,&strMagic,56);  
> MagicNum=算法 x(strMagic,MagicNum); //算法 x 太大但十分明了，所以单独放于文件"FuncX.txt"中，略微修改即可提取  
> array\_hash=算法 x(strMagic,MagicNum);  
> 看看包中 trailer 的结构:  
> "linkage" 8 字节  
> 自效验码 4 字节 必须是 0x78B6DE34  
> 标量 0xE700E7 4 字节  
> 空 4 字节  
> 从 trailer 中的自效验码开始 8 个字节每个字节与 array\_hash 的对应位进行异或  
> //////////////  
> 大功告成,此即为所求的 Response/Identity 包了  
>    
>    
>    
>    
>    
>    
> 附加数据  
>    
> 0040C335  |.  83C4 2C          add     esp, 2C                          ;  /////////////////////////////////////////////////  
> 0040C338  |>  A1 ECB84100      mov     eax, dword ptr \[41B8EC\]          ;  本机 MAC  
> 0040C33D  |>  8A13             mov     dl, byte ptr \[ebx\]               ;  Request,Identity 包  
> 0040C33F  |.  3A10             cmp     dl, byte ptr \[eax\]  
> 0040C341  |.  0F85 A5080000    jnz     0040CBEC  
> 0040C347  |.  8A4B 01          mov     cl, byte ptr \[ebx+1\]  
> 0040C34A  |.  3A48 01          cmp     cl, byte ptr \[eax+1\]  
> 0040C34D  |.  0F85 99080000    jnz     0040CBEC  
> 0040C353  |.  8A53 02          mov     dl, byte ptr \[ebx+2\]  
> 0040C356  |.  3A50 02          cmp     dl, byte ptr \[eax+2\]  
> 0040C359  |.  0F85 8D080000    jnz     0040CBEC  
> 0040C35F  |.  8A4B 03          mov     cl, byte ptr \[ebx+3\]  
> 0040C362  |.  3A48 03          cmp     cl, byte ptr \[eax+3\]  
> 0040C365  |.  0F85 81080000    jnz     0040CBEC  
> 0040C36B  |.  8A53 04          mov     dl, byte ptr \[ebx+4\]  
> 0040C36E  |.  3A50 04          cmp     dl, byte ptr \[eax+4\]  
> 0040C371  |.  0F85 75080000    jnz     0040CBEC  
> 0040C377  |.  8A4B 05          mov     cl, byte ptr \[ebx+5\]  
> 0040C37A  |.  3A48 05          cmp     cl, byte ptr \[eax+5\]             ;  这一段对比收到的包中的本地的 mac 地址是否正确  
> 0040C37D  |.  0F85 69080000    jnz     0040CBEC                         ;  /////////////////////////////////////////////////  
> 0040C383  |.  8A43 0C          mov     al, byte ptr \[ebx+C\]  
> 0040C386  |.  3C 88            cmp     al, 88  
> 0040C388  |.  0F85 1B060000    jnz     0040C9A9  
> 0040C38E  |.  807B 0D 8E       cmp     byte ptr \[ebx+D\], 8E             ;  这一段判断是不是 802.1x 认证方式  
> 0040C392  |.  0F85 11060000    jnz     0040C9A9                         ;  /////////////////////////////////////////////////  
> 0040C398  |.  8B15 88B14100    mov     edx, dword ptr \[41B188\]  
> 0040C39E  |.  833A 01          cmp     dword ptr \[edx\], 1               ;  判断协议版本  
> 0040C3A1  |.  0F85 02060000    jnz     0040C9A9  
> 0040C3A7  |.  0FB64B 12        movzx   ecx, byte ptr \[ebx+12\]  
> 0040C3AB  |.  8BC1             mov     eax, ecx  
> 0040C3AD  |.  48               dec     eax                              ;  从 Extensible Authentication Protocol 的标志位判断当前数据包是哪一个步骤的。这里有四个步骤; Switch (cases 1..4)  
> 0040C3AE  |.  0F84 D8020000    je      0040C68C                         ;  Request(code=1)  
> 0040C3B4  |.  83E8 02          sub     eax, 2  
> 0040C3B7  |.  0F84 80010000    je      0040C53D  
> 0040C3BD  |.  48               dec     eax  
> 0040C3BE  |.  74 32            je      short 0040C3F2 

 4dd466d0e1e0cc123afbe504da2462ca