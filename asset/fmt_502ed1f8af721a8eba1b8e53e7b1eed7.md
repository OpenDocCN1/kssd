
#### 4. 算法分析
- **类型**：数学计算

1.  先将软件内定的 `GOLDEN32` 和输入的注册码合成一个字符串，假设合为 `GOLDEN32XX99999`（`X` 代表字符，`9` 代表数字）
2.  从 `GOLDEN32XX99999` 字符串的尾部开始，到遇到第一个字符停止，分为两串，即 `GOLDEN32XX` 和 `99999`
3.  对第一部分做如下计算：
    -   例：取到第 `n` 位值，则将这个值的 `ASC` 码值乘以 `n`，再加上 `n*B`，得出一个数值
    -   最后将这些数值相加起来，得到一个十六进制的总数
4.  将这个十六进制的总数，转为十进制后，必须和第二部分，相等

所以，这里输入的注册码不能完全是数字，否则只有 `GOLDEN`，经过计算到十进制后是 `1773`，是无法和 `3299999`（`99999` 为输入的纯数字注册码）相等的。  
这里提供一个注册码就是：`NEWLAOS10076`

#### 5. 注册信息存放位置
注册信息存放在注册表中（删掉此键值就成未注册版本）：

`[HKEY_CURRENT_USER\Software\Benthic\Golden32\Login]`

`"RVal"="C327589CDA2D49A8EA2968AAD4"`  
`4dd466d0e1e0cc123afbe504da2462ca`

### 汇编代码分析

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A00(C)  
|  
:00516A06 8D45F0                 lea eax, dword ptr [ebp-10]  
:00516A09 50                     push eax  
:00516A0A 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A0D E842E3EEFF             call 00404D54  
:00516A12 8BC8                   mov ecx, eax  
:00516A14 2BCB                   sub ecx, ebx  
:00516A16 8D5301                 lea edx, dword ptr [ebx+01]  
:00516A19 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A1C E893E5EEFF             call 00404FB4  
:00516A21 8D45F4                 lea eax, dword ptr [ebp-0C]  
:00516A24 50                     push eax  
:00516A25 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A28 E827E3EEFF             call 00404D54  
:00516A2D 50                     push eax  
:00516A2E 8B45F0                 mov eax, dword ptr [ebp-10]  
:00516A31 E81EE3EEFF             call 00404D54  
:00516A36 59                     pop ecx  
:00516A37 2BC8                   sub ecx, eax  
:00516A39 BA01000000             mov edx, 00000001  
:00516A3E 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A41 E86EE5EEFF             call 00404FB4  
:00516A46 33F6                   xor esi, esi  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A6A(C)  
|  
:00516A6C BB01000000             mov ebx, 00000001   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A85(C)  
|  
:00516A71 8B55F4                 mov edx, dword ptr [ebp-0C]  
:00516A74 0FB6541AFF             movzx edx, byte ptr [edx+ebx-01]<===依次提取 GOLDEN 字串中每个字符的 ASC 码  
:00516A79 0FAFD3                 imul edx, ebx  
:00516A7C 6BCB0B                 imul ecx, ebx, 0000000B  
:00516A7F 03D1                   add edx, ecx  
:00516A81 03F2                   add esi, edx  
:00516A83 43                     inc ebx               
:00516A84 48                     dec eax            <===计数器 EAX 初始值为 6，也就是 GOLDEN 字串长度，           
:00516A85 75EA                   jne 00516A71      
   
   <===向上构成一个小循环，ESI=0   +47*1+1*B=52  
                      ESI=52  +4F*2+2*B=106  
                      ESI=106 +4C*3+3*B=20B  
                      ESI=20B +44*4+4*B=347  
                      ESI=347 +45*5+5*B=4D7  
                      ESI=4D7 +4E*6+6*B=6ED  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A6A(C)  
|  
:00516A87 8D55E8                 lea edx, dword ptr [ebp-18]  
:00516A8A 8BC6                   mov eax, esi      <===EAX=6ED  
:00516A8C E8F732EFFF             call 00409D88      
                <===EAX=97F380，位置上放了一个地址指针，指向 1773(正好是 6ED 的十进制表示形式)  
