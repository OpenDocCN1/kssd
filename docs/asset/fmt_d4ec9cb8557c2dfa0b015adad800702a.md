
```
00456930   /$  83EC 44         sub esp,44
00456933   |.  83C9 FF        or ecx,FFFFFFFF
00456936   |.  33C0           xor eax,eax
00456938   |.  53            push ebx
00456939   |.  55            push ebp
0045693A   |.  56            push esi
0045693B   |.  8B7424 54      mov esi,dword ptr ss:[esp+54]
0045693F   |.  57            push edi
00456940   |.  8BFE          mov edi,esi                                  ;  注册码放入 EDI
00456942   |.  F2:AE         repne scas byte ptr es:[edi]                 ;  计算注册码的长度
00456944   |.  F7D1          not ecx
00456946   |.  49            dec ecx
00456947   |.  83F9 14       cmp ecx,14                                   ;  注册码必须 20 位
0045694A   |.  0F85 BE010000  jnz ashsnap.00456B0E
00456950   |.  8A4E 06       mov cl,byte ptr ds:[esi+6]                  ;  注册码第七位必须是"-"
00456953   |.  B0 2D         mov al,2D
00456955   |.  3AC8          cmp cl,al
00456957   |.  0F85 B1010000  jnz ashsnap.00456B0E
0045695D   |.  3846 0D       cmp byte ptr ds:[esi+D],al                  ;  注册码第十四位必须是"-"
00456960   |.  0F85 A8010000  jnz ashsnap.00456B0E
00456966   |.  6A 04         push 4
00456968   |.  8D4424 20      lea eax,dword ptr ss:[esp+20]
0045696C   |.  56            push esi
0045696D   |.  50            push eax
0045696E   |.  E8 BD24FFFF    call ashsnap.00448E30                      ;  提取注册码前四位
00456973   |.  8D6E 04       lea ebp,dword ptr ds:[esi+4]
00456976   |.  6A 01         push 1
00456978   |.  8D4C24 68      lea ecx,dword ptr ss:[esp+68]
0045697C   |.  55            push ebp
0045697D   |.  51            push ecx
0045697E   |.  E8 AD24FFFF    call ashsnap.00448E30                      ;  提取注册码第五位
00456983   |.  8D56 05       lea edx,dword ptr ds:[esi+5]
00456986   |.  6A 01         push 1
00456988   |.  8D4424 48      lea eax,dword ptr ss:[esp+48]
0045698C   |.  52            push edx
0045698D   |.  50            push eax
0045698E   |.  E8 9D24FFFF    call ashsnap.00448E30                      ;  提取注册码第六位("-"被忽略)
00456993   |.  8D4E 07       lea ecx,dword ptr ds:[esi+7]
00456996   |.  6A 02         push 2
00456998   |.  8D5424 38      lea edx,dword ptr ss:[esp+38]
0045699C   |.  51            push ecx
0045699D   |.  52            push edx
0045699E   |.  E8 8D24FFFF    call ashsnap.00448E30                      ;  提取注册码第八九位
004569A3   |.  8D46 09       lea eax,dword ptr ds:[esi+9]
004569A6   |.  6A 02         push 2
004569A8   |.  8D4C24 61      lea ecx,dword ptr ss:[esp+61]
004569AC   |.  50            push eax
004569AD   |.  51            push ecx
004569AE   |.  E8 7D24FFFF    call ashsnap.00448E30                      ;  提取注册码第十、十一位
004569B3   |.  8D56 0B       lea edx,dword ptr ds:[esi+B]
004569B6   |.  6A 02         push 2
004569B8   |.  8D4424 54      lea eax,dword ptr ss:[esp+54]
004569BC   |.  52            push edx
004569BD   |.  50            push eax
004569BE   |.  E8 6D24FFFF    call ashsnap.00448E30                      ;  分析注册码第十二、十三位
004569C3   |.  83C4 48       add esp,48
004569C6   |.  8D4E 0E       lea ecx,dword ptr ds:[esi+E]
004569C9   |.  8D5424 2F      lea edx,dword ptr ss:[esp+2F]
004569CD   |.  6A 03         push 3
004569CF   |.  51            push ecx
004569D0   |.  52            push edx
004569D1   |.  E8 5A24FFFF    call ashsnap.00448E30                      ;  分离注册码 15-17 位
004569D6   |.  8D46 11       lea eax,dword ptr ds:[esi+11]
004569D9   |.  6A 02         push 2
004569DB   |.  8D4C24 26      lea ecx,dword ptr ss:[esp+26]
004569DF   |.  50            push eax
004569E0   |.  51            push ecx
004569E1   |.  E8 4A24FFFF    call ashsnap.00448E30
004569E6   |.  8D56 13       lea edx,dword ptr ds:[esi+13]
004569E9   |.  6A 01         push 1
004569EB   |.  8D4424 4E      lea eax,dword ptr ss:[esp+4E]
004569EF   |.  52            push edx
004569F0   |.  50            push eax
004569F1   |.  E8 3A24FFFF    call ashsnap.00448E30                      ;  分离出注册码最后一位
004569F6   |.  8D4C24 50      lea ecx,dword ptr ss:[esp+50]              ;  字符串组合“onh96HN”
......
00456A54   |.  E8 D723FFFF    call ashsnap.00448E30
00456A59   |.  8B7C24 68      mov edi,dword ptr ss:[esp+68]             ;  固定字符串"sk()44$$GFSNM099023$"
00456A5D   |.  8D4424 40      lea eax,dword ptr ss:[esp+40]             ;  字符串组合"onh96HNA77AMAS"
00456A61   |.  57            push edi
00456A62   |.  6A 0E         push 0E                                    ;  常数 14(参数之一)
00456A64   |.  50            push eax
00456A65   |.  E8 46040000    call ashsnap.00456EB0                     ;  截取"sk()44$$GFSNM099023$"后 6 位放入 EDX
00456A6A   |.  57            push edi
00456A6B   |.  8D4C24 50      lea ecx,dword ptr ss:[esp+50]
00456A6F   |.  6A 0E         push 0E                                    ;  常数 14(参数之一)
00456A71   |.  51            push ecx
00456A72   |.  E8 B9030000    call ashsnap.00456E30                     ;  这里直接关系到 00456A8E 的值,关键
00456A77   |.  50            push eax
00456A78   |.  8D5424 4C      lea edx,dword ptr ss:[esp+4C]
00456A7C   |.  68 B0B94900    push ashsnap.0049B9B0                     ;  %04x
00456A81   |.  52            push edx
00456A82   |.  E8 6B0AFFFF    call ashsnap.004474F2
00456A87   |.  83C4 30       add esp,30
00456A8A   |.  8D7C24 14      lea edi,dword ptr ss:[esp+14]             ;  从注册码中提取出的值"EBD2"
00456A8E   |.  8D4424 24      lea eax,dword ptr ss:[esp+24]             ;  这个值是上面的关键 call 计算出的
00456A92   |>  8A10           /mov dl,byte ptr ds:[eax]                 ;  这一段循环很显然是比较上面两个值是否相等
00456A94   |.  8ACA           |mov cl,dl
......
00456AAE   |.  3ACB          |cmp cl,bl
00456AB0   |.^ 75 E0         \\jnz short ashsnap.00456A92
00456AB2   |>  33C0          xor eax,eax
00456AB4   |.  EB 05        jmp short ashsnap.00456ABB
```

追入 `00456A72` 关键处

代码:
