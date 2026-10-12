
# 阳光个人助理 1.30

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-457.htm`](https://www.pediy.com/kssd/pediy08/pediy8-457.htm)

## 破文信息

- **【破文标题】**：`阳光个人助理 1.30 注册算法`
- **【破文作者】**：`XXNB`
- **【作者邮箱】**：
- **【作者主页】**：`http://free.ys168.com/?binbinbin7456`
- **【破解工具】**：`OD`
- **【破解平台】**：`xpsp2`
- **【软件名称】**：`阳光个人助理 1.30`
- **【软件大小】**：`2098KB`
- **【原版下载】**：`http://www.newhua.com/soft/47429.htm`
- **【保护方式】**：名+码

### 软件简介

`阳光个人助理` 是一款实用性极强的个人电脑超级助理软件，集同类软件之所长，具有丰富而强大的功能：有工作辅助、日常查询、文件处理、网络管理、系统优化等，它采用数据库加密方式保存，可以多个用户使用，安全便捷，大大提高信息管理的效率。无论是查询功能还是信息的处理，`阳光个人助理` 都将成为您最得力的助手。

软件共分五个部分：

- **工作辅助**：定时提醒、工作日志、密码保管箱、通讯录；
- **日常查询**：万年历、身份证查询、邮政编码查询、长途区号查询、手机归属地查询；
- **文件辅助**：`文件加密助手`、`文件分割助手`、`文件彻底删除助手`；
- **网络辅助**：常用网址管理、`IE` 恢复助手、网络信使；
- **系统辅助**：系统信息、内存整理、上机纪录清理、热键管理；

本软件具有以下特色：

- **功能强**：集成了日常工作生活中最常用的信息管理功能；
- **人性化**：各项功能精心打造，让您感觉周到体贴、用着舒心；
- **自动化**：软件中的处理、计算自动化程度高，省心省力；
- **安全性**：所有数据加密保存，个人资料使用安全、放心；
- **空间小**：程序精干、硬盘占据空间小，适合下载使用；
- **扩充性**：系统整体设计考虑全面，功能扩充的余地广阔；
- **易操作**：流程简洁、操作简便、好学易用、驾驭自如；

### 破解声明

向大侠们学习！！！只为学习！请尊重作者的劳动成功！

------------------------------------------------------------------------

### 破解过程

1、下断的办法很多，我使用的是 `bp RegQueryValueExA` 读取注册表的函数下断，因为我的 `OD` 在字符串搜索的时候会死机，郁闷～，然后可以得到下面代码：

