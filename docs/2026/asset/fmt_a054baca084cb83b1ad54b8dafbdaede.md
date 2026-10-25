
```c
int main(int argc, char* argv[])

{

    /*

    除法

    */

    __int64 NumberOne = 0;

    __int64 NumberTwo = 0;

    scanf("%I64d", &NumberOne);

    scanf("%I64d", &NumberTwo);



    __int64 Count1 = NumberOne / 3;

    __int64 Count2 = NumberTwo / 5;

    __int64 Count3 = NumberTwo / 6;

    __int64 Count4 = NumberTwo / 9;

    printf("%I64d%I64d%lld%lld", Count4, Count3, Count2, Count1);

    system("pause");



    return 0;

}
```

## 反汇编

```
text:00000001400010D0                 sub     rsp, 38h

.text:00000001400010D4                 xor     eax, eax

.text:00000001400010D6                 lea     rdx, [rsp+38h+arg_10]

.text:00000001400010DB                 lea     rcx, aI64d      ; "%I64d"

.text:00000001400010E2                 mov     [rsp+38h+arg_10], rax

.text:00000001400010E7                 mov     [rsp+38h+arg_18], rax

.text:00000001400010EC                 call    sub_140001080

.text:00000001400010F1                 lea     rdx, [rsp+38h+arg_18]

.text:00000001400010F6                 lea     rcx, aI64d      ; "%I64d"

.text:00000001400010FD                 call    sub_140001080

.text:0000000140001102                 mov     rcx, [rsp+38h+arg_18]



.text:0000000140001107                 mov     rax, 5555555555555556h

.text:0000000140001111                 imul    [rsp+38h+arg_10]

.text:0000000140001120                 mov     r10, rdx

.text:0000000140001123                 shr     r10, 3Fh

.text:0000000140001127                 add     r10, rdx

.text:000000014000112A                 imul    rcx

.text:000000014000112D                 mov     [rsp+38h+var_18], r10

.text:0000000140001132                 mov     r9, rdx

.text:0000000140001135                 sar     r9, 1

.text:0000000140001138                 mov     rax, r9

.text:000000014000113B                 shr     rax, 3Fh

.text:000000014000113F                 add     r9, rax



.text:0000000140001142                 mov     rax, 2AAAAAAAAAAAAAABh

.text:000000014000114C                 imul    rcx

.text:000000014000114F                 mov     rax, 1C71C71C71C71C72h

.text:0000000140001159                 mov     r8, rdx

.text:000000014000115C                 shr     r8, 3Fh

.text:0000000140001160                 add     r8, rdx

.text:0000000140001163                 imul    rcx

.text:0000000140001166                 lea     rcx, aI64dI64dLldLld ; "%I64d%I64d%lld%lld"

.text:000000014000116D                 mov     rax, rdx

.text:0000000140001170                 shr     rax, 3Fh

.text:0000000140001174                 add     rdx, rax

.text:0000000140001177                 call    sub_140001020

.text:000000014000117C                 lea     rcx, aPause     ; "pause"

.text:0000000140001183                 call    sub_1400045C4

.text:0000000140001188                 xor     eax, eax

.text:000000014000118A                 add     rsp, 38h

.text:000000014000118E                 retn
```

观看反汇编其实跟x86一致

这里还是去掉流水线优化 提取核心代码位置进行反汇编

```
.text:0000000140001107                 mov     rax, 5555555555555556h

.text:0000000140001111                 imul    [rsp+38h+arg_10]

.text:0000000140001120                 mov     r10, rdx

.text:0000000140001123                 shr     r10, 3Fh

.text:0000000140001127                 add     r10, rdx
```

`M`数值变大.还是代入公式即可. 这里使用的`mov r10,rdx` 说明直接使用的`rdx`. `rdx`是一个64位数. 所以 `2^64 / M = 2.9999999` 向上取整 = 3

### 5.3 linux gcc x86 有符号非2的幂

高级代码

```c
#include<stdio.h>



int main()

{



    int NumberOne = 0;

    int NumberTwo = 0;

    scanf("%d", &NumberOne);

    scanf("%d", &NumberTwo);

    int Count1 = NumberOne / 3;

    int Count2 = NumberTwo / 5;

    int Count3 = NumberTwo / 6;

    int Count4 = NumberTwo / 9;

    printf("%d%d%d%d", Count4, Count3, Count2, Count1);

    scanf("%d", &Count4);

    scanf("%d", &Count3);

    scanf("%d", &Count2);

    scanf("%d", &Count1);

    printf("%d%d%d%d", Count4, Count3, Count2, Count1);

    return 0;

}
```

如果你是64位系统.那么你使用`gcc`命令如下

```
gcc xxx.c -o2 -m32
```

编译出来即可. 其实下面反汇编跟x86 vc++6.0 vs2019 一样.可以不用看了.

反汇编
