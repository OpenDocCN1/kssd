
# Flash ScreenSaver Builder v4.7 算法分析

> **原文**：[`www.pediy.com/kssd/pediy09/pediy09-432.htm`](https://www.pediy.com/kssd/pediy09/pediy09-432.htm)
>
> **【文章标题】**: Flash ScreenSaver Builder v4.7 算法分析
>
> **【文章作者】**: Suyana
>
> **【作者邮箱】**: `Suyasha@163.com`
>
> **【作者 QQ 号】**: `517949855`(请注明来自看雪论坛)
>
> **【软件名称】**: Flash ScreenSaver Builder v4.7
>
> **【加壳方式】**: 无壳
>
> **【编写语言】**: Microsoft Visual C++ 6.0
>
> **【使用工具】**: `OD`
>
> **【软件介绍】**: 把 Flash 动画转换成屏保程序，除此之外，它也可以将图像、文本、各种音频视频文件组合起来制作成屏保程序。
>
> **【作者声明】**: 我只是一只小菜鸟，失误之处难免，敬望诸位大侠赐教!!
>
> ---
>
> **【详细过程】**
>
> 用 `OD` 载入，运行注册，输入注册码。
>
> 下断点：`GetWindowTextA`（VC++ 编译的程序基本都是这条），点 `ok`，中断在：
>
> ```
> 004AB0D3    call    [<&USER32.GetWindowTextA>]
> ```
>
> 运行到返回，来到：
>
> ```
> 0043B784    mov     edi, [esp+C]
> 0043B788    lea     eax, [esi+60]
> 0043B78B    push    eax                     ; /Arg3
> 0043B78C    push    3E8                     ; |Arg2 = 000003E8
> 0043B791    push    edi                     ; |Arg1
> 0043B792    call    004AB0A4                ; \\得到用户名
> 0043B797    add     esi, 64
> 0043B79A    push    esi                     ; /Arg3
> 0043B79B    push    3E9                     ; |Arg2 = 000003E9
> 0043B7A0    push    edi                     ; |Arg1
> 0043B7A1    call    004AB0A4                ; \\得到注册码
> 0043B7A6    pop     edi                     ;  00166C08
> 0043B7A7    pop     esi
> 0043B7A8    retn    4
> ```
>
> 运行到返回，来到：
>
> ```
> 0049ABE5    mov     ecx, esi
> 0049ABE7    call    [eax+84]                ; 得到用户名和注册码(怎么 VC++编译出的代码都这样)!
> 0049ABED    mov     dword ptr [ebp+8], 1    ; 返回到这里
> 0049ABF4    jmp     short 0049AC1D
> ```
>
> `0049AC1D`：
>
> 这里没做什么事，直接返回到：
>
> ```
> 0043B889    call    0049AB98
> 0043B88E    test    eax, eax                ; 返回到这里
> 0043B890    je      0043BA7C
> 0043B896    fld     dword ptr [4EA6BC]
> 0043B89C    sub     esp, 8
> 0043B89F    fstp    qword ptr [esp]
> 0043B8A2    push    ecx
> 0043B8A3    mov     ecx, esp
> 0043B8A5    mov     [esp+20], esp
> 0043B8A9    push    84EF
> 0043B8AE    call    0049CE49
> 0043B8B3    push    3
> 0043B8B5    lea     ecx, [esp+2C]
> 0043B8B9    call    00476340
> 0043B8BE    mov     eax, [esi+60]
> 0043B8C1    mov     dword ptr [esp+4C], 0
> 0043B8C9    mov     ecx, [eax-8]
> 0043B8CC    test    ecx, ecx
> 0043B8CE    je      0043B9E7
> 0043B8D4    mov     eax, [esi+64]           ; 注册码
> 0043B8D7    lea     ecx, [esp+1C]
> 0043B8DB    push    eax                   ; 注册码做参数
> 0043B8DC    call    00476940              ; 计算注册码(严格上说是比较)!
> 0043B8E1    test    al, al
> 0043B8E3    je      0043B9E7              ; al=0，则注册失败
> ```
>
> 进入 `0043B8DC` `call` `00476940`：
>
> ```
> 00476940  /$  53            push    ebx
> 00476941  |.  8B5C24 08     mov     ebx, [esp+8]
> 00476945  |.  55            push    ebp
> 00476946  |.  56            push    esi
> 00476947  |.  57            push    edi
> 00476948  |.  8BE9          mov     ebp, ecx
> 0047694A  |.  53            push    ebx                     ; /String
> 0047694B  |.  FF15 50544C00 call    [<&KERNEL32.lstrlenA>]            ; \\lstrlenA
> 00476951  |.  83F8 11       cmp     eax, 11                       ;  注册码要 17 位
> 00476954  |.  0F85 DA000000 jnz     00476A34
> 0047695A  |.  8A03          mov     al, [ebx]                     ;  注册码第 n 位
> 0047695C  |.  33FF          xor     edi, edi
> 0047695E  |.  84C0          test    al, al                        ;  是否为 0
> 00476960  |.  8BF3          mov     esi, ebx
> 00476962  |.  74 44         je      short 004769A8
> 00476964  |>  8A06          /mov     al, [esi]                     ;  注册码第 n 位
> 00476966  |.  3C 2D         |cmp     al, 2D                        ;  为'-' tiao
> 00476968  |.  74 2C         |je      short 00476996
> 0047696A  |.  0FBEC0        |movsx   eax, al
> 0047696D  |.  50            |push    eax
> 0047696E  |.  E8 47D00000   |call    004839BA
> 00476973  |.  83C4 04       |add     esp, 4
> 00476976  |.  85C0          |test    eax, eax
> 00476978  |.  8A06          |mov     al, [esi]
> 0047697A  |.  74 0A         |je      short 00476986
> 0047697C  |.  3C 30         |cmp     al, 30
> 0047697E  |.  7C 0E         |jl      short 0047698E                ; <=30 跳
> 00476980  |.  3C 39         |cmp     al, 39
> 00476982  |.  7E 1C         |jle     short 004769A0                ; <= 39,跳
> 00476984  |.  EB 08         |jmp     short 0047698E
> 00476986  |>  3C 41         |cmp     al, 41
> 00476988  |.  7C 04         |jl      short 0047698E
> 0047698A  |.  3C 46         |cmp     al, 46
> 0047698C  |.  7E 12         |jle     short 004769A0
> 0047698E  |>  3C 2D         |cmp     al, 2D
> 00476990  |.  0F85 9E000000 |jnz     00476A34
> 00476996  |>  47            |inc     edi
> 00476997  |.  83FF 02       |cmp     edi, 2
> 0047699A  |.  0F8F 94000000 |jg      00476A34                      ; 超过 2 个'-'就注册失败
> 004769A0  |>  8A46 01       |mov     al, [esi+1]
> 004769A3  |.  46            |inc     esi
> 004769A4  |.  84C0          |test    al, al
> 004769A6  |.^ 75 BC         \\jnz     short 00476964
> 004769A8  |>  6A 2D         push    2D
> ```
>
> ...下面还有一些函数，是拿注册码去做运算的，但最后好像都没用到，就不贴出来了...
>
> ```
> 004769DB    mov     esi, eax
> 004769DD    push    6                          ; /n = 6
> 004769DF    push    ebx                        ; |String2
> 004769E0    push    esi                        ; |String1
> 004769E1    call    [<&KERNEL32.lstrcpynA>]    ; \\lstrcpynA，复制注册码第一段
> 004769E7    mov     edx, [esp+14]
> 004769EB    add     edx, 3
> 004769EE    push    edx                        ; /StringToAdd
> 004769EF    push    esi                        ; |ConcatString
> 004769F0    call    [<&KERNEL32.lstrcatA>]     ; \\lstrcatA，加上注册码第三段的后 3 位
> 004769F6    lea     eax, [esp+14]
> 004769FA    push    10
> 004769FC    inc     edi
> 004769FD    push    eax
> 004769FE    push    edi
> 004769FF    call    004838A3                  ; 第二段注册码转成数字
> 00476A04    push    10
> 00476A06    mov     edi, eax                  ; eax=转换后的数字
> 00476A08    imul    edi, [ebp+8]              ; [ebp+8]=[0012e8b4]=5c，定值
> 00476A0C    push    0
> 00476A0E    push    esi                       ; 第一段注册码+第三段注册码的后 3 位
> 00476A0F    call    004838A3                  ; 转成数字
> 00476A14    mov     ebx, [ebp+10]             ; 25da8a35，定值
> 00476A17    push    esi
> 00476A18    add     eax, ebx                  ; 加上 25da8a35
> 00476A1A    imul    eax, [ebp+C]              ; 35，定值
> 00476A1E    cmp     edi, eax
> 00476A20    sete    bl                        ; 不等则 bl=0
> 00476A23    call    0049C002
> 00476A28    add     esp, 1C
> 00476A2B    mov     al, bl                    ; al=bl
> 00476A2D    pop     edi
> ```
>
> ---
>
> **注册码算法**：
>
> 第二段注册码转成数字 `*` `5c` = ((第一段注册码+第三段注册码的后 3 位)转成数字+`25da8a35`)*`35`
>
> 我用第二段注册码算出第一段注册码+第三段注册码的后 3 位：公式：
>
> `第二段注册码转成数字*5c/35-25da8a35`，但在除以 `35h` 时一定要整除，所以偶直接用 `35`
>
> 例如：第二段注册码:`35`，则注册码为：
>
> `(35*5c)/35-25da8a35=DA257627=第一段注册码+第三段注册码的后 3 位。`
>
> 第三段注册码的前 2 位随便填就可以了，得出的注册码。



**DA257-00035-77627**

注册码保存在：`HKEY_LOCAL_MACHINE\SOFTWARE\XemiComputers\FSSBuilder\Registration\4.70`

---

**附件**：注册机(`MASM32`)，这个注册机的模板是我自己写的，有兴趣的可以找我拿源代码(`MASM32`)![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

下面是计算注册码的函数：(用 `35h` 的倍数来算)

```
GetSN   proc uses ecx  hWnd
        local szBuffer[30]:byte, szKey1[6]:byte, szKey3[6]:byte

        add key2, 35h
        mov eax, key2
        push 5ch
        pop ecx
        imul ecx
        push 35h
        pop ecx
        idiv ecx
        sub eax, 25da8a35h
        invoke wsprintf, addr szBuffer, addr szFmtS, eax
        invoke lstrcpyn, addr szKey1, addr szBuffer, 6
        lea eax, szBuffer
        add eax, 5
        invoke lstrcpy, addr szKey3, eax
        invoke wsprintf, addr szBuffer, addr szFmt, addr szKey1, key2, addr szKey3
        invoke SetDlgItemText, hWnd, 1001, addr szBuffer    ; 显示注册码
        ret
        _GetSN        endp
```

代码写得不好，请指教。有更好算法的欢迎指出。

---

**【版权声明】**：本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

文章写于 `2007-08-18`

`4dd466d0e1e0cc123afbe504da2462ca`
