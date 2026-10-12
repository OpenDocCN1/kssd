
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



## 关键调用点分析

```
089F0CD4   |.  8B>|mov eax,dword ptr ss:\[ebp-4\]  

089F0CD7   |.  E8>|call Hrms.089F0B74                 ;  循环处理序列号的地方    进入关键点  

089F0CDC   |.  8B>|mov eax,dword ptr ds:\[ebx\]  

089F0CDE   |.  89>|mov dword ptr ds:\[ebx+8\],eax  

089F0CE1   |.  8B>|mov eax,dword ptr ds:\[ebx+4\]  

089F0CE4   |.  89>|mov dword ptr ds:\[ebx+C\],eax  

089F0CE7   |.  8B>|mov eax,dword ptr ss:\[ebp-20\]     ;  取刚才计算出来的数据  

089F0CEA   |.  89>|mov dword ptr ds:\[ebx\],eax  

089F0CEC   |.  8B>|mov eax,dword ptr ss:\[ebp-1C\]     ;    

089F0CEF   |.  89>|mov dword ptr ds:\[ebx+4\],eax  

089F0CF2   |.  83>|add esi,2  

089F0CF5   |>  8B> mov eax,dword ptr ss:\[ebp-8\]  

089F0CF8   |.  E8>|call Hrms.0879425C                 ;  得到序列号处理后 8 倍数的位数  

089F0CFD   |.  85>|test eax,eax  

089F0CFF   |.  79>|jns short Hrms.089F0D04  

089F0D01   |.  83>|add eax,3  

089F0D04   |>  C1>|sar eax,2  

089F0D07   |.  3B>|cmp esi,eax  

089F0D09   |.^ 72>\jb short Hrms.089F0CC2             ;  上面一段意思：用来计算序列号的，每次取 8 位计算，以最后一次 8 位序列号计算为最后结果  

089F0D0B   |.  8B>mov eax,dword ptr ss:\[ebp-14\]     ;  得到前 2 个注册码合并后的  

089F0D0E   |.  33>xor edx,edx                          ;  下面 2 段用来处理注册码的  

089F0D10   |.  52 push edx                              ; /Arg6 => 00000000  

089F0D11   |.  50 push eax                              ; |Arg5  

089F0D12   |.  FF>push dword ptr ds:\[8A18C8C\]        ; |Arg4 = 00000000  

089F0D18   |.  FF>push dword ptr ds:\[8A18C88\]        ; |Arg3 = 3F17DC15  

089F0D1E   |.  FF>push dword ptr ds:\[8A18C94\]        ; |Arg2 = 00000000  

089F0D24   |.  FF>push dword ptr ds:\[8A18C90\]        ; |Arg1 = 758F0581  

089F0D2A   |.  8B>mov eax,dword ptr ss:\[ebp-4\]       ; |  

089F0D2D   |.  E8>call Hrms.089F0AB0                  ; \关键点 2  进入  处理注册码 1 与注册码 2 合并后的  

089F0D32   |.  83>sub eax,2                           ;  结果减 2  

089F0D35   |.  89>mov dword ptr ss:\[ebp-28\],eax  

089F0D38   |.  8B>mov eax,dword ptr ss:\[ebp-10\]  

089F0D3B   |.  33>xor edx,edx  

089F0D3D   |.  52 push edx                              ; /Arg6 => 00000000  

089F0D3E   |.  50 push eax                              ; |Arg5  

089F0D3F   |.  FF>push dword ptr ds:\[8A18C8C\]        ; |Arg4 = 00000000  

089F0D45   |.  FF>push dword ptr ds:\[8A18C88\]        ; |Arg3 = 3F17DC15  

089F0D4B   |.  FF>push dword ptr ds:\[8A18C94\]        ; |Arg2 = 00000000  

089F0D51   |.  FF>push dword ptr ds:\[8A18C90\]        ; |Arg1 = 758F0581  

089F0D57   |.  8B>mov eax,dword ptr ss:\[ebp-4\]       ; |  

089F0D5A   |.  E8>call Hrms.089F0AB0                  ; \关键点 3  进入  处理注册码 3 与注册码 4 合并后的  

089F0D5F   |.  83>sub eax,2                           ;  结果减 2  

089F0D62   |.  89>mov dword ptr ss:\[ebp-24\],eax  

089F0D65   |.  C1>shl dword ptr ss:\[ebp-28\],2       ;  用注册码 1、2 处理后的结果左移 2 得到结果 1  

089F0D69   |.  8D>lea ecx,dword ptr ss:\[ebp-28\]  

089F0D6C   |.  8B>mov eax,dword ptr ds:\[ecx\]         ;  得到处理后的结果 1  

089F0D6E   |.  8B>mov edx,dword ptr ds:\[ecx+4\]       ;  得到注册码 3、4 处理后的数据  

089F0D71   |.  0F>shrd eax,edx,2                      ;  双精度数右移  

089F0D75   |.  C1>shr edx,2                           ;  得到注册码 3、4 处理后的数据右移 2 得到结果 2  

089F0D78   |.  89>mov dword ptr ds:\[ecx\],eax  

089F0D7A   |.  89>mov dword ptr ds:\[ecx+4\],edx  

089F0D7D   |.  8B>mov eax,dword ptr ss:\[ebp-28\]  

089F0D80   |.  3B>cmp eax,dword ptr ss:\[ebp-20\]      ;  比较   用注册码双精度数右移后的结果与序列号处理后的结果比较  

089F0D83   |.  74>je short Hrms.089F0D89               ;  相等后再去计算终端数  

089F0D85   |.  33>xor ebx,ebx  

089F0D87   |.  EB>jmp short Hrms.089F0D9A  

089F0D89   |>  66>mov ax,word ptr ss:\[ebp-24\]        ;  取结果 2  

089F0D8D   |.  66>and ax,0FFFF                        ;  取注册码 3、4 处理后的结果 2 的后 16 进制后 4 位作为终端数  

089F0D91   |.  8B>mov edx,dword ptr ss:\[ebp-C\]  

089F0D94   |.  66>mov word ptr ds:\[edx\],ax  

089F0D97   |.  83>or ebx,FFFFFFFF  

089F0D9A   |>  33>xor eax,eax  

089F0D9C   |.  5A pop edx  

089F0D9D   |.  59 pop ecx  

089F0D9E   |.  59 pop ecx  

089F0D9F   |.  64>mov dword ptr fs:\[eax\],edx  

089F0DA2   |.  68>push Hrms.089F0DBF  

089F0DA7   |>  8D>lea eax,dword ptr ss:\[ebp-18\]  

089F0DAA   |.  E8>call Hrms.08793FDC  

089F0DAF   |.  8D>lea eax,dword ptr ss:\[ebp-8\]  

089F0DB2   |.  E8>call Hrms.08793FDC  

089F0DB7   \.  C3 retn  
```

