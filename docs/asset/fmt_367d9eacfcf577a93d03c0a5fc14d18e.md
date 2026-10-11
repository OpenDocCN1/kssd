
```
:0040178D DD1C24                   fstp qword ptr [esp]

* Possible StringData Ref from Data Obj ->"%.0f-%.0f-%.0f-%.0f-%.0f-%.0f"

:00401790 6870914000               push 00409170
:00401795 52                       push edx
:00401796 E835020000               call 004019D0

//由输入的注册码进行运算后的结果应该
//等于给出的字符串

:0040179B 83C438                   add esp, 00000038
:0040179E 8DB4243C010000           lea esi, dword ptr [esp+0000013C]
:004017A5 8D8424A0010000           lea eax, dword ptr [esp+000001A0]
:004017AC 5D                       pop ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017CB(C)
|
:004017AD 8A10                     mov dl, byte ptr [eax]
:004017AF 8ACA                     mov cl, dl
:004017B1 3A16                     cmp dl, byte ptr [esi]
:004017B3 751C                     jne 004017D1
:004017B5 3ACB                     cmp cl, bl
:004017B7 7414                     je 004017CD
:004017B9 8A5001                   mov dl, byte ptr [eax+01]
:004017BC 8ACA                     mov cl, dl
:004017BE 3A5601                   cmp dl, byte ptr [esi+01]
:004017C1 750E                     jne 004017D1
:004017C3 83C002                   add eax, 00000002
:004017C6 83C602                   add esi, 00000002
:004017C9 3ACB                     cmp cl, bl
:004017CB 75E0                     jne 004017AD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017B7(C)
|
:004017CD 33C0                     xor eax, eax
:004017CF EB05                     jmp 004017D6

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004017B3(C), :004017C1(C)
|
:004017D1 1BC0                     sbb eax, eax
:004017D3 83D8FF                   sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017CF(U)
|
:004017D6 3BC3                     cmp eax, ebx
:004017D8 7524                     jne 004017FE
:004017DA 8B842404020000           mov eax, dword ptr [esp+00000204]
:004017E1 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Good Boy!"

:004017E3 6864914000               push 00409164

* Possible StringData Ref from Data Obj ->"You made it!"

:004017E8 6854914000               push 00409154
:004017ED 50                       push eax

* Reference To: USER32.MessageBoxA, Ord:0000h

:004017EE FF15CC804000             Call dword ptr [004080CC]
:004017F4 5F                       pop edi
:004017F5 5E                       pop esi
:004017F6 5B                       pop ebx
:004017F7 81C4F4010000            add esp, 000001F4
:004017FD C3                       ret
```

```
/////////////////////////////////////////////////////////////////////
//
// 我的结果：228838-461710-264942-646278-265453-42285
//
/////////////////////////////////////////////////////////////////////
//
// 呵呵，其实算法不难，就是方程组难解(数学忘光了)
//
// 至于注册机嘛，就是解方程了。不过这个方程解起来可以偷懒。
//
/////////////////////////////////////////////////////////////////////
```

`4dd466d0e1e0cc123afbe504da2462ca`
