
```asm
00431A0E     8975 F8      mov dword ptr ss:[ebp-8],esi

00431A11     8975 DC      mov dword ptr ss:[ebp-24],esi

00431A14     8975 CC      mov dword ptr ss:[ebp-34],esi

00431A17     8975 BC      mov dword ptr ss:[ebp-44],esi

00431A1A     6A 01        push 1

00431A1C     FF15 F8C4430>call dword ptr ds:[<&MSVBVM50.\_\_vbaOnErro>; MSVBVM50.\_\_vbaOnError

00431A22     66:8935 2C91>mov word ptr ds:[43912C],si

00431A29     68 38914300  push locplus-.00439138

00431A2E     8D45 CC      lea eax,dword ptr ss:[ebp-34]

00431A31     50           push eax

00431A32     E8 F90E0000  call locplus-.00432930

00431A37     8D4D CC      lea ecx,dword ptr ss:[ebp-34]

00431A3A     51           push ecx

00431A3B     8B3D 38C4430>mov edi,dword ptr ds:[<&MSVBVM50.\_\_vbaStr>; MSVBVM50.\_\_vbaStrVarMove

00431A41     FFD7         call edi

00431A43     8BD0         mov edx,eax                     ；eax存放着用户名地址

00431A45     B9 28914300  mov ecx,locplus-.00439128

00431A4A     8B35 18C7430>mov esi,dword ptr ds:[<&MSVBVM50.\_\_vbaStr>; MSVBVM50.\_\_vbaStrMove

00431A50     FFD6         call esi

00431A52     8D4D CC      lea ecx,dword ptr ss:[ebp-34]

00431A55     8B1D 2CC4430>mov ebx,dword ptr ds:[<&MSVBVM50.\_\_vbaFre>; MSVBVM50.\_\_vbaFreeVar

00431A5B     FFD3         call ebx

00431A5D     68 3C914300  push locplus-.0043913C

00431A62     8D55 CC      lea edx,dword ptr ss:[ebp-34]

00431A65     52           push edx

00431A66     E8 C50E0000  call locplus-.00432930

00431A6B     8D45 CC      lea eax,dword ptr ss:[ebp-34]

00431A6E     50           push eax

00431A6F     FFD7         call edi

00431A71     8BD0         mov edx,eax                      ；eax存放着注册码地址

00431A73     8D4D DC      lea ecx,dword ptr ss:[ebp-24]

00431A76     FFD6         call esi

00431A78     8D4D CC      lea ecx,dword ptr ss:[ebp-34]

00431A7B     FFD3         call ebx

00431A7D     8D4D DC      lea ecx,dword ptr ss:[ebp-24]

00431A80     51           push ecx

00431A81     E8 0AFBFFFF  call locplus-.00431590           ；关键算法call,必须跟进

00431A86     8BD0         mov edx,eax                      ；eax为新数值的地址

00431A88     8D4D DC      lea ecx,dword ptr ss:[ebp-24]

00431A8B     FFD6         call esi

00431A8D     8B55 DC      mov edx,dword ptr ss:[ebp-24]

00431A90     52           push edx

00431A91     FF15 30C4430>call dword ptr ds:[<&MSVBVM50.\_\_vbaLenBstr> ; 取新值的长度

00431A97     83F8 0C      cmp eax,0C                             ；检验新值的长度是否为12位

00431A9A     75 6A        jnz short locplus-.00431B06              ；跳走即失败

00431A9C     8D45 DC      lea eax,dword ptr ss:[ebp-24]

00431A9F     8945 C4      mov dword ptr ss:[ebp-3C],eax

00431AA2     C745 BC 0840>mov dword ptr ss:[ebp-44],4008

00431AA9     8D4D BC      lea ecx,dword ptr ss:[ebp-44]

00431AAC     51           push ecx

00431AAD     FF15 8CC5430>call dword ptr ds:[<&MSVBVM50.rtcIsNumeric> ; 检验新值是否为浮点数值

00431AB3     66:85C0      test ax,ax                                  ；是就通过

00431AB6     74 4E        je short locplus-.00431B06                  ；跳走即失败

00431AB8     8B55 DC      mov edx,dword ptr ss:[ebp-24]

00431ABB     52           push edx

00431ABC     FF15 58C7430>call dword ptr ds:[<&MSVBVM50.rtcR8ValFromBstr>];

00431AC2     FF15 00C7430>call dword ptr ds:[<&MSVBVM50.\_\_vbaFpI4>] ; MSVBVM50.\_\_vbaFpI4

00431AC8     8945 E0      mov dword ptr ss:[ebp-20],eax

00431ACB     83F8 01      cmp eax,1                                ; 检验浮点数值的HEX值

00431ACE     7C 36        jl short locplus-.00431B06                 ；跳走即失败

00431AD0     DB45 E0      fild dword ptr ss:[ebp-20]

00431AD3     DD5D A4      fstp qword ptr ss:[ebp-5C]

00431AD6     DD45 A4      fld qword ptr ss:[ebp-5C]

00431AD9     DC1D 5020400>fcomp qword ptr ds:[402050]   ; 浮点数比较

00431ADF     DFE0         fstsw ax                      ；[402050] 既定值“9900000000.00000000”

00431AE1     F6C4 41      test ah,41

00431AE4     74 20        je short locplus-.00431B06

00431AE6     66:C705 2C91>mov word ptr ds:[43912C],0FFFF

00431AEF     FF15 DCC4430>call dword ptr ds:[<&MSVBVM50.\_\_vbaExitPr>; MSVBVM50.\_\_vbaExitProc

00431AF5     9B           wait

00431AF6     68 281B4300  push locplus-.00431B28

00431AFB     EB 21        jmp short locplus-.00431B1E

00431AFD     66:C705 2C91>mov word ptr ds:[43912C],0

00431B06     FF15 DCC4430>call dword ptr ds:[<&MSVBVM50.\_\_vbaExitPr>; MSVBVM50.\_\_vbaExitProc

00431B0C     9B           wait

00431B0D     68 281B4300  push locplus-.00431B28

00431B12     EB 0A        jmp short locplus-.00431B1E

00431B14     8D4D CC      lea ecx,dword ptr ss:[ebp-34]

00431B17     FF15 2CC4430>call dword ptr ds:[<&MSVBVM50.\_\_vbaFreeVa>; MSVBVM50.\_\_vbaFreeVar

00431B1D     C3           retn

00431B1E     8D4D DC      lea ecx,dword ptr ss:[ebp-24]

00431B21   - FF25 54C7430>jmp dword ptr ds:[<&MSVBVM50.\_\_vbaFreeStr>; MSVBVM50.\_\_vbaFreeStr

00431B27     C3           retn

00431B28     8B4D E4      mov ecx,dword ptr ss:[ebp-1C]

00431B2B     64:890D 0000>mov dword ptr fs:[0],ecx

00431B32     5F           pop edi

00431B33     5E           pop esi

00431B34     5B           pop ebx

00431B35     8BE5         mov esp,ebp

00431B37     5D           pop ebp

00431B38     C3           retn
```

