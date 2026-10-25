
```asm
00413B70  MOV     BYTE PTR SS:[ESP+ESI+14], AL*********把计算得到的最后一组注册码的第一位：7保存!

00413B74  INC     ESI

00413B75  CMP     ESI, 4

00413B78  JL      SHORT Coursewa.00413B62**********跳回循环，计算最后一组注册码，得到：71jb

00413B7A  XOR     ECX, ECX

00413B7C  MOVSX   EAX, BYTE PTR DS:[EDI+ECX+F]*********第四组试验码的第一位：8(38)扩展到EAX

00413B81  CMP     EAX, 41

00413B84  JL      SHORT Coursewa.00413B8E

00413B86  CMP     EAX, 5A

00413B89  JG      SHORT Coursewa.00413B8E

00413B8B  ADD     EAX, 20

00413B8E  MOVSX   EDX, BYTE PTR SS:[ESP+ECX+14]*********第四组注册码的第一位：7(37)扩展到EDX

00413B93  CMP     EAX, EDX******************************比较试验码与注册码!

00413B95  JNZ     SHORT Coursewa.00413BAA**********不相等就跳，一跳就失败!

00413B97  INC     ECX

00413B98  CMP     ECX, 4

00413B9B  JL      SHORT Coursewa.00413B7C**********跳回循环，直至比较完4位注册码!

00413B9D  POP     EDI                              ;  0012FA30

00413B9E  MOV     EAX, 1

00413BA3  POP     ESI                              ;  0012FA30

00413BA4  ADD     ESP, 20

00413BA7  RETN    8
```

## 算法小结

软件的算法一环扣一环，先根据用户名计算出第一组注册码，根据第一组注册码计算出第二组注册码，根据第一组、第二组注册码计算出第三组注册码，根据第一组、第二组、第三组注册码计算出最后一组注册码。

给一个可用的注册码：

- 用户名：`yzez`
- 注册码：`3371-u1r1-ssr3-71jb`
