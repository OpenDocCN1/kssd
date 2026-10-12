
# Delphi 逆向工程笔记\[5\]

> 原文：[`www.pediy.com/kssd/pediy06/pediy6609.htm`](https://www.pediy.com/kssd/pediy06/pediy6609.htm)

OK，`LicenseProc` 的最后一部分——`WM_PAINT`。应该有却还没有的效果应该就在里面。

```
0000:00404EBD loc_404EBD:
0000:00404EBD             lea     eax, [ebp+Paint]
0000:00404EC0             push    eax             ; lpPaint
0000:00404EC1             mov     eax, [ebp+hWnd]
0000:00404EC4             push    eax             ; hWnd
0000:00404EC5             call    BeginPaint
0000:00404ECA             push    offset sRect
0000:00404ECF             push    0
0000:00404ED1             push    767676h
0000:00404ED6             mov     ecx, offset dword_4052F0
0000:00404EDB             mov     edx, ds:dword_4076E8
//结合下面可以看到，这是一个 hIcon，但奇怪的是，IDA 显示没有其它的
//ref。然而我翻了一下前面的代码，发现这就是主程序的 h_Icon！
//看来 IDA 的确有问题。
0000:00404EE1             call    near ptr sub_403FF8
//好吧，又来一个公共 call。
//记得以前专门研究过 TP/Delphi 的传值问题，但当时写的东西不知道丢
//哪里了。不过幸好 D5 的 HLP 已经很详细了。
//从这里看应该有 5 个参数，但是考虑到 Register 方式的特点，我怀疑还
//有第 6 个参数 eax。从逻辑上看也应该是这样。查看 sub_403FF8 发现，
//需要在 eax 里放一个 hdc，而 BeginPaint 的返回值就是 hdc。
//那么 dword_4052F0 是什么呢？查看 Hex 可知，就是窗口标题。这样，我
//以前的一个分析就被证明是错误的，不过不影响已经得到的代码。
0000:00404EE6             lea     eax, [ebp+Paint]
0000:00404EE9             push    eax             ; lpPaint
0000:00404EEA             mov     eax, [ebp+hWnd]
0000:00404EED             push    eax             ; hWnd
0000:00404EEE             call    EndPaint
0000:00404EF3             jmp     loc_404F8F
```

公共 `call`:

```
0000:00403FF8 sub_403FF8    proc far
0000:00403FF8
0000:00403FF8 rc            = RECT ptr -50h
0000:00403FF8 var_40        = LOGBRUSH ptr -40h
0000:00403FF8 Rect          = tagRECT ptr -34h
0000:00403FF8 var_24        = word ptr -24h
//这个是循环变量
0000:00403FF8 var_22        = word ptr -22h
0000:00403FF8 var_20        = word ptr -20h
0000:00403FF8 var_1E        = word ptr -1Eh
0000:00403FF8 var_1C        = word ptr -1Ch
0000:00403FF8 var_1A        = word ptr -1Ah
0000:00403FF8 var_18        = word ptr -18h
//从 var_18 到 var_22 都用了一次
0000:00403FF8 var_16        = word ptr -16h
0000:00403FF8 var_14        = word ptr -14h
0000:00403FF8 var_12        = word ptr -12h
//上面 3 个有点象 rgb，范围都是字节，变量
0000:00403FF8 hbr           = dword ptr -10h
//必须说明变量
0000:00403FF8 var_C         = dword ptr -0Ch
//仅用一次
0000:00403FF8 hIcon         = dword ptr -8
//内部没有改变
0000:00403FF8 hDC           = dword ptr -4
//内部没有改变
0000:00403FF8 arg_4         = dword ptr  10h
0000:00403FF8
0000:00403FF8             push    ebp
0000:00403FF9             mov     ebp, esp
0000:00403FFB             add     esp, 0FFFFFFB0h
0000:00403FFE             push    ebx
0000:00403FFF             push    esi
0000:00404000             push    edi
0000:00404001             mov     esi, [ebp+arg_4]
//公共的 Rect 变量
0000:00404004             lea     edi, [ebp+Rect]
//局部 Rect 变量
0000:00404007             push    ecx
0000:00404008             mov     ecx, 4
0000:0040400D             rep movsd
0000:0040400F             pop     ecx
//把公共变量复制过来
0000:00404010             mov     [ebp+var_C], ecx
0000:00404013             mov     [ebp+hIcon], edx
0000:00404016             mov     [ebp+hDC], eax
//三个通过 reg 传递的参数
0000:00404019             xor     eax, eax
0000:0040401B             call    nullsub_1
//空过程，为什么？延时？还是 Align？
0000:00404020             and     eax, 0FFh
0000:00404025             mov     [ebp+var_18], ax
//置 0
0000:00404029             xor     eax, eax
0000:0040402B             call    sub_403C90
//右移 8 位
0000:00404030             and     eax, 0FFh
0000:00404035             mov     [ebp+var_1A], ax
0000:00404039             xor     eax, eax
0000:0040403B             call    sub_403C94
//右移 16 位，就是取高位
0000:00404040             and     eax, 0FFh
0000:00404045             mov     [ebp+var_1C], ax
//置 0
//很多废代码
0000:00404049             mov     eax, 767676h
//这里很奇特，不知道和那个废参数有什么关系。怀疑原来就是
//那个参数，因为调试的需要临时注释掉了，或者相反。
0000:0040404E             call    nullsub_1
0000:00404053             and     eax, 0FFh
0000:00404058             mov     [ebp+var_1E], ax
0000:0040405C             mov     eax, 767676h
0000:00404061             call    sub_403C90
0000:00404066             and     eax, 0FFh
0000:0040406B             mov     [ebp+var_20], ax
0000:0040406F             mov     eax, 767676h
0000:00404074             call    sub_403C94
0000:00404079             and     eax, 0FFh
0000:0040407E             mov     [ebp+var_22], ax
//字节分离，太象 RGB 了！
0000:00404082             mov     ebx, [ebp+Rect.right]
0000:00404085             sub     ebx, [ebp+Rect.left]
0000:00404088             sar     ebx, 1
0000:0040408A             jns     short loc_40408F
0000:0040408C             adc     ebx, 0
//取水平中点，或者说水平长度的一半
0000:0040408F
0000:0040408F loc_40408F:
0000:0040408F             xor     eax, eax
0000:00404091             mov     [ebp+rc.top], eax
//清 0
0000:00404094             mov     eax, [ebp+Rect.bottom]
0000:00404097             sub     eax, [ebp+Rect.top]
0000:0040409A             mov     [ebp+rc.bottom], eax
//这里应该是求高
0000:0040409D             xor     eax, eax
0000:0040409F             mov     [ebp+var_40.lbStyle], eax
//清 0
0000:004040A2             xor     eax, eax
0000:004040A4             mov     [ebp+var_40.lbHatch], eax
//同上
0000:004040A7             test    bx, bx
0000:004040AA             jl      loc_4041C8
0000:004040B0             mov     eax, ebx
0000:004040B2             inc     eax
0000:004040B3             mov     [ebp+var_24], ax
//sar 以后最高位是保留的
0000:004040B7             xor     edi, edi
//从下面可以看到，edi 是个计数器，而 var_24 是循环变量
//一般说循环变量会用 reg 以加快速度的，但是这里例外
//看循环变量就看循环末尾的 dec 什么的
0000:004040B9
0000:004040B9 loc_4040B9:
//一个 Damn 循环，很长，到 004041C2
0000:004040B9             movsx   esi, bx
//注意 movsx，这说明 ebx 不会很大
//但是 ebx>$7f=127 可以说明什么呢?
0000:004040BC             push    esi             ; nDenominator
0000:004040BD             push    esi             ; nNumerator
0000:004040BE             movsx   eax, di
0000:004040C1             push    eax             ; nNumber
0000:004040C2             call    MulDiv
//这个地方虽然调用了 MulDiv，但是由于不知道它是不是由 Delphi 自动调用的，
//所以必须写段程序实验一下。用两个 Longword 相乘，再除以另外一个 Longword。
//可以看到，Delphi 用了浮点指令。所以结论是，MulDiv 是源程序调用的。
//另外，注意到 MulDiv 的参数意义，可以看出这个调用实际是没有实际作用的。
//这样的例子下面还有。
0000:004040C7             mov     [ebp+rc.left], eax
0000:004040CA             push    esi             ; nDenominator
0000:004040CB             push    esi             ; nNumerator
0000:004040CC             movsx   eax, di
0000:004040CF             inc     eax
0000:004040D0             push    eax             ; nNumber
0000:004040D1             call    MulDiv
0000:004040D6             mov     [ebp+rc.right], eax
0000:004040D9             push    esi             ; nDenominator
0000:004040DA             movsx   eax, [ebp+var_1E]
0000:004040DE             push    eax             ; nNumerator
0000:004040DF             movsx   eax, di
0000:004040E2             push    eax             ; nNumber
0000:004040E3             call    MulDiv
```
