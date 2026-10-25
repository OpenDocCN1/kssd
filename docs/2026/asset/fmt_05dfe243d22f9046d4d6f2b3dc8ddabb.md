
```
:10001DE6 2501000080              and eax,

80000001    //用掩码取出该数的第11 bit

:10001DEB 7905

  jns 10001DF2

:10001DED 48

    dec eax

:10001DEE 83C8FE

or eax, FFFFFFFE

:10001DF1 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001DEB(C)

|

:10001DF2 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001DF5 8BC6

  mov eax, esi

:10001DF7 99

    cdq

:10001DF8 83E207

and edx, 00000007

:10001DFB 03C2

  add eax, edx

:10001DFD C1F803

sar eax, 03

:10001E00 2501000080              and eax,

80000001  //用掩码取出该数的第7 bit

:10001E05 7905

  jns 10001E0C

:10001E07 48

    dec eax

:10001E08 83C8FE

or eax, FFFFFFFE

:10001E0B 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E05(C)

|

:10001E0C 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E0F 8BC1

  mov eax, ecx

:10001E11 99

    cdq

:10001E12 83E207

and edx, 00000007

:10001E15 03C2

  add eax, edx

:10001E17 C1F803

sar eax, 03

:10001E1A 2501000080              and eax,

80000001      //用掩码取出该数的第3 bit

:10001E1F 7905

  jns 10001E26

:10001E21 48

    dec eax

:10001E22 83C8FE

or eax, FFFFFFFE

:10001E25 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E1F(C)

|

:10001E26 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E29 8BC3

  mov eax, ebx

:10001E2B 99

    cdq

:10001E2C 83E203

and edx, 00000003

:10001E2F 03C2

  add eax, edx

:10001E31 C1F802

sar eax, 02

:10001E34 2501000080              and eax,

80000001    //用掩码取出该数的第14 bit

:10001E39 7905

  jns 10001E40

:10001E3B 48

    dec eax

:10001E3C 83C8FE

or eax, FFFFFFFE

:10001E3F 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E39(C)

|

:10001E40 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E43 8BC7

  mov eax, edi

:10001E45 99

    cdq

:10001E46 83E203

and edx, 00000003

:10001E49 03C2

  add eax, edx

:10001E4B C1F802

sar eax, 02

:10001E4E 2501000080              and eax,

80000001  //用掩码取出该数的第10 bit

:10001E53 7905

  jns 10001E5A

:10001E55 48

    dec eax

:10001E56 83C8FE

or eax, FFFFFFFE

:10001E59 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E53(C)

|

:10001E5A 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E5D 8BC6

  mov eax, esi

:10001E5F 99

    cdq

:10001E60 83E203

and edx, 00000003

:10001E63 03C2

  add eax, edx

:10001E65 C1F802

sar eax, 02

:10001E68 2501000080              and eax,

80000001 //用掩码取出该数的第6 bit

:10001E6D 7905

  jns 10001E74

:10001E6F 48

    dec eax

:10001E70 83C8FE

or eax, FFFFFFFE

:10001E73 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E6D(C)

|

:10001E74 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E77 8BC1

  mov eax, ecx

:10001E79 99

    cdq

:10001E7A 83E203

and edx, 00000003

:10001E7D 03C2

  add eax, edx

:10001E7F C1F802

sar eax, 02

:10001E82 2501000080              and eax,

80000001  //用掩码取出该数的第2 bit

:10001E87 7905

  jns 10001E8E

:10001E89 48

    dec eax

:10001E8A 83C8FE

or eax, FFFFFFFE

:10001E8D 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E87(C)

|

:10001E8E 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001E91 8BC3

  mov eax, ebx

:10001E93 99

    cdq

:10001E94 2BC2

  sub eax, edx

:10001E96 D1F8

  sar eax, 1

:10001E98 2501000080              and eax,

80000001  //用掩码取出该数的第13 bit

:10001E9D 7905

  jns 10001EA4

:10001E9F 48

    dec eax

:10001EA0 83C8FE

or eax, FFFFFFFE

:10001EA3 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001E9D(C)

|

:10001EA4 8D2C68

lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

:10001EA7 8BC7

  mov eax, edi

:10001EA9 99

    cdq

:10001EAA 2BC2

  sub eax, edx

:10001EAC D1F8

  sar eax, 1

:10001EAE 2501000080              and eax,

80000001  //用掩码取出该数的第9 bit

:10001EB3 7905

  jns 10001EBA

:10001EB5 48

    dec eax

:10001EB6 83C8FE

or eax, FFFFFFFE

:10001EB9 40

    inc eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001EB3(C)

|

> :10001EBA 8D2C68

>     lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

> :10001EBD 8BC6

>   mov eax, esi

> :10001EBF 99

>     cdq

> :10001EC0 2BC2

>   sub eax, edx

> :10001EC2 D1F8

>   sar eax, 1

> :10001EC4 2501000080              and eax,

> 80000001  //用掩码取出该数的第5 bit

> :10001EC9 7905

>   jns 10001ED0

> :10001ECB 48

>     dec eax

> :10001ECC 83C8FE

> or eax, FFFFFFFE

> :10001ECF 40

>     inc eax

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001EC9(C)

> |

> :10001ED0 8D2C68

> lea ebp, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

> :10001ED3 8BC1

>   mov eax, ecx

> :10001ED5 99

>     cdq

> :10001ED6 2BC2

>   sub eax, edx

> :10001ED8 D1F8

>   sar eax, 1

> :10001EDA 2501000080              and eax,

> 80000001 //用掩码取出该数的第1 bit

> :10001EDF 7905

>   jns 10001EE6

> :10001EE1 48

>     dec eax

> :10001EE2 83C8FE

> or eax, FFFFFFFE

> :10001EE5 40

>     inc eax

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001EDF(C)

> |

> :10001EE6 81E301000080            and ebx, 80000001

> //用掩码取出该数的第12 bit

> :10001EEC 8D1468

> lea edx, dword ptr [eax+2\*ebp] //乘以2（二进制）再加上新的bit

> :10001EEF 7905

>   jns 10001EF6

> :10001EF1 4B

>     dec ebx

> :10001EF2 83CBFE

> or ebx, FFFFFFFE

> :10001EF5 43

>     inc ebx

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001EEF(C)

> |

> :10001EF6 81E701000080            and edi, 80000001

>   //用掩码取出该数的第8 bit

> :10001EFC 8D0453

> lea eax, dword ptr [ebx+2\*edx] //乘以2（二进制）再加上新的bit

> :10001EFF 7905

>   jns 10001F06

> :10001F01 4F

>     dec edi

> :10001F02 83CFFE

> or edi, FFFFFFFE

> :10001F05 47

>     inc edi

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001EFF(C)

> |

> :10001F06 81E601000080            and esi, 80000001

>   //用掩码取出该数的第4 bit

> :10001F0C 8D1447

> lea edx, dword ptr [edi+2\*eax] //乘以2（二进制）再加上新的bit

> :10001F0F 7905

>   jns 10001F16

> :10001F11 4E

>     dec esi

> :10001F12 83CEFE

> or esi, FFFFFFFE

> :10001F15 46

>     inc esi

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001F0F(C)

> |

> :10001F16 81E101000080            and ecx, 80000001

>     //用掩码取出该数的第0 bit

> :10001F1C 8D0456

> lea eax, dword ptr [esi+2\*edx] //乘以2（二进制）再加上新的bit

> :10001F1F 7905

>   jns 10001F26

> :10001F21 49

>     dec ecx

> :10001F22 83C9FE

> or ecx, FFFFFFFE

> :10001F25 41

>     inc ecx

>

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10001F1F(C)

> |

> :10001F26 8B5C2424

> mov ebx, dword ptr [esp+24]

> :10001F2A 8D0441

> lea eax, dword ptr [ecx+2\*eax] //乘以2（二进制）再加上新的bit

> :10001F2D 6A10

>   push 00000010

> :10001F2F 53

>     push ebx

>       //ebx中存放变换后的整数

> :10001F30 50

>     push eax

> :10001F31 89442424

> mov dword ptr [esp+24], eax

> :10001F35 E812060000              call

> 1000254C                //sprintf(

> )
```

上面的变换看似复杂，实际上可以简化为一个循环，其逆运算就是把各个bit调换回去，如下：

```c
    int a = 0x0B26;      //变换后的
    int b = 0;          //变换前的
    int index[] = {15,11,7,3,14,10,6,2,13,9,5,1,12,8,4,0};

    for(int k = 0; k < 16; k++)
    {
        b <<= 1;
        b |=  (a >> index[k]) & 1;
    }
```

这样就可以写出keymaker来，一个注册码是`01-0184-4056-8888`。注意`VerifySN()`函数实际上应该返回`0x0B`，这点在安装时不判断，但是其主程序在运行时会判断，若不等于`0x0B`则无法运行，主要是因为它用这个返回值来区分该公司的不同软件。

```
\* Possible StringData Ref from Data Obj ->"CSPQueryApp::InitInstance - About
"

                  ->"to validate
SN"

            |
:00498348 689C305200              push
0052309C
:0049834D E8FE08F7FF              call
00408C50
:00498352 83C404
add esp, 00000004
```
