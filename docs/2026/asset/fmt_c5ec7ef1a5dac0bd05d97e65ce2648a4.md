
# 反汇编分析

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00412235(C), :0041223E(C)

|

``` 
:0041228C A064D15300              mov al, byte ptr [0053D164]
:00412291 8D65FC
lea esp, dword ptr [ebp-04]
:00412294 5B
    pop ebx
:00412295 5D
    pop ebp
:00412296 C3
    ret
```

`call 0040EE00`是调用读取注册表的函数，所以从上面的地方出来，到达下面的比较的地方，在这里你将得到关键的比较地方，进入其中

``` 
:00412B40 55
    push ebp
:00412B41 31C0
  xor eax, eax
:00412B43 89E5
  mov ebp, esp
:00412B45 53
    push ebx
:00412B46 56
    push esi
:00412B47 57
    push edi
:00412B48 83EC10
sub esp, 00000010
:00412B4B 8B7D0C
mov edi, dword ptr [ebp+0C]
:00412B4E B9FFFFFFFF              mov ecx, FFFFFFFF
:00412B53 8B7508
mov esi, dword ptr [ebp+08]
:00412B56 C745F000000000          mov [ebp-10], 00000000
:00412B5D F2
    repnz
:00412B5E AE
    scasb
:00412B5F B8FEFFFFFF              mov eax, FFFFFFFE
:00412B64 29C8
  sub eax, ecx
:00412B66 3D0C000000              cmp eax, 0000000C
:00412B6B 0F850E010000            jne 00412C7F
:00412B71 31DB
  xor ebx, ebx
:00412B73 31D2
  xor edx, edx
:00412B75 EB7B
  jmp 00412BF2
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00412C09(C)

|

``` 
:00412B77 0FBFCA
movsx ecx, dx
:00412B7A 0FBE040E
movsx eax, byte ptr [esi+ecx]
:00412B7E 2D41000000              sub eax, 00000041
:00412B83 3D35000000              cmp eax, 00000035
:00412B88 7767
  ja 00412BF1
:00412B8A FF248568D25300          jmp dword ptr [4*eax+0053D268]
:00412B91 81C320A10700            add ebx, 0007A120
:00412B97 EB58
  jmp 00412BF1
:00412B99 8D842000000000          lea eax, dword ptr [eax]
:00412BA0 81C3A0860100            add ebx, 000186A0
:00412BA6 EB49
  jmp 00412BF1
:00412BA8 81C350C30000            add ebx, 0000C350
:00412BAE EB41
  jmp 00412BF1
:00412BB0 81C310270000            add ebx, 00002710
:00412BB6 EB39
  jmp 00412BF1
:00412BB8 81C388130000            add ebx, 00001388
:00412BBE EB31
  jmp 00412BF1
:00412BC0 81C3E8030000            add ebx, 000003E8
:00412BC6 EB29
  jmp 00412BF1
:00412BC8 81C3F4010000            add ebx, 000001F4
:00412BCE EB21
  jmp 00412BF1
:00412BD0 83C364
add ebx, 00000064
:00412BD3 EB1C
  jmp 00412BF1
:00412BD5 83C332
add ebx, 00000032
:00412BD8 EB17
  jmp 00412BF1
:00412BDA 8D8000000000            lea eax, dword ptr [eax+00000000]
:00412BE0 83C30A
add ebx, 0000000A
:00412BE3 EB0C
  jmp 00412BF1
:00412BE5 83C305
add ebx, 00000005
:00412BE8 EB07
  jmp 00412BF1
:00412BEA 8D8000000000            lea eax, dword ptr [eax+00000000]
:00412BF0 43
    inc ebx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00412B88(C), :00412B97(U), :00412BA6(U), :00412BAE(U), :00412BB6(U)

|:00412BBE(U), :00412BC6(U), :00412BCE(U), :00412BD3(U), :00412BD8(U)

|:00412BE3(U), :00412BE8(U)

|

``` 
:00412BF1 42
    inc edx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00412B75(U)

|

``` 
:00412BF2 B9FFFFFFFF              mov ecx, FFFFFFFF
:00412BF7 89F7
  mov edi, esi
:00412BF9 31C0
  xor eax, eax
:00412BFB F2
    repnz
:00412BFC AE
    scasb
:00412BFD 0FBFC2
movsx eax, dx
:00412C00 BFFEFFFFFF              mov edi, FFFFFFFE
:00412C05 29CF
  sub edi, ecx
:00412C07 39F8
  cmp eax, edi
:00412C09 0F8268FFFFFF            jb 00412B77
:00412C0F 8B4D0C
mov ecx, dword ptr [ebp+0C]
:00412C12 8A4109
mov al, byte ptr [ecx+09]
:00412C15 8945EC
mov dword ptr [ebp-14], eax
:00412C18 89C8
  mov eax, ecx
