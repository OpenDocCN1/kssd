
02475521  |.  57            push    edi                                       ; /n => 10 (16.)
02475522  |.  6A 5C         push    5C                                        ; |c = 5C  ('\')
02475524  |.  56            push    esi                                       ; |s
02475525  |.  E8 84220200   call    <jmp.&MSVCR90.memset>                     ; \memset
0247552A  |.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]
0247552D  |.  83C4 0C       add     esp, 0C
02475530  |.  8BC6          mov     eax, esi
02475532  |.  2BCE          sub     ecx, esi
02475534  |.  8BD7          mov     edx, edi
02475536  |>  8A1C01        /mov     bl, byte ptr ds:[ecx+eax]
02475539  |.  3018          |xor     byte ptr ds:[eax], bl
0247553B  |.  40            |inc     eax
0247553C  |.  4A            |dec     edx
0247553D  |.^ 75 F7         \jnz     short 02475536
0247553F  |.  57            push    edi
02475540  |.  8D45 EC       lea     eax, dword ptr ss:[ebp-14]
02475543  |.  50            push    eax
02475544  |.  8D46 10       lea     eax, dword ptr ds:[esi+10]           复制第一处md5 hash的值
02475547  |.  57            push    edi
02475548  |.  50            push    eax
02475549  |.  FF15 BCF46502 call    dword ptr ds:[<&MSVCR90.memmove_s>]       ;  MSVCR90.memmove_s
0247554F  |.  83C4 10       add     esp, 10
02475552  |.  FF75 DC       push    dword ptr ss:[ebp-24]                     ; /Arg3
02475555  |.  6A 20         push    20                                        ; |Arg2 = 00000020
02475557  |.  56            push    esi                                       ; |Arg1
02475558  |.  E8 D9FEFFFF   call    02475436                                  md5

再进入`02493660`，这里主要将hash后的结果，前8位，与偶数的位异或，得到一个8字节的数组

```
02493678  |> /8A1C4F        /mov     bl, byte ptr ds:[edi+ecx*2]     取索引*2的值
0249367B  |. |321C06        |xor     bl, byte ptr ds:[esi+eax]            取当前索引的值，并异或
0249367E  |. |41            |inc     ecx
0249367F  |. |8818          |mov     byte ptr ds:[eax], bl          赋值
02493681  |. |40            |inc     eax
02493682  |. |3BCA          |cmp     ecx, edx                         循环8次
02493684  |.^\72 F2         \jb      short 02493678
```

最后返回到`024939AD`对比结果。

## 加密思路

1. 对输入的`Serial code`后21个字符数组转换为14个字节数组，14个字节数组简称`A`，且最后一位要求为0，具体算法看`02493AE9`的分析。
2. 输入的`Serial code`前7个字符调用`02492CF8`，产生`BlowFish`的`Key`，再调用`BlowFish Init`；
3. `Serial code`前7个字符与`A`组成新的数组`B`，20个字节，对`B`调用`BlowFish Decrypt`，还原第8个字节起的8个字节，产生`C`；
4. 数组`D`复制`C`数组，然后最后8个字节清零，然后调用`02475481`，产生一个hash，然后再调用`02493660`，将hash变为一个8字节的数组，赋值到数组`D`的最后8个字节。
5. 数组`D`与数组`C`比较，要求值一致。

## 解密思路

1. 用字符串`"DR15"` + 随机生成8位字节组成12位的数组`A`，再加上8个值为零的字节组成20个字节数组`B`；
2. 用`B`模拟调用`02475481`，产生一个hash，然后再模拟调用`02493660`，将hash变为一个8字节的数组，然后将此数组与`A`合并组成`C`；
3. 取`C`的前7个字节模拟调用`02492CF8`，产生`BlowFish`的`Key`，再调用`BlowFish Init`；
4. 对`C`第8个字节起的8个字节，调用`BlowFish Encrypt`，加密8个字节；
5. 对`C`第8个字节起的13个字节，加上一个值为零的字节，组成14个字节，用到105位的数据，要还原21个字符，每一次取5位的数据，按当前值的范围还原出原来的字符（还原规则看`02493AE9`），最后组成`Serial code`。

## 结语

这篇文章，只是列出关键的处理过程，只是向大家说明一个加密思路，细致的跟踪留给感兴趣的朋友们。

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢！

2011 年 11月 16日  by Daniel

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
