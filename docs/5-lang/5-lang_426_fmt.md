
# VB 系列之雨佳进销存管理系统简单算法详细分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-270.htm`](https://www.pediy.com/kssd/pediy07/pediy7-270.htm)

## 基本信息

- 【破解作者】`jsliyangsj`
- 【作者邮箱】`sjcrack@yahoo.com.cn`
- 【使用工具】`peid` `OllyDbg1.10`
- 【破解平台】`Winxp`
- 【软件名称】`雨佳进销存管理系统`
- 【编写语言】`VB`
- 【软件地址】`http://nj.onlinedown.net/soft/8853.htm`

## 分析过程

用万能断点，并返回到主程序。

```
00686BF6    .  FF>call dword ptr ds:[eax+A0]                    ;  万能断点 AIT+F9 返回后
00686BFC    .  85>test eax,eax
00686BFE    .  DB>fclex
…………省略………………
00686C65    .  8B>mov ecx,dword ptr ds:[eax]
00686C67    .  FF>call dword ptr ds:[ecx+E0]                    ;  关键的地方，计算机器码，与比较地方
00686C6D    .  85>test eax,eax
00686C6F    .  DB>fclex
00686C71    .  7D>jge short Waregood.00686C85
00686C73    .  68>push 0E0
00686C78    .  53 push ebx
00686C79    .  FF>push dword ptr ds:[6C3048]
00686C7F    .  50 push eax
00686C80    .  E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>
00686C85    >  66>cmp word ptr ss:[ebp-A0],1                    ;  标志位比较
00686C8D    .  8B>mov eax,dword ptr ds:[esi]
00686C8F    .  56 push esi
00686C90    .  0F>jnz Waregood.00686F7E                          ;  关键
00686C96    .  FF>call dword ptr ds:[eax+308]
00686C9C    .  50 push eax
00686C9D    .  8D>lea eax,dword ptr ss:[ebp-2C]
……………………………………………………………………………………………………………………
```

进入 `00686C67 . FF>call dword ptr ds:[ecx+E0]`

```
……………………………………………………………………………………………………………………
0065B8E2    . /0F>je Waregood.0065BA7B
0065B8E8    . |8D>lea esi,dword ptr ds:[edi+80]
0065B8EE    . |FF>push dword ptr ds:[esi]
0065B8F0    . |E8>call <jmp.&MSVBVM60.__vbaLenBstr>              ;  输入码的长度
0065B8F5    . |83>cmp eax,2
0065B8F8    . |0F>jl Waregood.0065BA88
0065B8FE    . |8B>mov eax,dword ptr ds:[edi]
0065B900    . |8D>lea ecx,dword ptr ss:[ebp-38]
0065B903    . |51 push ecx
0065B904    . |57 push edi
0065B905    . |FF>call dword ptr ds:[eax+D4]                    ;  这里得到机器码
0065B90B    . |3B>cmp eax,ebx
0065B90D    . |7D>jge short Waregood.0065B920
0065B90F    . |68>push 0D4
0065B914    . |68>push Waregood.00424F80
0065B919    . |57 push edi
0065B91A    . |50 push eax
0065B91B    . |E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>
0065B920    > |8B>mov edx,dword ptr ss:[ebp-38]
………………………………………………………………………………………………………………
```

进入机器码的地方，先看看如何得到机器码的

```
………………………………………………………………………………………………………………
0059B585    .  E8>call <jmp.&MSVBVM60.__vbaStrI4>               ;  得到了 C 盘序列号的 10 进制
0059B58A    .  8B>mov edx,eax
0059B58C    .  8D>lea ecx,dword ptr ss:[ebp-48]
0059B58F    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>
0059B594    >  68>push Waregood.0059B5FD
0059B599    .  EB>jmp short Waregood.0059B5E4
…………………………………………………………………………………………
```

```
0065B299    .  E8>call Waregood.0059B328                         ;  得到 C 盘序号
0065B29E    .  8B>mov edx,eax
0065B2A0    .  8D>lea ecx,dword ptr ss:[ebp-24]
0065B2A3    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>
0065B2A8    .  FF>push dword ptr ss:[ebp-24]                    ;  C 盘序列号的 10 进制的字符形式
0065B2AB    .  E8>call <jmp.&MSVBVM60.__vbaLenBstr>             ;  得到位数
0065B2B0    .  8B>mov ecx,eax
0065B2B2    .  E8>call <jmp.&MSVBVM60.__vbaI2I4>
0065B2B7    .  6A>push 1
………………………………………………………………………………………………
```

```
0065B2C0    > /8B>mov eax,dword ptr ss:[ebp-18]                 ;  下面将 C 盘的序列号进行伪装成机器码的过程
0065B2C3    . |66>cmp ax,word ptr ss:[ebp-78]                   ;  位数比较
0065B2C7    . |0F>jg Waregood.0065B3A5
0065B2CD    . |8D>lea ecx,dword ptr ss:[ebp-24]
0065B2D0    . |89>mov dword ptr ss:[ebp-34],ebx
0065B2D3    . |89>mov dword ptr ss:[ebp-54],ecx
0065B2D6    . |8D>lea ecx,dword ptr ss:[ebp-3C]
0065B2D9    . |0F>movsx eax,ax
0065B2DC    . |51 push ecx
0065B2DD    . |50 push eax
0065B2DE    . |8D>lea eax,dword ptr ss:[ebp-5C]
0065B2E1    . |C7>mov dword ptr ss:[ebp-3C],2
0065B2E8    . |50 push eax
0065B2E9    . |8D>lea eax,dword ptr ss:[ebp-4C]
0065B2EC    . |50 push eax
0065B2ED    . |C7>mov dword ptr ss:[ebp-5C],4008
0065B2F4    . |E8>call <jmp.&MSVBVM60.#632>
0065B2F9    . |8D>lea eax,dword ptr ss:[ebp-4C]
0065B2FC    . |50 push eax
0065B2FD    . |E8>call <jmp.&MSVBVM60.__vbaStrVarMove>
0065B302    . |8B>mov edx,eax
0065B304    . |8D>lea ecx,dword ptr ss:[ebp-20]
0065B307    . |E8>call <jmp.&MSVBVM60.__vbaStrMove>
0065B30C    . |8D>lea eax,dword ptr ss:[ebp-4C]
0065B30F    . |50 push eax
0065B310    . |8D>lea eax,dword ptr ss:[ebp-3C]
0065B313    . |50 push eax
0065B314    . |6A>push 2
0065B316    . |E8>call <jmp.&MSVBVM60.__vbaFreeVarList>
0065B31B    . |8B>mov eax,dword ptr ss:[ebp-1C]
0065B31E    . |83>add esp,0C
0065B321    . |89>mov dword ptr ss:[ebp-54],eax
0065B324    . |C7>mov dword ptr ss:[ebp-5C],8
0065B32B    . |FF>push dword ptr ss:[ebp-20]
0065B32E    . |E8>call <jmp.&MSVBVM60.#516>                     ;  依次得到每一位序列号的 ASCII
0065B333    . |66>add ax,3F                                     ;  得到的 ASCII 加上 3F
0065B337    . |66>mov cx,1B
0065B33B    . |0F>jo Waregood.0065B455
0065B341    . |66>cwd
0065B343    . |66>idiv cx                                       ;  结果除以 1B
0065B346    . |66>add dx,41                                     ;  余数加上 41 就是对应位的机器码了位数与 C 盘序列号一样，再由刚才计算出来的机器码计算出最后一位的机器码
0065B34A    . |0F>jo Waregood.0065B455
0065B350    . |0F>movsx eax,dx
0065B353    . |50 push eax
0065B354    . |8D>lea eax,dword ptr ss:[ebp-3C]
0065B357    . |50 push eax
0065B358    . |E8>call <jmp.&MSVBVM60.#608>                     ;  将刚才计算出来的数据转化为字符
0065B35D    . |8D>lea eax,dword ptr ss:[ebp-5C]
0065B360    . |50 push eax
0065B361    . |8D>lea eax,dword ptr ss:[ebp-3C]
0065B364    . |50 push eax
0065B365    . |8D>lea eax,dword ptr ss:[ebp-4C]
0065B368    . |50 push eax
0065B369    . |E8>call <jmp.&MSVBVM60.__vbaVarCat>              ;  组合收集字符
0065B36E    . |50 push eax
0065B36F    . |E8>call <jmp.&MSVBVM60.__vbaStrVarMove>
0065B374    . |8B>mov edx,eax
0065B376    . |8D>lea ecx,dword ptr ss:[ebp-1C]
0065B379    . |E8>call <jmp.&MSVBVM60.__vbaStrMove>             ;  收集字符在 EBP-1c
0065B37E    . |8D>lea eax,dword ptr ss:[ebp-4C]
0065B381    . |50 push eax
0065B382    . |8D>lea eax,dword ptr ss:[ebp-3C]
0065B385    . |50 push eax
0065B386    . |6A>push 2
………………………………………………………………………………………………
```

