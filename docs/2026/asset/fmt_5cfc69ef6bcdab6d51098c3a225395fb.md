
```assembly
by a (U)nconditional or (C)onditional Jump at Address:
|:00403AFA(C)
|
:00403B0E
750C                    jne
00403B1C
\* Possible
Reference to String Resource ID=00001: "AKoff Music Composer - Version 2.0"
         |
:00403B10 C705C8EC410001000000    mov
dword ptr [0041ECC8], 00000001
                   ====>注册标志位
 置1 ！    OK！
:00403B1A
EB07                    jmp
00403B23
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403AF6(C), :00403B02(C),
:00403B0E(C)
|
:00403B1C 33C0
       xor eax, eax
                       ====>EAX
清零
:00403B1E A3C8EC4100
             mov dword ptr [0041ECC8], eax
         ====>注册标志位  置0 ！    OVER！
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403AAF(U),
:00403AE9(U), :00403B1A(U)
|
:00403B23 83C414
         add esp, 00000014
:00403B26 5F
                 pop edi
:00403B27
5E                      pop
esi
:00403B28 C3
   ret
―――――――――――――――――――――――――――――――――
F8进入算法CALL：403AC0
call 004038B0
\*
Referenced by a CALL at Addresses:
|:00403AC0   , :0040427D
|
:004038B0
55                      push
ebp
:004038B1 8BEC
   mov ebp, esp
:004038B3 83C4E8
       add esp, FFFFFFE8
:004038B6 53
               push ebx
:004038B7 56
                     push
esi
:004038B8 57
   push edi
:004038B9 33FF
       xor edi, edi
:004038BB 68BCEA4100
         push 0041EABC
 ====>fly@263.net
:004038C0
E8F7210100              call 00415ABC
       ====>取E-mail的长度，检测其格式
:004038C5
59                      pop
ecx
:004038C6 83F805
 cmp eax, 00000005
                 ====>EAX=11
:004038C9
730B                    jnb
004038D6
             ====>不能少于5位！
:004038CB
8B4508                  mov eax,
dword ptr [ebp+08]
:004038CE C60000
     mov byte ptr [eax], 00
:004038D1 E99E000000
         jmp 00403974
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004038C9(C)
|
:004038D6
33DB                    xor
ebx, ebx
\* Possible
Reference to Dialog:
                   |
:004038D8
BEBCEA4100              mov esi, 0041EABC
         ====>ESI=fly@263.net
:004038DD
EB07                    jmp
004038E6
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:004038F3(C)
|
:004038DF
0FBE06                  movsx eax,
byte ptr [esi]
               ====>依次取fly@263.net字符的HEX值
      1、====>EAX=66
                  2、====>EAX=6C
      3、====>EAX=79
            …… ……  省  略  …… ……
     11、====>EAX=74
:004038E2
03F8                    add
edi, eax
             ====>累加！
  1、====>EDI=00+66=66
                2、====>EDI=66+6C=D2
      3、====>EDI=D2+79=14B
              …… ……  省  略  ……
……
       11、====>EDI=327+74=39B
:004038E4
43                      inc
ebx
             ====>依次增1
:004038E5
46                      inc
esi
\* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
|:004038DD(U)
|
:004038E6
68BCEA4100              push 0041EABC
:004038EB
E8CC210100              call 00415ABC
:004038F0
59                      pop
ecx
:004038F1 3BD8
   cmp ebx, eax
                   ====>共循环E-mail的位数11次
:004038F3
7CEA                    jl 004038DF
         ====>取完了吗？继续循环！
:004038F5
B97B000000              mov ecx, 0000007B
         ====>7B 入 ECX
:004038FA
8BC7                    mov
eax, edi
             ====>EAX=EDI=39B
:004038FC
99                      cdq
:004038FD
F7F9                    idiv
ecx
             ====>EAX=39B % 7B=余数：3E
:004038FF
89D7                    mov
edi, edx
             ====>3E 入 EDI
:00403901
33DB                    xor
ebx, ebx
:00403903 8D45E8
   lea eax, dword ptr [ebp-18]
\*
Possible Reference to Dialog:
                     |
:00403906
BABCEA4100              mov edx, 0041EABC
         ====>EDX=fly@263.net
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403926(C)
|
:0040390B
0FBE0A                  movsx ecx,
byte ptr [edx]
               ====>依次取fly@263.net前5位字符的HEX值
      1、====>ECX=66
                  2、====>ECX=6C
      3、====>ECX=79
                  4、====>ECX=40
      5、====>ECX=32
:0040390E
03CF                    add
ecx, edi
          1、====>EDI=66+3E=A4
2、====>EDI=6C+3E=AA
              3、====>EDI=79+3E=B7
    4、====>EDI=40+3E=7E
                  5、====>EDI=32+3E=70
:00403910
8908                    mov
dword ptr [eax], ecx
                 ====>结果依次存入 [EAX]
处
:00403912 81F9FF000000
           cmp ecx, 000000FF
   ====>结果大于FF？
:00403918
7E04                    jle
0040391E
             ====>结果不大于FF就跳。
:0040391A
33C9                    xor
ecx, ecx
             ====>否则 ECX 清零
:0040391C
8908                    mov
dword ptr [eax], ecx
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403918(C)
|
:0040391E
43                      inc
ebx
:0040391F 83C004
 add eax, 00000004
:00403922 42
         inc edx
:00403923 83FB05
           cmp ebx, 00000005
   ====>取前5位
:00403926
7CE3                    jl 0040390B
         ====>继续循环
:00403928
897DFC                  mov dword
ptr [ebp-04], edi
:0040392B 33DB
       xor ebx, ebx
:0040392D 8B4508
           mov eax, dword ptr [ebp+08]
:00403930
8BD0                    mov
edx, eax
:00403932 8D45E8
   lea eax, dword ptr [ebp-18]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040396B(C)
|
:00403935
8B08                    mov
ecx, dword ptr [eax]
              1、====>ECX=A4
  2、====>ECX=AA
              3、====>ECX=B7
  4、====>ECX=7E
              5、====>ECX=70
  6、====>ECX=3E
:00403937
85C9                    test
ecx, ecx
:00403939 7903
   jns 0040393E
:0040393B 83C10F
       add ecx, 0000000F
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403939(C)
|
:0040393E
C1F904                  sar ecx,
04
   （同0040394B处分别取前、后位） ====>算术右移4次！
                      1、====>ECX=A
      2、====>ECX=A
                  3、====>ECX=B
      4、====>ECX=7
                  5、====>ECX=7
      6、====>ECX=3
:00403941
8A8964F14100            mov cl, byte ptr [ecx+0041F164]
         ====>从 [ecx+0041F164] 处查表！
  1、====>CL=1
              2、====>CL=1
  3、====>CL=P
              4、====>CL=G
  5、====>CL=G
              6、====>CL=A
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
[0041F164]处的表值！
0041F164
 4B 33 35 41 37 43 32 47 30 34 31 50 39 36 44 38  K35A7C2G041P96D8
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
:00403947
880A                    mov
byte ptr [edx], cl
                 ====>结果保存 [EDX]
处
:00403949 8B08
                 mov ecx, dword ptr
[eax]
        1-1、====>ECX=A4
                    2-2、====>ECX=AA
    3-3、====>ECX=B7
                4-4、====>ECX=7E
    5-5、====>ECX=70
                6-6、====>ECX=3E
:0040394B
81E10F000080            and ecx, 8000000F
       ====>分别和8000000F进行“与”运算！
                      1-1、====>ECX=4
    2-2、====>ECX=A
                3-3、====>ECX=7
    4-4、====>ECX=E
                5-5、====>ECX=0
    6-6、====>ECX=E
:00403951
7905                    jns
00403958
:00403953 49
     dec ecx
:00403954 83C9F0
       or ecx, FFFFFFF0
:00403957 41
               inc ecx
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403951(C)
|
:00403958
8A8964F14100            mov cl, byte ptr [ecx+0041F164]
         ====>从 [ecx+0041F164] 处查表！
1-1、====>CL=7
          2-2、====>CL=1
                    3-3、====>CL=G
    4-4、====>CL=D
              5-5、====>CL=K
6-6、====>CL=D
:0040395E
884A01                  mov byte
ptr [edx+01], cl
               ====>结果保存 [EDX+01] 处
:00403961
43                      inc
ebx
:00403962 83C202
 add edx, 00000002
:00403965 83C004
       add eax, 00000004
:00403968 83FB06
             cmp ebx, 00000006
     ====>循环6次
:0040396B 7CC8
           jl 00403935
   ====>继续循环？
:0040396D
```
