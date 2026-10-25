
### ====>不能跳！

```
*
Possible StringData Ref from Data Obj -> "jetdown"
```

### ====>黑名单！

```
         |
:0040E5BF 68241A4600       push 00461A24
:0040E5C4 8D8C24F0000000     lea ecx, dword ptr [esp+000000F0]
:0040E5CB E802440200             call 004329D2
:0040E5D0 83F8FF                 cmp eax, FFFFFFFF
:0040E5D3 7406                    je 0040E5DB
```

### ====>应跳！

```
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040E591(C),
:0040E5A7(C), :0040E5BD(C)
|
:0040E5D5 899ED0010000     mov dword ptr [esi+000001D0], ebx
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E5D3(C)
|
:0040E5DB 55                      push ebp
:0040E5DC 8BAC24F0000000          mov ebp, dword ptr [esp+000000F0]
用户名送eax
:0040E5E3 B165                    mov cl, 65
:0040E5E5 B061                    mov al, 61
:0040E5E7 8B75F8                  mov esi, dword ptr [ebp-08]
   ====>用户名长度送esi
                   ====>?
ESI=3
```

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

===>下面这段代码是从[esp+10]依次处放入“searmake”字符串

:0040E5EA 884C240D                mov byte ptr [esp+0D], cl
:0040E5EE 884C2413                mov byte ptr [esp+13], cl
:0040E5F2 33C9       xor ecx, ecx
:0040E5F4 3BF3             cmp esi, ebx
:0040E5F6 C644240C73             mov [esp+0C], 73
:0040E5FB 8844240E                mov byte ptr [esp+0E], al
:0040E5FF C644240F72              mov [esp+0F], 72
:0040E604 C64424106D mov [esp+10], 6D
:0040E609 88442411     mov byte ptr [esp+11], al
:0040E60D C64424126B         mov [esp+12], 6B
:0040E612 885C2414             mov byte ptr [esp+14], bl
:0040E616 7E3D                    jle 0040E655
:0040E618 57     push edi
:0040E619 8D7C341B       lea edi, dword ptr [esp+esi+1B]
```

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E650(C)
         ====>以下就是运算核心了！

:0040E61D 8A0429                  mov al, byte ptr [ecx+ebp]
               ====>依次取用户名。
       ====>1、？AL=66 即f的HEX值
   ====>2、？AL=6C 即l的HEX值
                     ====>3、？AL=79 即y的HEX值
:0040E620 8BD1                   mov edx, ecx
:0040E622 81E207000080            and edx, 80000007
:0040E628 7905   jns 0040E62F
:0040E62A 4A         dec edx
:0040E62B 83CAF8           or edx, FFFFFFF8
:0040E62E 42                   inc edx
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E628(C)
|
:0040E62F 0FBE541410              movsx edx, byte ptr [esp+edx+10]
               ====>依次从“searmake”字符串中取字符入EDX
         ====>1、？EDX=73 即s的HEX值
       ====>2、？EDX=65 即e的HEX值
   ====>3、？EDX=61 即a的HEX值
:0040E634 0FBEC0             movsx eax, al
     ====>1、？EAX=66 即f的HEX值
   ====>2、？EAX=6C 即l的HEX值
                       ====>3、？EAX=79 即y的HEX值
:0040E637 8BD9                   mov ebx, ecx
             ====>1、？EBX=0
     ====>2、？EBX=1
                     ====>3、？EBX=2
:0040E639 03DA                    add ebx, edx
             ====>1、EBX=0+73=73
       ====>2、EBX=1+65=66
 ====>3、EBX=2+61=63
:0040E63B 03C3                    add eax, ebx
             ====>1、EAX=66+73=D9
       ====>2、EAX=6C+66=D2
 ====>3、EAX=79+63=DC
:0040E63D BB09000000              mov ebx, 00000009
         ====>9送ebx
:0040E642 03C6                    add eax, esi
             ====>esi是用户名长度
     ====>1、EAX=D9+3=DC
                       ====>2、EAX=D2+3=D5
           ====>3、EAX=DC+3=DF
:0040E644 99                      cdq
:0040E645 F7FB                    idiv ebx
             ====>EAX依次除以9
     ====>1、EAX=DC/9=18余4
                       ====>2、EAX=D5/9=17余6
           ====>3、EAX=DF/9=18余7
:0040E647 80C230                  add dl, 30
         ====>余数入DL，依次加30
         ====>1、DL=4+30=34
   ====>2、DL=6+30=36
                     ====>3、DL=7+30=37
:0040E64A 41                     inc ecx
             ====>ecx依次增1
:0040E64B 8817                    mov byte ptr [edi], dl
                 ====>DL->[edi]
           ====>循环3次后，D EDI=764
     ****这是真码的前3个数！！！
:0040E64D 4F                      dec edi
:0040E64E 3BCE   cmp ecx, esi
                   ====>比较用户名是否取完
:0040E650 7CCB                 jl 0040E61D
         ====>没有取完，跳上去继续循环
   ====>共循环3次。
:0040E652 33DB                    xor ebx, ebx
:0040E654 5F     pop edi
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E616(C)
|
:0040E655 8D4668                  lea eax, dword ptr [esi+68]
                 ====>？ESI=3
 实际上是用户名长度加上68的结果送eax
                   ====>过此
？EAX=6B
:0040E658 B909000000             mov ecx, 00000009
     ====>9送ecx
:0040E65D 99                      cdq
:0040E65E F7F9                    idiv ecx
             ====>EAX/9=B余8
     ====>余数8入DL
:0040E660 8B8424F4000000          mov eax, dword ptr [esp+000000F4]
         ====>用户输入的假注册码送eax
:0040E667 5D                      pop ebp
:0040E668 80C230                 add dl, 30
               ====>DL=8+30=38
         ****这是真码的最后1个数！！！
:0040E66B 88543414                mov byte ptr [esp+esi+14], dl
           ====>DL移入[esp+17]处
:0040E66F 885C3415                mov byte ptr [esp+esi+15], bl
:0040E673 8D742414                lea esi, dword ptr [esp+14]
                 ====>真正的注册码送ESI
```

