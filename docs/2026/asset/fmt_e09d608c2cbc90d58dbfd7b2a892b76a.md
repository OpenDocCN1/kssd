
# 无敌图像印章 3.06注册算法分析

无敌图像印章 3.06注册算法分析。

## 软件简介
批量为图像添加印章、制作缩略图、添加特效、增加蒙版效果、图像浏览

* 特别增加了“藏宝”功能，即将一幅图片藏在另外一幅图片中(请登录 0e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6G2M7$3g2F1M7#2)9J5k6h3y4G2L8b7`.`. 选择“Show”查看藏宝效果)

【下载地址】：839K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2M7$3g2F1M7#2)9J5k6h3y4G2L8g2)9J5c8X3y4Z5K9h3&6S2i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2S2M7%4l9`.

【作者声明】：初学Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

================================================================================

## 破解过程

用`PEiD`，`ASP`壳。从`CrazyMarker/language`目录里看`Simplified chinese`文件`611`代码就是已注册版本。

用查找字符串插件查找`611`双击来到 。。。

```
0055D6A7    |.  64:FF30          push dword ptr fs:[eax]
0055D6AA    |.  64:8920          mov dword ptr fs:[eax],esp
0055D6AD    |.  A1 98FF5600      mov eax,dword ptr ds:[56FF98]
0055D6B2    |.  8B00             mov eax,dword ptr ds:[eax]
0055D6B4    |.  8B40 08          mov eax,dword ptr ds:[eax+8]
0055D6B7    |.  E8 DC7CFAFF      call unpacked.00505398                      ;  关键算法
0055D6BC    |.  84C0             test al,al
0055D6BE    |.  74 45            je short unpacked.0055D705                  ;  跳就未注册
0055D6C0    |.  8D55 FC          lea edx,dword ptr ss:[ebp-4]
0055D6C3    |.  B8 28D85500      mov eax,unpacked.0055D828                   ;  611=已注册版本
0055D6C8    |.  E8 175BFAFF      call unpacked.005031E4
0055D6CD    |.  8B55 FC          mov edx,dword ptr ss:[ebp-4]
0055D6D0    |.  8B83 3C030000    mov eax,dword ptr ds:[ebx+33C]
0055D6D6    |.  E8 9986EFFF      call unpacked.00455D74
0055D6DB    |.  33D2             xor edx,edx
0055D6DD    |.  8B83 24030000    mov eax,dword ptr ds:[ebx+324]
0055D6E3    |.  E8 8C86EFFF      call unpacked.00455D74
0055D6E8    |.  8D55 F8          lea edx,dword ptr ss:[ebp-8]
0055D6EB    |.  B8 34D85500      mov eax,unpacked.0055D834                   ;  615=有了您的注册，凌丽软件将会制造更多更好的软件产品，谢谢！
0055D6F0    |.  E8 EF5AFAFF      call unpacked.005031E4
0055D6F5    |.  8B55 F8          mov edx,dword ptr ss:[ebp-8]
0055D6F8    |.  8B83 28030000    mov eax,dword ptr ds:[ebx+328]
0055D6FE    |.  E8 7186EFFF      call unpacked.00455D74
0055D703    |.  EB 51            jmp short unpacked.0055D756
0055D705    |>  8D55 F4          lea edx,dword ptr ss:[ebp-C]
0055D708    |.  B8 40D85500      mov eax,unpacked.0055D840                   ;  610=未注册版本
0055D70D    |.  E8 D25AFAFF      call unpacked.005031E4
0055D712    |.  8B55 F4          mov edx,dword ptr ss:[ebp-C]
0055D715    |.  8B83 3C030000    mov eax,dword ptr ds:[ebx+33C]
0055D71B    |.  E8 5486EFFF      call unpacked.00455D74
0055D720    |.  8D55 F0          lea edx,dword ptr ss:[ebp-10]
0055D723    |.  B8 4CD85500      mov eax,unpacked.0055D84C                   ;  613
```

----------------------------------------- `call unpacked.00505398`                      ;  关键算法--------------------------------------------------

```
00505398    /$  E8 B3FCFFFF      call unpacked.00505050跟进。。。。。。。。
0050539D    \.  C3               retn
```

对注册码进行运算处理

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

```
00505086     .  BA 74535000      mov edx,unpacked.00505374                   ;  -
0050508B     .  8BC3             mov eax,ebx
0050508D     .  E8 86A3F0FF      call unpacked.0040F418
00505092     .  8D55 EC          lea edx,dword ptr ss:[ebp-14]               ;  注册码
00505095     .  8B45 FC          mov eax,dword ptr ss:[ebp-4]
00505098     .  E8 EBFEFFFF      call unpacked.00504F88                      ;  关键算法
0050509D     .  8B55 EC          mov edx,dword ptr ss:[ebp-14]
005050A0     .  8D45 FC          lea eax,dword ptr ss:[ebp-4]
005050A3     .  E8 D4F8EFFF      call unpacked.0040497C
005050A8     .  8B45 FC          mov eax,dword ptr ss:[ebp-4]
```

===============================================================================

### 对注册码的处理函数

`call unpacked.00504F88`                      ;  关键算法

=============================================================================

```
00504F88    /$  55               push ebp
00504F89    |.  8BEC             mov ebp,esp
00504F8B    |.  33C9             xor ecx,ecx
00504F8D    |.  51               push ecx
00504F8E    |.  51               push ecx
00504F8F    |.  51               push ecx
00504F90    |.  51               push ecx
00504F91    |.  51               push ecx
00504F92    |.  53               push ebx
00504F93    |.  56               push esi
00504F94    |.  57               push edi
00504F95    |.  8955 F8          mov dword ptr ss:[ebp-8],edx
00504F98    |.  8945 FC          mov dword ptr ss:[ebp-4],eax
00504F9B    |.  33C0             xor eax,eax
00504F9D    |.  55               push ebp
00504F9E    |.  68 42505000      push unpacked.00505042
00504FA3    |.  64:FF30          push dword ptr fs:[eax]
00504FA6    |.  64:8920          mov dword ptr fs:[eax],esp
00504FA9    |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]
00504FAC    |.  E8 EBFBEFFF      call unpacked.00404B9C                      ;  注册码长度
00504FB1    |.  8BF0             mov esi,eax
00504FB3    |.  85F6             test esi,esi
00504FB5    |.  79 03            jns short unpacked.00504FBA
00504FB7    |.  83C6 03          add esi,3
00504FBA    |>  C1FE 02          sar esi,2
00504FBD    |.  85F6             test esi,esi
00504FBF    |.  7E 5B            jle short unpacked.0050501C
00504FC1    |.  BF 01000000      mov edi,1
00504FC6    |>  8D45 F0          /lea eax,dword ptr ss:[ebp-10]
00504FC9    |.  50               |push eax
00504FCA    |.  8BD7             |mov edx,edi
00504FCC    |.  C1E2 02          |shl edx,2
00504FCF    |.  83EA 03          |sub edx,3
00504FD2    |.  B9 04000000      |mov ecx,4
00504FD7    |.  8B45 FC          |mov eax,dword ptr ss:[ebp-4]
00504FDA    |.  E8 15FEEFFF      |call unpacked.00404DF4                     ;  依次取4位注册码
00504FDF    |.  8B45 F0          |mov eax,dword ptr ss:[ebp-10]
00504FE2    |.  B2 10            |mov dl,10
00504FE4    |.  E8 536DFDFF      |call unpacked.004DBD3C                     ;  转换成16进制数
00504FE9    |.  8BD8             |mov ebx,eax
00504FEB    |.  85DB             |test ebx,ebx
00504FED    |.  79 05            |jns short unpacked.00504FF4
00504FEF    |.  E8 14EAEFFF      |call unpacked.00403A08
00504FF4    |>  81F3 79900000    |xor ebx,9079                               ;  和0X9079 XOR
00504FFA    |.  8BC3             |mov eax,ebx
00504FFC    |.  33D2             |xor edx,edx
00504FFE    |.  52               |push edx                                   ; /Arg2 => 00000000
00504FFF    |.  50               |push eax                                   ; |Arg1
00505000    |.  8D55 EC          |lea edx,dword ptr ss:[ebp-14]              ; |
00505003    |.  B8 04000000      |mov eax,4                                  ; |
00505008    |.  E8 3347F0FF      |call unpacked.00409740                     ; \再转换成字符串
0050500D    |.  8B55 EC          mov edx,dword ptr ss:[ebp-14]
00505010    |.  8D45 F4          lea eax,dword ptr ss:[ebp-C]
00505013    |.  E8 8CFBEFFF      call unpacked.00404BA4
00505018    |.  47               inc edi
00505019    |.  4E               dec esi
0050501A    |.^ 75 AA            \jnz short unpacked.00504FC6
```

=========================================================================

对注册码简单加密后来到这里

```
005050B0     .  83F8 14          cmp eax,14                                  ;  注册码长度是否等于20位
005050B3     .  74 07            je short unpacked.005050BC
005050B5     .  33DB             xor ebx,ebx
005050B7     .  E9 6A020000      jmp unpacked.00505326
005050BC     >  8B45 FC          mov eax,dword ptr ss:[ebp-4]
005050BF     .  E8 3059FDFF      call unpacked.004DA9F4                      ;  关键函数
005050C4     .  84C0             test al,al
005050C6     .  75 07            jnz short unpacked.005050CF
005050C8     .  33DB             xor ebx,ebx
```
