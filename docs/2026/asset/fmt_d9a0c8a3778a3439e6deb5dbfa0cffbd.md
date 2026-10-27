
# 反汇编转储

## 初始段

```
0055F1F4   |.  8B8424 10010>|mov eax,dword ptr ss:[esp+110]

0055F1FB   |.  BF 04000000  |mov edi,4                             ;  做4轮变换

0055F200   |>  03D0         |/add edx,eax

0055F202   |.  03C2         ||add eax,edx

0055F204   |.  8BEA         ||mov ebp,edx

0055F206   |.  C1ED 07      ||shr ebp,7

0055F209   |.  33D5         ||xor edx,ebp

0055F20B   |.  03CA         ||add ecx,edx

0055F20D   |.  03D1         ||add edx,ecx

0055F20F   |.  8BE9         ||mov ebp,ecx

0055F211   |.  C1E5 0D      ||shl ebp,0D

0055F214   |.  33CD         ||xor ecx,ebp

0055F216   |.  03D9         ||add ebx,ecx

0055F218   |.  03CB         ||add ecx,ebx

0055F21A   |.  8BEB         ||mov ebp,ebx

0055F21C   |.  C1ED 11      ||shr ebp,11

0055F21F   |.  33DD         ||xor ebx,ebp

0055F221   |.  03C3         ||add eax,ebx

0055F223   |.  03D8         ||add ebx,eax

0055F225   |.  8BE8         ||mov ebp,eax

0055F227   |.  C1E5 09      ||shl ebp,9

0055F22A   |.  33C5         ||xor eax,ebp

0055F22C   |.  03D0         ||add edx,eax

0055F22E   |.  03C2         ||add eax,edx

0055F230   |.  8BEA         ||mov ebp,edx

0055F232   |.  C1ED 03      ||shr ebp,3

0055F235   |.  33D5         ||xor edx,ebp

0055F237   |.  03CA         ||add ecx,edx

0055F239   |.  03D1         ||add edx,ecx

0055F23B   |.  8BE9         ||mov ebp,ecx

0055F23D   |.  C1E5 07      ||shl ebp,7

0055F240   |.  33CD         ||xor ecx,ebp

0055F242   |.  03D9         ||add ebx,ecx

0055F244   |.  03CB         ||add ecx,ebx

0055F246   |.  8BE8         ||mov ebp,eax

0055F248   |.  C1ED 0F      ||shr ebp,0F

0055F24B   |.  33DD         ||xor ebx,ebp

0055F24D   |.  03C3         ||add eax,ebx

0055F24F   |.  03D8         ||add ebx,eax

0055F251   |.  8BE8         ||mov ebp,eax

0055F253   |.  C1E5 0B      ||shl ebp,0B

0055F256   |.  33C5         ||xor eax,ebp

0055F258   |.  4F           ||dec edi

0055F259   |.^ 75 A5        |\jnz short xx.0055F200

0055F25B   |.  899424 10010>|mov dword ptr ss:[esp+110],edx        ;  存储4个变量的变换结果

0055F262   |.  898C24 14010>|mov dword ptr ss:[esp+114],ecx

0055F269   |.  899C24 18010>|mov dword ptr ss:[esp+118],ebx

0055F270   |.  898424 1C010>|mov dword ptr ss:[esp+11C],eax

0055F277   |.  33C0         |xor eax,eax

0055F279   |.  898424 0C010>|mov dword ptr ss:[esp+10C],eax        ;  [12f804]=0

0055F280   |>  46           |inc esi

0055F281   |.  FF4C24 04    |dec dword ptr ss:[esp+4]

0055F285   |.^ 0F85 13FFFFF>\jnz xx.0055F19E

0055F28B   |>  8B1424       mov edx,dword ptr ss:[esp]

0055F28E   |.  8D4424 08    lea eax,dword ptr ss:[esp+8]

0055F292   |.  B9 18010000  mov ecx,118

0055F297   |.  E8 DCA70700  call xx.005D9A78                       ;  将[12f700]处的280个数据转存到[12f838]处

0055F29C   |.  81C4 2001000>add esp,120

0055F2A2   |.  5D           pop ebp

0055F2A3   |.  5F           pop edi

0055F2A4   |.  5E           pop esi

0055F2A5   |.  5B           pop ebx

0055F2A6   \.  C3           retn
```

## 跟进 `0055F37F` call `xx.Lbcipher::FinalizeLMD`

