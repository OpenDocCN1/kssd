
```
009530EB    .  B9>mov ecx,1
009530F0    .  BA>mov edx,1
009530F5    .  8B>mov eax,dword ptr ss:[ebp-10]          ;  得到机器码文件中的内容
009530F8    .  E8>call unpacked.004054C8
009530FD    .  8B>mov eax,dword ptr ss:[ebp-1C]
00953100    .  BA>mov edx,unpacked.00953584
00953105    .  E8>call unpacked.004053B4
0095310A    .  75>jnz short unpacked.00953124
0095310C    .  8D>lea eax,dword ptr ss:[ebp-10]
0095310F    .  50 push eax
00953110    .  B9>mov ecx,1
00953115    .  BA>mov edx,2
0095311A    .  8B>mov eax,dword ptr ss:[ebp-10]
0095311D    .  E8>call unpacked.004054C8
00953122    .  EB>jmp short unpacked.0095313A
00953124    >  8D>lea eax,dword ptr ss:[ebp-10]
00953127    .  50 push eax
00953128    .  B9>mov ecx,2
0095312D    .  BA>mov edx,1
00953132    .  8B>mov eax,dword ptr ss:[ebp-10]
00953135    .  E8>call unpacked.004054C8
0095313A    >  8B>mov eax,dword ptr ss:[ebp-8]
0095313D    .  E8>call unpacked.0040AA5C
00953142    .  84>test al,al
00953144    .  0F>jnz unpacked.009532AC                  ;  判断是否输入
0095314A    .  A1>mov eax,dword ptr ds:[95BFF0]
0095314F    .  C6>mov byte ptr ds:[eax],0
```

……………………省略………………………………

```
009532E2    .  8D>lea eax,dword ptr ss:[ebp-3BC]
009532E8    .  50 push eax
009532E9    .  B9>mov ecx,7B9
009532EE    .  BA>mov edx,7B8
009532F3    .  8B>mov eax,dword ptr ss:[ebp-18]          ;  又得到另一文件内容
009532F6    .  E8>call unpacked.0066DC58
009532FB    .  8B>mov edx,dword ptr ss:[ebp-3BC]         ;  将转化后储存的输入码又转化过来了
00953301    .  8D>lea eax,dword ptr ss:[ebp-18]
00953304    .  E8>call unpacked.00405040
00953309    .  8D>lea eax,dword ptr ss:[ebp-3B4]
0095330F    .  E8>call unpacked.004031C0
00953314    .  E8>call unpacked.004029AC
00953319    .  33>xor eax,eax
0095331B    .  55 push ebp
0095331C    .  68>push unpacked.00953339
00953321    .  64>push dword ptr fs:[eax]
00953324    .  64>mov dword ptr fs:[eax],esp
00953327    .  8D>lea eax,dword ptr ss:[ebp-14]
0095332A    .  E8>call unpacked.0092C714                 ;  将转化后储存的机器码又转化过来了
0095332F    .  33>xor eax,eax
00953331    .  5A pop edx
00953332    .  59 pop ecx
00953333    .  59 pop ecx
00953334    .  64>mov dword ptr fs:[eax],edx
00953337    .  EB>jmp short unpacked.00953350
00953339    .^ E9>jmp unpacked.00404598
0095333E    .  8D>lea eax,dword ptr ss:[ebp-14]
00953341    .  BA>mov edx,unpacked.009535D0              ;  ASCII "L8U9X575Y503Z31"
00953346    .  E8>call unpacked.00405040
0095334B    .  E8>call unpacked.004049C4
00953350    >  83>cmp dword ptr ss:[ebp-14],0            ;  判断机器码是否等于0
00953354    .  75>jnz short unpacked.00953363
00953356    .  8D>lea eax,dword ptr ss:[ebp-14]
00953359    .  BA>mov edx,unpacked.009535D0              ;  ASCII "L8U9X575Y503Z31"
0095335E    .  E8>call unpacked.00405040
00953363    >  8D>lea ecx,dword ptr ss:[ebp-3C0]
00953369    .  8B>mov edx,dword ptr ds:[95C6D4]          ;  用到一个固定值815E
0095336F    .  66>mov dx,word ptr ds:[edx]               ;  将这个固定值取出来
00953372    .  8B>mov eax,dword ptr ss:[ebp-18]          ;  得到输入码
00953375    .  E8>call unpacked.0092C69C                 ;  真正关键的地方  对输入码进行处理的地方
0095337A    .  8B>mov edx,dword ptr ss:[ebp-3C0]
00953380    .  8B>mov eax,dword ptr ss:[ebp-14]          ;  机器码
00953383    .  E8>call unpacked.004053B4                 ;  上面计算出来的值与机器码比较了！
00953388    .  75>jnz short unpacked.00953397
0095338A    .  A1>mov eax,dword ptr ds:[95BFF0]          ;  标志位置1 已注册啊
0095338F    .  C6>mov byte ptr ds:[eax],1
00953392    .  E9>jmp unpacked.009534F4
00953397    >  A1>mov eax,dword ptr ds:[95BFF0]
0095339C    .  C6>mov byte ptr ds:[eax],0                ;  标志位置0 未注册啊
0095339F    .  33>xor edx,edx
009533A1    .  55 push ebp
```

