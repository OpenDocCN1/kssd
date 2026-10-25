
```asm
:004AA226 D8C8                    fmul st(0), st(0)

:004AA228 DBAD48FFFFFF            fld tbyte ptr [ebp+FFFFFF48]

:004AA22E DEC1                    faddp st(1), st(0)

:004AA230 D9FA                    fsqrt

:004AA232 DD5DD0                  fstp qword ptr [ebp-30]

:004AA235 9B                      wait

:004AA236 6A00                    push 00000000

:004AA238 6A06                    push 00000006

:004AA23A 6A00                    push 00000000

:004AA23C 68A9A52513              push 1325A5A9

:004AA241 8B45D8                  mov eax, dword ptr [ebp-28]

:004AA244 8B55DC                  mov edx, dword ptr [ebp-24]

:004AA247 0FACD001                shrd eax, edx, 01

:004AA24B D1EA                    shr edx, 1

:004AA24D E862B2F5FF              call 004054B4

:004AA252 E839B2F5FF              call 00405490

:004AA257 81F0640AB302            xor eax, 02B30A64

:004AA25D 81F200000000            xor edx, 00000000

:004AA263 52                      push edx

:004AA264 50                      push eax

:004AA265 A1FCED4A00              mov eax, dword ptr [004AEDFC]

:004AA26A 33D2                    xor edx, edx

:004AA26C 3B542404                cmp edx, dword ptr [esp+04]

:004AA270 7503                    jne 004AA275

:004AA272 3B0424                  cmp eax, dword ptr [esp]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AA270(C)

|

:004AA275 5A                      pop edx

:004AA276 58                      pop eax

:004AA277 0F844B020000            je 004AA4C8

:004AA27D 8B45D8                  mov eax, dword ptr [ebp-28]

:004AA280 8B55DC                  mov edx, dword ptr [ebp-24]

:004AA283 81F0A32F1A02            xor eax, 021A2FA3

:004AA289 81F200000000            xor edx, 00000000

:004AA28F 81F0A7CDB004            xor eax, 04B0CDA7

:004AA295 81F200000000            xor edx, 00000000

:004AA29B 0DA00A8002              or eax, 02800AA0

:004AA2A0 83FA00                  cmp edx, 00000000

:004AA2A3 7509                    jne 004AA2AE

:004AA2A5 83F800                  cmp eax, 00000000

:004AA2A8 0F8482020000            je 004AA530

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AA2A3(C)

|

:004AA2AE A1FCED4A00              mov eax, dword ptr [004AEDFC]

:004AA2B3 3506BC0D02              xor eax, 020DBC06

:004AA2B8 35630AB302              xor eax, 02B30A63

:004AA2BD 33D2                    xor edx, edx

:004AA2BF 3B55E4                  cmp edx, dword ptr [ebp-1C]

:004AA2C2 7509                    jne 004AA2CD

:004AA2C4 3B45E0                  cmp eax, dword ptr [ebp-20]

:004AA2C7 0F84C7FBFFFF            je 004A9E94

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AA2C2(C)

|

:004AA2CD A1FCED4A00              mov eax, dword ptr [004AEDFC]

:004AA2D2 3559600500              xor eax, 00056059

:004AA2D7 33D2                    xor edx, edx

:004AA2D9 52                      push edx

:004AA2DA 50                      push eax

:004AA2DB 8B45D8                  mov eax, dword ptr [ebp-28]

:004AA2DE 8B55DC                  mov edx, dword ptr [ebp-24]

:004AA2E1 81F0F25F3400            xor eax, 00345FF2

:004AA2E7 81F200000000            xor edx, 00000000

:004AA2ED 81F0640AB302            xor eax, 02B30A64

:004AA2F3 81F200000000            xor edx, 00000000

:004AA2F9 3B542404                cmp edx, dword ptr [esp+04]

:004AA2FD 7503                    jne 004AA302

:004AA2FF 3B0424                  cmp eax, dword ptr [esp]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AA2FD(C)

|

:004AA302 5A                      pop edx

:004AA303 58                      pop eax

:004AA304 0F84D0010000            je 004AA4DA

///////////////////////////////////正式的比较/////////////////////////////////////////////

:004AA30A 8B45B0                  mov eax, dword ptr [ebp-50]    //EAX=移位之后的值(10706279)

:004AA30D 8A4002                  mov al, byte ptr [eax+02]      //取第3位

:004AA310 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]  //EDX=机器码(2654328163)

:004AA316 3A4202                  cmp al, byte ptr [edx+02]  //第3位比较

:004AA319 0F85E5030000            jne 004AA704

:004AA31F 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA322 8A00                    mov al, byte ptr [eax]  //取第1位

:004AA324 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA32A 3A02                    cmp al, byte ptr [edx]  //第1位比较

:004AA32C 0F85D2030000            jne 004AA704

:004AA332 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA335 8A4004                  mov al, byte ptr [eax+04]

:004AA338 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA33E 3A4204                  cmp al, byte ptr [edx+04]      //第5位比较

:004AA341 0F85BD030000            jne 004AA704

:004AA347 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA34A 8A4006                  mov al, byte ptr [eax+06]

:004AA34D 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA353 3A4206                  cmp al, byte ptr [edx+06]  //第7位比较

:004AA356 0F85A8030000            jne 004AA704

:004AA35C 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA35F 8A4003                  mov al, byte ptr [eax+03]

:004AA362 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA368 3A4203                  cmp al, byte ptr [edx+03]  //第4位比较

:004AA36B 0F8593030000            jne 004AA704

:004AA371 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA374 8A4005                  mov al, byte ptr [eax+05]

:004AA377 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA37D 3A4205                  cmp al, byte ptr [edx+05]  //第6位比较

:004AA380 0F857E030000            jne 004AA704

:004AA386 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA389 8A4001                  mov al, byte ptr [eax+01]

:004AA38C 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA392 3A4201                  cmp al, byte ptr [edx+01]  //第2位比较

:004AA395 0F8569030000            jne 004AA704

:004AA39B 8B45B0                  mov eax, dword ptr [ebp-50]

:004AA39E 8A4007                  mov al, byte ptr [eax+07]

:004AA3A1 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]

:004AA3A7 3A4207                  cmp al, byte ptr [edx+07]  //第8位比较

:004AA3AA 0F8580010000            jne 004AA530

//这里说明参与比较的机器码只有前8位,在求逆的时候用的着;

:004AA3B0 A1FCED4A00              mov eax, dword ptr [004AEDFC]

:004AA3B5 35BD44F001              xor eax, 01F044BD

:004AA3BA 83C00E                  add eax, 0000000E

:004AA3BD 33D2                    xor edx, edx

:004AA3BF 52                      push edx

:004AA3C0 50                      push eax

:004AA3C1 8B45D8                  mov eax, dword ptr [ebp-28]

:004AA3C4 8B55DC                  mov edx, dword ptr [ebp-24]

:004AA3C7 81F0C430C302            xor eax, 02C330C4

:004AA3CD 81F200000000            xor edx, 00000000

:004AA3D3 81F0A32F1A02            xor eax, 021A2FA3

:004AA3D9 81F200000000            xor edx, 00000000

:004AA3DF 3B542404                cmp edx, dword ptr [esp+04]

:004AA3E3 7503                    jne 004AA3E8

:004AA3E5 3B0424                  cmp eax, dword ptr [esp]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AA3E3(C)

|

:004AA3E8 5A                      pop edx

:004AA3E9 58                      pop eax

:004AA3EA 0F8414030000            je 004AA704

:004AA3F0 B201                    mov dl, 01

:004AA3F2 A12C354700              mov eax, dword ptr [0047352C]

:004AA3F7 E83092FCFF              call 0047362C

:004AA3FC 8945F8                  mov dword ptr [ebp-08], eax

:004AA3FF BA02000080              mov edx, 80000002

:004AA404 8B45F8                  mov eax, dword ptr [ebp-08]

:004AA407 E8C092FCFF              call 004736CC

:004AA40C 33C9                    xor ecx, ecx

                                                              //对要保存在注册表里的数据进行变形
```
