

# 关于对 TheMida 的 MessageBox 下断

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-203.htm`](https://www.pediy.com/kssd/pediy09/pediy09-203.htm)

> 1、goto MessageBoxA  
> 77D5058A >  8BFF            mov     edi, edi  
> 77D5058C    55              push    ebp  
> 77D5058D    8BEC            mov     ebp, esp  
> 77D5058F    833D BC04D777 0>cmp     dword ptr \[77D704BC\], 0    
> 77D50596    74 24           je      short 77D505BC  
> 77D50598    64:A1 18000000  mov     eax, dword ptr fs:\[18\]  
> 77D5059E    6A 00           push    0  
> 77D505A0    FF70 24         push    dword ptr \[eax+24\]  
> 77D505A3    68 240BD777     push    77D70B24  
> 77D505A8    FF15 C812D177   call    dword ptr \[<&KERNEL32.Interlocke>; kernel32.InterlockedCompareExchange  
> 77D505AE    85C0            test    eax, eax  
> 77D505B0    75 0A           jnz     short 77D505BC  
> 77D505B2    C705 200BD777 0>mov     dword ptr \[77D70B20\], 1  
> 77D505BC    6A 00           push    0  
> 77D505BE    FF75 14         push    dword ptr \[ebp+14\]  
> 77D505C1    FF75 10         push    dword ptr \[ebp+10\]  
> 77D505C4    FF75 0C         push    dword ptr \[ebp+C\]  
> 77D505C7    FF75 08         push    dword ptr \[ebp+8\]  
> 77D505CA    E8 2D000000     call    MessageBoxExA  
> 77D505CF    5D              pop     ebp  
> 77D505D0    C2 1000         retn    10  
>   
> 2、Ctrl+F2  
> 3、F9   //themida 在显示发现调试器时候必断，大家可以去试试，该方法适用新版 themida  
> \-----------------------------------------------------------------------------------------------------  
> 77D704BC 下硬件访问断点。  
> 原理：  
> themida 将 MessageBox 函数拷贝到自己代码空间去运行，这导致了我们对 MessageBox 下断确断不下来  
> 但是其将 user32.dll 的代码拷贝到自己分配的空间的时候，代码虽然在自己空间，但是数据扔用 user32.dll 中的原地址空间。  
>   
> 有兴趣逆向一下 softworm 大侠的 MessageBox 就会理解的更深  
>   
> http://bbs.pediy.com/showthread.php?t=23721  
>   
> 感觉 kanxue、softworm 

> 付上 softworm 大侠 MessageBox 里面的一个虚拟偏移转成物理偏移+载入基址逆向的注释  
>   
> 00401000  /$  56            push    esi  
> 00401001  |.  57            push    edi  
> 00401002  |.  8B7C24 0C     mov     edi, dword ptr \[esp+C\]        ;  edi 指向 user32.dll 载入基址  
> 00401006  |.  33D2          xor     edx, edx  
> 00401008  |.  33C0          xor     eax, eax  
> 0040100A  |.  8B4F 3C       mov     ecx, dword ptr \[edi+3C\]  
> 0040100D  |.  03CF          add     ecx, edi                      ;  ecx 指向 PE  
> 0040100F  |.  66:8B51 14    mov     dx, word ptr \[ecx+14\]         ;  可选头大小  
> 00401013  |.  8D740A 18     lea     esi, dword ptr \[edx+ecx+18\]   ;  esi 指向块表  
> 00401017  |.  33D2          xor     edx, edx  
> 00401019  |.  66:8B51 06    mov     dx, word ptr \[ecx+6\]          ;  区段数  
> 0040101D  |.  85D2          test    edx, edx  
> 0040101F  |.  7E 2B         jle     short 0040104C  
> 00401021  |.  53            push    ebx  
> 00401022  |.  8D4E 0C       lea     ecx, dword ptr \[esi+C\]        ;  块虚拟地址  
> 00401025  |.  8B7424 14     mov     esi, dword ptr \[esp+14\]       ;  函数的 RVA  
> 00401029  |.  55            push    ebp  
> 0040102A  |.  8BDA          mov     ebx, edx                      ;  ebx 块数计数  
> 0040102C  |>  8B11          /mov     edx, dword ptr \[ecx\]         ;  edx 虚拟偏移  
> 0040102E  |.  3BF2          |cmp     esi, edx  
> 00401030  |.  72 12         |jb      short 00401044               ;  esi 小于 edx 就跳  
> 00401032  |.  8B69 FC       |mov     ebp, dword ptr \[ecx-4\]       ;  块的虚拟大小  
> 00401035  |.  03EA          |add     ebp, edx  
> 00401037  |.  3BF5          |cmp     esi, ebp  
> 00401039  |.  77 09         |ja      short 00401044               ;  esi 大于 ebp 就跳  
> 0040103B  |.  8B41 08       |mov     eax, dword ptr \[ecx+8\]       ;  块在文件中的偏移  
> 0040103E  |.  2BC2          |sub     eax, edx                     ;  虚拟偏移和物理偏移的差值  
> 00401040  |.  03C7          |add     eax, edi  
> 00401042  |.  03C6          |add     eax, esi  
> 00401044  |>  83C1 28       |add     ecx, 28  
> 00401047  |.  4B            |dec     ebx                          ;  递扣 ebx 为 0 跳出循环  
> 00401048  |.^ 75 E2         \\jnz     short 0040102C  
> 0040104A  |.  5D            pop     ebp  
> 0040104B  |.  5B            pop     ebx  
> 0040104C  |>  5F            pop     edi  
> 0040104D  |.  5E            pop     esi  
> 0040104E  \\.  C3            retn 

 4dd466d0e1e0cc123afbe504da2462ca