…………………………………………………………………………………………………………………………

进入 `00953375    .  E8>call unpacked.0092C69C` ; 真正关键的地方

…………………………………………………………………………………………………………………………

```
0092C6A8   |.  8B>mov ebx,eax
0092C6AA   |.  33>xor eax,eax
0092C6AC   |.  55 push ebp
0092C6AD   |.  68>push unpacked.0092C6E4
0092C6B2   |.  64>push dword ptr fs:[eax]
0092C6B5   |.  64>mov dword ptr fs:[eax],esp
0092C6B8   |.  8D>lea edx,dword ptr ss:[ebp-4]
0092C6BB   |.  8B>mov eax,ebx                            ;  得到输入码
0092C6BD   |.  E8>call unpacked.0092C58C                 ;  关键处理1
0092C6C2   |.  8B>mov eax,dword ptr ss:[ebp-4]           ;  结果地址
0092C6C5   |.  8B>mov ecx,edi
0092C6C7   |.  8B>mov edx,esi                            ;  固定值 815E
0092C6C9   |.  E8>call unpacked.0092C628                 ;  再次处理
0092C6CE   |.  33>xor eax,eax
0092C6D0   |.  5A pop edx
0092C6D1   |.  59 pop ecx
0092C6D2   |.  59 pop ecx
0092C6D3   |.  64>mov dword ptr fs:[eax],edx
0092C6D6   |.  68>push unpacked.0092C6EB
0092C6DB   |>  8D>lea eax,dword ptr ss:[ebp-4]
```

…………………………………………………………………………………………………………………………

里面有两次对输入码的处理，处理的结果必须要与机器码相等，即为注册成功

进入处理1：`0092C6BD   |.  E8>call unpacked.0092C58C`

…………………………………………………………………………………………………………………………

