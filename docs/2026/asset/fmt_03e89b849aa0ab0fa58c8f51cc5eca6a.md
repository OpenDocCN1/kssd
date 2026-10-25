
```
99                      cdq
:00431E73
F7FD                    idiv
ebp
:00431E75 80C241
 add dl, 41
 余数+0x41,本例中为0x52(R)-->dl
:00431E78 81E2FF000000
 and edx, 000000FF
:00431E7E 3BFA
       cmp edi, edx
     比较，相等则继续，否则over，所以
第四组注册码的第四位应该为"R"
:00431E80 7409
         je 00431E8B
        跳，(爆破点)
:00431E82 5F
             pop edi
:00431E83 5E
                   pop esi
:00431E84
5D                      pop
ebp
:00431E85 33C0
   xor eax, eax
:00431E87 5B
         pop ebx
:00431E88 C21000
           ret 0010
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431E80(C)
|
:00431E8B
0FBE06                  movsx eax,
byte ptr [esi]       将第四组注册码的第一位-->eax，
在这里，应该用真码，否则得不到真码的最后一位，所以这里取"Q"(0x51)-->eax
:00431E8E
8D148B                  lea edx,
dword ptr [ebx+4\*ecx]  edx=0x19E
:00431E91 03CA
           add ecx, edx
           ecx=0x1F0
:00431E93 8D0440
               lea eax, dword ptr [eax+2\*eax]
 eax\*3-->eax,这里eax=0xF3
:00431E96 8D0C79
         lea ecx, dword ptr [ecx+2\*edi]  ecx=0x294
:00431E99
5F                      pop
edi
:00431E9A 03C1
   add eax, ecx
   eax=0x387
:00431E9C B91A000000
   mov ecx, 0000001A
0x1A-->ecx
:00431EA1 99
       cdq
:00431EA2 F7F9
         idiv ecx
:00431EA4 0FBE4604
           movsx eax, byte ptr [esi+04]    将第四组注册码的第五位-->eax
:00431EA8
33C9                    xor
ecx, ecx
:00431EAA 5E
     pop esi
:00431EAB 5D
           pop ebp
:00431EAC 5B
                 pop ebx
:00431EAD
80C241                  add dl, 41
                     余数+0x41,本例中为0x54(T)-->dl
:00431EB0
81E2FF000000            and edx, 000000FF
:00431EB6
3BC2                    cmp
eax, edx                    比较，相等则继续，否则over，所
以第四组注册码的第五位应该为"T"
:00431EB8 0F94C1
       sete cl
          相等则条件为真，(爆破点)
:00431EBB 8BC1
               mov eax, ecx
:00431EBD
C21000                  ret 0010
```

算法总结：

注册码格式：`s1s2s3s4s5-s6s7s8s9s10-s11s12s13s14s15s16-s17s18s19s20s21`

前16位任意，后五位由前16位推出。

算法：

1、`s17=((((s5%7)+(s3%5))+s2*2+s1)%0x1A+0x41)&0xff;`

2、`s18=(((s9%3)+s8+s10*5+s6*2+s7)%0x1A+0x41)&0xff;`

3、`s19=((s14*23+s16+s11)%0x1A+0x41)&0xff;`

4、`s20＝((s4+s8+s12*7+s13)%0x1A+0x41)&0xff;`

5、`s21=((s18*5+s19+s20*2+s17*3)%0x1A+0x41)&0xff.`

注册码与姓名，公司无关。

一个有用的注册码:`a1234-b2345-c34567-QRVRT`
