
```
  00454171  |.  56      push    esi  
  00454172  |.  57      push    edi  
  00454173  |.  55      push    ebp  
  00454174  |.  8BF9    mov     edi, ecx  
  00454176  |.  8BEA    mov     ebp, edx  
  00454178  |.  8BF0    mov     esi, eax  
  0045417A  |.  8B46 10 mov     eax, [esi+10]  
  0045417D  |.  C1E8 03 shr     eax, 3  
  00454180  |.  83E0 3F and     eax, 3F  
  00454183  |.  8BD7    mov     edx, edi  
  00454185  |.  C1E2 03 shl     edx, 3  
  00454188  |.  0156 10 add     [esi+10], edx  
  0045418B  |.  3B56 10 cmp     edx, [esi+10]  
  0045418E  |.  76 03   jbe     short 00454193  
  00454190  |.  FF46 14 inc     dword ptr [esi+14>  
  00454193  |>  8BD7    mov     edx, edi  
  00454195  |.  C1EA 1D shr     edx, 1D  
  00454198  |.  0156 14 add     [esi+14], edx  
  0045419B  |.  BB 4000>mov     ebx, 40  
  004541A0  |.  2BD8    sub     ebx, eax  
  004541A2  |.  3BDF    cmp     ebx, edi  
  004541A4  |.  77 32   ja      short 004541D8  
  004541A6  |.  8D4406 >lea     eax, [esi+eax+18]  
  004541AA  |.  8BCB    mov     ecx, ebx  
  004541AC  |.  8BD5    mov     edx, ebp  
  004541AE  |.  E8 652E>call    00407018  
  004541B3  |.  8BD6    mov     edx, esi  
  004541B5  |.  8D46 18 lea     eax, [esi+18]  
  004541B8  |.  E8 4FF8>call    00453A0C           ;  算法 call, F7 跟进  
  004541BD  |.  EB 0E   jmp     short 004541CD     ;  算法完后返回到此  
  /////////////////////////////////////////////////////////////// F7 跟进 call 00453A0C  

  00453A0C  /$  53      push    ebx  
  00453A0D  |.  56      push    esi  
  00453A0E  |.  57      push    edi  
  00453A0F  |.  55      push    ebp  
  00453A10  |.  83C4 A8 add     esp, -58  
  00453A13  |.  895424 >mov     [esp+4], edx  
  00453A17  |.  890424  mov     [esp], eax  
  00453A1A  |.  8D5C24 >lea     ebx, [esp+8]  
  00453A1E  |.  8D7424 >lea     esi, [esp+C]  
  00453A22  |.  8D7C24 >lea     edi, [esp+10]  
  00453A26  |.  8D6C24 >lea     ebp, [esp+14]  
  00453A2A  |.  8D5424 >lea     edx, [esp+18]  
  00453A2E  |.  B9 4000>mov     ecx, 40  
  00453A33  |.  8B0424  mov     eax, [esp]  
  00453A36  |.  E8 5DFF>call    00453998  
  00453A3B  |.  8B4424 >mov     eax, [esp+4]  
  00453A3F  |.  8B00    mov     eax, [eax]         ;  [EAX]=A  
  00453A41  |.  8903    mov     [ebx], eax  
  00453A43  |.  8B4424 >mov     eax, [esp+4]  
  00453A47  |.  8B40 04 mov     eax, [eax+4]       ;  EAX+4\]=B  
  00453A4A  |.  8906    mov     [esi], eax  
  00453A4C  |.  8B4424 >mov     eax, [esp+4]  
  00453A50  |.  8B40 08 mov     eax, [eax+8]       ;  [EAX+8]=C  
  00453A53  |.  8907    mov     [edi], eax  
  00453A55  |.  8B4424 >mov     eax, [esp+4]  
  00453A59  |.  8B40 0C mov     eax, [eax+C]       ;  [EAX+C]=D  
  00453A5C  |.  8945 00 mov     [ebp], eax  
  00453A5F  |.  8B45 00 mov     eax, [ebp]  
  00453A62  |.  50      push    eax                 ; /D 压栈  
  00453A63  |.  8B4424 >mov     eax, [esp+1C]       ; |\[ESP+1C\]=用户名前 4 位 16 进制数  
  00453A67  |.  50      push    eax                 ; |Arg3=696C6473  
  00453A68  |.  6A 07   push    7                   ; |Arg2 = 00000007  
  00453A6A  |.  68 78A4>push    D76AA478             ; |Arg1 = D76AA478  
  00453A6F  |.  8BC3    mov     eax, ebx            ; |  
  00453A71  |.  8B0F    mov     ecx, [edi]          ; |\[EDI\]=C  
  00453A73  |.  8B16    mov     edx, [esi]          ; |\[ESI\]=B  
  00453A75  |.  E8 4EFE>call    004538C8            ; \\MD5 第 1 次数据处理  F7 跟进  
  00453A7A  |.  8B07    mov     eax, [edi]          ;  Md5 第一次处理完成后继续处理  
  00453A7C  |.  50      push    eax                 ; /Arg4  
  00453A7D  |.  8B4424 >mov     eax, [esp+20]       ; |  
  00453A81  |.  50      push    eax                 ; |Arg3  
  00453A82  |.  6A 0C   push    0C                  ; |Arg2 = 0000000C  
  00453A84  |.  68 56B7>push    E8C7B756             ; |Arg1 = E8C7B756  
  00453A89  |.  8BC5    mov     eax, ebp            ; |  
  00453A8B  |.  8B0E    mov     ecx, [esi]          ; |  
  00453A8D  |.  8B13    mov     edx, [ebx]          ; |  
  00453A8F  |.  E8 34FE>call    004538C8            ; \\MD5 第 2 次数据处理  
  00453A94  |.  8B06    mov     eax, [esi]  
  00453A96  |.  50      push    eax                 ; /Arg4  
  00453A97  |.  8B4424 >mov     eax, [esp+24]       ; |  
  00453A9B  |.  50      push    eax                 ; |Arg3  
  00453A9C  |.  6A 11   push    11                  ; |Arg2 = 00000011  
  00453A9E  |.  68 DB70>push    242070DB             ; |Arg1 = 242070DB  
  00453AA3  |.  8BC7    mov     eax, edi            ; |  
  00453AA5  |.  8B0B    mov     ecx, [ebx]          ; |  
  00453AA7  |.  8B55 00 mov     edx, [ebp]          ; |  
  00453AAA  |.  E8 19FE>call    004538C8            ; \\MD5.第 3 次数据处理  
  00453AAF  |.  8B03    mov     eax, [ebx]  
  00453AB1  |.  50      push    eax                 ; /Arg4  
  00453AB2  |.  8B4424 >mov     eax, [esp+28]       ; |  
  00453AB6  |.  50      push    eax                 ; |Arg3  
  00453AB7  |.  6A 16   push    16                  ; |Arg2 = 00000016  
  00453AB9  |.  68 EECE>push    C1BDCEEE             ; |Arg1 = C1BDCEEE  
  00453ABE  |.  8BC6    mov     eax, esi            ; |  
  00453AC0  |.  8B4D 00 mov     ecx, [ebp]          ; |  
  00453AC3  |.  8B17    mov     edx, [edi]          ; |  
  00453AC5  |.  E8 FEFd>call    004538C8            ; \\MD5.第 4 次数据处理  
  //////////////////////////////////////////////// F7 跟进 call    004538C8  

  004538C8  /$  55      push    ebp  
  004538C9  |.  8BEC    mov     ebp, esp  
  004538CB  |.  53      push    ebx  
  004538CC  |.  56      push    esi  
  004538CD  |.  57      push    edi  
  004538CE  |.  8BF9    mov     edi, ecx  
  004538D0  |.  8BF2    mov     esi, edx  
  004538D2  |.  8BD8    mov     ebx, eax  
  004538D4  |.  8B4D 14 mov     ecx, [ebp+14]  
  004538D7  |.  8BD7    mov     edx, edi  
  004538D9  |.  8BC6    mov     eax, esi  
  004538DB  |.  E8 9CFF>call    0045387C             ;  关键 call F7 跟进  
  004538E0  |.  0345 10 add     eax, [ebp+10]        ;  EAX=C+696c6473=02274171  
  004538E3  |.  0345 08 add     eax, [ebp+8]         ;  EAX=2274171+D76AA478=D991E5E9  
  004538E6  |.  0103    add     [ebx], eax  
  004538E8  |.  8BC3    mov     eax, ebx  
  004538EA  |.  8A55 0C mov     dl, [ebp+C]          ;  7 入 dl  
  004538ED  |.  E8 B6FF>call    004538A8             ;  关键 call F7 进入  
  004538F2  |.  0133    add     [ebx], esi           ;  [ebx]=6B847520 and esi(=EFCDAB89)  
  004538F4  |.  5F      pop     edi  
  004538F5  |.  5E      pop     esi  
  004538F6  |.  5B      pop     ebx  
  004538F7  |.  5D      pop     ebp  
  004538F8  \\.  C2 1000 retn    10  
  /////////////////////////////////////////////// F7 跟进 call    0045387C  

  0045387C  /$  23D0    and     edx, eax            ;  EDX=C and  EAX=B  =88888888  
  0045387E  |.  F7D0    not     eax                ;  eax 取反=D  
  00453880  |.  23C8    and     ecx, eax            ;  ECX=D and D =10325476  
  00453882  |.  0BD1    or      edx, ecx            ;  EDX=88888888 or D =98BADCFE  
  00453884  |.  8BC2    mov     eax, edx  
  00453886  \\.  C3      ret                       ;  返回  
  //////////////////////////////////////////////F7 进入 call    004538A8  

  004538A8  /$  53      push    ebx  
  004538A9  |.  33C9    xor     ecx, ecx  
  004538AB  |.  8ACA    mov     cl, dl  
  004538AD  |.  51      push    ecx  
  004538AE  |.  B9 2000>mov     ecx, 20              ;  20 入 ecx  
  004538B3  |.  5B      pop     ebx  
  004538B4  |.  2BCB    sub     ecx, ebx              ;  ecx=20-7  
  004538B6  |.  8B18    mov     ebx, [eax]            ;  [eax]=40d708EA 入 ebx  
  004538B8  |.  D3EB    shr     ebx, cl               ;  ebx shr 19  =20  
  004538BA  |.  8BCA    mov     ecx, edx  
  004538BC  |.  8B10    mov     edx, [eax]            ;  [eax]=40d708EA 入 edx  
```
