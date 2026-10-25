
```
eax, bl    //依次取输入假码的hex值送eax
:00401B5D 83F830                  cmp eax, 00000030
:00401B60 7C05                    jl 00401B67
:00401B62 83F839                  cmp eax, 00000039
:00401B65 7E32                    jle 00401B99    //上面两个比较是看输入的注册码是否是数字，是则继续，有一个不是则over

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B60(C)
|
:00401B67 33C0                    xor eax, eax
:00401B69 BA02000000              mov edx, 00000002
:00401B6E 50                      push eax
:00401B6F 8D4508                  lea eax, dword ptr [ebp+08]
:00401B72 FF4F1C                  dec [edi+1C]
:00401B75 E8A6D51300              call 0053F120
:00401B7A FF4F1C                  dec [edi+1C]
:00401B7D 8D450C                  lea eax, dword ptr [ebp+0C]
:00401B80 BA02000000              mov edx, 00000002
:00401B85 E896D51300              call 0053F120
:00401B8A 58                      pop eax
:00401B8B 8B17                    mov edx, dword ptr [edi]
:00401B8D 64891500000000          mov dword ptr fs:[00000000], edx
:00401B94 E920020000              jmp 00401DB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B65(C)
|
:00401B99 66C747102000            mov [edi+10], 0020
:00401B9F 33C9                    xor ecx, ecx
:00401BA1 8D45F8                  lea eax, dword ptr [ebp-08]
:00401BA4 894DF8                  mov dword ptr [ebp-08], ecx
:00401BA7 50                      push eax
:00401BA8 FF471C                  inc [edi+1C]
:00401BAB 8D4508                  lea eax, dword ptr [ebp+08]
:00401BAE B901000000              mov ecx, 00000001
:00401BB3 8BD6                    mov edx, esi
:00401BB5 E8DED61300              call 0053F298
:00401BBA 8D45F8                  lea eax, dword ptr [ebp-08]
:00401BBD E8BAD71300              call 0053F37C
:00401BC2 8B55C8                  mov edx, dword ptr [ebp-38]
:00401BC5 8902                    mov dword ptr [edx], eax
:00401BC7 FF4F1C                  dec [edi+1C]
:00401BCA 8D45F8                  lea eax, dword ptr [ebp-08]
:00401BCD BA02000000              mov edx, 00000002
:00401BD2 E849D51300              call 0053F120
:00401BD7 46                      inc esi
:00401BD8 8345C804                add dword ptr [ebp-38], 00000004
:00401BDC 83FE0B                  cmp esi, 0000000B
:00401BDF 0F8E32FFFFFF            jle 00401B17
:00401BE5 BE01000000              mov esi, 00000001
:00401BEA 8D85F0FEFFFF            lea eax, dword ptr [ebp+FFFFFEF0]
:00401BF0 8945C4                  mov dword ptr [ebp-3C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401CBB(C)
|
:00401BF3 66C747102C00            mov [edi+10], 002C
:00401BF9 33D2                    xor edx, edx
:00401BFB 8D4DF4                  lea ecx, dword ptr [ebp-0C]
:00401BFE 8955F4                  mov dword ptr [ebp-0C], edx
:00401C01 51                      push ecx
:00401C02 FF471C                  inc [edi+1C]
:00401C05 B901000000              mov ecx, 00000001
:00401C0A 8BD6                    mov edx, esi
:00401C0C 8D450C                  lea eax, dword ptr [ebp+0C]
:00401C0F E884D61300              call 0053F298
:00401C14 837DF400                cmp dword ptr [ebp-0C], 00000000
:00401C18 7405                    je 00401C1F
:00401C1A 8B45F4                  mov eax, dword ptr [ebp-0C]
:00401C1D EB05                    jmp 00401C24

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C18(C)
|
:00401C1F B879815600              mov eax, 00568179

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C1D(U)
|
:00401C24 8A18                    mov bl, byte ptr [eax]
:00401C26 FF4F1C                  dec [edi+1C]
:00401C29 8D45F4                  lea eax, dword ptr [ebp-0C]
:00401C2C BA02000000              mov edx, 00000002
:00401C31 E8EAD41300              call 0053F120
:00401C36 0FBEC3                  movsx eax, bl
:00401C39 83F830                  cmp eax, 00000030
:00401C3C 7C05                    jl 00401C43
:00401C3E 83F839                  cmp eax, 00000039
:00401C41 7E32                    jle 00401C75

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C3C(C)
|
:00401C43 33C0                    xor eax, eax
:00401C45 BA02000000              mov edx, 00000002
:00401C4A 50                      push eax
:00401C4B 8D4508                  lea eax, dword ptr [ebp+08]
:00401C4E FF4F1C                  dec [edi+1C]
:00401C51 E8CAD41300              call 0053F120
:00401C56 FF4F1C                  dec [edi+1C]
:00401C59 8D450C                  lea eax, dword ptr [ebp+0C]
:00401C5C BA02000000              mov edx, 00000002
:00401C61 E8BAD41300              call 0053F120
:00401C66 58                      pop eax
:00401C67 8B17                    mov edx, dword ptr [edi]
:00401C69 64891500000000          mov dword ptr fs:[00000000], edx
:00401C70 E944010000              jmp 00401DB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C41(C)
|
:00401C75 66C747103800            mov [edi+10], 0038
:00401C7B 33C9                    xor ecx, ecx
:00401C7D 8D45F0                  lea eax, dword ptr [ebp-10]
:00401C80 894DF0                  mov dword ptr [ebp-10], ecx
:00401C83 50                      push eax
:00401C84 FF471C                  inc [edi+1C]
:00401C87 8D450C                  lea eax, dword ptr [ebp+0C]
:00401C8A B901000000              mov ecx, 00000001
:00401C8F 8BD6                    mov edx, esi
:00401C91 E802D61300              call 0053F298
:00401C96 8D45F0                  lea eax, dword ptr [ebp-10]
:00401C99 E8DED61300              call 0053F37C
:00401C9E 8B55C4                  mov edx, dword ptr [ebp-3C]
:00401CA1 8902                    mov dword ptr [edx], eax
:00401CA3 FF4F1C                  dec [edi+1C]
:00401CA6 8D45F0                  lea eax, dword ptr [ebp-10]
:00401CA9 BA02000000              mov edx, 00000002
:00401CAE E86DD41300              call 0053F120
:00401CB3 46                      inc esi
:00401CB4 8345C404                add dword ptr [ebp-3C], 00000004
:00401CB8 83FE09                  cmp esi, 00000009
:00401CBB 0F8E32FFFFFF            jle 00401BF3
:00401CC1 33F6                    xor esi, esi
:00401CC3 8D95BCFEFFFF            lea edx, dword ptr [ebp+FFFFFEBC]
:00401CC9 8D8548FFFFFF            lea eax, dword ptr [ebp+FFFFFF48]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401CE4(C)
|
:00401CCF 8B1A                    mov ebx, dword ptr [edx]
:00401CD1 8B08                    mov ecx, dword ptr [eax]
:00401CD3 83C004                  add eax, 00000004
:00401CD6 83C204                  add edx, 00000004
:00401CD9 46                      inc esi
:00401CDA 898C9D14FFFFFF          mov dword ptr [ebp+4*ebx-000000EC], ecx
:00401CE1 83FE0A                  cmp esi, 0000000A
:00401CE4 7EE9                    jle 00401CCF
:00401CE6 BE01000000              mov esi, 00000001
:00401CEB 8D55A0                  lea edx, dword ptr [ebp-60]
:00401CEE 8D8518FFFFFF            lea eax, dword ptr [ebp+FFFFFF18]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401D02(C)
|
   //上面都是一些循环，不知道什么用，没看明白。
:00401CF4 8B08                    mov ecx, dword ptr [eax]      //ECX=   (假码：98765432109)
1、取假码第7位(3)
2、取假码第6位(4)
3、取假码第8位(2)
4、取假码第1位(9)
5、取假码第9位(1)
6、取假码第3位(7)
7、取假码第10位(0)
8、取假码第4位(6)
9、取假码第5位(5)
没有假码第2位和第11位，大概是任意数字吧，往下看。
:00401CF6 890A                    mov dword ptr [edx], ecx     //ecx=342917065
:00401CF8 46                      inc esi
:00401CF9 83C204                  add edx, 00000004
:00401CFC 83C004                  add eax, 00000004
:00401CFF 83FE09                  cmp esi, 00000009
:00401D02 7EF0                    jle 00401CF4
:00401D04 BB04000000              mov ebx, 00000004
:00401D09 BE01000000              mov esi, 00000001
:00401D0E 3BDE                    cmp ebx, esi
:00401D10 7C2B                    jl 00401D3D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401D3B(C)
|
:00401D12 8D8574FFFFFF            lea eax, dword ptr [ebp+FFFFFF74]
:00401D18 50                      push eax
:00401D19 8D559C                  lea edx, dword ptr [ebp-64]
:00401D1C 52                      push edx
:00401D1D E8FAFCFFFF              call 00401A1C
:00401D22 83C408                  add esp, 00000008
:00401D25 8D4D9C                  lea ecx, dword ptr [ebp-64]
:00401D28 51                      push ecx
:00401D29 8D8574FFFFFF            lea eax, dword
```
