
```
004F55AB    8B00                    mov eax,dword ptr ds:[eax]

004F55AD    8B15 28B04700           mov edx,dword ptr ds:[47B028]                      ; Dumped.0047B074

004F55B3    E8 608BF7FF             call Dumped.0046E118

004F55B8    A1 6CA25100             mov eax,dword ptr ds:[51A26C]

004F55BD    8B00                    mov eax,dword ptr ds:[eax]

004F55BF    E8 D48BF7FF             call Dumped.0046E198

004F55C4    E8 D3F3F0FF             call Dumped.0040499C
```

修复好入口，再用`LordPE`修改导入表 `RAV=00339000`，大小=154，这样脱壳基本完成。

## 三、修复代码替换 Code Replace

`Code Replace`的基本原理是：选取5个字节两句指令，且一定是2+3或3+2，转换为一个`call`，`call`后还原时无规律的增加了如`add ecx,ebp`和`sub ecx,ebp`//`inc ebp`和`dec ebp`//`push esp`和`pop esp`//`xor esi,ebx`和`xor esi,ebx`等等一些垃圾代码。

`Code Replace`的地址也很容易得到

`Code Replace`代码:

```
0077702E    60                      pushad

0077702F    FC                      cld

----------------

----------------

----------------

007771D9    E8 22EF0000             call ACProtec.00786100//等于 mov ebp,37500

007771DE    8B4424 20               mov eax,dword ptr ss:[esp+20]

007771E2    33C9                    xor ecx,ecx

007771E4    8B9C8D 812E4000         mov ebx,dword ptr ss:[ebp+ecx\*4+402E81]//ebp+402E81=777E81

007771EB    039D 46F84000           add ebx,dword ptr ss:[ebp+40F846]

007771F1    3BC3                    cmp eax,ebx

007771F3    74 07                   je short ACProtec.007771FC

007771F5    90                      nop

007771F6    90                      nop

007771F7    90                      nop

007771F8    90                      nop

007771F9    41                      inc ecx

007771FA  ^ EB E8                   jmp short ACProtec.007771E4

007771FC    8DB5 615D4000           lea esi,dword ptr ss:[ebp+405D61]//ebp+405D61=77AD61

00777202    B8 0A000000             mov eax,0A

00777207    F7E1                    mul ecx

00777209    03F0                    add esi,eax

0077720B    8DBD 07184000           lea edi,dword ptr ss:[ebp+401807]

00777211    0FB6840D C9224000       movzx eax,byte ptr ss:[ebp+ecx+4022C9]

00777219    FEC0                    inc al

0077721B    88840D C9224000         mov byte ptr ss:[ebp+ecx+4022C9],al

00777222    3C 20                   cmp al,20

00777224    75 13                   jnz short ACProtec.00777239

00777226    90                      nop

00777227    90                      nop

00777228    90                      nop

00777229    90                      nop

0077722A    8BBD 4AF84000           mov edi,dword ptr ss:[ebp+40F84A]

00777230    B8 0A000000             mov eax,0A

00777235    F7E1                    mul ecx

00777237    03F8                    add edi,eax

00777239    8A9D 1E204000           mov bl,byte ptr ss:[ebp+40201E]//ss:[0077701E]=97

0077723F    B9 0A000000             mov ecx,0A

00777244    AC                      lods byte ptr ds:[esi]

00777245    32C3                    xor al,bl

00777247    AA                      stos byte ptr es:[edi]

00777248  ^ E2 FA                   loopd short ACProtec.00777244

0077724A    83EF 0A                 sub edi,0A

0077724D    57                      push edi

0077724E    8DB5 07184000           lea esi,dword ptr ss:[ebp+401807]

00777254    33F7                    xor esi,edi

00777256    74 19                   je short ACProtec.00777271

00777258    90                      nop

00777259    90                      nop

0077725A    90                      nop

0077725B    90                      nop

0077725C    8B7424 24               mov esi,dword ptr ss:[esp+24]

00777260    83EE 04                 sub esi,4

00777263    AD                      lods dword ptr ds:[esi]

00777264    81EF 2E204000           sub edi,ACProtec.0040202E

0077726A    2BFD                    sub edi,ebp

0077726C    03C7                    add eax,edi

0077726E    8946 FC                 mov dword ptr ds:[esi-4],eax

00777271    5F                      pop edi

00777272    57                      push edi

00777273    33C9                    xor ecx,ecx

00777275    83F9 08                 cmp ecx,8

00777278    74 0E                   je short ACProtec.00777288

0077727A    90                      nop

0077727B    90                      nop

0077727C    90                      nop

0077727D    90                      nop

0077727E    8B448C 04               mov eax,dword ptr ss:[esp+ecx\*4+4]

00777282    89048C                  mov dword ptr ss:[esp+ecx\*4],eax

00777285    41                      inc ecx

00777286  ^ EB ED                   jmp short ACProtec.00777275
```