```
004F72D4  /$  55            push    ebp
004F72D5  |.  8BEC          mov     ebp, esp
004F72D7  |.  6A 00         push    0
004F72D9  |.  6A 00         push    0
004F72DB  |.  6A 00         push    0
004F72DD  |.  6A 00         push    0
004F72DF  |.  53            push    ebx
004F72E0  |.  56            push    esi
004F72E1  |.  57            push    edi
004F72E2  |.  8BF9          mov     edi, ecx
004F72E4  |.  8BDA          mov     ebx, edx
004F72E6  |.  8BF0          mov     esi, eax
004F72E8  |.  33C0          xor     eax, eax
004F72EA  |.  55            push    ebp
004F72EB  |.  68 8E734F00   push    004F738E
004F72F0  |.  64:FF30       push    dword ptr fs:[eax]
004F72F3  |.  64:8920       mov     dword ptr fs:[eax], esp
004F72F6  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
004F72F9  |.  E8 62D1F0FF   call    00404460
004F72FE  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
004F7301  |.  E8 5AD1F0FF   call    00404460
004F7306  |.  C607 00       mov     byte ptr [edi], 0
004F7309  |.  8D55 F8       lea     edx, dword ptr [ebp-8]
004F730C  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
004F730F  |.  E8 8C88FAFF   call    0049FBA0                         ;  读取假码
004F7314  |.  837D FC 00    cmp     dword ptr [ebp-4], 0             ;  判断是否输入假码
004F7318  |.  74 59         je      short 004F7373
004F731A  |.  837D F8 00    cmp     dword ptr [ebp-8], 0             ;  判断是否输入用户名
004F731E  |.  74 53         je      short 004F7373
004F7320  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
004F7323  |.  B8 A4734F00   mov     eax, 004F73A4                    ;  固定字符串  "SOFTAA"
004F7328  |.  E8 C364FAFF   call    0049D7F0
004F732D  |.  8B55 F4       mov     edx, dword ptr [ebp-C]           ;  取计算机硬盘序列号
004F7330  |.  8BC3          mov     eax, ebx
004F7332  |.  E8 7DD1F0FF   call    004044B4
004F7337  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
004F733A  |.  50            push    eax
004F733B  |.  8B03          mov     eax, dword ptr [ebx]             ;  硬盘序列号到 eax
004F733D  |.  B1 61         mov     cl, 61                           ;  常数
004F733F  |.  66:BA A508    mov     dx, 8A5                          ;  常数
004F7343  |.  E8 346EFAFF   call    0049E17C
004F7348  |.  8B55 F0       mov     edx, dword ptr [ebp-10]          ;  机器码到 edx
004F734B  |.  8BC3          mov     eax, ebx
004F734D  |.  E8 62D1F0FF   call    004044B4
004F7352  |.  8B45 FC       mov     eax, dword ptr [ebp-4]           ;  假码出现了。
004F7355  |.  50            push    eax                               ;  假码入栈
004F7356  |.  57            push    edi
004F7357  |.  8BD3          mov     edx, ebx
004F7359  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]           ;  用户名到 ecx
004F735C  |.  8BC6          mov     eax, esi
004F735E  |.  E8 5D000000   call    004F73C0                         ;  算法 call，跟进《《《《《《《《《《《《《--------
004F7363  |.  84C0          test    al, al                           ;  标志位
004F7365  |.  74 0C         je      short 004F7373                   ;  关键跳
004F7367  |.  8BC3          mov     eax, ebx
004F7369  |.  BA B4734F00   mov     edx, 004F73B4                    ;  ASCII "YesIKnow"
004F736E  |.  E8 41D1F0FF   call    004044B4
004F7373  |>  33C0          xor     eax, eax
004F7375  |.  5A            pop     edx
004F7376  |.  59            pop     ecx
004F7377  |.  59            pop     ecx
004F7378  |.  64:8910       mov     dword ptr fs:[eax], edx
004F737B  |.  68 95734F00   push    004F7395
004F7380  |>  8D45 F0       lea     eax, dword ptr [ebp-10]
004F7383  |.  BA 04000000   mov     edx, 4
004F7388  |.  E8 F7D0F0FF   call    00404484
004F738D  \\.  C3           ret
```

2、跟进 `004F735E  |.  E8 5D000000   call    004F73C0` 这个算法 `call` 得到：

```
004F73C0  /$  55            push    ebp
004F73C1  |.  8BEC          mov     ebp, esp
004F73C3  |.  51            push    ecx
004F73C4  |.  B9 05000000   mov     ecx, 5
004F73C9  |>  6A 00         /push    0
004F73CB  |.  6A 00         |push    0
004F73CD  |.  49            |dec     ecx
004F73CE  |.^ 75 F9         \\jnz     short 004F73C9
004F73D0  |.  51            push    ecx
004F73D1  |.  874D FC       xchg    dword ptr [ebp-4], ecx
004F73D4  |.  53            push    ebx
004F73D5  |.  56            push    esi
004F73D6  |.  894D F8       mov     dword ptr [ebp-8], ecx
004F73D9  |.  8BF2          mov     esi, edx
004F73DB  |.  8945 FC       mov     dword ptr [ebp-4], eax
004F73DE  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
004F73E1  |.  E8 2AD5F0FF   call    00404910
004F73E6  |.  8B45 0C       mov     eax, dword ptr [ebp+C]
004F73E9  |.  E8 22D5F0FF   call    00404910
004F73EE  |.  33C0          xor     eax, eax
004F73F0  |.  55            push    ebp
004F73F1  |.  68 A0764F00   push    004F76A0
004F73F6  |.  64:FF30       push    dword ptr fs:[eax]
004F73F9  |.  64:8920       mov     dword ptr fs:[eax], esp
004F73FC  |.  33DB          xor     ebx, ebx
004F73FE  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
004F7401  |.  B8 BC764F00   mov     eax, 004F76BC                    ;  (ASCII "SOFTAA")固定字符串。（好像没有用的）
004F7406  |.  E8 E563FAFF   call    0049D7F0
004F740B  |.  8D45 E0       lea     eax, dword ptr [ebp-20]
004F740E  |.  50            push    eax
004F740F  |.  8B06          mov     eax, dword ptr [esi]             ;  机器码
004F7411  |.  B1 78         mov     cl, 78                           ;  常数
004F7413  |.  66:BA A508    mov     dx, 8A5                          ;  常数
004F7417  |.  E8 606DFAFF   call    0049E17C
004F741C  |.  8B55 E0       mov     edx, dword ptr [ebp-20]
004F741F  |.  8B45 F4       mov     eax, dword ptr [ebp-C]
004F7422  |.  E8 45D4F0FF   call    0040486C
004F7427  |.  0F85 50020000 jnz     004F767D
004F742D  |.  8B45 0C       mov     eax, dword ptr [ebp+C]           ;  假码
004F7430  |.  E8 EBD2F0FF   call    00404720                         ;  计算位数
004F7435  |.  8BF0          mov     esi, eax
004F7437  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
004F743A  |.  50            push    eax
004F743B  |.  8D45 DC       lea     eax, dword ptr [ebp-24]
004F743E  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]
004F7441  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
004F7444  |.  E8 23D3F0FF   call    0040476C
004F7449  |.  8B4D DC       mov     ecx, dword ptr [ebp-24]         ;  硬盘序列号和用户名连接成一个字符串
004F744C  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
```



