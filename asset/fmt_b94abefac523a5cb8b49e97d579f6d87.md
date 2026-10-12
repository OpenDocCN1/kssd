
```
:0041A820 3AC3                 cmp al, bl
:0041A822 751E                 jne 0041A842
:0041A824 84D2                 test dl, dl
:0041A826 7416                 je 0041A83E
:0041A828 8A4101               mov al, byte ptr [ecx+01]
:0041A82B 8A5E01               mov bl, byte ptr [esi+01]
:0041A82E 8AD0                 mov dl, al
:0041A830 3AC3                 cmp al, bl
:0041A832 750E                 jne 0041A842
:0041A834 83C102               add ecx, 00000002
:0041A837 83C602               add esi, 00000002
:0041A83A 84D2                 test dl, dl
:0041A83C 75DC                 jne 0041A81A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A826(C)
|

:0041A83E 33C9                 xor ecx, ecx
:0041A840 EB05                 jmp 0041A847

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A822(C), :0041A832(C)
|

:0041A842 1BC9                 sbb ecx, ecx
:0041A844 83D9FF               sbb ecx, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A840(U)
|

:0041A847 85C9                 test ecx, ecx
:0041A849 7446                 je 0041A891
:0041A84B 8DB4240C010000       lea esi, dword ptr [esp+0000010C]     \\DESI 可以看见真的注册码 B
:0041A852 8BC7                 mov eax, edi                          \\这里是我输入的注册码!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A876(C)                   \\从这里开始到 0041A876 是比较真假注册码每一位的 ASCII 码是否相等!
|

:0041A854 8A10                 mov dl, byte ptr [eax]
:0041A856 8A1E                 mov bl, byte ptr [esi]
:0041A858 8ACA                 mov cl, dl
:0041A85A 3AD3                 cmp dl, bl
:0041A85C 751E                 jne 0041A87C
:0041A85E 84C9                 test cl, cl
:0041A860 7416                 je 0041A878
:0041A862 8A5001               mov dl, byte ptr [eax+01]
:0041A865 8A5E01               mov bl, byte ptr [esi+01]
:0041A868 8ACA                 mov cl, dl
:0041A86A 3AD3                 cmp dl, bl
:0041A86C 750E                 jne 0041A87C
:0041A86E 83C002               add eax, 00000002
:0041A871 83C602               add esi, 00000002
:0041A874 84C9                 test cl, cl                          \\是否取完
:0041A876 75DC                 jne 0041A854                        \\没有取完就跳回去!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A860(C)
|

:0041A878 33C0                 xor eax, eax
:0041A87A EB05                 jmp 0041A881

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A85C(C), :0041A86C(C)
|

:0041A87C 1BC0                 sbb eax, eax
:0041A87E 83D8FF               sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A87A(U)
|

:0041A881 85C0                 test eax, eax
:0041A883 740C                 je 0041A891
:0041A885 5F                   pop edi
:0041A886 5E                   pop esi
:0041A887 33C0                 xor eax, eax                          \\清零
:0041A889 5B                   pop ebx
:0041A88A 81C400020000         add esp, 00000200
:0041A890 C3                   ret                                   \\返回

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A849(C), :0041A883(C)
|

:0041A891 5F                   pop edi
:0041A892 5E                   pop esi
:0041A893 B801000000           mov eax, 00000001                   \\赋值
:0041A898 5B                   pop ebx
:0041A899 81C400020000         add esp, 00000200
:0041A89F C3                   ret                                   \\返回

------------------------------------------------------------------
上面 0041A7ED 的 CALL 来到这里!注册码的生成和比较都再这里!关键!!!!!

* Referenced by a CALL at Addresses:
|:0041A575  , :0041A7ED   

|

:0041A1D0 53                   push ebx
:0041A1D1 56                   push esi
:0041A1D2 57                   push edi
:0041A1D3 8B7C2410             mov edi, dword ptr [esp+10]    \\DEDI 看见我输入的注册码!
:0041A1D7 32DB                 xor bl, bl
:0041A1D9 8BCF                 mov ecx, edi                  \\ECX=EDI
:0041A1DB 8A07                 mov al, byte ptr [edi]        \\我输入注册码的第一位数的 ASCII 码入 EAX 低位!
:0041A1DD 84C0                 test al, al                   \\是否为空!
:0041A1DF 740A                 je 0041A1EB                   \\为空的话就跳走!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1E9(C)                  \\从这里到 0041A1E9 是把我输入注册码的 ASCII 码加再一起放进 EBX 低位!
|

:0041A1E1 02D8                 add bl, al
:0041A1E3 8A4101               mov al, byte ptr [ecx+01]
:0041A1E6 41                   inc ecx                       \\计数器
:0041A1E7 84C0                 test al, al
:0041A1E9 75F6                 jne 0041A1E1                  \\循环!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1DF(C)
|

:0041A1EB A1C8AF4300           mov eax, dword ptr [0043AFC8]
:0041A1F0 33F6                 xor esi, esi
:0041A1F2 A3D8AF4300           mov dword ptr [0043AFD8], eax
:0041A1F7 A1C0AF4300           mov eax, dword ptr [0043AFC0]
:0041A1FC 85C0                 test eax, eax
:0041A1FE 7E2D                 jle 0041A22D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A22B(C)                  \\从这里开始到 0041A22B 就把用户名的 ASCII 码循环计算出正确注册码的!
                              \\你把内存区的地址指向 6AF20C 就可以看见注册码的变化了!
|

:0041A200 8A0C3E               mov cl, byte ptr [esi+edi]
:0041A203 32CB                 xor cl, bl
:0041A205 51                   push ecx
:0041A206 E895FFFFFF           call 0041A1A0                  \\这个是运算 CALL,F8 追进去!
:0041A20B 83C404               add esp, 00000004
:0041A20E 88043E               mov byte ptr [esi+edi], al
:0041A211 3C0A                 cmp al, 0A
:0041A213 0FBEC0               movsx eax, al
:0041A216 7D05                 jge 0041A21D
:0041A218 83C030               add eax, 00000030
:0041A21B EB03                 jmp 0041A220

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A216(C)
|

:0041A21D 83C041               add eax, 00000041

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A21B(U)
|

:0041A220 88043E               mov byte ptr [esi+edi], al
:0041A223 A1C0AF4300           mov eax, dword ptr [0043AFC0]
:0041A228 46                   inc esi                         \\计数器!
:0041A229 3BF0                 cmp esi, eax                    \\比较用户名是否取完!
:0041A22B 7CD3                 jl 0041A200                     \\没有就跳回去!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1FE(C)
|

:0041A22D C6043800             mov byte ptr [eax+edi], 00
:0041A231 5F                  pop edi
:0041A232 5E                  pop esi
:0041A233 5B                  pop ebx
:0041A234 C3                  ret

------------------------------------------------------------------
上面 0041A206 的 CALL 来到这里!

* Referenced by a CALL at Address:
|:0041A206
|

:0041A1A0 0FBE442404          movsx eax, byte ptr [esp+04]
:0041A1A5 0305D8AF4300        add eax, dword ptr [0043AFD8]
:0041A1AB 69C0697DAE42        imul eax, 42AE7D69
:0041A1B1 0531D40000          add eax, 0000D431
:0041A1B6 A3D8AF4300          mov dword ptr [0043AFD8], eax
:0041A1BB C1F810              sar eax, 10
```
