
对解密后的`0x68`个字节`0x67`个字节进行解码：

```
  004E08DE   .  833D 08336D00>cmp     dword ptr [6D3308], 1
  004E08E5   .  7F 23         jg      short 004E090A
  004E08E7   .  6A 10         push    10                               ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
  004E08E9   .  A1 34636C00   mov     eax, dword ptr [6C6334]          ; |
  004E08EE   .  50            push    eax                              ; |Title => "Total Commander"
  004E08EF   .  68 B80F4E00   push    004E0FB8                         ; |using cracks is unfair!\n\nplease get the 
                                                                       ; |official release from 7c4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6Z5K9i4y4D9k6i4u0Q4x3X3g2U0L8$3#2Q4x3X3f1`.
  004E08F4   .  6A 00         push    0                                ; |hOwner = NULL
  004E08F6   .  E8 4559F2FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
  004E08FB   .  B8 01000000   mov     eax, 1
  004E0900   .  E8 DB3DF2FF   call    004046E0
  004E0905   .  E9 AA050000   jmp     004E0EB4
  004E090A   >  8D95 6CFEFFFF lea     edx, dword ptr [ebp-194]
  004E0910   .  B9 67000000   mov     ecx, 67
  004E0915   .  8B85 E0FEFFFF mov     eax, dword ptr [ebp-120]
  004E091B   .  E8 98C2FFFF   call    <BitOutBytes>                    ;  0x67字节大数转为字节数组，高位字节抛弃
  004E0920   .  8D85 6CFEFFFF lea     eax, dword ptr [ebp-194]
  004E0926   .  B9 66000000   mov     ecx, 66                          ;  0x67个字节解码
  004E092B      BA BCA42000   mov     edx, 20A4BC                      ;  初始值
  004E0930   .  E8 17FAFFFF   call    <Decode>                         ;  解码

  004E034C >/$  55            push    ebp                              ;  Decode
  004E034D  |.  8BEC          mov     ebp, esp
  004E034F  |.  83C4 F8       add     esp, -8
  004E0352  |.  53            push    ebx
  004E0353  |.  56            push    esi
  004E0354  |.  57            push    edi
  004E0355  |.  8BF9          mov     edi, ecx                         ;  edi=解码字节长度-1
  004E0357  |.  8945 FC       mov     dword ptr [ebp-4], eax           ;  [ebp-4]=要解码的字节
  004E035A  |.  8915 2CE06C00 mov     dword ptr [6CE02C], edx          ;  [6CE02C]=初始值
  004E0360      C745 F8 14000>mov     dword ptr [ebp-8], 14
  004E0367      8BF7          mov     esi, edi
  004E0369      85F6          test    esi, esi
  004E036B  |.  7C 1C         |jl      short 004E0389
  004E036D  |.  46            |inc     esi
  004E036E  |.  8B5D FC       |mov     ebx, dword ptr [ebp-4]
  004E0371  |>  8D47 01       |/lea     eax, dword ptr [edi+1]         ;  eax=解码字节长度，解码开始，编码反着来
  004E0374  |.  E8 6327F2FF   ||call    <RandInt>                      ;  计算[6CEO2C]
  004E0379  |.  8B55 FC       ||mov     edx, dword ptr [ebp-4]
  004E037C  |.  03D0          ||add     edx, eax
  004E037E  |.  8BC3          ||mov     eax, ebx
  004E0380  |.  E8 BBFFFFFF   ||call    <ByteSwap>
  004E0385  |.  43            ||inc     ebx
  004E0386  |.  4E            ||dec     esi
  004E0387  |.^ 75 E8         |\jnz     short 004E0371
  004E0389  |>  8BF7          |mov     esi, edi
  004E038B  |.  85F6          |test    esi, esi
  004E038D  |.  7C 2B         |jl      short 004E03BA
  004E038F  |.  46            |inc     esi
  004E0390  |.  8B5D FC       |mov     ebx, dword ptr [ebp-4]
  004E0393  |>  B8 08000000   |/mov     eax, 8
  004E0398  |.  E8 3F27F2FF   ||call    <RandInt>
  004E039D  |.  8BD0          ||mov     edx, eax
  004E039F  |.  8BC3          ||mov     eax, ebx
  004E03A1  |.  E8 76FFFFFF   ||call    <ROR8>                         ;  ror8(x,n) x>>n|x<<(8-n)
  004E03A6  |.  B8 00010000   ||mov     eax, 100
  004E03AB  |.  E8 2C27F2FF   ||call    <RandInt>
  004E03B0  |.  8A13          ||mov     dl, byte ptr [ebx]
  004E03B2  |.  32C2          ||xor     al, dl
  004E03B4  |.  8803          ||mov     byte ptr [ebx], al
  004E03B6  |.  43            ||inc     ebx
  004E03B7  |.  4E            ||dec     esi
  004E03B8  |.^ 75 D9         |\jnz     short 004E0393
  004E03BA  |>  FF4D F8       |dec     dword ptr [ebp-8]               ;  循环了0x14次，有0x14*length*3个[6CE02C]要计算
  004E03BD  |.^ 75 A8         \jnz     short 004E0367
  004E03BF  |.  5F            pop     edi
  004E03C0  |.  5E            pop     esi
  004E03C1  |.  5B            pop     ebx
  004E03C2  |.  59            pop     ecx
  004E03C3  |.  59            pop     ecx
  004E03C4  |.  5D            pop     ebp
  004E03C5  \.  C3            retn
