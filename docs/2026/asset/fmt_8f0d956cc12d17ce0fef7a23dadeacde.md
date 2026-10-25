
# 反汇编分析

```asm
001B:0047CA86    PUSH    EDI            ;---->用户名指针入栈
001B:0047CA87    PUSH    00
001B:0047CA89    CALL    0047CA10        ;#C01.C2--->跟进去..
001B:0047CA8E    ADD    ESP,0C
...
```

进入 `#C01.C2 (CALL 0047CA10)`.

```asm
001B:0047CA10    MOV    ECX,[ESP+08]        ;---->ECX指向用户名'icytear'
001B:0047CA14    MOV    EAX,[ESP+0C]        ;---->EAX存用户名长度,
001B:0047CA18    PUSH    ESI
001B:0047CA19    LEA    ESI,[EAX+ECX]        ;---->ESI指向了用户名的末尾'\0'
001B:0047CA1C    CMP    ECX,ESI
001B:0047CA1E    JAE    0047CA4A        ;---->如果大于等于则跳转#exit.
001B:0047CA20    MOV    EAX,[ESP+08]
001B:0047CA24    PUSH    EBX
001B:0047CA25    MOV    EDX,EAX            ;#loop3
001B:0047CA27    XOR    EBX,EBX
001B:0047CA29    MOV    BL,[ECX]        ;---->开始逐个字符取用户名
001B:0047CA2B    AND    EDX,000000FF
001B:0047CA31    XOR    EDX,EBX
001B:0047CA33    XOR    EBX,EBX
001B:0047CA35    MOV    BL,AH
001B:0047CA37    MOV    AX,[EDX*2+00523410]    ;---->(00523410～00523610) #C01.C1 CALL 生成的东东
001B:0047CA3F    XOR    AX,BX
001B:0047CA42    INC    ECX
001B:0047CA43    CMP    ECX,ESI
001B:0047CA45    JB    0047CA25        ;---->如果小于跳转倒#loop3↑
001B:0047CA47    POP    EBX
001B:0047CA48    POP    ESI
001B:0047CA49    RET
001B:0047CA4A    MOV    AX,[ESP+08]        ;#exit.
001B:0047CA4F    POP    ESI
001B:0047CA50    RET
```

## `C01.C2 (CALL 0047CA10)` C语言代码如下：

```c
char username[]="icytear";

int sub_0047CA10(int n,char *pname,int unamelen)
{
    DWORD ieax,iebx,iedx,temp;
    char *iecx;
    iecx = pname;
    ieax = unamelen;
    if(*iecx == '\0')
        return 0;
    ieax = n;
    do{
        //MOV EDX,EAX
        iedx = ieax;
        //XOR EBX,EBX
        iebx = 0;
        //MOV BL,[ECX]
        temp = *iecx;
        iebx = iebx & 0xFFFFFF00;
        iebx = iebx ^ temp;
        //AND EDX,000000FF
        iedx = iedx & 0x000000FF;
        //XOR EDX,EBX
        iedx = iedx ^ iebx;
        //XOR EBX,EBX
        iebx = 0;
        //MOV BL,AH
        temp = ieax;
        temp = temp>>8;
        temp = temp & 0x000000FF;
        iebx = iebx & 0xFFFFFF00;
        iebx = iebx ^ temp;
        //MOV AX,[EDX*2+00523410]
        temp = list[iedx];
        ieax = ieax & 0xFFFF0000;
        ieax = ieax ^ temp;
        //XOR AX,BX
        temp = iebx;
        temp = temp & 0x0000FFFF;
        ieax = ieax ^ temp;
        iecx++;
    }while(*iecx != '\0');
    return ieax;
}
```

从 `#C01.C2 CALL` 出来到 `#C01CALL`