就是将 `C` 盘序列号的每一位 `ASCII` 加上 `3F` 结果除以 `1B` 余数加上 `41` 就是对应位的机器码了

机器码还有最后一位，其实不参与计算注册码

```
…………………………………………………………………………………………………………………………
0065B6D7    .  8B>mov ecx,dword ptr ds:[eax]
0065B6D9    .  FF>call dword ptr ds:[ecx+4]                    ;  得到刚才的处理结果
0065B6DC    .  8B>mov edx,dword ptr ss:[ebp+C]
0065B6DF    .  8D>lea ecx,dword ptr ss:[ebp-20]
0065B6E2    .  89>mov dword ptr ss:[ebp-1C],esi
0065B6E5    .  89>mov dword ptr ss:[ebp-20],esi
0065B6E8    .  89>mov dword ptr ss:[ebp-24],esi
0065B6EB    .  89>mov dword ptr ss:[ebp-28],esi
0065B6EE    .  89>mov dword ptr ss:[ebp-3C],esi
0065B6F1    .  89>mov dword ptr ss:[ebp-4C],esi
0065B6F4    .  89>mov dword ptr ss:[ebp-5C],esi
0065B6F7    .  E8>call <jmp.&MSVBVM60.__vbaStrCopy>             ;  拷贝一个结果
0065B6FC    .  8B>mov eax,dword ptr ss:[ebp+10]
0065B6FF    .  FF>push dword ptr ss:[ebp-20]
0065B702    .  89>mov dword ptr ds:[eax],esi
0065B704    .  E8>call <jmp.&MSVBVM60.__vbaLenBstr>             ;  得到处理后的长度
0065B709    .  8B>mov ecx,eax
0065B70B    .  E8>call <jmp.&MSVBVM60.__vbaI2I4>
0065B710    .  6A>push 1
0065B712    .  89>mov dword ptr ss:[ebp-74],eax                 ;  长度储存
0065B715    .  5E pop esi
0065B716    .  6A>push 2
0065B718    .  8B>mov edi,esi
0065B71A    .  5B pop ebx                                       ;  下面是计算机器码的最后一位
```



```
0065B71B    >   66>cmp di,word ptr ss:\[ebp-74\]  

0065B71F    .   7F>jg short Waregood.0065B79B  

0065B721    .   8D>lea eax,dword ptr ss:\[ebp-20\]  

0065B724    .   89>mov dword ptr ss:\[ebp-34\],esi  

0065B727    .   89>mov dword ptr ss:\[ebp-54\],eax  

0065B72A    .   8D>lea eax,dword ptr ss:\[ebp-3C\]  

0065B72D    .   50 push eax  

0065B72E    .   89>mov dword ptr ss:\[ebp-3C\],ebx  

0065B731    .   0F>movsx eax,di  

0065B734    .   50 push eax  

0065B735    .   8D>lea eax,dword ptr ss:\[ebp-5C\]  

0065B738    .   50 push eax  

0065B739    .   8D>lea eax,dword ptr ss:\[ebp-4C\]  

0065B73C    .   50 push eax  

0065B73D    .   C7>mov dword ptr ss:\[ebp-5C\],4008  

0065B744    .   E8>call <jmp.&MSVBVM60.#632>                   ;  要进入的  

0065B749    .   8D>lea eax,dword ptr ss:\[ebp-4C\]  

0065B74C    .   50 push eax  

0065B74D    .   E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ;  依次得到每一个机器码  

0065B752    .   8B>mov edx,eax  

0065B754    .   8D>lea ecx,dword ptr ss:\[ebp-28\]  

0065B757    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>  

0065B75C    .   8D>lea eax,dword ptr ss:\[ebp-4C\]  

0065B75F    .   50 push eax  

0065B760    .   8D>lea eax,dword ptr ss:\[ebp-3C\]  

0065B763    .   50 push eax  

0065B764    .   53 push ebx  

0065B765    .   E8>call <jmp.&MSVBVM60.__vbaFreeVarList>  

0065B76A    .   83>add esp,0C  

0065B76D    .   FF>push dword ptr ss:\[ebp-28\]  

0065B770    .   E8>call <jmp.&MSVBVM60.#516>                   ;  依次（从左到右）得到每个字符的 ASCII  

0065B775    .   66>add ax,word ptr ss:\[ebp-1C\]               ;  每个相加  

0065B779    .   6A>push 1  

0065B77B    .   0F>jo Waregood.0065B838  

0065B781    .   66>sub ax,bx                                   ;  结果减去 2  

0065B784    .   0F>jo Waregood.0065B838  

0065B78A    .   89>mov dword ptr ss:\[ebp-1C\],eax             ;  储存  

0065B78D    .   58 pop eax  

0065B78E    .   66>add ax,di  

0065B791    .   0F>jo Waregood.0065B838  

0065B797    .   8B>mov edi,eax  

0065B799    .   ^ EB>jmp short Waregood.0065B71B  

0065B79B    >   66>mov ax,word ptr ss:\[ebp-1C\]               ;  得到计算后的结果  

0065B79F    .   66>mov cx,1A  

0065B7A3    .   66>cwd  

0065B7A5    .   66>idiv cx                                     ;  结果除以 1A  

0065B7A8    .   66>add dx,41                                   ;  余数加上 41  

0065B7AC    .   0F>jo Waregood.0065B838  

0065B7B2    .   0F>movsx eax,dx                                ;  余数要了  

0065B7B5    .   50 push eax  

0065B7B6    .   8D>lea eax,dword ptr ss:\[ebp-3C\]  

0065B7B9    .   50 push eax  

0065B7BA    .   E8>call <jmp.&MSVBVM60.#608>                   ;  将计算好的转化为字符  

0065B7BF    .   8D>lea eax,dword ptr ss:\[ebp-3C\]  

0065B7C2    .   50 push eax  

0065B7C3    .   E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ;  取出结果  

0065B7C8    .   8B>mov edx,eax  

0065B7CA    .   8D>lea ecx,dword ptr ss:\[ebp-24\]  

0065B7CD    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>  

0065B7D2    .   8D>lea ecx,dword ptr ss:\[ebp-3C\]  

0065B7D5    .   E8>call <jmp.&MSVBVM60.__vbaFreeVar>  

0065B7DA    .   68>push Waregood.0065B813  

0065B7DF    .   EB>jmp short Waregood.0065B802  

……………………………………………………………………………………………………………………  

就是将前面又 C 盘序列好得到的机器码依次相加，并每次减 2，得到的数据除以 `1A` 余数加上 `41` 就是最后一位  

机器码了  

……………………………………………………………………………………………………………………  

```
0065B3AF    .   FF>call dword ptr ds:\[eax+DC\]                ;  将由 C 盘序列号转化来的机器码（10 位）再计算出第 11 位组合得到 11 位的机器码  

