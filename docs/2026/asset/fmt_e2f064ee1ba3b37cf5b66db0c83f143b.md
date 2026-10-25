
## 反汇编代码片段

```
call 004036D0

:0040358F 83C404
add esp, 00000004

:00403592 88443414
mov byte ptr [esp+esi+14], al

:00403596 46
inc esi

:00403597 83FE04
cmp esi, 00000004

:0040359A 7CE8
jl 00403584

:0040359C 33C9
xor ecx, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004035BD(C)

:0040359E 0FBE440F0F
movsx eax, byte ptr [edi+ecx+0F]

:004035A3 83F841
cmp eax, 00000041

:004035A6 7C08
jl 004035B0

:004035A8 83F85A
cmp eax, 0000005A

:004035AB 7F03
jg 004035B0

:004035AD 83C020
add eax, 00000020
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004035A6(C), :004035AB(C)

:004035B0 0FBE540C14
movsx edx, byte ptr [esp+ecx+14]

:004035B5 3BC2
cmp eax, edx /*下D esp+ecx+14看到注册码第四部分*/

:004035B7 7513
jne 004035CC /*在执行这一行之前，rfl z，不然就over了！*/

:004035B9 41
inc ecx

:004035BA 83F904
cmp ecx, 00000004

:004035BD 7CDF
jl 0040359E

:004035BF 5F
pop edi
```

至此就看到了全部注册码了，`x`退出，怎么样，注册成功了吧！

在这奉上内存注册机，它的注册算法太复杂，我的能力有限，有哪位大侠能写注册机，麻烦告诉我，谢谢！`chenqiang_81@163.com`
