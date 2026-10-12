
# Becky! Internet Mail Version 2 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-429.htm`](https://www.pediy.com/kssd/pediy09/pediy09-429.htm)

> 【文章标题】: Becky! Internet Mail Version 2 算法分析
> 【文章作者】: Suyana
> 【作者邮箱】: Suyasha@163.com
> 【作者 QQ 号】: 517949855(请注明来自看雪论坛)
> 【软件名称】: Becky! Internet Mail Version 2
> 【加壳方式】: 无壳
> 【使用工具】: OD
> 【软件介绍】: 优秀的邮件软件之一，支持多个信箱，Voice Mail 功能
> 【作者声明】: 我只是一只小菜鸟，失误之处难免，敬望诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

下断点：`MessageBoxA`，注册，输入注册信息：

Suyana

`0710-3437-9012` (要 14 位,你也可以随便输，但这样你要注册很多次 ^_\^![扮鬼脸](img/7c79845954f5fbf9e981f570ed213a8d.jpg))

`Suyasha@163.com`，确定

中断下来，第一次是确认信息，点"是"，中断下来，还是中断在：

```
00571D25  .  FF15 F4A75800 call    [<&USER32.MessageBox>; \MessageBoxA
```

按 `F8` 到返回，返回到：

```
00571D7C   jmp     short 00571D8E，再返回到：
```

...

```
0052D85D   cmp     eax, 6                ; 返回到这里
0052D860   jnz     0052DA19
0052D866   mov     edx, [5BB0B8]       ; B2.005BB0CC
0052D86C   lea     ebx, [ebp+64]
...
0052D8A1   push    005B17E0             ; ASCII "RBK"
...
0052D8CA   call    0056447C
0052D8CF   lea     ecx, [esp+18]         ; RBK-加上注册码
...
0052D8E5   call    004157D0             ; 计算注册码，进入，参数是 RBK-加上注册码
0052D8EA   test    eax, eax
0052D8EC   jnz     0052D9D7            ; 注册失败时跳
```

`004157D0`：

