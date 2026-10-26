
```
  004592CA  |.  BA 02000000        mov     edx, 2

  004592CF  |.  E8 ECDE0900        call    004F71C0

  004592D4  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004592D7  |.  8D85 44FFFFFF      lea     eax, dword ptr [ebp-BC]

  004592DD  |.  BA 02000000        mov     edx, 2

  004592E2  |.  E8 D9DE0900        call    004F71C0

  004592E7  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004592EA  |.  8D85 48FFFFFF      lea     eax, dword ptr [ebp-B8]

  004592F0  |.  BA 02000000        mov     edx, 2

  004592F5  |.  E8 C6DE0900        call    004F71C0

  004592FA  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004592FD  |.  8D85 4CFFFFFF      lea     eax, dword ptr [ebp-B4]

  00459303  |.  BA 02000000        mov     edx, 2

  00459308  |.  E8 B3DE0900        call    004F71C0

  0045930D  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459310  |.  8D85 50FFFFFF      lea     eax, dword ptr [ebp-B0]

  00459316  |.  BA 02000000        mov     edx, 2

  0045931B  |.  E8 A0DE0900        call    004F71C0

  00459320  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459323  |.  8D85 54FFFFFF      lea     eax, dword ptr [ebp-AC]

  00459329  |.  BA 02000000        mov     edx, 2

  0045932E  |.  E8 8DDE0900        call    004F71C0

  00459333  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459336  |.  8D85 58FFFFFF      lea     eax, dword ptr [ebp-A8]

  0045933C  |.  BA 02000000        mov     edx, 2

  00459341  |.  E8 7ADE0900        call    004F71C0

  00459346  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459349  |.  8D85 5CFFFFFF      lea     eax, dword ptr [ebp-A4]

  0045934F  |.  BA 02000000        mov     edx, 2

  00459354  |.  E8 67DE0900        call    004F71C0

  00459359  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  0045935C  |.  8D85 60FFFFFF      lea     eax, dword ptr [ebp-A0]

  00459362  |.  BA 02000000        mov     edx, 2

  00459367  |.  E8 54DE0900        call    004F71C0

  0045936C  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  0045936F  |.  8D85 64FFFFFF      lea     eax, dword ptr [ebp-9C]

  00459375  |.  BA 02000000        mov     edx, 2

  0045937A  |.  E8 41DE0900        call    004F71C0

  0045937F  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459382  |.  8D85 68FFFFFF      lea     eax, dword ptr [ebp-98]

  00459388  |.  BA 02000000        mov     edx, 2

  0045938D  |.  E8 2EDE0900        call    004F71C0

  00459392  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459395  |.  8D85 6CFFFFFF      lea     eax, dword ptr [ebp-94]

  0045939B  |.  BA 02000000        mov     edx, 2

  004593A0  |.  E8 1BDE0900        call    004F71C0

  004593A5  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004593A8  |.  8D85 70FFFFFF      lea     eax, dword ptr [ebp-90]

  004593AE  |.  BA 02000000        mov     edx, 2

  004593B3  |.  E8 08DE0900        call    004F71C0

  004593B8  |.  EB 3E              jmp     short 004593F8

  004593BA  |>  FF4B 1C            dec     dword ptr [ebx+1C]

  004593BD  |.  8D45 F4            lea     eax, dword ptr [ebp-C]

  004593C0  |.  BA 02000000        mov     edx, 2

  004593C5  |.  E8 F6DD0900        call    004F71C0

  004593CA  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004593CD  |.  8D45 F8            lea     eax, dword ptr [ebp-8]

  004593D0  |.  BA 02000000        mov     edx, 2

  004593D5  |.  E8 E6DD0900        call    004F71C0

  004593DA  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004593DD  |.  8D45 FC            lea     eax, dword ptr [ebp-4]

  004593E0  |.  BA 02000000        mov     edx, 2

  004593E5  |.  E8 D6DD0900        call    004F71C0

  004593EA  |.  8B0B               mov     ecx, dword ptr [ebx]

  004593EC  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx

  004593F3  |.  E9 6D030000        jmp     00459765                         ;  直接返回失败信息

  004593F8  |>  66:C743 10 7400    mov     word ptr [ebx+10], 74

  004593FE  |.  33C0               xor     eax, eax

  00459400  |.  B2 01              mov     dl, 1

  00459402  |.  8945 F0            mov     dword ptr [ebp-10], eax

  00459405  |.  FF43 1C            inc     dword ptr [ebx+1C]

  00459408  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8

  0045940E  |.  A1 20E75500        mov     eax, dword ptr [<&VCL50.Registry>

  00459413  |.  E8 52300B00        call    <jmp.&VCL50.Registry::TRegistry:>;  准备写入注册表

  00459418  |.  8985 DCFEFFFF      mov     dword ptr [ebp-124], eax

  0045941E  |.  8D85 3CFFFFFF      lea     eax, dword ptr [ebp-C4]

  00459424  |.  66:C743 10 8000    mov     word ptr [ebx+10], 80

  0045942A  |.  8D97 46010000      lea     edx, dword ptr [edi+146]

  00459430  |.  E8 E7DB0900        call    004F701C

  00459435  |.  FF43 1C            inc     dword ptr [ebx+1C]

  00459438  |.  33C9               xor     ecx, ecx

  0045943A  |.  898D 38FFFFFF      mov     dword ptr [ebp-C8], ecx

  00459440  |.  8D8D 38FFFFFF      lea     ecx, dword ptr [ebp-C8]

  00459446  |.  FF43 1C            inc     dword ptr [ebx+1C]

  00459449  |.  8D95 3CFFFFFF      lea     edx, dword ptr [ebp-C4]

  0045944F  |.  B8 80945500        mov     eax, 00559480

  00459454  |.  E8 ABDD0900        call    004F7204                         ;  写入注册表地址

  00459459  |.  8D95 38FFFFFF      lea     edx, dword ptr [ebp-C8]

  0045945F  |.  8B12               mov     edx, dword ptr [edx]             ;  ASCII "Software\Aoqun\Stock Expert\2.0\Proxy"

  00459461  |.  33C9               xor     ecx, ecx

  00459463  |.  8B85 DCFEFFFF      mov     eax, dword ptr [ebp-124]

  00459469  |.  E8 E42F0B00        call    <jmp.&VCL50.Registry::TRegistry:>;  打开注册表

  0045946E  |.  8885 DBFEFFFF      mov     byte ptr [ebp-125], al

  00459474  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  00459477  |.  8D85 38FFFFFF      lea     eax, dword ptr [ebp-C8]

  0045947D  |.  BA 02000000        mov     edx, 2

  00459482  |.  E8 39DD0900        call    004F71C0

  00459487  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  0045948A  |.  8D85 3CFFFFFF      lea     eax, dword ptr [ebp-C4]

  00459490  |.  BA 02000000        mov     edx, 2

  00459495  |.  E8 26DD0900        call    004F71C0

  0045949A  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8

  004594A0  |.  80BD DBFEFFFF 00   cmp     byte ptr [ebp-125], 0

  004594A7  |.  0F84 F1000000      je      0045959E

  004594AD  |.  66:C743 10 8C00    mov     word ptr [ebx+10], 8C

  004594B3  |.  8D97 4D010000      lea     edx, dword ptr [edi+14D]         ;  UseProxy

  004594B9  |.  8D85 34FFFFFF      lea     eax, dword ptr [ebp-CC]

  004594BF  |.  E8 58DB0900        call    004F701C                         ;  写入注册版本类型标识

  004594C4  |.  FF43 1C            inc     dword ptr [ebx+1C]

  004594C7  |.  8B10               mov     edx, dword ptr [eax]

  004594C9  |.  8B85 DCFEFFFF      mov     eax, dword ptr [ebp-124]

  004594CF  |.  E8 3C2F0B00        call    <jmp.&VCL50.Registry::TRegistry:>;  注册表数据类型：DWORD值

  004594D4  |.  8885 E7FEFFFF      mov     byte ptr [ebp-119], al

  004594DA  |.  FF4B 1C            dec     dword ptr [ebx+1C]

  004594DD  |.  8D85 34FFFFFF      lea     eax, dword ptr [ebp-CC]

  004594E3  |.  BA 02000000        mov     edx, 2

  004594E8  |.  E8 D3DC0900        call    004F71C0                         ;  ProxyHost

  004594ED  |.  66:C743 10 9800    mov     word ptr [ebx+10], 98

  004594F3  |.  8D97 56010000      lea     edx, dword ptr [edi+156]

  004594F9  |.  8D85 30FFFFFF      lea     eax, dword ptr [ebp-D0]

  004594FF  |.  E8 18DB0900        call    004F701C

  00459504  |.  FF43 1C            inc     dword ptr [ebx+1C]

  00459507  |.  8B10               mov     edx, dword ptr [eax]

  00459509  |.  33C0               xor     eax, eax

  0045950B  |.  8985 2CFFFFFF      mov     dword ptr [ebp-D4], eax

  00459511  |.  8D8D 2CFFFFFF      lea     ecx, dword ptr [ebp-D4]

  00459517  |.  FF43 1C            inc     dword ptr [ebx+1C]
```