进入 `089F0CD7 |. E8>|call Hrms.089F0B74 ; 循环处理序列号的地方` 是可逆的！！

## 序列号循环处理函数 (Hrms.089F0B74)

```
089F0B74   /$  53 push ebx  

089F0B75   |.  56 push esi  

089F0B76   |.  57 push edi  

089F0B77   |.  51 push ecx  

089F0B78   |.  89>mov dword ptr ss:\[esp\],edx  

089F0B7B   |.  BA>mov edx,20                     ;  参数 20 要循环 20（16 进制）次  

089F0B80   |.  8B>mov eax,dword ptr ss:\[esp\]  

089F0B83   |.  8B>mov eax,dword ptr ds:\[eax\]    ;  得到前 4 位 ASCII 码反  36303337  

089F0B85   |.  8B>mov ebx,dword ptr ss:\[esp\]  

089F0B88   |.  8B>mov ebx,dword ptr ds:\[ebx+4\]   ;  得到后四位 ASCII 码反  31323335  

089F0B8B   |.  33>xor esi,esi  

089F0B8D   |>  4A /dec edx                       ;  参数减 1  

089F0B8E      03>add esi,dword ptr ds:\[8A18C84\] ;  固定脂 9E3779B9  

089F0B94      8B>mov edi,ebx  

089F0B96   |.  C1>|shl edi,4                     ;  结果左移 4 位  

089F0B99      03>add eax,edi                     ;  与前 4 位相加  

089F0B9B   |.  8B>|mov edi,dword ptr ds:\[ecx\]   ;  取固定字符串的前 4 位 ASCII 码  2E2E6148  

089F0B9D   |.  33>|xor edi,ebx                   ;  取固定字符串的前 4 位 ASCII 码与后 4 位 XOR  

089F0B9F      03>add eax,edi                     ;  再加到上面的和  

089F0BA1   |.  8B>|mov edi,ebx  

089F0BA3   |.  C1>|shr edi,5                     ;  后 4 位再右移 5  

089F0BA6   |.  33>|xor edi,esi                   ;  后 4 位再右移 5 与固定值 XOR  

089F0BA8      03>add eax,edi                     ;  再加上取  

089F0BAA      03>add eax,dword ptr ds:\[ecx+4\]   ;  加上固定字符串的 5 到 8 位   2C61682E  

089F0BAD   |.  8B>|mov edi,eax  

089F0BAF   |.  C1>|shl edi,4                     ;  得到的结果左移 4  

089F0BB2      03>add ebx,edi                     ;  加上后 4 位  

089F0BB4      8B>mov edi,dword ptr ds:\[ecx+8\]   ;  再取固定字符串的 ASCII20756F59  

089F0BB7   |.  33>|xor edi,eax                   ;  固定字符串的 ASCII 码与计算好的 EAXXOR  

089F0BB9      03>add ebx,edi                     ;  结果加在 EBX 上 改变 EBX  

089F0BBB   |.  8B>|mov edi,eax  

089F0BBD   |.  C1>|shr edi,5                     ;  把上面计算好的 EAX 右移 5 位  

089F0BC0   |.  33>|xor edi,esi                   ;  结果与 9E3779B9 累加（每次这个数加自己）XOR  

089F0BC2      03>add ebx,edi                     ;  计算的结果加在 EBX 上  

089F0BC4      03>add ebx,dword ptr ds:\[ecx+C\]   ;  EBX 再加上固定字符串的 ASCII20657261  

089F0BC7   |.  85>|test edx,edx  

089F0BC9   |.^ 77>\ja short Hrms.089F0B8D  

089F0BCB   |.  8B>mov edx,dword ptr ss:\[esp\]  

089F0BCE   |.  89>mov dword ptr ds:\[edx\],eax    ;  储存了上面计算结果一半  

089F0BD0   |.  8B>mov eax,dword ptr ss:\[esp\]  

089F0BD3   |.  89>mov dword ptr ds:\[eax+4\],ebx   ;  储存了上面计算结果  

089F0BD6   |.  5A pop edx  

089F0BD7   |.  5F pop edi  

089F0BD8   |.  5E pop esi  

089F0BD9   |.  5B pop ebx  

089F0BDA   \.  C3 retn  
```

