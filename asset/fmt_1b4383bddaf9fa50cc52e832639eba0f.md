
```
0040CE40   mov dword ptr ss:[esp+8],eax             调用后将结果存入临时变量
0040CE44   cmp byte ptr ds:[42DC98],2D               查上面调用后第一个不在码表中的字符是否是“-”，
0040CE4B   je short PopChar.0040CE51                 否则不跳，码错
0040CE4D   xor eax,eax                                分析到此可以初步认定注册码前面应是“ABCD-”的形式
0040CE4F   pop ecx                                    “-”前的必须是大写字母，几个没有限制
0040CE50   retn

0040CE51   push ebx
0040CE52   push ebp
0040CE53   push esi
0040CE54   push edi
0040CE55   call PopChar.0040CD20                      又调用这两个函数，进行循环
0040CE5A   call PopChar.0040CD90                      结果存入临时变量，可以认定注册码是前面是“ABCD-ABCD”
0040CE5F   mov dword ptr ss:[esp+10],eax              的形式
```

下面连续调用 `40CDE0` 五次，`40CDE0` 的代码插在这里，并加以说明。

```
0040CDE0   mov al,byte ptr ds:[42DC98]
0040CDE5   xor edx,edx                                edx 是累加器，初始化为 0x0
0040CDE7   cmp al,2D                                  查字符是否是“-”，否则码错，返回
0040CDE9   je short PopChar.0040CDEE
0040CDEB   xor eax,eax
0040CDED   retn
0040CDEE   call PopChar.0040CD20                      调用 40CD20 取一个字符
0040CDF3   mov al,byte ptr ds:[42DC98]
0040CDF8   cmp al,30                                  检查字符的 ASCII 值是否小于 0x30
0040CDFA   jl short PopChar.0040CE1C
0040CDFC   lea esp,dword ptr ss:[esp]
0040CE00   cmp al,39                                  检查字符的 ASCII 值是否大于 0x39
0040CE02   jg short PopChar.0040CE1C
0040CE04   lea ecx,dword ptr ds:[edx+edx*4]
0040CE07   movsx edx,al
0040CE0A   lea edx,dword ptr ds:[edx+ecx*2-30]
0040CE0E   call PopChar.0040CD20
0040CE13   mov al,byte ptr ds:[42DC98]
0040CE18   cmp al,30                                  检查字符的 ASCII 值是否大于 0x39
0040CE1A   jge short PopChar.0040CE00
0040CE1C   mov eax,edx
0040CE1E   retn
```

`40CDE0` 这个函数负责取字符，查是“-”后开始循环取字符，检查字符是否是数字字符，转换为数字进行运算和累加，当字符不是数字时，循环结束。

循环计算实际上就是将连续的数字字符（如“-1234-”）转换为数字 1234。

```
0040CE63   call PopChar.0040CDE0                      第一次调用 40CDE0，检查是否“-”开始，
0040CE68   mov ebp,eax                                 并取连续的数字字符变为数字，存入临时变量
0040CE6A   call PopChar.0040CDE0                      第二次调用 40CDE0
0040CE6F   mov esi,eax
0040CE71   call PopChar.0040CDE0                      第三次调用 40CDE0
0040CE76   mov ebx,dword ptr ss:[esp+1C]
0040CE7A   mov dword ptr ds:[ebx],eax
0040CE7C   call PopChar.0040CDE0                      第四次调用 40CDE0
0040CE81   mov edi,eax
0040CE83   call PopChar.0040CDE0                      第五次调用 40CDE0
0040CE88   mov ecx,dword ptr ss:[esp+20]
0040CE8C   mov dword ptr ds:[ecx],eax
```

至此，可以认定码为“ABCD-EFGH-1234-5678-9012-3456-7890”的形式，为下面叙述和描述算法方便，以下我将注册码中“-”隔开的小码转换后的结果按顺序称为 `sn1`, `sn2`, `sn3`, `sn4`, `sn5`, `sn6`, `sn7`，同时定义四个代表中间结果的变量 `t1`，`t2`，`t3`，`t4`，`t5`。

