
# 对delphi中ImageEn组件的注册分析

**文章标题**：对delphi中ImageEn组件的注册分析

**文章作者**：sunsjw

**下载地址**：119K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2A6j5$3!0E0M7r3!0F1k6h3&6@1M7#2)9J5k6h3y4G2L8g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8U0t1K6x3W2)9J5c8V1c8u0L8h3q4Y4k6f1g2F1x3U0l9H3y4#2)9J5k6h3#2K6K9b7`.`.

**加壳方式**：无

**编写语言**：`delphi`

**作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

这是一个Delphi图像处理的组件，功能强大，用这个可以开发简单的图像处理软件。

我下载的是FOR `delphi2007`版本的，其它版本的算法就不知道了。

## 开始分析

注程序在安装目录里的`setup.exe`，先用`peid`查，无壳。既然是`delphi`的东西就要`dede`出场了。

用`dede`分析找到注册按钮的事件，反汇编如下

\* Reference to control `Edit2` : `TEdit`

|

`00447A9A`   `8B80DC020000`           `mov     eax, [eax+$02DC]`

\* Reference to: `controls.TControl.GetText(TControl):TCaption`;

|

`00447AA0`   `E8C3E2FDFF`             `call    00425D68`

`00447AA5`   `837DE000`               `cmp     dword ptr [ebp-$20], +$00`

`00447AA9`   `746D`                   `jz      00447B18`

`00447AAB`   `6A12`                   `push    $12`

`00447AAD`   `6873010000`             `push    $00000173`

`00447AB2`   `8D55DC`                 `lea     edx, [ebp-$24]`

`00447AB5`   `8B45FC`                 `mov     eax, [ebp-$04]`

\* Reference to control `Edit2` : `TEdit`

|

`00447AB8`   `8B80DC020000`           `mov     eax, [eax+$02DC]`

\* Reference to: `controls.TControl.GetText(TControl):TCaption`;

|

`00447ABE`   `E8A5E2FDFF`             `call    00425D68`

`00447AC3`   `8B45DC`                 `mov     eax, [ebp-$24]`

`00447AC6`   `8D55E5`                 `lea     edx, [ebp-$1B]`

`00447AC9`   `B912000000`             `mov     ecx, $00000012`

|

`00447ACE`   `E889040000`             `call    00447F5C`     可疑的Call，不能放过。这里要记下地址

`00447AD3`   `C645F700`               `mov     byte ptr [ebp-$09], $00`

`00447AD7`   `8D55D8`                 `lea     edx, [ebp-$28]`

`00447ADA`   `8B45FC`                 `mov     eax, [ebp-$04]`

好了，现在该`OD`上场了。用`OD`加载`setup.exe`。`Ctrl+G`来到`00447ACE`，`F2`下断点。

```
00447AAB   .  6A 12         PUSH 12                                    ;固定值，序列号的长度
00447AAD   .  68 73010000   PUSH 173                                   ;固定值，算法里有用到
00447AB2   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]
00447AB5   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00447AB8   .  8B80 DC020000 MOV EAX,DWORD PTR DS:[EAX+2DC]
00447ABE   .  E8 A5E2FDFF   CALL setup.00425D68
00447AC3   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]            ;用户名
00447AC6   .  8D55 E5       LEA EDX,DWORD PTR SS:[EBP-1B]            ;存放序列号的地方,没有算之前，这里是空
00447AC9   .  B9 12000000   MOV ECX,12                               ;
00447ACE   .  E8 89040000   CALL setup.00447F5C                      <-----断在这里,注册算法就在这个里面
00447AD3   .  C645 F7 00    MOV BYTE PTR SS:[EBP-9],0
00447AD7   .  8D55 D8       LEA EDX,DWORD PTR SS:[EBP-28]
00447ADA   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00447ADD   .  8B80 E0020000 MOV EAX,DWORD PTR DS:[EAX+2E0]
00447AE3   .  E8 80E2FDFF   CALL setup.00425D68
00447AE8   .  8B45 D8       MOV EAX,DWORD PTR SS:[EBP-28]
00447AEB   .  E8 64C3FBFF   CALL setup.00403E54
00447AF0   .  8BD0          MOV EDX,EAX                                ;用户输入的序列号
00447AF2   .  8D45 E5       LEA EAX,DWORD PTR SS:[EBP-1B]        ;这一步一执行，在EAX里直接就可以看到序列号了
00447AF5   .  E8 220AFCFF   CALL setup.0040851C                        ;序列号比较，就是一般的字符串比较
00447AFA   .  85C0          TEST EAX,EAX
00447AFC   .  74 1A         JE SHORT setup.00447B18                ;相等就跳，然后是一些登记操作，我没分析了
00447AFE   .  6A 00         PUSH 0                                   ; /Arg1 = 00000000
00447B00   .  66:8B0D A87C4>MOV CX,WORD PTR DS:[447CA8]              ; |
00447B07   .  33D2          XOR EDX,EDX                              ; |
00447B09   .  B8 B47C4400   MOV EAX,setup.00447CB4                   ; |ASCII "Invalid serial number"
00447B0E   .  E8 C9E4FFFF   CALL setup.00445FDC                      ; \setup.00445FDC
```

好了，到这里我们就要分析`CALL setup.00447F5C`里到底进行了些什么操作。