```
004157D0   mov     eax, [esp+4]
004157D4   sub     esp, 14
004157D7   mov     ecx, [eax-8]          ; RBK-加上注册码的长度
004157DA   push    edi
004157DB   xor     edi, edi
004157DD   cmp     ecx, 12
004157E0   jnz     00415992             ; 不为 12 跳，所以注册码长度应为 12h-4h=Eh=14d
004157E6   mov     dl, [eax+3]
004157E9   mov     cl, 2D
004157EB   cmp     dl, cl
004157ED   jnz     00415992
004157F3   cmp     [eax+8], cl           ; 输入的注册码的第 4 位是否为'-'
004157F6   jnz     00415992
004157FC   cmp     [eax+D], cl         ; 输入的注册码的第 9 位是否为'-'
004157FF   jnz     00415992
00415805   push    ebx
00415806   push    esi
00415807   lea     eax, [esp+18]       ; RBK-注册码
0041580B   push    3
0041580D   push    eax
0041580E   lea     ecx, [esp+2C]
00415812   call    0055B984
00415817   push    4
00415819   lea     ecx, [esp+14]
0041581D   push    4
0041581F   push    ecx
00415820   lea     ecx, [esp+30]
00415824   call    0055B872            ; 根据'-'分割注册码
00415829   push    4
0041582B   lea     edx, [esp+18]
0041582F   push    9
00415831   push    edx
00415832   lea     ecx, [esp+30]
00415836   call    0055B872            ; 分割我们输入的第 2 段注册码
0041583B   push    4
0041583D   lea     eax, [esp+10]
00415841   push    0E
00415843   push    eax
00415844   lea     ecx, [esp+30]
00415848   call    0055B872            ; 分割我们输入的第 3 段注册码
0041584D   mov     esi, 005B17E0       ; ASCII "RBK"
00415852   mov     eax, [esp+18]       ; RBK
00415856    /mov    dl, [eax]           ; RBK 的第 n 位
00415858    |mov    bl, [esi]           ; esi=RBK 的第 n 位
0041585A    |mov    cl, dl
0041585C    |cmp    dl, bl
0041585E    |jnz    short 0041587E
00415860    |test   cl, cl
00415862    |je     short 0041587A
00415864    |mov    dl, [eax+1]        ; 这段代码比较两个相同的字符串 RBK
00415867    |mov    bl, [esi+1]        ; 不知道为什么，但下面计算第二段注
0041586A    |mov    cl, dl            ; 册码用的跟这个是同样的方法，也就是
0041586C    |cmp    dl, bl            ; 两个字符串要相等
0041586E    |jnz    short 0041587E
00415870    |add    eax, 2
00415873    |add    esi, 2
00415876    |test   cl, cl
00415878    \\jnz    short 00415856
0041587A   xor     eax, eax
0041587C   jmp     short 00415883
0041587E   sbb     eax, eax
00415880   sbb     eax, -1
00415883   test    eax, eax
00415885   jnz     00415965
0041588B   lea     eax, [esp+1C]
0041588F   push    2
00415891   push    eax
00415892   lea     ecx, [esp+18]
00415896   call    0055B908
0041589B   mov     eax, [eax]           ; 10,即第一段注册码的后 2 位
0041589D   push    eax
0041589E   call    0054A44B            ; 转成 16 进制,即 A
004158A3   add     esp, 4
004158A6   lea     ecx, [esp+1C]
004158AA   mov     esi, eax
004158AC   call    0056424D
004158B1   mov     ecx, [esp+10]
004158B5   push    ecx                  ; 0710,第一段注册码
004158B6   call    0054A44B            ; 转成 16 进制,2c6
004158BB   add     esp, 4
004158BE   test    eax, eax
004158C0   je      00415965            ; 第一段注册码不能为 0
004158C6   cmp     esi, 1
004158C9   jl      00415965            ; 第一段注册码的后 2 位应>1
004158CF   cmp     esi, 0C
004158D2   jg      00415965            ; 第一段注册码的后 2 位应<Ch
004158D8   mov     eax, [esp+14]        ; 第二段注册码应等于 3437
004158DC   mov     esi, 005B17D8        ; ASCII "3437"
004158E1    /mov    dl, [eax]           ; 第二段注册码第 n 位
004158E3    |mov    bl, [esi]           ; 3437 的第 n 位
004158E5    |mov    cl, dl
004158E7    |cmp    dl, bl             ; 应该相等
004158E9    |jnz    short 00415909      ; 不能跳
004158EB    |test   cl, cl
004158ED    |je     short 00415905
004158EF    |mov    dl, [eax+1]
004158F2    |mov    bl, [esi+1]
004158F5    |mov    cl, dl
004158F7    |cmp    dl, bl             ; 应该相等
004158F9    |jnz    short 00415909      ; 不能跳
004158FB    |add    eax, 2
004158FE    |add    esi, 2
00415901    |test   cl, cl
00415903    \\jnz    short 004158E1
00415905   xor     eax, eax
00415907   jmp     short 0041590E
00415909   sbb     eax, eax
0041590B   sbb     eax, -1
0041590E   test    eax, eax
00415910   jnz     short 00415965
00415912   mov     eax, [esp+C]         ; 第三段注册码
00415916   movsx   ecx, byte ptr [eax+1] ; 第三段注册码的第 2 位的 ascii
0041591A   push    ecx
0041591B   call    0054A8F6
00415920   add     esp, 4
00415923   test    eax, eax              ; 不能为跳
00415925   je      short 00415965
00415927   mov     edx, [esp+C]         ; 第三段注册码
0041592B   movsx   eax, byte ptr [edx+2] ; 第三段注册码的第 3 位的 ascii
0041592F   push    eax
00415930   call    0054A8F6
00415935   add     esp, 4
00415938   test    eax, eax              ; 不能为跳
0041593A   je      short 00415965
0041593C   mov     ecx, [esp+C]
00415940   movsx   edx, byte ptr [ecx+3] ; 第三段注册码的第 4 位的 ascii
00415944   push    edx
00415945   call    0054A8F6
0041594A   add     esp, 4
0041594D   test    eax, eax              ; 不能为跳
0041594F   je      short 00415965
00415951   mov     eax, [esp+C]         ; 第三段注册码
00415955   movsx   ecx, byte ptr [eax]   ; 第三段注册码的第 1 位的 ascii
00415958   push    ecx
00415959   call    0054A8A0
0041595E   add     esp, 4
00415961   test    eax, eax              ; 要跳
00415963   jnz     short 0041596A
```

可看出，注册码主要在第三段，偶的运气真好，偶的注册码 `0710-3437-9012`，只有 9 这位不对，不然就是真的注册码了。程序用注册码的 `ascii` 做参数，调用 `0054A8F6`，返回 `0` 则下面的跳转将实现。但比较第三段注册码的第 1 位时调用的是 `0054A8A0`，进入该 `call`：

...

```
0054A8BA   mov     eax, [esp+4]          ; 取参数，即 39
```
