
```
   test eax, eax
:00403D11 75DA
         jne 00403CED
:00403D13 83FD02
             cmp ebp, 00000002
     ====>EBP不等于2则跳！
:00403D16
7523                    jne
00403D3B
             ====>跳则OVER！
:00403D18
85F6                    test
esi, esi
:00403D1A 741F
   je 00403D3B
:00403D1C 8B4C2414
     mov ecx, dword ptr [esp+14]
:00403D20 51
                 push ecx
       ====>ECX=2468
:00403D21
56                      push
esi
             ====>ESI=9012
:00403D22
57                      push
edi
             ====>EDI=1357
:00403D23
E808FFFFFF              call 00403C30
       ====>核心CALL！进入！
:00403D28
83C40C                  add esp,
0000000C
:00403D2B 8BF0
   mov esi, eax
:00403D2D 53
         push ebx
\* Reference
To: KERNEL32.LocalFree, Ord:01CCh
                     |
:00403D2E
FF154CD24100            Call dword ptr [0041D24C]
:00403D34
8BC6                    mov
eax, esi
:00403D36 5F
     pop edi
:00403D37 5E
           pop esi
:00403D38 5D
                 pop ebp
:00403D39
5B                      pop
ebx
:00403D3A C3
   ret
\* Referenced by a (U)nconditional
or (C)onditional Jump at Addresses:
|:00403CE7(C), :00403D16(C), :00403D1A(C)
|
:00403D3B
53                      push
ebx
\* Reference To: KERNEL32.LocalFree, Ord:01CCh
         |
:00403D3C FF154CD24100
     Call dword ptr [0041D24C]
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403C8C(C), :00403C95(C),
:00403CB3(C)
|
:00403D42 5F
       pop edi
:00403D43 5E
             pop esi
:00403D44 5D
                   pop ebp
:00403D45
33C0                    xor
eax, eax
             ====>清0则OVER！  爆破点①！
:00403D47
5B                      pop
ebx
:00403D48 C3
   ret
```

---

### 进入核心 CALL：`403D23`

```
 call 00403C30
\* Referenced by a CALL at
Address:
|:00403D23
|
:00403C30 8B442404
         mov eax, dword ptr [esp+04]
     ====>EAX=1357
:00403C34 53
                     push
ebx
:00403C35 85C0
   test eax, eax
:00403C37 57
           push edi
:00403C38 7440
               je 00403C7A
:00403C3A
8B7C2410                mov edi, dword
ptr [esp+10]
               ====>EDI=9012
:00403C3E
85FF                    test
edi, edi
:00403C40 7438
   je 00403C7A
:00403C42 8B5C2414
     mov ebx, dword ptr [esp+14]
 ====>EBX=2468
:00403C46 85DB
               test ebx, ebx
:00403C48
7430                    je 00403C7A
:00403C4A
56                      push
esi
:00403C4B 50
   push eax
:00403C4C E83FFFFFFF
   call 00403B90
      一、         ====>对1357进行运算得出C62B
:00403C51
57                      push
edi
:00403C52 8BF0
   mov esi, eax
                   ====>ESI=EAX=C62B
:00403C54
E837FFFFFF              call 00403B90
                  二、
    ====>对9012进行运算得出66866
:00403C59
53                      push
ebx
:00403C5A 8BF8
   mov edi, eax
                   ====>EDI=EAX=66866
:00403C5C
E82FFFFFFF              call 00403B90
                  三、
    ====>对2468进行运算得出181A0
:00403C61
57                      push
edi
:00403C62 56
   push esi
:00403C63 8BD8
       mov ebx, eax
                       ====>EBX=EAX=181A0
:00403C65
E886FFFFFF              call 00403BF0
                  四、
    ====>运算CALL!进入!
:00403C6A 83C414
                 add esp, 00000014
:00403C6D
33C9                    xor
ecx, ecx
:00403C6F 3BC3
   cmp eax, ebx
                   ====>比较了！如果这2者相等则OK！爆破点②！
         ====>EAX=498545E
   ====>EBX=181A0
:00403C71 0F94C1
                 sete cl
:00403C74
5E                      pop
esi
:00403C75 5F
   pop edi
:00403C76 8BC1
       mov eax, ecx
:00403C78 5B
             pop ebx
:00403C79 C3
                   ret
```

---

### 进入运算一、二、三：`call 00403B90`

