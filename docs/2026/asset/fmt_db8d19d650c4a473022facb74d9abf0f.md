
```asm
00511A6D  |.  E8 EE22EFFF     call    00403D60
00511A72  |>  8905 D8BF5100   mov     [51BFD8], eax                  ;  EAX(一系列运算完毕的16进制)送入[51BFD8]
00511A78  |.  8915 DCBF5100   mov     [51BFDC], edx
00511A7E  |.  833D DCBF5100 0>cmp     dword ptr [51BFDC], 0
00511A85  |.  75 0E           jnz     short 00511A95
00511A87  |.  813D D8BF5100 F>cmp     dword ptr [51BFD8], 3B9AC9FF   ;  真码16进制与"3B9AC9FF"作对比
00511A91  |.  76 2F           jbe     short 00511AC2                 ;  小于就跳
00511A93  |.  EB 02           jmp     short 00511A97
00511A95  |>  7E 2B           jle     short 00511AC2
00511A97  |>  6A 00           push    0
00511A99  |.  68 00CA9A3B     push    3B9ACA00
00511A9E  |.  8B05 D8BF5100   mov     eax, [51BFD8]
00511AA4  |.  8B15 DCBF5100   mov     edx, [51BFDC]
00511AAA  |.  E8 1143EFFF     call    00405DC0
00511AAF  |.  71 05           jno     short 00511AB6
00511AB1  |.  E8 AA22EFFF     call    00403D60
00511AB6  |>  8905 D8BF5100   mov     [51BFD8], eax
00511ABC  |.  8915 DCBF5100   mov     [51BFDC], edx
00511AC2  |>  33C0            xor     eax, eax
00511AC4  |.  5A              pop     edx
00511AC5  |.  59              pop     ecx
00511AC6  |.  59              pop     ecx
00511AC7  |.  64:8910         mov     fs:[eax], edx
00511ACA  |.  68 E41A5100     push    00511AE4
00511ACF  |>  8D45 F4         lea     eax, [ebp-C]
00511AD2  |.  BA 03000000     mov     edx, 3
00511AD7  |.  E8 5831EFFF     call    00404C34
00511ADC  \.  C3              retn                                   ;  返回
```

---

## 算法总结

1. 硬盘序列号必须为14位，可记为`A`
2. 取`A`的每一位ASCII值与`0FF`做`and`运算再加上7，最后全部累加为16进制，可记为`B`
3. 然后把`B`转换为10进制，即是注册码

### 后记

1. 其实在破解的时候：触发事件一断下来就给了我们真码的16进制。（当时没太在意，可一直找不出关键`call`，然后就一层层`call`跟进，最后找到正确的算法核心）
2. 以下出现一系列的`ascii"999999999999"`及`ascii"0000000000"`都没有与算法搭上边，当时被这些迷惑住了，呵呵~~~
3. 感谢你看到此处~~很简单的一个算法~ 其他也没啥好说的了。有不当之处，请各位大侠多多指正~~~

---

### 版权声明

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