此函数是可逆的！！也可以直接在 `OD` 中改代码达到目的！循环 32 次！

## 注册码处理函数 (Hrms.089F0AB0)

进入处理注册码 1、2 的地方 注册码 3、4 同一个函数！，`089F0D2D |. E8>call Hrms.089F0AB0 ; \关键点 2 进入`

```
089F0AB9   |.  C7>mov dword ptr ss:\[ebp-8\],1  

089F0AC0   |.  C7>mov dword ptr ss:\[ebp-4\],0  

089F0AC7   |.  E9>jmp Hrms.089F0B4C  

089F0ACC   |>  6A>/push 0  

089F0ACE   |.  6A>|push 2  

089F0AD0   |.  8B>|mov eax,dword ptr ss:\[ebp+10\]    ;  参数之一 最初 3F17DC15 以后的值已经修改  

089F0AD3   |.  8B>|mov edx,dword ptr ss:\[ebp+14\]    ;  参数之 2  最初为 0  

089F0AD6   |.  E8>|call Hrms.087978A5                ;  得到了 EAX 除以 2 的余数  

089F0ADB   |.  83>|cmp edx,0                          ;  固定不为 0  
```



### 算法逻辑分析

这里处理注册码的函数中总共包含两个不同算法（依赖与固定值参数初始值 `0x3F17DC15`）：

1.  每次固定值除以 2 如果没有余数（也就是偶数）并把固定值除以 2 覆盖原来：
    *   输入的注册码（合并后）乘以自己（也就是平方），然后再进行循环处理。
2.  如果固定值除以 2 没有余数（也就是奇数）则固定值-1：
    *   输入的注册码（合并后）乘以上一次这里计算出来的值，然后再进行循环处理。

这里的乘法要涉及到大数相乘。

再进入上面算法的 `call Hrms.089F0A58`。

### 汇编代码：主调用流程

