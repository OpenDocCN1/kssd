
```asm
0042DBA2    .   25>and eax,80000001  

0042DBA7    .   79>jns short NetSpeed.0042DBAE  

0042DBA9    .   48 dec eax  

0042DBAA    .   83>or eax,FFFFFFFE  

0042DBAD    .   40 inc eax  

0042DBAE    >   75>jnz short NetSpeed.0042DC16  

0042DBB0    .   8B>mov ecx,dword ptr ss:\[esp+2C\]  

0042DBB4    .   BB>mov ebx,3E9  

0042DBB9    .   0F>movsx ebp,byte ptr ds:\[ecx+5\]  

0042DBBD    >   8D>lea edx,dword ptr ss:\[esp+24\]  

0042DBC1    .   8D>lea ecx,dword ptr ss:\[esp+24\]  

0042DBC5    .   52 push edx  

0042DBC6    .   E8>call NetSpeed.004A4588  

0042DBCB    .   4B dec ebx  

0042DBCC    .^  75>jnz short NetSpeed.0042DBBD  

0042DBCE    .   51 push ecx  

0042DBCF    .   8B>mov ecx,dword ptr ss:\[esp+68\]  

0042DBD3    .   8B>mov eax,esp  

0042DBD5    .   89>mov dword ptr ss:\[esp+14\],esp  

0042DBD9    .   8D>lea edx,dword ptr ss:\[esp+14\]  

0042DBDD    .   89>mov dword ptr ds:\[eax\],ecx  

0042DBDF    .   8D>lea eax,dword ptr ss:\[esp+24\]  

0042DBE3    .   52 push edx  

0042DBE4    .   50 push eax  

0042DBE5    .   E8>call NetSpeed.00494552  

0042DBEA    .   8B>mov ecx,eax  

0042DBEC    .   E8>call NetSpeed.0040273E  

0042DBF1    .   8B>mov eax,dword ptr ds:\[eax\]  

0042DBF3    .   83>cmp eax,14  

0042DBF6    .   89>mov dword ptr ss:\[esp+1C\],eax  

0042DBFA    .   7C>jl short NetSpeed.0042DC04  

0042DBFC    .   6A>push 12                              ; /ExitCode = 12 (18.)  

0042DBFE    .   FF>call dword ptr ds:\[<&USER32.PostQuitMe>; \PostQuitMessage  

0042DC04    >   8B>mov eax,dword ptr ss:\[esp+2C\]  

0042DC08    .   8A>mov cl,byte ptr ds:\[eax+1\]  

0042DC0B    .   0F>movsx edx,byte ptr ds:\[eax+8\]  

0042DC0F    .   0F>movsx eax,cl  

0042DC12    .   03>add eax,ebp  

0042DC14    .   EB>jmp short NetSpeed.0042DC82  

0042DC16    >   8B>mov eax,dword ptr ss:\[esp+2C\]      ;  又得到注册码的全部  

0042DC1A    .   BD>mov ebp,3E9                          ;  准备循环  

0042DC1F    .   8A>mov cl,byte ptr ds:\[eax+5\]         ;  取注册码的第六位固定为 D  

0042DC22    .   0F>movsx ebx,byte ptr ds:\[eax+9\]      ;  取第 10 位  

0042DC26    .   0F>movsx edx,cl  

0042DC29    .   03>add ebx,edx                          ;  第 6 位加上第 10 位的 ASCII  

0042DC2B    >   8D>lea eax,dword ptr ss:\[esp+24\]  

0042DC2F    .   8D>lea ecx,dword ptr ss:\[esp+24\]  

0042DC33    .   50 push eax  

0042DC34    .   E8>call NetSpeed.004A4588  

0042DC39    .   4D dec ebp  

0042DC3A    .^  75>jnz short NetSpeed.0042DC2B          ;  和上一次一样无意义的循环  

0042DC3C    .   51 push ecx  

0042DC3D    .   8B>mov ecx,dword ptr ss:\[esp+68\]      ;  取时间计算值  

0042DC41    .   8B>mov eax,esp  

0042DC43    .   89>mov dword ptr ss:\[esp+14\],esp  

0042DC47    .   8D>lea edx,dword ptr ss:\[esp+14\]  

0042DC4B    .   89>mov dword ptr ds:\[eax\],ecx  

0042DC4D    .   8D>lea eax,dword ptr ss:\[esp+24\]  

0042DC51    .   52 push edx  

0042DC52    .   50 push eax  

0042DC53    .   E8>call NetSpeed.00494552  

0042DC58    .   8B>mov ecx,eax  

0042DC5A    .   E8>call NetSpeed.0040273E  

0042DC5F    .   8B>mov eax,dword ptr ds:\[eax\]  

0042DC61    .   83>cmp eax,14  

0042DC64    .   89>mov dword ptr ss:\[esp+1C\],eax  

0042DC68    .   7C>jl short NetSpeed.0042DC72           ;  如果小于 14 秒就出错  

0042DC6A    .   6A>push 12                              ; /ExitCode = 12 (18.)  

0042DC6C    .   FF>call dword ptr ds:\[<&USER32.PostQuitMe>; \PostQuitMessage  

0042DC72    >   8B>mov eax,dword ptr ss:\[esp+2C\]      ;  得到注册码  

0042DC76    .   8A>mov cl,byte ptr ds:\[eax+B\]         ;  取值第 12 位  

0042DC79    .   0F>movsx edx,byte ptr ds:\[eax+8\]      ;  取第九位  

0042DC7D    .   0F>movsx eax,cl  

0042DC80    .   03>add eax,ebx  

0042DC82    >   03>add eax,edx                          ;  把第 6 位，第 10 位，第 9 位第 12 的 ASCII 加起来  

0042DC84    .   8D>lea ecx,dword ptr ss:\[esp+34\]  

0042DC88    .   50 push eax  

0042DC89    .   68>push NetSpeed.0050F6B0               ;  ASCII "%d"  

0042DC8E    .   51 push ecx  

0042DC8F    .   E8>call NetSpeed.00493FA9               ;  转化位 10 进制  

0042DC94    .   8B>mov edx,dword ptr ss:\[esp+70\]      ;  又得到时间值  

0042DC98    .   83>add esp,0C  

0042DC9B    .   51 push ecx  

0042DC9C    .   8D>lea ecx,dword ptr ss:\[esp+24\]  

0042DCA0    .   8B>mov eax,esp  

0042DCA2    .   89>mov dword ptr ss:\[esp+14\],esp  

0042DCA6    .   89>mov dword ptr ds:\[eax\],edx  

0042DCA8    .   8D>lea eax,dword ptr ss:\[esp+14\]  

0042DCAC    .   50 push eax  

0042DCAD    .   51 push ecx  

0042DCAE    .   E8>call NetSpeed.00494552  

0042DCB3    .   8B>mov ecx,eax  

0042DCB5    .   E8>call NetSpeed.0040273E  

0042DCBA    .   8B>mov eax,dword ptr ds:\[eax\]  

0042DCBC    .   83>cmp eax,14  

0042DCBF    .   89>mov dword ptr ss:\[esp+1C\],eax  

0042DCC3    .   7C>jl short NetSpeed.0042DCCD           ;  小于 14 秒错误  

0042DCC5    .   6A>push 12                              ; /ExitCode = 12 (18.)  

0042DCC7    .   FF>call dword ptr ds:\[<&USER32.PostQuitMe>; \PostQuitMessage  

0042DCCD    >   8B>mov edx,dword ptr ss:\[esp+34\]      ;  得到上面之和 10 进制字符  

0042DCD1    .   BB>mov ebx,3  

0042DCD6    .   8B>mov eax,dword ptr ds:\[edx-8\]  

0042DCD9    .   3B>cmp eax,ebx                          ;  比较相加的和是否等于 3 位数  

0042DCDB    .   7D>jge short NetSpeed.0042DCF6  

0042DCDD    >   68>push NetSpeed.0050F0CC  

0042DCE2    .   8D>lea ecx,dword ptr ss:\[esp+38\]  

0042DCE6    .   E8>call NetSpeed.004A482B  

0042DCEB    .   8B>mov eax,dword ptr ss:\[esp+34\]  

0042DCEF    .   39>cmp dword ptr ds:\[eax-8\],ebx  

0042DCF2    .^  7C>jl short NetSpeed.0042DCDD  

0042DCF4    .   EB>jmp short NetSpeed.0042DD28  

0042DCF6    >   8D>lea ecx,dword ptr ss:\[esp+10\]  

0042DCFA    .   53 push ebx  

0042DCFB    .   51 push ecx  

0042DCFC    .   8D>lea ecx,dword ptr ss:\[esp+3C\]  

0042DD00    .   E8>call NetSpeed.00493AB7  

0042DD05    .   50 push eax  

0042DD06    .   8D>lea ecx,dword ptr ss:\[esp+38\]  

0042DD0A    .   C6>mov byte ptr ss:\[esp+B7C0\],2C  

0042DD12    .   E8>call NetSpeed.004A4588  

0042DD17    .   8D>lea ecx,dword ptr ss:\[esp+10\]  

0042DD1B    .   C6>mov byte ptr ss:\[esp+B7BC\],29  

0042DD23    .   E8>call NetSpeed.004A444F  

0042DD28    >   8B>mov edx,dword ptr ss:\[esp+24\]  

0042DD2C    .   68>push NetSpeed.0050F704               ; /Arg2 = 0050F704  

0042DD31    .   52 push edx                              ; |Arg1  

0042DD32    .   E8>call NetSpeed.00474355               ; \比较第 21 位是否位“－”  

0042DD37    .   83>add esp,8  

0042DD3A    .   85>test eax,eax  

0042DD3C    .   0F>je NetSpeed.0042DFA9  

0042DD42    .   8D>lea eax,dword ptr ss:\[esp+10\]  

0042DD46    .   6A>push 3  

0042DD48    .   50 push eax  

0042DD49    .   8D>lea ecx,dword ptr ss:\[esp+34\]  

0042DD4D    .   E8>call NetSpeed.00493AB7               ;  取最后 3 位  

0042DD52    .   8B>mov eax,dword ptr ds:\[eax\]  

0042DD54    .   8B>mov ecx,dword ptr ss:\[esp+34\]      ;  重要  

0042DD58    .   50 push eax                              ; /Arg2  

0042DD59    .   51 push ecx                              ; |Arg1  

0042DD5A    .   E8>call NetSpeed.00474355               ; \最后 3 位是否等于前面之和  

0042DD5F    .   83>add esp,8  

0042DD62    .   8D>lea ecx,dword ptr ss:\[esp+10\]  

0042DD66    .   85>test eax,eax  

0042DD68    .   0F>sete bl  

0042DD6B    .   E8>call NetSpeed.004A444F  

0042DD70    .   84>test bl,bl  

0042DD72    .   0F>je NetSpeed.0042DFA9  

0042DD78    .   E8>call NetSpeed.004023D8  

0042DD7D    .   84>test al,al  

0042DD7F    .   0F>je NetSpeed.0042DFA9
```

……………………………………………………………………………………………………………………

最后，把第 6 位，第 10 位，第 9 位第 12 的 ASCII 加起来，结果变成 10 进制数据。

如果你输入的最后 3 位等于上面的 10 进制之和，注册成功！

输入码与姓名与邮箱无关：

最终注册码：

`W2045D7090120456w0901223`

……………………………………………………………………………………………………………………

`4dd466d0e1e0cc123afbe504da2462ca`
