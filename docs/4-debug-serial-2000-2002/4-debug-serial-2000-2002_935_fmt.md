
# 冷雨飘心 crackme2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue139.htm`](https://www.pediy.com/kssd/pediy04/kanxue139.htm)

工具：`sice4.05`, `tc2`

环境：`win98`

对象：冷雨飘心的第二个 `crackme`

下载地址：`http://person.longcity.net/home3/feeling/crack/files/crkme2.zip`

下断点 `getdlgitemtexta`：

```
0167:0040115C  PUSH       32
0167:0040115E  PUSH       00403026
0167:00401163  PUSH       000003ED
0167:00401168  PUSH       DWORD PTR [EBP+08]
0167:0040116B  CALL       USER32!GetDlgItemTextA          //取得 name，放在 00403026
0167:00401170  MOV        [00403270],EAX                  //EAX 内是 name 的字符数
0167:00401175  CMP        EAX,00                           //看 name 是否为空
0167:00401178  JZ         00401389                         //是就报错
0167:0040117E  LEA        ESI,[00403026]
0167:00401184  CALL       004014B0                         //对输入的 name 加工。加工结果放在 004030DB
0167:00401189  MOV        EAX,[004030DB]
0167:0040118E  MOV        EDX,[004030DF]
0167:00401194  PUSH       004031BB
0167:00401199  PUSH       32
0167:0040119B  PUSH       0D
0167:0040119D  PUSH       DWORD PTR [00403494]
0167:004011A3  CALL       USER32!SendMessageA             //取得 serial,放在 004031bb
0167:004011A8  OR         EAX,EAX                         //看 serial 是否为空
0167:004011AA  JZ         00401389                         //是就报错
0167:004011B0  LEA        ESI,[004031BB]
0167:004011B6  MOV        DWORD PTR [ESI+EAX],00000000
0167:004011BD  PUSH       ESI
0167:004011BE  POP        EDI
0167:004011BF  MOV        AL,10
0167:004011C1  ADD        AL,1D
0167:004011C3  MOV        ECX,EAX
0167:004011C5  REPNZ SCASB                                //看 serial 内是否有"-"
0167:004011C7  JNZ        0040129                          //没有就说 serial 格式不对
0167:004011CD  MOV        EAX,[ESI]                        //取 serial 前 4 个字节
0167:004011CF  XOR        EAX,4743435F                     //看是否为"_CCG"。"_CCG"的 ASCII 码为 5F434347
0167:004011D4  JNZ        004012A4                         //不是就报错
0167:004011DA  LEA        ESI,[004031BB]
0167:004011E0  ADD        ESI,05                            //取 serial 第 5 个字节以后的数据
0167:004011E3  CALL       0040172F                         //检查 serial 是否符合要求。必须有 8 个字节，
                                                            //内容只能是 0-9 和 A-F，转换为 16 进制字符串。
                                                            //比如我输入的是_CCG-00000001-00000002,
                                                            //转换成 0x00000001
0167:004011E8  XOR        EAX,-01                          //输入的还不能全为 0 否则也报错
0167:004011EB  JZ         004012A4
0167:004011F1  XOR        EAX,-01
0167:004011F4  MOV        [004030E3],EAX
0167:004011F9  LEA        ESI,[004031BB]
0167:004011FF  ADD        ESI,0E                            //同上，检查 00000002
0167:00401202  CALL       0040172F
0167:00401207  XOR        EAX,-01
0167:0040120A  JZ         004012A4
0167:00401210  XOR        EAX,-01
0167:00401213  MOV        [004030E7],EAX
0167:00401218  MOV        EAX,[004030E3]
0167:0040121D  MOV        EDX,[004030E7]
0167:00401223  CALL       00401636                         //关键部分:通过 00000001 和 00000002 混合运算得到两 16 进制数
                                                            //为中间结果
0167:00401228  MOV        EAX,[004030EB]                    //00000001 的放在 EAX 里
0167:0040122D  MOV        EDX,[004030EF]                    //00000002 的放在 EDX 里
0167:00401233  XOR        EAX,[004030DB]                    //检查 0000001 的中间结果是否为[004030DB]，即 name 加工的结果
0167:00401239  JNZ        004012A4                         //不是就报错
0167:0040123B  XOR        EDX,[004030DF]                    //检查 0000002 的中间结果是否为 0x656d7568
0167:00401241  JNZ        004012A4                         //不是就报错
0167:00401243  JMP        004012BD                         //注册成功画面
```

