
```
10001298  |.  C1E9 02                 |shr     ecx, 2                            ;   
1000129B  |.  F3:A5                   |rep     movs dword ptr es:[edi], dword ptr [esi]  ;   
1000129D  |.  8BCA                    |mov     ecx, edx                          ;   
1000129F  |.  50                      |push    eax                               ;  解密 KEY  
100012A0  |.  83E1 03                 |and     ecx, 3                            ;   
100012A3  |.  F3:A4                   |rep     movs byte ptr es:[edi], byte ptr [esi]    ;   
100012A5  |.  8D4C24 14               |lea     ecx, [esp+14]  
100012A9  |.  51                      |push    ecx                               ;  注册码  
100012AA     E8 51FFFFFF              call    10001200                           ;  KEY 和注册码进去，计算  
100012AF  |.  8B4424 24               |mov     eax, [esp+24]                     ;  注册码地址  
100012B3  |.  8BCB                    |mov     ecx, ebx                          ;   
100012B5  |.  8BD1                    |mov     edx, ecx  
100012B7  |.  8D7424 18               |lea     esi, [esp+18]                    ;  刚计算完的中间结果  
100012BB  |.  8BF8                    |mov     edi, eax                          ;  注册码  
100012BD  |.  2BEB                    |sub     ebp, ebx  
100012BF  |.  C1E9 02                 |shr     ecx, 2  
100012C2  |.  F3:A5                   |rep     movs dword ptr es:[edi], dword ptr [esi]  
100012C4  |.  8BCA                    |mov     ecx, edx  
100012C6  |.  83C4 08                 |add     esp, 8  
100012C9  |.  83E1 03                 |and     ecx, 3  
100012CC  |.  03C3                    |add     eax, ebx  
100012CE  |.  85ED                    |test    ebp, ebp  
100012D0  |.  F3:A4                   |rep     movs byte ptr es:[edi], byte ptr [esi]  
100012D2  |.  894424 1C               |mov     [esp+1C], eax  
100012D6  |.^ 7F 9B                  \\jg     short 10001273  
100012D8  |.  5F                      |pop     edi  
100012D9  |.  5E                      |pop     esi  
100012DA  |.  5D                      |pop     ebp  
100012DB  |.  33C0                    |xor     eax, eax  
100012DD  |.  5B                      |pop     ebx  
100012DE  |.  83C4 08                 |add     esp, 8  
100012E1  \\.  C3                     |retn  
```

这里就是把进来的 24 字符分成 3 个 8 BYTE，然后同解密 KEY 一起调用 `10001200`。所以可以大概判定原始注册码应该是去掉 `'-'` 后有 48 个字符。

### 继续进到 `10001200`

