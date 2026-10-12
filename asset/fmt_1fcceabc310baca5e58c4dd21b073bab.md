
### 代码段 1：字符转换

```
:004059CA 8A4601                  mov al, byte ptr [esi+01]  
:004059CD 46                      inc esi  
:004059CE 84C0                    test al, al  
:004059D0 75C4                    jne 00405996  
```

//=========================================================

//上面这段简单明了，它对 `key` 中的字符进行了一次转换，把一些相对特殊的字符用普通字符代替

//我认为写注册机时无需将这段的逆过程写进去，因为转换前后都是可输入字符，没有本质区别

//为了便于说明，将经过这次转换后的 `key` 称为 `key1`，以后的 `key2`、`key3` 等类似

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00405994(C)

|

//=========================================================

### 代码段 2：Key1 到 Key2 转换

```
:004059D2 8A4500                  mov al, byte ptr [ebp+00]    //[ebp]=key1
:004059D5 8BCD                    mov ecx, ebp    //[ecx]->key1
:004059D7 84C0                    test al, al
:004059D9 742E                    je 00405A09
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00405A07(C)

|

```
:004059DB 8A01                   mov al, byte ptr [ecx]
:004059DD 3C30                   cmp al, 30
:004059DF 7C08                   jl 004059E9
:004059E1 3C39                   cmp al, 39
:004059E3 7F04                   jg 004059E9
:004059E5 2C30                   sub al, 30    //在'0'和'9'之间则减去 30h，变为 0 到 9
:004059E7 EB16                   jmp 004059FF
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004059DF(C), :004059E3(C)

|

```
:004059E9 3C40                   cmp al, 40
:004059EB 7C08                   jl 004059F5
:004059ED 3C5B                   cmp al, 5B
:004059EF 7F04                   jg 004059F5
:004059F1 2C36                   sub al, 36    //在'@'和'['之间则减去 36h，变为 0ah 到 25h
```

//注：ASCII 表中，`@`在`A`前一个，`[`在`Z`后一个

```
:004059F3 EB0A                   jmp 004059FF
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004059EB(C), :004059EF(C)

|

```
:004059F5 3C61                   cmp al, 61
:004059F7 7C08                   jl 00405A01
:004059F9 3C7A                   cmp al, 7A
:004059FB 7F04                   jg 00405A01
:004059FD 2C3B                   sub al, 3B    //在'a'和'z'之间则减去 3bh，变为 26h 到 3fh
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004059E7(U), :004059F3(U)

|

```
:004059FF 8801                   mov byte ptr [ecx], al
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004059F7(C), :004059FB(C)

|

```
:00405A01 8A4101                 mov al, byte ptr [ecx+01]
:00405A04 41                     inc ecx
:00405A05 84C0                   test al, al
:00405A07 75D2                   jne 004059DB
```

//=========================================================

//上面这段程序将 `key1` 中的 ASCII 字符转换成 0~3fh 的 16 进制数，产生 `key2`

//从 `key` 到 `key2`，其长度保持不变

//这个转换很简单，其逆过程也很容易写出来，继续往下看

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004059D9(C)

|

//=========================================================

### 代码段 3：Key2 到 Key3 转换

```
:00405A09 8B7C241C               mov edi, dword ptr [esp+1C]    //[edi]->key
:00405A0D 83C9FF                 or ecx, FFFFFFFF
:00405A10 33C0                   xor eax, eax
:00405A12 33F6                   xor esi, esi
:00405A14 33D2                   xor edx, edx
:00405A16 F2                     repnz
:00405A17 AE                     scasb
:00405A18 F7D1                   not ecx
:00405A1A 49                     dec ecx    //ecx->keyLength
:00405A1B 83E904                 sub ecx, 00000004
:00405A1E 785E                   js 00405A7E    //key 的长度小于 4 则跳转，正常情况下不会跳转
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00405A7C(C)

|

```
:00405A20 83FE0D                 cmp esi, 0000000D
```

//这是个长度界限，结合下面的程序可知，经过这次转换产生的 `key3` 最多有 15 个字节

//在这里说明一下，`[ebp]=key2`，`edx` 为其偏移量；`[ebx]=key3`，`esi` 为其偏移量

//值得注意的是，经过 `key1` 到 `key2` 的转换，正常情况下，`key2` 的每个字节均应在 0 到 3fh 之间，即高两位为 0

