
# [原创]简单的盛高商务之星 2.1注册算法

- 【破文标题】盛高商务之星 2.1注册算法
- 【破文作者】XXNB
- 【作者邮箱】支持PYG
- 【作者主页】531K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6J5k6h3g2Q4x3X3g2&6M7K6p5$3z5q4)9J5k6h3y4G2L8g2)9J5c8W2)9K6c8X3u0A6L8X3u0A6L8X3u0A6L8U0M7@1y4e0j5`
- 【破解工具】OD
- 【破解平台】xpsp2
- 【软件名称】盛高商务之星 2.1
- 【软件大小】1809KB
- 【原版下载】b08K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8$3k6@1i4K6u0r3x3U0R3#2y4U0y4Q4x3X3g2Z5N6r3@1`
- 【保护方式】码
- 【软件简介】易用进销存管理软件，秉承 易用软件 一贯容易又实用的理念；从使用者角度出发，突出实用性，操作方便、简单，界面简洁，不需要专门培训都能使用、操作。安装简单，不需要专门技术人员维护。本软件适合中小批发零售企业进销存业务管理，注册用户享受免费升级。易用进销存软件以业务为基础，整个系统一体化设计，功能流畅，操作方便，界面美观友好，是您商贸管理的好帮手！
- 【破解声明】向大侠们学习！！！只为学习！请尊重作者的劳动成功！

------------------------------------------------------------------------

## 【破解过程】

### 1、字符串查找“注册成功”得到下面关键代码：

```
005AFC9C  /.  55            push    ebp
005AFC9D  |.  8BEC          mov     ebp, esp
005AFC9F  |.  6A 00         push    0
005AFCA1  |.  53            push    ebx
005AFCA2  |.  8BD8          mov     ebx, eax
005AFCA4  |.  33C0          xor     eax, eax
005AFCA6  |.  55            push    ebp
005AFCA7  |.  68 49FD5A00   push    005AFD49
005AFCAC  |.  64:FF30       push    dword ptr fs:[eax]
005AFCAF  |.  64:8920       mov     dword ptr fs:[eax], esp
005AFCB2  |.  66:8339 0D    cmp     word ptr [ecx], 0D
005AFCB6  |.  75 7B         jnz     short 005AFD33
005AFCB8  |.  8D55 FC       lea     edx, dword ptr [ebp-4]
005AFCBB  |.  8B83 14030000 mov     eax, dword ptr [ebx+314]
005AFCC1  |.  E8 2690EBFF   call    00468CEC
005AFCC6  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]           ;  假码出现了。eax是位数
005AFCC9  |.  A1 30595B00   mov     eax, dword ptr [5B5930]
005AFCCE  |.  8B00          mov     eax, dword ptr [eax]
005AFCD0  |.  BA 60FD5A00   mov     edx, 005AFD60                    ;  pid
005AFCD5  |.  E8 B2120000   call    005B0F8C
005AFCDA  |.  A1 30595B00   mov     eax, dword ptr [5B5930]
005AFCDF  |.  8B00          mov     eax, dword ptr [eax]
005AFCE1  |.  E8 AA160000   call    005B1390                         ;  关键算法，跟进《《《《《《《《《《－－－－－－－
005AFCE6  |.  84C0          test    al, al                           ;  注册标志位
005AFCE8  |.  74 31         je      short 005AFD1B                   ;  关键跳转
005AFCEA  |.  6A 00         push    0
005AFCEC  |.  B9 64FD5A00   mov     ecx, 005AFD64                    ;  提示
005AFCF1  |.  BA 6CFD5A00   mov     edx, 005AFD6C                    ;  注册成功！
005AFCF6  |.  A1 D05A5B00   mov     eax, dword ptr [5B5AD0]
005AFCFB  |.  8B00          mov     eax, dword ptr [eax]
005AFCFD  |.  E8 369EEDFF   call    00489B38
005AFD02  |.  A1 30595B00   mov     eax, dword ptr [5B5930]
005AFD07  |.  8B00          mov     eax, dword ptr [eax]
005AFD09  |.  8B80 34030000 mov     eax, dword ptr [eax+334]
005AFD0F  |.  BA 80FD5A00   mov     edx, 005AFD80                    ;  已注册
005AFD14  |.  E8 0390EBFF   call    00468D1C
005AFD19  |.  EB 18         jmp     short 005AFD33
005AFD1B  |>  6A 00         push    0
005AFD1D  |.  B9 64FD5A00   mov     ecx, 005AFD64                    ;  提示
005AFD22  |.  BA 88FD5A00   mov     edx, 005AFD88                    ;  注册失败，请检查注册号码！
005AFD27  |.  A1 D05A5B00   mov     eax, dword ptr [5B5AD0]
005AFD2C  |.  8B00          mov     eax, dword ptr [eax]
005AFD2E  |.  E8 059EEDFF   call    00489B38
005AFD33  |>  33C0          xor     eax, eax
005AFD35  |.  5A            pop     edx
005AFD36  |.  59            pop     ecx
005AFD37  |.  59            pop     ecx
005AFD38  |.  64:8910       mov     dword ptr fs:[eax], edx
005AFD3B  |.  68 50FD5A00   push    005AFD50
005AFD40  |>  8D45 FC       lea     eax, dword ptr [ebp-4]
005AFD43  |.  E8 0C46E5FF   call    00404354
005AFD48  \.  C3            retn
```

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