```
10001200  /$  53                      |push    ebx  
10001201  |.  56                      |push    esi  
10001202  |.  8B7424 0C              |mov     esi, [esp+C]                      ;  值  
10001206  |.  57                      |push    edi  
10001207  |.  8B7C24 14              |mov     edi, [esp+14]                     ;  KEY  
1000120B  |.  6A 02                   |push    2  
1000120D  |.  57                      |push    edi                               ;  KEY  
1000120E  |.  56                      |push    esi                               ;  值地址，函数返回也在这里  
1000120F  |.  E8 3CFEFFFF             |call    10001050                          ;  计算，取 8 个值同 KEY 异或  
10001214  |.  6A 02                   |push    2                                 ;  进去个 2  
10001216  |.  68 20330010             |push    10003320                          ;  这是一个索引表的地址  
1000121B  |.  56                      |push    esi                               ;  再算  
1000121C  |.  E8 7FFEFFFF             |call    100010A0                          ;  查索引表，返回值还在 ESI 中  
10001221  |.  83C4 18                 |add     esp, 18  
10001224  |.  BB 05000000             |mov     ebx, 5                            ;  循环 5 次  
10001229  |>  6A 02                   |/push    2                                 ;  2 进去  
1000122B  |.  57                      |push    edi                               ;  KEY  
1000122C  |.  56                      |push    esi                               ;  计算中间结果  
1000122D  |.  E8 1EFEFFFF             |call    10001050                          ;  异或啊  
10001232  |.  6A 02                   |push    2                                 ;  2  
10001234  |.  56                      |push    esi                               ;  值进去，再计算  
10001235  |.  E8 A6FEFFFF             |call    100010E0                          ;  来回查表，完事，值还在 ESI 中  
1000123A  |.  6A 02                   |push    2  
1000123C  |.  68 20330010             |push    10003320  
10001241  |.  56                      |push    esi  
10001242  |.  E8 59FEFFFF             |call    100010A0                          ;  算出的东西再查表  
10001247  |.  83C4 20                 |add     esp, 20  
1000124A  |.  4B                      |dec     ebx  
1000124B  |.^ 75 DC                   |\jnz     short 10001229  
1000124D  |.  6A 02                   |push    2  
1000124F  |.  57                      |push    edi  
10001250  |.  56                      |push    esi  
10001251  |.  E8 FAFDFFFF             |call    10001050  
10001256  |.  83C4 0C                 |add     esp, 0C  
10001259  |.  33C0                    |xor     eax, eax  
1000125B  |.  5F                      |pop     edi  
1000125C  |.  5E                      |pop     esi  
1000125D  |.  5B                      |pop     ebx  
1000125E  \\.  C3                     |retn  
```

到这里，熟悉算法的人应该能大概看出是什么东西了。不过我跟的时候不熟。一直把里面的都跟完了。并把这个过程用 C 还原了。

我知道这个过程肯定是可逆的，但是我逆不出来。觉得应该不是作者自己开发的。所以到网上找对称算法。看到 AES 算法的介绍后，里面有 `b`, `d`, `9`, `e` 几个参数跟 `100010E0` 里的几个对应上了才知道。

### `100010E0` 处：

```
100010E0  /$  83EC 0C                 |sub     esp, 0C  
100010E3  |.  8B5424 14              |mov     edx, [esp+14]                     ;  参数 1  
100010E7  |.  53                      |push    ebx  
100010E8  |.  55                      |push    ebp  
100010E9  |.  8B6C24 18              |mov     ebp, [esp+18]                    ;  值地址  
100010ED  |.  56                      |push    esi  
100010EE  |.  57                      |push    edi  
100010EF  |.  33FF                    |xor     edi, edi  
100010F1  |.  81E2 FF000000           |and     edx, 0FF  
100010F7  |.  895424 10              |mov     [esp+10], edx                   ;  2  
100010FB  |.  0F8E B9000000          |jle     100011BA  
10001101  |>  8D443C 14               |/lea     eax, [esp+edi+14]              ;  地址定位  
10001105  |.  8D4C24 14              ||lea     ecx, [esp+14]  
10001109  |.  894424 24              ||mov     [esp+24], eax                  ;  地址定位  
1000110D  |.  8BC5                    ||mov     eax, ebp                      ;  值  
1000110F  |.  2BC1                    ||sub     eax, ecx                      ;  定位用  
10001111  |.  BE 03000000            ||mov     esi, 3                        ;   
10001116  |.  894424 20              ||mov     [esp+20], eax                 ;  存起来  
1000111A  |>  8D56 FE                ||/lea     edx, [esi-2]  
1000111D  |.  81E2 03000080          |||and     edx, 80000003                ;  测试是否是负数？  
10001123  |.  79 05                   ||jns      short 1000112A              ;  没有走  
10001125  |.  4A                      |||dec      edx  
10001126  |.  83CA FC                |||or       edx, FFFFFFC  
10001129  |.  42                      |||inc      edx  
1000112A  |>  8D0457                 |||lea     eax, [edi+edx*2]            ;   
1000112D  |.  8A0C28                 |||mov     cl, [eax+ebp]               ;  取得一个索引的值  
10001130  |.  51                      |||push    ecx                        ;   
10001131  |.  6A 0B                  |||push    0B                         ;  B  
```
