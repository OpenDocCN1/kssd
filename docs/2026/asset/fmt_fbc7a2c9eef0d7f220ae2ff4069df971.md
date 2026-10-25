
# 豪杰超级解霸3000[英雄版]

(15千字)

看见有朋友分析了一下XXXXX3000英雄版的注册码的比较过程的分析，就想也分析一下，写出它的注册码的生成的算法过程,也写了一个注册机。

所用工具：`TRW2000`、`W32DASM`

```
Referenced by a CALL at Addresses:
 |:00401C93  , :00401EF9
 |
 :00402840 83EC20
  sub esp, 00000020
 :00402843 56
          push esi
 :00402844 57
                push edi
 :00402845
B908000000              mov ecx, 00000008
:0040284A 33C0
xor eax, eax
 :0040284C 8D7C2408
    lea edi, dword ptr [esp+08]<----存入第一组注册码的局部变量，以下记为reg1，是一个DWORD类型，也可以看成4个字符形式。
 :00402850 F3
    repz
 :00402851 AB
          stosd
 :00402852 8B44242C
            mov eax, dword ptr [esp+2C]  <----得到用户名的地址
 :00402856 50
    push eax  <----将用户名作为参数传入
 :00402857 E8B4010000
            call 00402A10 <----此CALL（记为CALL 1）内将用户名进行变化后在EAX以长整数返回,按F8进入.
 :0040285C 83C404
add esp, 00000004
 :0040285F 89442408
    mov dword ptr [esp+08], eax <----将返回的值放入reg1中
 :00402863
33F6                    xor
esi, esi
 \* Referenced by a (U)nconditional or (C)onditional Jump at
Address:
 |:00402888(C)
 |
 :00402865 0FBE443408
        movsx eax, byte ptr [esp+esi+08]
 :0040286A 83F841
                cmp eax, 00000041
:0040286D 7C08
jl 00402877
 :0040286F 83F85A
    cmp eax, 0000005A
 :00402872 7F03
            jg 00402877
 :00402874 83C020
                add eax, 00000020 <----
如果是大写的，则变为小写。
 \* Referenced by a (U)nconditional or (C)onditional Jump
at Addresses:
 |:0040286D(C), :00402872(C)
 |
 :00402877 50
                    push eax
 :00402878 E863020000              call
00402AE0 <---- 将EAX 转化为一个字符。（记为CALL 3）
 :0040287D 83C404
            add esp, 00000004
 :00402880 88443408
              mov byte ptr [esp+esi+08], <----保存这个字符到reg1中。
 :00402884 46
    inc esi
 :00402885 83FE04
        cmp esi, 00000004
 :00402888 7CDB
                jl 00402865  <----是否完成第一组
 :0040288A 8B7C2430
mov edi, dword ptr [esp+30]
 :0040288E 8D4C2408
        lea ecx, dword ptr [esp+08]
 :00402892 8BF7
                  mov esi, edi
:00402894 33D2
xor edx, edx
 :00402896 2BF1
      sub esi, ecx
 \* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
 |:004028BC(C)
 |
 :00402898 8D4C1408
              lea ecx, dword ptr [esp+edx+08]
 :0040289C 0FBE040E
movsx eax, byte ptr [esi+ecx]
 :004028A0 83F841
          cmp eax, 00000041
 :004028A3 7C08
                  jl 004028AD
:004028A5 83F85A
cmp eax, 0000005A
 :004028A8 7F03
        jg 004028AD
 :004028AA 83C020
            add eax, 00000020
 \* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
 |:004028A3(C), :004028A8(C)
 |
 :004028AD 0FBE09
  movsx ecx, byte ptr [ecx]
 :004028B0 3BC1
            cmp eax, ecx <----比较第一组注册码
 :004028B2
0F8520010000            jne 004029D8
 :004028B8
42
inc edx
 :004028B9 83FA04
    cmp edx, 00000004
 :004028BC 7CDA
            jl 00402898
 :004028BE 8B442408
              mov eax, dword ptr [esp+08]
:004028C2 8D1480
lea edx, dword ptr [eax+4\*eax]
 :004028C5 8D0450
          lea eax, dword ptr [eax+2\*edx]
 :004028C8
03C0                    add
eax, eax
 :004028CA 33F6
    xor esi, esi
 :004028CC 8944240C
        mov dword ptr [esp+0C], eax <----将reg1的值进行第二次变化.放入第二个地址中，以下记为reg2,
reg2 = reg1 \*22;
 \* Referenced by a (U)nconditional or (C)onditional
Jump at Address:
 |:004028F1(C)
 |
 :004028D0 8A4C340C
            mov cl, byte ptr [esp+esi+0C]
 :004028D4
51
push ecx
 :004028D5 56
      push esi
 :004028D6 E815010000
      call 004029F0
 :004028DB 25FF000000
        and eax, 000000FF
 :004028E0 50
                push eax
 :004028E1
E8FA010000              call 00402AE0 <----转化成第二组注册码
 :004028E6 83C40C
add esp, 0000000C
 :004028E9 8844340C
    mov byte ptr [esp+esi+0C], al
 :004028ED 46
                inc esi
 :004028EE
83FE04                  cmp esi,
00000004
 :004028F1 7CDD
    jl 004028D0
 :004028F3 33C9
          xor ecx, ecx
 \* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
 |:00402918(C)
 |
 :004028F5 0FBE440F05
            movsx eax, byte ptr [edi+ecx+05]
:004028FA 83F841
cmp eax, 00000041
 :004028FD 7C08
        jl 00402907
 :004028FF 83F85A
            cmp eax, 0000005A
 :00402902 7F03
                  jg 00402907
:00402904 83C020
add eax, 00000020
 \* Referenced by a (U)nconditional or (C)onditional
Jump at Addresses:
 |:004028FD(C), :00402902(C)
 |
 :00402907 0FBE540C0C
            movsx edx, byte ptr [esp+ecx+0C]
:0040290C 3BC2
cmp eax, edx <----比较第二组注册码
 :0040290E 0F85C4000000
      jne 004029D8
 :00402914 41
              inc ecx
 :00402915 83F904
                cmp ecx, 00000004
:00402918 7CDB
jl 004028F5
 :0040291A 8B44240C
  mov eax, dword ptr [esp+0C]
 :0040291E 8B4C2408
          mov ecx, dword ptr [esp+08]
 :00402922
8BD0                    mov
edx, eax
 :00402924 33D1
    xor edx, ecx
 :00402926 8D0440
        lea eax, dword ptr [eax+2\*eax]
 :00402929 83C208
                add edx, 00000008
:0040292C 0FAFD1
imul edx, ecx
 :0040292F 03D0
      add edx, eax
 :00402931 33F6
            xor esi, esi
 :00402933 89542410
              mov dword ptr [esp+10], edx <----将用户名转化的值进行第三次变化记为
reg3 = reg2\*3 + ((reg2 ^ reg1)+8)\*reg1;
 \* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
 |:0040294D(C)
 |
 :00402937 0FBE4C3410
            movsx ecx, byte ptr [esp+esi+10]
:0040293C 51
  push ecx
 :0040293D E89E010000
  call 00402AE0 <----转化成第三组注册码
 :00402942 83C404
            add esp, 00000004
 :00402945 88443410
              mov byte ptr [esp+esi+10], al
 :00402949 46
    inc esi
 :0040294A 83FE04
        cmp esi, 00000004
 :0040294D 7CE8
                jl 00402937
 :0040294F
33C9                    xor
ecx, ecx
 \* Referenced by a (U)nconditional or (C)onditional Jump at
Address:
 |:00402970(C)
 |
 :00402951 0FBE440F0A
        movsx eax, byte ptr [edi+ecx+0A]
 :00402956 83F841
                cmp eax, 00000041
:00402959 7C08
jl 00402963
 :0040295B 83F85A
    cmp eax, 0000005A
 :0040295E 7F03
            jg 00402963
 :00402960 83C020
                add eax, 00000020
 \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402959(C), :0040295E(C)
 |
 :00402963 0FBE540C10
        movsx edx, byte ptr [esp+ecx+10]
 :00402968 3BC2
                  cmp eax, edx <----比较第三组注册码
 :0040296A 756C
  jne 004029D8
 :0040296C 41
          inc ecx
 :0040296D 83F904
              cmp ecx, 00000004
 :00402970
7CDF                    jl 00402951
 :00402972 8B4C240C
mov ecx, dword ptr [esp+0C]
 :00402976 8B442408
        mov eax, dword ptr [esp+08]
 :0040297A 0FAFC8
                imul ecx, eax
 :0040297D
83C106                  add ecx,
00000006
 :00402980 8D1480
    lea edx, dword ptr [eax+4\*eax]
 :00402983 0FAF4C2410
          imul ecx, dword ptr [esp+10]
 :00402988
03CA                    add
ecx, edx
 :0040298A 33F6
    xor esi, esi
 :0040298C 894C2414
        mov dword ptr [esp+14], ecx <----将用户名转化的值进行第四次变化.记为
reg4 = reg1\*5 + ((reg2 \* reg1)+6)\*reg3;
 \* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
 |:004029A6(C)
 |
 :00402990 0FBE443414
            movsx eax, byte ptr [esp+esi+14]
:00402995 50
  push eax
 :00402996 E845010000
  call 00402AE0 <----转化出第4组注册码
 :0040299B 83C404
            add esp, 00000004
 :0040299E 88443414
              mov byte ptr [esp+esi+14], al
 :004029A2 46
    inc esi
 :004029A3 83FE04
        cmp esi, 00000004
 :004029A6 7CE8
                jl 00402990
 :004029A8
33C9                    xor
ecx, ecx
 \* Referenced by a (U)nconditional or (C)onditional Jump at
Address:
 |:004029C9(C)
 |
 :004029AA 0FBE440F0F
        movsx eax, byte ptr [edi+ecx+0F]
 :004029AF 83F841
                cmp eax, 00000041
:004029B2 7C08
jl 004029BC
 :004029B4 83F85A
    cmp eax, 0000005A
 :004029B7 7F03
            jg 004029BC
 :004029B9 83C020
                add eax, 00000020
```
