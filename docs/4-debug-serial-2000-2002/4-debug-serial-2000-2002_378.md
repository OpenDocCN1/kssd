
# keygenning4newbies keygenme #3

> 原文：[`www.pediy.com/kssd/pediy03/forum1228.htm`](https://www.pediy.com/kssd/pediy03/forum1228.htm)

`keygenning4newbies keygenme #3` 注册算法,虽然说是初学者的 crackme,可是也不简单哟~~~

我输入的名字是:`TAE!`

注册码:`78787878`

下面是检查注册码个数的地方，和普通的检测方法不一样，瞧瞧：

```asm
:004011F5 6BC003             imul eax, 00000003           ;eax 是假注册码的个数
:004011F8 C1E002             shl eax, 02                  ;左移两位
:004011FB 05CD000000         add eax, 00000CD             ;加上 0xCD
:00401200 8945FC             mov dword ptr [ebp-04], eax   ;保存
:00401203 817DFCA5010000     cmp dword ptr [ebp-04], 000001A5;和 0x1A5 比较
:0040120A 0F85BC000000       jne 004012CC                 ;不同就跳走了
:00401210 33C0               xor eax, eax                 ;清零
:00401212 8A4594             mov al, byte ptr [ebp-6C]     ;依次取注册码字符
:00401215 84C0               test al, al                  ;是最后一个字符吗？
:00401217 7413               je 0040122C                  ;是就跳走了！准备计算
:00401219 8D4D94             lea ecx, dword ptr [ebp-6C]   ;指向注册码
```

根据上面的算法,可以得到这样一个方程式:

`(个数 * 3 * 2^2) + 0xCD = 0x1A5` (注：`2^2` 指的是 2 的平方!)

既然知道了这个，求“个数”应该就非常简单了吧?用 Windows 的计算器简单的算一下,

`0x1A5 - 0xCD = 0xD8`, 用 `0xD8 / 2^2 = 0x36`, 最后用 `0x36 / 3 = 0x12`, 换算成 10 进制,那么就是 18,

所以,根据上面的算法可以知道注册码必须是 18 个字符！那么输入 `787878787878787878`

继续跟踪来到下面，这里是检查注册码中是否有小于 0 的 Ascii 码的字符。有的话程序就判断注册码是错的！

```asm
:0040121C 3C30               cmp al, 30                   ;小于零吗？
:0040121E 0F82C6000000       jb 004012EA                  ;小于就跳走了！完蛋！
:00401224 8A4101             mov al, byte ptr [ecx+01]    ;否则就将下一个字符给 al
:00401227 41                 inc ecx                      ;指向下一个字符
:00401228 84C0               test al, al                  ;是空吗
:0040122A 75F0               jne 0040121C                 ;没有结束就跳，形成循环
```

出了上面那个循环,我们便来到了下面:

```asm
:0040122C E8CFFDFFFF         call 00401000                ;这里面会将 eax,ebx,ecx,edx 清零
:00401231 8D852CFFFFFF       lea eax, dword ptr [ebp+FFFFFF2C];指向用户名
:00401237 50                 push eax                     ;将用户名压入堆栈
:00401238 E843FEFFFF         call 00401080                ;里面会对用户名进行操作
```

给初学者介绍一点经验,如果你发现程序先将用户名压入堆栈,后面紧跟着一个 Call,那你就得非常注意了,除了计算个数,那么很可能就是计算注册码了!所以我们一定得进去看看到底发生了什么!

```asm
:00401080 55                 push ebp
:00401081 8BEC               mov ebp, esp
:00401083 51                 push ecx
:00401084 53                 push ebx
:00401085 56                 push esi
:00401086 57                 push edi
* Possible StringData Ref from Data Obj ->"eheh"       //注意这个哟
                   |
:00401087 6880504000         push 00405080
:0040108C 6A00               push 00000000
:0040108E E8ADFFFFFF         call 00401040      ;将上面的字符 Ascii 反相放入 eax
:00401093 83C408             add esp, 00000008
:00401096 8BD8               mov ebx, eax       ;再放到 ebx 中去
:00401098 E863FFFFFF         call 00401000
* Possible StringData Ref from Data Obj ->" is a whore." //什么?妓女??
                   |
:0040109D BF70504000         mov edi, 00405070
:004010A2 83C9FF             or ecx, FFFFFFFF
:004010A5 33C0               xor eax, eax
:004010A7 F2                 repnz
.......
:004010CC 8B4508             mov eax, dword ptr [ebp+08]
:004010CF 50                 push eax           ;到了这里,用户名就和"is a whore"给串连了!
:004010D0 56                 push esi           ;妈的,竟然说咱是妓女!
:004010D1 E86AFFFFFF         call 00401040      ;每次取串连后的四个字符
:004010D6 8B8E30504000       mov ecx, dword ptr [esi+00405030] ;将"12"放入 ecx
:004010DC 83C408             add esp, 00000008
:004010DF 33CF               xor ecx, edi       ;异或结果放入 ecx
:004010E1 03C1               add eax, ecx       ;用四个字符的十六进制加上 ecx
:004010E3 8945FC             mov dword ptr [ebp-04], eax;保存
:004010E6 C145FC07           rol dword ptr [ebp-04], 07 ;带进位的左移七位
:004010EA 8B45FC             mov eax, dword ptr [ebp-04];结果放入 eax
:004010ED 83C604             add esi, 00000004
:004010F0 33D8               xor ebx, eax       ;异或(ebx 是"eheh"的十六进制)
:004010F2 47                 inc edi
:004010F3 83FE40             cmp esi, 00000040  ;哇!做六十四次!!
:004010F6 7CD4               jl 004010CC        ;跳转形成循环!
:004010F8 5F                 pop edi
:004010F9 8BC3               mov eax, ebx
:004010FB 5E                 pop esi
:004010FC 5B                 pop ebx
:004010FD 8BE5               mov esp, ebp
:004010FF 5D                 pop ebp
:00401100 C3                 ret
```

上面算出一个结果在 `eax` 和 `ebx` 中

好啦,终于可以回去了!

```asm
:0040123D 8945FC             mov dword ptr [ebp-04], eax ;保存
:00401240 E8BBFDFFFF         call 00401000                ;这里面会将 eax,ebx,ecx,edx 清零
:00401245 8D8D2CFFFFFF       lea ecx, dword ptr [ebp+FFFFFF2C];"TAE! is a whore"字符串
:0040124B 56                 push esi
:0040124C 51                 push ecx
:0040124D E8BEFDFFFF         call 00401010
:00401252 83C40C             add esp, 0000000C
:00401255 33C9               xor ecx, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401284(C)
|

:00401257 8B45FC             mov eax, dword ptr [ebp-04] ;记得这个吗?就是上面计算出的值啦!
:0040125A 33D2               xor edx, edx                 ;清零
:0040125C BE1A000000         mov esi, 0000001A            ;0x1A 放入 esi
:00401261 F7F6               div esi                      ;用上面的值除以 0x1A
:00401263 8A941510FFFFFF     mov dl, byte ptr [ebp+edx-000000F0] ;查一张由 26 个大写字母组成的表
:0040126A 88540DC8           mov byte ptr [ebp+ecx-38], dl    ;保存
:0040126E 8B45FC             mov eax, dword ptr [ebp-04]      ;取值放入 eax
:00401271 C1E003             shl eax, 03                      ;将它乘以 8
:00401274 BA45230100         mov edx, 00012345                ;0x12345 放入 edx
:00401279 F7E8               imul eax                         ;乘以 eax
:0040127B 03C2               add eax, edx                     ;和 0x12345 相加
:0040127D 8945FC             mov dword ptr [ebp-04], eax      ;又保存喽!
:00401280 41                 inc ecx                          ;计数器
:00401281 83F912             cmp ecx, 00000012                ;结束了吗?
:00401284 72D1               jb 00401257                      ;没有就跳转
:00401286 E875FDFFFF         call 00401000
:0040128B 33C0               xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004012A2(C)
|

:0040128D 8A4C0594           mov cl, byte ptr [ebp+eax-6C]    ;输入的注册码
:00401291 8A5405C8           mov dl, byte ptr [ebp+eax-38]    ;查表得到的字母
:00401295 80E930             sub cl, 30                       ;Ascii 减 0x30
:00401298 32D1               xor dl, cl                       ;和查表后的字母异或
:0040129A 885405C8           mov byte ptr [ebp+eax-38], dl    ;保存
:0040129E 40                 inc eax                          ;计数器
:0040129F 83F812             cmp eax, 00000012                ;结束了?
:004012A2 72E9               jb 0040128D                      ;未结束就跳!
:004012A4 E857FDFFFF         call 00401000
:004012A9 8D55C8             lea edx, dword ptr [ebp-38]
```



:004012AC `52` `push edx`
:004012AD `E8 5E FE FF FF` `call 00401110` ; 这里面是最后的运算!
:004012B2 `E8 49 FD FF FF` `call 00401000`
:004012B7 `8D 45 C8` `lea eax, dword ptr [ebp-38]`

进入 `call 00401110`

```assembly
:00401110 8B4C2404            mov ecx, dword ptr [esp+04]
:00401114 33C0               xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401122(C)
|
:00401116 8A1408             mov dl, byte ptr [eax+ecx]       ; 上面结果的一个数
:00401119 32D0               xor dl,al                         ; 和位数异或(从零开始)
:0040111B 881408             mov byte ptr [eax+ecx], dl        ; 保存!
:0040111E 40                 inc eax
:0040111F 83F812             cmp eax, 00000012                  ; 又是这么多的循环
:00401122 72F2               jb 00401116                        ; 循环
:00401124 C3                 ret
```

回去喽!

下面就是最后的比较了! 呼, 好累!

```assembly
* Possible StringData Ref from Data Obj -> "KEYGENNING4NEWBIES"
                     |
:004012BA 6814514000   push 00405114                            ; 这个是 "KEYGENNING4NEWBIES"
:004012BF 50           push eax                                 ; 这个是最后运算出的结果
:004012C0 E86BFEFFFF   call 00401130                            ; 比较是否相同
:004012C5 83C40C       add esp, 0000000C
:004012C8 85C0         test eax, eax
:004012CA 753C         jne 00401308
```

所以咱们的注册码经过运算后一定要和 `"KEYGENNING4NEWBIES"` 这个字符串相同,

好了, 算法已经清楚了, 谁来写个注册机? 伪装者大侠? 这个够级别吧? 试试呀! 也好让咱这些向往搞注册机的朋友学习学习;)