:00516A91 8B45E8                 mov eax, dword ptr [ebp-18] <===EAX=1773  
:00516A94 8B55F0                 mov edx, dword ptr [ebp-10] <===EDX=3278787878  
:00516A97 E804E4EEFF             call 00404EA0     <===关键的 CALL，如果 EAX 和 EDX 相等，这里就对了。如何才能相等请看下面的算法分析  
:00516A9C 7504                   jne 00516AA2      <===如果输入有误，就从这里跳走，也就错了。因为下一行一定要经过  
:00516A9E C645FB01               mov [ebp-05], 01 <===这一行是关键的标志位的赋值，一定要经过。  

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00516A52(C), :00516A5E(C), :00516A9C(C)  
|  
:00516AA2 33C0                   xor eax, eax  
:00516AA4 5A                     pop edx  
:00516AA5 59                     pop ecx  
:00516AA6 59                     pop ecx  
:00516AA7 648910                 mov dword ptr fs:[eax], edx  
:00516AAA 68CC6A5100             push 00516ACC  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516ACA(U)  
|  
:00516AAF 8D45E8                 lea eax, dword ptr [ebp-18]  
:00516AB2 BA04000000             mov edx, 00000004  
:00516AB7 E8D0DFEEFF             call 00404A8C  
:00516ABC 8D45FC                 lea eax, dword ptr [ebp-04]  
:00516ABF E8A4DFEEFF             call 00404A68  
:00516AC4 C3                     ret  

:00516AC5 E9E2D8EEFF             jmp 004043AC  
:00516ACA EBE3                   jmp 00516AAF  
:00516ACC 8A45FB                 mov al, byte ptr [ebp-05]  <===[ebp-05]位置上的值太关键了，向上看  
:00516ACF 5E                     pop esi  
:00516AD0 5B                     pop ebx  
:00516AD1 8BE5                   mov esp, ebp  
:00516AD3 5D                     pop ebp  
:00516AD4 C3                     ret  
```

* 注：代码块中包含了一些非标准汇编标记（如 `* Referenced by...`）和注释，已保留原文语义。

### 额外汇编片段

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A48 8B45F4                mov eax, dword ptr [ebp-0C]  
:00516A4B E804E3EEFF             call 00404D54<===算长度的 CALL  
:00516A50 85C0                   test eax, eax  
:00516A52 7E4E                   jle 00516AA2  
:00516A54 8B45F4                 mov eax, dword ptr [ebp-0C]  
:00516A57 E8F8E2EEFF             call 00404D54  
:00516A5C 85C0                   test eax, eax  
:00516A5E 7E42                   jle 00516AA2  
:00516A60 8B45F4                 mov eax, dword ptr [ebp-0C]  
:00516A63 E8ECE2EEFF             call 00404D54  
:00516A68 85C0                   test eax, eax  
:00516A6A 7E1B                   jle 00516A87  

* 下面这段是花指令

:00516A6C BB01000000             mov ebx, 00000001   
```

注意：原文中 `* Referenced by...` 和花指令部分在逻辑上穿插在主代码流中，为了保持代码块的可读性，我将整个汇编部分整理为一个大的代码块，并在代码块内部保留了原有的结构和注释。但为了严格遵循“代码块前后加上三个反引号”且保持逻辑清晰，我将汇编代码作为一个整体处理。上面的“额外汇编片段”其实是主代码流的一部分，为了准确还原，我将重新整合代码块。

修正后的代码块如下：

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A00(C)  
|  
:00516A06 8D45F0                 lea eax, dword ptr [ebp-10]  
:00516A09 50                     push eax  
:00516A0A 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A0D E842E3EEFF             call 00404D54  
:00516A12 8BC8                   mov ecx, eax  
:00516A14 2BCB                   sub ecx, ebx  
:00516A16 8D5301                 lea edx, dword ptr [ebx+01]  
:00516A19 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A1C E893E5EEFF             call 00404FB4  
:00516A21 8D45F4                 lea eax, dword ptr [ebp-0C]  
:00516A24 50                     push eax  
:00516A25 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A28 E827E3EEFF             call 00404D54  
:00516A2D 50                     push eax  
:00516A2E 8B45F0                 mov eax, dword ptr [ebp-10]  
:00516A31 E81EE3EEFF             call 00404D54  
:00516A36 59                     pop ecx  
:00516A37 2BC8                   sub ecx, eax  
:00516A39 BA01000000             mov edx, 00000001  
:00516A3E 8B45FC                 mov eax, dword ptr [ebp-04]  
:00516A41 E86EE5EEFF             call 00404FB4  
:00516A46 33F6                   xor esi, esi  