```assembly
089F0ADE   |.  75>|jnz short Hrms.089F0B1B  
089F0AE0   |.  83>|cmp eax,0  
089F0AE3   |.  75>|jnz short Hrms.089F0B1B             ;  余数不为 0 就跳，就是说奇数就跳另一种计算  
089F0AE5   |.  6A>|push 0                              ;  偶数到这里  压入 0  
089F0AE7   |.  6A>|push 2                              ;  压入 2  
089F0AE9   |.  8B>|mov eax,dword ptr ss:[ebp+10]        ;  又得到参数 3F17DC14  
089F0AEC   |.  8B>|mov edx,dword ptr ss:[ebp+14]        ;  0  
089F0AEF   |.  E8>|call Hrms.087977B2                   ;  得到 3F17DC14 除以 2 的商  
089F0AF4   |.  89>|mov dword ptr ss:[ebp+10],eax        ;  得到 3F17DC14 除以 2 的商保存在原来覆盖  
089F0AF7   |.  89>|mov dword ptr ss:[ebp+14],edx        ;  必为 0  
089F0AFA   |.  FF>|push dword ptr ss:[ebp+1C]           ;  /Arg6  
089F0AFD   |.  FF>|push dword ptr ss:[ebp+18]           ;  |Arg5  
089F0B00   |.  FF>|push dword ptr ss:[ebp+1C]           ;  |Arg4  
089F0B03   |.  FF>|push dword ptr ss:[ebp+18]           ;  |Arg3  
089F0B06   |.  FF>|push dword ptr ss:[ebp+C]            ;  |Arg2  
089F0B09   |.  FF>|push dword ptr ss:[ebp+8]            ;  |Arg1  
089F0B0C   |.  8B>|mov eax,ebx                           ;  |  
089F0B0E   |.  E8>|call Hrms.089F0A58                     ;  \\这个 CALL 就是为了改变 EBP+18 和 EBP+1C  
089F0B13   |.  89>|mov dword ptr ss:[ebp+18],eax  
089F0B16   |.  89>|mov dword ptr ss:[ebp+1C],edx  
089F0B19   |.  EB>|jmp short Hrms.089F0B4C  
089F0B1B   |>  8B>|mov eax,dword ptr ss:[ebp+10]          ;  再次得到参数之一 最初 3F17DC15  
089F0B1E   |.  8B>|mov edx,dword ptr ss:[ebp+14]          ;  参数之 2  最初为 0  
089F0B21   |.  83>|sub eax,1                               ;  减 1 变成了偶数  
089F0B24   |.  83>|sbb edx,0  
089F0B27   |.  89>|mov dword ptr ss:[ebp+10],eax          ;  储存刚才被减 1 后的参数 3F17DC14 偶数了  
089F0B2A   |.  89>|mov dword ptr ss:[ebp+14],edx          ;  还是 0  
089F0B2D   |.  FF>|push dword ptr ss:[ebp+1C]             ;  /Arg6  
089F0B30   |.  FF>|push dword ptr ss:[ebp+18]             ;  |我的前两位输入码开始为 11112222  
089F0B33   |.  FF>|push dword ptr ss:[ebp-4]              ;  |Arg4  
089F0B36   |.  FF>|push dword ptr ss:[ebp-8]              ;  |第一次是上面 05290AB9 压入的 1  此后固定为 11112222  
089F0B39   |.  FF>|push dword ptr ss:[ebp+C]              ;  |参数 0  
089F0B3C   |.  FF>|push dword ptr ss:[ebp+8]              ;  |参数 758F0581  
089F0B3F   |.  8B>|mov eax,ebx                             ;  |  
089F0B41   |.  E8>|call Hrms.089F0A58                       ;  \\Hrms.08A50A58  
089F0B46   |.  89>|mov dword ptr ss:[ebp-8],eax            ;  用相乘的结果覆盖了第一次 EBP-8 固定值 1  
089F0B49   |.  89>|mov dword ptr ss:[ebp-4],edx            ;   
089F0B4C   |>  83>  cmp dword ptr ss:[ebp+14],0             ;  压入参数的地方，第一次为 0  
089F0B50   |.  75>|jnz short Hrms.089F0B5E  
089F0B52   |.  83>|cmp dword ptr ss:[ebp+10],0             ;  第 2 个参数 是 052B8C88 地址处的值不为 0  
089F0B56   |.^ 0F>|ja Hrms.089F0ACC                         ;  大于 0  
089F0B5C   |.  EB>|jmp short Hrms.089F0B64  
089F0B5E   |>^ 0F>\\jg Hrms.089F0ACC  
```

### 汇编代码：乘法子程序 (Hrms.089F0A58)

```assembly
089F0A58   /$  55 push ebp  
089F0A59   |.  8B>mov ebp,esp  
089F0A5B   |.  83>add esp,-10  
089F0A5E   |.  8B>mov eax,dword ptr ss:[ebp+18]        ;  得到我的输入码  
089F0A61   |.  89>mov dword ptr ss:[ebp-10],eax        ;  储存  
089F0A64   |.  8B>mov eax,dword ptr ss:[ebp+1C]  
089F0A67   |.  89>mov dword ptr ss:[ebp-C],eax  
089F0A6A   |.  FF>push dword ptr ss:[ebp+14]  
089F0A6D   |.  FF>push dword ptr ss:[ebp+10]           ;  第一次为 1  
089F0A70   |.  8B>mov eax,dword ptr ss:[ebp-10]  
089F0A73   |.  8B>mov edx,dword ptr ss:[ebp-C]         ;  0  
089F0A76   |.  E8>call Hrms.0879776C                    ;  是相乘的函数（大数相乘）EAX 中放积，EDX 中放相乘的进位  
089F0A7B   |.  89>mov dword ptr ss:[ebp-10],eax        ;  相乘的结果储存覆盖原来的  
089F0A7E   |.  89>mov dword ptr ss:[ebp-C],edx         ;  如果数据过大 相乘超过了 8 位，则进位  
089F0A81   |.  FF>push dword ptr ss:[ebp+C]            ;  0  
089F0A84   |.  FF>push dword ptr ss:[ebp+8]            ;  得到另一个参数 758F0581  
089F0A87   |.  8B>mov eax,dword ptr ss:[ebp-10]        ;  得到相乘的结果  
089F0A8A   |.  8B>mov edx,dword ptr ss:[ebp-C]         ;  进位的 数据  
089F0A8D   |.  E8>call Hrms.087978A5                    ;  这里可能要进行循环（如果上面的大数相乘有进位的话）  
089F0A92   |.  89>mov dword ptr ss:[ebp-10],eax  
089F0A95   |.  89>mov dword ptr ss:[ebp-C],edx  
089F0A98   |.  8B>mov eax,dword ptr ss:[ebp-10]  
089F0A9B   |.  89>mov dword ptr ss:[ebp-8],eax  
089F0A9E   |.  8B>mov eax,dword ptr ss:[ebp-C]  
089F0AA1   |.  89>mov dword ptr ss:[ebp-4],eax  
089F0AA4   |.  8B>mov eax,dword ptr ss:[ebp-8]         ;  取出循环的结果  
089F0AA7   |.  8B>mov edx,dword ptr ss:[ebp-4]         ;  取出循环的结果  
089F0AAA   |.  8B>mov esp,ebp  
089F0AAC   |.  5D pop ebp  
089F0AAD   \\.  C2>retn 18  
```

