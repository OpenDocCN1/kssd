
# estiprojm 注册 (12千字)

时间有些长了,具体记不清过程了,有两部分,都贴上

estiprojm 加密软件:`004997AD` `5A`

```
pop edx

:004997AE E88DFBFFFF              call 00499340

:004997B3 8845FB
mov byte ptr [ebp-05], al

:004997B6 807DFB00                cmp
byte ptr [ebp-05], 00

:004997BA 0F84AD000000            je 0049986D

:004997C0 8B45FC
mov eax, dword ptr [ebp-04]

:004997C3 C680EC02000001          mov byte ptr [eax+000002EC],
01
* Possible StringData Ref from Code Obj ->"Registration complete."
            |

:004997CA B8C0984900              mov eax,
004998C0

:004997CF E83010FCFF              call 0045A804

:004997D4 B201
  mov dl, 01

:004997D6 A178314600              mov eax,
dword ptr [00463178]

:004997DB E8449BFCFF              call 00463324

:004997E0 8945F4
mov dword ptr [ebp-0C], eax

:004997E3 33C9
  xor ecx, ecx
* Possible StringData Ref from Code Obj ->"Software\Secretec Software\Invisible
"
                  ->"Secrets Pro\Version"
            |

:004997E5 BAE0984900              mov edx,
004998E0

:004997EA 8B45F4
mov eax, dword ptr [ebp-0C]

:004997ED E8269DFCFF              call 00463518

:004997F2 8D55EC
lea edx, dword ptr [ebp-14]

:004997F5 8B45FC
mov eax, dword ptr [ebp-04]

:004997F8 8B80C8020000            mov eax, dword
ptr [eax+000002C8]

:004997FE E85D86F9FF              call 00431E60

:00499803 8B4DEC
mov ecx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"Name"
            |

:00499806 BA24994900              mov edx,
00499924

:0049980B 8B45F4
mov eax, dword ptr [ebp-0C]

:0049980E E831A2FCFF              call 00463A44

:00499813 8D55EC
lea edx, dword ptr [ebp-14]

:00499816 8B45FC
mov eax, dword ptr [ebp-04]

:00499819 8B80D0020000            mov eax, dword
ptr [eax+000002D0]

:0049981F E83C86F9FF              call 00431E60

:00499824 8B4DEC
mov ecx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"Key"
            |

:00499827 BA34994900              mov edx,
00499934

:0049982C 8B45F4
mov eax, dword ptr [ebp-0C]

:0049982F E810A2FCFF              call 00463A44

:00499834 8D55EC
lea edx, dword ptr [ebp-14]

:00499837 8B45FC
mov eax, dword ptr [ebp-04]

:0049983A 8B80CC020000            mov eax, dword
ptr [eax+000002CC]

:00499840 E81B86F9FF              call 00431E60

:00499845 8B4DEC
mov ecx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"Organization"
            |

:00499848 BA40994900              mov edx,
00499940

:0049984D 8B45F4
mov eax, dword ptr [ebp-0C]

:00499850 E8EFA1FCFF              call 00463A44

:00499855 8B45F4
mov eax, dword ptr [ebp-0C]

:00499858 E82F9BFCFF              call 0046338C

:0049985D 8B45F4
mov eax, dword ptr [ebp-0C]

:00499860 E86B98F6FF              call 004030D0

:00499865 8B45FC
mov eax, dword ptr [ebp-04]

:00499868 E8C332FBFF              call 0044CB30
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004997BA(C)
|

:0049986D 807DFB00                cmp
byte ptr [ebp-05], 00

:00499871 750A
  jne 0049987D
* Possible StringData Ref from Code Obj ->"Sorry! Invalid registration key!"
            |

:00499873 B858994900              mov eax,
00499958
```
```
:004997AE E88DFBFFFF              call 00499340
* Referenced by a CALL at Addresses:
|:004997AE  , :004D21BE  , :004DA606
|

:00499340 55
  push ebp

:00499341 8BEC
  mov ebp, esp

:00499343 83C4E4
add esp, FFFFFFE4

:00499346 53
  push ebx

:00499347 33DB
  xor ebx, ebx

:00499349 895DEC
mov dword ptr [ebp-14], ebx

:0049934C 894DF4
mov dword ptr [ebp-0C], ecx

:0049934F 8955F8
mov dword ptr [ebp-08], edx

:00499352 8945FC
mov dword ptr [ebp-04], eax

:00499355 8B45FC
mov eax, dword ptr [ebp-04]

:00499358 E81BAEF6FF              call 00404178

:0049935D 8B45F8
mov eax, dword ptr [ebp-08]

:00499360 E813AEF6FF              call 00404178

:00499365 33C0
  xor eax, eax

:00499367 55
  push ebp

:00499368 687E944900              push 0049947E

:0049936D 64FF30
push dword ptr fs:[eax]

:00499370 648920
mov dword ptr fs:[eax], esp

:00499373 8B45F8
mov eax, dword ptr [ebp-08]

:00499376 E849ACF6FF              call 00403FC4

:0049937B 85C0
  test eax, eax

:0049937D 7E40
  jle 004993BF

:0049937F 8945E4
mov dword ptr [ebp-1C], eax

:00499382 C745E801000000          mov [ebp-18], 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993BD(C)
|

:00499389 8B45F8
mov eax, dword ptr [ebp-08]

:0049938C 8B55E8
mov edx, dword ptr [ebp-18]

:0049938F 8A4410FF                mov
al, byte ptr [eax+edx-01]

:00499393 E81498F6FF              call 00402BAC

:00499398 04BF
  add al, BF

:0049939A 2C06
  sub al, 06

:0049939C 7219
  jb 004993B7

:0049939E 8B45F8
mov eax, dword ptr [ebp-08]

:004993A1 8B55E8
mov edx, dword ptr [ebp-18]

:004993A4 8A4410FF                mov
al, byte ptr [eax+edx-01]

:004993A8 04D0
  add al, D0

:004993AA 2C0A
  sub al, 0A

:004993AC 7209
  jb 004993B7

:004993AE C645F300                mov
[ebp-0D], 00

:004993B2 E9A4000000              jmp 0049945B
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0049939C(C), :004993AC(C)
|

:004993B7 FF45E8
inc [ebp-18]

:004993BA FF4DE4
dec [ebp-1C]

:004993BD 75CA
  jne 00499389
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049937D(C)
|

:004993BF 8B45FC
mov eax, dword ptr [ebp-04]

:004993C2 E8FDABF6FF              call 00403FC4

:004993C7 83F805
cmp eax, 00000005        <--名字是否大于等于5个字符

:004993CA 7D09
  jge 004993D5

:004993CC C645F300                mov
[ebp-0D], 00

:004993D0 E986000000              jmp 0049945B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993CA(C)
|

:004993D5 8B45F8
mov eax, dword ptr [ebp-08]

:004993D8 E8E7ABF6FF              call 00403FC4

:004993DD 83F80C
cmp eax, 0000000C        <-KEY是否大于等于十二个字符

:004993E0 7D06
  jge 004993E8

:004993E2 C645F300                mov
[ebp-0D], 00

:004993E6 EB73
  jmp 0049945B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993E0(C)
|

:004993E8 8B45F4
mov eax, dword ptr [ebp-0C]

:004993EB C6403801                mov
[eax+38], 01

:004993EF 8B55F8
mov edx, dword ptr [ebp-08]

:004993F2 8B45F4
mov eax, dword ptr [ebp-0C]

:004993F5 E88EFBFFFF              call 00498F88
* Possible StringData Ref from Code Obj ->"Reia-mi al nemuririi nimb/Si focul
"
                  ->"din privire/Si
pentru toate da-mi "
                  ->"in schimb/O
ora de iubire."
            |

:004993FA BA98944900              mov edx,
00499498

:004993FF 8B45F4
mov eax, dword ptr [ebp-0C]

:00499402 E81DFBFFFF              call 00498F24

:00499407 8B45F4
mov eax, dword ptr [ebp-0C]

:0049940A E861FEFFFF              call 00499270
          <--F8

:0049940F 8D45EC
lea eax, dword ptr [ebp-14]

:00499412 8B55F4
mov edx, dword ptr [ebp-0C]

:00499415 8B5228
mov edx, dword ptr [edx+28]

:00499418 E8C3A9F6FF              call 00403DE0

:0049941D C645F301                mov
[ebp-0D], 01

:00499421 8B45EC
mov eax, dword ptr [ebp-14]

:00499424 E89BABF6FF              call 00403FC4

:00499429 85C0
  test eax, eax

:0049942B 7E2E
  jle 0049945B

:0049942D 8945E4
mov dword ptr [ebp-1C], eax

:00499430 C745E801000000          mov [ebp-18], 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00499459(C)
|

:00499437 8B45EC
mov eax, dword ptr [ebp-14]

:0049943A 8B55E8
mov edx, dword ptr [ebp-18]

:0049943D 8A4410FF                mov
al, byte ptr [eax+edx-01]

:00499441 8B55FC
mov edx, dword ptr [ebp-04]

:00499444 8B4DE8
mov ecx, dword ptr [ebp-18]

:00499447 3A440AFF                cmp
al, byte ptr [edx+ecx-01]

:0049944B 7406
  je 00499453

:0049944D C645F300                mov
[ebp-0D], 00

:00499451 EB08
  jmp 0049945B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049944B(C)
|
```
