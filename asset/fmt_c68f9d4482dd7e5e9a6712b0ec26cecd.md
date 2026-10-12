
# 00403930 Assembly Trace Analysis

```
:00403930 8BD0                     mov edx, eax
:00403932 8D45E8                   lea eax, dword ptr [ebp-18]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040396B(C)

:00403935 8B08                     mov ecx, dword ptr [eax]
  1、====>ECX=A4
  2、====>ECX=AA
  3、====>ECX=B7
  4、====>ECX=7E
  5、====>ECX=70
  6、====>ECX=3E

:00403937 85C9                     test ecx, ecx
:00403939 7903                     jns 0040393E
:0040393B 83C10F                   add ecx, 0000000F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00403939(C)

:0040393E C1F904                   sar ecx, 04
  (Same as taking front/back bits at 0040394B) ====> Arithmetic right shift 4 times!
  1、====>ECX=A
  2、====>ECX=A
  3、====>ECX=B
  4、====>ECX=7
  5、====>ECX=7
  6、====>ECX=3

:00403941 8A8964F14100             mov cl, byte ptr [ecx+0041F164]
  =====> Lookup table from [ecx+0041F164]!
  1、====>CL=1
  2、====>CL=1
  3、====>CL=P
  4、====>CL=G
  5、====>CL=G
  6、====>CL=A

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
[0041F164] Table values!
0041F164   4B 33 35 41 37 43 32 47 30 34 31 50 39 36 44 38   K35A7C2G041P96D8
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

:00403947 880A                     mov byte ptr [edx], cl
  =====> Save result at [EDX]

:00403949 8B08                     mov ecx, dword ptr [eax]
  1-1、====>ECX=A4
  2-2、====>ECX=AA
  3-3、====>ECX=B7
  4-4、====>ECX=7E
  5-5、====>ECX=70
  6-6、====>ECX=3E

:0040394B 81E10F000080             and ecx, 8000000F
  =====> Perform 'AND' operation with 8000000F respectively!
  1-1、====>ECX=4
  2-2、====>ECX=A
  3-3、====>ECX=7
  4-4、====>ECX=E
  5-5、====>ECX=0
  6-6、====>ECX=E

:00403951 7905                     jns 00403958
:00403953 49                       dec ecx
:00403954 83C9F0                   or ecx, FFFFFFF0
:00403957 41                       inc ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00403951(C)

:00403958 8A8964F14100             mov cl, byte ptr [ecx+0041F164]
  =====> Lookup table from [ecx+0041F164]!
  1-1、====>CL=7
  2-2、====>CL=1
  3-3、====>CL=G
  4-4、====>CL=D
  5-5、====>CL=K
  6-6、====>CL=D

:0040395E 884A01                   mov byte ptr [edx+01], cl
  =====> Save result at [EDX+01]

:00403961 43                       inc ebx
:00403962 83C202                   add edx, 00000002
:00403965 83C004                   add eax, 00000004
:00403968 83FB06                   cmp ebx, 00000006
  =====> Loop 6 times

:0040396B 7CC8                     jl 00403935
  =====> Continue loop?

:0040396D 8B4508                   mov eax, dword ptr [ebp+08]
  =====> Result from above loop goes into EAX
  =====> EAX=1711PGGDGKAD (Ha, this is the real code!)

:00403970 C6400C00                 mov [eax+0C], 00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004038D1(U)

:00403974 5F                       pop edi
:00403975 5E                       pop esi
:00403976 5B                       pop ebx
:00403977 8BE5                     mov esp, ebp
:00403979 5D                       pop ebp
:0040397A C3                       ret
```

---

# KeyMake Memory Register Details

- Interruption Address: `404370`
- Interruption Count: `1`
- First Byte: `E8`
- Instruction Length: `5`

- Interruption Address: `403AC5`
- Interruption Count: `1`
- First Byte: `59`
- Instruction Length: `1`

Memory Mode: `EAX`

---

# Registration Info Saving

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\AKoff\Composer]

"UserMail"="fly@263.net"
"KeyCode"="1711PGGDGKAD"
```

---

# Final Output

Your E-mail: `fly@263.net`
Your Code: `1711PGGDGKAD`

---

Cracked By 巢水工作坊——fly【OCN】
2:58 03-3-7
`4dd466d0e1e0cc123afbe504da2462ca`
