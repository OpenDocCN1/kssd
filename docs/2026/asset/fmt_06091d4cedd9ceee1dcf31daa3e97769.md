
```asm
:0043F1B1 0FB607                  movzx eax, byte ptr [edi]

* Possible StringData Ref from
Data Obj ->"         (((((

         "

                 ->"

 H"

           |

:0043F1B4 8B0D40E04400            mov ecx, dword ptr [0044E040]
:0043F1BA 8A0441                  mov al, byte ptr [ecx+2*eax]
:0043F1BD 83E008                  and eax, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1AF(U)
|
:0043F1C0 85C0                    test eax, eax
:0043F1C2 7403                    je 0043F1C7
:0043F1C4 47                      inc edi
:0043F1C5 EBD2                    jmp 0043F199

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1C2(C)
|
:0043F1C7 0FB637                  movzx esi, byte ptr [edi]
:0043F1CA 47                      inc edi
:0043F1CB 83FE2D                  cmp esi, 0000002D <---第九位是否为"-"
:0043F1CE 8BEE                    mov ebp, esi
:0043F1D0 7405                    je 0043F1D7 <-----是则有另一种算法，最后取eax的补码。
:0043F1D2 83FE2B                  cmp esi, 0000002B <----第九位是不是"+"？
:0043F1D5 7504                    jne 0043F1DB <---不是则跳！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1D0(C)
|
:0043F1D7 0FB637                  movzx esi, byte ptr [edi] <-对于第九位“-”或“+”的对其后数字进行计算。
:0043F1DA 47                      inc edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1D5(C)
|
:0043F1DB 33DB                    xor ebx, ebx <--否则直接进行计算。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F20C(U)
|
:0043F1DD 833D4CE2440001          cmp dword ptr [0044E24C], 00000001 <--以下好像是在验证是否取完数码。
:0043F1E4 7E0C                    jle 0043F1F2
:0043F1E6 6A04                    push 00000004
:0043F1E8 56                      push esi
:0043F1E9 E8D1220000              call 004414BF
:0043F1EE 59                      pop ecx
:0043F1EF 59                      pop ecx
:0043F1F0 EB0B                    jmp 0043F1FD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1E4(C)
|
* Possible StringData Ref from
Data Obj ->"         (((((

         "

                 ->"

 H"

           |
:0043F1F2 A140E04400              mov eax, dword ptr [0044E040]
:0043F1F7 8A0470                  mov al, byte ptr [eax+2*esi]
:0043F1FA 83E004                  and eax, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1F0(U)
|
:0043F1FD 85C0                    test eax, eax
:0043F1FF 740D                    je 0043F20E
:0043F201 8D049B                  lea eax, dword ptr [ebx+4*ebx] <---对eax取值！
:0043F204 8D5C46D0                lea ebx, dword ptr [esi+2*eax-30] <---对ebx取值！！
:0043F208 0FB637                  movzx esi, byte ptr [edi] <----esi为相应数字的ASC码。
:0043F20B 47                      inc edi <-- 下一位
:0043F20C EBCF                    jmp 0043F1DD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F1FF(C)
|
:0043F20E 83FD2D                  cmp ebp, 0000002D <--算法选择。
:0043F211 8BC3                    mov eax, ebx <----返回eax的值！
:0043F213 7502                    jne 0043F217
:0043F215 F7D8                    neg eax <--是否取反，视有无"-"而定。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F213(C)
|
:0043F217 5F                      pop edi
:0043F218 5E                      pop esi
:0043F219 5D                      pop ebp
:0043F21A 5B                      pop ebx
:0043F21B C3                      ret
```

在`00431494`处的比较决定了注册成功与否。相等既大功告成！

在对注册码的计算中实际上是对输入的九至`n`位数字转化为十六进制，而在对注册名的算法中是在`esi`的初始值上再加上各位字符和位数的积的和除`17be`的余数。两者相等既可成功。在前面还有一个对第五位的测试，时间原因就不能再分析了

我的结果：注册名：`happycreator`
注册码：`mg37s6784216`

注册后信息会保存在注册表中：`HKCU\Software\gamani\GIFMovieGear\2.0`

删除相关信息后又变为未注册。注册码第6、7、8三位无关。