```
004F744F  |.  8B40 60       mov     eax, dword ptr [eax+60]  
004F7452  |.  BA C8010000   mov     edx, 1C8                     ; 常数。这个常数厉害了，贯穿整个算法  
004F7457  |.  E8 FC88FDFF   call    004CFD58                     ; 算法 call，跟进《《《《《《《《《《《《《  
004F745C  |.  8B45 EC       mov     eax, dword ptr [ebp-14]      ; 这个难道是真码了！！  
004F745F  |.  8B55 0C       mov     edx, dword ptr [ebp+C]       ; 假码。看到气势都知道是真假比较了。收工  
004F7462  |.  E8 05D4F0FF   call    0040486C                     ; 经典比较函数  
004F7467  |.  75 07         jnz     short 004F7470               ; 关键跳  
004F7469  |.  B3 01         mov     bl, 1                        ; 标志位  
004F746B  |.  E9 0D020000   jmp     004F767D                     ; 跳向光明....  
004F7470  |>  8D45 E8       lea     eax, dword ptr [ebp-18]  
004F7473  |.  50            push    eax  
004F7474  |.  8D45 D8       lea     eax, dword ptr [ebp-28]  
004F7477  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]  
004F747A  |.  8B55 F4       mov     edx, dword ptr [ebp-C]  
004F747D  |.  E8 EAD2F0FF   call    0040476C  
004F7482  |.  8B4D D8       mov     ecx, dword ptr [ebp-28]      ; 又是连接硬盘序列号和用户名字符串  
004F7485  |.  8B45 FC       mov     eax, dword ptr [ebp-4]  
004F7488  |.  8B40 60       mov     eax, dword ptr [eax+60]  
004F748B  |.  BA 2E000000   mov     edx, 2E                      ; 常数 2E。又是贯穿整个算法的常数，  
004F7490  |.  E8 C388FDFF   call    004CFD58                     ; 作者难道给两次机会我们。所以应该有两组注册码  
004F7495  |.  8B45 E8       mov     eax, dword ptr [ebp-18]      ; 第二组真码出现了  
004F7498  |.  8B55 0C       mov     edx, dword ptr [ebp+C]       ; 假码  
004F749B  |.  E8 CCD3F0FF   call    0040486C                     ; 比较  
004F74A0  |.  75 07         jnz     short 004F74A9               ; 关键跳  
004F74A2  |.  B3 01         mov     bl, 1                        ; 注册标志位  
004F74A4  |.  E9 D4010000   jmp     004F767D  
004F74A9  |>  8B45 E8       mov     eax, dword ptr [ebp-18]  
004F74AC  |.  E8 6FD2F0FF   call    00404720  
004F74B1  |.  3BF0          cmp     esi, eax  
004F74B3  |.  7E 0A         jle     short 004F74BF  
004F74B5  |.  8B45 E8       mov     eax, dword ptr [ebp-18]  
004F74B8  |.  E8 63D2F0FF   call    00404720  
004F74BD  |.  8BF0          mov     esi, eax  
004F74BF  |>  8BC6          mov     eax, esi  
004F74C1  |.  83F8 01       cmp     eax, 1  
004F74C4  |.  7C 21         jl      short 004F74E7  
004F74C6  |>  84DB          /test   bl, bl  
004F74C8  |.  74 10         |je     short 004F74DA  
004F74CA  |.  8B55 0C       |mov    edx, dword ptr [ebp+C]  
004F74CD  |.  8A5402 FF     |mov    dl, byte ptr [edx+eax-1]  
004F74D1  |.  8B4D E8       |mov    ecx, dword ptr [ebp-18]  
004F74D4  |.  3A5401 FF     |cmp    dl, byte ptr [ecx+eax-1]  
004F74D8  |.  74 04         |je     short 004F74DE  
004F74DA  |>  33D2          |xor    edx, edx  
004F74DC  |.  EB 02         |jmp    short 004F74E0  
004F74DE  |>  B2 01         |mov    dl, 1  
004F74E0  |>  8BDA          |mov    ebx, edx  
004F74E2  |.  48            |dec    eax  
004F74E3  |.  85C0          |test   eax, eax  
004F74E5  |.^ 75 DF         \jnz    short 004F74C6  
004F74E7  |>  8D45 E4       lea     eax, dword ptr [ebp-1C]  
004F74EA  |.  50            push    eax  
004F74EB  |.  8D45 D4       lea     eax, dword ptr [ebp-2C]  
004F74EE  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]  
004F74F1  |.  8B55 F4       mov     edx, dword ptr [ebp-C]  
004F74F4  |.  E8 73D2F0FF   call    0040476C                     ; 不会吧。给三次机会？？  
004F74F9  |.  8B4D D4       mov     ecx, dword ptr [ebp-2C]      ; 又是硬盘序列号和用户名连接后的字符串  
004F74FC  |.  8B45 FC       mov     eax, dword ptr [ebp-4]  
004F74FF  |.  8B40 60       mov     eax, dword ptr [eax+60]  
004F7502  |.  BA 38000000   mov     edx, 38                      ; 贯穿算法的常数：38H  
004F7507  |.  E8 4C88FDFF   call    004CFD58                     ; 计算第三组注册码  
004F750C  |.  8B45 0C       mov     eax, dword ptr [ebp+C]       ; 假码  
004F750F  |.  E8 0CD2F0FF   call    00404720                     ; 计算位数  
004F7514  |.  8BF0          mov     esi, eax  
004F7516  |.  8B45 E4       mov     eax, dword ptr [ebp-1C]      ; 第三组注册码  
004F7519  |.  E8 02D2F0FF   call    00404720                     ; 计算位数  
004F751E  |.  3BF0          cmp     esi, eax                     ; 假码和第三组注册码的比较  
004F7520  |.  7E 0A         jle     short 004F752C  
004F7522  |.  8B45 E4       mov     eax, dword ptr [ebp-1C]  
004F7525  |.  E8 F6D1F0FF   call    00404720  
004F752A  |.  8BF0          mov     esi, eax  
004F752C  |>  8B45 E4       mov     eax, dword ptr [ebp-1C]  
004F752F  |.  8B55 0C       mov     edx, dword ptr [ebp+C]       ; 假码和第三组注册码比较  
004F7532  |.  E8 35D3F0FF   call    0040486C                     ; 比较函数  
004F7537  |.  75 07         jnz     short 004F7540  
004F7539  |.  B3 01         mov     bl, 1                        ; 注册标志位  
004F753B  |.  E9 3D010000   jmp     004F767D  
004F7540  |>  8BC6          mov     eax, esi  
004F7542  |.  83F8 01       cmp     eax, 1  
004F7545  |.  7C 21         jl      short 004F7568  
004F7547  |>  84DB          /test   bl, bl  
004F7549  |.  74 10         |je     short 004F755B  
004F754B  |.  8B55 0C       |mov    edx, dword ptr [ebp+C]  
004F754E  |.  8A5402 FF     |mov    dl, byte ptr [edx+eax-1]  
004F7552  |.  8B4D E4       |mov    ecx, dword ptr [ebp-1C]  
004F7555  |.  3A5401 FF     |cmp    dl, byte ptr [ecx+eax-1]  
004F7559  |.  74 04         |je     short 004F755F  
004F755B  |>  33D2          |xor    edx, edx  
004F755D  |.  EB 02         |jmp    short 004F7561  
004F755F  |>  B2 01         |mov    dl, 1  
004F7561  |>  8BDA          |mov    ebx, edx  
004F7563  |.  48            |dec    eax  
004F7564  |.  85C0          |test   eax, eax  
004F7566  |.^ 75 DF         \jnz    short 004F7547  
004F7568  |>  B3 01         mov     bl, 1  
004F756A  |.  8D45 F0       lea     eax, dword ptr [ebp-10]  
004F756D  |.  50            push    eax  
004F756E  |.  8D45 D0       lea     eax, dword ptr [ebp-30]  
004F7571  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]  
004F7574  |.  8B55 F4       mov     edx, dword ptr [ebp-C]  
004F7577  |.  E8 F0D1F0FF   call    0040476C                     ; 第四次运算。  
004F757C  |.  8B4D D0       mov     ecx, dword ptr [ebp-30]      ; 用户名和硬盘序列号的连接字符串  
004F757F  |.  8B45 FC       mov     eax, dword ptr [ebp-4]  
004F7582  |.  8B40 60       mov     eax, dword ptr [eax+60]  
004F7585  |.  BA 59010000   mov     edx, 159                     ; 常数  
004F758A  |.  E8 C987FDFF   call    004CFD58                     ; 堆栈里面看第四组注册码  
004F758F  |.  8D45 EC       lea     eax, dword ptr [ebp-14]  
004F7592  |.  BA CC764F00   mov     edx, 004F76CC                ; (ASCII "tH4gI8fsU0")固定字符串  
004F7597  |.  E8 5CCFF0FF   call    004044F8  
004F759C  |.  8B45 0C       mov     eax, dword ptr [ebp+C]       ; 假码  
004F759F  |.  E8 7CD1F0FF   call    00404720                     ; 计算位数  
004F75A4  |.  8BF0          mov     esi, eax  
004F75A6  |.  8B45 F0       mov     eax, dword ptr [ebp-10]      ; 第四组  
004F75A9  |.  E8 72D1F0FF   call    00404720                     ; 计算位数  
004F75AE  |.  3BF0          cmp     esi, eax                     ; 位数比较  
004F75B0  |.  74 07         je      short 004F75B9  
004F75B2  |.  33DB          xor     ebx, ebx  
004F75B4  |.  E9 C4000000   jmp     004F767D                     ; 终于在这里跳走了  
004F75B9  |>  8BC6          mov     eax, esi  
004F75BB  |.  83E8 04       sub     eax, 4  
004F75BE  |.  83F8 08       cmp     eax, 8  
```

