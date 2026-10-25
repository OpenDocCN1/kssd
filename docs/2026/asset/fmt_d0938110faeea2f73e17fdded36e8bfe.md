
## 汇编代码分析

```asm
:004B0A6A 0FB64008                movzx eax, byte ptr [eax+08]  <--输入的ID的倒数第二个字符的ASCII码送入EAX
```

```asm
:004B0A6E 8B55F8                  mov edx, dword ptr [ebp-08]
```

```asm
:004B0A71 0FB65209                movzx edx, byte ptr [edx+09]  <--输入的ID的倒数最后一个字符的ASCII码送入EDX
```

```asm
:004B0A75 03C2                    add eax, edx
```

```asm
:004B0A77 B90A000000              mov ecx, 0000000A
```

```asm
:004B0A7C 33D2                    xor edx, edx
```

```asm
:004B0A7E F7F1                    div ecx  <--EAX除以10
```

```asm
:004B0A80 83FA04                  cmp edx, 00000004  <--比较余数是否等于4
```

```asm
:004B0A83 7504                    jne 004B0A89  <--不等于4的话则跳转，一定不能跳转
```

```asm
:004B0A85 C645F701                mov [ebp-09], 01
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004B0A19(C), :004B0A39(C), :004B0A4E(C), :004B0A5C(C), :004B0A83(C)

|

```asm
:004B0A89 33C0                    xor eax, eax
```

```asm
:004B0A8B 5A                      pop edx
```

```asm
:004B0A8C 59                      pop ecx
```

```asm
:004B0A8D 59                      pop ecx
```

```asm
:004B0A8E 648910                  mov dword ptr fs:[eax], edx
```

```asm
:004B0A91 68A60A4B00              push 004B0AA6
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B0AA4(U)

|

```asm
:004B0A96 8D45F8                  lea eax, dword ptr [ebp-08]
```

```asm
:004B0A99 E81A31F5FF              call 00403BB8
```

```asm
:004B0A9E C3                      ret
```

现在我们知道了注册码的形式为`A1910xxxxx`或`A1423xxxxx`，其中第6、7、8个字符为任意字符，而第9、10个字符的ASCII码的和的个位数为4就可以正确的注册了！！