### 汇编代码：循环/取余子程序 (Hrms.087978A5)

上面循环的 `call Hrms.087978A5`。

```assembly
087978A5   /$  55 push ebp  
087978A6   |.  53 push ebx  
087978A7   |.  56 push esi  
087978A8   |.  57 push edi  
087978A9   |.  33>xor edi,edi  
087978AB   |.  8B>mov ebx,dword ptr ss:[esp+14]        ;  固定值 758F0581  
087978AF   |.  8B>mov ecx,dword ptr ss:[esp+18]        ;  固定值 0  
087978B3   |.  0B>or ecx,ecx  
087978B5   |.  75>jnz short Hrms.087978BF               ;  ECX 必为 0   此必不跳  
087978B7   |.  0B>or edx,edx                             ;  也就是进位  
087978B9   |.  74>je short Hrms.08797918                ;  进位为 0 就跳   第一次不跳它乘以 1  
087978BB   |.  0B>or ebx,ebx  
087978BD   |.  74>je short Hrms.08797918  
087978BF   |>  0B>or edx,edx  
087978C1   |.  79>jns short Hrms.087978CD  
087978C3   |.  F7>neg edx                                ;  这里必不经过  
087978C5   |.  F7>neg eax                                ;  这里必不经过  
087978C7   |.  83>sbb edx,0                              ;  这里必不经过  
087978CA   |.  83>or edi,1                               ;  这里必不经过  
087978CD   |>  0B>or ecx,ecx  
087978CF   |.  79>jns short Hrms.087978D8  
087978D1   |.  F7>neg ecx                                ;  这里必不经过  
087978D3   |.  F7>neg ebx                                ;  这里必不经过  
087978D5   |.  83>sbb ecx,0                              ;  这里必不经过  
087978D8   |>  8B>mov ebp,ecx  
087978DA   |.  B9>mov ecx,40                             ;  循环 64 次  
087978DF   |.  57 push edi                                 ;  必为 0  
087978E0   |.  33>xor edi,edi  
087978E2   |.  33>xor esi,esi  
087978E4   |>  D1>/shl eax,1                              ;  乘以 2   11110C84  
087978E6   |.  D1>|rcl edx,1                              ;  乘以 2 加 1  012347AE（是否加 1 依赖余是否有进位标志位 C）  
087978E8   |.  D1>|rcl esi,1                              ;  开始都是 0 后来会变成 1 当 EDX RCL 有进位时，此时 ESI 加 1  
087978EA   |.  D1>|rcl edi,1                              ;  EDI 必等于 0 不可能等于 1 因为上面不可能有进位  
087978EC   |.  3B>|cmp edi,ebp                             ;  EDI 与 0 比较  
087978EE   |.  72>|jb short Hrms.087978FB  
087978F0   |.  77>|ja short Hrms.087978F6  
087978F2   |.  3B>|cmp esi,ebx                             ;  ESI 与固定值 758F0581 比较  
087978F4   |.  72>|jb short Hrms.087978FB                  ;  小于则跳过  
087978F6   |>  2B>|sub esi,ebx                              ;  当 ESI 超过固定值 758F0581 时，就减去 758F0581  
087978F8   |.  1B>|sbb edi,ebp  
087978FA   |.  40 |inc eax                                  ;  EAX 加 1  
087978FB   |>^ E2>\\loopd short Hrms.087978E4  
087978FD   |.  8B>mov eax,esi  
087978FF   |.  8B>mov edx,edi  
08797901   |.  5B pop ebx                                  ;  必为 0  
08797902   |.  F7>test ebx,1  
08797908   |.  74>je short Hrms.08797911                   ;  必跳  
0879790A   |.  F7>neg edx                                  ;  这里必不经过  
0879790C   |.  F7>neg eax                                  ;  这里必不经过  
```



