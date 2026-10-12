
# CSharp

> 原文：[`www.pediy.com/kssd/pediy03/forum438.htm`](https://www.pediy.com/kssd/pediy03/forum438.htm)

该软件的注册码存放在其目录下的 `CSharpEd.bin` 文件中。文件结构如下：

```csharp
struct RegInfo
{
    DWORD UserNameLength;                 //值等于 USER_NAME_LENGTH
    char  UserName[USER_NAME_LENGTH];      //包括末尾的'\0'字符
    DWORD RegCodeLength;
    char  RegCode[REG_CODE_LENGTH];
}
```

启动时判断注册码：

```asm
* Possible StringData Ref from Data Obj ->"CSharpEd.bin"

                                  |
:00406D45 689C244F00              push 004F249C
:00406D4A 8D4C2410                lea ecx, dword ptr [esp+10]
:00406D4E 50                      push eax
:00406D4F 51                      push ecx
:00406D50 C744243000000000        mov [esp+30], 00000000
:00406D58 E80E760800             call 0048E36B
:00406D5D 8D4C2410                lea ecx, dword ptr [esp+10]
:00406D61 C644242402             mov [esp+24], 02
:00406D66 E86B730800             call 0048E0D6
:00406D6B 8B54240C                mov edx, dword ptr [esp+0C]

* Possible StringData Ref from Data Obj ->"S3"

                                  |
:00406D6F 6898244F00              push 004F2498
:00406D74 8D442408                lea eax, dword ptr [esp+08]
:00406D78 52                      push edx
:00406D79 50                      push eax
:00406D7A E8814B0100             call 0041B900
:00406D7F 83C40C                  add esp, 0000000C
:00406D82 8B4C2404                mov ecx, dword ptr [esp+04]
:00406D86 C644242403             mov [esp+24], 03
:00406D8B 8B41F8                  mov eax, dword ptr [ecx-08]
:00406D8E 85C0                    test eax, eax
:00406D90 751A                    jne 00406DAC
:00406D92 E8F9180000             call 00408690
:00406D97 8B501C                  mov edx, dword ptr [eax+1C]
:00406D9A 6A00                    push 00000000
:00406D9C 6A00                    push 00000000
:00406D9E 6A10                    push 00000010
:00406DA0 52                      push edx

* Reference To: USER32.PostMessageA, Ord:01DEh

                                  |
:00406DA1 FF1544774C00           Call dword ptr [004C7744]
:00406DA7 E9CD000000             jmp 00406E79

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406D90(C)
|
:00406DAC 8D442404                lea eax, dword ptr [esp+04]
:00406DB0 56                      push esi
:00406DB1 50                      push eax
:00406DB2 8D4C2410                lea ecx, dword ptr [esp+10]

* Possible StringData Ref from Data Obj ->"Antechinus C# Editor Licensed "
                                  ->"by "
                                  |
:00406DB6 6874244F00              push 004F2474
:00406DBB 51                      push ecx
:00406DBC E81E760800             call 0048E3DF
```

跟进上面判断注册码的地方，用 `ReadFile` 设断点，最终会看见计算注册码的地方，拷下来就可以做一个 keyfile maker 了。

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BB4E(C)
|
:0041BB45 0FBE1431                movsx edx, byte ptr [ecx+esi]
:0041BB49 03C2                    add eax, edx
:0041BB4B 41                      inc ecx
:0041BB4C 3BCF                    cmp ecx, edi
:0041BB4E 7CF5                    jl 0041BB45

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BB43(C)
|
:0041BB50 99                      cdq
:0041BB51 F7FF                    idiv edi
:0041BB53 53                      push ebx
:0041BB54 55                      push ebp
:0041BB55 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BB59 8BE8                    mov ebp, eax
:0041BB5B 55                      push ebp
:0041BB5C 6A00                    push 00000000
:0041BB5E E8B92A0700             call 0048E61C
:0041BB63 8BC7                    mov eax, edi
:0041BB65 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BB69 99                      cdq
:0041BB6A 2BC2                    sub eax, edx
:0041BB6C D1F8                    sar eax, 1
:0041BB6E 8D1C30                  lea ebx, dword ptr [eax+esi]
:0041BB71 0FBE0430                movsx eax, byte ptr [eax+esi]
:0041BB75 03C5                    add eax, ebp
:0041BB77 99                      cdq
:0041BB78 2BC2                    sub eax, edx
:0041BB7A D1F8                    sar eax, 1
:0041BB7C 50                      push eax
:0041BB7D 6A01                    push 00000001
:0041BB7F E8982A0700             call 0048E61C
:0041BB84 8A4437FD                mov al, byte ptr [edi+esi-03]
:0041BB88 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BB8C 50                      push eax
:0041BB8D 6A02                    push 00000002
:0041BB8F E8882A0700             call 0048E61C
:0041BB94 0FBE4E05                movsx ecx, byte ptr [esi+05]
:0041BB98 0FBE5604                movsx edx, byte ptr [esi+04]
:0041BB9C 0FBE4603                movsx eax, byte ptr [esi+03]
:0041BBA0 03CA                    add ecx, edx
:0041BBA2 03C8                    add ecx, eax
:0041BBA4 B856555555              mov eax, 55555556
:0041BBA9 F7E9                    imul ecx
:0041BBAB 8BCA                    mov ecx, edx
:0041BBAD C1E91F                  shr ecx, 1F
:0041BBB0 8D440A03                lea eax, dword ptr [edx+ecx+03]
:0041BBB4 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BBB8 89442410                mov dword ptr [esp+10], eax
:0041BBBC 0420                    add al, 20
:0041BBBE 50                      push eax
:0041BBBF 6A03                    push 00000003
:0041BBC1 E8562A0700             call 0048E61C
:0041BBC6 8B542410                mov edx, dword ptr [esp+10]
:0041BBCA 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BBCE 8D042A                  lea eax, dword ptr [edx+ebp]
:0041BBD1 99                      cdq
:0041BBD2 2BC2                    sub eax, edx
:0041BBD4 D1F8                    sar eax, 1
:0041BBD6 2C1C                    sub al, 1C
:0041BBD8 50                      push eax
:0041BBD9 6A04                    push 00000004
:0041BBDB E83C2A0700             call 0048E61C
:0041BBE0 8A4437FE                mov al, byte ptr [edi+esi-02]
:0041BBE4 8D6C37FE                lea ebp, dword ptr [edi+esi-02]
:0041BBE8 0420                    add al, 20
:0041BBEA 8D4C242C                lea ecx, dword ptr [esp+2C]
:0041BBEE 50                      push eax
:0041BBEF 6A05                    push 00000005
:0041BBF1 E8262A0700             call 0048E61C
:0041BBF6 8A4B02                  mov cl, byte ptr [ebx+02]
:0041BBF9 80C103                  add cl, 03
:0041BBFC 51                      push ecx
:0041BBFD 6A06                    push 00000006
:0041BBFF 8D4C2434                lea ecx, dword ptr [esp+34]
:0041BC03 E8142A0700             call 0048E61C
:0041BC08 0FBE4C37FF              movsx ecx, byte ptr [edi+esi-01]
:0041BC0D 0FBE5602                movsx edx, byte ptr [esi+02]
:0041BC11 0FBE4601                movsx eax, byte ptr [esi+01]
:0041BC15 03CA                    add ecx, edx
:0041BC17 0FBE5500                movsx edx, byte ptr [ebp+00]
:0041BC1B 03C8                    add ecx, eax
:0041BC1D 0FBE06                  movsx eax, byte ptr [esi]
:0041BC20 03CA                    add ecx, edx
:0041BC22 03C8                    add ecx, eax
:0041BC24 B867666666              mov eax, 66666667
:0041BC29 F7E9                    imul ecx
:0041BC2B D1FA                    sar edx, 1
:0041BC2D 8BCA                    mov ecx, edx
```



### 汇编代码片段

```
:0041BC2F C1E91F                shr ecx, 1F

:0041BC32 03D1                  add edx, ecx

:0041BC34 8D4C242C              lea ecx, dword ptr [esp+2C]

:0041BC38 80EA03                sub dl, 03

:0041BC3B 52                    push edx

:0041BC3C 6A07                  push 00000007

:0041BC3E E8D9290700            call 0048E61C

:0041BC43 33DB                  xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BC75(C)

|

:0041BC45 8B54242C              mov edx, dword ptr [esp+2C]

:0041BC49 8A0413                mov al, byte ptr [ebx+edx]

:0041BC4C 3C5B                  cmp al, 5B

:0041BC4E 7C08                  jl 0041BC58

:0041BC50 3C60                  cmp al, 60

:0041BC52 7F04                  jg 0041BC58

:0041BC54 2C2A                  sub al, 2A

:0041BC56 EB0E                  jmp 0041BC66

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0041BC4E(C), :0041BC52(C)

|

:0041BC58 3C41                  cmp al, 41

:0041BC5A 7C04                  jl 0041BC60

:0041BC5C 3C7A                  cmp al, 7A

:0041BC5E 7E11                  jle 0041BC71

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BC5A(C)

|

:0041BC60 8AC3                  mov al, bl

:0041BC62 D0E0                  shl al, 1

:0041BC64 045D                  add al, 5D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BC56(U)

|

:0041BC66 50                    push eax

:0041BC67 53                    push ebx

:0041BC68 8D4C2434              lea ecx, dword ptr [esp+34]

:0041BC6C E8AB290700            call 0048E61C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BC5E(C)

|

:0041BC71 43                    inc ebx

:0041BC72 83FB08                cmp ebx, 00000008

:0041BC75 7CCE                  jl 0041BC45

:0041BC77 8B742428              mov esi, dword ptr [esp+28]

:0041BC7B 8D4C242C              lea ecx, dword ptr [esp+2C] //计算出来的注册码

:0041BC7F 51                    push ecx

:0041BC80 8BCE                  mov ecx, esi

:0041BC82 E8C4210700            call 0048DE4B
```

### 注册码比较

算完之后就是比较注册码：

```
:0041B86C E85F020000            call 0041BAD0      //计算注册码

:0041B871 8B44241C              mov eax, dword ptr [esp+1C]

:0041B875 83C408                add esp, 00000008

:0041B878 8D74245C              lea esi, dword ptr [esp+5C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B89E(C)

|

:0041B87C 8A10                  mov dl, byte ptr [eax]    //正确的注册码

:0041B87E 8A1E                  mov bl, byte ptr [esi]    //假注册码

:0041B880 8ACA                  mov cl, dl

:0041B882 3AD3                  cmp dl, bl

:0041B884 751E                  jne 0041B8A4

:0041B886 84C9                  test cl, cl

:0041B888 7416                  je 0041B8A0

:0041B88A 8A5001                mov dl, byte ptr [eax+01]

:0041B88D 8A5E01                mov bl, byte ptr [esi+01]

:0041B890 8ACA                  mov cl, dl

:0041B892 3AD3                  cmp dl, bl

:0041B894 750E                  jne 0041B8A4

:0041B896 83C002                add eax, 00000002

:0041B899 83C602                add esi, 00000002

:0041B89C 84C9                  test cl, cl

:0041B89E 75DC                  jne 0041B87C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B888(C)

|

:0041B8A0 33C0                  xor eax, eax

:0041B8A2 EB05                  jmp 0041B8A9
```

### 设置全局标志

比较完毕之后它会在下面的地方放一个全局标志：

```
* Referenced by a CALL at Addresses:

|:00406534  , :0040B7DB  , :0040BA5E 

|

:00409160 A0A9A24F00              mov al, byte ptr [004FA2A9]

:00409165 C3                      ret
```

### 检查标志位

以后就判断这个标志位：

```
:0040B7DB E880D9FFFF              call 00409160  //取标志位

:0040B7E0 84C0                    test al, al

:0040B7E2 751E                    jne 0040B802

:0040B7E4 6AFF                    push FFFFFFFF

:0040B7E6 6A00                    push 00000000

* Possible Reference to String Resource ID=00223: "Working with projects is only available to registered users"

|

:0040B7E8 68DF000000              push 000000DF
```

BTW：老外可真及时，`C#`刚出不久就出了相应的软件产品，值得学习。 `4dd466d0e1e0cc123afbe504da2462ca`
