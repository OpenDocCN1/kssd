
# estiprojm

> 原文：[`www.pediy.com/kssd/pediy04/kanxue228.htm`](https://www.pediy.com/kssd/pediy04/kanxue228.htm)

时间有些长了，具体记不清过程了，有两部分，都贴上

estiprojm 加密软件:

```
004997AD 5A                          pop edx

004997AE E88DFBFFFF                  call 00499340

004997B3 8845FB                      mov byte ptr [ebp-05], al

004997B6 807DFB00                    cmp byte ptr [ebp-05], 00

004997BA 0F84AD000000                je 0049986D

004997C0 8B45FC                      mov eax, dword ptr [ebp-04]

004997C3 C680EC02000001              mov byte ptr [eax+000002EC], 01

* Possible StringData Ref from Code Obj ->"Registration complete."

004997CA B8C0984900                  mov eax, 004998C0

004997CF E83010FCFF                  call 0045A804

004997D4 B201                        mov dl, 01

004997D6 A178314600                  mov eax, dword ptr [00463178]

004997DB E8449BFCFF                  call 00463324

004997E0 8945F4                      mov dword ptr [ebp-0C], eax

004997E3 33C9                        xor ecx, ecx

* Possible StringData Ref from Code Obj ->"Software\Secretec Software\Invisible "
                                     ->"Secrets Pro\Version"

004997E5 BAE0984900                  mov edx, 004998E0

004997EA 8B45F4                      mov eax, dword ptr [ebp-0C]

004997ED E8269DFCFF                  call 00463518

004997F2 8D55EC                      lea edx, dword ptr [ebp-14]

004997F5 8B45FC                      mov eax, dword ptr [ebp-04]

004997F8 8B80C8020000                mov eax, dword ptr [eax+000002C8]

004997FE E85D86F9FF                  call 00431E60

00499803 8B4DEC                      mov ecx, dword ptr [ebp-14]

* Possible StringData Ref from Code Obj ->"Name"

00499806 BA24994900                  mov edx, 00499924

0049980B 8B45F4                      mov eax, dword ptr [ebp-0C]

0049980E E831A2FCFF                  call 00463A44

00499813 8D55EC                      lea edx, dword ptr [ebp-14]

00499816 8B45FC                      mov eax, dword ptr [ebp-04]

00499819 8B80D0020000                mov eax, dword ptr [eax+000002D0]

0049981F E83C86F9FF                  call 00431E60

00499824 8B4DEC                      mov ecx, dword ptr [ebp-14]

* Possible StringData Ref from Code Obj ->"Key"

00499827 BA34994900                  mov edx, 00499934

0049982C 8B45F4                      mov eax, dword ptr [ebp-0C]

0049982F E810A2FCFF                  call 00463A44

00499834 8D55EC                      lea edx, dword ptr [ebp-14]

00499837 8B45FC                      mov eax, dword ptr [ebp-04]

0049983A 8B80CC020000                mov eax, dword ptr [eax+000002CC]

00499840 E81B86F9FF                  call 00431E60

00499845 8B4DEC                      mov ecx, dword ptr [ebp-14]

* Possible StringData Ref from Code Obj ->"Organization"

00499848 BA40994900                  mov edx, 00499940

0049984D 8B45F4                      mov eax, dword ptr [ebp-0C]

00499850 E8EFA1FCFF                  call 00463A44

00499855 8B45F4                      mov eax, dword ptr [ebp-0C]

00499858 E82F9BFCFF                  call 0046338C

0049985D 8B45F4                      mov eax, dword ptr [ebp-0C]

00499860 E86B98F6FF                  call 004030D0

00499865 8B45FC                      mov eax, dword ptr [ebp-04]

00499868 E8C332FBFF                  call 0044CB30

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004997BA(C)

0049986D 807DFB00                    cmp byte ptr [ebp-05], 00

00499871 750A                        jne 0049987D

* Possible StringData Ref from Code Obj ->"Sorry! Invalid registration key!"

00499873 B858994900                  mov eax, 00499958

//////////////////////////////////////////////////////////////////////////////////////////////

004997AE E88DFBFFFF                  call 00499340

* Referenced by a CALL at Addresses:
|:004997AE , :004D21BE , :004DA606

00499340 55                          push ebp

00499341 8BEC                        mov ebp, esp

00499343 83C4E4                      add esp, FFFFFFE4

00499346 53                          push ebx

00499347 33DB                        xor ebx, ebx

00499349 895DEC                      mov dword ptr [ebp-14], ebx

0049934C 894DF4                      mov dword ptr [ebp-0C], ecx

0049934F 8955F8                      mov dword ptr [ebp-08], edx

00499352 8945FC                      mov dword ptr [ebp-04], eax

00499355 8B45FC                      mov eax, dword ptr [ebp-04]

00499358 E81BAEF6FF                  call 00404178

0049935D 8B45F8                      mov eax, dword ptr [ebp-08]

00499360 E813AEF6FF                  call 00404178

00499365 33C0                        xor eax, eax

00499367 55                          push ebp

00499368 687E944900                  push 0049947E

0049936D 64FF30                      push dword ptr fs:[eax]

00499370 648920                      mov dword ptr fs:[eax], esp

00499373 8B45F8                      mov eax, dword ptr [ebp-08]

00499376 E849ACF6FF                  call 00403FC4

0049937B 85C0                        test eax, eax

0049937D 7E40                        jle 004993BF

0049937F 8945E4                      mov dword ptr [ebp-1C], eax

00499382 C745E801000000              mov [ebp-18], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993BD(C)

00499389 8B45F8                      mov eax, dword ptr [ebp-08]

0049938C 8B55E8                      mov edx, dword ptr [ebp-18]

0049938F 8A4410FF                    mov al, byte ptr [eax+edx-01]

00499393 E81498F6FF                  call 00402BAC

00499398 04BF                        add al, BF

0049939A 2C06                        sub al, 06

0049939C 7219                        jb 004993B7

0049939E 8B45F8                      mov eax, dword ptr [ebp-08]

004993A1 8B55E8                      mov edx, dword ptr [ebp-18]

004993A4 8A4410FF                    mov al, byte ptr [eax+edx-01]

004993A8 04D0                        add al, D0

004993AA 2C0A                        sub al, 0A

004993AC 7209                        jb 004993B7

004993AE C645F300                    mov [ebp-0D], 00

004993B2 E9A4000000                  jmp 0049945B

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0049939C(C), :004993AC(C)

004993B7 FF45E8                      inc [ebp-18]

004993BA FF4DE4                      dec [ebp-1C]

004993BD 75CA                        jne 00499389

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049937D(C)

004993BF 8B45FC                      mov eax, dword ptr [ebp-04]

004993C2 E8FDABF6FF                  call 00403FC4

004993C7 83F805                      cmp eax, 00000005     <--名字是否大于等于 5 个字符

004993CA 7D09                        jge 004993D5

004993CC C645F300                    mov [ebp-0D], 00

004993D0 E986000000                  jmp 0049945B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993CA(C)

004993D5 8B45F8                      mov eax, dword ptr [ebp-08]

004993D8 E8E7ABF6FF                  call 00403FC4

004993DD 83F80C                      cmp eax, 0000000C     <-KEY 是否大于等于十二个字符

004993E0 7D06                        jge 004993E8

004993E2 C645F300                    mov [ebp-0D], 00

004993E6 EB73                        jmp 0049945B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004993E0(C)
```



:004993E8 `8B45F4`               `mov eax, dword ptr [ebp-0C]`

:004993EB `C6403801`             `mov [eax+38], 01`

:004993EF `8B55F8`               `mov edx, dword ptr [ebp-08]`

:004993F2 `8B45F4`               `mov eax, dword ptr [ebp-0C]`

:004993F5 `E88EFBFFFF`           `call 00498F88`

* Possible StringData Ref from Code Obj ->"Reia-mi al nemuririi nimb/Si focul "
  ->"din privire/Si pentru toate da-mi "
  ->"in schimb/O ora de iubire."
  |

:004993FA `BA98944900`           `mov edx, 00499498`

:004993FF `8B45F4`               `mov eax, dword ptr [ebp-0C]`

:00499402 `E81DFBFFFF`           `call 00498F24`

:00499407 `8B45F4`               `mov eax, dword ptr [ebp-0C]`

:0049940A `E861FEFFFF`           `call 00499270`      <--F8

:0049940F `8D45EC`               `lea eax, dword ptr [ebp-14]`

:00499412 `8B55F4`               `mov edx, dword ptr [ebp-0C]`

:00499415 `8B5228`               `mov edx, dword ptr [edx+28]`

:00499418 `E8C3A9F6FF`           `call 00403DE0`

:0049941D `C645F301`             `mov [ebp-0D], 01`

:00499421 `8B45EC`               `mov eax, dword ptr [ebp-14]`

:00499424 `E89BABF6FF`           `call 00403FC4`

:00499429 `85C0`                 `test eax, eax`

:0049942B `7E2E`                 `jle 0049945B`

:0049942D `8945E4`               `mov dword ptr [ebp-1C], eax`

:00499430 `C745E801000000`       `mov [ebp-18], 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00499459(C)
|

:00499437 `8B45EC`               `mov eax, dword ptr [ebp-14]`

:0049943A `8B55E8`               `mov edx, dword ptr [ebp-18]`

:0049943D `8A4410FF`             `mov al, byte ptr [eax+edx-01]`

:00499441 `8B55FC`               `mov edx, dword ptr [ebp-04]`

:00499444 `8B4DE8`               `mov ecx, dword ptr [ebp-18]`

:00499447 `3A440AFF`             `cmp al, byte ptr [edx+ecx-01]`

:0049944B `7406`                 `je 00499453`

:0049944D `C645F300`             `mov [ebp-0D], 00`

:00499451 `EB08`                 `jmp 0049945B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049944B(C)
|

:00499453 `FF45E8`               `inc [ebp-18]`

:00499456 `FF4DE4`               `dec [ebp-1C]`

:00499459 `75DC`                 `jne 00499437`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004993B2(U), :004993D0(U), :004993E6(U), :0049942B(C), :00499451(U)
|

:0049945B `33C0`                 `xor eax, eax`

:0049945D `5A`                   `pop edx`

:0049945E `59`                   `pop ecx`

:0049945F `59`                   `pop ecx`

:00499460 `648910`               `mov dword ptr fs:[eax], edx`

:00499463 `6885944900`           `push 00499485`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00499483(U)
|

:00499468 `8D45EC`               `lea eax, dword ptr [ebp-14]`

:0049946B `E8D8A8F6FF`           `call 00403D48`

:00499470 `8D45F8`               `lea eax, dword ptr [ebp-08]`

:00499473 `BA02000000`           `mov edx, 00000002`

:00499478 `E8EFA8F6FF`           `call 00403D6C`

:0049947D `C3`                   `ret`

:0049947E `E901A3F6FF`           `jmp 00403784`

:00499483 `EBE3`                 `jmp 00499468`

:00499485 `8A45F3`               `mov al, byte ptr [ebp-0D]`

:00499488 `5B`                   `pop ebx`

:00499489 `8BE5`                 `mov esp, ebp`

:0049948B `5D`                   `pop ebp`

:0049948C `C3`                   `ret`

//////////////////////////////////////////////////////////////////////////////////////////////

:0049940A `E861FEFFFF`           `call 00499270`

* Referenced by a CALL at Addresses:
|:0049940A , :004C0B39 , :004C1C13 , :004C25DD , :004C2875
|:004CA88B , :004CAD7F , :004DAC6C , :004DACE0 , :004DAD4D
|:004DADD0 , :004DAEC5
|

:00499270 `55`                   `push ebp`

:00499271 `8BEC`                 `mov ebp, esp`

:00499273 `83C4F4`               `add esp, FFFFFFF4`

........................................................

........................................................

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004992B5(U)
|

:004992BB `8A45FB`               `mov al, byte ptr [ebp-05]`

:004992BE `50`                   `push eax`

:004992BF `8D45F4`               `lea eax, dword ptr [ebp-0C]`

:004992C2 `50`                   `push eax`

:004992C3 `8B45FC`               `mov eax, dword ptr [ebp-04]`

:004992C6 `8B4834`               `mov ecx, dword ptr [eax+34]`

:004992C9 `8B45FC`               `mov eax, dword ptr [ebp-04]`

:004992CC `8B5024`               `mov edx, dword ptr [eax+24]`

:004992CF `8B45FC`               `mov eax, dword ptr [ebp-04]`

:004992D2 `E815FDFFFF`           `call 00498FEC`      <-F8

:004992D7 `8B55F4`               `mov edx, dword ptr [ebp-0C]`

:004992DA `8B45FC`               `mov eax, dword ptr [ebp-04]`

:004992DD `83C028`               `add eax, 00000028`

:004992E0 `E8B7AAF6FF`           `call 00403D9C`

//////////////////////////////////////////////////////////////////////////////////////////////

:004992D2 `E815FDFFFF`           `call 00498FEC`
* Referenced by a CALL at Address:
|:004992D2
|

:00498FEC `55`                   `push ebp`

:00498FED `8BEC`                 `mov ebp, esp`

:00498FEF `83C4BC`               `add esp, FFFFFFBC`

........................................................

........................................................

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004991F5(C)
|

:00499158 `8D45BC`               `lea eax, dword ptr [ebp-44]`

:0049915B `50`                   `push eax`

:0049915C `B902000000`           `mov ecx, 00000002`

首先声名一下：`XX XX XX XX XX XX ......`

    `G1 G2 G3 G4 G5 G6`
    `^  ^  ^  ^  ^  ^`
    `|  |  |  |  |  |`
    `|  |  |  |  |  |----------每两个注册码一组,此为第六组`
    `|  |  |  |  |-------------每两个注册码一组,此为第五组`
    `|  |  |  |-----------------每两个注册码一组,此为第四组`
    `|  |  |--------------------每两个注册码一组,此为第三组`
    `|  |-----------------------每两个注册码一组,此为第二组`
    `|---------------------------每两个注册码一组,此为第一组`

程序从下列代码中

`Reia-mi al nemuririi nimb/Si focul din privire/Si pentru toate da-mi in schimb/O ora de iubire..............KeyE`

`Reia-mi al nemuririi nimb/Si focul din privire/Si pentru toate da-mi in schimb/O ora de iubire.`

依次取出其 ACSII 码 `R` 为 `52`..将取出的码与 `G2` 异或,异或的结果,如果大于 `G1` 则减去 `G1`,其结果作为 `NAME` 的第一个字母,否则加 `FF` 再减去 `G1` 其结果作为 `NAME` 的第一个字母,再取 `e` 的 ACSSII 码 `65` 与 `G3` 异或,异或的结果,如果大于 `G2` 则减去 `G2`,其结果作为 `NAME` 的第二个字母,否则加 `FF` 再减去 `G2` 其结果作为 `NAME` 的第二个字母,以后过程都一样了.简单的很!!!!(大家看起来简单的很,可是我当时研究其注册过程时,可费了整整一晚上的时间,爱!!!!!!!!)

刚出炉,还烫着的注册过程

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004991F5(C)
|

:00499158 `8D45BC`               `lea eax, dword ptr [ebp-44]`

:0049915B `50`                   `push eax`

:0049915C `B902000000`           `mov ecx, 00000002`

:00499161 `8B55E0`               `mov edx, dword ptr [ebp-20]`

:00499164 `8B45F8`               `mov eax, dword ptr [ebp-08]`

:00499167 `E85CB0F6FF`           `call 004041C8`

:0049916C `8B4DBC`               `mov ecx, dword ptr [ebp-44]`

:0049916F `8D45C0`               `lea eax, dword ptr [ebp-40]`

:00499172 `BA6C924900`           `mov edx, 0049926C`

:00499177 `E894AEF6FF`           `call 00404010`



```
:0049917C 8B45C0                 mov eax, dword ptr [ebp-40]

:0049917F E840F9F6FF             call 00408AC4

:00499184 8945DC                 mov dword ptr [ebp-24], eax

:00499187 8B45EC                 mov eax, dword ptr [ebp-14]

:0049918A 3B45F0                 cmp eax, dword ptr [ebp-10]

:0049918D 7D05                   jge 00499194

:0049918F FF45EC                 inc [ebp-14]

:00499192 EB07                   jmp 0049919B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049918D(C)
|

:00499194 C745EC01000000         mov [ebp-14], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00499192(U)
|

:0049919B 8B45F4                 mov eax, dword ptr [ebp-0C]

:0049919E 8B55EC                 mov edx, dword ptr [ebp-14]

:004991A1 0FB64410FF             movzx eax, byte ptr [eax+edx-01]     <-取 Reia-mi al nemuririi      <-nimb/Si focul

:004991A6 3345DC                 xor eax, dword ptr [ebp-24]        <-与 G(X)异或

:004991A9 8945D8                 mov dword ptr [ebp-28], eax

:004991AC 8B45D8                 mov eax, dword ptr [ebp-28]

:004991AF 3B45E8                 cmp eax, dword ptr [ebp-18]        <-大于 G(X-1)?

:004991B2 7F10                   jg 004991C4

:004991B4 8B45D8                 mov eax, dword ptr [ebp-28]

:004991B7 05FF000000             add eax, 000000FF       <-小于 G(X-1),加  FF

:004991BC 2B45E8                 sub eax, dword ptr [ebp-18]        <-减 G(X-1)

:004991BF 8945D8                 mov dword ptr [ebp-28], eax

:004991C2 EB06                   jmp 004991CA

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004991B2(C)
|

:004991C4 8B45E8                 mov eax, dword ptr [ebp-18]        <-减 G(X-1)

:004991C7 2945D8                 sub dword ptr [ebp-28], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004991C2(U)
|

:004991CA 8D45C0                 lea eax, dword ptr [ebp-40]

:004991CD 8B55D8                 mov edx, dword ptr [ebp-28]

:004991D0 E817ADF6FF             call 00403EEC

:004991D5 8B55C0                 mov edx, dword ptr [ebp-40]

:004991D8 8D45E4                 lea eax, dword ptr [ebp-1C]

:004991DB E8ECADF6FF             call 00403FCC

:004991E0 8B45DC                 mov eax, dword ptr [ebp-24]

:004991E3 8945E8                 mov dword ptr [ebp-18], eax

:004991E6 8345E002               add dword ptr [ebp-20], 00000002

:004991EA 8B45F8                 mov eax, dword ptr [ebp-08]

:004991ED E8D2ADF6FF             call 00403FC4

:004991F2 3B45E0                 cmp eax, dword ptr [ebp-20]

:004991F5 0F8F5DFFFFFF           jg 00499158

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0049909A(C), :0049911B(U)
|

:004991FB 8B4508                 mov eax, dword ptr [ebp+08]

:004991FE 8B55E4                 mov edx, dword ptr [ebp-1C]

:00499201 E8DAABF6FF             call 00403DE0

:00499206 33C0                   xor eax, eax

:00499208 5A                     pop edx

:00499209 59                     pop ecx

:0049920A 59                     pop ecx

:0049920B 648910                 mov dword ptr fs:[eax], edx

:0049920E 683D924900             push 0049923D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049923B(U)
|

:00499213 8D45BC                 lea eax, dword ptr [ebp-44]

:00499216 BA02000000             mov edx, 0000002

:0049921B E84CABF6FF             call 00403D6C

:00499220 8D45E4                 lea eax, dword ptr [ebp-1C]

:00499223 E820ABF6FF             call 00403D48

:00499228 8D45F4                 lea eax, dword ptr [ebp-0C]

:0049922B BA02000000             mov edx, 0000002

:00499230 E837ABF6FF             call 00403D6C

:00499235 C3                     ret
```

DATA SEGMENT

```
TABLE     DB 52H,65H,69H,61H,2DH,6DH,69H,20H,61H,6CH,20H,6EH,65H,6DH,75H,72H
          DB 69H,72H,69H,69H,20H,6EH,69H,6DH,62H,2FH,53H,69H,20H,66H,6FH,63H
          DB 75H,6CH,20H,64H,69H,6EH,20H,70H,72H,69H,76H,69H,72H,65H,2FH,53H
          DB 69H,20H,70H,65H,6EH,74H,72H,75H,20H,74H,6FH,61H,74H,65H,20H,64H
          DB 61H,2DH,6DH,69H,20H,69H,6EH,20H,73H,63H,68H,69H,6DH,62H,2FH,4FH
          DB 20H,6FH,72H,61H,20H,64H,65H,20H,69H,75H,62H,69H,72H,65H,2EH    ;5EH GE

INNAME    DB 'INPUT YOUR NAME: $'
NAMEBUFF  DB 40H
          DB ?
          DB 40H DUP(?)
DATABUFF  DB 042H
          DB ?
          DB 45H DUP(?)
YKEY      DB 'YOUR KEY: $'
CLF       DB 0DH,0AH,'$'
DISPLAY   DB 0A0H DUP(?)
DATA      ENDS
```

CODE SEGMENT

```
      ASSUME CS:CODE, DS:DATA

START:
        MOV   AX,DATA
        MOV   DS,AX
        MOV   DX,OFFSET INNAME
        MOV   AH,09H
        INT   21H
        MOV   DX,OFFSET NAMEBUFF
        MOV   AH,0AH
        INT   21H
        MOV   DX,OFFSET CLF
        MOV   AH,09H
        INT   21H
        XOR   CX,CX
        XOR   BX,BX
        XOR   AX,AX
        XOR   DX,DX
        MOV   SI, OFFSET NAMEBUFF+1
        MOV   CL,[SI]
        MOV   DL,[SI]
        MOV   AL,011H

CALCUL:
        MOV   BYTE PTR [DATABUFF+BX+2],AL
        ADD   AL,[NAMEBUFF+BX+2]
        ;CMP   AX,0FFH
        JNC   BELOW
        ADD   AL,01H

BELOW:
        XOR   AL,[TABLE+BX]
        CMP   BX,CX
        INC   BX
        JB    CALCUL
        MOV   CL,04H
        XOR   BX,BX
        XOR   SI,SI
        ;INC   DX
        ADD   DX,DX

CONVER:
        MOV   AL, [DATABUFF+SI+2]
        MOV   AH,AL
        SHR   AH,CL
        CMP   AH,09H
        JA    CH1
        ADD   AH,30H
        JMP   STRMOV1

CH1:
        ADD   AH,037H

STRMOV1:
        MOV   BYTE PTR [DISPLAY+BX],AH
        ;CMP   BX,DX
        INC   BX
        MOV   AH,AL
        AND   AH,0FH
        CMP   AH,09H
        JA    CH2
        ADD   AH,30H
        JMP   STRMOV2

CH2:
        ADD   AH,037H

STRMOV2:
        MOV   BYTE PTR [DISPLAY+BX],AH
        CMP   BX,DX
        INC   BX
        INC   SI
        JB    CONVER

        MOV   BYTE PTR [DISPLAY+BX+1],0DH
        MOV   BYTE PTR [DISPLAY+BX+2],0AH
        MOV   BYTE PTR [DISPLAY+BX+3],'$'
        MOV   DX,OFFSET YKEY
        MOV   AH,09H
        INT   21H
        MOV   DX,OFFSET DISPLAY
        MOV   AH,09H
        INT   21H
        MOV   AH,4CH
        INT   21H
```

CODE ENDS

```
      END   START
```

`4dd466d0e1e0cc123afbe504da2462ca`
