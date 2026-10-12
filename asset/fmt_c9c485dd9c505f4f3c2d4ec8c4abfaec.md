
```
0040A1FC   cmp edi,ebx  
0040A1FE   jle short smrpro.0040A210  
0040A200   mov dl,byte ptr ss:[esp+eax+6C] ;取复制的另一份字串,dl=53("S")  
0040A204   xor dl,cl                         ;与上面运算后的结果或  
0040A206   inc ecx  
0040A207   mov byte ptr ss:[esp+eax+6C],dl ;结果存回原位置  
0040A20B   inc eax  
0040A20C   cmp eax,edi                       ;比较是否取完字串  
0040A20E   jl short smrpro.0040A200  
```

-----------------------------------------

上面这一小段更简单了,把上面计算得到的值 `0x18h` 再与字串 `"SMRPRO0000"` 的每位或存回原处,在内存中其 Hex 值显示为 `4B 54 48 4B 4E 52 2E 2F 10 11`

★★★★★★★★★★★★★★★★★★★★★★★

```
0040A210   xor esi,esi  
0040A212   cmp edi,ebx  
0040A214   jle short smrpro.0040A230  
0040A216   xor eax,eax  
0040A218   mov ebp,35                        ;ebp 赋值 35,后面将参与运算  
0040A21D   mov al,byte ptr ss:[esp+esi+6C];上面运算后的结果再按位取:4B 54 48 4B 4E 52 2E 2F 10 11  
0040A221   cdq                               ;扩展成双字  
0040A222   idiv ebp  
0040A224   inc esi  
0040A225   cmp esi,edi  
0040A227   mov byte ptr ss:[esp+esi+A3],dl;余数存入堆栈中"16 1F 13 16 19 1D 2E 2F 10 11"  
0040A22E   jl short smrpro.0040A216  
```

★★★★★★★★★★★★★★★★★★★★★★★

这上面一小段可描述为将上面或运算后得到的值每位再分别除以 35 将余数存入一地址

```
0040A230   mov eax,ecx                       ;eax=ecx=22,即(0A+1)*2,往上看应该不难理解  
0040A232   mov ecx,35                        
0040A237   cdq  
0040A238   idiv ecx                          ;余数为 22  
0040A23A   mov eax,dword ptr ss:[esp+11C] ;ss:[esp+11c],可能作 buffer 用  
0040A241   xor esi,esi  
0040A243   xor cl,cl  
```

------------------------------------------

```
0040A245   mov byte ptr ss:[esp+13],bl  
0040A249   mov byte ptr ss:[esp+18],cl  
0040A24D   mov byte ptr ss:[esp+14],bl  
0040A251   mov dword ptr ss:[esp+54],ebx  
0040A255   mov dword ptr ss:[esp+58],eax  
0040A259   mov dword ptr ss:[esp+5C],edx ;余数存入 ss:[esp+5C]  
```

------------------------------------------

这一段给特定临时变量赋值 0,后面会以用到

```
0040A25D   mov edx,dword ptr ss:[esp+118]  
0040A264   sub edx,eax                       ;edx=18  
0040A266   mov dword ptr ss:[esp+60],edx  
```

