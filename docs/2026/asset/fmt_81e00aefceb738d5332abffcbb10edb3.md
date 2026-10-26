
```
00493616   .  8D85 0CFFFFFF lea     eax, dword ptr [ebp-F4]

0049361C   .  50            push    eax

0049361D   .  8D4D AC       lea     ecx, dword ptr [ebp-54]

00493620   .  51            push    ecx

00493621   .  FFD3          call    ebx

00493623   .  50            push    eax

00493624   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

0049362A   .  DD9D 24FCFFFF fstp    qword ptr [ebp-3DC]              ;  “7”

00493630   .  8D95 9CFEFFFF lea     edx, dword ptr [ebp-164]

00493636   .  52            push    edx

00493637   .  6A 01         push    1                                ;  取第一位

00493639   .  8D45 BC       lea     eax, dword ptr [ebp-44]

0049363C   .  50            push    eax

0049363D   .  8D8D 8CFEFFFF lea     ecx, dword ptr [ebp-174]

00493643   .  C785 A4FEFFFF>mov     dword ptr [ebp-15C], 1

0049364D   .  C785 9CFEFFFF>mov     dword ptr [ebp-164], 2

00493657   .  51            push    ecx

00493658   .  FFD7          call    edi

0049365A   .  8D95 8CFEFFFF lea     edx, dword ptr [ebp-174]

00493660   .  52            push    edx

00493661   .  8D45 A4       lea     eax, dword ptr [ebp-5C]

00493664   .  50            push    eax

00493665   .  FFD3          call    ebx

00493667   .  50            push    eax

00493668   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

0049366E   .  DD9D 1CFCFFFF fstp    qword ptr [ebp-3E4]              ;  “5”

00493674   .  8D8D 7CFEFFFF lea     ecx, dword ptr [ebp-184]

0049367A   .  51            push    ecx

0049367B   .  B8 02000000   mov     eax, 2                           ;  这里又显示要取两位了。

00493680   .  6A 07         push    7                                ;  从第7位开始取

00493682   .  8D55 BC       lea     edx, dword ptr [ebp-44]

00493685   .  8985 84FEFFFF mov     dword ptr [ebp-17C], eax

0049368B   .  8985 7CFEFFFF mov     dword ptr [ebp-184], eax

00493691   .  52            push    edx

00493692   .  8D85 6CFEFFFF lea     eax, dword ptr [ebp-194]

00493698   .  50            push    eax

00493699   .  FFD7          call    edi

0049369B   .  8D8D 6CFEFFFF lea     ecx, dword ptr [ebp-194]

004936A1   .  51            push    ecx

004936A2   .  8D55 A0       lea     edx, dword ptr [ebp-60]

004936A5   .  52            push    edx

004936A6   .  FFD3          call    ebx

004936A8   .  50            push    eax                              ;  取得“08”

004936A9   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

004936AF   .  DD9D 14FCFFFF fstp    qword ptr [ebp-3EC]              ;  “8”转成实数当然去掉0

004936B5   .  8D85 5CFEFFFF lea     eax, dword ptr [ebp-1A4]

004936BB   .  50            push    eax

004936BC   .  6A 0C         push    0C                               ;  取第12位

004936BE   .  8D4D BC       lea     ecx, dword ptr [ebp-44]

004936C1   .  51            push    ecx

004936C2   .  8D95 4CFEFFFF lea     edx, dword ptr [ebp-1B4]

004936C8   .  52            push    edx

004936C9   .  C785 64FEFFFF>mov     dword ptr [ebp-19C], 1

004936D3   .  C785 5CFEFFFF>mov     dword ptr [ebp-1A4], 2

004936DD   .  FFD7          call    edi

004936DF   .  8D85 4CFEFFFF lea     eax, dword ptr [ebp-1B4]

004936E5   .  50            push    eax

004936E6   .  8D4D 9C       lea     ecx, dword ptr [ebp-64]

004936E9   .  51            push    ecx

004936EA   .  FFD3          call    ebx

004936EC   .  50            push    eax

004936ED   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

004936F3   .  DD9D 0CFCFFFF fstp    qword ptr [ebp-3F4]              ;  7

004936F9   .  8D95 CCFDFFFF lea     edx, dword ptr [ebp-234]

004936FF   .  52            push    edx

00493700   .  6A 01         push    1                                ;  取第一位

00493702   .  8D45 BC       lea     eax, dword ptr [ebp-44]

00493705   .  50            push    eax

00493706   .  8D8D BCFDFFFF lea     ecx, dword ptr [ebp-244]

0049370C   .  51            push    ecx

0049370D   .  C785 D4FDFFFF>mov     dword ptr [ebp-22C], 1

00493717   .  C785 CCFDFFFF>mov     dword ptr [ebp-234], 2

00493721   .  FFD7          call    edi

00493723   .  8D95 BCFDFFFF lea     edx, dword ptr [ebp-244]

00493729   .  52            push    edx

0049372A   .  8D45 94       lea     eax, dword ptr [ebp-6C]

0049372D   .  50            push    eax

0049372E   .  FFD3          call    ebx

00493730   .  50            push    eax

00493731   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

00493737   .  DD9D 04FCFFFF fstp    qword ptr [ebp-3FC]              ;  5

0049373D   .  8D8D ACFDFFFF lea     ecx, dword ptr [ebp-254]

00493743   .  51            push    ecx

00493744   .  B8 02000000   mov     eax, 2                           ;  又取两位

00493749   .  6A 04         push    4                                ;  从第四位开始

0049374B   .  8D55 BC       lea     edx, dword ptr [ebp-44]

0049374E   .  8985 B4FDFFFF mov     dword ptr [ebp-24C], eax

00493754   .  8985 ACFDFFFF mov     dword ptr [ebp-254], eax

0049375A   .  52            push    edx

0049375B   .  8D85 9CFDFFFF lea     eax, dword ptr [ebp-264]

00493761   .  50            push    eax

00493762   .  FFD7          call    edi

00493764   .  8D8D 9CFDFFFF lea     ecx, dword ptr [ebp-264]

0049376A   .  51            push    ecx

0049376B   .  8D55 90       lea     edx, dword ptr [ebp-70]

0049376E   .  52            push    edx

0049376F   .  FFD3          call    ebx

00493771   .  50            push    eax                              ;  得到“02”

00493772   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

00493778   .  DD9D FCFBFFFF fstp    qword ptr [ebp-404]              ;  "2"转成实数，02就是2

0049377E   .  8D85 8CFDFFFF lea     eax, dword ptr [ebp-274]

00493784   .  50            push    eax

00493785   .  6A 0A         push    0A                               ;  又取第十位

00493787   .  8D4D BC       lea     ecx, dword ptr [ebp-44]

0049378A   .  51            push    ecx

0049378B   .  8D95 7CFDFFFF lea     edx, dword ptr [ebp-284]

00493791   .  52            push    edx

00493792   .  C785 94FDFFFF>mov     dword ptr [ebp-26C], 1

0049379C   .  C785 8CFDFFFF>mov     dword ptr [ebp-274], 2

004937A6   .  FFD7          call    edi

004937A8   .  8D85 7CFDFFFF lea     eax, dword ptr [ebp-284]

004937AE   .  50            push    eax

004937AF   .  8D4D 8C       lea     ecx, dword ptr [ebp-74]

004937B2   .  51            push    ecx

004937B3   .  FFD3          call    ebx

004937B5   .  50            push    eax

004937B6   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

004937BC   .  DD9D F4FBFFFF fstp    qword ptr [ebp-40C]              ;  “8”

004937C2   .  8D95 7CFFFFFF lea     edx, dword ptr [ebp-84]          ;  终于取完了。

004937C8   .  52            push    edx                              ;  下面开始对取出来的数进行运算

004937C9   .  6A 05         push    5                                ;  注意啊，这里还要取第五位。

004937CB   .  8D45 BC       lea     eax, dword ptr [ebp-44]

004937CE   .  50            push    eax

004937CF   .  8D8D 6CFFFFFF lea     ecx, dword ptr [ebp-94]

004937D5   .  51            push    ecx

004937D6   .  C745 84 01000>mov     dword ptr [ebp-7C], 1

004937DD   .  C785 7CFFFFFF>mov     dword ptr [ebp-84], 2

004937E7   .  FFD7          call    edi

004937E9   .  8D95 6CFFFFFF lea     edx, dword ptr [ebp-94]

004937EF   .  52            push    edx

004937F0   .  8D45 B8       lea     eax, dword ptr [ebp-48]

004937F3   .  50            push    eax

004937F4   .  FFD3          call    ebx                              ;  我的机器码：577020080877

004937F6   .  50            push    eax                              ;  下面是取得的第五位直接参与运算。

004937F7   .  FF15 9C124000 call    dword ptr [<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

004937FD   .  DC8D 34FCFFFF fmul    qword ptr [ebp-3CC]              ;  2\*8
```
