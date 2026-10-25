
## 拆解代码片段

```asm
:00472528 50
   push eax
```
*Reference To: MSVBVM60.rtcReplace, Ord:02C8h*

```
|
:00472529 FF152C124000            Call dword ptr [0040122C]
```
===>去除试炼码中的-

```
:0047252F 8BD0
 mov edx, eax
```
===>EDX=fly12345678fly[OCN][FCG]E

```
:00472531 8D4DD4
             lea ecx, dword ptr [ebp-2C]
```
*Reference To: MSVBVM60.__vbaStrMove, Ord:0000h*

```
|
:00472534 FF1578134000            Call dword ptr [00401378]
:0047253A 8B0B
 mov ecx, dword ptr [ebx]
```
*Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h*

```
|
:0047253C 8B1D34104000            mov ebx, dword ptr [00401034]
:00472542 51
   push ecx
```
===>ECX=fly-12345678-fly[OCN][FCG]-E

```
:00472543 FFD3
             call ebx
```
===>取fly-12345678-fly[OCN][FCG]-E的长度

```
:00472545 8BD0
             mov edx, eax
```
===>EDX=1C

```
:00472547 8B45D4
             mov eax, dword ptr [ebp-2C]
:0047254A 50
   push eax
```
===>EAX=fly12345678fly[OCN][FCG]E

```
:0047254B 899528FFFFFF
       mov dword ptr [ebp+FFFFFF28], edx
```
===>[ebp+FFFFFF28]=EDX=1C

```
:00472551 FFD3
             call ebx
```
===>取fly12345678fly[OCN][FCG]E的长度=19

```
:00472553 8B8D28FFFFFF
       mov ecx, dword ptr [ebp+FFFFFF28]
```
===>ECX=1C

```
:00472559 8B55D4
             mov edx, dword ptr [ebp-2C]
:0047255C 33DB
 xor ebx, ebx
:0047255E 3BC1
 cmp eax, ecx
```
===>比较2者长度是否相同？既检测试炼码中是否有-

```
:00472560 52
               push edx
:00472561 0F9DC3                  setnl bl
```
===>设置BL值！有-则长度不同则BL=0

*Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h*

```
|
:00472564 FF1534104000            Call dword ptr [00401034]
```
===>取fly12345678fly[OCN][FCG]E的长度=19

```
:0047256A 33C9
             xor ecx, ecx
:0047256C 83F819                  cmp eax, 00000019
```
===>去除试炼码中的-后是否是25位？

```
:0047256F 0F9CC1
             setl cl
```
===>设置CL值！是25位则CL=0

```
:00472572 0BD9
             or ebx, ecx
:00472574 0F850C010000            jne 00472686
```
===>如果上面2个条件都符合则此处不跳！
===>若此处跳就直接OVER了！爆破点①！

```
:0047257A 8B55D4
             mov edx, dword ptr [ebp-2C]
```
===>EDX=fly12345678fly[OCN][FCG]E

```
:0047257D A110804A00
         mov eax, dword ptr [004A8010]
```
===>EAX=211C1E09
C盘的硬盘序列号

```
:00472582 8D4DA4
             lea ecx, dword ptr [ebp-5C]
:00472585 89955CFFFFFF            mov dword ptr [ebp+FFFFFF5C], edx
:0047258B 2DCF337B00              sub eax, 007B33CF
```
===>EAX=211C1E09 - 007B33CF=20A0EA3A

```
:00472590 51
               push ecx
:00472591 8D5594                  lea edx, dword ptr [ebp-6C]
:00472594 0F8020050000            jo 00472ABA
:0047259A 52
   push edx
:0047259B C78554FFFFFF08000000    mov dword ptr [ebp+FFFFFF54], 00000008
:004725A5 8945AC                  mov dword ptr [ebp-54], eax
:004725A8 C745A403000000          mov [ebp-5C], 00000003
```
*Reference To: MSVBVM60.rtcHexVarFromVar, Ord:023Dh*

```
|
:004725AF FF15D8124000            Call dword ptr [004012D8]
:004725B5 6A01
 push 00000001
:004725B7 8D8554FFFFFF            lea eax, dword ptr [ebp+FFFFFF54]
:004725BD 50
   push eax
:004725BE 8D4D94                  lea ecx, dword ptr [ebp-6C]
:004725C1 51
   push ecx
:004725C2 6A01
 push 00000001
:004725C4 8D5584                  lea edx, dword ptr [ebp-7C]
:004725C7 52
   push edx
:004725C8 89BD4CFFFFFF            mov dword ptr [ebp+FFFFFF4C], edi
:004725CE C78544FFFFFF02800000    mov dword ptr [ebp+FFFFFF44], 00008002
```
*Reference To: MSVBVM60.__vbaInStrVar, Ord:0000h*

```
|
:004725D8 FF1570124000            Call dword ptr [00401270]
```
===>比较CALL！进入！有点特别呀 ^O^ ^O^

```
:004725DE 50
               push eax
:004725DF 8D8544FFFFFF            lea eax, dword ptr [ebp+FFFFFF44]
:004725E5 50
   push eax
```
*Reference To: MSVBVM60.__vbaVarTstGt, Ord:0000h*

