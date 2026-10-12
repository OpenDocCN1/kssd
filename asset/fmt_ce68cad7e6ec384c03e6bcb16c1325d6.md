
### 反汇编代码分析

```
:00411123 55                           push ebp  
:00411124 8BEC                         mov ebp, esp  
:00411126 837D1400                     cmp dword ptr [ebp+14], 00000000  
:0041112A 8B4D0C                       mov ecx, dword ptr [ebp+0C]  
:0041112D 53                           push ebx  
:0041112E 56                           push esi  
:0041112F 57                           push edi  
:00411130 740B                         je 0041113D  
:00411132 8B7508                       mov esi, dword ptr [ebp+08]  
:00411135 C6012D                       mov byte ptr [ecx], 2D  
:00411138 41                           inc ecx  
:00411139 F7DE                         neg esi  
:0041113B EB03                         jmp 00411140

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00411130(C)  
|  
:0041113D 8B7508                       mov esi, dword ptr [ebp+08]  
                                   ====>ESI=2E844ECD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041113B(U)  
|  
:00411140 8BF9                         mov edi, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00411166(C)  
|  
:00411142 8BC6                         mov eax, esi  
:00411144 33D2                         xor edx, edx  
:00411146 F77510                       div [ebp+10]  
                                   ===>[ebp+10]=24  
                            1、   ====>EDX=2E844ECD  %  24=1D  
                            2、   ====>EDX=014AC94C  %  24=04  
                            3、   ====>EDX=00093042  %  24=06  
                            4、   ====>EDX=00004157  %  24=17  
                            5、   ====>EDX=000001D0  %  24=20  
                            6、   ====>EDX=0000000C  %  24=0C

:00411149 8BC6                         mov eax, esi  
:0041114B 8BDA                         mov ebx, edx  
:0041114D 33D2                         xor edx, edx  
:0041114F F77510                       div [ebp+10]  
                            1、   ====>EAX=2E844ECD  /  24=014AC94C  
                            2、   ====>EAX=014AC94C  /  24=00093042  
                            3、   ====>EAX=00093042  /  24=00004157  
                            4、   ====>EAX=00004157  /  24=000001D0  
                            5、   ====>EAX=000001D0  /  24=0000000C  
                            6、   ====>EAX=0000000C  /  24=00000000

:00411152 83FB09                       cmp ebx, 00000009  
:00411155 8BF0                         mov esi, eax  
                                   ====>ESI=EAX   用商继续运算! 直至为 0!
:00411157 7605                         jbe 0041115E  
:00411159 80C357                       add bl, 57  
                            1、   ====>BL=1D  +  57=74    即字符：t  
                            4、   ====>BL=17  +  57=6E    即字符：n  
                            5、   ====>BL=20  +  57=77    即字符：w  
                            6、   ====>BL=0C  +  57=63    即字符：c

:0041115C EB03                         jmp 00411161

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00411157(C)  
|  
:0041115E 80C330                       add bl, 30  
                            2、   ====>BL=04  +  30=34    即字符：4  
                            3、   ====>BL=06  +  30=36    即字符：6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041115C(U)  
|  
:00411161 8819                         mov byte ptr [ecx], bl  
                                   ====>BL 入 [ecx]处  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
循环结束后[ECX]内存中的值：

0068F528  74 34 36 6E 77 63                     t46nwc  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00411163 41                           inc ecx  
:00411164 85F6                         test esi, esi  
:00411166 77DA                         ja 00411142  
                                   ====>循环！

:00411168 802100                       and byte ptr [ecx], 00  
:0041116B 49                           dec ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00411178(C)  
|  
:0041116C 8A17                         mov dl, byte ptr [edi]  
:0041116E 8A01                         mov al, byte ptr [ecx]  
:00411170 8811                         mov byte ptr [ecx], dl  
:00411172 8807                         mov byte ptr [edi], al  
:00411174 49                           dec ecx  
:00411175 47                           inc edi  
:00411176 3BF9                         cmp edi, ecx  
:00411178 72F2                         jb 0041116C  
                                   ====>这个小循环是将 t46nwc 倒序为：cwn64t

:0041117A 5F                           pop edi  
:0041117B 5E                           pop esi  
:0041117C 5B                           pop ebx  
:0041117D 5D                           pop ebp  
:0041117E C3                           ret
```

### 算法总结

一、取硬盘序列号 `211C1E09` XOR `161D40D3`=`37015EDA`，得到软件序列号。

二、软件序列号 `37015EDA` XOR `19851017`=`2E844ECD`，用 `2E844ECD` 循环和 `24` 求模，直至商为 0。  
余数若小于 `9` 则加 `30`，转化为数字；否则加 `57` 转化为小写字母。再倒序排列。  
得到注册码前半部分：`cwn64t`

三、软件序列号 `2E844ECD`*`1`*`2`*`3`*`4`*`5`*`6`*`7`*`8`*`9`=`9A541B80`，用 `9A541B80` 循环和 `24` 求模，直至商为 0。  
余数若小于 `9` 则加 `30`，转化为数字；否则加 `57` 转化为小写字母。再倒序排列。  
得到注册码后半部分：`16tjm2o`

所以，我的注册码是：`cwn64t-16tjm2o`

### 完美爆破

1、`004069B5` `83D8FF` `sbb eax, FFFFFFFF`  
改为：`33C090` `xor eax, eax` 补一个 `NOP`

2、`00406A19` `83D8FF` `sbb eax, FFFFFFFF`  
改为：`33C090` `xor eax, eax` 补一个 `NOP`

### 注册信息保存

同目录下的 `sign.ini` 中：`cwn64t-16tjm2o`

### 整理

序列号：`922836698`  
注册码：`cwn64t-16tjm2o`

Cracked By 巢水工作坊——fly【OCN】  
2003-10-11 15:51  
呵呵，还是 fly 兄分析，我写注册机！！

### 注册机源码

`CB v6.0` `win98 se` 下调试通过！

```cpp
//---------------------------------------------------------------------------
void __fastcall Tform1::OKBtnClick(TObject *Sender)
{
    String key_1, key_11, key_2, key_22;
    __int64 Ma;
    int a, b, c = 1, d, b_1;
    unsigned long e = 0, f = 0, g = 0;

    if (MEdit->Text != "")
    {
        Ma = StrToInt64(MEdit->Text);
        e = Ma ^ 0x19851017;
        while (e > 0)
        {
            a = e % 0x24;
            e = e / 0x24;
            if (a <= 9)
                a = a + 0x30;
            else
                a = a + 0x57;
            key_1 = key_1 + char(a);
        }

        b = key_1.Length();
        while (b >= 1)
        {
            f = key_1[b];
            b--;
            key_11 = key_11 + char(f);
        }

        e = Ma ^ 0x19851017;
        while (c <= 9)
        {
            e = e * c;
            c++;
        }

        while (e > 0)
        {
            d = e % 0x24;
            e = e / 0x24;
            if (d <= 9)
                d = d + 0x30;
            else
                d = d + 0x57;
            key_2 = key_2 + char(d);
        }

        b_1 = key_2.Length();
        while (b_1 >= 1)
        {
            g = key_2[b_1];
            b_1--;
            key_22 = key_22 + char(g);
        }

        CEdit->Text = CEdit->Text + key_11 + "-" + key_22;
    }
}
```
