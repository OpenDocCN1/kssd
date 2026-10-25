
## 三丶除数为2的幂表现形式

```c
int main(int argc, char* argv[])
{
    /* 除法 */
    unsigned int NumberOne = 0;
    unsigned int NumberTwo = 0;

    scanf("%u", &NumberOne);
    scanf("%u", &NumberTwo);

    unsigned Count1 = NumberOne / NumberTwo;
    unsigned Count2 = NumberOne / 2;
    unsigned Count3 = NumberTwo / 4;
    unsigned Count5 = NumberTwo / 8;

    printf("%d%d%d%d%d", Count5, Count3, Count2, Count1);
    system("pause");
    return 0;
}
```

### 核心汇编

```asm
.text:0040102C                 mov     esi, [esp+1Ch+var_8]
.text:00401030                 mov     ecx, [esp+1Ch+var_4]
.text:00401034                 mov     eax, esi
.text:00401036                 xor     edx, edx
.text:00401038                 div     ecx                 ; 变量/变量 符号位不适用cdq因为无符号所以直接清空edx
.text:0040103A                 mov     edx, ecx
.text:0040103C                 shr     esi, 1              ; 直接使用逻辑右移来进行计算结果.符号位补0
.text:0040103E                 shr     edx, 2
.text:00401041                 shr     ecx, 3
```

可以看到，如果我们做无符号除以 `2` 的幂，直接使用 `shr` 逻辑右移来进行优化了。

反汇编的时候，取 `n` 值进行还原即可。

```asm
.text:0040103C                 shr     esi, 1              ; 除数还原为 esi / 2^1
.text:0040103E                 shr     edx, 2              ; 除数还原为 edx / 2^2
.text:00401041                 shr     ecx, 3              ; 除数还原为 ecx / 2^3
```

## 四丶除数为负2的幂表现形式

### 4.1 高级代码与核心汇编代码

```c
int main(int argc, char* argv[])
{
    /* 除法 */
    int NumberOne = 0;
    int NumberTwo = 0;

    scanf("%d", &NumberOne);
    scanf("%d", &NumberTwo);

    int Count1 = NumberOne / NumberTwo;
    int Count2 = NumberOne / -2;
    int Count3 = NumberTwo / -4;
    int Count5 = NumberTwo / -8;

    printf("%d%d%d%d%d", Count5, Count3, Count2, Count1);
    system("pause");
    return 0;
}
```

汇编

```asm
mov     esi, [esp+1Ch+var_8]
mov     ecx, [esp+1Ch+var_4]
mov     eax, esi
cdq
idiv    ecx

mov     eax, esi
cdq
sub     eax, edx
sar     eax, 1
neg     eax

mov     eax, ecx
cdq
and     edx, 3
add     eax, edx
sar     eax, 2
neg     eax

mov     eax, ecx
cdq
and     edx, 7
add     eax, edx
sar     eax, 3
neg     eax
```

观看汇编代码，可以看到其代码定式跟除数为2的幂一样。唯一不同的就是商的结果进行求补。

`neg` 指令的作用就是 `0 - xxx`，比如 `eax = 1`，`neg(eax)` 等于是 `0 - 1 = FFFFFFFF`。

这里加这条指令的含义就是：如果商为负数，那么进行求补，商的结果就为正数了。

还原方式还是取 `n` 值，唯一不同的是因为有 `neg`，所以前边加负数。

```asm
sar eax, 1     ; 还原为 eax / (-2^1)  下面同上
neg eax
sar eax, 2
neg eax
sar eax, 3
neg eax
```

## 五丶 Visual Studio 2019 x86x64 下除数为2的幂的优化方式

其实这个主题主要做一个对比，目的就是告诉大家 `vc6.0` 与 Visual Studio 2019 下代码优化方式一样。

### 5.1 x86下 除数为正数2的幂 高级代码与反汇编

```c
int main(int argc, char* argv[])
{
    /* 除法 */
    int NumberOne = 0;
    int NumberTwo = 0;

    scanf("%d", &NumberOne);
    scanf("%d", &NumberTwo);

    int Count1 = NumberOne / NumberTwo;
    int Count2 = NumberOne / 2;
    int Count3 = NumberTwo / 4;
    int Count5 = NumberTwo / 8;

    printf("%d%d%d%d%d", Count5, Count3, Count2, Count1);
    system("pause");
    return 0;
}
```

汇编代码

```asm
.text:00401080                 push    ebp
.text:00401081                 mov     ebp, esp
.text:00401083                 sub     esp, 8
.text:00401086                 push    esi
.text:00401087                 lea     eax, [ebp+var_4]
.text:0040108A                 mov     [ebp+var_4], 0
.text:00401091                 push    eax
.text:00401092                 push    offset unk_41ECDC
.text:00401097                 mov     [ebp+var_8], 0
.text:0040109E                 call    sub_401050
.text:004010A3                 lea     eax, [ebp+var_8]
.text:004010A6                 push    eax
.text:004010A7                 push    offset unk_41ECDC
.text:004010AC                 call    sub_401050

; 核心汇编
.text:004010B1                 mov     eax, [ebp+var_4]
.text:004010B4                 mov     esi, [ebp+var_8]
.text:004010B7                 cdq
.text:004010B8                 idiv    esi
.text:004010BA                 push    eax
.text:004010BB                 mov     eax, [ebp+var_4]
.text:004010BE                 cdq
.text:004010BF                 sub     eax, edx
.text:004010C1                 sar     eax, 1
.text:004010C3                 push    eax
.text:004010C4                 mov     eax, esi
.text:004010C6                 cdq
.text:004010C7                 and     edx, 3
.text:004010CA                 add     eax, edx
.text:004010CC                 sar     eax, 2
.text:004010CF                 push    eax
.text:004010D0                 mov     eax, esi
.text:004010D2                 cdq
.text:004010D3                 and     edx, 7
.text:004010D6                 add     eax, edx
.text:004010D8                 sar     eax, 3
.text:004010DB                 push    eax

.text:004010DC                 push    offset aDDDDD   ; "%d%d%d%d%d"
.text:004010E1                 call    sub_401020
.text:004010E6                 push    offset aPause   ; "pause"
.text:004010EB                 call    sub_4048C7
.text:004010F0                 add     esp, 28h
.text:004010F3                 xor     eax, eax
.text:004010F5                 pop     esi
.text:004010F6                 mov     esp, ebp
.text:004010F8                 pop     ebp
.text:004010F9                 retn
.text:004010F9 sub_401080      endp
```

可以看到，同 `vc6.0` 的代码并没有什么变化。因为优化是根据数学定理优化的，除非数学变了，否则不会发生改变。

### 5.2 x86下除数为负数2的幂 高级代码与反汇编

高级代码

```c
int main(int argc, char* argv[])
{
    /* 除法 */
    int NumberOne = 0;
    int NumberTwo = 0;
    scanf("%d", &NumberOne);
    scanf("%d", &NumberTwo);

    int Count1 = NumberOne / NumberTwo;
    int Count2 = NumberOne / -2;
    int Count3 = NumberTwo / -4;
    int Count5 = NumberTwo / -8;

    printf("%d%d%d%d%d", Count5, Count3, Count2, Count1);
    system("pause");

    return 0;
}
```

汇编代码
