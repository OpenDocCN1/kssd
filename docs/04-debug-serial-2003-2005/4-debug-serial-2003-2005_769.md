
# PCMark04 的简单算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-16.htm`](https://www.pediy.com/kssd/pediy07/pediy7-16.htm)

`PCMark04` 的算法，很简单，高手略过吧。

`PCMark04` 用来给电脑打分的，未注册有好多功能限制，本来打算爆破的，没想到它的算法如此简单，就算一下吧；

`VC7.0` + 无壳，直接分析了；

用 `MessageBoxA` 直接就断下了

```
00413DC0   /$  56               push esi  
00413DC1   |.  8B7424 08        mov esi,dword ptr ss:[esp+8]  
00413DC5   |.  57               push edi  
00413DC6   |.  8BF9             mov edi,ecx  
00413DC8   |.  6A 05            push 5                         ;  提取第 5 位值  
00413DCA   |.  8BCE             mov ecx,esi  
00413DCC   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413DD2   |.  8038 2D          cmp byte ptr ds:[eax],2D       ;  检测第 5 位是否“-”  
00413DD5   |.  75 0D            jnz short PCMark04.00413DE4      ;  不是的话就检测第 4 位  
00413DD7   |.  56               push esi                       ; /Arg1  
00413DD8   |.  8BCF             mov ecx,edi                     ; |  
00413DDA   |.  E8 D1F9FFFF      call PCMark04.004137B0          ; \PCMark04.004137B0  
00413DDF   |.  5F               pop edi  
00413DE0   |.  5E               pop esi  
00413DE1   |.  C2 0400          retn 4  
00413DE4   |>  6A 04            push 4                         ;  提取第 4 位值  
00413DE6   |.  8BCE             mov ecx,esi  
00413DE8   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413DEE   |.  8038 2D          cmp byte ptr ds:[eax],2D       ;  检测第 4 位是否“-”  
00413DF1   |.  75 0D            jnz short PCMark04.00413E00      ;  也不是的话就 OVER  
00413DF3   |.  56               push esi                       ; /Arg1  
00413DF4   |.  8BCF             mov ecx,edi                     ; |  
00413DF6   |.  E8 15FDFFFF      call PCMark04.00413B10          ; \PCMark04.00413B10  
00413DFB   |.  5F               pop edi  
00413DFC   |.  5E               pop esi  
00413DFD   |.  C2 0400          retn 4  
00413E00   |>  5F               pop edi  
00413E01   |.  32C0             xor al,al                      ;al＝0 啦  
00413E03   |.  5E               pop esi  
00413E04   \.  C2 0400          retn 4  
```

我追进 `00413DF6`：