```
 call 00403B90
\* Referenced
by a CALL at Addresses:
|:00403C4C   , :00403C54   , :00403C5C
|
:00403B90 8B542404                mov
edx, dword ptr [esp+04]
:00403B94 33C0
       xor eax, eax
:00403B96 85D2
             test edx, edx
:00403B98 744F
                   je 00403BE9
:00403B9A
8A0A                    mov
cl, byte ptr [edx]
    一、         ====>CL=依次取1357字符的HEX值
                ―――――――――――――――――――――――――
                     二、
        ====>CL=依次取9012字符的HEX值
            ―――――――――――――――――――――――――
                  三、
    ====>CL=依次取2468字符的HEX值
:00403B9C
84C9                    test
cl, cl
:00403B9E 744C
   je 00403BEC
\* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00403BE6(C)
|
:00403BA0 80F930
               cmp cl, 30
:00403BA3
7C11                    jl 00403BB6
:00403BA5
80F939                  cmp cl, 39
:00403BA8
7F0C                    jg 00403BB6
:00403BAA
0FBEC9                  movsx ecx,
cl
:00403BAD 8D04C0
 lea eax, dword ptr [eax+8\*eax]
       一、    1、   ====>EAX=00   \* 9=0
   2、   ====>EAX=01   \* 9=9
                   3、
====>EAX=27   \* 9=15F
             4、   ====>EAX=581  \*
9=3189
      ―――――――――――――――――――――――――
           二、    1、   ====>EAX=00
  \* 9=0
         2、   ====>EAX=09   \* 9=51
 3、   ====>EAX=144  \* 9=B64
                   4、
====>EAX=2D91 \* 9=19A19
      ―――――――――――――――――――――――――
           三、    1、   ====>EAX=00
  \* 9=0
         2、   ====>EAX=02   \* 9=12
 3、   ====>EAX=4C   \* 9=2AC
                   4、
====>EAX=AB6  \* 9=6066
:00403BB0 8D4481D0
               lea eax, dword ptr [ecx+4\*eax-30]
                   一、
 1、   ====>EAX=31 + 4\*0    -30=1
                     2、
  ====>EAX=33 + 4\*9    -30=27
                   3、
====>EAX=35 + 4\*15F  -30=581
               4、   ====>EAX=37
+ 4\*3189 -30=C62B
        ―――――――――――――――――――――――――
             二、    1、   ====>EAX=39
+ 4\*0     -30=9
             2、   ====>EAX=30 + 4\*51
   -30=144
           3、   ====>EAX=31 + 4\*B64
-30=2D91
       4、   ====>EAX=32 + 4\*19A19 -30=66866
―――――――――――――――――――――――――
     三、    1、   ====>EAX=32 + 4\*0    -30=2
   2、   ====>EAX=34 + 4\*12   -30=4C
                     3、
  ====>EAX=36 + 4\*2AC  -30=AB6
                 4、   ====>EAX=38
+ 4\*6066 -30=181A0
:00403BB4 EB2A
               jmp 00403BE0
         ====>如果是字母的话则进行下面的运算！
\* Referenced by
a (U)nconditional or (C)onditional Jump at Addresses:
|:00403BA3(C), :00403BA8(C)
|
:00403BB6
80F941                  cmp cl, 41
:00403BB9
7C11                    jl 00403BCC
:00403BBB
80F95A                  cmp cl, 5A
:00403BBE
7F0C                    jg 00403BCC
:00403BC0
0FBEC9                  movsx ecx,
cl
:00403BC3 8D04C0
 lea eax, dword ptr [eax+8\*eax]
:00403BC6 8D4481C9
         lea eax, dword ptr [ecx+4\*eax-37]
:00403BCA
EB14                    jmp
00403BE0
\* Referenced by a (U)nconditional or (C)onditional
Jump at Addresses:
|:00403BB9(C), :00403BBE(C)
|
:00403BCC 80F961
               cmp cl, 61
:00403BCF
7C18                    jl 00403BE9
:00403BD1
80F97A                  cmp cl, 7A
:00403BD4
7F13                    jg 00403BE9
:00403BD6
0FBEC9                  movsx ecx,
cl
:00403BD9 8D04C0
 lea eax, dword ptr [eax+8\*eax]
:00403BDC 8D4481A9
         lea eax, dword ptr [ecx+4\*eax-57]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403BB4(U),
:00403BCA(U)
|
:00403BE0 8A4A01
     mov cl, byte ptr [edx+01]
:00403BE3 42
               inc edx
:00403BE4 84C9
                   test cl,
cl
:00403BE6 75B8
 jne 00403BA0
                 ====>继续循环？！
:00403BE8
C3                      ret
```

---

根据 afanty 大哥的指教，这段代码其实是将 `1357`、`2468`、`9012` 看成36进制数，然后转换成16进制数的结果。

`X=1357`（36进制）=`C62B`（H）；`Y=2468`（36）=`181A0`（H）；`Z=9012`（36）=`66866`（H）

---

### 进入运算 CALL 四：`00403C65`

```
 call 00403BF0
\* Referenced by a CALL at
Address:
|:00403C65
|
:00403BF0 8B4C2404
         mov ecx, dword ptr [esp+04]
     ====>ECX=[esp+04]=C62B
:00403BF4
56                      push
esi
:00403BF5 8BC1
   mov eax, ecx
                   ====>EAX=ECX=C62B
:00403BF7
```
