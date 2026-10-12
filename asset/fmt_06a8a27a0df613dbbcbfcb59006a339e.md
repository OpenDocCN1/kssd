
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