```
00413B10   /$  6A FF            push -1  
00413B12   |.  68 21A04300      push PCMark04.0043A021          ;  SE handler installation  
00413B17   |.  64:A1 00000000   mov eax,dword ptr fs:[0]  
00413B1D   |.  50               push eax  
00413B1E   |.  64:8925 00000000 mov dword ptr fs:[0],esp  
00413B25   |.  83EC 3C          sub esp,3C  
00413B28   |.  A1 A02D4500      mov eax,dword ptr ds:[452DA0]  
00413B2D   |.  55               push ebp  
00413B2E   |.  57               push edi  
00413B2F   |.  8B7C24 54        mov edi,dword ptr ss:[esp+54]  
00413B33   |.  8BE9             mov ebp,ecx  
00413B35   |.  8BCF             mov ecx,edi  
00413B37   |.  894424 40        mov dword ptr ss:[esp+40],eax  
00413B3B   |.  FF15 8CE74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size  
00413B41   |.  83F8 18          cmp eax,18                       ;  长度＝0x18  
00413B44   |.  0F85 69020000    jnz PCMark04.00413DB3  
00413B4A   |.  6A 04            push 4                         ;  检测第 4 位是否“-”  
00413B4C   |.  8BCF             mov ecx,edi  
00413B4E   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413B54   |.  8038 2D          cmp byte ptr ds:[eax],2D       ;  同上  
00413B57   |.  0F85 56020000    jnz PCMark04.00413DB3  
00413B5D   |.  6A 09            push 9  
00413B5F   |.  8BCF             mov ecx,edi  
00413B61   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413B67   |.  8038 2D          cmp byte ptr ds:[eax],2D  
00413B6A   |.  0F85 43020000    jnz PCMark04.00413DB3  
00413B70   |.  6A 0E            push 0E  
00413B72   |.  8BCF             mov ecx,edi  
00413B74   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413B7A   |.  8038 2D          cmp byte ptr ds:[eax],2D  
00413B7D   |.  0F85 30020000    jnz PCMark04.00413DB3  
00413B83   |.  6A 13            push 13  
00413B85   |.  8BCF             mov ecx,edi  
00413B87   |.  FF15 80E74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413B8D   |.  8038 2D          cmp byte ptr ds:[eax],2D       ;公 6 次，也就是说注册码格式要像.....往下看！！！  
00413B90   |.  0F85 1D020000    jnz PCMark04.00413DB3  
00413B96   |.  68 0C134400      push PCMark04.0044130C          ;  ASCII "1XB4-ZVYV-72X4-QXJH-19LR"  
00413B9B   |.  57               push edi  
00413B9C   |.  FF15 74E74300    call dword ptr ds:[<&MSVCP71.??$?8DU?$cha>;  上面是黑名单  
00413BA2   |.  83C4 08          add esp,8  
00413BA5   |.  84C0             test al,al  
00413BA7   |.  0F85 06020000    jnz PCMark04.00413DB3  
00413BAD   |.  53               push ebx  
00413BAE   |.  56               push esi  
00413BAF   |.  8D4C24 2C        lea ecx,dword ptr ss:[esp+2C]  
00413BB3   |.  FF15 4CE74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::basic_string<char,std::char_traits<char>,std::allocator<char> >  
00413BB9   |.  8D4424 2C        lea eax,dword ptr ss:[esp+2C]  
00413BBD   |.  33DB             xor ebx,ebx  
00413BBF   |.  50               push eax  
00413BC0   |.  8BCD             mov ecx,ebp  
00413BC2   |.  895C24 58        mov dword ptr ss:[esp+58],ebx  
00413BC6   |.  E8 E5F5FFFF      call PCMark04.004131B0          ;  “B2D6FHJ1LN5ST9E3VWX8YZQ7U40KRAMCP”出来了  
00413BCB   |.  8BCF             mov ecx,edi                     ;   关键的字符串  
00413BCD   |.  FF15 8CE74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size  
00413BD3   |.  50               push eax  
00413BD4   |.  8D4C24 20        lea ecx,dword ptr ss:[esp+20]  
00413BD8   |.  E8 A3FBFFFF      call PCMark04.00413780  
00413BDD   |.  8BCF             mov ecx,edi  
00413BDF   |.  C64424 54 01     mov byte ptr ss:[esp+54],1  
00413BE4   |.  FF15 8CE74300    call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size  
00413BEA   |.  85C0             test eax,eax  
00413BEC   |.  8B7424 20        mov esi,dword ptr ss:[esp+20]  
00413BF0   |.  76 37            jbe short PCMark04.00413C29  
00413BF2   |>  53               /push ebx  
00413BF3   |.  8BCF             |mov ecx,edi  
00413BF5   |.  FF15 80E74300    |call dword ptr ds:[<&MSVCP71.std::basic_>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator[]  
00413BFB   |.  33C9             |xor ecx,ecx  
00413BFD   |.  8A08             |mov cl,byte ptr ds:[eax]       ;  注册码第一组第一位  
00413BFF   |.  6A 00            |push 0  
00413C01   |.  51               |push ecx  
00413C02   |.  8D4C24 34        |lea ecx,dword ptr ss:[esp+34]  
```



### 汇编代码分析