该`IDA`上场了，因为`IDA`看起来比`OD`标的更清楚些。。。

`IDA`加载`setup`，跳转到指定地址，输入 `00447F5C`，下面是`IDA`的反汇编代码,这里要说下的是`delphi`的参数传递，前两个参数用寄存器`EAX`,`EDX`后面两个参数用堆栈传递。

```
sub_447F5C      proc near               ; CODE XREF: sub_4474CC+1D7p
CODE:00447F5C                                         ; sub_4474CC+2C3p ...
CODE:00447F5C
CODE:00447F5C var_c           = dword ptr -0Ch
CODE:00447F5C sn              = dword ptr -8                        ;这里我重新命名了
CODE:00447F5C username        = dword ptr -4                        ;这里我重新命名了
CODE:00447F5C const_173       = dword ptr  8                        ;这里我重新命名了
CODE:00447F5C const_12        = dword ptr  0Ch                        ;这里我重新命名了

CODE:00447F5C                 push    ebp
CODE:00447F5D                 mov     ebp, esp
CODE:00447F5F                 add     esp, 0FFFFFFF4h
CODE:00447F62                 push    ebx
CODE:00447F63                 push    esi
CODE:00447F64                 push    edi
CODE:00447F65                 mov     [ebp+sn], edx                ;保存序列号地址
CODE:00447F68                 mov     [ebp+username], eax        ;保存用名名地址
CODE:00447F6B                 mov     edi, [ebp+const_173]
CODE:00447F6E                 mov     ebx, [ebp+const_12]         ; ebx固定值 0x12（18）
CODE:00447F71                 dec     ebx
CODE:00447F72                 test    ebx, ebx
CODE:00447F74                 jl      short sum_SN
CODE:00447F76                 inc     ebx
CODE:00447F77                 mov     [ebp+var_c], 0
CODE:00447F7E                 mov     esi, offset init_table
```

`init_table` 是一个整型数组，我已经把它弄出来了，见下面

从`Init_SN`开始依据输入的用户名对序列号进行初始化操作：

循环变量`var_c`从`0`开始对用户名字符串长度求余得`rem`,然后取出用户名字符串中的第`rem`位字符`x`,从`init_table`数组中取出第`var_c`位的数字`y`,然后把字符`x`放入序列号数组的第`y`位，循环`18`次。

```
CODE:00447F83
CODE:00447F83 Init_SN:                                ; CODE XREF: sub_447F5C+50
CODE:00447F83                 mov     eax, [ebp+username]
CODE:00447F86                 call    Length
CODE:00447F8B                 push    eax                     ; username的长度
CODE:00447F8C                 mov     eax, [ebp+var_c]         ; eax <- var_c
CODE:00447F8F                 pop     edx
CODE:00447F90                 mov     ecx, edx                ; username的长度 给ecx
CODE:00447F92                 cdq
CODE:00447F93                 idiv    ecx                     ; var_c / 长度
CODE:00447F95                 inc     edx                     ; edx余数
CODE:00447F96                 mov     eax, [ebp+username]         ;eax用户名
CODE:00447F99                 mov     al, [eax+edx-1]                ;取用户名的第 edx 个字符
CODE:00447F9D                 mov     edx, [esi]                ;从init_table取一个数字edx
CODE:00447F9F                 mov     ecx, [ebp+sn]                ;取序列号地址
CODE:00447FA2                 mov     [ecx+edx], al                ;把取出的字符入到序列号的第edx位
CODE:00447FA5                 inc     [ebp+var_c]                ;var_c+1
CODE:00447FA8                 add     esi, 4
CODE:00447FAB                 dec     ebx
CODE:004FAC                 jnz     short Init_SN
```

循环结束，在序列号数组里填充了依据用户名字符中计算得的一组字符。

```
CODE:00447FAE
CODE:00447FAE sum_SN:                                 ; CODE XREF: sub_447F5C+18
CODE:00447FAE                 mov     ebx, [ebp+const_12]
CODE:00447FB1                 dec     ebx
CODE:00447FB2                 test    ebx, ebx
CODE:00447FB4                 jl      short loc_447FDF
CODE:00447FB6                 inc     ebx
CODE:00447FB7                 mov     eax, [ebp+sn]
```

从这里开始对序列号进行一些逻辑运算。循环变量`i`从`0`到`18`，从序列号数组中取了第`i`个字符`a`,变量`di`的初始值为`173`，字符`a`和`di`右移`8`的值进行异或操作后得到值`x`放入序列号的第`i`位，然后再对`di`进行如下操作`edi = (x+di)*0CE6Dh + 58BFh`

```
CODE:00447FBA
CODE:00447FBA XXX_SN1:                                ; CODE XREF: sub_447F5C+81
CODE:00447FBA                 xor     edx, edx
CODE:00447FBC                 mov     dl, [eax]                ;从序列号数组中取出一个字符放入dl
CODE:00447FBE                 movzx   ecx, di                ;di的值给ecx也就是173
CODE:00447FC1                 shr     ecx, 8                ;173 右移8位 放入ecx
CODE:00447FC4                 xor     edx, ecx                ;刚取出的符和右移后的值进行xor操作。
CODE:00447FC6                 mov     [eax], dl                ;xor后的值再放回序列号数组中
CODE:00447FC8                 xor     edx, edx
```
