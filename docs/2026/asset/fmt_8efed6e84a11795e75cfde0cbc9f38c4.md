
7473                    je 004890B9

:00489046 31C0
xor eax, eax

:00489048 31DB
xor ebx, ebx

:0048904A BFCCCCCC0C
mov edi, 0CCCCCCC

下面是逐个验证注册码各个字符计算出的数值的“合法性”，实际上间接验证了输入注册码的“合法性”。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00489055(C)
|

:0048904F 8A1E
mov bl, byte ptr [esi]

:00489051 46
inc esi

:00489052 80FB20
cmp bl, 20

:00489055 74F8
je 0048904F

:00489057 B500
mov ch, 00

:00489059 80FB2D
cmp bl, 2D

:0048905C
7469                    je 004890C7

:0048905E 80FB2B
cmp bl, 2B

:00489061 7466
je 004890C9

:00489063 80FB24
cmp bl, 24

:00489066 7466
je 004890CE

:00489068
80FB78                  cmp bl, 78

:0048906B 7461
je 004890CE

:0048906D 80FB58
cmp bl, 58

:00489070 745C
je 004890CE

:00489072 80FB30
cmp bl, 30

:00489075
7513                    jne 0048908A

:00489077 8A1E
mov bl, byte ptr [esi]

:00489079 46
inc esi

:0048907A 80FB78
cmp bl, 78

:0048907D
744F                    je 004890CE

:0048907F 80FB58
cmp bl, 58

:00489082 744A
je 004890CE

:00489084 84DB
test bl, bl

:00489086 7420
je 004890A8

:00489088 EB04
jmp 0048908E

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00489075(C), :004890CC(U)
|

:0048908A 84DB
test bl, bl

:0048908C 7434
je 004890C2

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00489088(U), :004890A6(C)
|

:0048908E 80EB30
sub bl, 30

:00489091
80FB09                  cmp bl, 09
//关键，从中我们可以推算出各位注册码的取值范围

:00489094 772C
ja 004890C2    //跳？？？不要！！！

:00489096 39F8
cmp eax, edi

:00489098 7728
ja 004890C2

:0048909A 8D0480
lea eax, dword ptr [eax+4\*eax]

:0048909D 01C0
add eax, eax

:0048909F 01D8
add eax, ebx  //每次 "bl-30" 的值组成串再转换成数值

:004890A1
8A1E                    mov
bl, byte ptr [esi]

:004890A3 46
inc esi

:004890A4 84DB
test bl, bl

:004890A6 75E6
jne 0048908E

计算完毕，回到……

:0041BA1B B97C000000
mov ecx, 0000007C

:0041BA20 99
cdq

:0041BA21 F7F9
idiv ecx

:0041BA23
83C064                  add eax, 00000064

:0041BA26 894584
mov dword ptr [ebp-7C], eax

:0041BA29 8B4584
mov eax, dword ptr [ebp-7C]  //eax=eax/0x7C+0x64

:0041BA2C 2B4588
sub eax, dword ptr [ebp-78]  //eax-用户名计算出的值

:0041BA2F 83F864
cmp eax, 00000064　　　　　　 //等于 0x64 则注册成功

:0041BA32 0F85BC020000
jne 0041BCF4

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

实例分析：
以用户名 `PaulYoung` 为例，
用户名计算：
`0xFFFFFF9C+0x50+0x61+0x75+0x6C+0x59+0x6F0x75+0x6E+0x67=0x340=832`
`832*0.6480041472265422897+1233.99999999999999957`结果的整数为1773
`1773*3121.14159259999996697388632872504`结果的整数为`0x547058`

注册码计算：
假设某位注册码字符为 `S` ，则
奇数位计算：`S+0xFFFFFFEC+(x/3` 的商)  （`x` 依次等于0,1,2,3...）
偶数位计算：`S+0xFFFFFF-0xA+(x/3` 的商)

而从
:0048908E
80EB30                  sub bl, 30
:00489091 80FB09
cmp bl, 09
:00489094 772C
ja 004890C2
我们可以可以看到，奇数位及偶数位注册码取值范围分别是：
奇数位范围：`0x30≤S+0xFFFFFFEC+(x/3)≤0x39`
偶数位范围：`0x30≤S+0xFFFFFF-0xA+(x/3)≤0x39`

由此我们可以推算出注册码的“合法”取值范围，分别是：
第一位：D,E,F,G,H,I,J,K,L,M
第二位：N,O,P,Q,R,S,T,U,V,W
第三位：D,E,F,G,H,I,J,K,L,M
第四位：M,N,O,P,Q,R,S,T,U,V
第五位：C,D,E,F,G,H,I,J,K,L
第六位：M,N,O,P,Q,R,S,T,U,V
第七位：B,C,D,E,F,G,H,I,J,K
第八位：L,M,N,O,P,Q,R,S,T,U
第九位：B,C,D,E,F,G,H,I,J,K
...
注册码计算出的值假设为 `N` ，(`N/0x7C`的商)+0x4-