```
00413C06   |.  FF15 20E74300     |call dword ptr ds:[<&MSVCP71.std::basic_>;  注册码在上面字符串的位置
00413C0C   |.  8B15 88E74300     |mov edx,dword ptr ds:[<&MSVCP71.std::bas>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::npos
00413C12   |.  3B02              |cmp eax,dword ptr ds:[edx]                ;  不在的话就赋值 0xFFFFFFFF
00413C14   |.  75 03             |jnz short PCMark04.00413C19
00413C16   |.  8B45 00           |mov eax,dword ptr ss:[ebp]
00413C19   |>  89049E            |mov dword ptr ds:[esi+ebx*4],eax         ;  转移
00413C1C   |.  8BCF              |mov ecx,edi
00413C1E   |.  43                |inc ebx
00413C1F   |.  FF15 8CE74300     |call dword ptr ds:[<&MSVCP71.std::basic_>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413C25   |.  3BD8              |cmp ebx,eax                             ;  没完继续
00413C27   |.^ 72 C9             \jb short PCMark04.00413BF2
00413C29   |>  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413C2D   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  字符串长度
00413C33   |.  8B3E              mov edi,dword ptr ds:[esi]                ;  第一组第一位
00413C35   |.  8B56 08           mov edx,dword ptr ds:[esi+8]              ;  第一组第三位
00413C38   |.  8BC8              mov ecx,eax                              ;  字符串长度
00413C3A   |.  8B46 04           mov eax,dword ptr ds:[esi+4]             ;  第一组第二位
00413C3D   |.  03C7              add eax,edi
00413C3F   |.  03C2              add eax,edx
00413C41   |.  33D2              xor edx,edx
00413C43   |.  F7F1              div ecx                                   ;  相加后与字符串长度(0x21)取余
00413C45   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413C49   |.  8BFA              mov edi,edx                               ;  余数转移给 edi
00413C4B   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413C51   |.  8B6E 18           mov ebp,dword ptr ds:[esi+18]
00413C54   |.  8B5E 1C           mov ebx,dword ptr ds:[esi+1C]
00413C57   |.  8BC8              mov ecx,eax
00413C59   |.  8B46 14           mov eax,dword ptr ds:[esi+14]
00413C5C   |.  03C5              add eax,ebp
00413C5E   |.  03C3              add eax,ebx
00413C60   |.  03C7              add eax,edx                               ;  同上，只不过加了上次运算的余数
00413C62   |.  33D2              xor edx,edx
00413C64   |.  F7F1              div ecx
00413C66   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413C6A   |.  8BDA              mov ebx,edx
00413C6C   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413C72   |.  8B6E 2C           mov ebp,dword ptr ds:[esi+2C]
00413C75   |.  8BC8              mov ecx,eax
00413C77   |.  8B46 28           mov eax,dword ptr ds:[esi+28]
00413C7A   |.  03C5              add eax,ebp
00413C7C   |.  8B6E 30           mov ebp,dword ptr ds:[esi+30]
00413C7F   |.  03C3              add eax,ebx
00413C81   |.  03C5              add eax,ebp
00413C83   |.  33D2              xor edx,edx
00413C85   |.  F7F1              div ecx
00413C87   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413C8B   |.  8BEA              mov ebp,edx
00413C8D   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413C93   |.  8B56 3C           mov edx,dword ptr ds:[esi+3C]
00413C96   |.  8BC8              mov ecx,eax
00413C98   |.  8B46 44           mov eax,dword ptr ds:[esi+44]
00413C9B   |.  03C2              add eax,edx
00413C9D   |.  0346 40           add eax,dword ptr ds:[esi+40]
00413CA0   |.  03C5              add eax,ebp
00413CA2   |.  33D2              xor edx,edx
00413CA4   |.  F7F1              div ecx
00413CA6   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413CAA   |.  895424 5C         mov dword ptr ss:[esp+5C],edx
00413CAE   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413CB4   |.  8B56 10           mov edx,dword ptr ds:[esi+10]
00413CB7   |.  8BC8              mov ecx,eax
00413CB9   |.  8B46 34           mov eax,dword ptr ds:[esi+34]
00413CBC   |.  03C2              add eax,edx
00413CBE   |.  0346 20           add eax,dword ptr ds:[esi+20]
00413CC1   |.  33D2              xor edx,edx
00413CC3   |.  F7F1              div ecx
00413CC5   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413CC9   |.  895424 14         mov dword ptr ss:[esp+14],edx
00413CCD   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413CD3   |.  8B56 20           mov edx,dword ptr ds:[esi+20]
00413CD6   |.  8BC8              mov ecx,eax
00413CD8   |.  8B46 48           mov eax,dword ptr ds:[esi+48]
00413CDB   |.  03C2              add eax,edx
00413CDD   |.  0346 34           add eax,dword ptr ds:[esi+34]
00413CE0   |.  33D2              xor edx,edx
00413CE2   |.  F7F1              div ecx
00413CE4   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413CE8   |.  895424 18         mov dword ptr ss:[esp+18],edx
00413CEC   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413CF2   |.  8B56 54           mov edx,dword ptr ds:[esi+54]
00413CF5   |.  8BC8              mov ecx,eax
00413CF7   |.  8B46 14           mov eax,dword ptr ds:[esi+14]
00413CFA   |.  03C2              add eax,edx
00413CFC   |.  0306              add eax,dword ptr ds:[esi]
00413CFE   |.  33D2              xor edx,edx
00413D00   |.  F7F1              div ecx
00413D02   |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]
00413D06   |.  895424 10         mov dword ptr ss:[esp+10],edx
00413D0A   |.  FF15 8CE74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >::size
00413D10   |.  8B56 58           mov edx,dword ptr ds:[esi+58]
00413D13   |.  8BC8              mov ecx,eax
00413D15   |.  8B46 54           mov eax,dword ptr ds:[esi+54]
00413D18   |.  03C2              add eax,edx
00413D1A   |.  0346 50           add eax,dword ptr ds:[esi+50]
00413D1D   |.  33D2              xor edx,edx
00413D1F   |.  F7F1              div ecx
00413D21   |.  397E 0C           cmp dword ptr ds:[esi+C],edi              ;  第一组第四位＝(前三位之和%0x21)
00413D24   |.  75 6E             jnz short PCMark04.00413D94               ;  不相等跳下去 xor al,al
00413D26   |.  395E 20           cmp dword ptr ds:[esi+20],ebx             ;  第二组第四位＝(前三位之和%0x21)
00413D29   |.  75 69             jnz short PCMark04.00413D94
00413D2B   |.  396E 34           cmp dword ptr ds:[esi+34],ebp             ;  以下都基本相似
00413D2E   |.  75 64             jnz short PCMark04.00413D94
00413D30   |.  8B4424 5C         mov eax,dword ptr ss:[esp+5C]
00413D34   |.  3946 48           cmp dword ptr ds:[esi+48],eax
00413D37   |.  75 5B             jnz short PCMark04.00413D94
00413D39   |.  8B4C24 14         mov ecx,dword ptr ss:[esp+14]
00413D3D   |.  394E 50           cmp dword ptr ds:[esi+50],ecx
00413D40   |.  75 52             jnz short PCMark04.00413D94
00413D42   |.  8B4424 18         mov eax,dword ptr ss:[esp+18]
```



