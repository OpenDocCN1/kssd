
# 大管家工资管理系统 740

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-165.htm`](https://www.pediy.com/kssd/pediy07/pediy7-165.htm)

大管家通用人事工资管理系统的注册分析——`F1(序列号）=F2（注册码）`

【破解作者】`jsliyangsj`

【作者邮箱】`sjcrack@yahoo.com.cn`

【使用工具】`peid` `OllyDbg1.10`

【破解平台】`Winxp`

【软件名称】`大管家通用人事工资管理系统_ 专业版 740`

【软件地址】`http://www.lansom.com/`

软件有壳，`ASPack 2.12 -> Alexey Solodovnikov`，脱壳后，点击“系统注册”，发现程序打开了新进程 `Hrms`。

在 `Hrms` 中查找字符串能直接找到错误提示“注册码错误……”。

对于 `F1(序列号）=F2（注册码）`的软件只要逆转一个就够了，这里逆转的是 `F1(序列号）`就是用注册码得到序列号。

分析：

```
08A0A2D2    .  8B>mov eax,dword ptr ds:[eax+2D8]
08A0A2D8    .  E8>call Hrms.087C7A28                 ;  得到序列号
08A0A2DD    .  8B>mov eax,dword ptr ss:[ebp-14]
08A0A2E0    .  E8>call Hrms.0879425C                 ;  得到序列号的位数
08A0A2E5    .  83>cmp eax,4                         ;  位数一定要大于等于 4
08A0A2E8    .  7D>jge short Hrms.08A0A318
08A0A2EA    .  6A>push 10
08A0A2EC    .  B9>mov ecx,Hrms.08A0A610
08A0A2F1    .  BA>mov edx,Hrms.08A0A618
08A0A2F6    .  A1>mov eax,dword ptr ds:[8A19470]
08A0A2FB    .  8B>mov eax,dword ptr ds:[eax]
08A0A2FD    .  E8>call Hrms.087E7400                 ;  如果小于 4，到这里出错了！！
08A0A302    .  8B>mov eax,dword ptr ss:[ebp-4]
………………省略……………………
08A0A32C    .  8B>mov eax,dword ptr ds:[eax+2DC]
08A0A332    .  E8>call Hrms.087C7A28                 ;  得到第 1 个注册码返回注册码 1 的位数
08A0A337    .  8D>lea eax,dword ptr ss:[ebp-18]
08A0A33A    .  50 push eax
08A0A33B    .  8D>lea edx,dword ptr ss:[ebp-1C]
08A0A33E    .  8B>mov eax,dword ptr ss:[ebp-4]
08A0A341    .  8B>mov eax,dword ptr ds:[eax+2E0]
08A0A347    .  E8>call Hrms.087C7A28                 ;  得到第 2 个注册码返回注册码 2 的位数
08A0A34C    .  8B>mov edx,dword ptr ss:[ebp-1C]
08A0A34F    .  58 pop eax
08A0A350    .  E8>call Hrms.08794264                 ;  把 1，2 注册码组合，共 8 位先放后几位注册码 2，前几位不足用注册码填，多余去掉注册码 1 前面的
08A0A355    .  8B>mov edx,dword ptr ss:[ebp-18]
08A0A358    .  A1>mov eax,dword ptr ds:[8A1977C]
08A0A35D    .  8B>mov eax,dword ptr ds:[eax]
08A0A35F    .  E8>call Hrms.089F0E64                 ;  取合并后的注册码的前 8 位当做 16 进制（不是转化）
08A0A364    .  89>mov dword ptr ss:[ebp-C],eax       ;  储存在 EBP-C
08A0A367    .  8D>lea edx,dword ptr ss:[ebp-20]
08A0A36A    .  8B>mov eax,dword ptr ss:[ebp-4]
08A0A36D    .  8B>mov eax,dword ptr ds:[eax+2E4]
08A0A373    .  E8>call Hrms.087C7A28                 ;  得到第 3 个注册码返回注册码 3 的位数
08A0A378    .  8D>lea eax,dword ptr ss:[ebp-20]
08A0A37B    .  50 push eax
08A0A37C    .  8D>lea edx,dword ptr ss:[ebp-24]
08A0A37F    .  8B>mov eax,dword ptr ss:[ebp-4]
08A0A382    .  8B>mov eax,dword ptr ds:[eax+2EC]
08A0A388    .  E8>call Hrms.087C7A28                 ;  得到第 4 个注册码返回注册码 4 的位数
08A0A38D    .  8B>mov edx,dword ptr ss:[ebp-24]
08A0A390    .  58 pop eax
08A0A391    .  E8>call Hrms.08794264                 ;  把 3，4 注册码组合，共 8 位先放后几位注册码 4，前几位不足用注册码 3 填，多余去掉注册码 3 前面的
08A0A396    .  8B>mov edx,dword ptr ss:[ebp-20]
08A0A399    .  A1>mov eax,dword ptr ds:[8A1977C]
08A0A39E    .  8B>mov eax,dword ptr ds:[eax]
08A0A3A0    .  E8>call Hrms.089F0E64                 ;  取合并后的注册码的前 8 位当做 16 进制（不是转化）
08A0A3A5    .  89>mov dword ptr ss:[ebp-8],eax       ;  储存在 EBP-8
08A0A3A8    .  33>xor eax,eax
………………………省略…………………………
08A0A3EA    .  E8>call Hrms.08793A08
08A0A3EF    >  8D>lea eax,dword ptr ss:[ebp-C]       ;  下面是关键
08A0A3F2    .  50 push eax
08A0A3F3    .  8D>lea edx,dword ptr ss:[ebp-28]
08A0A3F6    .  8B>mov eax,dword ptr ss:[ebp-4]
08A0A3F9    .  8B>mov eax,dword ptr ds:[eax+2D8]
08A0A3FF    .  E8>call Hrms.087C7A28                 ;  再次得到序列号
08A0A404    .  8B>mov edx,dword ptr ss:[ebp-28]
08A0A407    .  8D>lea ecx,dword ptr ss:[ebp-E]
08A0A40A    .  A1>mov eax,dword ptr ds:[8A1977C]
08A0A40F    .  8B>mov eax,dword ptr ds:[eax]
08A0A411    .  E8>call Hrms.089F0BDC                 ;  关键点， 进入
…………………………………………………………………………………………………………………………
```

进入 `08A0A411 . E8>call Hrms.089F0BDC`

…………………………………………………………………………………………………………………………

```
089F0C0E  |.  64>mov dword ptr fs:[eax],esp           ;  下面是判断用哪个字符串作为参数用的！
089F0C11  |.  A1>mov eax,dword ptr ds:[8A1C468]       ;  固定值 9DD
089F0C16  |.  B9>mov ecx,3E8                          ;  固定的 3E8
089F0C1B  |.  99 cdq
089F0C1C  |.  F7>idiv ecx                             ;  9DD 除以 3E8
089F0C1E  |.  48 dec eax                               ;  减 1; Switch (cases 1..4)
089F0C1F  |.  74>je short Hrms.089F0C2C
089F0C21  |.  48 dec eax                               ;  再减 1 必为 0
089F0C22  |.  74>je short Hrms.089F0C3B
089F0C24  |.  48 dec eax
089F0C25  |.  74>je short Hrms.089F0C4A
089F0C27  |.  48 dec eax
089F0C28  |.  74>je short Hrms.089F0C59
089F0C2A  |.  EB>jmp short Hrms.089F0C66
089F0C2C  |>  8D>lea eax,dword ptr ss:[ebp-18]         ;  Case 1 of switch 089F0C1E
089F0C2F  |.  BA>mov edx,Hrms.089F0DD4                ;  ASCII "Oh,You are fat pig!"
089F0C34  |.  E8>call Hrms.08794074
089F0C39  |.  EB>jmp short Hrms.089F0C66
089F0C3B  |>  8D>lea eax,dword ptr ss:[ebp-18]         ;  Case 2 of switch 089F0C1E
089F0C3E  |.  BA>mov edx,Hrms.089F0DF0                ;  字符串作为参数 Ha...ha,You arefat pig!
089F0C43  |.  E8>call Hrms.08794074
089F0C48  |.  EB>jmp short Hrms.089F0C66
089F0C4A  |>  8D>lea eax,dword ptr ss:[ebp-18]         ;  Case 3 of switch 089F0C1E
089F0C4D  |.  BA>mov edx,Hrms.089F0E14                ;  ASCII "Oh,grandeur for company!"
089F0C52  |.  E8>call Hrms.08794074
089F0C57  |.  EB>jmp short Hrms.089F0C66
089F0C59  |>  8D>lea eax,dword ptr ss:[ebp-18]         ;  Case 4 of switch 089F0C1E
089F0C5C  |.  BA>mov edx,Hrms.089F0E38                ;  ASCII "Oh,grandeur for rsglpro!"
089F0C61  |.  E8>call Hrms.08794074
089F0C66  |>  8B>mov eax,dword ptr ss:[ebp-8]          ;  取出序列号地址; Default case of switch 089F0C1E
089F0C69  |.  E8>call Hrms.0879425C                   ;  得到序列号的位数
089F0C6E  |.  25>and eax,80000007                     ;  与 7 相乘
089F0C73  |.  79>jns short Hrms.089F0C7A
089F0C75  |.  48 dec eax
089F0C76  |.  83>or eax,FFFFFFF8
089F0C79  |.  40 inc eax
089F0C7A  |>  85>test eax,eax
089F0C7C  |.  74>je short Hrms.089F0CAA
089F0C7E  |.  8D>lea eax,dword ptr ss:[ebp-8]
089F0C81  |.  BA>mov edx,Hrms.089F0E5C                ;  得到固定值 1234567
089F0C86  |.  E8>call Hrms.08794264                   ;  把序列号与固定值合并 73065321234567
089F0C8B  |.  8B>mov eax,dword ptr ss:[ebp-8]          ;  得到地址
089F0C8E  |.  E8>call Hrms.0879425C                   ;  得到合并之后的位数
089F0C93  |.  85>test eax,eax
089F0C95  |.  79>jns short Hrms.089F0C9A
089F0C97  |.  83>add eax,7
089F0C9A  |>  C1>sar eax,3
089F0C9D  |.  8B>mov edx,eax
089F0C9F  |.  C1>shl edx,3                            ;  左移动 3
089F0CA2  |.  8D>lea eax,dword ptr ss:[ebp-8]          ;  合并后的地址
089F0CA5  |.  E8>call Hrms.08794590                   ;  将第 9 位 0  上面一段的意思是：如果你输入的小于 8 的倍数则后面不足的就用 1234567 补足。取 8 的倍数位
089F0CAA  |>  33>xor esi,esi
089F0CAC  |.  8D>lea eax,dword ptr ss:[ebp-8]
089F0CAF  |.  E8>call Hrms.0879442C
089F0CB4  |.  8B>mov edi,eax
089F0CB6  |.  8D>lea eax,dword ptr ss:[ebp-18]
089F0CB9  |.  E8>call Hrms.0879442C                   ;  得到字符串参数的 Ha...ha,You arefat pig!的地址
089F0CBE  |.  8B>mov ebx,eax
089F0CC0  |.  EB>jmp short Hrms.089F0CF5
089F0CC2  |>  8B>/mov eax,dword ptr ds:[edi+esi*4]    ;  得到合并取前 8 位的前 4 位的 ASCII 码反过来放的
089F0CC5  |.  89>|mov dword ptr ss:[ebp-20],eax        ;  储存
089F0CC8  |.  8B>|mov eax,dword ptr ds:[edi+esi*4+4]  ;  取后四位反着放
089F0CCC  |.  89>|mov dword ptr ss:[ebp-1C],eax        ;  储存
089F0CCF  |.  8B>|mov ecx,ebx                           ;  得到字符串
089F0CD1  |.  8D>|lea edx,dword ptr ss:[ebp-20]
```