下面开始一个大循环

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
0040A26A   mov edx,dword ptr ss:[esp+60]     
0040A26E   mov eax,dword ptr ss:[esp+58]  
0040A272   mov edi,dword ptr ss:[esp+120] ;edi=0A  
0040A279   movsx eax,byte ptr ds:[edx+eax]  
```

-----------------------------------------------

`ds:[edx+eax]` 指向我们的假码转换后的值 `4D 40 20 00 B0 78 07 12 87 12`

```
0040A27D   mov edx,dword ptr ss:[esp+54] ;还记得上面的一小段赋值吗?现在用到了,后面还会用到  
0040A281   cmp edx,edi                       ;循环判断,edi 中为 0A  
0040A283   jnb smrpro.0040A3AF              
0040A289   mov edx,dword ptr ss:[esp+128]  
0040A290   test edx,edx  
0040A292   je short smrpro.0040A2D3          ;第 1 次此处会跳  
```

下面的这一段看起来有点乱,我的描述能力有限,只好在每行后加注释,看不明白的话最好自己跟一下

就清楚了

```
0040A294   mov bl,byte ptr ss:[esp+esi+6C]  bl=4B ,"  
0040A298   mov dl,byte ptr ss:[esp+esi+DC]  dl=53 ,"  
0040A29F   mov edi,ebx  
0040A2A1   mov ebp,edx  
0040A2A3   and edi,0FF  
0040A2A9   and ebp,0FF  
0040A2AF   xor edi,ebp  
0040A2B1   mov ebp,dword ptr ss:[esp+14]  
0040A2B5   and ebp,0FF  
0040A2BB   mov byte ptr ss:[esp+14],cl  
0040A2BF   xor edi,ebp  
0040A2C1   mov ebp,dword ptr ss:[esp+18]  
0040A2C5   and ebp,0FF  
0040A2CB   xor edi,ebp  
0040A2CD   xor eax,edi  
0040A2CF   mov cl,al  
0040A2D1   jmp short smrpro.0040A316  
0040A2D3   mov bl,byte ptr ss:[esp+esi+6C];接上跳转,bl=4B,"4B 54 48 4B 4E 52 2E 2F 10 11"  
0040A2D7   mov dl,byte ptr ss:[esp+esi+DC];dl=53,"53 4D 52 50 52 4F 30 30 30 30"  
0040A2DE   mov edi,ebx                       
0040A2E0   mov ebp,edx  
0040A2E2   and edi,0FF                       ;取低位,edi=4B  
0040A2E8   and ebp,0FF                       ;取低位,edi=53  
0040A2EE   xor edi,ebp                       ;edi=4B⁵³=18  
0040A2F0   mov ebp,dword ptr ss:[esp+14]   ;ss:[esp+14]前面赋过值  
0040A2F4   and ebp,0FF                       ;取低位,ebp=0  
0040A2FA   mov dword ptr ss:[esp+68],eax   ;eax=4D,往前看,假码转换后的  
0040A2FE   xor edi,ebp                       ;edi=18⁰=18  
0040A300   mov ebp,dword ptr ss:[esp+18]    
0040A304   and ebp,0FF                       ;ebp=0  
0040A30A   mov byte ptr ss:[esp+14],cl     ;cl 的值先保存在 ss:[esp+14]下一轮循环用  
0040A30E   mov cl,byte ptr ss:[esp+68]     ;cl=4D  
0040A312   xor edi,ebp                       ;edi=18⁰=18  
0040A314   xor eax,edi                       ;eax=4D¹⁸=55  
0040A316   mov edi,dword ptr ss:[esp+58]   ;ss:[esp+58]可能作为一个缓冲地址  
0040A31A   mov ebp,dword ptr ss:[esp+64]  
0040A31E   mov byte ptr ss:[esp+18],cl     ;cl=4D 存入 ss:[esp+18],下一轮循环用  
0040A322   mov byte ptr ds:[edi],al        ;果然,al 的值存入  
0040A324   mov eax,dword ptr ss:[esp+54]  ;ss:[esp+54]初始为 0,保存控制循环  
0040A328   inc eax                           ;步进,调整指针  
0040A329   inc edi                           ;同上  
0040A32A   mov dword ptr ss:[esp+54],eax   ;果然,再存入  
0040A32E   xor eax,eax  
0040A330   mov al,byte ptr ss:[esp+esi+A4] ;这里存着什么呢,al=16,"16 1F 13 16 19 1D 2E 2F 10 11"  
0040A337   mov dword ptr ss:[esp+58],edi   ;保存 edi  
0040A33B   mov edi,dword ptr ss:[esp+5C]    ;ss:[esp+5C]中存放着余数 22  
0040A33F   mov al,byte ptr ss:[esp+eax+1C] ;还记得刚开始建的一张大表吗,这里就是由 eax 值查表,al=59  
0040A343   add al,dl                         ;al=59+53=AC  
0040A345   mov dl,byte ptr ss:[esp+edi+1C] ;由 edi 值查表,dl=97  
0040A349   add dl,bl                         ;dl=97+4B=E2  
0040A34B   mov byte ptr ss:[esp+esi+DC],al ;结果存回原处  
0040A352   mov byte ptr ss:[esp+esi+6C],dl ;结果存回原处  
0040A356   inc esi  
0040A357   cmp esi,ebp  
0040A359   jnz smrpro.0040A26A  
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

