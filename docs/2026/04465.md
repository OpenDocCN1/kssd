
# audioconverterV31B (6千字)

说明: 本文仅供研究用;这个软件很不错,希望有经济实力的同志还是支持一下软件作者吧! ^\_^

## 1. 去掉版权提示窗口

\* Reference To: `audconv.CreateLicenseManager`, Ord:0019h

```
|
:00408317 FF1520044200            Call dword ptr
[00420420]
:0040831D 83C41C
add esp, 0000001C
:00408320 85C0
  test eax, eax
:00408322 0F8C12010000            jl 0040843A
:00408328 8B4DFC
mov ecx, dword ptr [ebp-04]
:0040832B 8B01
  mov eax, dword ptr [ecx]
:0040832D FF501C
call [eax+1C]            ;<--此call执行后,eax=0
:00408330 85C0
  test eax, eax            ;
```

程序将跳到`408386`

```
:00408332 7452
  je 00408386
:00408334 8B4DFC
mov ecx, dword ptr [ebp-04]
:00408337 33FF
  xor edi, edi
:00408339 57
  push edi
:0040833A 8B01
  mov eax, dword ptr [ecx]
:0040833C FF10
  call dword ptr [eax]
:0040833E 3D05400080              cmp eax,
80004005
:00408343 740A
  je 0040834F
:00408345 3BC7
  cmp eax, edi
:00408347 0F85ED000000            jne 0040843A
:0040834D EB39
  jmp 00408388
.
.
.
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00408332(C)
|
:00408386 33FF
  xor edi, edi
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:0040834D(U)
|
:00408388 57
  push edi
```

\* Reference To: `audconv.ShowEULA`, Ord:002Dh

```
|
:00408389 FF1524044200            Call dword ptr
[00420424]
:0040838F 85C0
  test eax, eax
:00408391 59
  pop ecx
:00408392 0F8CA2000000            jl 0040843A
:00408398 8B4DFC
mov ecx, dword ptr [ebp-04]
:0040839B 57
  push edi
:0040839C 8B01
  mov eax, dword ptr [ecx]
:0040839E FF5014
call [eax+14]            ;<--呼叫协议窗口
:004083A1 85C0
  test eax, eax            ;<--按Agree按钮,eax=0;
              ;
按Cancel按钮,eax=1
:004083A3 0F8591000000            jne 0040843A
          ;<--eax=1,则跳到40843a
              ;
一跳到40843a,就game over了
:004083A9 833DE462420001          cmp dword ptr [004262E4],
00000001
:004083B0 7E62
  jle 00408414            ;<--从此处,跳到408414
              ;
408414是主程序开始的地方
.
.
.
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
|:004083B0(C), :004083DC(C)
|
:00408414 FF7514
push [ebp+14]
:00408417 8D8DB8FEFFFF            lea ecx, dword
ptr [ebp+FFFFFEB8]
:0040841D FF7510
push [ebp+10]
:00408420 E894170000              call 00409BB9
:00408425 8945F8
mov dword ptr [ebp-08], eax
```

经过动态跟踪,发现:从`00408388`到`00408414`,`ecx`和`edx`的值有变化(`edx=0`),而`408417`的指令将`ecx`的值覆盖.

所以,只要在`00408388`处,改指令为:

```
    mov edx,0
    jmp 408414
```

即可.

因此,在`audconv.exe`文件中的`00008388`处,找到`57 FF 15 24 04 42 00 85 C0 59` ,

`^^ ^^ ^^ ^^ ^^`

`^^ ^^ ^^ ^^ ^^`

将其改成 `BA 00 00 00 00 E9 82 00 00 00`

## 2. 去掉版权提示窗口后,如果过期,关闭时还有提示窗口出现,下面将去掉该提示窗口

\* Reference To: `audconv.CreateLicenseManager`, Ord:0019h

```
|
:0040C029 FF1520044200            Call dword ptr
[00420420]
:0040C02F 85C0
  test eax, eax
:0040C031 59
  pop ecx
:0040C032 7C13
  jl 0040C047        ;<--若在此处跳转,则过期提示窗口不会出现
:0040C034 8B4DFC
mov ecx, dword ptr [ebp-04]
:0040C037 FF7604
push [esi+04]
:0040C03A 8B01
  mov eax, dword ptr [ecx]
:0040C03C FF5018
call [eax+18]
  ;<--出现过期提示窗口
:0040C03F 8B4DFC
mov ecx, dword ptr [ebp-04]
:0040C042 8B01
  mov eax, dword ptr [ecx]
:0040C044 FF5020
call [eax+20]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:0040C032(C)
|
:0040C047 8B4514
mov eax, dword ptr [ebp+14]
:0040C04A 5E
  pop esi
:0040C04B 832000
and dword ptr [eax], 00000000
:0040C04E 33C0
  xor eax, eax
:0040C050 C9
  leave
:0040C051 C21000
ret 0010
```

所以,在`audconv.exe`文件的`0000C032`处,将`7C`改为`7D`.

## 3. 在转换文件时,还弹出评估版提示窗口,程序用的是`audconv.dll`中的代码.

代码如下:

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10004485(U)
|
:10004489 8B4DFC
mov ecx, dword ptr [ebp-04]
:1000448C 8B01
  mov eax, dword ptr [ecx]
:1000448E FF5010
call [eax+10]
:10004491 85C0
  test eax, eax
:10004493 0F8540030000            jne 100047D9
:10004499 8B45E0
mov eax, dword ptr [ebp-20]
```

\* Possible Reference to String Resource ID=00010: "MPEG Layer-1 Audio Files

(.mp1)"

```
|
:1000449C 6A0A
  push 0000000A
:1000449E 33D2
  xor edx, edx
:100044A0 59
  pop ecx
:100044A1 F7F1
  div ecx
:100044A3 85D2
  test edx, edx
:100044A5 751B
  jne 100044C2
:100044A7 8B4DFC
mov ecx, dword ptr [ebp-04]
:100044AA 6A40
  push 00000040            ;<--由这里跳到100044c2,
              ;
评估版提示窗口就不会出现
```

\* Possible StringData Ref from Data Obj ->"Audio Converter"

```
|
:100044AC 6800530810              push 10085300
```

\* Possible StringData Ref from Data Obj ->"This is an evaluation version "

->"of Audio Converter."

```
|
:100044B1 681C5E0810              push 10085E1C
:100044B6 8B01
  mov eax, dword ptr [ecx]
:100044B8 FF5024
call [eax+24]
:100044BB 50
  push eax
```

\* Reference To: `USER32.MessageBoxA`, Ord:01C3h

```
|
:100044BC FF15E0430710            Call dword ptr
[100743E0]    ;<--call评估版提示窗口
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100044A5(C)
|
:100044C2 8B47FC
mov eax, dword ptr [edi-04]
```

因此,将`audconv.dll`中的从`44aa`开始的代码改为`e9 13 00 00 00 90 90`,即可.

另:`metadata`窗口的限制没有解除,特别是`comments`的限制,还望高手指教!