```

其中`004E0374`处的`RandInt`出现了问题，不知道是不是bug

```
  00402ADC >    6915 2CE06C00>imul    edx, dword ptr [6CE02C], 8088405 ;  RandInt
  00402AE6      42            inc     edx                              ;  当[6CE02C]==0x20A48C时，某些地址计算不对
  00402AE7      8915 2CE06C00 mov     dword ptr [6CE02C], edx
  00402AED      F7E2          mul     edx
  00402AEF      89D0          mov     eax, edx
  00402AF1      C3            retn
```

对解码的`0x67`个字节后`0x57`个字节进行`MD5`计算，必须等于前`0x10`个字节，`MD5`已变异

```
004E0935   .  33C9          xor     ecx, ecx
  004E0937   .  B2 01         mov     dl, 1
  004E0939   .  B8 C06F4700   mov     eax, 00476FC0
  004E093E   .  E8 BD00F3FF   call    00410A00
  004E0943   .  8BF0          mov     esi, eax
  004E0945   .  8D85 7CFEFFFF lea     eax, dword ptr [ebp-184]         ;  0x67解码字节偏移0x10
  004E094B   .  8946 20       mov     dword ptr [esi+20], eax
  004E094E   .  8D85 5CFAFFFF lea     eax, dword ptr [ebp-5A4]         ;  MD5存放处
  004E0954   .  8946 24       mov     dword ptr [esi+24], eax
  004E0957   .  C746 28 57000>mov     dword ptr [esi+28], 57           ;  0x57个字节
  004E095E   .  8BC6          mov     eax, esi
  004E0960   .  E8 CF66F9FF   call    <MD5Init>
  004E0965   .  33D2          xor     edx, edx
  004E0967   .  8BC6          mov     eax, esi
  004E0969   .  E8 5275F9FF   call    <CustomMD5>
  004E096E   .  8BC6          mov     eax, esi
  004E0970   .  E8 4B76F9FF   call    <HashOutput>
  004E0975   .  8BC6          mov     eax, esi
  004E0977   .  E8 CC23F2FF   call    <Free>
  004E097C   .  8D85 5CFAFFFF lea     eax, dword ptr [ebp-5A4]         ;  MD5值
  004E0982   .  8D95 6CFEFFFF lea     edx, dword ptr [ebp-194]         ;  0x67解码字节前0x10字节
  004E0988   .  B9 10000000   mov     ecx, 10
  004E098D   .  E8 BA20F2FF   call    <StrCmp>
  004E0992      0F84 AE000000 je      004E0A46                         ;  必须相等，否则挂
```

`MD5`主要变异在消息的填充和函数上