***下面这段是花指令***  
:00516A48 8B45F4                 mov eax, dword ptr [ebp-0C]  
:00516A4B E804E3EEFF             call 00404D54<===算长度的 CALL  
:00516A50 85C0                   test eax, eax  
:00516A52 7E4E                   jle 00516AA2  
:00516A54 8B45F4                 mov eax, dword ptr [ebp-0C]  
:00516A57 E8F8E2EEFF             call 00404D54  
:00516A5C 85C0                   test eax, eax  
:00516A5E 7E42                   jle 00516AA2  
:00516A60 8B45F4                 mov eax, dword ptr [ebp-0C]  
:00516A63 E8ECE2EEFF             call 00404D54  
:00516A68 85C0                   test eax, eax  
:00516A6A 7E1B                   jle 00516A87  
*****************

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A85(C)  
|  
:00516A6C BB01000000             mov ebx, 00000001   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A71 8B55F4                 mov edx, dword ptr [ebp-0C]  
:00516A74 0FB6541AFF             movzx edx, byte ptr [edx+ebx-01]<===依次提取 GOLDEN 字串中每个字符的 ASC 码  
:00516A79 0FAFD3                 imul edx, ebx  
:00516A7C 6BCB0B                 imul ecx, ebx, 0000000B  
:00516A7F 03D1                   add edx, ecx  
:00516A81 03F2                   add esi, edx  
:00516A83 43                     inc ebx               
:00516A84 48                     dec eax            <===计数器 EAX 初始值为 6，也就是 GOLDEN 字串长度，           
:00516A85 75EA                   jne 00516A71      
   
   <===向上构成一个小循环，ESI=0   +47*1+1*B=52  
                      ESI=52  +4F*2+2*B=106  
                      ESI=106 +4C*3+3*B=20B  
                      ESI=20B +44*4+4*B=347  
                      ESI=347 +45*5+5*B=4D7  
                      ESI=4D7 +4E*6+6*B=6ED

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516A6A(C)  
|  
:00516A87 8D55E8                 lea edx, dword ptr [ebp-18]  
:00516A8A 8BC6                   mov eax, esi      <===EAX=6ED  
:00516A8C E8F732EFFF             call 00409D88      
                <===EAX=97F380，位置上放了一个地址指针，指向 1773(正好是 6ED 的十进制表示形式)  
:00516A91 8B45E8                 mov eax, dword ptr [ebp-18] <===EAX=1773  
:00516A94 8B55F0                 mov edx, dword ptr [ebp-10] <===EDX=3278787878  
:00516A97 E804E4EEFF             call 00404EA0     <===关键的 CALL，如果 EAX 和 EDX 相等，这里就对了。如何才能相等请看下面的算法分析  
:00516A9C 7504                   jne 00516AA2      <===如果输入有误，就从这里跳走，也就错了。因为下一行一定要经过  
:00516A9E C645FB01               mov [ebp-05], 01 <===这一行是关键的标志位的赋值，一定要经过。  

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00516A52(C), :00516A5E(C), :00516A9C(C)  
|  
:00516AA2 33C0                   xor eax, eax  
:00516AA4 5A                     pop edx  
:00516AA5 59                     pop ecx  
:00516AA6 59                     pop ecx  
:00516AA7 648910                 mov dword ptr fs:[eax], edx  
:00516AAA 68CC6A5100             push 00516ACC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00516ACA(U)  
|  
:00516AAF 8D45E8                 lea eax, dword ptr [ebp-18]  
:00516AB2 BA04000000             mov edx, 00000004  
:00516AB7 E8D0DFEEFF             call 00404A8C  
:00516ABC 8D45FC                 lea eax, dword ptr [ebp-04]  
:00516ABF E8A4DFEEFF             call 00404A68  
:00516AC4 C3                     ret  

:00516AC5 E9E2D8EEFF             jmp 004043AC  
:00516ACA EBE3                   jmp 00516AAF  
:00516ACC 8A45FB                 mov al, byte ptr [ebp-05]  <===[ebp-05]位置上的值太关键了，向上看  
:00516ACF 5E                     pop esi  
:00516AD0 5B                     pop ebx  
:00516AD1 8BE5                   mov esp, ebp  
:00516AD3 5D                     pop ebp  
:00516AD4 C3                     ret
```

### 4. 算法分析
- **类型**：数学计算

1.  先将软件内定的 `GOLDEN32` 和输入的注册码合成一个字符串，假设合为 `GOLDEN32XX99999`（`X` 代表字符，`9` 代表数字）
2.  从 `GOLDEN32XX99999` 字符串的尾部开始，到遇到第一个字符停止，分为两串，即 `GOLDEN32XX` 和 `99999`
3.  对第一部分做如下计算：
    -   例：取到第 `n` 位值，则将这个值的 `ASC` 码值乘以 `n`，再加上 `n*B`，得出一个数值
    -   最后将这些数值相加起来，得到一个十六进制的总数
4.  将这个十六进制的总数，转为十进制后，必须和第二部分，相等

所以，这里输入的注册码不能完全是数字，否则只有 `GOLDEN`，经过计算到十进制后是 `1773`，是无法和 `3299999`（`99999` 为输入的纯数字注册码）相等的。  
这里提供一个注册码就是：`NEWLAOS10076`

### 5. 注册信息存放位置
注册信息存放在注册表中（删掉此键值就成未注册版本）：

`[HKEY_CURRENT_USER\Software\Benthic\Golden32\Login]`

`"RVal"="C327589CDA2D49A8EA2968AAD4"`  
`4dd466d0e1e0cc123afbe504da2462ca`