为了完美还原回原处，我写了段还原代码如下：

```
0077702E    60                      pushad

0077702F    33C0                    xor eax,eax

00777031    BF 61AD7700             mov edi,ACProtec.0077AD61

00777036    833C38 00               cmp dword ptr ds:[eax+edi],0

0077703A    74 07                   je short ACProtec.00777043

0077703C    803438 97               xor byte ptr ds:[eax+edi],97

00777040    40                      inc eax

00777041  ^ EB F3                   jmp short ACProtec.00777036

00777043    33C9                    xor ecx,ecx

00777045    8B348D 817E7700         mov esi,dword ptr ds:[ecx\*4+777E81]

0077704C    83FE 00                 cmp esi,0

0077704F    75 06                   jnz short ACProtec.00777057

00777051    61                      popad

00777052    832C24 05               sub dword ptr ss:[esp],5

00777056    C3                      retn

00777057    BF 61AD7700             mov edi,ACProtec.0077AD61

0077705C    81C6 FBFF3F00           add esi,3FFFFB

00777062    B8 0A000000             mov eax,0A

00777067    F7E1                    mul ecx

00777069    03F8                    add edi,eax

0077706B    33C0                    xor eax,eax

0077706D    803C38 03               cmp byte ptr ds:[eax+edi],3

00777071    74 7D                   je short ACProtec.007770F0

00777073    803C38 33               cmp byte ptr ds:[eax+edi],33

00777077    0F84 D7000000           je ACProtec.00777154

0077707D    803C38 40               cmp byte ptr ds:[eax+edi],40

00777081    72 06                   jb short ACProtec.00777089

00777083    803C38 58               cmp byte ptr ds:[eax+edi],58

00777087    72 03                   jb short ACProtec.0077708C

00777089    40                      inc eax

0077708A  ^ EB E1                   jmp short ACProtec.0077706D

0077708C    8A1438                  mov dl,byte ptr ds:[eax+edi]

0077708F    80C2 08                 add dl,8

00777092    8BD8                    mov ebx,eax

00777094    43                      inc ebx

00777095    3A143B                  cmp dl,byte ptr ds:[ebx+edi]

00777098    74 07                   je short ACProtec.007770A1

0077709A    83FB 08                 cmp ebx,8

0077709D  ^ 72 F5                   jb short ACProtec.00777094

0077709F  ^ EB E8                   jmp short ACProtec.00777089

007770A1    2BD8                    sub ebx,eax

007770A3    83F8 00                 cmp eax,0

007770A6    75 22                   jnz short ACProtec.007770CA

007770A8    83FB 02                 cmp ebx,2

007770AB  ^ 72 DC                   jb short ACProtec.00777089

007770AD    4B                      dec ebx

007770AE    47                      inc edi

007770AF    33ED                    xor ebp,ebp

007770B1    8A142F                  mov dl,byte ptr ds:[edi+ebp]

007770B4    88142E                  mov byte ptr ds:[esi+ebp],dl

007770B7    83FD 04                 cmp ebp,4

007770BA    0F84 F3000000           je ACProtec.007771B3

007770C0    45                      inc ebp

007770C1    3BDD                    cmp ebx,ebp

007770C3  ^ 75 EC                   jnz short ACProtec.007770B1

007770C5    47                      inc edi
```
