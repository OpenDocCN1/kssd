
```
:004034CD 43
   inc ebx
:004034CE E82D010000
   call 00403600
                   ====>对64
8A 4E 4E进行运算得出一组新值B2 45 13 09
:004034D3
83C408                  add esp,
00000008
:004034D6 33C9
   xor ecx, ecx
:004034D8 8AC8
         mov cl, al
:004034DA 51
               push ecx
:004034DB E810020000
             call 004036F0
     ====>对B2 45 13 09进行运算得出一组新值32 65 33 37
:004034E0
83C404                  add esp,
00000004
:004034E3 8807
   mov byte ptr [edi], al
                     ====>[edi]=2e37
      保存结果
:004034E5
83FB04                  cmp ebx,
00000004
:004034E8 7CDB
   jl 004034C5
                   ====>循环4次
:004034EA
33C9                    xor
ecx, ecx
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0040350F(C)
|
:004034EC
0FBE440E05              movsx eax, byte ptr
[esi+ecx+05]
:004034F1 83F841
   cmp eax, 00000041
:004034F4 7C08
         jl 004034FE
:004034F6 83F85A
             cmp eax, 0000005A
:004034F9
7F03                    jg 004034FE
:004034FB
83C020                  add eax,
00000020
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:004034F4(C), :004034F9(C)
|
:004034FE
0FBE540C10              movsx edx, byte ptr
[esp+ecx+10]
:00403503 3BD0
     cmp edx, eax
                     ====>逐位比较比较6-9位！
         ====>[esp+10]=2e37
   ====>[esi+05]=5678
          ★★★★
    二、        ====>所以注册码的6-9位是：2e37
:00403505
0F85C8000000            jne 004035D3
       ====>跳则OVER！
:0040350B
41                      inc
ecx
:0040350C 83F904
 cmp ecx, 00000004
:0040350F 7CDB
       jl 004034EC
:00403511 8B442410
         mov eax, dword ptr [esp+10]
     ====>EAX=37336532
:00403515
33DB                    xor
ebx, ebx
:00403517 3344240C
 xor eax, dword ptr [esp+0C]
                     ====>EAX=37336532
XOR 31313633=06025301
:0040351B
83C002                  add eax,
00000002
             ====>EAX=06025301 + 00000002=06025303
:0040351E
0FAF44240C              imul eax, dword ptr
[esp+0C]
             ====>EAX=06025303 \* 31313633=268C2B99
:00403523
03442410                add eax, dword
ptr [esp+10]
               ====>EAX=268C2B99 +
37336532=5DBF90CB   得出运算11-14位注册码的参数！
:00403527
89442414                mov dword ptr
[esp+14], eax
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00403541(C)
|
:0040352B
8D7C1C14                lea edi, dword
ptr [esp+ebx+14]
:0040352F 43
       inc ebx
:00403530 0FBE07
         movsx eax, byte ptr [edi]
:00403533 50
                   push eax
:00403534
E8B7010000              call 004036F0
       ====>对CB 90 BF 5D进行运算得出一组新值6B 31 31 73
:00403539
83C404                  add esp,
00000004
:0040353C 8807
   mov byte ptr [edi], al
                     ====>[edi]=k11s
      保存结果
:0040353E
83FB04                  cmp ebx,
00000004
:00403541 7CE8
   jl 0040352B
                   ====>循环4次
:00403543
33C9                    xor
ecx, ecx
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00403564(C)
|
:00403545
0FBE440E0A              movsx eax, byte ptr
[esi+ecx+0A]
:0040354A 83F841
   cmp eax, 00000041
:0040354D 7C08
         jl 00403557
:0040354F 83F85A
             cmp eax, 0000005A
:00403552
7F03                    jg 00403557
:00403554
83C020                  add eax,
00000020
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040354D(C), :00403552(C)
|
:00403557
0FBE540C14              movsx edx, byte ptr
[esp+ecx+14]
:0040355C 3BD0
     cmp edx, eax
                     ====>逐位比较比较后4位！
         ====>[esp+14]=k11s
   ====>[esi+0A]=90AB
          ★★★★
    三、        ====>所以注册码的11-14位是：k11s
:0040355E
757E                    jne
004035DE
             ====>跳则OVER！
:00403560
41                      inc
ecx
:00403561 83F904
 cmp ecx, 00000004
:00403564 7CDF
       jl 00403545
:00403566 8B442410
         mov eax, dword ptr [esp+10]
     ====>EAX=37336532
:0040356A
33FF                    xor
edi, edi
:0040356C 0FAF442414              imul
eax, dword ptr [esp+14]
                 ====>EAX=37336532
\* 7331316B=BD6BDDE6
:00403571
40                      inc
eax
:00403572 0FAF44240C              imul
eax, dword ptr [esp+0C]
                 ====>EAX=BD6BDDE6
\* 31313633=2D82EF05
:00403577
03442414                add eax, dword
ptr [esp+14]
               ====>EAX=2D82EF05 +
7331316B=A0B42070   得出运算最后位注册码的参数！
:0040357B
89442418                mov dword ptr
[esp+18], eax
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00403595(C)
|
:0040357F
8D5C3C18                lea ebx, dword
ptr [esp+edi+18]
:00403583 47
       inc edi
:00403584 0FBE03
         movsx eax, byte ptr [ebx]
:00403587 50
                   push eax
:00403588
E863010000              call 004036F0
       ====>对70 20 B4 A0进行运算得出一组新值70 31 34 31
:0040358D
83C404                  add esp,
00000004
:00403590 8803
   mov byte ptr [ebx], al
                     ====>[ebx]=p141
      保存结果
:00403592
83FF04                  cmp edi,
00000004
:00403595 7CE8
   jl 0040357F
                   ====>循环4次
:00403597
33C9                    xor
ecx, ecx
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:004035B8(C)
|
:00403599
0FBE440E0F              movsx eax, byte ptr
[esi+ecx+0F]
:0040359E 83F841
   cmp eax, 00000041
:004035A1 7C08
         jl 004035AB
:004035A3 83F85A
             cmp eax, 0000005A
:004035A6
7F03                    jg 004035AB
:004035A8
83C020                  add eax,
00000020
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:004035A1(C), :004035A6(C)
|
:004035AB
0FBE540C18              movsx edx, byte ptr
[esp+ecx+18]
:004035B0 3BD0
     cmp edx, eax
                     ====>逐位比较比较后4位！
         ====>[esp+18]=p141
   ====>[esi+0F]=CDEF
          ★★★★
    四、        ====>所以注册码的最后4位是：p141
:004035B2
7535                    jne
004035E9
             ====>跳则OVER！
:004035B4
41                      inc
ecx
:004035B5 83F904
 cmp ecx, 00000004
:004035B8 7CDF
       jl 00403599
\*
Possible Reference to String Resource ID=00001: "Register Success"
         |
:004035BA B801000000
       mov eax, 00000001
                       ====>置1则OK！
:004035BF
5F                      pop
edi
:004035C0 5E
   pop esi
:004035C1 5B
         pop ebx
:004035C2 83C420
           add esp, 00000020
:004035C5 C20800
                 ret 0008
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004034A9(C)
|
:004035C8
33C0                    xor
eax, eax
             ====>清0则OVER！
:004035CA
5F                      pop
edi
:004035CB 5E
   pop esi
:004035CC 5B
         pop ebx
:004035CD 83C420
           add esp, 00000020
:004035D0 C20800
                 ret 0008
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403505(C)
|
:004035D3
33C0                    xor
eax, eax
             ====>清0则OVER！
:004035D5
5F                      pop
edi
:004035D6 5E
   pop esi
:004035D7 5B
         pop ebx
:004035D8 83C420
           add esp, 00000020
:004035DB C20800
                 ret 0008
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040355E(C)
|
:004035DE
33C0                    xor
eax, eax
             ====>清0则OVER！
:004035E0
5F                      pop
edi
:004035E1 5E
   pop esi
:004035E2 5B
         pop ebx
:004035E3 83C420
           add esp, 00000020
:004035E6 C20800
                 ret 0008
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004035B2(C)
|
:004035E9
33C0                    xor
eax, eax
             ====>清0则OVER！
:004035EB
5F                      pop
edi
:004035EC 5E
   pop esi
:004035ED 5B
         pop ebx
:004035EE 83C420
           add esp, 00000020
:004035F1 C20800
```