---

关键算法，由`00431a81`跟进。

```asm
00431590     55     push ebp

00431591     8BEC   mov ebp,esp

00431593     83EC 0>sub esp,0C

00431596     68 C62>push <jmp.&MSVBVM50.\_\_vbaExceptHandler>

0043159B     64:A1 >mov eax,dword ptr fs:[0]

004315A1     50     push eax

004315A2     64:892>mov dword ptr fs:[0],esp

004315A9     81EC E>sub esp,0E0

004315AF     A1 589>mov eax,dword ptr ds:[439158]

004315B4     53     push ebx

004315B5     56     push esi

004315B6     57     push edi

004315B7     8965 F>mov dword ptr ss:[ebp-C],esp

004315BA     33FF   xor edi,edi

004315BC     50     push eax                        ; eax=001a7514，unicode "ONLocPlus"

004315BD     C745 F>mov dword ptr ss:[ebp-8],locplus-.00402068

004315C4     897D D>mov dword ptr ss:[ebp-24],edi

004315C7     897D D>mov dword ptr ss:[ebp-2C],edi

004315CA     897D C>mov dword ptr ss:[ebp-3C],edi

004315CD     897D C>mov dword ptr ss:[ebp-40],edi

004315D0     897D B>mov dword ptr ss:[ebp-48],edi

004315D3     897D A>mov dword ptr ss:[ebp-54],edi

004315D6     897D A>mov dword ptr ss:[ebp-58],edi

004315D9     897D 9>mov dword ptr ss:[ebp-68],edi

004315DC     897D 8>mov dword ptr ss:[ebp-78],edi

004315DF     89BD 7>mov dword ptr ss:[ebp-88],edi

004315E5     89BD 6>mov dword ptr ss:[ebp-98],edi

004315EB     89BD 5>mov dword ptr ss:[ebp-A8],edi

004315F1     89BD 4>mov dword ptr ss:[ebp-B8],edi

004315F7     89BD 3>mov dword ptr ss:[ebp-C8],edi

004315FD     89BD 2>mov dword ptr ss:[ebp-D8],edi

00431603     89BD 1>mov dword ptr ss:[ebp-E8],edi

00431609     FF15 3>call dword ptr ds:[<&MSVBVM50.\_\_vbaLenBstr>] ; 取"ONLocPlus"的长度放eax，为9

0043160F     8BC8   mov ecx,eax

00431611     FF15 9>call dword ptr ds:[<&MSVBVM50.\_\_vbaI2I4>]     ; MSVBVM50.\_\_vbaI2I4

00431617     8B4D 0>mov ecx,dword ptr ss:[ebp+8]

0043161A     BE 020>mov esi,2

0043161F     68 00D>push locplus-.0040D900                  ; UNICODE "&H"

00431624     56     push esi

00431625     8B11   mov edx,dword ptr ds:[ecx]

00431627     8945 B>mov dword ptr ss:[ebp-4C],eax        ; 把长度值9放入ebp-4c,在00431786会用到

0043162A     52     push edx                             ; 注册码地址

0043162B     FF15 F>call dword ptr ds:[<&MSVBVM50.rtcLeftCharBstr>  ; 取左边两个字符，

00431631     8B1D 1>mov ebx,dword ptr ds:[<&MSVBVM50.\_\_vbaStrMove>; MSVBVM50.\_\_vbaStrMove

00431637     8BD0   mov edx,eax

00431639     8D4D A>lea ecx,dword ptr ss:[ebp-54]

0043163C     FFD3   call ebx                           ; 把宽型第1、2字符的地址放入ebp-54中

0043163E     50     push eax

0043163F     FF15 8>call dword ptr ds:[<&MSVBVM50.\_\_vbaStrCat>] ; &H和第1、2字符连在一起

00431645     8BD0   mov edx,eax

00431647     8D4D A>lea ecx,dword ptr ss:[ebp-58]
```
