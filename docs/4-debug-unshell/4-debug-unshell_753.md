

# 读取 MoleBox 打包的可执行文件中的数据文件一法

> 原文：[`www.pediy.com/kssd/pediy10/103847.html`](https://www.pediy.com/kssd/pediy10/103847.html)

【软件大小】: 70M+  
【加壳方式】: MoleBox  
【使用工具】: OD VC2005  
【软件介绍】: 一个三国群英传 2 的 MOD 版本。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
    前一段时间，有个大学同学想对一个三国群英传 2 的 MOD 游戏进行些数据调整和修改，不过这个 MOD 游戏  
只有一个可执行文件，他没法获得数据文件，所以就找我帮忙。拿到文件后，首先用 PEID 查看，显示为：  
    MoleBox V2.X -> MoleStudio.com \[Overlay\] \*  
很不妙，以前没听说过。再用 KANAL 插件扫描下，显示文件中有使用 ZIP 压缩算法的痕迹。  
    在论坛里搜索了一下，一般谈到的都是获取 PE 文件的方法，PE 文件由于其特殊性,Molebox 一般会把它解压  
到硬盘上。同时这类文件不大，用 dump 内存然后粘贴成一个文件的工作量还可以忍受。不过现在我要读取的文  
件是游戏数据文件，尺寸较大(文件的实际尺寸是 140M), 并且不能保证内存中的数据是按顺序读取出来的，所  
以 dump 内存粘贴的方法也不是很可行。  
    不过通过吸收大家的成果，我还是找到了一种方法，现在分享给大家，希望能给需要的人一些借鉴。  
    现在让我们开始吧。  
step 1:  
    用 OD 载入文件，在 GetFileTime 方法上下断点。这一步主要是借鉴了其他文章中的成果，在此对那些作者表  
示感谢。  
   F9 运行，断下后返回到用户代码中如下位置  
   00560211   .  52            push    edx                              ; /pLastWrite  
00560212   .  8D45 D0       lea     eax, dword ptr \[ebp-30\]          ; |  
00560215   .  50            push    eax                              ; |pLastAccess  
00560216   .  8B4D E0       mov     ecx, dword ptr \[ebp-20\]          ; |  
00560219   .  83C1 1C       add     ecx, 1C                          ; |  
0056021C   .  51            push    ecx                              ; |pCreationTime  
0056021D   .  8B55 D8       mov     edx, dword ptr \[ebp-28\]          ; |  
00560220   .  52            push    edx                              ; |hFile  
00560221   .  FF15 98D75600 call    dword ptr \[56D798\]               ; \\GetFileTime  
00560227   .  8B45 E0       mov     eax, dword ptr \[ebp-20\]  
0056022A   .  8378 20 00    cmp     dword ptr \[eax+20\], 0  
0056022E   .  75 16         jnz     short 00560246  
00560230   .  8B4D E0       mov     ecx, dword ptr \[ebp-20\]  
00560233   .  8379 1C 00    cmp     dword ptr \[ecx+1C\], 0  
00560237   .  75 0D         jnz     short 00560246  
00560239   .  8B55 E0       mov     edx, dword ptr \[ebp-20\]  
0056023C   .  83C2 1C       add     edx, 1C  
0056023F   .  52            push    edx                              ; /pFileTime  
00560240   .  FF15 C0D75600 call    dword ptr \[56D7C0\]               ; \\GetSystemTimeAsFileTime  
00560246   >  C745 A4 00000>mov     dword ptr \[ebp-5C\], 0  
0056024D   .  EB 09         jmp     short 00560258                   ; 一个无条件跳转  
0056024F   > /8B45 A4       mov     eax, dword ptr \[ebp-5C\]  
00560252   . |83C0 01       add     eax, 1  
00560255   .  8945 A4       mov     dword ptr \[ebp-5C\], eax          ;  \[ebp - 5c\]应该是一个计数器  
00560258   > |8B4D A4       mov     ecx, dword ptr \[ebp-5C\]  
0056025B   .  3B4D 94       cmp     ecx, dword ptr \[ebp-6C\]          ;  \[ebp-6c\]存放着打包的文件数  
0056025E   . |0F83 E3000000 jnb     00560347  
00560264   .  8B55 A4       mov     edx, dword ptr \[ebp-5C\]  
00560267   .  C1E2 04       shl     edx, 4  
0056026A   .  8B45 E0       mov     eax, dword ptr \[ebp-20\]  
0056026D   .  8B48 04       mov     ecx, dword ptr \[eax+4\]  
00560270   .  8B45 DC       mov     eax, dword ptr \[ebp-24\]          ;  文件名  
00560273   .  030411        add     eax, dword ptr \[ecx+edx\]  
00560276   .  8B4D A4       mov     ecx, dword ptr \[ebp-5C\]  
00560279   .  C1E1 04       shl     ecx, 4  
0056027C   .  8B55 E0       mov     edx, dword ptr \[ebp-20\]  
0056027F   .  8B52 04       mov     edx, dword ptr \[edx+4\]  
00560282   .  89040A        mov     dword ptr \[edx+ecx\], eax  
00560285   .  8B45 A4       mov     eax, dword ptr \[ebp-5C\]  
00560288   .  C1E0 04       shl     eax, 4  
0056028B   .  8B4D E0       mov     ecx, dword ptr \[ebp-20\]  
0056028E   .  8B51 04       mov     edx, dword ptr \[ecx+4\]  
00560291   .  8B3C02        mov     edi, dword ptr \[edx+eax\]  
00560294   .  83C9 FF       or      ecx, FFFFFFFF  
00560297   .  33C0          xor     eax, eax  
00560299   .  F2:AE         repne   scas byte ptr es:\[edi\]  
0056029B   .  F7D1          not     ecx  
0056029D   .  83C1 FF       add     ecx, -1  
005602A0   .  898D 48FDFFFF mov     dword ptr \[ebp-2B8\], ecx         ;  计算出文件名的长度  
005602A6   .  8B85 48FDFFFF mov     eax, dword ptr \[ebp-2B8\]  
005602AC   .  83C0 01       add     eax, 1  
005602AF   .  50            push    eax  
005602B0   .  E8 7A430000   call    0056462F                         ;  分配文件名长度的内存空间  
005602B5   .  83C4 04       add     esp, 4  
005602B8   .  8985 04FDFFFF mov     dword ptr \[ebp-2FC\], eax  
005602BE   .  8B4D E0       mov     ecx, dword ptr \[ebp-20\]  
005602C1   .  8B51 08       mov     edx, dword ptr \[ecx+8\]  
005602C4   .  8B45 A4       mov     eax, dword ptr \[ebp-5C\]  
005602C7   .  8B8D 04FDFFFF mov     ecx, dword ptr \[ebp-2FC\]  
005602CD   .  890C82        mov     dword ptr \[edx+eax\*4\], ecx  
005602D0   .  8B8D 48FDFFFF mov     ecx, dword ptr \[ebp-2B8\]  
005602D6   .  83C1 01       add     ecx, 1  
005602D9   .  8B55 A4       mov     edx, dword ptr \[ebp-5C\]  
005602DC   .  C1E2 04       shl     edx, 4  
005602DF   .  8B45 E0       mov     eax, dword ptr \[ebp-20\]  
005602E2   .  8B40 04       mov     eax, dword ptr \[eax+4\]  
005602E5   .  8B3410        mov     esi, dword ptr \[eax+edx\]  
005602E8   .  8B55 E0       mov     edx, dword ptr \[ebp-20\]  
005602EB   .  8B42 08       mov     eax, dword ptr \[edx+8\]  
005602EE   .  8B55 A4       mov     edx, dword ptr \[ebp-5C\]  
005602F1   .  8B3C90        mov     edi, dword ptr \[eax+edx\*4\]  
005602F4   .  8BC1          mov     eax, ecx  
005602F6   .  C1E9 02       shr     ecx, 2  
005602F9   .  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
005602FB   .  8BC8          mov     ecx, eax  
005602FD   .  83E1 03       and     ecx, 3  
00560300   .  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr>;  把文件名复制到分配的空间中  
00560302   .  8B8D 48FDFFFF mov     ecx, dword ptr \[ebp-2B8\]  
00560308   .  51            push    ecx                              ; /Count  
00560309   .  8B55 A4       mov     edx, dword ptr \[ebp-5C\]          ; |  
0056030C   .  C1E2 04       shl     edx, 4                           ; |  
0056030F   .  8B45 E0       mov     eax, dword ptr \[ebp-20\]          ; |  
00560312   .  8B48 04       mov     ecx, dword ptr \[eax+4\]           ; |  
00560315   .  8B1411        mov     edx, dword ptr \[ecx+edx\]         ; |  
00560318   .  52            push    edx                              ; |String  
00560319   .  FF15 5CD85600 call    dword ptr \[56D85C\]               ; \\CharUpperBuffA 把文件名变大写  
0056031F   .  8B45 A4       mov     eax, dword ptr \[ebp-5C\]  
00560322   .  C1E0 04       shl     eax, 4  
00560325   .  8B4D E0       mov     ecx, dword ptr \[ebp-20\]  
00560328   .  8B51 04       mov     edx, dword ptr \[ecx+4\]  
0056032B   .  8B4402 04     mov     eax, dword ptr \[edx+eax+4\]  
0056032F   .  0345 A8       add     eax, dword ptr \[ebp-58\]  
00560332   .  8B4D A4       mov     ecx, dword ptr \[ebp-5C\]  
00560335   .  C1E1 04       shl     ecx, 4                           ;  计数器\*16 说明这里有个 16 字节的数据结构  
00560338   .  8B55 E0       mov     edx, dword ptr \[ebp-20\]  
0056033B   .  8B52 04       mov     edx, dword ptr \[edx+4\]  
0056033E   .  89440A 04     mov     dword ptr \[edx+ecx+4\], eax       ;  在这里让我们来数据窗口看一看 edx【003B3033】附近的数据是什么  
00560342   .^ E9 08FFFFFF   jmp     0056024F                         ;  继续处理  
  
数据窗口中的数据  
003B3023  00 92 07 18 00 53 41 4E 47 4F 32 2E 50 41 4B 00  .?.SANGO2.PAK.  
003B3033  28 30 3B 00 34 DA 04 00 29 7C 84 08 33 00 00 00  (0;.4?.)|?3...  
003B3043  00 00 00 00 00 AB AB AB AB AB AB AB AB 00 00 00  ........  
  
我们来看看 003B3033 处开始的 4 个 dword 数据：  
003B3028  
0004DA34(对应的 10 进制数 318004)  
08847C29(对应的 10 进制数 142900265)  
00000033  
第一个明显是文件名地址, 那么第二，第 3 个数是什么呢？我猜测第二个为给文件在 exe 文件中存储的偏移  
地址，第 3 个是文件的原始长度，第 4 个估计是文件处理方法的标志。 如果打包的文件大小 142M+，而现在  
的 exe 本身只有 70M+,那么参考 PEID 检查的结果，数据文件估计被用 zip 算法压缩过了。  
       
现在我们来在这些数据的基础上来进行一些猜测，molebox 通过打开 exe 文件本身，然后把文件指针移动到  
数据文件的开始处，然后再开始解析数据呢。如果这样那么数据文件根本不需要解压到硬盘，就可以完成  
在程序运行过程中对数据文件的读取。  
  
下面让我们进入第二步.  
  
step 2: 找到 OEP  
  最好的办法是 ESP 定律，这里不再赘述,读者可以参考相关文章来完成这一步。  
step 3: 找到 molebox 中读取文件的函数  
   现在我假定 EIP 指针已经位于 OEP 处，现在在 SetFilePointer 函数上下断点，寻找第二个参数为 0x4DA34  
的调用（这一不可以通过条件断点来实现）。断下后返回到用户代码空间。  
  
0055C50B  |.  8945 F8       mov     dword ptr \[ebp-8\], eax  
0055C50E  |.  6A 02         push    2                                ; /Options = DUPLICATE\_SAME\_ACCESS  
0055C510  |.  6A 00         push    0                                ; |Inheritable = FALSE  
0055C512  |.  6A 00         push    0                                ; |Access = 0  
0055C514  |.  FF75 14       push    dword ptr \[ebp+14\]               ; |phTarget  
0055C517  |.  FF15 78D75600 call    dword ptr \[56D778\]               ; |\[GetCurrentProcess  
0055C51D  |.  50            push    eax                              ; |hTargetProcess  
0055C51E  |.  8B45 FC       mov     eax, dword ptr \[ebp-4\]           ; |  
0055C521  |.  8B40 0C       mov     eax, dword ptr \[eax+C\]           ; |  
0055C524  |.  FF70 24       push    dword ptr \[eax+24\]               ; |hSource  
0055C527  |.  FF15 78D75600 call    dword ptr \[56D778\]               ; |\[GetCurrentProcess  
0055C52D  |.  50            push    eax                              ; |hSourceProcess  
0055C52E  |.  FF15 24A05600 call    dword ptr \[<&KERNEL32.DuplicateH>; \\DuplicateHandle  
0055C534  |.  8B45 14       mov     eax, dword ptr \[ebp+14\]  
0055C537  |.  8338 FF       cmp     dword ptr \[eax\], -1  
0055C53A  |.  74 6B         je      short 0055C5A7  
0055C53C  |.  6A 00         push    0                                ; /Origin = FILE\_BEGIN  
0055C53E  |.  6A 00         push    0                                ; |pOffsetHi = NULL  
0055C540  |.  8B45 F8       mov     eax, dword ptr \[ebp-8\]           ; |  
0055C543  |.  FF70 04       push    dword ptr \[eax+4\]                ; |OffsetLo  
0055C546  |.  8B45 14       mov     eax, dword ptr \[ebp+14\]          ; |  
0055C549  |.  FF30          push    dword ptr \[eax\]                  ; |hFile  
0055C54B  |.  FF15 1CD85600 call    dword ptr \[56D81C\]               ; \\SetFilePointer  
0055C551  |.  6A 10         push    10                               ; 我们现在在这里  
0055C553  |.  E8 D7800000   call    0056462F  
0055C558  |.  59            pop     ecx  
0055C559  |.  8945 EC       mov     dword ptr \[ebp-14\], eax  
0055C55C  |.  8B45 EC       mov     eax, dword ptr \[ebp-14\]  
0055C55F  |.  8945 F0       mov     dword ptr \[ebp-10\], eax  
0055C562  |.  8B45 F0       mov     eax, dword ptr \[ebp-10\]  
0055C565  |.  8B4D FC       mov     ecx, dword ptr \[ebp-4\]  
0055C568  |.  8908          mov     dword ptr \[eax\], ecx  
0055C56A  |.  8B45 F0       mov     eax, dword ptr \[ebp-10\]  
0055C56D  |.  8B4D 14       mov     ecx, dword ptr \[ebp+14\]  
0055C570  |.  8B09          mov     ecx, dword ptr \[ecx\]  
0055C572  |.  8948 08       mov     dword ptr \[eax+8\], ecx  
0055C575  |.  8B45 F0       mov     eax, dword ptr \[ebp-10\]  
0055C578  |.  8360 04 00    and     dword ptr \[eax+4\], 0  
0055C57C  |.  0FB645 18     movzx   eax, byte ptr \[ebp+18\]  
0055C580  |.  F7D8          neg     eax  
0055C582  |.  1BC0          sbb     eax, eax  
0055C584  |.  25 000000C0   and     eax, C0000000  
0055C589  |.  05 00000040   add     eax, 40000000  
0055C58E  |.  8B4D F0       mov     ecx, dword ptr \[ebp-10\]  
0055C591  |.  8941 0C       mov     dword ptr \[ecx+C\], eax  
0055C594  |.  FF75 F0       push    dword ptr \[ebp-10\]  
0055C597  |.  8B45 14       mov     eax, dword ptr \[ebp+14\]  
0055C59A  |.  FF30          push    dword ptr \[eax\]  
0055C59C  |.  8B0D 74D95600 mov     ecx, dword ptr \[56D974\]  
0055C5A2  |.  E8 59BAFFFF   call    00558000  
0055C5A7  |>  6A 01         push    1  
0055C5A9  |.  58            pop     eax  
0055C5AA  |.  EB 02         jmp     short 0055C5AE  
0055C5AC  |>  33C0          xor     eax, eax  
0055C5AE  |>  C9            leave  
0055C5AF  \\.  C2 1400       retn    14  
  
    看到 SetFileHandle 前面的 DuplicateHandle 没有，这里首先复制了一个 handle,然后再设置 handle 的  
文件指针。我们看到复制后的 handle 保存在函数的第 4 个参数(\[ebp+14\])中，这说明真正使用 handle 的地方该函数的调用者。  
现在让我们返回到调用者。  
00567929  /.  55            push    ebp  
0056792A  |.  8BEC          mov     ebp, esp  
0056792C  |.  51            push    ecx  
0056792D  |.  51            push    ecx  
0056792E  |.  6A 00         push    0                                ; /Arg5 = 00000000  
00567930  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]           ; |  
00567933  |.  50            push    eax                              ; |Arg4  
00567934  |.  6A 00         push    0                                ; |Arg3 = 00000000  
00567936  |.  6A 00         push    0                                ; |Arg2 = 00000000  
00567938  |.  FF75 08       push    dword ptr \[ebp+8\]                ; |Arg1  
0056793B  |.  E8 944BFFFF   call    0055C4D4                         ; \\suitang.0055C4D4  
00567940  |.  85C0          test    eax, eax                         ; 现在我们在这里  
00567942  |.  74 05         je      short 00567949  
00567944  |.  8B45 FC       mov     eax, dword ptr \[ebp-4\]           ; 返回 handle  
00567947  |.  EB 1D         jmp     short 00567966  
00567949  |>  68 BCA55600   push    0056A5BC                         ;  ASCII "kernel32.dll"  
0056794E  |.  68 8CA85600   push    0056A88C                         ;  ASCII "\_lopen"  
00567953  |.  E8 BDE5FFFF   call    00565F15  
00567958  |.  59            pop     ecx  
00567959  |.  59            pop     ecx  
0056795A  |.  8945 F8       mov     dword ptr \[ebp-8\], eax  
0056795D  |.  FF75 0C       push    dword ptr \[ebp+C\]  
00567960  |.  FF75 08       push    dword ptr \[ebp+8\]  
00567963  |.  FF55 F8       call    dword ptr \[ebp-8\]  
00567966  |>  C9            leave  
00567967  \\.  C2 0800       retn    8  
  
这里我们看到前面处理好的 handle 作为参数返回了。现在让我们再返回上一级。  
00455A10    55              push    ebp  
00455A11    8BEC            mov     ebp, esp  
00455A13    8B45 0C         mov     eax, dword ptr \[ebp+C\]  
00455A16    50              push    eax  
00455A17    8B4D 08         mov     ecx, dword ptr \[ebp+8\]  
00455A1A    51              push    ecx  
00455A1B    FF15 EC434D00   call    dword ptr \[4D43EC\]               ; kernel32.\_lcreat  
00455A21    5D              pop     ebp  
00455A22    C3              retn  
00455A23    55              push    ebp  
00455A24    8BEC            mov     ebp, esp  
00455A26    8B45 0C         mov     eax, dword ptr \[ebp+C\]  
00455A29    50              push    eax  
00455A2A    8B4D 08         mov     ecx, dword ptr \[ebp+8\]  
00455A2D    51              push    ecx  
00455A2E    FF15 F0434D00   call    dword ptr \[4D43F0\]               ; suitang.00567929  
00455A34    5D              pop     ebp                              ; 这次我们在这里  
00455A35    C3              retn  
再 Ctrl + F9 一次:  
00452488    55              push    ebp  
00452489    8BEC            mov     ebp, esp  
0045248B    83EC 1C         sub     esp, 1C  
0045248E    6A 00           push    0  
00452490    8B45 08         mov     eax, dword ptr \[ebp+8\]  
00452493    50              push    eax  
00452494    E8 8A350000     call    00455A23                         ; handle = fn0();  
00452499    83C4 08         add     esp, 8                           ; 现在我们在这里  
0045249C    8945 FC         mov     dword ptr \[ebp-4\], eax  
0045249F    837D FC FF      cmp     dword ptr \[ebp-4\], -1  
004524A3    75 04           jnz     short 004524A9                   ; handle != 0 go  
004524A5    33C0            xor     eax, eax  
004524A7    EB 30           jmp     short 004524D9  
004524A9    6A 18           push    18  
004524AB    8D4D E4         lea     ecx, dword ptr \[ebp-1C\]  
004524AE    51              push    ecx  
004524AF    8B55 FC         mov     edx, dword ptr \[ebp-4\]  
004524B2    52              push    edx  
004524B3    E8 E3350000     call    00455A9B                         ; fn1(Handle, buf, len)  
004524B8    83C4 0C         add     esp, 0C  
004524BB    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
004524BE    50              push    eax  
004524BF    E8 72350000     call    00455A36                         ; fn2(handle)  
004524C4    83C4 04         add     esp, 4  
004524C7    817D E4 50414B5>cmp     dword ptr \[ebp-1C\], 534B4150  
004524CE    75 07           jnz     short 004524D7  
004524D0    B8 01000000     mov     eax, 1  
004524D5    EB 02           jmp     short 004524D9  
004524D7    33C0            xor     eax, eax  
004524D9    8BE5            mov     esp, ebp  
004524DB    5D              pop     ebp  
004524DC    C3              retn  
  
    看到这个地方的 fn0, fn1, fn2，我想那些熟悉文件读写的朋友肯定会想到 CreateFile、 ReadFile、   
CloseHandle。什么你没想到，那咱俩没啥共同语言。^\_^  
    如果 fn2 是实现 CloseHande 的功能（这是真的），那么我们想在内存中找出完整文件的数据的希望  
就…… 啥也不说了，眼泪汪汪的。  
  
step 3 : 他山之石，可以攻玉  
    分析 fn1 是比较复杂的。从前面的分析看，很可能要和 zip 算法做斗争。这里我们偷个懒，利用下 fn1,让  
程序自动帮我把数据写到一个文件里。  
    现在祭出 VS2005，新建一个 windows 命令行程序。  
    把下面的代码拷贝进去  
    #include "stdafx.h"  
    #include <windows.h>  
      
    #define ADDR\_lopen      0xAAAAAAAA         
    #define ADDR\_lwrite      0xBBBBBBBB  
    #define ADDR\_lclose      0xCCCCCCCC  
    #define ADDR\_myReadFile 0xDDDDDDDD  
      
    char\* gFileName = (char\*)0xEEEEEEEE;  
      
    typedef HFILE  (WINAPI\* myOpen)(char \* fn, UINT mode);  
    typedef UINT   (\* myRead)(HFILE, LPVOID, UINT);  
    typedef UINT   (WINAPI\* myWrite)(HFILE, LPVOID, UINT);  
    typedef HFILE  (WINAPI\* myClose)(HFILE);  
      
    void SaveFile(HFILE f){  
      char buf\[1024\];  
      myOpen fnOpen = (myOpen)(ADDR\_lopen);  
      myRead fnRead = (myRead)(ADDR\_myReadFile);  
      myWrite fnWrite = (myWrite)(ADDR\_lwrite);  
      myClose fnClose = (myClose)(ADDR\_lclose);  
      
      int len = 0;  
      HFILE fW = fnOpen(gFileName, OF\_CREATE | OF\_WRITE);  
      while ( (len = fnRead(f, buf, 1024)) > 0){  
        fnWrite(fW, buf, len);  
      }  
      fnClose(fW);  
    }  
      
    int \_tmain(int argc, \_TCHAR\* argv\[\])  
    {  
      HMODULE hm = ::LoadLibrary("kernel32.dll");  
      myOpen fnOpen = (myOpen)(ADDR\_lopen);  
      myRead fnRead = (myRead)(ADDR\_myReadFile);  
      myWrite fnWrite = (myWrite)(ADDR\_lwrite);  
      myClose fnClose = (myClose)(ADDR\_lclose);  
      HFILE f = fnOpen("", OF\_READ);  
      ::SaveFile(f);  
      fnClose(f);  
      ::FreeLibrary(hm);  
        
      return 0;  
    }  
  
    生成可执行文件的 release 版本，然后用 OD 打开，在 0x401000 出找到我们 SaveFile 函数。汇编代码  
如下：  
00401000  /$  81EC 00040000 sub     esp, 400  
00401006  |.  56            push    esi  
00401007  |?  68 01100000   push    1001  
0040100C  |?  68 EEEEEEEE   push    EEEEEEEE                         ;EEEEEEEE 表示的是目标文件名的地址，我们用前面找到的 003B3028 替换       
把下面的 AAAAAAAA, BBBBBBBB, CCCCCCCC 分别用 kernel32.\_lopen, kernel32.\_write 和 kernel32.\_lclose 替换  
把 DDDDDDDD 用前面找到的函数 fn1 的地址 00455A9B 替换  
00401011  |?  B8 AAAAAAAA   mov     eax, AAAAAAAA  
00401016  |?  FFD0          call    eax  
00401018  |?  68 00040000   push    400  
0040101D  |?  8D4C24 08     lea     ecx, dword ptr \[esp+8\]  
00401021  |?  51            push    ecx  
00401022  |?  57            push    edi  
00401023  |?  BA DDDDDDDD   mov     edx, DDDDDDDD  
00401028  |?  8BF0          mov     esi, eax  
0040102A  |?  FFD2          call    edx  
0040102C  |?  83C4 0C       add     esp, 0C  
0040102F  |.  85C0          test    eax, eax  
00401031  |.  7E 27         jle     short 0040105A  
00401033  |?  50            push    eax  
00401034  |?  8D4424 08     lea     eax, dword ptr \[esp+8\]  
00401038  |.  50            push    eax  
00401039  |?  56            push    esi  
0040103A  |.  B9 BBBBBBBB   mov     ecx, BBBBBBBB  
0040103F  |.  FFD1          call    ecx  
00401041  |>  68 00040000   /push    400  
00401046  |.  8D5424 08     |lea     edx, dword ptr \[esp+8\]  
0040104A  |?  52            push    edx  
0040104B  |?  57            push    edi  
0040104C  |?  B8 DDDDDDDD   mov     eax, DDDDDDDD  
00401051  |?  FFD0          call    eax  
00401053  |?  83C4 0C       add     esp, 0C  
00401056  |?  85C0          test    eax, eax  
00401058  |.^ 7F D9         |jg      short 00401033  
0040105A  |.  56            |push    esi  
0040105B  |?  B9 CCCCCCCC   mov     ecx, CCCCCCCC  
00401060  |?  FFD1          call    ecx  
00401062  |?  5E            pop     esi  
00401063  |?  81C4 00040000 add     esp, 400  
00401069  |.  C3            retn  
  
  
这里有几点要注意：  
  1.  fn1 是个 c 调用的函数，  
  2. 上面的汇编代码显示, edi 被用来传递参数。  
  3. 我们的代码假定了要写的目标文件已经存在了。所以要现在程序的目录下生成一个 0 字节的同名文件。  
  
    现在我们重新运行程序，并让程序停在 0045249C(fn0 返回的地方)处。  
    然后我们需要在程序空间中找一个空白处例如 0047EF00（选择 00 开始的地址便于实际操作中调整跳  
转地址的位置），从 0047EF00 处开始依次输入前面的汇编代码，同时做函数地址的替换和跳转地址的调  
整。并在 0047EF69 处下断点。输入汇编代码这个不知道大家有没有什么好方法，这里代码比较少，还可  
以接受，代码多了还真不好办。哪位过路的有啥好办法，可一定要告诉我啊。  
    现在万事具备，只欠东风了。  
  
step 4: 功成  
   现在回到 fn0 返回处，修改代码如下  
   00452494    E8 8A350000     call    00455A23                      ; fn0  
00452499    83C4 08         add     esp, 8                           ; 现在我们在这里  
  
以下两句为修改  
0045249C    8BF8            mov     edi, eax                           
0045249E    E8 5DCA0200     call    0047EF00  
  
    Now F9 运行程序，当程序停在 0047EF69 处时，我们需要的文件就生成了。现在我们可以证实下第 3 个字段是  
不是文件长度。  
     
  
最后：  
    一点说明，因为我要处理的这个游戏里面只有一个数据文件，并且我也不知道生成这个程序的 molebox 程序  
版本（我到写这篇文章时还没有见过这个程序），所以不能保证这个方法对您的文件有效。  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 12 月 26 日 上午 09:49:33