这一段看起来不少,我们尽量简单描述成如下：(40A2D3~40A322 段代码为以下运算)

`4B 54 48 4B 4E 52 2E 2F 10 11` 看上面的解释吧  
`53 4D 52 50 52 4F 30 30 30 30` 对应字串 `"SMRPRO0000"`  
`00 00 4D 40 20 00 20 78 07 12`  
`00 4D 40 20 00 20 78 07 12 87`  
`4D 40 20 00 20 78 07 12 87 12`  
---------------------------------按位或运算  
`55 14 37 7B 1C 45 41 72 B2 A6`

而 40A330 至 40A359 处代码实现由余数值 `"16 1F 13 16 19 1D 2E 2F 10 11"`  
查表与 `"53 4D 52 50 52 4F 30 30 30 30"` 的每位加运算,结果为  
`ss:[esp+esi+DC]` 中:`AC D6 9B A9 B9 CE 0F 13 6D 73`  
以及用余数值 22 查表与 `"4B 54 48 4B 4E 52 2E 2F 10 11 "` 的每位加  
运算(即每位加上 97 舍去高位),结果  
在 `ss:[esp+esi+6C]` 中为 `E2 EB DF E2 E5 E9 C5 C6 A7 A8` (为定值)

```
0040A35F   mov al,byte ptr ss:[esp+13]  
0040A363   inc al  
0040A365   mov byte ptr ss:[esp+13],al  
0040A369   jnz short smrpro.0040A3A8  
0040A36B   xor edx,edx  
0040A36D   xor eax,eax  
0040A36F   cmp ebp,edx  
0040A371   jle short smrpro.0040A3A8  
0040A373   mov bl,byte ptr ss:[esp+eax+A4]  
0040A37A   inc bl  
0040A37C   cmp bl,35  
0040A37F   mov byte ptr ss:[esp+eax+A4],bl  
0040A386   jnz short smrpro.0040A38F  
0040A388   mov byte ptr ss:[esp+eax+A4],dl  
0040A38F   mov bl,byte ptr ss:[esp+eax+1C]  
0040A393   add byte ptr ss:[esp+eax+6C],bl  
0040A397   inc edi  
0040A398   cmp edi,35  
0040A39B   jnz short smrpro.0040A39F  
0040A39D   xor edi,edi  
0040A39F   inc eax  
0040A3A0   cmp eax,ebp  
0040A3A2   jl short smrpro.0040A373  
0040A3A4   mov dword ptr ss:[esp+5C],edi  
0040A3A8   xor esi,esi  
0040A3AA   jmp smrpro.0040A26A  
0040A3AF   mov eax,dword ptr ss:[esp+11C]  
0040A3B6   pop edi  
0040A3B7   pop esi  
0040A3B8   pop ebp  
0040A3B9   mov byte ptr ds:[edx+eax],0  
0040A3BD   xor eax,eax  
0040A3BF   pop ebx  
0040A3C0   add esp,104  
0040A3C6   retn  
```

这个函数的返回值为或运算后的结果 `55 14 37 7B 1C 45 41 72 B2 A6`,其余两个查表后得到的值在注册码的校验中没有用到,在后面的写注册信息时用到!对此我不感兴趣,有兴趣的自己跟一下!

### 算法总结

说实话,写到这有点头晕眼花了,死老外,弄个这么麻烦的算法!还是稍微梳理一下吧!

注册码与用户名无关,注册码须为 20 位,其中第 6、7、8 位须为 0,第 9 位不得为 0,假设我们的注册码为 `abcde000fghijklmnopq`,经 `004208D4~00420909` 处的代码转换为 `n1n2n3n4n5n6n7n8n9n10`

`a b c d e 0 0 0 f g h i j k l m n o p q`  
`┕┘ ┕┘  ┕┘  ┕┘  ┕┘  ┕┘  ┕┘  ┕┘  ┕┘  ┕┘`  
`n1 n2 n3 n4 n5 n6 n7 n8 n9 n10`

然后后将此值与几组程序内定值按位或运算后得到一个值

`4B 54 48 4B 4E 52 2E 2F 10 11` 固定值