### 反汇编代码

```
00413D46   |.   3946 54           cmp dword ptr ds:[esi+54],eax  

00413D49   |.   75 49             jnz short PCMark04.00413D94  

00413D4B   |.   8B4C24 10         mov ecx,dword ptr ss:[esp+10]  

00413D4F   |.   394E 58           cmp dword ptr ds:[esi+58],ecx  

00413D52   |.   75 40             jnz short PCMark04.00413D94  

00413D54   |.   3956 5C           cmp dword ptr ds:[esi+5C],edx  

00413D57   |.   75 3B             jnz short PCMark04.00413D94  

00413D59   |.   8D4C24 1C         lea ecx,dword ptr ss:[esp+1C]  

00413D5D   |.   E8 4EAE0000       call PCMark04.0041EBB0  

00413D62   |.   8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]  

00413D66   |.   C74424 54 FFFFFF>mov dword ptr ss:[esp+54],-1  

00413D6E   |.   FF15 04E74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::~basic_string<char,std::char_traits<char>,std::allocator<char> >  

00413D74   |.   B0 01             mov al,1                    ;  全都对的话就 mov al,1  

00413D76   |>   5E                pop esi  

00413D77   |.   5B                pop ebx  

00413D78   |>   8B4C24 44         mov ecx,dword ptr ss:[esp+44]  

00413D7C   |.   5F                pop edi  

00413D7D   |.   64:890D 00000000  mov dword ptr fs:[0],ecx  

00413D84   |.   8B4C24 3C         mov ecx,dword ptr ss:[esp+3C]  

00413D88   |.   5D                pop ebp  

00413D89   |.   E8 503E0200       call PCMark04.00437BDE  

00413D8E   |.   83C4 48           add esp,48  

00413D91   |.   C2 0400           retn 4  

00413D94   |>   8D4C24 1C         lea ecx,dword ptr ss:[esp+1C]  

00413D98   |.   E8 13AE0000       call PCMark04.0041EBB0  

00413D9D   |.   8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]  

00413DA1   |.   C74424 54 FFFFFF>mov dword ptr ss:[esp+54],-1  

00413DA9   |.   FF15 04E74300     call dword ptr ds:[<&MSVCP71.std::basic_s>;  MSVCP71.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::~basic_string<char,std::char_traits<char>,std::allocator<char> >  

00413DAF   |.   32C0              xor al,al                   ;在这儿可以爆破  

00413DB1   |.^  EB C3             jmp short PCMark04.00413D76  

00413DB3   |>   32C0              xor al,al                   ;在这儿可以爆破  

00413DB5   \.^  EB C1             jmp short PCMark04.00413D78  
```

### 算法说明