```
0040CE8E   mov ecx,dword ptr ds:[ebx]
0040CE90   test ecx,ecx                               sn5 是否为空
0040CE92   je PopChar.0040CFC5
0040CE98   test esi,esi                               sn4 是否为空
0040CE9A   je PopChar.0040CFC5
0040CEA0   test edi,edi                               sn6 是否为空
0040CEA2   je PopChar.0040CFC5
0040CEA8   test eax,eax                               sn7 是否为空
0040CEAA   je PopChar.0040CFC5
0040CEB0   mov al,byte ptr ds:[42DC98]               查整个注册码字串后面是否还有字符
0040CEB5   test al,al
0040CEB7   jnz PopChar.0040CFC5
0040CEBD   mov eax,edi
0040CEBF   cdq
0040CEC0   idiv dword ptr ds:[42DCA4]                 sn6 除以 0x40, [42DCA4]在开始初始化是被写入常数 0x40
0040CEC6   mov eax,dword ptr ds:[42DCA0]              [42DCA0]初始化时写入 42BA20，是一个码表
0040CECB   sub ecx,dword ptr ds:[eax+edx*4]           sn5 减去码表中取的数 ：sn5－码表[(sn6 mod 0x40)*0x4]
0040CECE   mov eax,dword ptr ds:[42DCA8]              变量，与时间锁有关，可以认为总为 0x0
0040CED3   mov edx,dword ptr ds:[42BB20]              [42BB20]在开始初始化时被写入常数 0x5B31
0040CED9   sub ecx,eax                                减去 0x0
0040CEDB   sub ecx,edx                                减去 0x5B31，
0040CEDD   mov eax,ebp
0040CEDF   mov dword ptr ds:[ebx],ecx                 临时结果存入变量，我用 t1 表示这个中间结果
0040CEE1   cdq
0040CEE2   mov ebx,2710
0040CEE7   idiv ebx                                   eax 是 sn3，就是 sn3 除以 0x2710
0040CEE9   test ecx,ecx
0040CEEB   mov ebp,edx                                余数送 ebp，我用 t3 表示这个中间结果
0040CEED   mov ebx,eax                                商送 ebx，我用 t2 表示这个中间结果
0040CEEF   jle PopChar.0040CFC5                       检测 ecx 不能为 0，也就是 t1 不能为 0
0040CEF5   cmp ebp,7D2                                要求余数>=0x7D2
0040CEFB   jl PopChar.0040CFC5
0040CF01   cmp ebp,802                                要求余数<=0x802
0040CF07   jg PopChar.0040CFC5
0040CF0D   test ebx,ebx
0040CF0F   jle PopChar.0040CFC5                       要求商<0xD
0040CF15   cmp ebx,0D
0040CF18   jge PopChar.0040CFC5                       以上几个比较 0x7D2 是 2002，0x802 是 2050，0xD 是 13
0040CF1E   mov edx,dword ptr ss:[esp+20]              可以推测认定，sn3 中的信息为年和月
0040CF22   mov eax,dword ptr ds:[edx]                 取 sn7 送 eax，并检测不能为 0
0040CF24   test eax,eax
0040CF26   jle PopChar.0040CFC5
0040CF2C   cmp eax,esi                                 si 中是 sn4，这里比较，要求 sn7<=sn4
0040CF2E   jg PopChar.0040CFC5
0040CF34   mov edx,dword ptr ss:[esp+18]              取 sn1 送 eax
0040CF38   lea eax,dword ptr ds:[ecx+ecx*2]            这句和下句加起来等同于 eax=eax*0xC+t2（这时 ebx 中是
0040CF3B   lea eax,dword ptr ds:[ebx+eax*4]            上面 40CEE7 除运算的商）
0040CF3E   imul eax,eax,7                              上面计算的结果乘以 0x7
0040CF41   add eax,esi                                 上面计算的结果加上 sn4
0040CF43   lea edx,dword ptr ds:[edx+eax*2]            这句和下句加起来等同于 eax=eax*2+sn3（这时 edx 中是
0040CF46   add eax,edx                                 sn3）
0040CF48   mov edx,dword ptr ss:[esp+10]              sn2 送 edx
0040CF4C   lea eax,dword ptr ss:[ebp+eax*2-7A5]        eax=t3+eax*2-0x7A5,（ebp 中是上面 40CEE7 除运算的余数）
0040CF53   lea edx,dword ptr ds:[edx+eax*4]            这句和下句加起来等同于 eax=eax*5+sn2
0040CF56   add eax,edx
0040CF58   cdq
0040CF59   mov esi,0D59D3
0040CF5E   idiv esi                                   上面计算的结果除以 0xD59D3
0040CF60   mov esi,edx                                余数送 esi
0040CF62   add esi,2D187                              余数加上 0x2d187
0040CF68   mov eax,esi                                结果送 eax，这个中间结果我用 t4 表示
0040CF6A   cdq
0040CF6B   idiv dword ptr ds:[42DCA4]                 t4 除以 0x40, [42DCA4]在开始初始化是被写入常数 0x40
0040CF71   mov eax,dword ptr ds:[42DCA0]              码表开始，[42DCA0]初始化时写入的 42BA20
0040CF76   mov edx,dword ptr ds:[eax+edx*4]            用 t4 除以 0x40 的余数在码表中取数: 码表[(t4 mod 0x40)*0x4]
0040CF79   add edx,dword ptr ds:[42DCA8]              加上变量，与时间锁有关，可以认为总为 0x0
0040CF7F   add edx,esi                                加上 t4
0040CF81   cmp edx,edi                                比较结果（我用 t5 表示）是否等于 sn6，此时 edi 中是 sn6
0040CF83   jnz short PopChar.0040CFC5                如果不等，注册码错误，跳走
0040CF85   push ecx                                   t1 入栈传给函数
0040CF86   call dword ptr ds:[42BB2C]                ;  PopChar.0040C9F0
0040CF8C   mov esi,eax
0040CF8E   mov eax,dword ptr ss:[esp+1C]
0040CF92   push eax                                   sn1 入栈传给函数
0040CF93   call dword ptr ds:[42BB30]                ;  PopChar.0040CA40
0040CF99   add esp,8
0040CF9C   cmp esi,eax
0040CF9E   jge short PopChar.0040CFC5
```

`40CF86` 处调用的 `40C9F0` 和 `40CF93` 处调用的 `40CA40` 的很简单，在这里说明。