```
:00405A23 7359                   jnb 00405A7E
:00405A25 8A4C2A01               mov cl, byte ptr [edx+ebp+01]    //取出 key2 的第 2、6、10、14 等字节
:00405A29 8A042A                 mov al, byte ptr [edx+ebp]    //取出 key2 的第 1、5、9、13 等字节
:00405A2C C0E106                 shl cl, 06
:00405A2F 243F                   and al, 3F
:00405A31 8B7C241C               mov edi, dword ptr [esp+1C]
:00405A35 02C8                   add cl, al    //此时 cl 低 6 位为 0，al 高 2 位为 0
:00405A37 880C1E                 mov byte ptr [esi+ebx], cl    //存入 key3 的第 1、4、7、10 等字节
:00405A3A 8A4C2A01               mov cl, byte ptr [edx+ebp+01]    //取出 key2 的第 2、6、10、14 等字节
:00405A3E 8A442A02               mov al, byte ptr [edx+ebp+02]    //取出 key2 的第 3、7、11、15 等字节
:00405A42 46                     inc esi
:00405A43 42                     inc edx
:00405A44 C0F902                 sar cl, 02
:00405A47 80E10F                 and cl, 0F
:00405A4A C0E004                 shl al, 04
:00405A4D 02C8                   add cl, al    //此时 cl 高 4 位为 0，al 低 4 位为 0
:00405A4F 880C1E                 mov byte ptr [esi+ebx], cl    //存入 key3 的第 2、5、8、11 等字节
:00405A52 8A4C2A01               mov cl, byte ptr [edx+ebp+01]    //取出 key2 的第 3、7、11、15 等字节
:00405A56 8A442A02               mov al, byte ptr [edx+ebp+02]    //取出 key2 的第 4、8、12、16 等字节
:00405A5A 46                     inc esi
:00405A5B 42                     inc edx
:00405A5C C0F904                 sar cl, 04
:00405A5F 80E103                 and cl, 03
:00405A62 83C202                 add edx, 00000002
:00405A65 C0E002                 shl al, 02
:00405A68 02C8                   add cl, al    //此时 cl 高 6 位为 0，al 低 2 位为 0
:00405A6A 33C0                   xor eax, eax
:00405A6C 880C1E                 mov byte ptr [esi+ebx], cl    //存入 key3 的第 3、6、9、12 等字节
:00405A6F 83C9FF                 or ecx, FFFFFFFF
:00405A72 46                     inc esi
:00405A73 F2                     repnz
:00405A74 AE                     scasb
:00405A75 F7D1                   not ecx
:00405A77 83C1FB                 add ecx, FFFFFFFB    //ecx->keyLength-4
:00405A7A 3BD1                   cmp edx, ecx
:00405A7C 7EA2                   jle 00405A20
```

//=========================================================

//上面这段似乎挺麻烦，不过稍加分析就可理解

//`key2` 的每个字节均应在 0 到 3fh 之间，即只有低 6 位是有效的，4 个 `key2` 字节便有 24 位有效位，正好占满 3 字节

//上面的程序正是将 `key2` 中相邻的 4 字节压缩为 3 字节，去掉无用的永 0 位

//显然，它的逆过程就是将 `key3` 中相邻的 3 字节扩充为 4 字节，其中插入 8 个 0 位

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00405A1E(C), :00405A23(C)

|

//=========================================================

### 代码段 4：Key3 到 Key4 转换

```
:00405A7E 33FF                   xor edi, edi    //edi->0
:00405A80 33F6                   xor esi, esi
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00405AA9(C)

|

```
:00405A82 8BC6                   mov eax, esi    //eax->0
:00405A84 B930000000             mov ecx, 00000030
:00405A89 99                     cdq
:00405A8A F7F9                   idiv ecx
:00405A8C 52                     push edx
:00405A8D E88EFEFFFF             call 00405920
```

//`eax`->看似没有规律的大数，这个 `call` 的子程序很短，有点像随机数发生器，不过种子应该是固定的

```
:00405A92 99                     cdq
:00405A93 B90D000000             mov ecx, 0000000D
:00405A98 83C404                 add esp, 00000004
:00405A9B F7F9                   idiv ecx    //作了个除法，edx->余数，在 0 到 0dh 之间
:00405A9D 8A041E                 mov al, byte ptr [esi+ebx]    //[ebx]->key3，esi 是偏移量
:00405AA0 02C2                   add al, dl    //给 key3 的每个字节加上了一个 0 到 0dh 之间的数
:00405AA2 88041E                 mov byte ptr [esi+ebx], al
:00405AA5 46                     inc esi
:00405AA6 83FE10                 cmp esi, 00000010
:00405AA9 72D7                   jb 00405A82
```

//=========================================================

//这段程序对 `key3` 作了一次转换产生 `key4`，其中调用的子程序我没看太懂，不过好像也没什么影响。