0065B3B5    .   3B>cmp eax,esi  

0065B3B7    .   7D>jge short Waregood.0065B3CA  

0065B3B9    .   68>push 0DC  

0065B3BE    .   68>push Waregood.00424F80  

0065B3C3    .   57 push edi  

0065B3C4    .   50 push eax  

0065B3C5    .   E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>  

0065B3CA    >   FF>push dword ptr ss:\[ebp-1C\]                ;  又得到机器码  

0065B3CD    .   FF>push dword ptr ss:\[ebp-2C\]  

0065B3D0    .   E8>call <jmp.&MSVBVM60.__vbaStrCat>           ;  将由 C 盘序列号转化来的机器码（10 位）再计算出第 11 位组合得到 11 位的机器码  

0065B3D5    .   8B>mov edx,eax  

0065B3D7    .   8D>lea ecx,dword ptr ss:\[ebp-28\]  

0065B3DA    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>  

………………………………………………………………………………………………………………  

上面是如何产生机器码，下面对输入吗的要求  

……………………………………………………………………………………………………………………  

```
0065B920    >   \\8B>mov edx,dword ptr ss:\[ebp-38\]  

0065B923    .   89>mov dword ptr ss:\[ebp-38\],ebx  

0065B926    .   8D>lea ecx,dword ptr ss:\[ebp-2C\]  

0065B929    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>  

0065B92E    .   6A>push 1  

0065B930    .   FF>push dword ptr ds:\[esi\]                 ;  得到我的输入码  

0065B932    .   68>push Waregood.00435C6C                   ;  这个是固定值“ - ”  

0065B937    .   53 push ebx  

0065B938    .   E8>call <jmp.&MSVBVM60.__vbaInStr>           ;  检查我的输入码中是否存在“ - ”3 个字符  

0065B93D    .   8B>mov ecx,eax                               ;  如果有返回是“ - ”在字符串的位数包括空格本身  

0065B93F    .   E8>call <jmp.&MSVBVM60.__vbaI2I4>  

0065B944    .   89>mov dword ptr ss:\[ebp-24\],eax  

0065B947    .   66>cmp ax,bx  

……………………………………………………………………………………………………………………  

进入 `0065B938 . E8>call <jmp.&MSVBVM60.__vbaInStr>`  

……………………………………………………………………………………………………………………  

```
733A45A5 M> $  55 push ebp  
733A45A6    .   8B>mov ebp,esp  
733A45A8    .   81>sub esp,0BC  
733A45AE    .   83>and dword ptr ss:\[ebp-14\],0  
733A45B2    .   53 push ebx  
733A45B3    .   56 push esi  
733A45B4    .   8B>mov esi,dword ptr ss:\[ebp+C\]  
733A45B7    .   57 push edi  
733A45B8    .   8B>mov edi,dword ptr ss:\[ebp+10\]            ;  得到输入码  
733A45BB    .   8D>lea eax,dword ptr ss:\[ebp-BC\]  
733A45C1    .   89>mov dword ptr ss:\[ebp-8\],edi  
733A45C4    .   85>test edi,edi                               ;  检查是否有输入码  
733A45C6    .   89>mov dword ptr ss:\[ebp-C\],eax  
733A45C9    .   89>mov dword ptr ss:\[ebp-4\],esi  
733A45CC    .   0F>je MSVBVM60.733D7ADB  
733A45D2    .   8B>mov eax,dword ptr ds:\[edi-4\]  
733A45D5    .   D1>shr eax,1                                   ;  输入码的位数  
733A45D7    .   89>mov dword ptr ss:\[ebp-1C\],eax            ;  位数保存  
733A45DA    .   0F>je MSVBVM60.733D7ADB                       ;  是否等于 0  
733A45E0    .   85>test esi,esi  
733A45E2    .   0F>je MSVBVM60.733D7AD3  
733A45E8    .   8B>mov eax,dword ptr ds:\[esi-4\]             ;  固定值 6  
733A45EB    .   D1>shr eax,1                                   ;  得到固定值 3  
733A45ED    .   89>mov dword ptr ss:\[ebp-1C\],eax            ;  储存  
733A45F0    .   0F>je MSVBVM60.733D7AD3  
733A45F6    .   8B>mov eax,dword ptr ss:\[ebp+14\]  
733A45F9    .   8D>lea ebx,dword ptr ds:\[eax-1\]  
733A45FC    .   85>test ebx,ebx  
733A45FE    .   0F>jl MSVBVM60.733D7937  
733A4604    .   81>cmp ebx,3FFFFFFF  
733A460A    .   0F>ja MSVBVM60.733D7937  
733A4610    .   8B>mov eax,dword ptr ss:\[ebp+8\]  
733A4613    .   89>mov dword ptr ss:\[ebp-18\],ebx  
733A4616    .   85>test eax,eax  
733A4618    .   0F>jnz MSVBVM60.733D793E  
733A461E    >   8B>mov eax,dword ptr ss:\[ebp-8\]             ;  得到输入吗  
733A4621    .   85>test eax,eax  
733A4623    .   0F>je MSVBVM60.733D7A2F  
733A4629    .   8B>mov ecx,dword ptr ds:\[eax-4\]  
733A462C    .   D1>shr ecx,1                                   ;  又得到输入吗的位数  
733A462E    >   85>test esi,esi  
733A4630    .   0F>je MSVBVM60.733D7A36  
733A4636    .   8B>mov edx,dword ptr ds:\[esi-4\]  
733A4639    .   D1>shr edx,1                                   ;  得到固定值 3  
733A463B    >   8B>mov edi,dword ptr ss:\[ebp-18\]  
733A463E    .   3B>cmp edi,ecx  
733A4640    .   73>jnb short MSVBVM60.733A46B6                ;  输入吗是否等于 0  
733A4642    .   85>test edx,edx  
733A4644    .   0F>je MSVBVM60.733D7A3D  
733A464A    .   3B>cmp edx,ecx                                 ;  这里说明输入吗的位数必须大于 3  
733A464C    .   0F>ja MSVBVM60.733D7A48  
733A4652    .   8D>lea eax,dword ptr ds:\[eax+edi*2\]  
733A4655    .   8B>mov edi,dword ptr ss:\[ebp-8\]  
733A4658    .   2B>sub ecx,edx                                 ;  输入吗的位数减去 3  
733A465A    .   8D>lea ebx,dword ptr ds:\[edi+ecx*2+2\]        ;  得到倒数 2 位的地址  
```



`733A465E    .   0F>movzx ecx,word ptr ds:[esi]                            ;  得到一个空格的 ASCII20`  

`733A4661    .   89>mov dword ptr ss:[ebp+14],ecx`  

`733A4664    .   8D>lea ecx,dword ptr ds:[edx+edx-2]`  

`733A4668    .   3B>cmp eax,ebx`  

`733A466A    .   89>mov dword ptr ss:[ebp-1C],ecx`  

`733A466D    .   73>jnb short MSVBVM60.733A46B6`  

`733A466F    >   8B>mov ecx,ebx`  

`733A4671    .   2B>sub ecx,eax`  

`733A4673    .   D1>sar ecx,1                                                ;  得到输入码去后两位的位数`  

`733A4675    .   51 push ecx                                                 ; /Arg3`  

`733A4676    .   FF>push dword ptr ss:[ebp+14]                                ; |固定值 20（空格的 ASCII）`  

`733A4679    .   50 push eax                                                 ; |我的输入码`  

`733A467A    .   E8>call MSVBVM60.733A46C5                                    ; \\在去掉后两位的输入码中寻找是否有空格 返回结果是从空格向后的输入码`  

`733A467F    .   85>test eax,eax                                             ;  没有空格就会出错`  

`733A4681    .   74>je short MSVBVM60.733A46B6`  

`733A4683    .   8B>mov ecx,dword ptr ss:[ebp-1C]                             ;  固定值 4`  

`733A4686    .   40 inc eax`  

`733A4687    .   40 inc eax                                                  ;  上面两个 INC 就是去掉前面的一个空格`  

`733A4688    .   8D>lea edi,dword ptr ds:[esi+2]                              ;  得到固定值“- ”两个`  

`733A468B    .   8B>mov esi,eax                                              ;  刚才检查剩下的输入码`  

`733A468D    .   33>xor edx,edx`  

`733A468F    .   F3>repe cmps byte ptr es:[edi],byte ptr ds:[esi>;  在刚才检查剩下的输入吗中寻找是否还有“- ”两个字符`  

`733A4691    .   75>jnz short MSVBVM60.733A46AD                               ;  如果没有返回不为 0  就出错`  

`733A4693    .   8B>mov ecx,eax`  

`733A4695    .   2B>sub ecx,dword ptr ss:[ebp-8]`  

`733A4698    .   D1>sar ecx,1                                                ;  得到输入码“ - ”前面的字符与空格的位数`  

`733A469A    .   83>cmp dword ptr ss:[ebp+8],0`  

`733A469E    .   0F>jnz MSVBVM60.733D7A51`  

`733A46A4    .   8B>mov eax,ecx`  

`733A46A6    >   5F pop edi`  

`733A46A7    .   5E pop esi`  

`733A46A8    .   5B pop ebx`  

`733A46A9    .   C9 leave`  

`733A46AA    .   C2>retn 10`  

…………………………………………………………………………………………………………………………  

注册码必须分为两部分，用“ - ”分开。如果这里不对，不进行注册码的计算，直接出现出错窗口。  

…………………………………………………………………………………………………………………………  

`0065B93F    .   E8>call <jmp.&MSVBVM60.__vbaI2I4>`  

`0065B944    .   89>mov dword ptr ss:[ebp-24],eax`  

`0065B947    .   66>cmp ax,bx`  

`0065B94A    .   0F>jle Waregood.0065BA7B`  

`0065B950    .   89>mov dword ptr ss:[ebp-58],esi`  

`0065B953    .   BB>mov ebx,4008                                             ;  固定值 4008`  

`0065B958    .   89>mov dword ptr ss:[ebp-60],ebx`  

`0065B95B    .   66>sub ax,1                                                 ;  位数减 1 去掉了空格`  

`0065B95F    .   0F>jo Waregood.0065BB01`  

`0065B965    .   0F>movsx eax,ax`  

`0065B968    .   50 push eax`  

`0065B969    .   8D>lea eax,dword ptr ss:[ebp-60]                             ;  固定值 4008`  

`0065B96C    .   50 push eax`  

`0065B96D    .   8D>lea eax,dword ptr ss:[ebp-50]`  

`0065B970    .   50 push eax`  

`0065B971    .   E8>call <jmp.&MSVBVM60.#617>                                  ;  取出输入码“ - ”前面的一组部分用于最后的比较`  

`0065B976    .   8D>lea eax,dword ptr ss:[ebp-50]`  

`0065B979    .   50 push eax`  

`0065B97A    .   E8>call <jmp.&MSVBVM60.__vbaStrVarMove>                        ;  取出前半段的输入吗`  

`0065B97F    .   8B>mov edx,eax`  

`0065B981    .   8D>lea ecx,dword ptr ss:[ebp-28]`  

`0065B984    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>`  

`0065B989    .   8D>lea ecx,dword ptr ss:[ebp-50]`  

`0065B98C    .   E8>call <jmp.&MSVBVM60.__vbaFreeVar>`  

`0065B991    .   89>mov dword ptr ss:[ebp-58],esi`  

`0065B994    .   89>mov dword ptr ss:[ebp-60],ebx`  

`0065B997    .   FF>push dword ptr ds:[esi]`  

`0065B999    .   E8>call <jmp.&MSVBVM60.__vbaLenBstr>                            ;  得到输入吗的总长度`  

`0065B99E    .   0F>movsx ecx,word ptr ss:[ebp-24]`  

`0065B9A2    .   2B>sub eax,ecx`  

`0065B9A4    .   0F>jo Waregood.0065BB01`  

`0065B9AA    .   83>sub eax,2`  

`0065B9AD    .   0F>jo Waregood.0065BB01`  

`0065B9B3    .   50 push eax`  

`0065B9B4    .   8D>lea eax,dword ptr ss:[ebp-60]`  

`0065B9B7    .   50 push eax`  

`0065B9B8    .   8D>lea eax,dword ptr ss:[ebp-50]`  

`0065B9BB    .   50 push eax`  

`0065B9BC    .   E8>call <jmp.&MSVBVM60.#619>                                  ;  得到” - “后面的输入吗的字符`  

`0065B9C1    .   8D>lea eax,dword ptr ss:[ebp-50]`  

`0065B9C4    .   50 push eax`  

`0065B9C5    .   E8>call <jmp.&MSVBVM60.__vbaStrVarMove>                        ;  取出后半段的输入吗用于后面比较`  

`0065B9CA    .   8B>mov edx,eax`  

`0065B9CC    .   8D>lea ecx,dword ptr ss:[ebp-34]`  

`0065B9CF    .   E8>call <jmp.&MSVBVM60.__vbaStrMove>`  

`0065B9D4    .   8D>lea ecx,dword ptr ss:[ebp-50]`  

`0065B9D7    .   E8>call <jmp.&MSVBVM60.__vbaFreeVar>`  

`0065B9DC    .   8B>mov eax,dword ptr ds:[edi]`  

`0065B9DE    .   8D>lea ecx,dword ptr ss:[ebp-38]`  

`0065B9E1    .   51 push ecx`  

`0065B9E2    .   FF>push dword ptr ss:[ebp-2C]                                ;  得到机器码实际上最后一位没有用到`  

`0065B9E5    .   57 push edi`  

`0065B9E6    .   FF>call dword ptr ds:[eax+EC]                                ;  关键进入  用机器码计算注册码的地方`  

`0065B9EC    .   85>test eax,eax`  

`0065B9EE    .   7D>jge short Waregood.0065BA01`  

`0065B9F0    .   68>push 0EC`  

`0065B9F5    .   68>push Waregood.00424F80`  

`0065B9FA    .   57 push edi`  

`0065B9FB    .   50 push eax`  

`0065B9FC    .   E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>`  

`0065BA01    >   33>xor edx,edx`  

`0065BA03    .   8D>lea ecx,dword ptr ss:[ebp-3C]`  

`0065BA06    .   E8>call <jmp.&MSVBVM60.__vbaStrCopy>`  

`0065BA0B    .   8B>mov eax,dword ptr ds:[edi]`  

`0065BA0D    .   8D>lea ecx,dword ptr ss:[ebp-40]`  

`0065BA10    .   51 push ecx`  

`0065BA11    .   8D>lea ecx,dword ptr ss:[ebp-3C]`  

`0065BA14    .   51 push ecx`  

`0065BA15    .   57 push edi`  

`0065BA16    .   FF>call dword ptr ds:[eax+D8]                                ;  关键进入  用单位名称计算注册码的后半部分`  

`0065BA1C    .   85>test eax,eax`  

`0065BA1E    .   7D>jge short Waregood.0065BA31`  

`0065BA20    .   68>push 0D8`  

`0065BA25    .   68>push Waregood.00424F80`  

`0065BA2A    .   57 push edi`  

`0065BA2B    .   50 push eax`  

`0065BA2C    .   E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>`  

`0065BA31    >   FF>push dword ptr ss:[ebp-34]                                ;  输入码的后半段`  

`0065BA34    .   FF>push dword ptr ss:[ebp-40]                                ;  单位名称计算出来的注册码`  

`0065BA37    .   E8>call <jmp.&MSVBVM60.__vbaStrCmp>                           ;  比较后面的`  

`0065BA3C    .   8B>mov esi,eax`  

`0065BA3E    .   F7>neg esi`  

`0065BA40    .   1B>sbb esi,esi`  

`0065BA42    .   46 inc esi`  

`0065BA43    .   F7>neg esi`  

`0065BA45    .   FF>push dword ptr ss:[ebp-28]                                ;  输入码的前半段`  

`0065BA48    .   FF>push dword ptr ss:[ebp-38]                                ;  机器码得到注册码`  

`0065BA4B    .   E8>call <jmp.&MSVBVM60.__vbaStrCmp>                           ;  比较前面的`  

`0065BA50    .   F7>neg eax`  

`0065BA52    .   1B>sbb eax,eax`  

`0065BA54    .   40 inc eax`  

`0065BA55    .   F7>neg eax`  

`0065BA57    .   23>and esi,eax`  

`0065BA59    .   8D>lea eax,dword ptr ss:[ebp-40]`  

`0065BA5C    .   50 push eax`  

`0065BA5D    .   8D>lea eax,dword ptr ss:[ebp-3C]`  

`0065BA60    .   50 push eax`  

`0065BA61    .   8D>lea eax,dword ptr ss:[ebp-38]`  

`0065BA64    .   50 push eax`  

`0065BA65    .   6A>push 3`  

`0065BA67    .   E8>call <jmp.&MSVBVM60.__vbaFreeStrList>`  

`0065BA6C    .   83>add esp,10`  

`0065BA6F    .   66>test si,si`  

`0065BA72    .   74>je short Waregood.0065BA7B`  

`0065BA74    .   C7>mov dword ptr ss:[ebp-30],1                               ;  标志位`  

`0065BA7B    >   66>mov ax,word ptr ss:[ebp-30]`  

`0065BA7F    .   66>mov word ptr ds:[edi+84],ax`  

`0065BA86    .   EB>jmp short Waregood.0065BA96`  

………………………………………………………………………………………………………………  

首先进入由机器码计算前半段注册码的过程。  

`0065B9E6    .   FF>call dword ptr ds:[eax+EC]                                ;  关键进入  用机器码计算注册码的地方`  

……………………………………………………………………………………………………………………  

`0065C03A    .   FF>call dword ptr ds:[ecx+4]`  

`0065C03D    .   8B>mov edx,dword ptr ss:[ebp+C]                              ;  得到机器码`  

`0065C040    .   8D>lea ecx,dword ptr ss:[ebp-1C]`  

`0065C043    .   89>mov dword ptr ss:[ebp-1C],esi`  

`0065C046    .   89>mov dword ptr ss:[ebp-20],esi`  

`0065C049    .   89>mov dword ptr ss:[ebp-24],esi`  

`0065C04C    .   89>mov dword ptr ss:[ebp-28],esi`  



```
0065C04F    .  89>mov dword ptr ss:\[ebp-2C\],esi

0065C052    .  89>mov dword ptr ss:\[ebp-30\],esi

0065C055    .  89>mov dword ptr ss:\[ebp-34\],esi

0065C058    .  89>mov dword ptr ss:\[ebp-44\],esi

0065C05B    .  89>mov dword ptr ss:\[ebp-54\],esi

0065C05E    .  89>mov dword ptr ss:\[ebp-64\],esi

0065C061    .  E8>call <jmp.&MSVBVM60.__vbaStrCopy>            ;  拷贝机器码

0065C066    .  FF>push dword ptr ss:\[ebp-1C\]                  ;  得到机器码

0065C069    .  8B>mov eax,dword ptr ss:\[ebp+10\]

0065C06C    .  BF>mov edi,Waregood.0042501C

0065C071    .  57 push edi

0065C072    .  89>mov dword ptr ds:\[eax\],esi

0065C074    .  E8>call <jmp.&MSVBVM60.__vbaStrCmp>             ;  机器码与 0 比较，就是是否有机器码

0065C079    .  85>test eax,eax

0065C07B    .  0F>je Waregood.0065C293

0065C081    .  FF>push dword ptr ss:\[ebp-1C\]                 ;  机器码

0065C084    .  8D>lea eax,dword ptr ss:\[ebp-1C\]

0065C087    .  BE>mov esi,4008                                 ;  固定值 4008

0065C08C    .  89>mov dword ptr ss:\[ebp-5C\],eax

0065C08F    .  89>mov dword ptr ss:\[ebp-64\],esi

0065C092    .  E8>call <jmp.&MSVBVM60.__vbaLenBstr>            ;  机器码的长度

0065C097    .  6A>push 1

0065C099    .  5B pop ebx

0065C09A    .  2B>sub eax,ebx                                   ;  机器码 的长度减 1 就是不要用到最后一位

0065C09C    .  0F>jo Waregood.0065C311

0065C0A2    .  50 push eax

0065C0A3    .  8D>lea eax,dword ptr ss:\[ebp-64\]

0065C0A6    .  50 push eax

0065C0A7    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C0AA    .  50 push eax

0065C0AB    .  E8>call <jmp.&MSVBVM60.#617>                    ;  从左边得到机器码去掉最后一位

0065C0B0    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C0B3    .  50 push eax

0065C0B4    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ;  得到将机器码取最后一位的字符

0065C0B9    .  8B>mov edx,eax

0065C0BB    .  8D>lea ecx,dword ptr ss:\[ebp-24\]

0065C0BE    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>

0065C0C3    .  8D>lea ecx,dword ptr ss:\[ebp-44\]

0065C0C6    .  E8>call <jmp.&MSVBVM60.__vbaFreeVar>

0065C0CB    .  8D>lea eax,dword ptr ss:\[ebp-1C\]

0065C0CE    .  53 push ebx

0065C0CF    .  89>mov dword ptr ss:\[ebp-5C\],eax

0065C0D2    .  8D>lea eax,dword ptr ss:\[ebp-64\]

0065C0D5    .  50 push eax

0065C0D6    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C0D9    .  50 push eax

0065C0DA    .  89>mov dword ptr ss:\[ebp-64\],esi

0065C0DD    .  E8>call <jmp.&MSVBVM60.#619>                    ;  从右边得到机器码的最后一位

0065C0E2    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C0E5    .  50 push eax

0065C0E6    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ;  得到最后一位的地址 用不到

0065C0EB    .  8B>mov edx,eax

0065C0ED    .  8D>lea ecx,dword ptr ss:\[ebp-30\]

0065C0F0    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>

0065C0F5    .  8D>lea ecx,dword ptr ss:\[ebp-44\]

0065C0F8    .  E8>call <jmp.&MSVBVM60.__vbaFreeVar>

0065C0FD    .  8B>mov edx,edi

0065C0FF    .  8D>lea ecx,dword ptr ss:\[ebp-20\]

0065C102    .  E8>call <jmp.&MSVBVM60.__vbaStrCopy>            ;  得到机器吗的前 10 位就是去掉机器码的最后一位

0065C107    .  FF>push dword ptr ss:\[ebp-24\]

0065C10A    .  E8>call <jmp.&MSVBVM60.__vbaLenBstr>            ;  得到长度（去机器吗的最后一位）

0065C10F    .  8B>mov ecx,eax

0065C111    .  E8>call <jmp.&MSVBVM60.__vbaI2I4>

0065C116    >  6A>push 1

0065C118    .  8B>mov edi,eax

0065C11A    .  58 pop eax                                       ;  固定值 1

0065C11B    .  66>cmp di,ax                                    ;  DI 是这次准备要取位数 与 1 比较是否小于说明已经取完了

0065C11E    .  0F>jl Waregood.0065C1FD

0065C124    .  8D>lea eax,dword ptr ss:\[ebp-24\]

0065C127    .  89>mov dword ptr ss:\[ebp-3C\],ebx

0065C12A    .  89>mov dword ptr ss:\[ebp-5C\],eax

0065C12D    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C130    .  50 push eax

0065C131    .  C7>mov dword ptr ss:\[ebp-44\],2

0065C138    .  0F>movsx eax,di

0065C13B    .  50 push eax

0065C13C    .  8D>lea eax,dword ptr ss:\[ebp-64\]

0065C13F    .  50 push eax

0065C140    .  8D>lea eax,dword ptr ss:\[ebp-54\]

0065C143    .  50 push eax

0065C144    .  89>mov dword ptr ss:\[ebp-64\],esi

0065C147    .  E8>call <jmp.&MSVBVM60.#632>                    ;  得到机器吗的每一位（从机器码去掉最后一位的最后一位起向前取值）

0065C14C    .  8D>lea eax,dword ptr ss:\[ebp-54\]

0065C14F    .  50 push eax

0065C150    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ;  取出得到每一位机器吗的 ASCII

0065C155    .  8B>mov edx,eax

0065C157    .  8D>lea ecx,dword ptr ss:\[ebp-2C\]

0065C15A    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>

0065C15F    .  8D>lea eax,dword ptr ss:\[ebp-54\]

0065C162    .  50 push eax

0065C163    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C166    .  50 push eax

0065C167    .  6A>push 2

0065C169    .  E8>call <jmp.&MSVBVM60.__vbaFreeVarList>

0065C16E    .  8B>mov eax,dword ptr ss:\[ebp-20\]

0065C171    .  83>add esp,0C

0065C174    .  89>mov dword ptr ss:\[ebp-5C\],eax

0065C177    .  C7>mov dword ptr ss:\[ebp-64\],8

0065C17E    .  FF>push dword ptr ss:\[ebp-2C\]

0065C181    .  E8>call <jmp.&MSVBVM60.#516>                    ;  将机器码的最后一位向前取值得到 ASCII 放到 EAX 中

0065C186    .  66>add ax,di                                      ;  取机器码加上本身的位数

0065C189    .  66>mov cx,1A

0065C18D    .  0F>jo Waregood.0065C311

0065C193    .  66>add ax,17                                      ;  取机器码加上本身的位数再加上 17

0065C197    .  0F>jo Waregood.0065C311

0065C19D    .  66>cwd

0065C19F    .  66>idiv cx                                         ;  上面的和除以 1A

0065C1A2    .  66>add dx,41                                       ;  余数加上 41  就是最后注册码了

0065C1A6    .  0F>jo Waregood.0065C311

0065C1AC    .  0F>movsx eax,dx

0065C1AF    .  50 push eax

0065C1B0    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C1B3    .  50 push eax

0065C1B4    .  E8>call <jmp.&MSVBVM60.#608>                    ;  将上面的数据转化为字符

0065C1B9    .  8D>lea eax,dword ptr ss:\[ebp-64\]

0065C1BC    .  50 push eax

0065C1BD    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C1C0    .  50 push eax

0065C1C1    .  8D>lea eax,dword ptr ss:\[ebp-54\]

0065C1C4    .  50 push eax

0065C1C5    .  E8>call <jmp.&MSVBVM60.__vbaVarCat>             ;  每次得到的字符组合起来

0065C1CA    .  50 push eax

0065C1CB    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>

0065C1D0    .  8B>mov edx,eax

0065C1D2    .  8D>lea ecx,dword ptr ss:\[ebp-20\]

0065C1D5    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>            ;  组合后 拷贝到在 EBP-20

0065C1DA    .  8D>lea eax,dword ptr ss:\[ebp-54\]

0065C1DD    .  50 push eax

0065C1DE    .  8D>lea eax,dword ptr ss:\[ebp-44\]

0065C1E1    .  50 push eax

0065C1E2    .  6A>push 2

0065C1E4    .  E8>call <jmp.&MSVBVM60.__vbaFreeVarList>

0065C1E9    .  83>or eax,FFFFFFFF

0065C1EC    .  83>add esp,0C

0065C1EF    .  66>add ax,di

0065C1F2    .  0F>jo Waregood.0065C311

0065C1F8    .^ E9>jmp Waregood.0065C116

0065C1FD    >  8B>mov esi,dword ptr ss:\[ebp+8\]

0065C200    .  8D>lea ecx,dword ptr ss:\[ebp-34\]

0065C203    .  51 push ecx

0065C204    .  FF>push dword ptr ss:\[ebp-20\]

0065C207    .  8B>mov eax,dword ptr ds:\[esi\]

0065C209    .  56 push esi

0065C20A    .  FF>call dword ptr ds:\[eax+DC\]                 ;  要进入，由前面得到的 10 位得到第 11 位

0065C210    .  85>test eax,eax

0065C212    .  7D>jge short Waregood.0065C225

0065C214    .  68>push 0DC

0065C219    .  68>push Waregood.00424F80

0065C21E    .  56 push esi

0065C21F    .  50 push eax

0065C220    .  E8>call <jmp.&MSVBVM60.__vbaHresultCheckObj>

0065C225    >  FF>push dword ptr ss:\[ebp-20\]                 ;  前 10 位

0065C228    .  FF>push dword ptr ss:\[ebp-34\]                 ;  由前 10 位得到第 11 位

0065C22B    .  E8>call <jmp.&MSVBVM60.__vbaStrCat>            ;  这里组合了，11 位这里就是前一部分注册码

0065C230    .  8B>mov edx,eax

0065C232    .  8D>lea ecx,dword ptr ss:\[ebp-20\]
```



## 机器码分析与注册码计算逻辑

### 第一部分：基于机器码的注册码生成

以下汇编指令片段展示了处理机器码的相关逻辑：

```asm
0065C235    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>  
0065C23A    .  8D>lea ecx,dword ptr ss:[ebp-34]  
0065C23D    .  E8>call <jmp.&MSVBVM60.__vbaFreeStr>  
0065C242    .  8B>mov eax,dword ptr ds:[esi]  
0065C244    .  8D>lea ecx,dword ptr ss:[ebp-34]  
```

**算法描述：**

1.  从机器码的倒数第 `2` 位开始，依次向前取每一位的 `ASCII` 码。
2.  将该 `ASCII` 码加上该位在机器码中的位置索引（位数）。
3.  再加上 `0x17`，结果除以 `1A`。
4.  余数加上 `0x41`，得到的每一位即为第一部分注册码。
5.  第一部分注册码的最后一位由前面已计算的注册码字符推导得出，计算方法与机器码处理类似。

### 第二部分：基于单位名称的注册码生成

进入处理第二部分注册码的地方，程序使用单位名称作为参数。

**关键调用：**

```asm
0065BA16    .  FF>call dword ptr ds:[eax+D8]                  ; 关键进入，用单位名称计算注册码的后半部分
```

**详细汇编分析：**

```asm
0065B4FC    .  8B>mov edx,dword ptr ds:[eax+48]               ; 得到单位名称
0065B4FF    .  E8>call <jmp.&MSVBVM60.__vbaStrCopy>  
0065B504    >  8D>lea eax,dword ptr ss:[ebp-1C]  
0065B507    .  89>mov dword ptr ss:[ebp-58],ebx  
0065B50A    .  89>mov dword ptr ss:[ebp-50],eax  
0065B50D    .  8D>lea eax,dword ptr ss:[ebp-58]  
0065B510    .  50 push eax  
0065B511    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B514    .  50 push eax  
0065B515    .  E8>call <jmp.&MSVBVM60.#520>  
0065B51A    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B51D    .  89>mov dword ptr ss:[ebp-60],edi  
0065B520    .  50 push eax  
0065B521    .  8D>lea eax,dword ptr ss:[ebp-68]  
0065B524    .  50 push eax  
0065B525    .  C7>mov dword ptr ss:[ebp-68],8008  
0065B52C    .  E8>call <jmp.&MSVBVM60.__vbaVarTstEq>  
0065B531    .  8D>lea ecx,dword ptr ss:[ebp-38]  
0065B534    .  8B>mov edi,eax  
0065B536    .  E8>call <jmp.&MSVBVM60.__vbaFreeVar>  
0065B53B    .  66>cmp di,si  
0065B53E    .  0F>jnz Waregood.0065B63A  
0065B544    .  FF>push dword ptr ss:[ebp-1C]  
0065B547    .  E8>call <jmp.&MSVBVM60.__vbaLenBstr>            ; 得到单位名称长度
0065B54C    .  8B>mov ecx,eax  
0065B54E    .  E8>call <jmp.&MSVBVM60.__vbaI2I4>  
0065B553    .  6A>push 1  
0065B555    .  89>mov dword ptr ss:[ebp-74],eax  
0065B558    .  5F pop edi  
0065B559    .  6A>push 2  
0065B55B    .  5E pop esi  
0065B55C    >  66>cmp di,word ptr ss:[ebp-74]  
0065B560    .  0F>jg Waregood.0065B62F  
0065B566    .  8D>lea eax,dword ptr ss:[ebp-1C]  
0065B569    .  C7>mov dword ptr ss:[ebp-30],1  
0065B570    .  89>mov dword ptr ss:[ebp-50],eax  
0065B573    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B576    .  50 push eax  
0065B577    .  89>mov dword ptr ss:[ebp-38],esi  
0065B57A    .  0F>movsx eax,di  
0065B57D    .  50 push eax  
0065B57E    .  8D>lea eax,dword ptr ss:[ebp-58]  
0065B581    .  50 push eax  
0065B582    .  8D>lea eax,dword ptr ss:[ebp-48]  
0065B585    .  50 push eax  
0065B586    .  89>mov dword ptr ss:[ebp-58],ebx  
0065B589    .  E8>call <jmp.&MSVBVM60.#632>                    ; 依次（从左到右）得到单位名称的每一位 ASCII
0065B58E    .  8D>lea eax,dword ptr ss:[ebp-48]  
0065B591    .  50 push eax  
0065B592    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>         ; 得到 ASCII 地址
0065B597    .  8B>mov edx,eax  
0065B599    .  8D>lea ecx,dword ptr ss:[ebp-24]  
0065B59C    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>  
0065B5A1    .  8D>lea eax,dword ptr ss:[ebp-48]  
0065B5A4    .  50 push eax  
0065B5A5    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B5A8    .  50 push eax  
0065B5A9    .  56 push esi  
0065B5AA    .  E8>call <jmp.&MSVBVM60.__vbaFreeVarList>  
0065B5AF    .  8B>mov eax,dword ptr ss:[ebp-20]  
0065B5B2    .  83>add esp,0C  
0065B5B5    .  89>mov dword ptr ss:[ebp-50],eax  
0065B5B8    .  C7>mov dword ptr ss:[ebp-58],8  
0065B5BF    .  FF>push dword ptr ss:[ebp-24]  
0065B5C2    .  E8>call <jmp.&MSVBVM60.#516>                    ; 将得到 ASCII 放于 EAX 中
0065B5C7    .  8B>mov ecx,eax  
0065B5C9    .  E8>call <jmp.&MSVBVM60.__vbaI2Abs>               ; 取绝对值
0065B5CE    .  66>cwd  
0065B5D0    .  66>mov cx,1A  
0065B5D4    .  66>idiv cx                                       ; 得到的 ASCII 除以 1A
0065B5D7    .  66>add dx,41                                     ; 余数加上 41
0065B5DB    .  0F>jo Waregood.0065B6A0  
0065B5E1    .  0F>movsx eax,dx  
0065B5E4    .  50 push eax  
0065B5E5    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B5E8    .  50 push eax  
0065B5E9    .  E8>call <jmp.&MSVBVM60.#608>                     ; 将得到的 ASCII 转化为字符
0065B5EE    .  8D>lea eax,dword ptr ss:[ebp-58]  
0065B5F1    .  50 push eax  
0065B5F2    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B5F5    .  50 push eax  
0065B5F6    .  8D>lea eax,dword ptr ss:[ebp-48]  
0065B5F9    .  50 push eax  
0065B5FA    .  E8>call <jmp.&MSVBVM60.__vbaVarCat>              ; 组合了
0065B5FF    .  50 push eax  
0065B600    .  E8>call <jmp.&MSVBVM60.__vbaStrVarMove>          ; 得到组合后的地址
0065B605    .  8B>mov edx,eax  
0065B607    .  8D>lea ecx,dword ptr ss:[ebp-20]  
0065B60A    .  E8>call <jmp.&MSVBVM60.__vbaStrMove>             ; 放于 EBP-20 中
0065B60F    .  8D>lea eax,dword ptr ss:[ebp-48]  
0065B612    .  50 push eax  
0065B613    .  8D>lea eax,dword ptr ss:[ebp-38]  
0065B616    .  50 push eax  
0065B617    .  56 push esi  
0065B618    .  E8>call <jmp.&MSVBVM60.__vbaFreeVarList>  
0065B61D    .  83>add esp,0C  
0065B620    .  6A>push 1  
0065B622    .  58 pop eax  
0065B623    .  66>add ax,di  
0065B626    .  70>jo short Waregood.0065B6A0  
0065B628    .  8B>mov edi,eax  
0065B62A    .^ E9>jmp Waregood.0065B55C  
0065B62F    >  8B>mov edx,dword ptr ss:[ebp-20]  
0065B632    .  8D>lea ecx,dword ptr ss:[ebp-28]  
```

**算法描述：**

依次取单位名称的 `ASCII` 码，每次得到的 `ASCII` 码除以 `1A`，得到的余数加上 `0x41` 即为第二部分注册码。

### 算法总结

注册码由两部分组成，中间用 `" - "`（空格、减号、空格）3 个字符隔开。

**1. 注册码前部分（基于机器码计算）：**

*   从机器码的倒数第 `2` 位开始，依次向前取每一位的 `ASCII` 码。
*   将该 `ASCII` 码加上该位在机器码中的位置索引。
*   再加上 `0x17`。
*   结果除以 `1A`，余数加上 `0x41`，得到的每一位即为第一部分注册码。
*   **最后一位计算：** 将前面已计算的注册码字符依次相加，并每次减 `2`，得到的数据除以 `1A`，余数加上 `41` 即为最后一位。

**2. 注册码后半部分（基于单位名称计算）：**

*   依次取单位名称的 `ASCII` 码。
*   每次得到的 `ASCII` 码除以 `1A`，余数加上 `0x41` 即为第二部分注册码。
*   注意：单位名称后软件会自动添加一个空格，该空格也会参与计算生成注册码的一部分。

### 注册机 C 语言原代码

```c
main(  )  

{  

static char jiqima[20],danwei[20],qian[20],hou[20];  

int i,len1,len2,zhj,yushu,zuih=0;  

scanf("%s",jiqima);  

scanf("%s",danwei);  

len1=strlen(jiqima)-1;  

len2=strlen(danwei);  

danwei[len2]=0x20;  

for(i=len1-1;i>=0;i--)  

{  

zhj=jiqima[i];  

zhj=zhj+i+0x18;  

yushu=zhj%0x1a;  

qian[len1-1-i]=yushu+0x41;  

zuih=zuih+qian[len1-1-i]-2;  

}  

qian[len1]=(zuih%0x1a)+0x41;  

for(i=0;i<=len2;i++)  

{  

hou[i]=(danwei[i]%0x1a)+0x41;  

}  

for(i=0;i<=len1;i++)  

printf("%c",qian[i]);  

printf(" - ");  

for(i=0;i<=len2;i++)  

printf("%c",hou[i]);  

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
