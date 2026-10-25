
```text
E8E6EBF5FF              call 004046B8

:004A5AD2

8B45D8                  mov eax,

dword ptr [ebp-28]

:004A5AD5 E87EEBF5FF

   call 00404658

:004A5ADA 8A00

         mov al, byte ptr [eax]

                    1、

 ====>AL=66

          2、    ====>AL=6C

                      3、

   ====>AL=79

:004A5ADC

25FF000000              and eax, 000000FF

:004A5AE1

03D8                    add

ebx, eax

      1、    ====>EBX=00000000 + 66=00000066

2、    ====>EBX=042EB57D + 6C=042EB5E9

                  3、    ====>EBX=085D6B00

+ 79=085D6B79

:004A5AE3

81C3C6CA5503            add ebx, 0355CAC6

1、    ====>EBX=00000066 + 0355CAC6=0355CB2C

                    2、

 ====>EBX=042EB5E9 + 0355CAC6=078480AF

                  3、    ====>EBX=085D6B79

+ 0355CAC6=0BB3363F

:004A5AE9

035DE8                  add ebx,

dword ptr [ebp-18]

          1、    ====>EBX=0355CB2C + 00D8EA48=042EB574

  2、    ====>EBX=078480AF + 00D8EA48=085D6AF7

                      3、

   ====>EBX=0BB3363F + 00D8EA48=0C8C2087

:004A5AEC

035DE4                  add ebx,

dword ptr [ebp-1C]

          1、    ====>EBX=042EB574 + 9=042EB57D

  2、    ====>EBX=085D6AF7 + 9=085D6B00

                    3、

 ====>EBX=0C8C2087 + 9=0C8C2090

               呵呵，0C8C2090的10进制值就是我的注册码了！

:004A5AEF

FF45EC                  inc [ebp-14]

:004A5AF2

4E                      dec

esi

             ====>ESI 依次减1

:004A5AF3

75C9                    jne

004A5ABE

             ====>循环用户名位数次

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A5AB5(C)

|

:004A5AF5

8D55D4                  lea edx,

dword ptr [ebp-2C]

:004A5AF8 8B45FC

     mov eax, dword ptr [ebp-04]

:004A5AFB 8B80E0040000

         mov eax, dword ptr [eax+000004E0]

       ====>EAX=00CD45A0

:004A5B01

E87A73FBFF              call 0045CE80

:004A5B06

8B45D4                  mov eax,

dword ptr [ebp-2C]

                 ====>EAX=13572468

:004A5B09

E81A2CF6FF              call 00408728

       ====>把13572468（D）转换成16进制值EAX=00CF1974（H）

:004A5B0E

3BD8                    cmp

ebx, eax

             ====>EBX=0C8C2090（H）=210509968（D）

 注册码！

               ====>EAX=00CF1974（H）=13572468

（D）  试炼码

                  呵呵，比较注册码了。如果相等就OK了！所以我的注册码就是EBX里的10进制值！

:004A5B10

7519                    jne

004A5B2B

             ====>跳则OVER！

:004A5B12

C645FB01                mov [ebp-05],

01

           ====>置1则OK！

:004A5B16

B854F14A00              mov eax, 004AF154

:004A5B1B

8B55F4                  mov edx,

dword ptr [ebp-0C]

:004A5B1E E8C9E6F5FF

   call 004041EC

:004A5B23 891D58F14A00

   mov dword ptr [004AF158], ebx

:004A5B29 EB04

             jmp 004A5B2F

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A5B10(C)

|

:004A5B2B

C645FB00                mov [ebp-05],

00

           ====>清0则OVER！

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A5B29(U)

|

:004A5B2F

33C0                    xor

eax, eax

:004A5B31 5A

     pop edx

:004A5B32 59

           pop ecx

:004A5B33 59

                 pop ecx

:004A5B34

648910                  mov dword

ptr fs:[eax], edx

:004A5B37 68665B4A00

 push 004A5B66

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A5B64(U)

|

:004A5B3C

8D45D4                  lea eax,

dword ptr [ebp-2C]

:004A5B3F E854E6F5FF

   call 00404198

:004A5B44 8D45D8

       lea eax, dword ptr [ebp-28]

:004A5B47 BA03000000

             mov edx, 00000003

:004A5B4C

E86BE6F5FF              call 004041BC

:004A5B51

8D45F0                  lea eax,

dword ptr [ebp-10]

:004A5B54 BA02000000

   mov edx, 00000002

:004A5B59 E85EE6F5FF

     call 004041BC

:004A5B5E C3

             ret

:004A5B5F

E994DFF5FF              jmp 00403AF8

:004A5B64

EBD6                    jmp

004A5B3C

:004A5B66 8A45FB

   mov al, byte ptr [ebp-05]

                       ====>标志位

值入 AL

:004A5B69 5F

                   pop edi

:004A5B6A

5E                      pop

esi

:004A5B6B 5B

   pop ebx

:004A5B6C 8BE5

       mov esp, ebp

:004A5B6E 5D

             pop ebp

:004A5B6F C3

                   ret
```

―――――――――――――――――――――――――――――――――

### 算法总结

姓名字符的HEX值累加，再加（0355CAC6+申请号+9）* 姓名的位数

―――――――――――――――――――――――――――――――――

### KeyMake之内存注册机

中断地址：`4A5B0E`

中断次数：1

第一字节：`3B`

指令长度：2

寄存器方式：`EBX` 十进制

―――――――――――――――――――――――――――――――――

### 注册信息保存

`REGEDIT4`

`[HKEY\_USERS\.DEFAULT\Software\xt-studio\windows\exe]`

`"Name"="fly"`

`"Pass"=dword:0c8c2090`

―――――――――――――――――――――――――――――――――

### 整理

申请号：14215752

姓 名：fly

公 司：【OCN】

授权码：210509968

―――――――――――――――――――――――――――――――――

Cracked By 巢水工作坊――fly【OCN】

2003-4-6 15:20