### 汇编代码分析：注册码处理逻辑

```
0879790E   |.  83>sbb edx,0                         ;  这里必不经过  
08797911   |>  5F pop edi  
08797912   |.  5E pop esi  
08797913   |.  5B pop ebx  
08797914   |.  5D pop ebp  
08797915   |.  C2>retn 8  
08797918   |>  F7>div ebx                     ;  如果进位为 0 把固定值 3F17DC15 除以 2  
0879791A   |.  92 xchg eax,edx                ;  交换 EAX 与 EDX  为了得到余数  
0879791B   |.  33>xor edx,edx                ;  把商清零  
0879791D   \\.^ EB>jmp short Hrms.08797911  

…………………………………………………………………………………………………………………………  

发现处理注册码的函数不可逆！函数出来后到了这里：比较终端数目！  

…………………………………………………………………………………………………………………………  

08A0A416    .  85>test eax,eax  
08A0A418    .  75>jnz short Hrms.08A0A448     ;  如果 EAX 等于 0 就错误  
08A0A41A    .  6A>push 10  
08A0A41C    .  B9>mov ecx,Hrms.08A0A610  
08A0A421    .  BA>mov edx,Hrms.08A0A624  
08A0A426    .  A1>mov eax,dword ptr ds:\[8A19470\]  
08A0A42B    .  8B>mov eax,dword ptr ds:\[eax\]  
08A0A42D    .  E8>call Hrms.087E7400         ;  错误窗口！  
08A0A432    .  8B>mov eax,dword ptr ss:\[ebp-4\]  

……………………省略……………………  

08A0A454    .  E8>call Hrms.087C7A28         ;  得到输入的终端数 返回位数  
08A0A459    .  8B>mov eax,dword ptr ss:\[ebp-30\]  
08A0A45C    .  8D>lea edx,dword ptr ss:\[ebp-2C\]  
08A0A45F    .  E8>call Hrms.08799DE4  
08A0A464    .  83>cmp dword ptr ss:\[ebp-2C\],0  ;  比较看有没有输入数据  
08A0A468    .  74>je short Hrms.08A0A48B  
08A0A46A    .  8D>lea edx,dword ptr ss:\[ebp-34\]  
08A0A46D    .  8B>mov eax,dword ptr ss:\[ebp-4\]  
08A0A470    .  8B>mov eax,dword ptr ds:\[eax+2FC\]  
08A0A476    .  E8>call Hrms.087C7A28         ;  得到输入的终端数 返回位数  
08A0A47B    .  8B>mov eax,dword ptr ss:\[ebp-34\]  
08A0A47E    .  E8>call Hrms.0879A138         ;  将终端数转化为 16 进制数  
08A0A483    .  0F>movzx edx,word ptr ss:\[ebp-E\]    ;  取前面计算出来的结果 2 的 16 进制后 4 位  
08A0A487    .  3B>cmp eax,edx                  ;  与输入的终端数比较  
08A0A489    .  74>je short Hrms.08A0A4B9  

…………………………………………………………………………………………………………………………  

由上面可以知道只能逆转处理序列号的函数了，这里发现可以直接在 `OD` 中逆循环只要改变一下代码  

如果你已经到了 `089F0D80` 处的指令：

```
089F0D80   |.  3B>cmp eax,dword ptr ss:\[ebp-20\]    ;  比较   用注册码双精度数右移后的结果与序列号处理后的结果比较  
```

看到了处理序列号的应该的值，就可以逆转了。修改代码：`089F0B74` 的 `push` 段的代码：

…………………………………………………………………………………………………………………………

### 逆向工程：序列号验证算法逆向

```
08BA0B8D   |> /4A /dec edx                     ;   
08BA0B8E     |2B>sub esi,dword ptr ds:\[8BC8C84\]   ;  注意这里走过这个代码后修改把 ESI 改成 C6EF3720  
08BA0B94     |8B>mov edi,ebx  
08BA0B96   |. |C1>|shl edi,4                  ;   
08BA0B99     |2B>sub eax,edi                  ;   
08BA0B9B   |. |8B>|mov edi,dword ptr ds:\[ecx\]     ;   
08BA0B9D   |. |33>|xor edi,ebx               ;   
08BA0B9F     |2B>sub eax,edi                  ;   
08BA0BA1   |. |8B>|mov edi,ebx  
08BA0BA3   |. |C1>|shr edi,5                  ;   
08BA0BA6   |. |33>|xor edi,esi               ;   
08BA0BA8     |2B>sub eax,edi                  ;   
08BA0BAA     |2B>sub eax,dword ptr ds:\[ecx+4\]    ;   
08BA0BAD   |. |8B>|mov edi,eax  
08BA0BAF   |. |C1>|shl edi,4                  ;   
08BA0BB2     |2B>sub ebx,edi                  ;   
08BA0BB4     |8B>mov edi,dword ptr ds:\[ecx+8\]    ;   
08BA0BB7   |. |33>|xor edi,eax  
08BA0BB9     |2B>sub ebx,edi  
08BA0BBB   |. |8B>|mov edi,eax  
08BA0BBD   |. |C1>|shr edi,5  
08BA0BC0   |. |33>|xor edi,esi  
08BA0BC2     |2B>sub ebx,edi  
08BA0BC4     |2B>sub ebx,dword ptr ds:\[ecx+C\]  
08BA0BC7   |. |85>|test edx,edx  
08BA0BC9   |.^\\77>\\ja short Hrms.08BA0B8D  
08BA0BCB   |.  8B>mov edx,dword ptr ss:\[esp\]  
```

也就是把加改成减，而且把最后要进行比较的值放在 `EBX` 中，`EAX` 中的数据随便填。

并把计算的参数前八位与后 8 位交换变成：

```
59 6F 75 20 61 72 65 20  You are  
48 61 2E 2E 2E 68 61 2C  Ha...ha,  
66 61 74 20 70 69 67 21  fat pig!  
```

当然最后的结果要稍微处理一下。

…………………………………………………………………………………………………………

### 附注册程序代码

```c
unsigned long int xiangx(shuru,b,p)
unsigned long int shuru,b,*p;
{
    unsigned long int jie[8],c,d,e=0xffffffff,jinwei=0,qian8=0,i,g,j,k,h,hou8;
    static long int a[8];
    unsigned static long int carry[8];

    for(i=1;i<=8;i++)
    {
        k=shuru<<(32-i*4);
        k=k>>(32-i*4);
        a[8-i]=k>>((i-1)*4);
    }

    for(i=0;i<8;i++)
    {
        h=b;
        jinwei=0;
        for(j=1;j<a[7-i];j++)
        {
            if((e-h)<=b)
            {
                jinwei++;
                h=h+b;
            }
            else
            {
                h=h+b;
            }
        }
        d=jinwei;
        carry[i]=d;
        jie[i]=b*a[7-i];
    }

    for(i=1;i<8;i++)
    {
        carry[i]=carry[i]<<(i*4);
        g=jie[i]>>(32-i*4);
        carry[i]=carry[i]|g;
    }

    for(i=0;i<8;i++)
    {
        qian8=qian8+carry[i];
    }

    jinwei=0;
    for(i=1;i<8;i++)
    {
        if((e-jie[0])<=(jie[i]<<(4*i)))
        {
            jinwei++;
            jie[0]=jie[0]+(jie[i]<<(4*i));
        }
        else
        {
            jie[0]=jie[0]+(jie[i]<<(4*i));
        }
    }

    qian8=qian8+jinwei;
    hou8=jie[0];
    *p=qian8;
    return(hou8);
}

