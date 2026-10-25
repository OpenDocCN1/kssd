
# Golden 5.7 Build 391

> 原文：[`www.pediy.com/kssd/pediy05/pediy50512.htm`](https://www.pediy.com/kssd/pediy05/pediy50512.htm)

Golden 5.7 Build 391 破解手记--算法分析

作者：`newlaos[DFCG]`

软件名称：`Golden 5.7 Build 391` (编程工具)

整理日期：`2003.3.15`

最新版本：`5.7 Build 391`

文件大小：`2562KB`

软件授权：共享软件

使用平台：`Win9x/Me/NT/2000`

发布公司：`http://www.benthicsoftware.com`

软件简介：是一个 32 位多线程的应用程序，具有多种功能，类似于 `SQL`，包括变量提示和参数传递脚本显示等，能编写和运行程序，使用非常简单，速度快，界面好。

加密方式：注册码

功能限制：`30` 天试用

`PJ` 工具：`TRW20001.23` 注册版、`W32Dasm8.93` 黄金版，`FI2.5`

`PJ` 日期：`2003-03-27`

作者 `newlaos` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

### 1. 静态分析

1. 先用 `FI2.5` 看一下主文件 `Golden32.exe`，没有加壳，程序是用 `DELPHI` 编的。
2. 用 `W32Dasm8.93` 黄金版对 `Golden32.exe` 进行静态反汇编，再用串式数据参考，找到 `"Incorrect Registration Code"` (很经典的句子)，双击来到下面代码段。这样就找到注册码的计算部分。

### 2. 动态跟踪与断点

3. 再用 `TRW20001.23` 注册版进行动态跟踪，下断 `BPX 005171A8` (通常在注册成功与否的前面一些下断，这样，才能找到关键部分)，先输入假码 `78787878`。

```
.......
.......

* Possible StringData Ref from Code Obj ->"坃 H"
                                      |
:005171A8 A1044F5100                  mov eax, dword ptr [00514F04]
:005171AD E8D20BF8FF                  call 00497D84
:005171B2 8B1518186B00                mov edx, dword ptr [006B1818]
:005171B8 8902                        mov dword ptr [edx], eax
:005171BA 33C0                        xor eax, eax
:005171BC 55                          push ebp
:005171BD 687F725100                  push 0051727F
:005171C2 64FF30                      push dword ptr fs:[eax]
:005171C5 648920                      mov dword ptr fs:[eax], esp
:005171C8 A118186B00                  mov eax, dword ptr [006B1818]
:005171CD 8B00                        mov eax, dword ptr [eax]
:005171CF 8B10                        mov edx, dword ptr [eax]
:005171D1 FF92EC000000                call dword ptr [edx+000000EC]
:005171D7 48                          dec eax
:005171D8 0F8587000000                jne 00517265
:005171DE 8D55F8                      lea edx, dword ptr [ebp-08]
:005171E1 A118186B00                  mov eax, dword ptr [006B1818]
:005171E6 8B00                        mov eax, dword ptr [eax]
:005171E8 8B80FC020000                mov eax, dword ptr [eax+000002FC]
:005171EE E84979F6FF                  call 0047EB3C
:005171F3 8B4DF8                      mov ecx, dword ptr [ebp-08] <===ECX=78787878
:005171F6 8B9318030000                mov edx, dword ptr [ebx+00000318] <===EDX=Golden32
:005171FC 8D45FC                      lea eax, dword ptr [ebp-04]
:005171FF E89CDBEEFF                  call 00404DA0
:00517204 8B45FC                      mov eax, dword ptr [ebp-04] <===EAX=Golden3278787878(将它全起来了)，EDX=78787878
:00517207 E894F7FFFF                  call 005169A0      <===如果要正确，则这个 CALL 返回时，AL 不能为 0，F8 跟进
:0051720C 84C0                        test al, al         <===AL 不能为 0
:0051720E 743C                        je 0051724C         <===呵呵，只有一处跳向失败
:00517210 8D55F4                      lea edx, dword ptr [ebp-0C]
:00517213 A118186B00                  mov eax, dword ptr [006B1818]
:00517218 8B00                        mov eax, dword ptr [eax]
:0051721A 8B80FC020000                mov eax, dword ptr [eax+000002FC]
:00517220 E81779F6FF                  call 0047EB3C
:00517225 8B55F4                      mov edx, dword ptr [ebp-0C]
:00517228 8D831C030000                lea eax, dword ptr [ebx+0000031C]
:0051722E E889D8EEFF                  call 00404ABC
:00517233 8B8304030000                mov eax, dword ptr [ebx+00000304]
:00517239 B201                        mov dl, 01
:0051723B 8B08                        mov ecx, dword ptr [eax]
:0051723D FF5164                      call [ecx+64]
:00517240 C7834C02000001000000        mov dword ptr [ebx+0000024C], 00000001
:0051724A EB19                        jmp 00517265

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0051720E(C)
|
:0051724C 6A00                        push 00000000

* Possible StringData Ref from Code Obj ->"Benthic Software"
                                      |
:0051724E 68B8725100                  push 005172B8

* Possible StringData Ref from Code Obj ->"Incorrect Registration Code"
                                      |
:00517253 68CC725100                  push 005172CC    <===错误的注册码
:00517258 8BC3                        mov eax, ebx
:0051725A E811E2F6FF                  call 00485470
:0051725F 50                          push eax

* Reference To: user32.MessageBoxA, Ord:0000h
                                      |
:00517260 E81F0DEFFF                  Call 00407F84

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:005171D8(C), :0051724A(U)
|
:00517265 33C0                        xor eax, eax
:00517267 5A                          pop edx
:00517268 59                          pop ecx
:00517269 59                          pop ecx
:0051726A 648910                      mov dword ptr fs:[eax], edx
:0051726D 6886725100                  push 00517286

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00517284(U)
|
:00517272 A118186B00                  mov eax, dword ptr [006B1818]
:00517277 8B00                        mov eax, dword ptr [eax]
:00517279 E89AC9EEFF                  call 00403C18
:0051727E C3                          ret

.......
.......
```

### 3. 关键算法分析

`00517207 call 005169A0` -----关键的算法 CALL，F8 跟进来到下列代码段-------------------------------

要求：如果要正确注册，则返回时 `AL` 不能为 0

初始值：`EAX=Golden3278787878`，`EDX=78787878`

```
:005169A0 55                          push ebp
:005169A1 8BEC                        mov ebp, esp
:005169A3 33C9                        xor ecx, ecx
:005169A5 51                          push ecx
:005169A6 51                          push ecx
:005169A7 51                          push ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00516941(C)
|
:005169A8 51                          push ecx
:005169A9 51                          push ecx
:005169AA 51                          push ecx
:005169AB 53                          push ebx
:005169AC 56                          push esi
:005169AD 8945FC                      mov dword ptr [ebp-04], eax
:005169B0 8B45FC                      mov eax, dword ptr [ebp-04]
:005169B3 E88CE5EEFF                  call 00404F44
:005169B8 33C0                        xor eax, eax
:005169BA 55                          push ebp
:005169BB 68C56A5100                  push 00516AC5
:005169C0 64FF30                      push dword ptr fs:[eax]
:005169C3 648920                      mov dword ptr fs:[eax], esp
:005169C6 C645FB00                    mov [ebp-05], 00
:005169CA 8D55EC                      lea edx, dword ptr [ebp-14]
:005169CD 8B45FC                      mov eax, dword ptr [ebp-04]
:005169D0 E82F2CEFFF                  call 00409604
:005169D5 8B55EC                      mov edx, dword ptr [ebp-14]
:005169D8 8D45FC                      lea eax, dword ptr [ebp-04]
:005169DB E820E1EEFF                  call 00404B00
:005169E0 8D45F0                      lea eax, dword ptr [ebp-10]
:005169E3 E880E0EEFF                  call 00404A68
:005169E8 8B45FC                      mov eax, dword ptr [ebp-04]
:005169EB E864E3EEFF                  call 00404D54        <===这个 CALL 算出 Golden3278787878 的长度 10，放入 EAX
:005169F0 8BD8                        mov ebx, eax
:005169F2 EB01                        jmp 005169F5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00516A04(C)
|
:005169F4 4B                          dec ebx               <===计数器 EBX=EBX-1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005169F2(U)
|
:005169F5 8B45FC                      mov eax, dword ptr [ebp-04]
:005169F8 8A4418FF                    mov al, byte ptr [eax+ebx-01]
:005169FC 04D0                        add al, D0
:005169FE 2C0A                        sub al, 0A
:00516A00 7304                        jnb 00516A06
:00516A02 85DB                        test ebx, ebx
:00516A04 7FEE                        jg 005169F4  <===这里构成一个小循环，主要功能是从尾部开始，找只要一遇到字符就跳出，用在后面提取字串，进行运算，我们这里，是 GOLDEN 字串了
```



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