```c
#include "stdio.h"
#include "string.h"

#define ROTATE(a, n) (((a) << (n)) | (((a) & 0xffffffff) >> (32 - (n))))

void main()
{
    FILE *p = fopen("sn.txt", "wt");
    unsigned char string[80];
    int i, d, c;
    int tb1[16] = {0x12, 0x5c, 0x34, 0x22, 0xab, 0x9d, 0x54, 0x00, 0xdd, 0x84, 0xae, 0x66, 0x31, 0x78, 0x73, 0xcf};
    unsigned tb2[18];
    char key[18] = "KEYGENNING4NEWBIES";
    unsigned long f = 0, e = 0x68656865;

    printf("**********KEYGENME #3**********\nThis keygen is made by Pretender\nPlease input your name : ");
    gets(string);

    fputs("Name : ", p);
    fputs(string, p);
    fputc('\r', p);
    fputc('\f', p);
    fputs("SN : ", p);

    printf("Your Register code is : ");
    strcat(string, " is a whore.");

    d = strlen(string);
    if (d % 4 != 0)
    {
        for (i = d; i < d + 4 - d % 4; i++)
            string[i] = '\0';
        d = i;
    }

    for (i = 0; i < 16; i++)
    {
        if (i * 4 < d)
            f = string[i * 4] + string[i * 4 + 1] * 256 + string[i * 4 + 2] * 65536 + string[i * 4 + 3] * 256 * 65536;
        else
            f = 0;
        c = tb1[i] ^ i;
        f += c;
        f = ROTATE(f, 7);
        e ^= f;
    }

    for (i = 0; i < 18; i++)
    {
        tb2[i] = e % 26;
        tb2[i] += 65;
        __asm
        {
            mov eax, e
            shl eax, 3
            mov edx, 0x12345
            imul eax
            add eax, edx
            mov e, eax
        }
    }

    for (i = 0; i < 18; i++)
    {
        c = key[i];
        tb2[i] ^= i;
        c ^= tb2[i];
        c += 0x30;
        printf("%c", c);
        fputc(c, p);
    }

    printf("\nYou can get the code in sn.txt");
    printf("\n ***************--- --- ---\n *Welcome to WWW.CRACKNOW.COM* / / / --\n ***************--- --- --/\n");
}
```
