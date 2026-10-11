
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
