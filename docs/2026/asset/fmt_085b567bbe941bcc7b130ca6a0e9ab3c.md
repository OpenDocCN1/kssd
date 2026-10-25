
# eBook Workshop v1.4

## 破解信息

破解者：HMILY[CCG][BCG]

破解于：2003-4-16

软件名称：eBook Workshop v1.4

## 说明

软件的算法不难，就是写注册机时有点乱，头都写大了，还没有写完，晕！！！

贴个过程吧先！！！！

## 注册码分析

注册码有三个框，分别设为`x1`, `x2`, `x3`。

```
* Possible StringData Ref from Code Obj ->"bestloveyang"
       |
:004C7B24 BA10854C00
     mov edx, 004C8510 ; 字符串 bestloveyang 传入edx 设该字符串为 C_1
:004C7B29
E83EC2F3FF              call 00403D6C
:004C7B2E
8D55F8                  lea edx,
dword ptr [ebp-08]
:004C7B31 8B45FC
     mov eax, dword ptr [ebp-04]
:004C7B34 E85B6CFFFF
           call 004BE794
:004C7B39 8B45F8
               mov eax, dword ptr [ebp-08]
;注册名计算得到的字符串传入eax 设为C_2
:004C7B3C E813C4F3FF
     call 00403F54
:004C7B41 8945EC
         mov dword ptr [ebp-14], eax
:004C7B44 8B45F4
                 mov eax, dword ptr
[ebp-0C]
:004C7B47 E808C4F3FF              call
00403F54
:004C7B4C 8945E8
   mov dword ptr [ebp-18], eax
:004C7B4F C745F063000000
     mov [ebp-10], 00000063 ; 有用 把 0x63 保存到ebp-10
:004C7B56
8D45E0                  lea eax,
dword ptr [ebp-20]
:004C7B59 B9FF000000
   mov ecx, 000000FF
:004C7B5E BA01000000
     mov edx, 00000001
:004C7B63 E834C6F3FF
       call 0040419C
:004C7B68 8D45D8
           lea eax, dword ptr [ebp-28]
:004C7B6B
B9FF000000              mov ecx, 000000FF
:004C7B70
BA01000000              mov edx, 00000001
:004C7B75
E822C6F3FF              call 0040419C
:004C7B7A
C745E401000000          mov [ebp-1C], 00000001
:004C7B81
C645DE01                mov [ebp-22],
01
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:004C7C22(C)
|
:004C7B85 8D45E0
               lea eax, dword ptr [ebp-20]
:004C7B88
50                      push
eax
:004C7B89 8B4DE8
 mov ecx, dword ptr [ebp-18]
:004C7B8C 8B55E4
         mov edx, dword ptr [ebp-1C]
:004C7B8F 8B45F8
                 mov eax, dword ptr
[ebp-08]
:004C7B92 E8C5C5F3FF              call
0040415C
:004C7B97 8B45E4
   mov eax, dword ptr [ebp-1C]
:004C7B9A 0345E8
           add eax, dword ptr [ebp-18]
:004C7B9D
48                      dec
eax
:004C7B9E 3B45EC
 cmp eax, dword ptr [ebp-14]
:004C7BA1 7E16
           jle 004C7BB9
:004C7BA3 8B45EC
               mov eax, dword ptr [ebp-14]
:004C7BA6
40                      inc
eax
:004C7BA7 2B45E4
 sub eax, dword ptr [ebp-1C]
:004C7BAA 8945E8
         mov dword ptr [ebp-18], eax
:004C7BAD 807DDE00
               cmp byte ptr [ebp-22],
00
:004C7BB1 7406
 je 004C7BB9
:004C7BB3 8B45EC
     mov eax, dword ptr [ebp-14]
:004C7BB6 8945E8
             mov dword ptr [ebp-18], eax
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004C7BA1(C),
:004C7BB1(C)
|
:004C7BB9 C645DE00
   mov [ebp-22], 00
:004C7BBD 8B75E8
         mov esi, dword ptr [ebp-18]
:004C7BC0 85F6
                   test esi,
esi
:004C7BC2 7E47
   jle 004C7C0B
:004C7BC4 BB01000000
     mov ebx, 00000001
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7C09(C)
|
:004C7BC9
8B45E0                  mov eax,
dword ptr [ebp-20]
:004C7BCC 0FB67C18FF
   movzx edi, byte ptr [eax+ebx-01] ;依次取C_2
:004C7BD1 8B45F4
                 mov eax, dword ptr
[ebp-0C]
:004C7BD4 0FB64418FF              movzx
eax, byte ptr [eax+ebx-01] ;依次取 C_1；第二次循环C_1又从头取
:004C7BD9 33F8
                 xor edi, eax
                  ; edi=C_2^C_1
:004C7BDB
8D45E0                  lea eax,
dword ptr [ebp-20]
:004C7BDE B901000000
   mov ecx, 00000001
:004C7BE3 8BD3
         mov edx, ebx
:004C7BE5 E8B2C5F3FF
           call 0040419C
:004C7BEA 8BC7
                 mov eax, edi
:004C7BEC
8845DF                  mov byte
ptr [ebp-21], al
:004C7BEF 8D45CC
     lea eax, dword ptr [ebp-34]
:004C7BF2 8A55DF
             mov dl, byte ptr [ebp-21]
:004C7BF5
E882C2F3FF              call 00403E7C
:004C7BFA
8B45CC                  mov eax,
dword ptr [ebp-34]
:004C7BFD 8D55E0
     lea edx, dword ptr [ebp-20]
:004C7C00 8BCB
               mov ecx, ebx
:004C7C02
E8DDC5F3FF              call 004041E4
:004C7C07
43                      inc
ebx
:004C7C08 4E
   dec esi
:004C7C09 75BE
       jne 004C7BC9
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7BC2(C)
|
:004C7C0B
8D45D8                  lea eax,
dword ptr [ebp-28]
:004C7C0E 8B55E0
     mov edx, dword ptr [ebp-20] ; 把计算得到的字符串传入edx 此字符串设为 S_1
:004C7C11
E846C3F3FF              call 00403F5C
:004C7C16
8B45E8                  mov eax,
dword ptr [ebp-18]
:004C7C19 0145E4
     add dword ptr [ebp-1C], eax
:004C7C1C 8B45E4
             mov eax, dword ptr [ebp-1C]
:004C7C1F
3B45EC                  cmp eax,
dword ptr [ebp-14]
:004C7C22 0F8E5DFFFFFF
 jle 004C7B85
:004C7C28 8D45F8
     lea eax, dword ptr [ebp-08]
:004C7C2B 8B55D8
             mov edx, dword ptr [ebp-28]
:004C7C2E
E839C1F3FF              call 00403D6C
:004C7C33
8B45F8                  mov eax,
dword ptr [ebp-08]
:004C7C36 E819C3F3FF
   call 00403F54
:004C7C3B 83F804
       cmp eax, 00000004
:004C7C3E 7D10
             jge 004C7C50
:004C7C40 8D45F8
                 lea eax, dword ptr
[ebp-08]
:004C7C43 8B4DF8
   mov ecx, dword ptr [ebp-08]
*
Possible StringData Ref from Code Obj ->"love"
     |
:004C7C46 BA28854C00
   mov edx, 004C8528
:004C7C4B E850C3F3FF
     call 00403FA0
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7C3E(C)
|
:004C7C50
8B45F8                  mov eax,
dword ptr [ebp-08]
:004C7C53 E8FCC2F3FF
   call 00403F54
:004C7C58 8BC8
         mov ecx, eax
:004C7C5A 83E904
             sub ecx, 00000004
:004C7C5D
8D45F8                  lea eax,
dword ptr [ebp-08]
:004C7C60 BA01000000
   mov edx, 00000001
:004C7C65 E832C5F3FF
     call 0040419C
:004C7C6A 8B45F8
         mov eax, dword ptr [ebp-08]
:004C7C6D E8E2C2F3FF
             call 00403F54
:004C7C72 8945EC
                 mov dword ptr [ebp-14],
eax
:004C7C75 8D45D8
 lea eax, dword ptr [ebp-28]
:004C7C78 B9FF000000
       mov ecx, 000000FF
:004C7C7D BA01000000
         mov edx, 00000001
:004C7C82 E815C5F3FF
           call 0040419C
:004C7C87 8B75EC
               mov esi, dword ptr [ebp-14]
:004C7C8A
85F6                    test
esi, esi
:004C7C8C 7E2B
   jle 004C7CB9
:004C7C8E BB01000000
     mov ebx, 00000001
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7CB7(C)
|
:004C7C93
8D45D4                  lea eax,
dword ptr [ebp-2C]
:004C7C96 50
       push eax
:004C7C97 8B55EC
         mov edx, dword ptr [ebp-14]
:004C7C9A 2BD3
                   sub edx,
ebx
:004C7C9C 42
   inc edx
:004C7C9D B901000000
   mov ecx, 00000001
:004C7CA2 8B45F8
         mov eax, dword ptr [ebp-08]
:004C7CA5 E8B2C4F3FF
             call 0040415C
:004C7CAA 8D45D8
                 lea eax, dword ptr
[ebp-28]
:004C7CAD 8B55D4
   mov edx, dword ptr [ebp-2C]
:004C7CB0 E8A7C2F3FF
         call 00403F5C   ; 这一段是把计算得到的后四位字符串取反
此字符串设为 S_2
:004C7CB5 43
     inc ebx
:004C7CB6 4E
           dec esi
:004C7CB7 75DA
               jne 004C7C93    ;
未取完，继续
* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:004C7C8C(C)
|
:004C7CB9
8D45F8                  lea eax,
dword ptr [ebp-08]
:004C7CBC 8B55D8
     mov edx, dword ptr [ebp-28]
:004C7CBF E8A8C0F3FF
           call 00403D6C
:004C7CC4 8D45D8
               lea eax, dword ptr [ebp-28]
:004C7CC7
B9FF000000              mov ecx, 000000FF
:004C7CCC
BA01000000              mov edx, 00000001
:004C7CD1
E8C6C4F3FF              call 0040419C
:004C7CD6
8D55D8                  lea edx,
dword ptr [ebp-28]
:004C7CD9 A140034D00
   mov eax, dword ptr [004D0340]
:004C7CDE 8B00
             mov eax, dword ptr [eax]
:004C7CE0
8B80D4020000            mov eax, dword ptr [eax+000002D4]
:004C7CE6
E8A1B4F6FF              call 0043318C
:004C7CEB
8B75EC                  mov esi,
dword ptr [ebp-14]
:004C7CEE 85F6
       test esi, esi
:004C7CF0 0F8EB1000000
```