***

至此算法分析完毕。呵呵，追注册码挺快。

但是分析、整理成这篇心得却花了我半天的工夫！

希望能够给如我般的初学者一点用处！！!

***

```
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E695(C)
         这里向下是将真假注册码逐位的进行比较，一个经典的组合！

:0040E677 8A10                    mov dl, byte ptr [eax]
                 ====>D EAX=试炼码
:0040E679 8ACA                    mov cl, dl
:0040E67B 3A16   cmp dl, byte ptr [esi]
                     ====>D ESI=真码！！！！
:0040E67D 751C                    jne 0040E69B
:0040E67F 3ACB   cmp cl, bl
:0040E681 7414       je 0040E697
:0040E683 8A5001           mov dl, byte ptr [eax+01]
:0040E686 8ACA                    mov cl, dl
:0040E688 3A5601 cmp dl, byte ptr [esi+01]
:0040E68B 750E           jne 0040E69B
:0040E68D 83C002               add eax, 00000002
:0040E690 83C602                  add esi, 00000002
:0040E693 3ACB   cmp cl, bl
:0040E695 75E0       jne 0040E677
```

---

## 【KeyMake之内存注册机】：

- 中断地址：`40E67B`
- 中断次数：`1`
- 第一字节：`3A`
- 指令长度：`2`
- 内存方式：`ESI`

---

## 【注册信息保存】：

```
[HKEY_CURRENT_USER\Software\SeaMoonTech\SEARCHMAKER\RegInfo]
"RegUserName"="fly"
"RegCode"="7648"
```

---

## 【整理】：

```
Registartion Name：fly
Registartion Code：7648
```

---

Cracked By 巢水工作坊――fly【OCN】

2003-1-24  17:00