```
0092C5A9   |.  8D>lea eax,dword ptr ss:[ebp-4]
0092C5AC   |.  8B>mov edx,esi                            ;  得到输入码
0092C5AE   |.  E8>call unpacked.00405040
0092C5B3   |.  8B>mov eax,ebx
0092C5B5   |.  E8>call unpacked.00404FA8
0092C5BA   |.  EB>jmp short unpacked.0092C5F9
0092C5BC   |>  8D>/lea eax,dword ptr ss:[ebp-C]
0092C5BF   |.  50 |push eax
0092C5C0   |.  B9>|mov ecx,4
0092C5C5   |.  BA>|mov edx,1
0092C5CA   |.  8B>|mov eax,dword ptr ss:[ebp-4]          ;  得到输入码
0092C5CD   |.  E8>|call unpacked.004054C8                ;  每次取4位进行计算
0092C5D2   |.  8B>|mov eax,dword ptr ss:[ebp-C]          ;  取4位的结果
0092C5D5   |.  8D>|lea edx,dword ptr ss:[ebp-8]
0092C5D8   |.  E8>|call unpacked.0092C458                ;  关键  每4位一组进行计算   计算的结果在EDX中
0092C5DD   |.  8B>|mov edx,dword ptr ss:[ebp-8]          ;  得到计算好的地址
0092C5E0   |.  8B>|mov eax,ebx
0092C5E2   |.  E8>|call unpacked.00405278                ;  把每次计算好的组合了
0092C5E7   |.  8D>|lea eax,dword ptr ss:[ebp-4]
0092C5EA   |.  B9>|mov ecx,4
0092C5EF   |.  BA>|mov edx,1
0092C5F4   |.  E8>|call unpacked.00405508                ;  去掉已经计算过的4位
0092C5F9   |>  83> cmp dword ptr ss:[ebp-4],0            ;  是否取完
0092C5FD   |.^ 75>\jnz short unpacked.0092C5BC
0092C5FF   |.  33>xor eax,eax
0092C601   |.  5A pop edx
0092C602   |.  59 pop ecx
0092C603   |.  59 pop ecx
0092C604   |.  64>mov dword ptr fs:[eax],edx
0092C607   |.  68>push unpacked.0092C621
```

………………………………………………………………………………………………………………………………

每次4位输入码进行计算，计算为3组结果，若最后一次如果没有4位：  
若只有3位了，计算出2位结果，  
如果只有2位了，计算出1位结果，输入码的位数不可能除以4后余1的。

下面进入：`0092C5D8   |.  E8>|call unpacked.0092C458` ; 关键 每4位一组进行计算

……………………………………………………………………………………………………………………

```
0092C45B   |.  8B>mov esi,edx
0092C45D   |.  8B>mov ebx,eax
0092C45F   |.  8B>mov eax,ebx
0092C461   |.  E8>call unpacked.00405270                 ;  得到位数！！
0092C466   |.  83>sub eax,2                              ;  Switch (cases 2..4)
0092C469   |.  74>je short unpacked.0092C47A             ;  这里跳到是最后只有2位的情况
0092C46B   |.  48 dec eax
0092C46C   |.  74>je short unpacked.0092C4C5             ;  这里跳到是最后只有3位的情况
0092C46E   |.  48 dec eax
0092C46F   |.  0F>je unpacked.0092C51E                   ;  这里跳到是有4位的正常情况
0092C475   |.  E9>jmp unpacked.0092C586
0092C47A   |>  33>xor eax,eax                            ;  这里处理还有2位的情况; Case 2 of switch 0092C466
0092C47C   |.  8A>mov al,byte ptr ds:[ebx]
0092C47E   |.  0F>movzx eax,byte ptr ds:[eax+95BB5C]     ;  第1位在密码表中取值
0092C485   |.  33>xor edx,edx
0092C487   |.  8A>mov dl,byte ptr ds:[ebx+1]
0092C48A   |.  0F>movzx edx,byte ptr ds:[edx+95BB5C]     ;  第2位在密码表中取值
0092C491   |.  C1>shl edx,6
0092C494   |.  03>add eax,edx                            ;  第2次取值的结果，左移6，加上第1位取值的结果
0092C496   |.  89>mov dword ptr ss:[esp],eax
0092C499   |.  8B>mov eax,esi
0092C49B   |.  BA>mov edx,1
0092C4A0   |.  E8>call unpacked.004055F4
0092C4A5   |.  8B>mov eax,dword ptr ds:[esi]
0092C4A7   |.  E8>call unpacked.00405270
0092C4AC   |.  50 push eax
0092C4AD   |.  8B>mov eax,esi
```