unsigned long int chuli2(jie,jinwei,gud)
unsigned long int jie,jinwei,gud;
{
    unsigned long int jieguo=0,i,e=0xffffffff,c;

    if(jinwei==0)
    {
        jieguo=jie%gud;
    }
    else
    {
        for(i=64;i>0;i--)
        {
            if((e-jie)<=jie)
            {
                c=1;
                jie=jie+jie;
            }
            else
            {
                c=0;
                jie=jie+jie;
            }

            if((e-jinwei)<=jinwei)
            {
                jinwei=2*jinwei+c;
                c=1;
            }
            else
            {
                jinwei=2*jinwei+c;
                c=0;
            }

            jieguo=2*jieguo+c;

            if(jieguo>gud)
            {
                jieguo=jieguo-gud;
                jie++;
            }
        }
    }
    return(jieguo);
}

unsigned long int fun(x,y,z)
unsigned long int x,y,z;
{
    unsigned long int jie,jinwei,*p=&jinwei,jieguo;
    jie=xiangx(x,y,p);
    jieguo=chuli2(jie,jinwei,z);
    return(jieguo);
}

unsigned long int zong(shuru)
unsigned long int shuru;
{
    unsigned long int xunh=0x3f17dc15,gud=0x758f0581,bchenji=1;

    for(;xunh>0;)
    {
        if((xunh%2)==0)
        {
            xunh=xunh/2;
            shuru=fun(shuru,shuru,gud);
        }
        else
        {
            xunh--;
            bchenji=fun(shuru,bchenji,gud);
        }
    }
    return(bchenji);
}