```
0055F2A8 x>/$  53           push ebx

0055F2A9   |.  56           push esi

0055F2AA   |.  57           push edi

0055F2AB   |.  55           push ebp

0055F2AC   |.  81C4 60FEFFF>add esp,-1A0

0055F2B2   |.  8BE9         mov ebp,ecx

0055F2B4   |.  8BFA         mov edi,edx

0055F2B6   |.  8BD8         mov ebx,eax

0055F2B8   |.  8BD4         mov edx,esp

0055F2BA   |.  8BC3         mov eax,ebx

0055F2BC   |.  B9 18010000  mov ecx,118

0055F2C1   |.  E8 B2A70700  call xx.005D9A78                       ;  将[12f838]处的280个数据转存到[12f678]处

0055F2C6   |.  B9 08000000  mov ecx,8

0055F2CB   |.  2B8C24 04010>sub ecx,dword ptr ss:[esp+104]         ;  ecx=6

0055F2D2   |.  BA B8126C00  mov edx,xx.006C12B8

0055F2D7   |.  8BC3         mov eax,ebx

0055F2D9   |.  E8 8EFEFFFF  call xx.Lbcipher::UpdateLMD            ;  继续用MD5加密01 00 00 00 00 00这6个字符

0055F2DE   |.  8BD4         mov edx,esp

0055F2E0   |.  8BC3         mov eax,ebx

0055F2E2   |.  B9 18010000  mov ecx,118

0055F2E7   |.  E8 8CA70700  call xx.005D9A78

0055F2EC   |.  6A 01        push 1                                 ; /Arg1 = 00000001

0055F2EE   |.  8D9424 1C010>lea edx,dword ptr ss:[esp+11C]         ; |

0055F2F5   |.  8D8424 0C010>lea eax,dword ptr ss:[esp+10C]         ; |

0055F2FC   |.  B9 08000000  mov ecx,8                              ; |

0055F301   |.  E8 2EE4FFFF  call xx.Lbcipher::InitEncryptLBC       ; \将MD5加密后的16位散列值在变换成为128位散列值

0055F306   |.  BE 10000000  mov esi,10                             ;  esi计数器

0055F30B   |.  8D5C24 04    lea ebx,dword ptr ss:[esp+4]

0055F30F   |>  8BD3         mov edx,ebx

0055F311   |.  8D8424 18010>lea eax,dword ptr ss:[esp+118]         ;  eax是4个MD5变量值变换后的地址

0055F318   |.  E8 4FE0FFFF  call xx.Lbcipher::EncryptLBC           ;  依次取128位散列值的16位进行加密运算

0055F31D   |.  83C3 10      add ebx,10

0055F320   |.  4E           dec esi

0055F321   |.^ 75 EC        jnz short xx.0055F30F                  ;  循环变换128位散列值

0055F323   |.  8BD5         mov edx,ebp

0055F325   |.  B8 00010000  mov eax,100

0055F32A   |.  E8 25DFFFFF  call xx.Lbutils::Min

0055F32F   |.  8BC8         mov ecx,eax

0055F331   |.  8BD7         mov edx,edi

0055F333   |.  8D4424 04    lea eax,dword ptr ss:[esp+4]

0055F337   |.  E8 3CA70700  call xx.005D9A78                       ;  将[12f67c]处的16个数据转存到[11f21a8]处

0055F33C   |.  81C4 A001000>add esp,1A0                            ;  这16位数为变换结果

0055F342   |.  5D           pop ebp

0055F343   |.  5F           pop edi

0055F344   |.  5E           pop esi

0055F345   |.  5B           pop ebx

0055F346   \.  C3           retn
```

## 跟进 `0055F37F0055F301` call `xx.Lbcipher::InitEncryptLBC`

```
0055D734 x>/$  55           push ebp

0055D735   |.  8BEC         mov ebp,esp

0055D737   |.  81C4 5CFFFFF>add esp,-0A4

0055D73D   |.  53           push ebx

0055D73E   |.  56           push esi

0055D73F   |.  57           push edi

0055D740   |.  8BD9         mov ebx,ecx

0055D742   |.  8955 FC      mov [local.1],edx

0055D745   |.  8945 F8      mov [local.2],eax

0055D748   |.  8B45 FC      mov eax,[local.1]

0055D74B   |.  8A55 08      mov dl,byte ptr ss:[ebp+8]

0055D74E   |.  8810         mov byte ptr ds:[eax],dl               ;  [12f790]=1

0055D750   |.  8BD3         mov edx,ebx

0055D752   |.  B8 10000000  mov eax,10

0055D757   |.  E8 F8FAFFFF  call xx.Lbutils::Min

0055D75C   |.  8BD0         mov edx,eax

0055D75E   |.  B8 04000000  mov eax,4

0055D763   |.  E8 F4FAFFFF  call xx.Lbutils::Max

0055D768   |.  8B55 FC      mov edx,[local.1]

0055D76B   |.  8942 04      mov dword ptr ds:[edx+4],eax           ;  [12f794]=8

0055D76E   |.  C745 E0 0400>mov [local.8],4                        ;  [12f64c]=4计算器

0055D775   |.  C745 DC F8D6>mov [local.9],xx.006BD6F8              ;  载入字符55 55 55 55 AA AA AA AA 33 33 33 33 CC CC CC CC

0055D77C   |.  8B45 FC      mov eax,[local.1]

0055D77F   |.  83C0 08      add eax,8

0055D782   |>  8B55 F8      /mov edx,[local.2]

0055D785   |.  8B12         |mov edx,dword ptr ds:[edx]            ;  取第1个变量

0055D787   |.  8B4D DC      |mov ecx,[local.9]

0055D78A   |.  8B31         |mov esi,dword ptr ds:[ecx]            ;  依次取4个载入的字符

0055D78C   |.  8BCE         |mov ecx,esi

0055D78E   |.  8B5D F8      |mov ebx,[local.2]

0055D791   |.  8B5B 04      |mov ebx,dword ptr ds:[ebx+4]          ;  取第2个变量

0055D794   |.  8975 F4      |mov [local.3],esi

0055D797   |.  8B7D F8      |mov edi,[local.2]

0055D79A   |.  8B7F 08      |mov edi,dword ptr ds:[edi+8]          ;  取第3个变量

0055D79D   |.  897D F0      |mov [local.4],edi

0055D7A0   |.  8975 EC      |mov [local.5],esi

0055D7A3   |.  8B7D F8      |mov edi,[local.2]

0055D7A6   |.  8B7F 0C      |mov edi,dword ptr ds:[edi+C]          ;  取第4个变量

0055D7A9   |.  897D E8      |mov [local.6],edi

0055D7AC   |.  8975 E4      |mov [local.7],esi

0055D7AF   |.  BE 08000000  |mov esi,8                             ;  做8轮变换
```
