
# 从逆向工程的角度来看 C++ (四)

> 原文：[`www.pediy.com/kssd/pediy10/87590.html`](https://www.pediy.com/kssd/pediy10/87590.html)

进入本次正题：

(四) C++ 之 流程控制

### 源码示例

```cpp
// Lesson4.cpp : Defines the entry point for the console application.

#include "stdafx.h"
#include <stdio.h>

int main(int argc, char* argv[])
{
    int i;
    int max = 0x100;
    int sum = 0;

    __asm int 3

    // for loop
    for (i = 0; i < max; i++)
    {
        sum += i;
    }

    // while do
    while (max-- > 0)
    {
        sum += max;
    }

    max = 0x99;
    do
    {
        sum -= max;
        i += sum;
    } while (max-- > 10);

    return 0;
}
```

### 汇编分析

```asm
// Dasm
00401010  > 55               push     ebp
00401011    8BEC             mov      ebp, esp
00401013    83EC 4C          sub      esp, 4C
00401016    53               push     ebx
00401017    56               push     esi
00401018    57               push     edi
00401019    8D7D B4          lea      edi, dword ptr ss:[ebp-4C]
0040101C    B9 13000000      mov      ecx, 13
00401021    B8 CCCCCCCCC     mov      eax, CCCCCCCC
00401026    F3:AB            rep stos dword ptr es:[edi]
00401028    C745 F8 0001000  mov      dword ptr ss:[ebp-8], 100
0040102F    C745 F4 0000000  mov      dword ptr ss:[ebp-C], 0
00401036    90               nop
00401037    C745 FC 0000000  mov      dword ptr ss:[ebp-4], 0            ; for 循环的初始条件弄好,再 jmp 过去   i=0
0040103E    EB 09            jmp      short Lesson4.00401049             ; jmp 到循环块.
00401040    8B45 FC          mov      eax, dword ptr ss:[ebp-4]          ; 这个块的开始.
00401043    83C0 01          add      eax, 1
00401046    8945 FC          mov      dword ptr ss:[ebp-4], eax          ; i++
00401049    8B4D FC          mov      ecx, dword ptr ss:[ebp-4]
0040104C    3B4D F8          cmp      ecx, dword ptr ss:[ebp-8]           ; i > max ?
0040104F    7D 0B            jge      short Lesson4.0040105C             ; 跳出循环,执行完毕
00401051    8B55 F4          mov      edx, dword ptr ss:[ebp-C]
00401054    0355 FC          add      edx, dword ptr ss:[ebp-4]
00401057    8955 F4          mov      dword ptr ss:[ebp-C], edx          ; sum +=i
0040105A   ^ EB E4          jmp      short Lesson4.00401040
0040105C    837D F8 00       cmp      dword ptr ss:[ebp-8], 0            ; while(max>0) ?
00401060    7E 0B            jle      short Lesson4.0040106D
00401062    8B45 F4          mov      eax, dword ptr ss:[ebp-C]
00401065    0345 F8          add      eax, dword ptr ss:[ebp-8]
00401068    8945 F4          mov      dword ptr ss:[ebp-C], eax          ; sum +=max
0040106B   ^ EB EF          jmp      short Lesson4.0040105C             ; 呵呵, 忘了 max--
0040106D    C745 F8 9900000  mov      dword ptr ss:[ebp-8], 99
00401074    8B4D F4          mov      ecx, dword ptr ss:[ebp-C]
00401077    2B4D F8          sub      ecx, dword ptr ss:[ebp-8]
0040107A    894D F4          mov      dword ptr ss:[ebp-C], ecx          ; sum -=max
0040107D    8B55 FC          mov      edx, dword ptr ss:[ebp-4]
00401080    0355 F4          add      edx, dword ptr ss:[ebp-C]
00401083    8955 FC          mov      dword ptr ss:[ebp-4], edx          ; i += sum
00401086    837D F8 0A       cmp      dword ptr ss:[ebp-8], 0A           ; max>10 ?
0040108A   ^ 7F E8          jg       short Lesson4.00401074             ; 又忘了 max-- .呵呵
0040108C    33C0             xor      eax, eax
0040108E    5F               pop      edi
0040108F    5E               pop      esi
00401090    5B               pop      ebx
00401091    83C4 4C          add      esp, 4C
00401094    3BEC             cmp      ebp, esp
00401096    E8 D5C30000      call     Lesson4._chkesp
0040109B    8BE5             mov      esp, ebp
0040109D    5D               pop      ebp
0040109E    C3               retn
```

来看下 Release 版本的.

```asm
00401002    90               nop
00401003    33C9             xor      ecx, ecx
00401005    03C1             add      eax, ecx
00401007    41               inc      ecx
00401008    81F9 00010000    cmp      ecx, 100
0040100E   ^ 7C F5          jl       short Lesson4.00401005
00401010    56               push     esi
00401011    BA FF000000      mov      edx, 0FF
00401016    03C2             add      eax, edx
00401018    8BF2             mov      esi, edx
0040101A    4A               dec      edx
0040101B    85F6             test     esi, esi
0040101D   ^ 7F F7          jg       short Lesson4.00401016
0040101F    BA 99000000      mov      edx, 99
00401024    2BC2             sub      eax, edx
00401026    8BF2             mov      esi, edx
00401028    03C8             add      ecx, eax
0040102A    4A               dec      edx
0040102B    83FE 0A          cmp      esi, 0A
0040102E   ^ 7F F4          jg       short Lesson4.00401024
00401030    33C0             xor      eax, eax
00401032    5E               pop      esi
00401033    C3               retn
```

这几个循环太简单了,懒得解释.

### 小结

其实对于循环控制的逆向没什么好研究的, 传统的 `if` `else` 什么的, 都是 `j**`的指令, 循环控制的一般也很轻易看出来, 还有 `switch` 语句也都容易看出来, 就不多讲什么了.

### PS

这里追加一点其他的知识:

1. 缺省参数只能放在函数声明中
2. 函数重载是 `C++`的特性, 不要刻意去和对象相联系, 重载而已, just so so !
3. 引用传参的一般表现形式是指针传值, 但是不代表全部. 引用就是"引用"了.

引用上了就绑死了, 一改皆改.如 : `int a =9; int b = 8; int &c = b; c = a ;` 执行 result: `a = b = c = 9 ;`