:00412C1A C6400900
mov [eax+09], 00
:00412C1E FF750C
push [ebp+0C]
:00412C21 E84AC30000              call 0041EF70
:00412C26 89C7
  mov edi, eax
:00412C28 59
    pop ecx
:00412C29 8B450C
mov eax, dword ptr [ebp+0C]
:00412C2C 8B4DEC
mov ecx, dword ptr [ebp-14]
:00412C2F 884809
mov byte ptr [eax+09], cl
:00412C32 29DF
  sub edi, ebx
:00412C34 8B450C
mov eax, dword ptr [ebp+0C]
:00412C37 0509000000              add eax, 00000009
:00412C3C 50
    push eax
:00412C3D E82EC30000              call 0041EF70
:00412C42 81FFFFFFFF0E            cmp edi, 0EFFFFFF
:00412C48 59
    pop ecx
:00412C49 89C3
  mov ebx, eax
:00412C4B 7432
  je 00412C7F
:00412C4D FF750C
push [ebp+0C]
:00412C50 56
    push esi
:00412C51 E82AFAFFFF              call 00412680
:00412C56 84C0
  test al, al
:00412C58 59
    pop ecx
:00412C59 59
    pop ecx
:00412C5A 7423
  je 00412C7F
:00412C5C B85F763708              mov eax, 0837765F
:00412C61 F7E7
  mul edi
:00412C63 055F763708              add eax, 0837765F
:00412C68 89F8
  mov eax, edi
:00412C6A 83D200
adc edx, 00000000
:00412C6D C1EA05
shr edx, 05
:00412C70 69D2E5030000            imul edx, 000003E5
:00412C76 29D0
  sub eax, edx
:00412C78 39C3
  cmp ebx, eax
:00412C7A 7503
  jne 00412C7F
:00412C7C 897DF0
mov dword ptr [ebp-10], edi
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00412B6B(C), :00412C4B(C), :00412C5A(C), :00412C7A(C)

|

``` 
:00412C7F 8165F000004000          and dword ptr [ebp-10], 00400000
:00412C86 8B45F0
mov eax, dword ptr [ebp-10]
:00412C89 8D65F4
lea esp, dword ptr [ebp-0C]
:00412C8C 5F
    pop edi
:00412C8D 5E
    pop esi
:00412C8E 5B
    pop ebx
:00412C8F 5D
    pop ebp
:00412C90 C3
    ret
```

这里就是比较的大本营了，如果你要得到这个软件的注册机，就应当仔细研究一下这个程序是如何生成注册码的，其实你就是想得到这个软件的注册码也要明白这个软件是如何生成注册码的过程。现在我简单的介绍一下是如何生成注册码的。

## 注册码生成流程

1. 首先检查你输入的注册码是否为12位

2. 将你的注册名每个字符-41后比较35(H)，如果大于就到达`[4*eax+0053D268]`其中`eax`就是计算后的值。如果小于就到达`00412BF1`这里，不参与计算。

3. 计算后得到过度值`ebx`

4. 将注册码的前9位值取出为`edi`（十六进位）

5. 使用`edi=edi-ebx`得到第6步运算的`edi`值

6. 下面是生成第二个比较值的过程，相信大家一定能够看懂

``` 
:00412C5C B85F763708              mov eax, 0837765F
:00412C61 F7E7
  mul edi
:00412C63 055F763708              add eax, 0837765F
:00412C68 89F8
  mov eax, edi
:00412C6A 83D200
adc edx, 00000000
:00412C6D C1EA05
shr edx, 05
:00412C70 69D2E5030000            imul edx, 000003E5
:00412C76 29D0
  sub eax, edx
```

在这里将得到`eax`，同注册码的后三位比较，如果相等就在下面的地方不跳跃，但是不也不表明注册成功。如下

``` 
:00412C78 39C3
  cmp ebx, eax
:00412C7A 7503
  jne 00412C7F
:00412C7C 897DF0
mov dword ptr [ebp-10], edi
:00412C7F 8165F000004000          and dword ptr [ebp-10], 00400000
:00412C86 8B45F0
mov eax, dword ptr [ebp-10]
```

因为这里`edi`是最后的返回值，所以如果让其值为0，就一定要使`edi AND 00400000`不为0。那么查看`AND`表可以知道只要使`edi`中第三位值为4，5，6，7，C,D,E,F就可以了。因为我使用`name:dahuilang`得到的`ebx=164F`，那么只要使最后的`edi`的第三位为4就可以了。这个就可以得到前9位值为`00400000 + 164F = 0040164F(H)=004200015`，通过上面的计算得到`eax = 39A`，所以最后三位就为`922`。现在我的注册码如下：

`name:dahuilang`

`RN:004200015922`

输入后注册成功。

破解成功。

欢迎访问我的主页：`http://programhunter.myetang.com`
