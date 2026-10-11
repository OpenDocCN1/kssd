
```asm
:10002FA5 85D2             test edx, edx <--看注册码是否为空

:10002FA7 0F841D010000     je 100030CA

:10002FAD 8BFD             mov edi, ebp

:10002FAF 83C9FF           or ecx, FFFFFFFF

:10002FB2 33C0             xor eax, eax

:10002FB4 F2               repnz

:10002FB5 AE               scasb

:10002FB6 F7D1             not ecx

:10002FB8 49               dec ecx

:10002FB9 0F840B010000     je 100030CA

:10002FBF 8BFA             mov edi, edx

:10002FC1 83C9FF           or ecx, FFFFFFFF

:10002FC4 F2               repnz

:10002FC5 AE               scasb

:10002FC6 F7D1             not ecx

:10002FC8 49               dec ecx

:10002FC9 0F84FB000000     je 100030CA

:10002FCF 53               push ebx

:10002FD0 32DB             xor bl, bl

:10002FD2 B908000000       mov ecx, 00000008

:10002FD7 8D7C2411         lea edi, dword ptr [esp+11]

:10002FDB 885C2410         mov byte ptr [esp+10], bl

:10002FDF 33D2             xor edx, edx

:10002FE1 F3               repz

:10002FE2 AB               stosd

:10002FE3 8BFD             mov edi, ebp

:10002FE5 83C9FF           or ecx, FFFFFFFF

:10002FE8 F2               repnz

:10002FE9 AE               scasb

:10002FEA F7D1             not ecx

:10002FEC 49               dec ecx

:10002FED 56               push esi

:10002FEE 85C9             test ecx, ecx

:10002FF0 894C243C         mov dword ptr [esp+3C], ecx

:10002FF4 7E0A             jle 10003000
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10002FFE(C)
```

```asm
:10002FF6 8A042A           mov al, byte ptr [edx+ebp]

:10002FF9 32D8             xor bl, al

:10002FFB 42               inc edx

:10002FFC 3BD1             cmp edx, ecx

:10002FFE 7CF6             jl 10002FF6
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10002FF4(C)
```

```asm
:10003000 33F6             xor esi, esi
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10003073(U)
```

```asm
:10003002 83FE08           cmp esi, 00000008

:10003005 7C07             jl 1000300E

:10003007 8D0409           lea eax, dword ptr [ecx+ecx]

:1000300A 3BF0             cmp esi, eax

:1000300C 7D67             jge 10003075
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10003005(C)
```

```asm
:1000300E 8BFD             mov edi, ebp

:10003010 83C9FF           or ecx, FFFFFFFF

:10003013 33C0             xor eax, eax

:10003015 F2               repnz

:10003016 AE               scasb

:10003017 8BC6             mov eax, esi

:10003019 99               cdq

:1000301A F7D1             not ecx

:1000301C 2BC2             sub eax, edx

:1000301E 49               dec ecx

:1000301F D1F8             sar eax, 1

:10003021 33D2             xor edx, edx

:10003023 F7F1             div ecx

:10003025 83FE01           cmp esi, 00000001

:10003028 8BFA             mov edi, edx

:1000302A 7E05             jle 10003031

:1000302C 8D46FF           lea eax, dword ptr [esi-01]

:1000302F EB02             jmp 10003033
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:1000302A(C)
```

```asm
:10003031 33C0             xor eax, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:1000302F(U)
```

```asm
:10003033 8A4C0414         mov cl, byte ptr [esp+eax+14]

:10003037 32CB             xor cl, bl

:10003039 80F18D           xor cl, 8D

:1000303C 8AD9             mov bl, cl

:1000303E 885C2410         mov byte ptr [esp+10], bl

:10003042 8B542410         mov edx, dword ptr [esp+10]

:10003046 52               push edx

:10003047 E8F4FEFFFF       call 10002F40

:1000304C 8B4C2440         mov ecx, dword ptr [esp+40]

:10003050 88443418         mov byte ptr [esp+esi+18], al

:10003054 2BCF             sub ecx, edi

:10003056 8A5429FF         mov dl, byte ptr [ecx+ebp-01]

:1000305A 32D0             xor dl, al

:1000305C 80F2D8           xor dl, D8

:1000305F 52               push edx

:10003060 E8DBFEFFFF       call 10002F40

:10003065 8B4C2444         mov ecx, dword ptr [esp+44]

:10003069 83C408           add esp, 00000008

:1000306C 88443415         mov byte ptr [esp+esi+15], al

:10003070 83C602           add esi, 00000002

:10003073 EB8D             jmp 10003002
```

这上面一打断又看不懂了，如果那位大哥看懂了的话，还请和我说说它的算法，让我彻底弄懂一下，先谢谢了！

下面开始将错误的注册码和正确的注册码进行比较：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:1000300C(C)
```

```asm
:10003075 8B742440         mov esi, dword ptr [esp+40] 这里是假码

:10003079 8D442414         lea eax, dword ptr [esp+14] 这里是真码
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:1000309F(C)
```

```asm
:1000307D 8A10             mov dl, byte ptr [eax]

:1000307F 8A1E             mov bl, byte ptr [esi]

:10003081 8ACA             mov cl, dl

:10003083 3AD3             cmp dl, bl

:10003085 752D             jne 100030B4

:10003087 84C9             test cl, cl

:10003089 7416             je 100030A1

:1000308B 8A5001           mov dl, byte ptr [eax+01]

:1000308E 8A5E01           mov bl, byte ptr [esi+01]

:10003091 8ACA             mov cl, dl

:10003093 3AD3             cmp dl, bl

:10003095 751D             jne 100030B4

:10003097 83C002           add eax, 00000002

:1000309A 83C602           add esi, 00000002

:1000309D 84C9             test cl, cl

:1000309F 75DC             jne 1000307D 前两位正确的话，继续比较后面的，挺有意思的:)
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10003089(C)
```

```asm
:100030A1 33C0             xor eax, eax

:100030A3 33C9             xor ecx, ecx

:100030A5 85C0             test eax, eax

:100030A7 5E               pop esi

:100030A8 5B               pop ebx

:100030A9 0F94C1           sete cl

:100030AC 5F               pop edi

:100030AD 8BC1             mov eax, ecx

:100030AF 5D               pop ebp

:100030B0 83C428           add esp, 00000028

:100030B3 C3               ret
```

大概就这么些吧，最后整理一下我的注册信息：

注册名：`Turkey`

注册码：`GHFwZETtIJRP`

哈哈，可以收工了，比较简单，对于我来说，实在是个练手的好东东。因为半个月前，我还只会爆破它，且爆破的不完全，这下终于搞定了，好开心啊，真是没有白来看雪的论坛，真心希望它能越办越好，越办越火。

哦，忘了写软件的下载地址了，加上：

`http://turkey.363.net/palplaza/download/ddr.zip` `4dd466d0e1e0cc123afbe504da2462ca`