unsigned long int xlh(qian4,hou4,p)
unsigned long int qian4,hou4,*p;
{
    unsigned long int lei,guding1,guding2,guding3,guding4,guding5;
    int i;

    guding1=0x20756f59;
    guding2=0x20657261;
    guding3=0x2e2e6148;
    guding4=0x2c61682e;
    guding5=0x9e3779b9;

    lei=0xC6EF3720;

    for(i=0;i<0x20;i++)
    {
        hou4=hou4-(qian4<<4);
        hou4=hou4-(qian4^guding1);
        hou4=hou4-((qian4>>5)^lei);
        hou4=hou4-guding2;

        qian4=qian4-(hou4<<4);
        qian4=qian4-(hou4^guding3);
        qian4=qian4-((hou4>>5)^lei);
        qian4=qian4-guding4;

        lei=lei-guding5;
    }
    *p=hou4;
    return(qian4);
}

unsigned long int jianpan(p)
unsigned long int *p;
{
    unsigned long int shuru1=0,shuru2=0;
    int n,i,zj;
    char zifu1[20],zifu2[20],zifu3[20],zifu4[20];

    scanf("%s",zifu1);
    scanf("%s",zifu2);
    scanf("%s",zifu3);
    scanf("%s",zifu4);

    strcat(zifu1,zifu2);
    strcat(zifu3,zifu4);

    n=strlen(zifu1);
    for(i=0;i<n;i++)
    {
        if((zifu1[i]>0x29)&(zifu1[i]<0x40))
        {
            zj=zifu1[i]-0x30;
            shuru1=(shuru1<<4)|zj;
        }
        else if((zifu1[i]>0x40)&(zifu1[i]<0x47))
        {
            zj=zifu1[i]-0x37;
            shuru1=(shuru1<<4)|zj;
        }
        else if((zifu1[i]>0x60)&(zifu1[i]<0x67))
        {
            zj=zifu1[i]-0x57;
            shuru1=(shuru1<<4)|zj;
        }
        else
        {
            shuru1=0;
            break;
        }
    }

    n=strlen(zifu3);
    for(i=0;i<n;i++)
    {
        if((zifu3[i]>0x29)&(zifu3[i]<0x40))
        {
            zj=zifu3[i]-0x30;
            shuru2=(shuru2<<4)|zj;
        }
        else if((zifu3[i]>0x40)&(zifu3[i]<0x47))
        {
            zj=zifu3[i]-0x37;
            shuru2=(shuru2<<4)|zj;
        }
        else if((zifu3[i]>0x60)&(zifu3[i]<0x67))
        {
            zj=zifu3[i]-0x57;
            shuru2=(shuru2<<4)|zj;
        }
        else
        {
            shuru2=0;
            break;
        }
    }
    *p=shuru2;
    return(shuru1);
}

main()
{
    unsigned long int shuru1,shuru2,jieguo1,jieguo2,f=0x0000000f,d,xulihao1,xulihao2,*p=&xulihao2,b;
}
```



```
unsigned long int g=0x000000ff,h=0x0000ff00,i=0x00ff0000,j=0xff000000,zhduan,k=0x0000ffff,shu[4];

unsigned long int *zhizhen=&shuru2;

char ascii[12];

int a;

shuru1=jianpan(zhizhen);

b=0x235678;

jieguo1=zong(shuru1);

jieguo2=zong(shuru2);

jieguo1=jieguo1-2;

jieguo2=jieguo2-2;

jieguo1=jieguo1<<2;

jieguo1=jieguo1>>2;

d=f&jieguo2;

d=d<<2;

d=d&f;

d=d<<28;

jieguo1=jieguo1+d;

jieguo2=jieguo2>>2;

zhduan=jieguo2&k;

for(a=0;a<200;a++)

{

xulihao1=xlh(jieguo1,b,p);

ascii[0]=xulihao1&g;

ascii[1]=(xulihao1&h)>>8;

ascii[2]=(xulihao1&i)>>16;

ascii[3]=(xulihao1&j)>>24;

ascii[4]=xulihao2&g;

ascii[5]=(xulihao2&h)>>8;

ascii[6]=(xulihao2&i)>>16;

ascii[7]=(xulihao2&j)>>24;

ascii[8]='\0';

ascii[9]='\0';

ascii[10]='\0';

ascii[11]='\0';

printf("%s\n",ascii);

b++;

}

printf("zhong duan shu mu:");

printf("%d\n",zhduan);

}
```

输入`4`个注册码；

里面有`200`个序列号可选择，但有些没有用，

因为字符无法显示，也就无法输入，自己选择一下，最后的数据就是终端数目。

`4dd466d0e1e0cc123afbe504da2462ca`
