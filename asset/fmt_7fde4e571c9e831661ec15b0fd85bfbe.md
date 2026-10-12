
```
:00402BAB 41                 inc ecx
:00402BAC 83F90E             cmp ecx, 0000000E
:00402BAF 7CDE               jl 00402B8F -----------------------+
:00402BB1 83FE04             cmp esi, 00000004
:00402BB4 750F               jne 00402BC5  〈== if eax=4, you'll recover more 30
                                   days to evaluate it
:00402BB6 FF7510             push [ebp+10]
:00402BB9 FF7508             push [ebp+08]
:00402BBC E8A3000000         call 00402C64  <==写注册表
:00402BC1 59                 pop ecx
:00402BC2 59                 pop ecx
:00402BC3 EB3D               jmp 00402C02

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402BB4(C)

:00402BC5 8B4510             mov eax, dword ptr [ebp+10]  <==eax=5
:00402BC8 3BF0               cmp esi, eax
:00402BCA 7508               jne 00402BD4  <==good/bad boy
:00402BCC 48                 dec eax
:00402BCD 7425               je 00402BF4
:00402BCF 83E804             sub eax, 00000004
:00402BD2 7419               je 00402BED

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00402B82(C), :00402BCA(C)

:00402BD4 6A10               push 00000010

* Possible StringData Ref from Data Obj ->"Invalid Code"

:00402BD6 6888134400        push 00441388

* Possible StringData Ref from Data Obj ->"Sorry, Invalid Code!"

:00402BDB 6870134400        push 00441370
:00402BE0 FF7508            push [ebp+08]

* Reference To: USER32.MessageBoxA, Ord:0195h

:00402BE3 FF1574BD4400      Call dword ptr [0044BD74]
:00402BE9 33C0              xor eax, eax
:00402BEB EB15              jmp 00402C02

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402BD2(C)

* Possible StringData Ref from Data Obj ->"{34ee9750-9350-11d1-8fee-004095e2400c}"

:00402BED 6890114400        push 00441190
:00402BF2 EB05              jmp 00402BF9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402BCD(C)

* Possible StringData Ref from Data Obj ->"{6ea280e0-9350-11d1-8fee-004095e2400c}"

:00402BF4 68E0114400        push 004411E0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402BF2(U)

:00402BF9 E808000000        call 00402C06  <==写注册表
:00402BFE 59                pop ecx
:00402BFF 6A01             push 00000001
:00402C01 58                pop eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00402BC3(U), :00402BEB(U)

:00402C02 5F                pop edi
:00402C03 5E                pop esi
:00402C04 5D                pop ebp
:00402C05 C3                ret
```

从`:00402B8F`到`:00402BAC`计算注册码，你能看懂吗？其中，`edi`指向`code`字符串的首地址，我们可以将我们输入的`code`看作一个字符串数组（假设为`reg(I)`），按顺序取字符串的每一位，共进行十四次循环，每次将该位字符的ASCII码值与`0x41`（字符"A"）相比，若小于则将码值减去`0x30`（字符"0"），所得值加上`esi`（初始值为零）再赋给`esi`；若大于等于，则减去`0x41`，其他步骤同前。每次循环的最后将`esi`与`9`比较，大于`9`则令`esi = esi - 9`。

循环计算结束，所得`esi`值若等于`4`，你将重新获得`30`天试用期，等于`5`，你就是注册用户了，等于其他数，程序就认为你输入了无效的注册码。

为了便于理解，我将上面计算注册码的部分翻译成BASIC语言（注：`48`, `65`代表"0", "A"十进制ASCII值）：

```basic
esi=0
for i=1 to 14
  a=asc(reg(i))-65
    if a<0 then
      esi=esi+asc(reg(i))-48
    else
      esi=esi+asc(reg(i))-65
    end if
  if esi>9 then
    esi=esi-9
  end if
next i
if esi=5 then goto goodbay
else
  if esi=4 then goto more30days
end if
goto badboy
```

经过上面的讲述你能独立算出正确的注册码吗？可以看出，有无数个`14`位字符组成的字符串符合上述条件。最简单、直接的推算，可以写出`50000000000000`, `5jjjjjjjjjjjjj`, `11111111111111`.......你能类推吗？

If you have any questions, please free drop me at `dreamtheater@263.net`

`dREAMtHEATER`

`4dd466d0e1e0cc123afbe504da2462ca`