注册码可以是 `5` 位一组或 `4` 位一组，两种算法基本相同，我只说 `4` 位一组的；

注册码长度 `24` 位，每四位间隔一个“-”；

设注册码为 `1234-5678-9012-3456-7890`

程序先把每一位注册码在字符串`"B2D6FHJ1LN5ST9E3VWX8YZQ7U40KRAMCP"`中的位置化为`int`类型；

例如`"1"`在字符串的位置就是是`0x7`，`"2"`在字符串的位置就是是`0x1`，而`"-"`没有出现在字符串里；就被赋值`"0xFFFFFFFF"`

这样就是：

```
0x7 0x1 0xf 0x19 0xFFFFFFFF 0xa 0x3 0x17 0x13 0xFFFFFFFF 0xd 0x1a 0x7 0x1 0xFFFFFFFF

0xf 0x19 0xa 0x3 0xFFFFFFFF 0x17 0x13 0xd 0x1
```

第一组第四位的值＝前三位之和%`0x21`;`0x21` 是字符串长度，这样可以使余数小于 `0x21`，才能算出答案；

(`0x7`+`0x1`+`0xf`)%`0x21`=`0x17` 即`"7"`

所以第一组可改为`"1237"`；

则第二组可改为`"5670"`；

第 1 组第 4 位的值＝前三位值之和与 `0x21` 取余；

第 2 组第 4 位的值＝(前三位值+上一组第 4 位值)与 `0x21` 取余；

第 3 组第 4 位的值＝(前三位值+上一组第 4 位值)与 `0x21` 取余；

第 4 组第 4 位的值＝(前三位值+上一组第 4 位值)与 `0x21` 取余；

第 5 组第 1 位的值=(`0xFFFFFFFF`+第 2 组第 4 位值+第 3 组第 4 位值)与 `0x21` 取余

第 5 组第 2 位的值=(第 2 组第 4 位值+第 3 组第 4 位值+第 4 组第 4 位值)与 `0x21` 取余

第 5 组第 3 位的值=(第 5 组第 2 位值+第 2 组第 1 位值+第 1 组第 1 位值)与 `0x21` 取余

第 5 组第 4 位的值=(第 5 组第 3 位值+第 5 组第 2 位值+第 5 组第 1 位值)与 `0x21` 取余

### C# 注册机代码

```csharp
//省略初始化代码

int[,] sn=new int[5,4]; //用个二维数组来存放注册码的值；
Random rand=new Random(); // Random 类，用来产生随机数；
private void button1_Click(object sender, System.EventArgs e)
{
    work(); //计算前 4 组；
    sn[4,0]=work1(-1,sn[1,3],sn[2,3],0);
    sn[4,1]=work1(sn[1,3],sn[2,3],sn[3,3],0);
    sn[4,2]=work1(sn[4,1],sn[1,0],sn[0,0],0);
    sn[4,3]=work1(sn[4,2],sn[4,1],sn[4,0],0);

    char[] codechar="B2D6FHJ1LN5ST9E3VWX8YZQ7U40KRAMCP".ToCharArray() ;
    string sntxt="";
    for (int i=0;i<5;i++)
    {
        for(int j=0;j<4;j++)
        {
            sntxt =sntxt+codechar[sn[i,j]];
            textBox1.Text =sntxt;
        }
        sntxt =sntxt+"-";
    }
}
public void work()
{
    sn[0,3]=0;
    int div=0;
    for(int j=0;j<4;j++)
    {
        for(int i =0;i<3;i++)
        {
            sn[j,i]=rand.Next (32);
        }
        sn[j,3]=work1(sn[j,0],sn[j,1],sn[j,2],div); //计算第四位的值；
        div = sn[j,3];
    }
}
public int work1(int sn0,int sn1,int sn2,int sn3)
{
    sn3=(sn0+sn1+sn2+sn3) % 0x21;
    return sn3;
}
```

有不对的地方还请老大们多多指教！！

`PCMark04` 注册机完成于 `04` 年的最后一天(虽然有点晚，但毕竟在 `04` 年^-^)

### 机器配置

* `XP`+`SP1`
* 赛扬 `2.4` 超到 `2.58`
* `256MB` `DDR333` 内存(`CL` 值调到 `2`)
* `80G` `2MB` 缓存硬盘
* `845GL` 主板
* 主板集成声卡，显卡
* 能开的都开了
* 显卡才 `400` 多分
* 总分 `2412` 分,郁闷！！！

`4dd466d0e1e0cc123afbe504da2462ca`