```asm
001B:0047CA8E    ADD    ESP,0C            ;---->#C01.C2 CALL 出来到这里
001B:0047CA91    AND    EAX,0000FFFF        ;---->AX是用用户名算出的结果
001B:0047CA96    PUSH    EAX
001B:0047CA97    LEA    EAX,[ESP+10]
001B:0047CA9B    PUSH    0051824C        ;---->;"%02X"
001B:0047CAA0    PUSH    EAX            ;
001B:0047CAA1    CALL    004AEABC        ;---->sprintf,把用户名算出的16进制结果以字符形式保存
001B:0047CAA6    MOV    ESI,[ESP+2C]
001B:0047CAAA    MOV    AL,[ESP+18]        ;---->[ESP+18] = "5CF",由"icytear"算出的值
001B:0047CAAE    ADD    ESP,0C
001B:0047CAB1    MOV    CL,[ESI+02]        ;---->[ESI+02] 注册码的第三个字符
001B:0047CAB4    LEA    EDI,[ESI+02]
001B:0047CAB7    CMP    CL,AL            ;---->比较,在这里比较注册码第三个字符是否为'5'
001B:0047CAB9    JZ    0047CAC4
001B:0047CABB    POP    EDI
001B:0047CABC    POP    ESI
001B:0047CABD    XOR    EAX,EAX
001B:0047CABF    POP    EBX
001B:0047CAC0    ADD    ESP,0C
001B:0047CAC3    RET
001B:0047CAC4    MOV    DL,[ESI+03]        ;---->注册码第四个字符
001B:0047CAC7    MOV    AL,[ESP+0D]        ;---->"5CF"的第二个字符
001B:0047CACB    CMP    DL,AL            ;---->比较注册码的第四个字符是否为'C'
001B:0047CACD    JZ    0047CAD8
001B:0047CACF    POP    EDI
001B:0047CAD0    POP    ESI
001B:0047CAD1    XOR    EAX,EAX
001B:0047CAD3    POP    EBX
001B:0047CAD4    ADD    ESP,0C
001B:0047CAD7    RET
001B:0047CAD8    PUSH    ESI            ;---->注册码
001B:0047CAD9    CALL    EBX            ;---->KERNEL!lstrlen
001B:0047CADB    SUB    EAX,02
001B:0047CADE    PUSH    EAX            ;---->EAX == 注册码长度 - 2;
001B:0047CADF    PUSH    EDI            ;---->第三位以后的注册码
001B:0047CAE0    PUSH    00
001B:0047CAE2    CALL    0047CA10        ;---->前面出现过这个函数，用注册码第三为以后的字符计算出一个值
001B:0047CAE7    ADD    ESP,0C
001B:0047CAEA    AND    EAX,0000FFFF
001B:0047CAEF    PUSH    EAX
001B:0047CAF0    LEA    EAX,[ESP+10]
001B:0047CAF4    PUSH    0051824C        ;---->"%02X"
001B:0047CAF9    PUSH    EAX
001B:0047CAFA    CALL    004AEABC        ;---->sprintf
001B:0047CAFF    MOV    CL,[ESI]        ;---->取注册码的第一个字符
001B:0047CB01    MOV    AL,[ESP+18]        ;---->由第三位以后的注册码计算出的值（第一个字符）
001B:0047CB05    ADD    ESP,0C
001B:0047CB08    CMP    CL,AL            ;---->比较注册码的第一位
001B:0047CB0A    JZ    0047CB15
001B:0047CB0C    POP    EDI
001B:0047CB0D    POP    ESI
001B:0047CB0E    XOR    EAX,EAX
001B:0047CB10    POP    EBX
001B:0047CB11    ADD    ESP,0C
001B:0047CB14    RET
001B:0047CB15    MOV    DL,[ESI+01]        ;---->取注册码的第二个字符
001B:0047CB18    MOV    CL,[ESP+0D]        ;---->由第三位以后的注册码计算出的值（第二个字符）
001B:0047CB1C    XOR    EAX,EAX
001B:0047CB1E    CMP    DL,CL            ;---->比较注册码的第二位
001B:0047CB20    POP    EDI
001B:0047CB21    POP    ESI
001B:0047CB22    SETZ    AL
001B:0047CB25    POP    EBX
001B:0047CB26    ADD    ESP,0C
001B:0047CB29    RET                ;---->出#C01 CALL
```

### 验证结论