### 跟进总共调用了 4 次的算法 call

就是生成注册码的：`004F7457 |. E8 FC88FDFF call 004CFD58` 可以得到下面代码：



# 程序分析总结

## 汇编代码段

以下代码段展示了注册码验证逻辑的核心汇编指令。该部分代码负责处理字符串连接、字符累加、常量比较及多段校验码的生成与拼接。

```asm
004CFD58  $   55            push    ebp  
004CFD59  |.  8BEC          mov     ebp, esp  
004CFD5B  |.  6A 00         push    0  
004CFD5D  |.  6A 00         push    0  
004CFD5F  |.  6A 00         push    0  
004CFD61  |.  6A 00         push    0  
004CFD63  |.  6A 00         push    0  
004CFD65  |.  6A 00         push    0  
004CFD67  |.  6A 00         push    0  
004CFD69  |.  53            push    ebx  
004CFD6A  |.  56            push    esi  
004CFD6B  |.  894D F8       mov     dword ptr [ebp-8], ecx          ; 连接后的字符串  
004CFD6E  |.  8955 FC       mov     dword ptr [ebp-4], edx  
004CFD71  |.  8B45 F8       mov     eax, dword ptr [ebp-8]  
004CFD74  |.  E8 974BF3FF   call    00404910  
004CFD79  |.  33C0          xor     eax, eax  
004CFD7B  |.  55            push    ebp  
004CFD7C  |.  68 86FE4C00   push    004CFE86  
004CFD81  |.  64:FF30       push    dword ptr fs:[eax]  
004CFD84  |.  64:8920       mov     dword ptr fs:[eax], esp  
004CFD87  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
004CFD8A  |.  BA A0FE4C00   mov     edx, 004CFEA0                   ; ASCII "Error"  
004CFD8F  |.  E8 6447F3FF   call    004044F8  
004CFD94  |.  33DB          xor     ebx, ebx  
004CFD96  |.  837D FC 00    cmp     dword ptr [ebp-4], 0            ; 常数 1C8  
004CFD9A  |.  0F84 B3000000 je      004CFE53  
004CFDA0  |.  837D F8 00    cmp     dword ptr [ebp-8], 0  
004CFDA4  |.  0F84 A9000000 je      004CFE53  
004CFDAA  |.  8B45 F8       mov     eax, dword ptr [ebp-8]          ; 连接后的字符串到 eax  
004CFDAD  |.  E8 6E49F3FF   call    00404720                        ; 计算位数  
004CFDB2  |.  8945 F0       mov     dword ptr [ebp-10], eax  
004CFDB5  |.  8B45 F0       mov     eax, dword ptr [ebp-10]  
004CFDB8  |.  85C0          test    eax, eax  
004CFDBA  |.  7E 13         jle     short 004CFDCF                  ; 一看都知道要循环了  
004CFDBC  |.  BA 01000000   mov     edx, 1                          ; 计数器  
004CFDC1  |>  8B4D F8       /mov    ecx, dword ptr [ebp-8]  
004CFDC4  |.  0FB64C11 FF   |movzx  ecx, byte ptr [ecx+edx-1]      ; 逐个取那连接后的字符串  
004CFDC9  |.  03D9          |add     ebx, ecx                      ; 累加  
004CFDCB  |.  42            |inc     edx  
004CFDCC  |.  48            |dec     eax  
004CFDCC  |.^ 75 F2         \jnz    short 004CFDC1                ; 这个循环目的是 Ascii 码值累加  
004CFDCF  |>  81FB 204E0000 cmp     ebx, 4E20                     ; 累加后的结果和常数 4E20 比较  
004CFDD5  |.  7E 0C         jle     short 004CFDE3                ; 小于等于就跳。相信很多人的都是小于的  
004CFDD7  |.  8BC3          mov     eax, ebx  
004CFDD9  |.  B9 204E0000   mov     ecx, 4E20  
004CFDDE  |.  99            cdq  
004CFDDF  |.  F7F9          idiv    ecx  
004CFDE1  |.  8BDA          mov     ebx, edx  
004CFDE3  |>  8B75 FC       mov     esi, dword ptr [ebp-4]        ; 常数 1C8  
004CFDE6  |.  0FAF75 F0     imul    esi, dword ptr [ebp-10]       ; 1C8 乘以那长字符串的位数  
004CFDEA  |.  6BC3 49       imul    eax, ebx, 49                  ; 累加的结果*49H。结果放在 eax  
004CFDED  |.  03F0          add     esi, eax                      ; 相加  
004CFDEF  |.  8D55 EC       lea     edx, dword ptr [ebp-14]  
004CFDF2  |.  8BC6          mov     eax, esi                      ; 上面那堆运算的结果放到 eax  
004CFDF4  |.  E8 4B98F3FF   call    00409644                       ; 转成 10 进制字符串函数。  
004CFDF9  |.  8B55 EC       mov     edx, dword ptr [ebp-14]       ; 到这里位置算得的 10 进制字符串，就是真码的第一部分  
004CFDFC  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
004CFDFF  |.  B9 B0FE4C00   mov     ecx, 004CFEB0  
004CFE04  |.  E8 6349F3FF   call    0040476C  
004CFE09  |.  8B45 FC       mov     eax, dword ptr [ebp-4]        ; 又是常数 1C8  
004CFE0C  |.  F7EB          imul    ebx                             ; 1C8 乘以累加结果  
004CFE0E  |.  6BF0 15       imul    esi, eax, 15                   ; 再乘以 15H。结果放到 esi  
004CFE11  |.  FF75 F4       push    dword ptr [ebp-C]              ; 第一部分入栈  
004CFE14  |.  8D55 E8       lea     edx, dword ptr [ebp-18]  
004CFE17  |.  8BC6          mov     eax, esi  
004CFE19  |.  E8 2698F3FF   call    00409644                       ; 转成 10 进制字符串函数。  
004CFE1E  |.  FF75 E8       push    dword ptr [ebp-18]             ; 真码的第二部分出现了，记为 BB  
004CFE21  |.  68 B0FE4C00   push    004CFEB0  
004CFE26  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
004CFE29  |.  BA 03000000   mov     edx, 3                        ;   
004CFE2E  |.  E8 AD49F3FF   call    004047E0  
004CFE33  |.  8B45 F0       mov     eax, dword ptr [ebp-10]  
004CFE36  |.  F7EB          imul    ebx                             ; 长字符串位数乘以累加结果  
004CFE38  |.  6BF0 73       imul    esi, eax, 73                   ; 再乘以 73H  
004CFE3B  |.  0375 FC       add     esi, dword ptr [ebp-4]         ; 结果又加上常数 1C8（“456”）  
004CFE3E  |.  8D55 E4       lea     edx, dword ptr [ebp-1C]  
004CFE41  |.  8BC6          mov     eax, esi  
004CFE43  |.  E8 FC97F3FF   call    00409644                       ; 转成 10 进制字符串函数。  
004CFE48  |.  8B55 E4       mov     edx, dword ptr [ebp-1C]        ; 真码的第三部分出现了  
004CFE4B  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
004CFE4E  |.  E8 D548F3FF   call    00404728                       ; 连接字符串  
004CFE53  |>  8B45 08       mov     eax, dword ptr [ebp+8]  
004CFE56  |.  8B55 F4       mov     edx, dword ptr [ebp-C]         ; 前面的三个部分连接后。真码出现了。《《《《  
004CFE59  |.  E8 5646F3FF   call    004044B4  
004CFE5E  |.  33C0          xor     eax, eax  
004CFE60  |.  5A            pop     edx  
004CFE61  |.  59            pop     ecx  
004CFE62  |.  59            pop     ecx  
004CFE63  |.  64:8910       mov     dword ptr fs:[eax], edx  
004CFE66  |.  68 8DFE4C00   push    004CFE8D  
004CFE6B  |>  8D45 E4       lea     eax, dword ptr [ebp-1C]  
004CFE6E  |.  BA 03000000   mov     edx, 3  
004CFE73  |.  E8 0C46F3FF   call    00404484  
004CFE78  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
004CFE7B  |.  BA 02000000   mov     edx, 2  
004CFE80  |.  E8 FF45F3FF   call    00404484  
004CFE85  \.  C3            retn  
```

