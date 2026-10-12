
```
00644424    .  FF91 0C030000   call dword ptr ds:[ecx+30C]
0064442A    .  8D95 70FFFFFF   lea edx,dword ptr ss:[ebp-90]
00644430    .  50              push eax
00644431    .  52              push edx
00644432    .  FF15 8C104000   call dword ptr ds:[<&MSVBVM60.__vbaObjSet>]     ;  MSVBVM60.__vbaObjSet
00644438    .  8BD8            mov ebx,eax
0064443A    .  8D4D 88         lea ecx,dword ptr ss:[ebp-78]
0064443D    .  51              push ecx
0064443E    .  53              push ebx
0064443F    .  8B03            mov eax,dword ptr ds:[ebx]
00644441    .  FF90 A0000000   call dword ptr ds:[eax+A0]
00644447    .  3BC6            cmp eax,esi
00644449    .  DBE2            fclex
0064444B    .  7D 12          jge short yyjxc.0064445F
0064444D    .  68 A0000000     push 0A0
00644452    .  68 585E4300     push yyjxc.00435E58
00644457    .  53              push ebx
00644458    .  50              push eax
00644459    .  FF15 68104000   call dword ptr ds:[<&MSVBVM60.__vbaHresultCheck>;  MSVBVM60.__vbaHresultCheckObj
0064445F    >  8B45 88         mov eax,dword ptr ss:[ebp-78]                 ;  取机器码的字符串
00644462    .  8D95 00FFFFFF   lea edx,dword ptr ss:[ebp-100]
00644468    .  8985 08FFFFFF   mov dword ptr ss:[ebp-F8],eax
0064446E    .  8D85 F0FEFFFF   lea eax,dword ptr ss:[ebp-110]
00644474    .  52              push edx
00644475    .  50              push eax
00644476    .  8975 88         mov dword ptr ss:[ebp-78],esi
00644479    .  C785 00FFFFFF 0>mov dword ptr ss:[ebp-100],8
00644483    .  FF15 B0104000   call dword ptr ds:[<&MSVBVM60.#520>]          ;  MSVBVM60.rtcTrimVar
00644489    .  8D8D F0FEFFFF   lea ecx,dword ptr ss:[ebp-110]
0064448F    .  8D55 84         lea edx,dword ptr ss:[ebp-7C]
00644492    .  51              push ecx
00644493    .  52              push edx
00644494    .  FF15 58114000   call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>]  ;  得到机器码的字符串
0064449A    .  50              push eax
0064449B    .  FF15 18124000   call dword ptr ds:[<&MSVBVM60.#581>]          ;  将字符串转化为 10 进制放于 ST0
006444A1    .  DD9D 8CFEFFFF   fstp qword ptr ss:[ebp-174]
006444A7    .  DD85 9CFEFFFF   fld qword ptr ss:[ebp-164]
006444AD    .  6A 02           push 2
006444AF    .  FF15 D4114000   call dword ptr ds:[<&MSVBVM60.__vbaFpI4>]     ;  把在 ST0 中的 10 进制的机器码转化为 16 进制放于 EAX 中
006444B5    .  8BC8            mov ecx,eax
006444B7    .  B8 398EE338     mov eax,38E38E39
006444BC    .  F7E9            imul ecx                                  ;  16 进制的机器码与 38E338E39 相乘结果放于 EAX 与 EDX 中
006444BE    .  D1FA            sar edx,1                                 ;  EDX 中的进行 SAR 运算
006444C0    .  8BC2            mov eax,edx
006444C2    .  C1E8 1F         shr eax,1F                                ;  计算结果向右移
006444C5    .  03D0            add edx,eax                               ;  将右移的结果加上刚才 EDX 的值
006444C7    .  52              push edx
006444C8    .  FF15 18104000   call dword ptr ds:[<&MSVBVM60.__vbaStrI4>]    ;  把上面结果 EDX 的 16 进制转化为 10 进制
006444CE    .  8B1D E0114000   mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrMove>] ;  出现 10 进制的数
006444D4    .  8BD0            mov edx,eax
006444D6    .  8D4D 90         lea ecx,dword ptr ss:[ebp-70]
006444D9    .  FFD3            call ebx                                  ;  <&MSVBVM60.__vbaStrMove>
006444DB    .  50              push eax
006444DC    .  FF15 D0114000   call dword ptr ds:[<&MSVBVM60.#616>]          ;  取前为做为注册码的前两位
006444E2    .  DD85 94FEFFFF   fld qword ptr ss:[ebp-16C]                  ;  取机器码
006444E8    .  8985 28FFFFFF   mov dword ptr ss:[ebp-D8],eax
006444EE    .  C785 20FFFFFF 0>mov dword ptr ss:[ebp-E0],8
006444F8    .  FF15 D4114000   call dword ptr ds:[<&MSVBVM60.__vbaFpI4>]     ;  MSVBVM60.__vbaFpI4
006444FE    .  8BC8            mov ecx,eax                         ;  又把机器码转化为 16 进制
00644500    .  B8 310CC330     mov eax,30C30C31
00644505    .  F7E9            imul ecx                                ;  与 30C30C31 相称
00644507    .  C1FA 02         sar edx,2                               ;  得到的结果放于 EAX 与 EDX 中，让 EDX 中的值进行 SAR2 运算
0064450A    .  8BCA            mov ecx,edx
0064450C    .  6A 09           push 9
0064450E    .  C1E9 1F         shr ecx,1F                              ;  取最高位
00644511    .  03D1            add edx,ecx                             ;  刚才 EDX 的最高位加上 EDX
00644513    .  8D85 30FFFFFF   lea eax,dword ptr ss:[ebp-D0]
00644519    .  81C2 B700B228   add edx,28B200B7                          ;  与 28B200B7 相加它的 10 进制很有用
0064451F    .  C785 40FFFFFF 0>mov dword ptr ss:[ebp-C0],3
00644529    .  0F80 23020000   jo yyjxc.00644752
0064452F    .  8995 48FFFFFF   mov dword ptr ss:[ebp-B8],edx            ;  把刚才的结果 EDX 储存到 EBP-B8 中
00644535    .  8D95 40FFFFFF   lea edx,dword ptr ss:[ebp-C0]
0064453B    .  52              push edx
0064453C    .  50              push eax
0064453D    .  FF15 EC114000   call dword ptr ds:[<&MSVBVM60.#619>]          ;  把上面的结果转化为 10 进制字符串的形式存与 EBP-C8
00644543    .  DD85 8CFEFFFF   fld qword ptr ss:[ebp-174]                  ;  取机器吗
00644549    .  DC0D 68124000   fmul qword ptr ds:[401268]                 ;  与 9 相称
0064454F    .  6A 04           push 4
00644551    .  83EC 08         sub esp,8
00644554    .  DFE0            fstsw ax
00644556    .  A8 0D           test al,0D
00644558    .  0F85 EF010000   jnz yyjxc.0064474D
0064455E    .  DD1C24          fstp qword ptr ss:[esp]
00644561    .  FF15 10114000   call dword ptr ds:[<&MSVBVM60.__vbaStrR8>]     ;  取实数部分作为字符串
00644567    .  8BD0            mov edx,eax
00644569    .  8D4D 80         lea ecx,dword ptr ss:[ebp-80]
0064456C    .  FFD3            call ebx
0064456E    .  50              push eax
0064456F    .  FF15 D0114000   call dword ptr ds:[<&MSVBVM60.#616>]          ;  取左边前 4 位字符串
00644575    .  8D8D 20FFFFFF   lea ecx,dword ptr ss:[ebp-E0]
0064457B    .  8985 E8FEFFFF   mov dword ptr ss:[ebp-118],eax
00644581    .  8D95 30FFFFFF   lea edx,dword ptr ss:[ebp-D0]
00644587    .  51              push ecx
00644588    .  8D85 10FFFFFF   lea eax,dword ptr ss:[ebp-F0]
0064458E    .  52              push edx
0064458F    .  50              push eax
00644590    .  C785 E0FEFFFF 0>mov dword ptr ss:[ebp-120],8
0064459A    .  FF15 5C114000   call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]    ;  把第一次前 2 位与第二次组合成字符串
006445A0    .  8D8D E0FEFFFF   lea ecx,dword ptr ss:[ebp-120]
006445A6    .  50              push eax
006445A7    .  51              push ecx
006445A8    .  8D95 D0FEFFFF   lea edx,dword ptr ss:[ebp-130]
006445AE    .  52              push edx
006445AF    .  FF15 5C114000   call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]    ;  把上次的结果与第三次的前 4 位组合
006445B5    .  50              push eax
006445B6    .  FF15 28104000   call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>] ;  拷贝
006445BC    .  8BD0            mov edx,eax                        ;  出现正确的！（三次组合）
```

## 总结算法

一共分为 3 次计算，取第一次计算结果的前 2 位，取第二次计算的全部，取第三次计算的前 4 位，三次组合即为注册码：

- 第一次：将机器码转化为 16 进制与 `38E338E39` 相乘，取后 8 位进行 `SAR1` 运算，运算的结果加上自己的 2 进制的最高位，然后把此次计算得到的结果转化为 10 进制，取前 2 位。
- 第二次：将机器码转化为 16 进制与 `30C30C31` 相乘，取后 8 位进行 `SAR2` 运算，运算的结果加上自己的 2 进制的最高位，得到的结果与 `28B200B7` 相加，将结果转化为 10 进制，取其全部
- 第三次：机器码本身与 9 相乘，直接取前 4 位

`4dd466d0e1e0cc123afbe504da2462ca`
