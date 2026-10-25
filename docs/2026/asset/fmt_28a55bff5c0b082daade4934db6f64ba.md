
# 初学者-Winzip8.0

## Winzip V8.0

输入

用户名: Liu Tong

注册码: 87654321

设断点`bpx hmemcpy`

点OK键,被中断后按F12键9次(第10次出错)

便按F10便用`D`命令找输入的注册码(如`D ESI`, `D EDI`....)

如找到注册码的地址`****:********`，设断点`bpx ****:********`

`CTRL+D`返回Winzip,再次被中断后注意找,下面的程序,若不是继续按`CTRL+D`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004692F8(C), :00469318(C)
|
:004692EC 0AC0
  or al, al
:004692EE 742E
  je 0046931E
:004692F0 8A06
  mov al, byte ptr [esi]<----此处用D ESI命令可看到输入码
:004692F2 46
  inc esi
:004692F3 8A27
  mov ah, byte ptr [edi]<----此处用D EDI命令可看到注册码
:004692F5 47
  inc edi
:004692F6 38C4
  cmp ah, al
:004692F8 74F2
  je 004692EC
:004692FA 2C41
  sub al, 41
:004692FC 3C1A
  cmp al, 1A
:004692FE 1AC9
  sbb cl, cl
:00469300 80E120
and cl, 20
:00469303 02C1
  add al, cl
:00469305 0441
  add al, 41
:00469307 86E0
  xchg al, ah
:00469309 2C41
  sub al, 41
:0046930B 3C1A
  cmp al, 1A
:0046930D 1AC9
  sbb cl, cl
:0046930F 80E120
and cl, 20
:00469312 02C1
  add al, cl
:00469314 0441
  add al, 41
:00469316 38E0
  cmp al, ah
:00469318 74D2
  je 004692EC
:0046931A 1AC0
  sbb al, al
:0046931C 1CFF
  sbb al, FF
```

下面是找注册码生成的过程:

通常,注册码是由用户名(或用户名加公司名)计算出来的.

因此,设断点`bpx ****:********`(`****:********`是用户名存储的地址)

会找到两段程序(软件的注册码是由两个4位码组合成的):

第一段程序生成后4位码:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407B6C(U)
|
:00407B5A 84D2
  test dl, dl
:00407B5C 7410
  je 00407B6E
:00407B5E 660FB6D2                movzx
dx, dl
:00407B62 0FAFD7
imul edx, edi
:00407B65 03DA
  add ebx, edx
:00407B67 8A5601
mov dl, byte ptr [esi+01]
:00407B6A 47
  inc edi
:00407B6B 46
  inc esi
:00407B6C EBEC
  jmp 00407B5A
```

第二段程序生成前4位码:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407B97(U)
|
:00407B7C 84C9
  test cl, cl
:00407B7E 7419
  je 00407B99
:00407B80 660FB6C9                movzx
cx, cl
:00407B84 6821100000              push 00001021
:00407B89 51
  push ecx
:00407B8A 50
  push eax
:00407B8B E829000000              call 00407BB9
:00407B90 8A4E01
mov cl, byte ptr [esi+01]
:00407B93 83C40C
add esp, 0000000C
:00407B96 46
  inc esi
:00407B97 EBE3
  jmp 00407B7C
```

======================================================================

```
* Referenced by a CALL at Addresses:
|:00407B8B  , :00407C75
|
:00407BB9 55
  push ebp
:00407BBA 8BEC
  mov ebp, esp
:00407BBC 8B4508
mov eax, dword ptr [ebp+08]
:00407BBF 56
  push esi
:00407BC0 33C9
  xor ecx, ecx
* Possible Ref to Menu: RBUTTONMENU1, Item: "Delete..."
            |
* Possible Reference to String Resource ID=00008: "Delete files from %s"
            |
:00407BC2 6A08
  push 00000008
:00407BC4 8A6D0C
mov ch, byte ptr [ebp+0C]
:00407BC7 5A
  pop edx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407BDF(C)
|
:00407BC8 8BF0
  mov esi, eax
:00407BCA 33F1
  xor esi, ecx
:00407BCC 66F7C60080              test si,
8000
:00407BD1 7407
  je 00407BDA
:00407BD3 03C0
  add eax, eax
:00407BD5 334510
xor eax, dword ptr [ebp+10]
:00407BD8 EB02
  jmp 00407BDC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407BD1(C)
|
:00407BDA D1E0
  shl eax, 1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407BD8(U)
|
:00407BDC D1E1
  shl ecx, 1
:00407BDE 4A
  dec edx
:00407BDF 75E7
  jne 00407BC8
:00407BE1 5E
  pop esi
:00407BE2 5D
  pop ebp
:00407BE3 C3
  ret
```