## 算法逻辑（类 Python/伪代码）

以下代码块对应上述汇编逻辑的高层描述，展示了注册码各部分的计算过程。

```python
# 版本 2
# 程序集 窗口程序集 1

# 子程序 _ 按钮 1_ 被单击

# 局部变量
name = ""       # 文本型
jiqima = ""     # 文本型
len = 0         # 整数型
eax = 0         # 整数型
i = 0           # 整数型
stringchang = ""# 文本型
sum = 0         # 整数型
code1 = 0       # 整数型
a = 0           # 整数型
sn1 = ""        # 文本型
code2 = 0       # 整数型
sn2 = ""        # 文本型
code3 = 0       # 整数型
sn3 = ""        # 文本型

# 获取输入
name = 编辑框_1.内容
jiqima = 编辑框_2.内容

# 组合字符串
stringchang = name + jiqima
len = len(stringchang)

# 计算字符累加和 (sum)
# 计次循环首 (len, i)
for i in range(len):
    eax = ord(stringchang[i-1])  # 取代码
    sum = sum + eax
# 计次循环尾

# 第一部分计算
code1 = len * 345
a = sum * 73
code1 = code1 + a
sn1 = str(code1)

# 第二部分计算
code2 = sum * 345
code2 = code2 * 21
sn2 = str(code2)

# 第三部分计算
code3 = len * sum
code3 = code3 * 115 + 345
sn3 = str(code3)

# 组合最终注册码
编辑框_3.内容 = sn1 + "-" + sn2 + "-" + sn3

--------------------------------------------------------
```

## 破解总结

1.  **注册码逻辑异常**：作者在这个软件总共计算了 `4` 组类似注册码的字符串，但是只有最后一组是能注册成功的。原因未知，可能存在暗桩。且该结果与机器码完全没有关系。
2.  **算法常数**：每次（`4` 次）算法由一个常数贯穿整个算法。它们分别是 `"456"`、`"46"`、`"56"`、`"345"`。其中 `"345"` 这个常数的计算成为了最终真码。
3.  **结构示例**：算法分三个部分，如：`90459-8498385-1888875`。逻辑比较简单。

## 注册信息存储位置

注册信息存储在以下注册表路径中：

`HKEY_LOCAL_MACHINE\SOFTWARE\lsjsoft\spa\SerialNo`

此外，还识别到以下哈希值：

`4dd466d0e1e0cc123afbe504da2462ca`