以上代码，验证了注册码的前四位，以上可知注册码的第3、4位是由用户名用 `sub_0047CA10(int,char*,int)` 函数计算出的值16进制码的字符格式的前两位, 在这里为 `"5C"`; 注册码的第1、2位是由注册码第三位以后的字符用 `sub_0047CA10` 函数计算出的16进制码的字符格式的前两位,用 `"5C60443"` 计算的结果为 `"EF"`,所以在这时注册码应为 `"ef5c60443"`.

回到注册对话框重新输入注册码 `"ef5c60443"`,提示需要重启EditPlus激活注册码,看来第一步验证通过了.

### 第二步验证

重启EditPlus,提示 `"Invalid registration code."`,看来其它的地方还有注册码的验证过程,好我们回到程序继续跟踪...

这次跟踪注册码用 `"EF5C60443"`.

出 `#C01 CALL` 到...

```asm
001B:0047C942    ADD    ESP,08            ;---->返回到这里
001B:0047C945    TEST    EAX,EAX            ;---->
001B:0047C947    JNZ    0047C961        ;---->跳转则第一步注册成功
001B:0047C949    PUSH    FF
001B:0047C94B    PUSH    10
001B:0047C94D    PUSH    00005F81
001B:0047C952    CALL    004C7E94        ;---->出错对话框,
001B:0047C957    POP    EDI
001B:0047C958    POP    ESI
001B:0047C959    POP    EBX
001B:0047C95A    ADD    ESP,00000230
001B:0047C960    RET
===>:0047C961    PUSH    EDI            ;---->注册码
001B:0047C962    PUSH    EBX            ;---->用户名
001B:0047C963    CALL    0047C3B0        ;#C02---->关于注册表的动作????,跟进去看看..
001B:0047C968    ADD    ESP,08
001B:0047C96B    TEST    EAX,EAX
001B:0047C96D    JNZ    0047C986        ;--->输入"ef5c60443"时,在这里我们跳转了..
001B:0047C96F    PUSH    FF
001B:0047C971    PUSH    EAX
001B:0047C972    PUSH    00004E7F
001B:0047C977    CALL    004C7E94        ;--->注册失败!!
001B:0047C97C    POP    EDI
001B:0047C97D    POP    ESI
001B:0047C97E    POP    EBX
001B:0047C97F    ADD    ESP,00000230
001B:0047C985    RET
===>:0047C986    MOV    ECX,[ESI+000000D8]
001B:0047C98C    PUSH    EBX            ;---->用户名
001B:0047C98D    CALL    004C0367        ;---->CString::=
001B:0047C992    MOV    ECX,[ESI+000000DC]
001B:0047C998    PUSH    EDI            ;---->注册码"EF5C60443"
001B:0047C999    CALL    004C0367        ;---->CString::=
001B:0047C99E    MOV    EAX,[ESI+000000E4]
001B:0047C9A4    MOV    ECX,ESI
001B:0047C9A6    MOV    DWORD PTR [EAX],00000000
001B:0047C9AC    CALL    004BF482        ;---->CDialog::0nOK(void)
001B:0047C9B1    POP    EDI
001B:0047C9B2    POP    ESI
001B:0047C9B3    POP    EBX
001B:0047C9B4    ADD    ESP,00000230
001B:0047C9BA    RET
001B:0047C9BB    NOP
...
```

进入 `#C02` ...

```asm
001B:0047C3B0    SUB    ESP,000000E0
001B:0047C3B6    PUSH    EBX            ;---->用户名(保护现场)
001B:0047C3B7    PUSH    EBP
001B:0047C3B8    PUSH    ESI
001B:0047C3B9    MOV    ESI,[ESP+000000F0]    ;---->让ESI指向用户名
001B:0047C3C0    PUSH    EDI            ;---->注册码(保护现场)
001B:0047C3C1    MOV    EDI,[KERNEL32!lstrlen]
001B:0047C3C7    PUSH    ESI            ;---->用户名入栈了
001B:0047C3C8    CALL    EDI            ;---->得到用户名长度
001B:0047C3CA    MOV    EBX,EAX
001B:0047C3CC    MOV    EAX,[ESP+000000F8]
001B:0047C3D3    PUSH    EAX            ;---->EAX指向注册码
```