下面是序列号生成程序，在 `TC2` 下调试通过。

```c
#include <stdio.h>
#include <dos.h>

unsigned long ESI[] = {
    0x6d797468,
    0x6d797468,
    0x6d797468,
    0x6c6f7665,
    0x6c6f7665,
    0x6c6f7665,
    0x6c6f7665,
    0x43525950,
    0x43525950,
    0x43525950,
    0x43525950,
    0x68656865,
    0x68656865,
    0x68656865,
    0x68656865,
    0x476f6f64
};

unsigned long ror(unsigned long a, unsigned long b)
{
    for(; b > 0; b --)
    {
        if((a & 0x00000001) == 0x00000001)
        {
            a = a >> 1;
            a = a + 0x80000000;
        }
        else
            a = a >> 1;
    }
    return a;
}

unsigned long rol(unsigned long a, unsigned long b)
{
    for(; b > 0; b --)
    {
        if((a & 0x80000000) == 0x80000000)
        {
            a = a << 1;
            a = a + 0x00000001;
        }
        else
            a = a << 1;
    }
    return a;
}

void main()
{
    unsigned char name[32], name_length;
    unsigned long t = 0, ans = 0;
    unsigned char a, b, l;
    unsigned long ecx = 0, edi = 0;
    int i;
    unsigned long eax = 0x00000000;
    unsigned long edx = 0x656d7568;

    for(i = 0; i < 32; i++)
        name[i] = 0;

    printf("\nPlease input your name:");
    gets(name);

    name_length = strlen(name);

    if(name_length >= 20)
    {
        ans = name_length;
        ans = ans << 16;
        t = 0;
        for(i = 1; i < 32; i++)
            t = t + name[i] * i;
        t = t ^ 0xffffffff;
        t = t & 0x0000ffff;
        ans = ans + t;
    }
    else
    {
        l = name_length;
        for(i = strlen(name) - 1; i < 32; i++)
        {
            a = name[i];
            b = a;
            a = a / l;
            b = b * l;
            a = a + b;
            name[l] = (char)a;
            l++;
        }
        a = 0;
        for(i = 0; i < name_length; i++)
            a = a + name[i];
        ans = ans + a;
        ans = ror(ans, 8);
        ans = ans + 0x200000;
        t = 0;
        for(i = 1; i < 32; i++)
            t = t + name[i] * i;
        t = t ^ 0xffffffff;
        t = t & 0x0000ffff;
        ans = ans + t;
    }

    eax = ans + 0x20434347;
    edx = edx + 0x26424347;
    i = 15;

    for(edi = 1; edi <= 8; edi++)
    {
        ecx = edx;
        eax = eax ^ edx;
        ecx = ecx & 0x0000001f;
        eax = rol(eax, ecx);
        eax = eax + ESI[i];
        i--;
        ecx = eax;
        edx = edx ^ eax;
        ecx = ecx & 0x0000001f;
        edx = rol(edx, ecx);
        edx = edx + ESI[i];
        i--;
    }

    printf("\n 下面是您的序列号:");
    printf("\n_CCG-%08lx-%08lx\n", eax, edx);
    printf("里面的字母应该为大写，请自己修正。");
}
```

请大家指正，谢谢！

`shengzan@263.net`
`4dd466d0e1e0cc123afbe504da2462ca`
