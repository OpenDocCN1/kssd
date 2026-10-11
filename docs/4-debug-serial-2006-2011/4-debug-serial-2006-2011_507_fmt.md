
# 电脑 XX 助手算法分析

> 原文：[`www.pediy.com/kssd/pediy10/83019.html`](https://www.pediy.com/kssd/pediy10/83019.html)

**【文章标题】**: 电脑 XX 算法分析  
**【文章作者】**: 丑男无敌  
**【软件名称】**: 电脑 XX 助手  
**【加壳方式】**: 无壳  
**【编写语言】**: `Delphi`  
**【使用工具】**: `OD`  
**【操作平台】**: `XP`  
**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

因为是国内软件，就不写出名字了。而且作者在网上放出的软件是 `demo` 版，代码和资源都不全，仅当练习分析算法之用。

作者耍了一个小把戏，安装完的程序只是个幌子，只是调用的作用，真正的程序放在了 `C` 盘，这个用任务管理器一看就看出来了。注册软件会有错误提示，随便输入假吗，出现错误提示后用 `F12` 暂停法，断下后，通过几层回溯找到错误处，很快就找到关键 `call` 了。

```
00498828  |.  FF92 04010000 call    dword ptr [edx+104]           ;  关键 call,F7 进去
0049882E  |.  8B15 ACFF4900 mov     edx, dword ptr [49FFAC]      
00498834  |.  8802           mov     byte ptr [edx], al
00498836  |.  A1 ACFF4900    mov     eax, dword ptr [49FFAC]     
0049883B  |.  8038 00        cmp     byte ptr [eax], 0
0049883E  |.  75 19         jnz     short 00498859                
00498840  |.  6A 00         push    0
00498842  |.  A1 00044A00   mov     eax, dword ptr [4A0400]      
00498847  |.  8B00          mov     eax, dword ptr [eax]
00498849  |.  66:8B0D C0884>mov     cx, word ptr [4988C0]        
00498850  |.  B2 01         mov     dl, 1
00498852  |.  E8 EDA4F9FF   call    00432D44                       ;  注册码错误提示
00498857  |.  EB 3E         jmp     short 00498897                
00498754  /.  55           push    ebp
00498755  |.  8BEC         mov     ebp, esp
00498757  |.  6A 00        push    0
00498759  |.  6A 00        push    0
0049875B  |.  53           push    ebx
0049875C  |.  8BD8         mov     ebx, eax
0049875E  |.  33C0         xor     eax, eax
00498760  |.  55           push    ebp
00498761  |.  68 B6874900  push    004987B6
00498766  |.  64:FF30      push    dword ptr fs:[eax]
00498769  |.  64:8920      mov     dword ptr fs:[eax], esp
0049876C  |.  8D55 FC      lea     edx, dword ptr [ebp-4]
0049876F  |.  8BC3         mov     eax, ebx
00498771  |.  8B08         mov     ecx, dword ptr [eax]
00498773  |.  FF91 F4000000 call    dword ptr [ecx+F4]
00498779  |.  8B45 FC      mov     eax, dword ptr [ebp-4]
0049877C  |.  50           push    eax
0049877D  |.  8D55 F8      lea     edx, dword ptr [ebp-8]
00498780  |.  8BC3         mov     eax, ebx
00498782  |.  8B08         mov     ecx, dword ptr [eax]
00498784  |.  FF91 F8000000 call    dword ptr [ecx+F8]
0049878A  |.  8B45 F8      mov     eax, dword ptr [ebp-8]        ;  机器码出现 3414745805
0049878D  |.  8B8B 20030000 mov    ecx, dword ptr [ebx+320]
00498793  |.  5A           pop     edx
00498794  |.  E8 B7ECFFFF  call    00497450                       ;  关键 call，继续 F7 进
00497450  /$  55          push    ebp
00497451  |.  8BEC        mov     ebp, esp
00497453  |.  81C4 FCFFFFFF add    esp, -104
00497459  |.  53          push    ebx
0049745A  |.  56          push    esi
0049745B  |.  57          push    edi
0049745C  |.  33DB        xor     ebx, ebx
0049745E  |.  895D FC     mov     dword ptr [ebp-4], ebx
00497461  |.  8BF9        mov     edi, ecx
00497463  |.  8BF2        mov     esi, edx
00497465  |.  8BD8        mov     ebx, eax
00497467  |.  33C0        xor     eax, eax
00497469  |.  55          push    ebp
0049746A  |.  68 B7744900 push    004974B7
0049746F  |.  64:FF30     push    dword ptr fs:[eax]
00497472  |.  64:8920     mov     dword ptr fs:[eax], esp
00497475  |.  8D8D FCFFFFFF lea    ecx, dword ptr [ebp-104]
0049747B  |.  8BD7        mov     edx, edi
0049747D  |.  8BC3        mov     eax, ebx
0049747F  |.  E8 64FEFFFF  call    004972E8                       ;  关键算法处，F7 跟进
00497484  |.  8D95 FCFFFFFF lea    edx, dword ptr [ebp-104]
0049748A  |.  8D45 FC     lea     eax, dword ptr [ebp-4]
0049748D  |.  E8 4ACFF6FF  call    004043DC
00497492  |.  8B45 FC     mov     eax, dword ptr [ebp-4]
00497495  |.  8BD6        mov     edx, esi
00497497  |.  E8 E8D0F6FF  call    00404584                       ;  真假比较,明码的软件
0049749C  |.  0F94C0      sete    al                             ;  标志位，可爆破，改 setne al 或 mov al,1 均可
0049749F  |.  8BD8        mov     ebx, eax
004974A1  |.  33C0        xor     eax, eax
004972E8  /$  55          push    ebp
004972E9  |.  8BEC        mov     ebp, esp
004972EB  |.  83C4 E0     add     esp, -20
004972EE  |.  53          push    ebx
004972EF  |.  56          push    esi
004972F0  |.  57          push    edi
004972F1  |.  33DB        xor     ebx, ebx
004972F3  |.  895D E0     mov     dword ptr [ebp-20], ebx
004972F6  |.  895D E4     mov     dword ptr [ebp-1C], ebx
004972F9  |.  895D E8     mov     dword ptr [ebp-18], ebx
004972FC  |.  8BF9        mov     edi, ecx
004972FE  |.  8955 F8     mov     dword ptr [ebp-8], edx
00497301  |.  8945 FC     mov     dword ptr [ebp-4], eax
00497304  |.  8B45 FC     mov     eax, dword ptr [ebp-4]
00497307  |.  E8 1CD3F6FF  call    00404628
0049730C  |.  8B45 F8     mov     eax, dword ptr [ebp-8]
0049730F  |.  E8 14D3F6FF  call    00404628
00497314  |.  33C0        xor     eax, eax
00497316  |.  55          push    ebp
00497317  |.  68 41744900 push    00497441
0049731C  |.  64:FF30     push    dword ptr fs:[eax]
0049731F  |.  64:8920     mov     dword ptr fs:[eax], esp
00497322  |.  837D FC 00  cmp     dword ptr [ebp-4], 0
00497326  |.  74 6F       je      short 00497397
00497328  |.  BB 01000000 mov     ebx, 1
0049732D  |.  8D75 EF     lea     esi, dword ptr [ebp-11]
00497330  |>  8B45 FC     /mov    eax, dword ptr [ebp-4]
00497333  |.  E8 00D1F6FF  |call   00404438                       ;  取机器码位数，10 位
00497338  |.  50          |push    eax
00497339  |.  8BC3        |mov     eax, ebx
0049733B  |.  48          |dec     eax
0049733C  |.  5A          |pop     edx
0049733D  |.  8BCA        |mov     ecx, edx
0049733F  |.  99          |cdq
00497340  |.  F7F9        |idiv    ecx
00497342  |.  8B45 FC     |mov     eax, dword ptr [ebp-4]
00497345  |.  8A0410      |mov     al, byte ptr [eax+edx]          ;  顺序逐次取机器码
00497348  |.  50          |push    eax
00497349  |.  8B45 FC     |mov     eax, dword ptr [ebp-4]
0049734C  |.  E8 E7D0F6FF  |call   00404438
00497351  |.  5A          |pop     edx
00497352  |.  32D0        |xor     dl, al                          ;  机器码与机器码的位数异或
00497354  |.  32D3        |xor     dl, bl                          ;  机器码与其序号异或，得 X
00497356  |.  8816        |mov     byte ptr [esi], dl
00497358  |.  43          |inc     ebx
00497359  |.  46          |inc     esi
0049735A  |.  83FB 0A     |cmp     ebx, 0A
0049735D  |.^ 75 D1       \jnz    short 00497330                   ;  循环
0049735F  |.  8B45 FC     mov     eax, dword ptr [ebp-4]
00497362  |.  E8 D1D0F6FF  call    00404438
00497367  |.  8BF0        mov     esi, eax
00497369  |.  85F6        test    esi, esi
0049736B  |.  7E 2A       jle     short 00497397
0049736D  |.  BB 01000000 mov     ebx, 1
00497372  |>  8B45 FC     /mov    eax, dword ptr [ebp-4]
00497375  |.  E8 BED0F6FF  |call   00404438
0049737A  |.  2BC3        |sub     eax, ebx
0049737C  |.  8B55 FC     |mov     edx, dword ptr [ebp-4]
0049737F  |.  8A0C02      |mov     cl, byte ptr [edx+eax]          ;  倒序逐次取机器码
00497382  |.  8BC3        |mov     eax, ebx
```



```
  00497384  |.  48             |dec     eax  

  00497385  |.  51             |push    ecx  

  00497386  |.  B9 09000000    |mov     ecx, 9  

  0049738B  |.  99             |cdq  

  0049738C  |.  F7F9          |idiv    ecx  

  0049738E  |.  59             |pop     ecx  

  0049738F  |.  304C15 EF      |xor     byte ptr [ebp+edx-11], cl       ;  倒序取出的机器码与上面得出的 X 异或,得 Y  

  00497393  |.  43             |inc     ebx  

  00497394  |.  4E             |dec     esi  

  00497395  |.^ 75 DB         \|jnz     short 00497372                  ;  循环  

  00497397  |>  837D F8 00    cmp     dword ptr [ebp-8], 0  

  0049739B  |.  74 39         je      short 004973D6  

  0049739D  |.  BB 01000000   mov     ebx, 1  

  004973A2  |.  8D75 EF       lea     esi, dword ptr [ebp-11]         ;  将 Y 的最后一位替换第一位  

  004973A5  |>  8B45 F8       /mov    eax, dword ptr [ebp-8]         ;  固定码 weEF789Ld0121 参与运算  

  004973A8  |.  E8 8BD0F6FF   call    00404438                        ;  取固定码位数  

  004973AD  |.  50            |push    eax  

  004973AE  |.  8BC3         |mov     eax, ebx  

  004973B0  |.  48            |dec     eax  

  004973B1  |.  5A            |pop     edx  

  004973B2  |.  8BCA         |mov     ecx, edx  

  004973B4  |.  99            |cdq  

  004973B5  |.  F7F9         |idiv    ecx  

  004973B7  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]  

  004973BA  |.  8A0410        |mov     al, byte ptr [eax+edx]        ;  顺序逐次取固定码  

  004973BD  |.  3206         |xor     al, byte ptr [esi]            ;  Y 与固定码异或，得 Z  

  004973BF  |.  50            |push    eax  

  004973C0  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]  

  004973C3  |.  E8 70D0F6FF   |call    00404438  

  004973C8  |.  5A            |pop     edx  

  004973C9  |.  32D0         |xor     dl, al                      ;  Z 与固定码位数异或，得 A  

  004973CB  |.  32D3         |xor     dl, bl                      ;  A 与固定码序号异或，得 B  

  004973CD  |.  8816         |mov     byte ptr [esi], dl          ;  传送 B 到 EDX  

  004973CF  |.  43            |inc     ebx  

  004973D0  |.  46            |inc     esi  

  004973D1  |.  83FB 0A       |cmp     ebx, 0A  

  004973D4  |.^ 75 CF        \|jnz     short 004973A5  

  004973D6  |>  8D45 E8       lea     eax, dword ptr [ebp-18]  

  004973D9  |.  E8 9ACDF6FF   call    00404178  

  004973DE  |.  BB 09000000   mov     ebx, 9  

  004973E3  |.  8D75 EF       lea     esi, dword ptr [ebp-11]  

  004973E6  |>  8D45 E4       /lea    eax, dword ptr [ebp-1C]      ;  此循环是把 B 合并起来  

  004973E9  |.  8A16         |mov     dl, byte ptr [esi]  

  004973EB  |.  E8 70CFF6FF   call    00404360  

  004973F0  |.  8B55 E4       |mov     edx, dword ptr [ebp-1C]  

  004973F3  |.  8D45 E8       lea     eax, dword ptr [ebp-18]  

  004973F6  |.  E8 45D0F6FF   call    00404440  

  004973FB  |.  46            |inc     esi  

  004973FC  |.  4B            |dec     ebx  

  004973FD  |.^ 75 E7        \|jnz     short 004973E6  

  004973FF  |.  8D55 E0       lea     edx, dword ptr [ebp-20]  

  00497402  |.  8B45 E8       mov     eax, dword ptr [ebp-18]      ;  合并起来的 B EfK@3  8D45 E0       lea     eax, dword ptr [ebp-20]  

  00497429  |.  BA 03000000   mov     edx, 3  

  0049742E  |.  E8 69CDF6FF   call    0040419C  

  00497433  |.  8D45 F8       lea     eax, dword ptr [ebp-8]  

  00497436  |.  BA 02000000   mov     edx, 2  

  0049743B  |.  E8 5CCDF6FF   call    0040419C  

  00497440  \\.  C3           retn  

  00497441   .^ E9 36C7F6FF   jmp     00403B7C  

  00497446   .^ EB DE         jmp     short 00497426  

  00497448   .  5F            pop     edi  

  00497449   .  5E            pop     esi  

  0049744A   .  5B            pop     ebx  

  0049744B   .  8BE5         mov     esp, ebp  

  0049744D   .  5D            pop     ebp  

  0049744E   .  C3            retn  

  004971A4  /$  55            push    ebp  

  004971A5  |.  8BEC         mov     ebp, esp  

  004971A7  |.  83C4 F0       add     esp, -10  

  004971AA  |.  53            push    ebx  

  004971AB  |.  56            push    esi  

  004971AC  |.  57            push    edi  

  004971AD  |.  33C9         xor     ecx, ecx  

  004971AF  |.  894D F0       mov     dword ptr [ebp-10], ecx  

  004971B2  |.  8BFA         mov     edi, edx  

  004971B4  |.  8945 FC       mov     dword ptr [ebp-4], eax  

  004971B7  |.  8B45 FC       mov     eax, dword ptr [ebp-4]  

  004971BA  |.  E8 69D4F6FF   call    00404628  

  004971BF  |.  33C0         xor     eax, eax  

  004971C1  |.  55            push    ebp  

  004971C2  |.  68 D8724900   push    004972D8  

  004971C7  |.  64:FF30       push    dword ptr fs:[eax]  

  004971CA  |.  64:8920       mov     dword ptr fs:[eax], esp  

  004971CD  |.  8BC7         mov     eax, edi  

  004971CF  |.  E8 A4CFF6FF   call    00404178  

  004971D4  |.  E9 D7000000   jmp     004972B0  

  004971D9  |>  8B45 FC       /mov    eax, dword ptr [ebp-4]      ;  此 call 是把 B 分成三段来计算，以第一段 EfK 来演示  

  004971DC  |.  E8 57D2F6FF   call    00404438  

  004971E1  |.  8BC8         mov     ecx, eax  

  004971E3  |.  8BC1         mov     eax, ecx  

  004971E5  |.  BB 03000000   mov     ebx, 3  

  004971EA  |.  99            cdq  

  004971EB  |.  F7FB         idiv    ebx  

  004971ED  |.  85C0         test    eax, eax  

  004971EF  |.  7E 07         jle     short 004971F8  

  004971F1  |.  BB 03000000   mov     ebx, 3  

  004971F6  |.  EB 02         jmp     short 004971FA  

  004971F8  |>  8BD9         mov     ebx, ecx  

  004971FA  |>  8D45 F9       lea     eax, dword ptr [ebp-7]  

  004971FD  |.  33C9         xor     ecx, ecx  

  004971FF  |.  BA 03000000   mov     edx, 3  

  00497204  |.  E8 9BBBF6FF   call    00402DA4  

  00497209  |.  8D45 F5       lea     eax, dword ptr [ebp-B]  

  0049720C  |.  B9 40000000   mov     ecx, 40  

  00497211  |.  BA 04000000   mov     edx, 4  

  00497216  |.  E8 89BBF6FF   call    00402DA4  

  0049721B  |.  8D45 FC       lea     eax, dword ptr [ebp-4]  

  0049721E  |.  E8 6DD4F6FF   call    00404690  

  00497223  |.  8D55 F9       lea     edx, dword ptr [ebp-7]  

  00497226  |.  8BCB         mov     ecx, ebx  

  00497228  |.  E8 5FB7F6FF   call    0040298C  

  0049722D  |.  83FB 03       cmp     ebx, 3  

  00497230  |.  7C 08         jl      short 0049723A  

  00497232  |.  8A45 FB       mov     al, byte ptr [ebp-5]        ;  取第 3 字节 4B  

  00497235  |.  24 3F         and     al, 3F                      ;  与运算  

  00497237  |.  8845 F8       mov     byte ptr [ebp-8], al        ;  得 0B  计算得的第 1 个字节  

  0049723A  |>  83FB 02       cmp     ebx, 2  

  0049723D  |.  7C 15         jl      short 00497254  

  0049723F  |.  8A45 FA       mov     al, byte ptr [ebp-6]        ;  第 2 字节 66  

  00497242  |.  C1E0 02       shl     eax, 2                      ;  左移两位  

  00497245  |.  33D2         xor     edx, edx  

  00497247  |.  8A55 FB       mov     dl, byte ptr [ebp-5]        ;  第 3 字节 4B  

  0049724A  |.  C1EA 06       shr     edx, 6                      ;  右移两位  

  0049724D  |.  0AC2         or      al, dl                      ;  移位后的两者异或  

  0049724F  |.  24 3F         and     al, 3F                      ;  与运算  

  00497251  |.  8845 F7       mov     byte ptr [ebp-9], al        ;  得 19  计算得的第 2 个字节  

  00497254  |>  8A45 F9       mov     al, byte ptr [ebp-7]        ;  第 1 字节 45  

  00497257  |.  8BD0         mov     edx, eax  

  00497259  |.  C1E2 04       shl     edx, 4                      ;  左移四位  

  0049725C  |.  33C9         xor     ecx, ecx  

  0049725E  |.  8A4D FA       mov     cl, byte ptr [ebp-6]        ;  第 2 字节 66  

  00497261  |.  C1E9 04       shr     ecx, 4                      ;  右移四位  
```



```
00497264  |.  0AD1          |or      dl, cl                   ; 移位后两者异或  
00497266  |.  80E2 3F       |and     dl, 3F                   ; 与运算  
00497269  |.  8855 F6       |mov     byte ptr [ebp-A], dl     ; 得 16  计算得的第 3 个字节  
0049726C  |.  25 FF000000   |and     eax, 0FF                 ; 与远算  
00497271  |.  C1E8 02       |shr     eax, 2                   ; 右移两位  
00497274  |.  24 3F         |and     al, 3F                   ; 与运算  
00497276  |.  8845 F5       |mov     byte ptr [ebp-B], al     ; 得 11   计算得的第 4 个字节  
00497279  |.  8D45 FC       |lea     eax, dword ptr [ebp-4]  
0049727C  |.  8BCB          |mov     ecx, ebx  
0049727E  |.  BA 01000000   |mov     edx, 1  
00497283  |.  E8 50D4F6FF   |call    004046D8                 ; 取剩下的 B  @3  8D45 F0       |/lea      eax, dword ptr [ebp-10]  
00497293  |.  33D2          ||xor     edx, edx  
00497295  |.  8A13          ||mov     dl, byte ptr [ebx]       ; 倒序取刚才计算出的 4 个字节，放入 edx  
00497297  |.  8A92 2DFD4900 ||mov     dl, byte ptr [edx+49FD2D] ; 注意 [edx+49FD2D] 的地址，通过查表得注册码  
0049729D  |.  E8 BED0F6FF   ||call    00404360  
004972A2  |.  8B55 F0       ||mov     edx, dword ptr [ebp-10]  
004972A5  |.  8BC7          ||mov     eax, edi  
004972A7  |.  E8 94D1F6FF   ||call    00404440  
004972AC  |.  43            ||inc     ebx  
004972AD  |.  4E            ||dec     esi  
004972AE  |.^ 75 E0        |\jnz     short 00497290           ; 循环 4 次  
004972B0  |>  837D FC 00     cmp     dword ptr [ebp-4], 0     ;   
004972B4  |.^ 0F85 1FFFFFFF \jnz     004971D9                 ; 循环 2 次，把 B 计算完  
004972BA  |.  33C0          |xor     eax, eax  
004972BC  |.  5A            |pop     edx  
004972BD  |.  59            |pop     ecx  
004972BE  |.  59            |pop     ecx  
004972BF  |.  64:8910       |mov     dword ptr fs:[eax], edx  
004972C2  |.  68 DF724900   |push    004972DF  
004972C7  |>  8D45 F0       |lea     eax, dword ptr [ebp-10]  
004972CA  |.  E8 A9CEF6FF   |call    00404178  
004972CF  |.  8D45 FC       |lea     eax, dword ptr [ebp-4]  
004972D2  |.  E8 A1CEF6FF   |call    00404178  
004972D7  \\.  C3            |retn  
004972D8   .^ E9 9FC8F6FF   |jmp     00403B7C  
004972DD   .^ EB E8         |jmp     short 004972C7  
004972DF   .  5F            |pop     edi  
004972E0   .  5E            |pop     esi                  ; 0012CBD0  
004972E1   .  5B            |pop     ebx  
004972E2   .  8BE5          |mov     esp, ebp  
004972E4   .  5D            |pop     ebp  
004972E5   .  C3            |retn  

表：

0049FD27           47 00 41 49 59     .AIY
0049FD2F  41 47 50 58 44 4A 51 57  AGPXDJQW
0049FD37  4D 48 56 43 4E 46 55 5A  MHVCNFUZ
0049FD3F  52 42 4B 45 53 4F 4C 54  RBKESOLT
0049FD47  74 66 6B 79 73 62 6F 68  tfkysboh
0049FD4F  6C 75 6A 77 65 63 70 6D  lujwecpm
0049FD57  69 61 71 6E 64 78 7A 76  iaqndxzv
0049FD5F  67 72 34 36 2B 30 32 35  gr46+025
0049FD67  37 33 2F 38 31 3D 39 8B  73/81=9
0049FD6F  C0 30 31 32 33 34 35 36  ?123456
0049FD77  37 38 39 61 62 63 64 65  789abcde
0049FD7F  66                       f
```

### 经验总结

1. 顺序逐次取机器码，然后与其位数异或，再与其序号异或，得 `X`。
2. 倒序逐次取机器码，与上面得出的 `X` 异或，得 `Y`，并把 `Y` 的最后替换到第一位。
3. 固定码 `weEF789Ld0121` 参与运算，顺序逐次取固定码与 `Y` 异或，然后分别与固定码的位数和序号异或，得 `B`。
4. 把 `B` 分成三段来计算，每一段经过计算得 4 个字节，将字节倒序放入 `edx`，通过 `[edx+49FD2D]` 的地址查到对应的字母，再把 12 个字母合起来就是注册码了。

软件是明码比较的，一下就爆出来了，但是放出来的 demo 版，也拿它没办法。算法虽然有很多段，但是都是比较简单的运算。注册码的最后一段算法，其实就是首先设置一个密码表，然后通过一个循环，让内存地址的指针指向密码表的某个位置，所指的字符就是注册码，再把这 12 个字符合起来就是注册码了，这样的算法不知道该怎么写算法注册机。

---

2009 年 03 月 01 日 12:33:52

补充汇编注册机源码，关键代码如下。全部代码在附件中。

```asm
;--------------------------------------------------------------------
;关键算法
;--------------------------------------------------------------------
_Reg  proc
    pushad
    invoke   lstrlen,addr szCode    ;取机器码长度
    mov  ecx,eax         
    lea  esi,szCode     ;机器码放到 esi
    xor  ebx,ebx
    xor  edx,edx
_again:
    movzx  eax,byte ptr [esi+ebx]    ;顺序逐位取机器码
    inc  ebx   
    xor  eax,ecx        ;机器码与机器码的位数异或
    xor  eax,ebx        ;机器码与其序号异或
    mov  dword ptr [edi+ebx-1],eax  ;结果存入堆栈,假设为 X
    cmp  ebx,ecx
    jnz  _again        ;循环
    
    mov  ebx,1
_again1:   
    push  ecx
    sub  ecx,ebx
    mov  al,byte ptr [esi+ecx]    ;倒序逐次取机器码
    push  eax
    mov  eax,ebx
    dec  eax
    mov  ecx,9
    cdq
    idiv  ecx
    pop  eax
    xor  byte ptr [edi+edx],al    ;倒序逐位取出的机器码与上面得出的 X 异或,得 Y
    inc  ebx
    pop  ecx
    cmp  ebx,0Bh
    jnz  _again1        ;循环
    
    mov  ebx,1
_again2:
    lea  esi,szGD      ;取固定码 weEF789Ld0121 到 esi
    invoke   lstrlen,addr szGD    ;取其长度
    push  eax
    mov  eax,ebx
    dec  eax
    pop  edx
    mov  ecx,edx
    cdq
    idiv  ecx
    mov  al,byte ptr [esi+edx]    ;顺序逐位取固定码
    xor  al,byte ptr [edi]    ;Y 与固定码异或，得 Z
    push  eax
    invoke   lstrlen,addr szGD
    pop  edx
    xor  dl,al        ;Z 与固定码位数异或
    xor  dl,bl        ;结果再与固定码的序号异或，得 B
    mov  byte ptr [edi+9],dl    ;保存结果 B
    inc  ebx
    inc  edi
    cmp  ebx,0Ah
    jnz  _again2        ;循环
    
    mov  esi,edi
    mov  ebp,esp
_Call:                ;下面的循环将 B 分成 3 个字节为一段来循环计算
    mov  al,byte ptr [edi+2]    ;取第 3 个字节
    and  al,3Fh
    mov  byte ptr [ebp-8],al    ;计算得的第 1 个字节
    mov  al,byte ptr [edi+1]    ;取第 2 字节
    shl  eax,2
    xor  edx,edx
    mov  dl,byte ptr [edi+2]    ;取第 3 字节
    shr  edx,6
    or  al,dl
    and  al,3Fh
    mov  byte ptr [ebp-9],al    ;计算得的第 2 个字节
    mov  al,byte ptr [edi]    ;取第 1 字节
    mov  edx,eax
    shl  edx,4
    xor  ecx,ecx
    mov  cl,byte ptr [edi+1]    ;第 2 字节
    shr  ecx,4
    or  dl,cl
    and  dl,3Fh
    mov  byte ptr [ebp-0Ah],dl    ;计算得的第 3 个字节
    and  eax,0FFh
    shr  eax,2
    and  al,3Fh
    mov  byte ptr [ebp-0Bh],al    ;计算得的第 4 个字节
   
    lea  ebx,dword ptr [ebp-0Bh]    ;将上面计算得的 4 个字节放到 ebx
    mov  ecx,4
    xor  eax,eax
    xor  edx,edx
_CallA:
    lea  ebp,szSZ      ;将码表放到 ebp
    mov  dl,byte ptr [ebx]    ;逐次取出 ebx 处的值到 edx
    mov  dl,byte ptr [ebp+edx]    ;[ebp+edx]地址处的数值即为注册码
    mov  byte ptr [esi+eax+0Fh],dl  ;保存结果
    inc  eax
    inc  ebx
    dec  ecx
    jnz  _CallA        ;循环 ecx=4 次
    add  esi,eax        ;eax 为 4，esi 的地址前移 4 位
    mov  eax,3
    add  edi,eax        ;存放 B 的地址前移 3 位，以进行下一段的运算
    mov  ebp,esp
    cmp  dword ptr [edi],0    ;直到存放 B 的地址再没有数据则退出循环
    jnz  _Call
    
    mov  byte ptr [esi+0Fh],0    ;填上 0 作为结束
    lea  eax,dword ptr [esi+3]    ;将计算的注册码放到 eax
    invoke   lstrcpy,addr dwTemp,eax    ;再放到缓冲区中
    popad
  ret

_Reg endp
```
