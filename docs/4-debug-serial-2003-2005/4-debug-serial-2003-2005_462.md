
# 起名正宗 1.32

> 原文：[`www.pediy.com/kssd/pediy06/pediy6323.htm`](https://www.pediy.com/kssd/pediy06/pediy6323.htm)

起名正宗 1.32

平原的软件

`Delphi` 的作品，`ASP` 1.00b 加壳，`pe-scan` 可以脱壳

`win98` 下破解

破解的时候用的是 1.31，后来发现他的最新版本 1.32 也是一样的注册

但是 1.32 使用的时候有非法的提示，不知道是不是还有什么鬼

根据对此软件的使用，输入注册名和密码，输入不正确时出现对话框“注册码错误”

用 `WinDasm` 反汇编，查找字符串参考，得到如下部分

* Possible StringData Ref from Code Obj -> "起名正宗"

|

:004C5F42 BA14604C00 mov edx, 004C6014
:004C5F47 E8F4ECF3FF call 00404C40
:004C5F4C 8B45EC mov eax, dword ptr [ebp-14]
:004C5F4F 50 push eax
:004C5F50 8B8B34030000 mov ecx, dword ptr [ebx+00000334]
:004C5F56 8B9338030000 mov edx, dword ptr [ebx+00000338]
:004C5F5C 8B832C030000 mov eax, dword ptr [ebx+0000032C]
:004C5F62 E841F7FFFF call 004C56A8------------>看看下面的过程初步判断此处为关键 CALL
:004C5F67 84C0 test al, al
:004C5F69 7420 je 004C5F8B
:004C5F6B 6A00 push 00000000
:004C5F6D 668B0DF05F4C00 mov cx, word ptr [004C5FF0]
:004C5F74 B202 mov dl, 02

* Possible StringData Ref from Code Obj -> "您已经注册成功！"

|

:004C5F76 B828604C00 mov eax, 004C6028
:004C5F7B E868BDF7FF call 00441CE8
:004C5F80 B201 mov dl, 01
:004C5F82 8BC3 mov eax, ebx
:004C5F84 E8CFFCFFFF call 004C5C58
:004C5F89 EB15 jmp 004C5FA0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C5F69(C)

|

:004C5F8B 6A00 push 00000000
:004C5F8D 668B0DF05F4C00 mov cx, word ptr [004C5FF0]
:004C5F94 33D2 xor edx, edx

* Possible StringData Ref from Code Obj -> "注册码错误！"

|

:004C5F96 B844604C00 mov eax, 004C6044
:004C5F9B E848BDF7FF call 00441CE8

************************************************************************************************************************************

************************************************************************************************************************************

************************************************************************************************************************************

* Referenced by a CALL at Address:

|:004C5F62

|

:004C56A8 55 push ebp\------------------------------------->在 V1.32 下此处对应着 004C5094
:004C56A9 8BEC mov ebp, esp
:004C56AB 81C460FDFFFF add esp, FFFFFD60
:004C56B1 53 push ebx
:004C56B2 56 push esi
:004C56B3 57 push edi
:004C56B4 33DB xor ebx, ebx
:004C56B6 899D60FEFFFF mov dword ptr [ebp+FFFFFE60], ebx
:004C56BC 895DF8 mov dword ptr [ebp-08], ebx
:004C56BF 894DFC mov dword ptr [ebp-04], ecx
:004C56C2 8BDA mov ebx, edx
:004C56C4 8BF8 mov edi, eax
:004C56C6 8B4508 mov eax, dword ptr [ebp+08]
:004C56C9 E80EF7F3FF call 00404DDC
:004C56CE 33C0 xor eax, eax
:004C56D0 55 push ebp
:004C56D1 6851584C00 push 004C5851
:004C56D6 64FF30 push dword ptr fs:[eax]
:004C56D9 648920 mov dword ptr fs:[eax], esp
:004C56DC 8D9560FEFFFF lea edx, dword ptr [ebp+FFFFFE60]
:004C56E2 8B4508 mov eax, dword ptr [ebp+08]
:004C56E5 E8C638F4FF call 00408FB0
:004C56EA 83BD60FEFFFF00 cmp dword ptr [ebp+FFFFFE60], 00000000
:004C56F1 7507 jne 004C56FA------------------------------>JMP
:004C56F3 33DB xor ebx, ebx
:004C56F5 E92E010000 jmp 004C5828

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C56F1(C)

|

:004C56FA 8B4D08 mov ecx, dword ptr [ebp+08]
:004C56FD 8BD3 mov edx, ebx
:004C56FF 8BC7 mov eax, edi
:004C5701 E872FAFFFF call 004C5178------------------------------>生成真注册码的过程----->V1.32 下为 004C50ED
:004C5706 8BF0 mov esi, eax\------------------------------->真注册码 C2ABDB27
:004C5708 3B75FC cmp esi, dword ptr [ebp-04]---------------->真假注册码比较
:004C570B 0F8515010000 jne 004C5826
:004C5711 E8EA58F4FF call 0040B000
:004C5716 DD5DB0 fstp qword ptr [ebp-50]
:004C5719 9B wait
:004C571A E8E158F4FF call 0040B000
:004C571F DD5DB8 fstp qword ptr [ebp-48]
:004C5722 9B wait
:004C5723 C645C001 mov [ebp-40], 01
:004C5727 C645C100 mov [ebp-3F], 00
:004C572B 895DC4 mov dword ptr [ebp-3C], ebx
:004C572E 8D8560FDFFFF lea eax, dword ptr [ebp+FFFFFD60]
:004C5734 8B5508 mov edx, dword ptr [ebp+08]
:004C5737 B9FF000000 mov ecx, 000000FF
:004C573C E88FF4F3FF call 00404BD0
:004C5741 8D9560FDFFFF lea edx, dword ptr [ebp+FFFFFD60]
:004C5747 8D45C8 lea eax, dword ptr [ebp-38]
:004C574A B114 mov cl, 14
:004C574C E893D8F3FF call 00402FE4
:004C5751 8975E0 mov dword ptr [ebp-20], esi
:004C5754 8D8560FDFFFF lea eax, dword ptr [ebp+FFFFFD60]
:004C575A 8B574C mov edx, dword ptr [edi+4C]
:004C575D B9FF000000 mov ecx, 000000FF
:004C5762 E869F4F3FF call 00404BD0
:004C5767 8D9560FDFFFF lea edx, dword ptr [ebp+FFFFFD60]
:004C576D 8D45E4 lea eax, dword ptr [ebp-1C]
:004C5770 B10A mov cl, 0A
:004C5772 E86DD8F3FF call 00402FE4
:004C5777 B301 mov bl, 01
:004C5779 8D4DF8 lea ecx, dword ptr [ebp-08]
:004C577C 8B5748 mov edx, dword ptr [edi+48]
:004C577F 8BC7 mov eax, edi
:004C5781 E866FBFFFF call 004C52EC
:004C5786 8B45F8 mov eax, dword ptr [ebp-08]
:004C5789 E8DA3DF4FF call 00409568
:004C578E 84C0 test al, al
:004C5790 0F8492000000 je 004C5828
:004C5796 8B45F8 mov eax, dword ptr [ebp-08]
:004C5799 E8DA3DF4FF call 00409578
:004C579E 8BF0 mov esi, eax
:004C57A0 8BFE mov edi, esi
:004C57A2 83E701 and edi, 00000001
:004C57A5 83FF01 cmp edi, 00000001
:004C57A8 750B jne 004C57B5
:004C57AA 8BD6 mov edx, esi
:004C57AC 4A dec edx
:004C57AD 8B45F8 mov eax, dword ptr [ebp-08]
:004C57B0 E8D73DF4FF call 0040958C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C57A8(C)

|

:004C57B5 8B55F8 mov edx, dword ptr [ebp-08]
:004C57B8 8D8564FEFFFF lea eax, dword ptr [ebp+FFFFFE64]
:004C57BE E8E9D5F3FF call 00402DAC
:004C57C3 BA40000000 mov edx, 00000040
:004C57C8 8D8564FEFFFF lea eax, dword ptr [ebp+FFFFFE64]
:004C57CE E81DDBF3FF call 004032F0
:004C57D3 E840D1F3FF call 00402918
:004C57D8 33D2 xor edx, edx



```
:004C57DA 8D8564FEFFFF            lea eax, dword ptr [ebp+FFFFFE64]

:004C57E0 E827DBF3FF              call 0040330C

:004C57E5 E82ED1F3FF              call 00402918

:004C57EA 8D45F4                  lea eax, dword ptr [ebp-0C]

:004C57ED 50                      push eax

:004C57EE 8D55B0                  lea edx, dword ptr [ebp-50]

:004C57F1 B901000000              mov ecx, 00000001

:004C57F6 8D8564FEFFFF            lea eax, dword ptr [ebp+FFFFFE64]

:004C57FC E82FD7F3FF              call 00402F30

:004C5801 E812D1F3FF              call 00402918

:004C5806 8D8564FEFFFF            lea eax, dword ptr [ebp+FFFFFE64]

:004C580C E83FD7F3FF              call 00402F50

:004C5811 E802D1F3FF              call 00402918

:004C5816 4F                      dec edi

:004C5817 740F                    je 004C5828

:004C5819 8D5601                  lea edx, dword ptr [esi+01]

:004C581C 8B45F8                  mov eax, dword ptr [ebp-08]

:004C581F E8683DF4FF              call 0040958C

:004C5824 EB02                    jmp 004C5828

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C570B(C)

|

:004C5826 33DB                    xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004C56F5(U), :004C5790(C), :004C5817(C), :004C5824(U)

|

:004C5828 33C0                    xor eax, eax

:004C582A 5A                      pop edx

:004C582B 59                      pop ecx

:004C582C 59                      pop ecx

:004C582D 648910                  mov dword ptr fs:[eax], edx

:004C5830 6858584C00              push 004C5858

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C5856(U)

|

:004C5835 8D8560FEFFFF            lea eax, dword ptr [ebp+FFFFFE60]

:004C583B E8FCF0F3FF              call 0040493C

:004C5840 8D45F8                  lea eax, dword ptr [ebp-08]

:004C5843 E8F4F0F3FF              call 0040493C

:004C5848 8D4508                  lea eax, dword ptr [ebp+08]

:004C584B E8ECF0F3FF              call 0040493C

:004C5850 C3                      ret

:004C5851 E90EEBF3FF              jmp 00404364

:004C5856 EBDD                    jmp 004C5835

:004C5858 8BC3                    mov eax, ebx

:004C585A 5F                      pop edi

:004C585B 5E                      pop esi

:004C585C 5B                      pop ebx

:004C585D 8BE5                    mov esp, ebp

:004C585F 5D                      pop ebp

:004C5860 C20400                  ret 0004

******************************************************************************

******************************************************************************

******************************************************************************

以下是 V1.32 版本的

* Referenced by a CALL at Address:

|:004C50ED

|

:004C4B70 55                      push ebp

:004C4B71 8BEC                    mov ebp, esp

:004C4B73 83C4F4                  add esp, FFFFFFF4

:004C4B76 53                      push ebx

:004C4B77 56                      push esi

:004C4B78 57                      push edi

:004C4B79 894DF8                  mov dword ptr [ebp-08], ecx

:004C4B7C 8955FC                  mov dword ptr [ebp-04], edx

:004C4B7F 8B45F8                  mov eax, dword ptr [ebp-08]

:004C4B82 E87502F4FF              call 00404DFC

:004C4B87 33C0                    xor eax, eax

:004C4B89 55                      push ebp

:004C4B8A 68314C4C00              push 004C4C31

:004C4B8F 64FF30                  push dword ptr fs:[eax]

:004C4B92 648920                  mov dword ptr fs:[eax], esp

:004C4B95 8B45F8                  mov eax, dword ptr [ebp-08]

:004C4B98 E87700F4FF              call 00404C14

:004C4B9D 85C0                    test eax, eax

:004C4B9F 7504                    jne 004C4BA5

:004C4BA1 33DB                    xor ebx, ebx

:004C4BA3 EB76                    jmp 004C4C1B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4B9F(C)

|

:004C4BA5 33F6                    xor esi, esi

:004C4BA7 85C0                    test eax, eax

:004C4BA9 7E13                    jle 004C4BBE

:004C4BAB BA01000000              mov edx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4BBC(C)

|

:004C4BB0 8B4DF8                  mov ecx, dword ptr [ebp-08]

:004C4BB3 0FB64C11FF              movzx ecx, byte ptr [ecx+edx-01]

:004C4BB8 03F1                    add esi, ecx

:004C4BBA 42                      inc edx

:004C4BBB 48                      dec eax

:004C4BBC 75F2                    jne 004C4BB0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4BA9(C)

|

:004C4BBE 8BC6                    mov eax, esi

:004C4BC0 B9DF000000              mov ecx, 000000DF

:004C4BC5 99                      cdq

:004C4BC6 F7F9                    idiv ecx

:004C4BC8 8BDA                    mov ebx, edx

:004C4BCA 8BC6                    mov eax, esi

:004C4BCC B985000000              mov ecx, 00000085

:004C4BD1 99                      cdq

:004C4BD2 F7F9                    idiv ecx

:004C4BD4 8855F7                  mov byte ptr [ebp-09], dl

:004C4BD7 8BC6                    mov eax, esi

:004C4BD9 B9AF000000              mov ecx, 000000AF

:004C4BDE 99                      cdq

:004C4BDF F7F9                    idiv ecx

:004C4BE1 8BC2                    mov eax, edx

:004C4BE3 33D2                    xor edx, edx

:004C4BE5 8AD3                    mov dl, bl

:004C4BE7 8BF2                    mov esi, edx

:004C4BE9 C1E60C                  shl esi, 0C

:004C4BEC 2BF2                    sub esi, edx

:004C4BEE 33C9                    xor ecx, ecx

:004C4BF0 8A4DF7                  mov cl, byte ptr [ebp-09]

:004C4BF3 8BF9                    mov edi, ecx

:004C4BF5 C1E708                  shl edi, 08

:004C4BF8 2BF9                    sub edi, ecx

:004C4BFA 03F7                    add esi, edi

:004C4BFC 8BF8                    mov edi, eax

:004C4BFE 81E7FF000000            and edi, 000000FF

:004C4C04 6BFF0F                  imul edi, 0000000F

:004C4C07 03F7                    add esi, edi

:004C4C09 03F2                    add esi, edx

:004C4C0B 03F1                    add esi, ecx

:004C4C0D 25FF000000              and eax, 000000FF

:004C4C12 03F0                    add esi, eax

:004C4C14 8BC6                    mov eax, esi

:004C4C16 8B5DFC                  mov ebx, dword ptr [ebp-04]

:004C4C19 33D8                    xor ebx, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4BA3(U)

|

:004C4C1B 33C0                    xor eax, eax

:004C4C1D 5A                      pop edx

:004C4C1E 59                      pop ecx

:004C4C1F 59                      pop ecx

:004C4C20 648910                  mov dword ptr fs:[eax], edx

:004C4C23 68384C4C00              push 004C4C38

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4C36(U)

|

:004C4C28 8D45F8                  lea eax, dword ptr [ebp-08]

:004C4C2B E82CFDF3FF              call 0040495C

:004C4C30 C3                      ret
```

***

因为我的机器上这个软件已经注册了，所以我还要首先改一下，如果有和我相同情况的朋友请如下更改：

```
004C5767  |.  8A93 3000000  MOV DL,BYTE PTR DS:[EBX+330]------>这个是注册与否得关键标志（`DL` 为 1 表示已注册）

004C576D  |.  8BC3          MOV EAX,EBX
```



`004C576F` |. `E8 D0FEFFFF` `CALL QM.004C5644`------------------------->这个调用显示关于对话框

下面的是我的分析过程:

以下是 `V1.32` 版本的注册码生成过程

最后返回生成的注册码在 `EAX` 中(跟踪可以发现,在本段中是在 `EBX` 中)

\* Referenced by a CALL at Address:

`:004C50ED`

```
:004C4B70  55                      push ebp
:004C4B71  8BEC                    mov ebp, esp
:004C4B73  83C4F4                  add esp, FFFFFFFF4
:004C4B76  53                      push ebx
:004C4B77  56                      push esi
:004C4B78  57                      push edi
:004C4B79  894DF8                  mov dword ptr [ebp-08], ecx\--------->“起名正宗 dnpf”
:004C4B7C  8955FC                  mov dword ptr [ebp-04], edx\--------->机器码
:004C4B7F  8B45F8                  mov eax, dword ptr [ebp-08]--------->
:004C4B82  E87502F4FF              call 00404DFC
:004C4B87  33C0                    xor eax, eax
:004C4B89  55                      push ebp
:004C4B8A  68314C4C00              push 004C4C31
:004C4B8F  64FF30                  push dword ptr fs:[eax]
:004C4B92  648920                  mov dword ptr fs:[eax], esp
:004C4B95  8B45F8                  mov eax, dword ptr [ebp-08]--------->"起名正宗 dnpf"
:004C4B98  E87700F4FF              call 00404C14----------------------->取长度
:004C4B9D  85C0                    test eax, eax\----------------------->测试长度
:004C4B9F  7504                    jne 004C4BA5------------------------>跳走
:004C4BA1  33DB                    xor ebx, ebx
:004C4BA3  EB76                    jmp 004C4C1B

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004C4B9F(C)

:004C4BA5  33F6                    xor esi, esi\----------------------->保存结果
:004C4BA7  85C0                    test eax, eax\---------------------->不可小于等于 0
:004C4BA9  7E13                    jle 004C4BBE
:004C4BAB  BA01000000              mov edx, 00000001------------------>指针初始化

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004C4BBC(C)

:004C4BB0  8B4DF8                  mov ecx, dword ptr [ebp-08]------->"起名正宗 dnpf"
:004C4BB3  0FB64C11FF              movzx ecx, byte ptr [ecx+edx-01]-->取首位值
:004C4BB8  03F1                    add esi, ecx\---------------------->结果保存在 ESI
:004C4BBA  42                      inc edx\--------------------------->指针+1,指向下一位
:004C4BBB  48                      dec eax\--------------------------->长度值-1
:004C4BBC  75F2                    jne 004C4BB0---------------------->判断结束否

\--------------------------------------------->这个循环把"起名正宗 dnpf"的 ASCII 值累加到 ESI

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004C4BA9(C)

:004C4BBE  8BC6                    mov eax, esi\--------------------->累加值(我的是 89F)
:004C4BC0  B9DF000000              mov ecx, 000000DF---------------->初始化 ECX
:004C4BC5  99                      cdq\------------------------------>EAX 高位扩展
:004C4BC6  F7F9                    idiv ecx\------------------------->整数除法
:004C4BC8  8BDA                    mov ebx, edx\--------------------->保存余数----------->1
:004C4BCA  8BC6                    mov eax, esi\--------------------->累加值
:004C4BCC  B985000000              mov ecx, 00000085---------------->初始化 ECX
:004C4BD1  99                      cdq\------------------------------>EAX 高位扩展
:004C4BD2  F7F9                    idiv ecx\------------------------->整数除法
:004C4BD4  8855F7                  mov byte ptr [ebp-09], dl\-------->保存余数----------->2
:004C4BD7  8BC6                    mov eax, esi\--------------------->累加值
:004C4BD9  B9AF000000              mov ecx, 000000AF---------------->初始化 ECX
:004C4BDE  99                      cdq\------------------------------>EAX 高位扩展
:004C4BDF  F7F9                    idiv ecx\------------------------->整数除法
:004C4BE1  8BC2                    mov eax, edx\--------------------->保存余数----------->3
:004C4BE3  33D2                    xor edx, edx\--------------------->EDX 清零
:004C4BE5  8AD3                    mov dl, bl\----------------------->第一次除法运算的余数
:004C4BE7  8BF2                    mov esi, edx\--------------------->同时到 ESI
:004C4BE9  C1E60C                  shl esi, 0C---------------------->逻辑左移 0C 位
:004C4BEC  2BF2                    sub esi, edx\--------------------->减法--------------->4
:004C4BEE  33C9                    xor ecx, ecx\--------------------->ECX 清零
:004C4BF0  8A4DF7                  mov cl, byte ptr [ebp-09]-------->第二次运算的结果
:004C4BF3  8BF9                    mov edi, ecx\--------------------->同时到 EDI
:004C4BF5  C1E708                  shl edi, 08---------------------->逻辑左移 08 位
:004C4BF8  2BF9                    sub edi, ecx\--------------------->减法-------------->5
:004C4BFA  03F7                    add esi, edi\--------------------->上面两个结果相加-->6
:004C4BFC  8BF8                    mov edi, eax\--------------------->第三次运算的余数到 EDI
:004C4BFE  81E7FF000000            and edi, 000000FF---------------->只保留最后两位
:004C4C04  6BFF0F                  imul edi, 0000000F--------------->乘法运算---------->7
:004C4C07  03F7                    add esi, edi\--------------------->再相加------------>8
:004C4C09  03F2                    add esi, edx\--------------------->再相加------------>9
:004C4C0B  03F1                    add esi, ecx\--------------------->再相加------------>10
:004C4C0D  25FF000000              and eax, 000000FF---------------->与
:004C4C12  03F0                    add esi, eax\--------------------->再相加------------>11
:004C4C14  8BC6                    mov eax, esi\--------------------->运算结果
:004C4C16  8B5DFC                  mov ebx, dword ptr [ebp-04]------>机器码到 EBX
:004C4C19  33D8                    xor ebx, eax\---->机器码和上面运算做异或运算-->过了这个运算 EBX 的结果就是最终结果

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004C4BA3(U)

:004C4C1B  33C0                    xor eax, eax
:004C4C1D  5A                      pop edx
:004C4C1E  59                      pop ecx
:004C4C1F  59                      pop ecx
:004C4C20  648910                  mov dword ptr fs:[eax], edx
:004C4C23  68384C4C00              push 004C4C38

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004C4C36(U)

:004C4C28  8D45F8                  lea eax, dword ptr [ebp-08]
:004C4C2B  E82CFDF3FF              call 0040495C
:004C4C30  C3                      ret
```

上面的注册码生成过程最重要的就是 1～11 步,使用 C 语言描述如下(别的不会了):

假设:

"起名正宗 dnpf"的 ASCII 累加值为 `name`

则前面 11 步的运算如下:

1. `1 = name % 223`
2. `2 = name % 133`
3. `3 = name % 175`
4. `4 = (name % 223) * 4096 - name % 223`
5. `5 = (name % 133) * 64 - name % 133`
6. `6 = [(name % 223) * 4096 - name % 223] + [(name % 133) * 64 - name % 133]`
7. `7 = [(name % 175) & 255] * 15`
8. `8 = [(name % 223) * 4096 - name % 223] + [(name % 133) * 64 - name % 133] + [(name % 175) & 255] * 15`
9. `9 = [(name % 223) * 4096 - name % 223] + [(name % 133) * 64 - name % 133] + [(name % 175) & 255] * 15 + (name % 223)`
10. `10 = [(name % 223) * 4096 - name % 223] + [(name % 133) * 64 - name % 133] + [(name % 175) & 255] * 15 + (name % 223) + name % 133`
11. `11 = [(name % 223) * 4096 - name % 223] + [(name % 133) * 64 - name % 133] + [(name % 175) & 255] * 15 + (name % 223) + name % 133 + [(name % 175) & 255]`

到第 11 步的结果之后剩下的工作就简单了: `注册码 = 11 步的结果 ^ 机器码`

`4dd466d0e1e0cc123afbe504da2462ca`
