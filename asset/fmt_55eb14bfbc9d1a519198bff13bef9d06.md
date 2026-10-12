
```
:0049B4A1 8B45FC                 mov eax, dword ptr [ebp-04]  
                                  ====>EAX=ABC-123456-7890-ROCFLY

:0049B4A4 E81F93F6FF             call 004047C8  

:0049B4A9 8BF0                   mov esi, eax  

:0049B4AB 8BC6                   mov eax, esi  

:0049B4AD 5A                     pop edx  

:0049B4AE E881DFF6FF             call 00409434  

                                  ====>比较试炼码前 4 位是否是 KGL-  
                    可以把试炼码的前 4 位改为 KGL-   也可以在下面 R FL Z  改变跳转！

:0049B4B3 8BF8                   mov edi, eax  

:0049B4B5 3BFE                   cmp edi, esi  

:0049B4B7 740D                   je 0049B4C6  

                                  ====>不跳则 OVER！

:0049B4B9 33C0                   xor eax, eax  

:0049B4BB 5A                     pop edx  

:0049B4BC 59                    pop ecx  

:0049B4BD 59                    pop ecx  

:0049B4BE 648910                 mov dword ptr fs:[eax], edx  

:0049B4C1 E9E0000000             jmp 0049B5A6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B4B7(C)  

|  

:0049B4C6 B804000000             mov eax, 00000004  

                                  ====>EAX=4

:0049B4CB 8B55FC                 mov edx, dword ptr [ebp-04]  

                                  ====>EDX=ABC-123456-7890-ROCFLY

:0049B4CE 48                    dec eax  

:0049B4CF 85D2                  test edx, edx  

:0049B4D1 7405                  je 0049B4D8  

:0049B4D3 3B42FC                cmp eax, dword ptr [edx-04]  

:0049B4D6 7205                  jb 0049B4DD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B4D1(C)  

|  

:0049B4D8 E80780F6FF             call 004034E4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B4D6(C)  

|  

:0049B4DD 40                    inc eax  

:0049B4DE 807C02FF2D             cmp byte ptr [edx+eax-01], 2D  

                                  ====>比较第 4 位是否是 -

:0049B4E3 753E                  jne 0049B523  

                                  ====>跳则 OVER！

:0049B4E5 B80B000000             mov eax, 0000000B  

                                  ====>EAX=B

:0049B4EA 8B55FC                 mov edx, dword ptr [ebp-04]  

                                  ====>EDX=ABC-123456-7890-ROCFLY

:0049B4ED 48                    dec eax  

:0049B4EE 85D2                  test edx, edx  

:0049B4F0 7405                  je 0049B4F7  

:0049B4F2 3B42FC                cmp eax, dword ptr [edx-04]  

:0049B4F5 7205                  jb 0049B4FC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B4F0(C)  

|  

:0049B4F7 E8E87FF6FF             call 004034E4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B4F5(C)  

|  

:0049B4FC 40                    inc eax  

:0049B4FD 807C02FF2D             cmp byte ptr [edx+eax-01], 2D  

                                  ====>比较第 11 位是否是 -

:0049B502 751F                  jne 0049B523  

                                  ====>跳则 OVER！

:0049B504 B810000000             mov eax, 00000010  

                                  ====>EAX=10

:0049B509 8B55FC                 mov edx, dword ptr [ebp-04]  

                                  ====>EDX=ABC-123456-7890-ROCFLY

:0049B50C 48                    dec eax  

:0049B50D 85D2                  test edx, edx  

:0049B50F 7405                  je 0049B516  

:0049B511 3B42FC                cmp eax, dword ptr [edx-04]  

:0049B514 7205                  jb 0049B51B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B50F(C)  

|  

:0049B516 E8C97FF6FF             call 004034E4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B514(C)  

|  

:0049B51B 40                    inc eax  

:0049B51C 807C02FF2D             cmp byte ptr [edx+eax-01], 2D  

                                  ====>比较第 16 位是否是 -

:0049B521 740A                  je 0049B52D  

                                  ====>不跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:0049B4E3(C), :0049B502(C)  

|  

:0049B523 33C0                  xor eax, eax  

:0049B525 5A                    pop edx  

:0049B526 59                   pop ecx  

:0049B527 59                   pop ecx  

:0049B528 648910                mov dword ptr fs:[eax], edx  

:0049B52B EB79                  jmp 0049B5A6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B521(C)  

|  

:0049B52D 8D45F8                lea eax, dword ptr [ebp-08]  

:0049B530 50                    push eax  

:0049B531 B906000000             mov ecx, 00000006  

:0049B536 BA05000000             mov edx, 00000005  

:0049B53B 8B45FC                mov eax, dword ptr [ebp-04]  

                                  ====>EAX=ABC-123456-7890-ROCFLY

:0049B53E E8E592F6FF             call 00404828  

:0049B543 8D45F4                lea eax, dword ptr [ebp-0C]  

:0049B546 50                    push eax  

:0049B547 B904000000             mov ecx, 00000004  

:0049B54C BA0C000000             mov edx, 0000000C  

:0049B551 8B45FC                mov eax, dword ptr [ebp-04]  

:0049B554 E8CF92F6FF             call 00404828  

:0049B559 8D45EC                lea eax, dword ptr [ebp-14]  

:0049B55C 50                    push eax  

:0049B55D B906000000             mov ecx, 00000006  

:0049B562 BA11000000             mov edx, 00000011  

:0049B567 8B45FC                mov eax, dword ptr [ebp-04]  

:0049B56A E8B992F6FF             call 00404828  

:0049B56F 8D4DF0                lea ecx, dword ptr [ebp-10]  

                                  ====>ECX=KGL-

:0049B572 8B55F4                mov edx, dword ptr [ebp-0C]  

                                  ====>EDX=7890

:0049B575 8B45F8                mov eax, dword ptr [ebp-08]  

                                  ====>EAX=123456

:0049B578 E86FFAFFFF             call 0049AFEC  

                                  ====>关键 CALL！运算后 6 位注册码！进入！

:0049B57D 8B45EC                mov eax, dword ptr [ebp-14]  

                                  ====>EAX=ROCFLY

:0049B580 8B55F0                mov edx, dword ptr [ebp-10]  

                                  ====>EDX=UXBDYV

:0049B583 E88C91F6FF             call 00404714  

                                  ====>比较后 6 位注册码！

:0049B588 7504                  jne 0049B58E  

                                  ====>跳则 OVER！

:0049B58A B301                  mov bl, 01  

:0049B58C EB02                  jmp 0049B590

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B588(C)  

|  

:0049B58E 33DB                  xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B58C(U)  

|  

:0049B590 33C0                  xor eax, eax  

:0049B592 5A                    pop edx  

:0049B593 59                   pop ecx  

:0049B594 59                   pop ecx  

:0049B595 648910                mov dword ptr fs:[eax], edx  

:0049B598 EB0C                  jmp 0049B5A6  

:0049B59A E9ED84F6FF             jmp 00403A8C  

:0049B59F 33DB                  xor ebx, ebx  

:0049B5A1 E84E88F6FF             call 00403DF4

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:0049B457(U), :0049B4C1(U), :0049B52B(U), :0049B598(U)  

|  

:0049B5A6 33C0                  xor eax, eax  

:0049B5A8 5A                    pop edx  

:0049B5A9 59                   pop ecx  

:0049B5AA 59                   pop ecx  

:0049B5AB 648910                mov dword ptr fs:[eax], edx  

:0049B5AE 68C8B54900             push 0049B5C8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049B5C6(U)  

|  

:0049B5B3 8D45EC                lea eax, dword ptr [ebp-14]  
```
