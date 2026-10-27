
`:0072B90C 83C8FF`                  `or eax, FFFFFFFF`

`:0072B90F C705641BB20001000000`    `mov dword ptr [00B21B64], 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0072B8DE(C), :0072B8E4(C)

|

`:0072B919 803A8D`                  `cmp byte ptr [edx], 8D`

`:0072B91C 0F8590000000`            `jne 0072B9B2`

`:0072B922 807A01C0`                `cmp byte ptr [edx+01], C0`

`:0072B926 0F8586000000`            `jne 0072B9B2`

`:0072B92C 8B4604`                  `mov eax, dword ptr [esi+04]`                        ; ExceptionAddress 处是 8D C0

`:0072B92F 8B90B8000000`            `mov edx, dword ptr [eax+000000B8]`

`:0072B935 03D1`                    `add edx, ecx`

`:0072B937 8990B8000000`            `mov dword ptr [eax+000000B8], edx`                ; context->eip +=2

`:0072B93D 8B4604`                  `mov eax, dword ptr [esi+04]`

`:0072B940 8B88C0000000`            `mov ecx, dword ptr [eax+000000C0]`

`:0072B946 80E5F6`                  `and ch, F6`

`:0072B949 8988C0000000`            `mov dword ptr [eax+000000C0], ecx`                ; context->EFlags &= ~900 // 设置CPU单步 flags

`:0072B94F 83C8FF`                  `or eax, FFFFFFFF`

`:0072B952 891D641BB200`            `mov dword ptr [00B21B64], ebx`

`:0072B958 EB58`                    `jmp 0072B9B2`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072B706(C)

|

`:0072B95A 81E995C00000`            `sub ecx, 0000C095`

`:0072B960 7440`                    `je 0072B9A2`

`:0072B962 49`                      `dec ecx`

`:0072B963 7411`                    `je 0072B976`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072B72B(C)

|

`:0072B965 8B0D681BB200`            `mov ecx, dword ptr [00B21B68]`

`:0072B96B 80CD08`                  `or ch, 08`

`:0072B96E 890D681BB200`            `mov dword ptr [00B21B68], ecx`

`:0072B974 EB3C`                    `jmp 0072B9B2`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072B963(C)

|

`:0072B976 803A0F`                  `cmp byte ptr [edx], 0F`                                ; xxyy == C096 到这

`:0072B979 7537`                    `jne 0072B9B2`

`:0072B97B 807A0134`                `cmp byte ptr [edx+01], 34`

`:0072B97F 7531`                    `jne 0072B9B2`

`:0072B981 8B3D681BB200`            `mov edi, dword ptr [00B21B68]`                        ; ExceptionAddress 处是 0F 34

`:0072B987 83C8FF`                  `or eax, FFFFFFFF`

`:0072B98A 81CF00100000`            `or edi, 00001000`

`:0072B990 893D681BB200`            `mov dword ptr [00B21B68], edi`                        ; [00B21B68] |= 1000;

`:0072B996 8B4E04`                  `mov ecx, dword ptr [esi+04]`

`:0072B999 8381B800000002`          `add dword ptr [ecx+000000B8], 00000002`        ; context->eip +=2

`:0072B9A0 EB10`                    `jmp 0072B9B2`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072B960(C)

|

`:0072B9A2 A1681BB200`              `mov eax, dword ptr [00B21B68]`                        ; xxyy == C095 到这

`:0072B9A7 80CC04`                  `or ah, 04`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072B896(U)

|

`:0072B9AA A3681BB200`              `mov dword ptr [00B21B68], eax`                        ; [00B21B68] |= 400;

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0072B86F(U), :0072B889(C)

|

`:0072B9AF 83C8FF`                  `or eax, FFFFFFFF`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0072B734(C), :0072B73E(C), :0072B876(C), :0072B87E(U), :0072B89E(C)

|:0072B8D1(U), :0072B91C(C), :0072B926(C), :0072B958(U), :0072B974(U)

|:0072B979(C), :0072B97F(C), :0072B9A0(U)

|

`:0072B9B2 8B5604`                  `mov edx, dword ptr [esi+04]`

`:0072B9B5 5F`                      `pop edi`

`:0072B9B6 895A10`                  `mov dword ptr [edx+10], ebx`                        ; context->dr3 = 0, 清除硬件断点

`:0072B9B9 8B4E04`                  `mov ecx, dword ptr [esi+04]`

`:0072B9BC 89590C`                  `mov dword ptr [ecx+0C], ebx`                        ; context->dr2 = 0

`:0072B9BF 8B5604`                  `mov edx, dword ptr [esi+04]`

`:0072B9C2 895A08`                  `mov dword ptr [edx+08], ebx`                        ; context->dr1 = 0

`:0072B9C5 8B4E04`                  `mov ecx, dword ptr [esi+04]`

`:0072B9C8 5E`                      `pop esi`

`:0072B9C9 5D`                      `pop ebp`

`:0072B9CA 895904`                  `mov dword ptr [ecx+04], ebx`                        ; context->dr0 = 0

`:0072B9CD 5B`                      `pop ebx`

`:0072B9CE C20400`                  `ret 0004`

这段CODE 只是处理 Exception, 跟完了这段, 还是不太明白, 再想跟一跟 Exception 是如何触发的.

于是, 在 `0072B9CE` 处F8 返回到 kernel32.dll 领空后, 在 Memory Map 的 `00401000` CODE 段上 F2 下断, 然后F9

断在`0091885F`

```
00918854    8B49 04               MOV ECX,DWORD PTR DS:[ECX+4]
00918857    50                    PUSH EAX
00918858    51                    PUSH ECX
00918859    FF15 ACA0A000         CALL DWORD PTR DS:[A0A0AC]            ; kernel32.WaitForSingleObject
0091885F    85C0                  TEST EAX,EAX                                                        ; 返回到这儿了???????? 
00918861    74 12                 JE SHORT Skype.00918875
00918863    33D2                  XOR EDX,EDX
00918865    3D 02010000           CMP EAX,102
0091886A    0F94C2                SETE DL
```

前后看看, 感觉不应该是这样的. 刚开始也是一阵迷惑不解, 后来弄了好久才想可能是这样:

程序是多线程的, 有个线程由于在`00918859` `CALL kernel32.WaitForSingleObject` 而进入kernel32 的领空等待,

而我所跟的线程在处理好 Exception 后, 回到 kernel32.dll 领空, 再F9, 却被等待的线程抢先返回到 CODE 段的

`0091885F` 而触发了在 `0401000` CODE 段上 的 F2 断点.

所以这条路走不通, 再想别的法子了.

### 5. 程序中的 anti-debug CODE 分析 (2)

根据 `SetUnhandledExceptionFilter` 找到下面的code:

* Referenced by a CALL at Address:

|:0072BF8C

|

`:0072B9F0 53`                      `push ebx`

`:0072B9F1 56`                      `push esi`

`:0072B9F2 57`                      `push edi`

* Reference To: kernel32.GetCurrentThreadId, Ord:0111h

                                  |

`:0072B9F3 FF15909FA000`            `Call dword ptr [00A09F90]`

`:0072B9F9 A3F017B200`              `mov dword ptr [00B217F0], eax`                        ; ThreadID

`:0072B9FE E83DF2FFFF`              `call 0072AC40`                                                        ; 将 SEH 链的最后一项填到第一项, 实际上是reset SEH 链

                                                                                                                                                ; 并返回第一项

* Reference To: kernel32.SetUnhandledExceptionFilter, Ord:02BFh

                                  |

`:0072BA03 8B3D949FA000`            `mov edi, dword ptr [00A09F94]`

`:0072BA09 6870B67200`              `push 0072B670`

`:0072BA0E 8BF0`                    `mov esi, eax`                                                        ; 保存第一项

`:0072BA10 FFD7`                    `call edi`                                                                ; Call SetUnhandledExceptionFilter

`:0072BA12 A3EC17B200`              `mov dword ptr [00B217EC], eax`

`:0072BA17 B904000000`              `mov ecx, 00000004`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0072BA54(C)

|

`:0072BA1C A1381BB200`              `mov eax, dword ptr [00B21B38]`                        ; rand的seed

`:0072BA21 8B1D581BB200`            `mov ebx, dword ptr [00B21B58]`                        ; VirtualAlloc 的一起始地址,  将用做 STACK

`:0072BA27 8D1440`                  `lea edx, dword ptr [eax+2\*eax]`                ; 下面这几行就是 rand()

`:0072BA2A 8D1490`                  `lea edx, dword ptr [eax+4\*edx]`

`:0072BA2D C1E204`                  `shl edx, 04`

`:0072BA30 03D0`                    `add edx, eax`

`:0072BA32 C1E208`                  `shl edx, 08`

`:0072BA35 2BD0`                    `sub edx, eax`

`:0072BA37 8D8490C39E2600`          `lea eax, dword ptr [eax+4\*edx+00269EC3]`

`:0072BA3E 8B155C1BB200`            `mov edx, dword ptr [00B21B5C]`                        ; VirtualAlloc 的 STACK size

`:0072BA44 2BD1`                    `sub edx, ecx`

`:0072BA46 83C104`                  `add ecx, 00000004`

`:0072BA49 A3381BB200`              `mov dword ptr [00B21B38], eax`

`:0072BA4E 83F920`                  `cmp ecx, 00000020`
