
# [原创]xx外挂浮点注册算法分析

## 文章信息

【软件名称】 xx外挂浮点注册算法分析

【文章作者】 仙剑太郎

【网站地址】 中国X黑客小组 72cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6f1y4F1h3p5S2S2j5$3E0W2M7W2)9J5k6h3y4G2L8b7`.

【破解工具】 Ollydbg

【软件语言】 Microsoft Visual C++

【破解声明】 我是一只小菜鸟，偶的一点心得，愿与大家分享：）

-------------------------------------------------------------

## 破解过程

一个冒险岛的小外挂,无壳,VC写的,偶们菜鸟拿来做练习最好不过了~

软件使用了浮点运算,正好复习一下浮点指令:)好,下面开工了..

我这里的机器码是`DFMPHHCCF4`,随便输入注册码,根据出错提示很容易找到关键地方,向上翻,会找到算法开始的地方,如下

```
004028B9   .  33C0  xor eax,eax
004028BB   .  8B35 >mov esi,dword ptr ds:[41F54C]         ;  038版本?0041F560
004028C1   .  89442>mov dword ptr ss:[esp+2D],eax
004028C5   .  33D2  xor edx,edx
004028C7   .  89442>mov dword ptr ss:[esp+31],eax
004028CB   .  89542>mov dword ptr ss:[esp+44],edx
004028CF   .  89442>mov dword ptr ss:[esp+35],eax
004028D3   .  88542>mov byte ptr ss:[esp+2C],dl
004028D7   .  66:89>mov word ptr ss:[esp+39],ax
004028DC   .  89742>mov dword ptr ss:[esp+10],esi
004028E0   .  88442>mov byte ptr ss:[esp+3B],al
004028E4   .  8B7C2>mov edi,dword ptr ss:[esp+18]         ;  机器码指针放进edi
004028E8   .  83C9 >or ecx,FFFFFFFF
004028EB   .  33ED  xor ebp,ebp
004028ED   .  C6442>mov byte ptr ss:[esp+44],1
004028F2   .  F2:AE repne scas byte ptr es:[edi]          ;  计算机器码长度
004028F4   .  F7D1  not ecx
004028F6   .  49    dec ecx                               ;  长度为0xA
004028F7   .  89542>mov dword ptr ss:[esp+14],edx
004028FB   .  8BF9  mov edi,ecx                           ;  保存机器码长度
004028FD   .  0F84 >je 038版本?00402A9E
00402903   .  BE 01>mov esi,1
00402908   .  3BFE  cmp edi,esi                           ;  机器码长度是否小于1?
0040290A   .  89742>mov dword ptr ss:[esp+1C],esi
0040290E   .  0F8C >jl 038版本?004029C3                   ;  小于则over
00402914   .  DD05 >fld qword ptr ds:[41A768]             ;  将0.0放入st0
0040291A   >  8B4C2>mov ecx,dword ptr ss:[esp+18]         ;  循环的开始,机器码指针保存到ecx中
0040291E   .  DB442>fild dword ptr ss:[esp+1C]            ;  第N位放到st0
00402922   .  8A5C3>mov bl,byte ptr ds:[ecx+esi-1]        ;  机器码逐字符ASCII放到bl
00402926   .  0FBED>movsx edx,bl                          ;  保存到edx中
00402929   .  DD5C2>fstp qword ptr ss:[esp+24]            ;  取st0值到esp+0x24中,出栈
0040292D   .  89542>mov dword ptr ss:[esp+1C],edx         ;  将第N个ASCII放到esp+0x1c
00402931   .  DB442>fild dword ptr ss:[esp+1C]            ;  第N个ASCII整数值到st0中
00402935   .  0FBEC>movsx eax,bl                          ;  第N个ASCII数值保存到eax
00402938   .  D9FA  fsqrt                                 ;  取st0第N个ASCII平方根
0040293A   .  0FAFC>imul eax,esi                          ;  第N个ASCII*N
0040293D   .  DC4C2>fmul qword ptr ss:[esp+24]            ;  st0*N=st0
00402941   .  DC05 >fadd qword ptr ds:[41A760]            ;  平方根的值+1
00402947   .  0FAFC>imul eax,esi                          ;  已经*N的ASCII再*N
0040294A   .  89442>mov dword ptr ss:[esp+1C],eax         ;  *完的结果放到esp+0x1c里
0040294E   .  DB442>fild dword ptr ss:[esp+1C]            ;  *完的结果到st0中,以十进制表示
00402952   .  DEC9  fmulp st(1),st                        ;  *完的结果再*st1,即st0=st0*st1
00402954   .  D8C1  fadd st,st(1)                         ;  st0+st1
00402956   .  E8 75>call 038版本?00403CD0                 ;  将st0取整,st0出栈
0040295B   .  99    cdq
0040295C   .  DDD8  fstp st                               ;  出栈一次
0040295E   .  B9 A0>mov ecx,186A0                         ;  常数100000
00402963   .  F7F9  idiv ecx                              ;  取整后的数余100000
00402965   .  89542>mov dword ptr ss:[esp+14],edx         ;  余数为密钥1,Key1★
00402969   .  0FBED>movsx edx,bl                          ;  第N位ASCII放到ebx
0040296C   .  89542>mov dword ptr ss:[esp+1C],edx         ;  第N位ASCII放到esp+1c
00402970   .  DB442>fild dword ptr ss:[esp+1C]            ;  第N位ASCII放到存进st0,十进制表示
00402974   .  DD05 >fld qword ptr ds:[41A758]             ;  将2.00放到st0,作为底数
0040297A   .  E8 31>call 038版本?00403AB0                 ;  求对数LOG等运算的CALL,跟进
```

================================

`0040297A`此CALL用来求对数等其它运算,F7跟进看看,来到下面:

================================

```
00403AB0  /$  83EC >sub esp,10
00403AB3  |.  D9C9  fxch st(1)                            ;  st0和st1互换
00403AB5  |.  DD1C2>fstp qword ptr ss:[esp]               ;  st0出栈
00403AB8  |.  DD542>fst qword ptr ss:[esp+8]              ;  将浮点2.00以实数形式放到esp+8,值为0
00403ABC  |.  8B442>mov eax,dword ptr ss:[esp+C]          ;  将40000000放eax
00403AC0  |.  E8 0D>call 038版本?00403AD2                 ;  求LOG的CALL,跟进
00403AC5  |.  83C4 >add esp,10
00403AC8  \.  C3    retn
```

================================

再跟进一下`00403AC0`,来到下面:

================================

```
00403AD2   $  8BC8  mov ecx,eax
00403AD4   .  50    push eax
00403AD5   .  9B    wait
00403AD6   .  D93C2>fstcw word ptr ss:[esp]
00403AD9   .  66:81>cmp word ptr ss:[esp],27F
00403ADF   .  74 05 je short 038版本?00403AE6
00403AE1   .  E8 4F>call 038版本?00406935
00403AE6   >  81E1 >and ecx,7FF00000                      ;  40000000 and 7FF00000
00403AEC   .  8D542>lea edx,dword ptr ss:[esp+8]
00403AF0   .  81F9 >cmp ecx,7FF00000
00403AF6   .  0F84 >je 038版本?00403B99
00403AFC   .  E8 64>call 038版本?00406965
00403B01   .  0F84 >je 038版本?00403B95
00403B07   .  A9 00>test eax,7FF00000
00403B0C   .  0F84 >je 038版本?00403C08
00403B12   >  8A4C2>mov cl,byte ptr ss:[esp+F]            ;  40?
00403B16   .  80E1 >and cl,80                             ;  40 and 80=0
00403B19   .  0F85 >jnz 038版本?00403C80
00403B1F   >  D9F1  fyl2x                                 ;  st1 * log2(st0)
00403B21   .  E8 FA>call 038版本?00406920                 ;  四舍五入等运算,跟进
```

================================

`00403B21`的CALL也是关键的计算,F7跟进,来到下面:

================================

```
00406920  /$  D9C0  fld st
00406922  |.  D9FC  frndint                               ;  四舍五入
00406924  |.  DCE1  fsubr st(1),st                        ;  st1-st0,负值
00406926  |.  D9C9  fxch st(1)                            ;  结果交换
00406928  |.  D9E0  fchs                                  ;  求负数,前面加-
0040692A  |.  D9F0  f2xm1                                 ;  st(0) <- (2 ^ st(0)) - 1
0040692C  |.  D9E8  fld1                                  ;  1放入st0
0040692E  |.  DEC1  faddp st(1),st                        ;  相加然后出栈
00406930  |.  D9FD  fscale                                ;  2 ^ st1*st(0)
00406932  |.  DDD9  fstp st(1)                            ;  出栈
00406934  \.  C3    retn
```

上面CALL的运算流程一目了然,我们retn回去,中途的很多东西都是编译器生成的容错代码,与注册码算法无关,所以中间有些东西不用理会

继续返回到这里:

================================

```
0040297F   .  DC4C2>fmul qword ptr ss:[esp+24]            ;  求对数的结果*N
00402983   .  E8 48>call 038版本?00403CD0                 ;  将st0取整
00402988   .  DB442>fild dword ptr ss:[esp+14]            ;  将余数放st0
0040298C   .  8BCE  mov ecx,esi                           ;  esi=N
0040298E   .  0FAFC>imul ecx,ebp                          ;  ebp为新变量*N
00402991   .  D9C0  fld st                                ;  入栈st1
00402993   .  D9FA  fsqrt                                 ;  *完的结果开平方
00402995   .  03C1  add eax,ecx                           ;  开平方的结果与*的结果相加
00402997   .  B9 A0>mov ecx,186A0                         ;  常数10000
0040299C   .  99    cdq
0040299D   .  F7F9  idiv ecx                              ;  相加后的数余100000
0040299F   .  8BEA  mov ebp,edx                           ;  保存结果为密钥2,Key2★
004029A1   .  E8 2A>call 038版本?00403CD0                 ;  取整
004029A6   .  03C5  add eax,ebp                           ;  取整结果与相加结果再加
004029A8   .  B9 A0>mov ecx,186A0                         ;  常数10000
004029AD   .  99    cdq
004029AE   .  F7F9  idiv ecx                              ;  余数为密钥3,Key3★
004029B0   .  46    inc esi                               ;  esi+1
004029B1   .  3BF7  cmp esi,edi                           ;  完了没?
004029B3   .  89742>mov dword ptr ss:[esp+1C],esi         ;  esi放到esp+1c保存
004029B7   .^ 0F8E >jle 038版本?0040291A                  ;  未完跳回去,完了则向下
```

这里根据机器码算出来的三个注册密钥已经生成完毕,我的机器码对应的三个密钥是:

`Key1=0x11F73` , `Key2=0x18088` , `Key3=18197`