### 2、跟进算法`call 005AFCE1 |. E8 AA160000 call 005B1390`后得到：

```
005B1390  /$  55            push    ebp
005B1391  |.  8BEC          mov     ebp, esp
005B1393  |.  33C9          xor     ecx, ecx
005B1395  |.  51            push    ecx
005B1396  |.  51            push    ecx
005B1397  |.  51            push    ecx
005B1398  |.  51            push    ecx
005B1399  |.  51            push    ecx
005B139A  |.  51            push    ecx
005B139B  |.  53            push    ebx
005B139C  |.  56            push    esi
005B139D  |.  8BD8          mov     ebx, eax
005B139F  |.  33C0          xor     eax, eax
005B13A1  |.  55            push    ebp
005B13A2  |.  68 57145B00   push    005B1457
005B13A7  |.  64:FF30       push    dword ptr fs:[eax]
005B13AA  |.  64:8920       mov     dword ptr fs:[eax], esp
005B13AD  |.  8D45 F4       lea     eax, dword ptr [ebp-C]
005B13B0  |.  BA 70145B00   mov     edx, 005B1470                    ;  固定字符串(ASCII "0AB12CD34EF56789")
005B13B5  |.  E8 3230E5FF   call    004043EC
005B13BA  |.  8D55 FC       lea     edx, dword ptr [ebp-4]
005B13BD  |.  8BC3          mov     eax, ebx
005B13BF  |.  E8 F0FEFFFF   call    005B12B4                         ;  生成机器码的call
005B13C4  |.  8D4D F8       lea     ecx, dword ptr [ebp-8]
005B13C7  |.  BA 8C145B00   mov     edx, 005B148C                    ;  ASCII "PID"
005B13CC  |.  8BC3          mov     eax, ebx
005B13CE  |.  E8 EDFAFFFF   call    005B0EC0
005B13D3  |.  837D F8 00    cmp     dword ptr [ebp-8], 0             ;  看看有没有输入注册码
005B13D7  |.  75 04         jnz     short 005B13DD
005B13D9  |.  33DB          xor     ebx, ebx
005B13DB  |.  EB 5F         jmp     short 005B143C
005B13DD  |>  8D45 F0       lea     eax, dword ptr [ebp-10]
005B13E0  |.  E8 6F2FE5FF   call    00404354
005B13E5  |.  8B45 FC       mov     eax, dword ptr [ebp-4]           ;  机器码出现了
005B13E8  |.  E8 3732E5FF   call    00404624                         ;  计算位数
005B13ED  |.  8BD8          mov     ebx, eax
005B13EF  |.  85DB          test    ebx, ebx
005B13F1  |.  7E 36         jle     short 005B1429
005B13F3  |.  BE 01000000   mov     esi, 1                           ;  计数器
005B13F8  |>  8D45 EC       /lea     eax, dword ptr [ebp-14]         ;  循环开始
005B13FB  |.  8B55 FC       |mov     edx, dword ptr [ebp-4]          ;  机器码
005B13FE  |.  8A5432 FF     |mov     dl, byte ptr [edx+esi-1]        ;  逐个取机器码Ascii码值
005B1402  |.  E8 3531E5FF   |call    0040453C
005B1407  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]          ;  固定字符串到edx
005B140A  |.  8B45 EC       |mov     eax, dword ptr [ebp-14]
005B140D  |.  E8 5635E5FF   |call    00404968                        ;  看看取出的机器码在固定字符的下标数
005B1412  |.  8D55 E8       |lea     edx, dword ptr [ebp-18]
005B1415  |.  E8 CE7EE5FF   |call    004092E8                        ;  得到下标的10进制数
005B141A  |.  8B55 E8       |mov     edx, dword ptr [ebp-18]
005B141D  |.  8D45 F0       |lea     eax, dword ptr [ebp-10]
005B1420  |.  E8 0732E5FF   |call    0040462C                        ;  连接
005B1425  |.  46            |inc     esi
005B1426  |.  4B            |dec     ebx
005B1427  |.^ 75 CF         \jnz     short 005B13F8                  ;  循环结束
005B1429  |>  8B45 F0       mov     eax, dword ptr [ebp-10]          ;  上面循环的结果就是这个了
005B142C  |.  8B55 F8       mov     edx, dword ptr [ebp-8]           ;  假码到eax。看来要比较了
005B142F  |.  E8 3C33E5FF   call    00404770
005B1434  |.  75 04         jnz     short 005B143A                   ;  经典比较函数
005B1436  |.  B3 01         mov     bl, 1                            ;  注册标志位
005B1438  |.  EB 02         jmp     short 005B143C
005B143A  |>  33DB          xor     ebx, ebx
005B143C  |>  33C0          xor     eax, eax
005B143E  |.  5A            pop     edx
005B143F  |.  59            pop     ecx
005B1440  |.  59            pop     ecx
005B1441  |.  64:8910       mov     dword ptr fs:[eax], edx
005B1444  |.  68 5E145B00   push    005B145E
005B1449  |>  8D45 E8       lea     eax, dword ptr [ebp-18]
005B144C  |.  BA 06000000   mov     edx, 6
005B1451  |.  E8 222FE5FF   call    00404378
005B1456  \.  C3            retn
```