```
|
:004725E6 FF1504104000            Call dword ptr [00401004]
:004725EC 8D4D84                  lea ecx, dword ptr [ebp-7C]
:004725EF 51
   push ecx
:004725F0 8D5594                  lea edx, dword ptr [ebp-6C]
:004725F3 668BD8                  mov bx, ax
```
===>爆破点②！ ^O^ ^O^

```
:004725F6 52
               push edx
:004725F7 8D45A4                  lea eax, dword ptr [ebp-5C]
:004725FA 50
   push eax
:004725FB 6A03
 push 00000003
```
*Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h*

```
|
:004725FD FF1544104000            Call dword ptr [00401044]
:00472603 83C410                  add esp, 00000010
:00472606 663BDF                  cmp bx, di
:00472609 0F84E3000000            je 004726F2
```
===>跳则OVER！

```
:0047260F 8B0E
             mov ecx, dword ptr [esi]
:00472611 56
   push esi
:00472612 C745D0FFFFFFFF          mov [ebp-30], FFFFFFFF
:00472619 FF912C060000            call dword ptr [ecx+0000062C]
:0047261F 50
   push eax
:00472620 8D55B8                  lea edx, dword ptr [ebp-48]
:00472623 52
   push edx
```
*Reference To: MSVBVM60.__vbaObjSet, Ord:0000h*

```
|
:00472624 FF15F4104000            Call dword ptr [004010F4]
:0047262A 8D4DB4                  lea ecx, dword ptr [ebp-4C]
:0047262D 51
   push ecx
:0047262E 8BF0
 mov esi, eax
:00472630 8B06
 mov eax, dword ptr [esi]
:00472632 6A03
 push 00000003
:00472634 56
   push esi
:00472635 FF5040                  call [eax+40]
:00472638 DBE2
 fclex
:0047263A 3BC7
 cmp eax, edi
:0047263C 7D0F
 jge 0047264D
:0047263E 6A40
 push 00000040
:00472640 68BC654200              push 004265BC
:00472645 56
   push esi
:00472646 50
   push eax
―――――――――――――――――――――――――――――――――
```

### 进入比较CALL：`004725D8` Call dword ptr [00401270]

再进入：`7347A9CC` Call `MSVBVM60.__vbaInStr`

```asm
733A45A5 >  55
   push ebp
733A45A6    8BEC
   mov ebp,esp
733A45A8    81EC BC000000         sub esp,0BC
733A45AE    8365 EC 00            and dword ptr ss:[ebp-14],0
733A45B2    53
     push ebx
733A45B3    56
     push esi
733A45B4    8B75 0C
mov esi,dword ptr ss:[ebp+C]
```
===>ESI=20A0EA3A

```asm
733A45B7    57
                 push edi
733A45B8    8B7D 10
mov edi,dword ptr ss:[ebp+10]
```
===>EDI=fly12345678fly[OCN][FCG]E

```asm
733A45BB    8D85 44FFFFFF
        lea eax,dword ptr ss:[ebp-BC]
733A45C1    897D F8
mov dword ptr ss:[ebp-8],edi
733A45C4    85FF
   test edi,edi
733A45C6    8945 F4
mov dword ptr ss:[ebp-C],eax
733A45C9    8975 FC
mov dword ptr ss:[ebp-4],esi
733A45CC    0F84 09350300         je MSVBVM60.733D7ADB
733A45D2    8B47 FC
mov eax,dword ptr ds:[edi-4]
733A45D5    D1E8
   shr eax,1
```
===>取fly12345678fly[OCN][FCG]E长度

```asm
733A45D7    8945 E4
            mov dword ptr ss:[ebp-1C],eax
```
===>EAX=19

```asm
733A45DA    0F84 FB340300
        je MSVBVM60.733D7ADB
733A45E0    85F6
   test esi,esi
733A45E2    0F84 EB340300         je MSVBVM60.733D7AD3
733A45E8    8B46 FC
mov eax,dword ptr ds:[esi-4]
733A45EB    D1E8
   shr eax,1
```
===>取20A0EA3A的长度

```asm
733A45ED    8945 E4
            mov dword ptr ss:[ebp-1C],eax
```
===>EAX=8

```asm
733A45F0    0F84 DD340300
        je MSVBVM60.733D7AD3
733A45F6    8B45 14
mov eax,dword ptr ss:[ebp+14]
733A45F9    8D58 FF
lea ebx,dword ptr ds:[eax-1]
733A45FC    85DB
   test ebx,ebx
733A45FE    0F8C 33330300         jl MSVBVM60.733D7937
733A4604    81FB FFFFFF3F         cmp ebx,3FFFFFFF
733A460A    0F87 27330300         ja MSVBVM60.733D7937
733A4610    8B45 08
mov eax,dword ptr ss:[ebp+8]
733A4613    895D E8
mov dword ptr ss:[ebp-18],ebx
733A4616    85C0
   test eax,eax
733A4618    0F85 20330300         jnz MSVBVM60.733D793E
```
===>跳下去，转变大写字母为小写字母！

```asm
733A461E    8B45 F8
            mov eax,dword ptr ss:[ebp-8]
```
===>转变完了再跳回来！

```asm
733A4621    85C0
               test eax,eax
```
===>EAX=fly12345678fly[ocn][fcg]e

```asm
733A4623    0F84 06340300
        je MSVBVM60.733D7A2F
733A4629    8B48 FC
mov ecx,dword ptr ds:[eax-4]
733A462C    D1E9
```
