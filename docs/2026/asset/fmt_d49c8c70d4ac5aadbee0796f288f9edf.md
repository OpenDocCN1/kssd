
```asm
0042B46B  |.  8B4424 30     mov     eax, [esp+30]
0042B46F  |.  33F6          xor     esi, esi
0042B471  |.  C64424 24 02  mov     byte ptr [esp+24], 2
0042B476  |.  8B48 F8       mov     ecx, [eax-8]
0042B479  |.  85C9          test    ecx, ecx
0042B47B  |.  7E 4D         jle     short 0042B4CA
0042B47D  |>  8A1406        /mov     dl, [esi+eax]                   ;  取字符串“注册名@机器码”的第esi个字符的ASCII值赋给dl(设该值为x),开始计算注册码！
0042B480  |.  8BFD          |mov     edi, ebp                        ;  edi指向字符ProcessJudger
0042B482  |.  83C9 FF       |or      ecx, FFFFFFFF
0042B485  |.  33C0          |xor     eax, eax
0042B487  |.  F2:AE         |repne   scas byte ptr es:[edi]
0042B489  |.  F7D1          |not     ecx
0042B48B  |.  885424 14     |mov     [esp+14], dl
0042B48F  |.  49            |dec     ecx                             ;  ecx为字符串“Processjudger"的长度
0042B490  |.  8D041E        |lea     eax, [esi+ebx]                  ;  eax=esi+ebx
0042B493  |.  33D2          |xor     edx, edx
0042B495  |.  F7F1          |div     ecx                             ;  将eax除以ecx的余数赋给edx
0042B497  |.  8B4C24 14     |mov     ecx, [esp+14]                   ;  将x赋给ecx
0042B49B  |.  33C0          |xor     eax, eax
0042B49D  |.  81E1 FF000000 |and     ecx, 0FF
0042B4A3  |.  8A042A        |mov     al, [edx+ebp]                   ;  取字符串“Processjudger"的第edx个字符的ASCII值赋给al(设该值为y)!
0042B4A6  |.  8B5424 10     |mov     edx, [esp+10]
0042B4AA  |.  33C1          |xor     eax, ecx                        ;  x与y进行异或
0042B4AC  |.  50            |push    eax                             ;  异或结果入栈
0042B4AD  |.  52            |push    edx
0042B4AE  |.  8D4424 18     |lea     eax, [esp+18]
0042B4B2  |.  68 8CCB4600   |push    0046CB8C                        ;  %s%02x
0042B4B7  |.  50            |push    eax
0042B4B8  |.  E8 3F270200   |call    <jmp.&mfc42.#2818\_CString::Form>;  将异或结果保存并连接起来！
0042B4BD  |.  8B4424 40     |mov     eax, [esp+40]
0042B4C1  |.  83C4 10       |add     esp, 10
0042B4C4  |.  46            |inc     esi
0042B4C5  |.  3B70 F8       |cmp     esi, [eax-8]                    ;  计算完了吗？
0042B4C8  |.^ 7C B3         \jl      short 0042B47D                  ;  没计算完就跳回去继续计算
0042B4CA  |>  55            push    ebp                              ; /block
0042B4CB  |.  E8 2A260200   call    <jmp.&mfc42.#825\_operator delete>; \free
0042B4D0  |.  8B7424 30     mov     esi, [esp+30]
0042B4D4  |.  83C4 04       add     esp, 4
0042B4D7  |.  8D4C24 10     lea     ecx, [esp+10]
0042B4DB  |.  51            push    ecx
0042B4DC  |.  8BCE          mov     ecx, esi
0042B4DE  |.  E8 F7270200   call    <jmp.&mfc42.#535\_CString::CStrin>
0042B4E3  |.  C74424 18 010>mov     dword ptr [esp+18], 1
0042B4EB  |.  8D4C24 10     lea     ecx, [esp+10]
0042B4EF  |.  C64424 24 01  mov     byte ptr [esp+24], 1
0042B4F4  |.  E8 D9260200   call    <jmp.&mfc42.#800\_CString::~CStri>
0042B4F9  |.  8D4C24 30     lea     ecx, [esp+30]
0042B4FD  |.  C64424 24 00  mov     byte ptr [esp+24], 0
0042B502  |.  E8 CB260200   call    <jmp.&mfc42.#800\_CString::~CStri>
0042B507  |.  8B4C24 1C     mov     ecx, [esp+1C]
0042B50B  |.  8BC6          mov     eax, esi
0042B50D  |.  5F            pop     edi
0042B50E  |.  5E            pop     esi
0042B50F  |.  5D            pop     ebp
0042B510  |.  5B            pop     ebx
0042B511  |.  64:890D 00000>mov     fs:[0], ecx
0042B518  |.  83C4 18       add     esp, 18
0042B51B  \.  C3            retn
```

----------------------------------------------------------------

算法总结：设用户名`+"@"+`机器码组成的字符串为`S`，`S`的长度为`L`; `L`除以13（13为字符串“Processjudger"的长度）的余数为`n`,则字符串“Processjudger"从第`n`位（注：字符串“Processjudger"的“P”为第“0”位，第一位为“r”，其余类推）开始与字符串`S`循环异或，将异或结果的前16位保存并连接起来就是真正的注册码！

VB注册机源代码：

```vb
Dim str1 As String
Dim str2 As String
Dim c As String

str1 = Text1.Text + "@" + Text2.Text   //Text1.Text=注册用户名，Text2.Text＝机器码。
str2 = "ProcessJudger"
nlen = Len(str1)
For i = 1 To 8
n = nlen Mod 13
n = n + 1           //汇编语言中的第n个字符相当于VB中的第n+1个字符
b = Hex(Asc(Mid(str2, n, 1)) Xor Asc(Mid(str1, i, 1)))
If Len(b) < 2 Then b = "0" & b  //如果异或结果不是两位数则在前面加“0”
c = c & b
nlen = nlen + 1
Next i
Text3.Text = c    //Text3.Text 显示注册码
```

----------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

![](https://bbs.kanxue.com/upload/attach/200612/12239_shm4xxhk6teyvst.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [od3.jpg](javascript:void(0);)

  （27.34kb，35次下载）
