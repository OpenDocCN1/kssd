
# （BASE64）冰盾系统安全专家 V3.3 破解教程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-254.htm`](https://www.pediy.com/kssd/pediy07/pediy7-254.htm)

【破解作者】 `kyc\[dfcg\]\[czg\]`
【作者邮箱】 `muyang008@163.com`
【使用工具】 `FLYold1.10c`
【破解平台】 `win2003`
【软件名称】 冰盾系统安全专家 `V3.3`

软件大小： `2582` `KB`
软件语言： 简体中文
软件类别： 国产软件 / 共享版 / 系统安全
应用平台： `Win9x/NT/2000/XP`
软件介绍：嵌入式内核技术全面替代硬盘保护卡领跑系统安全软件。本软件可以极其有效的防止病毒、木马及其他恶意代码或人为对系统造成的不同程度的危害，真正实现注册表内容防删除、防篡改，文件防删、硬盘防格式化及其它众多系统安全功能，如：文件加密、隐藏、黑客软件扫描、病毒预警、系统敏感功能限制等 `40` 余项系统底层功能，可全面深层的保护您的计算机免受侵害！
下载地址：`http://www.skycn.com/soft/2992.html`

【加壳方式】 无壳
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------
【破解内容】`vc` 编写保护方式 `BASE64` 加密，但注册码以明文出现，如果觉得简单高手就略过吧。

```
00409A32   |.   68 FFF24200        push Registry.0042F2FF                 ;  SE handler installation
00409A37   |.   64:A1 00000000     mov eax,dword ptr fs:[0]
00409A3D   |.   50                 push eax                                 ;  因为使用了 BASE64 加密算法利用 PEID 显示的地址下断
00409A3E   |.   64:8925 00000000   mov dword ptr fs:[0],esp
00409A45   |.   83EC 30            sub esp,30
00409A48   |.   56                 push esi
00409A49   |.   8BF1              mov esi,ecx
00409A4B   |.   C74424 10 00000000 mov dword ptr ss:[esp+10],0
00409A53   |.   A1 F0CB4300        mov eax,dword ptr ds:[43CBF0]
00409A58   |.   C74424 3C 01000000 mov dword ptr ss:[esp+3C],1
00409A60   |.   894424 04          mov dword ptr ss:[esp+4],eax
00409A64   |.   8D4C24 48          lea ecx,dword ptr ss:[esp+48]               ;  ecx=机器码
00409A68   |.   C64424 3C 02       mov byte ptr ss:[esp+3C],2
00409A6D   |.   51                 push ecx
00409A6E   |.   8D4C24 10          lea ecx,dword ptr ss:[esp+10]
00409A72   |.   E8 02CB0100        call Registry.00426579
00409A77   |.   8D5424 0C          lea edx,dword ptr ss:[esp+C]               ;  eDx=机器码
00409A7B   |.   8D4C24 18          lea ecx,dword ptr ss:[esp+18]
00409A7F   |.   52                 push edx                                   ;  eDx=机器码
00409A80   |.   6A 01              push 1
00409A82   |.   6A 00              push 0
00409A84   |.   C64424 48 03       mov byte ptr ss:[esp+48],3
00409A89   |.   E8 E298FFFF        call Registry.00403370                     ;  f7 base64 加密
```

`****************************************************************`

```
00403370   /$   8B4424 0C          mov eax,dword ptr ss:[esp+C]
00403374   |.   56                 push esi
00403375   |.   8BF1              mov esi,ecx
00403377   |.   8B5424 0C          mov edx,dword ptr ss:[esp+C]
0040337B   |.   C706 58174300      mov dword ptr ds:[esi],Registry.00431758
00403381   |.   8B00              mov eax,dword ptr ds:[eax]                   ;  机器码
00403383   |.   8B48 F8           mov ecx,dword ptr ds:[eax-8]                 ;  机器码长度
00403386   |.   51                 push ecx
00403387   |.   50                 push eax
00403388   |.   8B4424 10          mov eax,dword ptr ss:[esp+10]
0040338C   |.   52                 push edx
0040338D   |.   50                 push eax
0040338E   |.   8BCE              mov ecx,esi
00403390   |.   E8 6B000000        call Registry.00403400                     ******f7 base64 加密
```

`////////////////////////////////////////////////////////////////////////////////////////////////////////`

```
00403400   /$   64:A1 00000000     mov eax,dword ptr fs:[0]
00403406   |.   6A FF             push -1
00403408   |.   68 A6EC4200        push Registry.0042ECA6
0040340D   |.   50                 push eax
0040340E   |.   8B4424 14          mov eax,dword ptr ss:[esp+14]
00403412   |.   64:8925 00000000   mov dword ptr fs:[0],esp
00403419   |.   53                 push ebx
0040341A   |.   56                 push esi
0040341B   |.   8BF1              mov esi,ecx
0040341D   |.   57                 push edi
0040341E   |.   85C0              test eax,eax
00403420   |.   8D7E 08           lea edi,dword ptr ds:[esi+8]
00403423   |.   8D5E 0C           lea ebx,dword ptr ds:[esi+C]
00403426   |.   C707 00000000      mov dword ptr ds:[edi],0
0040342C   |.   C703 00000000      mov dword ptr ds:[ebx],0
00403432   |.   75 23             jnz short Registry.00403457
00403434   |.   6A 04             push 4
00403436   |.   E8 0A310200        call Registry.00426545
0040343B   |.   83C4 04           add esp,4
0040343E   |.   894424 20         mov dword ptr ss:[esp+20],eax
00403442   |.   85C0              test eax,eax
00403444   |.   C74424 14 00000000 mov dword ptr ss:[esp+14],0
0040344C   |.   74 31             je short Registry.0040347F
0040344E   |.   8BC8              mov ecx,eax
00403450   |.   E8 1BFCFFFF        call Registry.00403070
00403455   |.   EB 2A             jmp short Registry.00403481
00403457   |>   83F8 01           cmp eax,1
0040345A   |.   75 30             jnz short Registry.0040348C
0040345C   |.   6A 60             push 60
0040345E   |.   E8 E2300200        call Registry.00426545
00403463   |.   83C4 04           add esp,4
00403466   |.   894424 20         mov dword ptr ss:[esp+20],eax
0040346A   |.   85C0              test eax,eax
0040346C   |.   C74424 14 01000000 mov dword ptr ss:[esp+14],1
00403474   |.   74 09             je short Registry.0040347F
00403476   |.   8BC8              mov ecx,eax
00403478   |.   E8 C3F2FFFF        call Registry.00402740                     ;  BASE64 加密
```

`///////////////////////////////////////////////////////////////////////////////////////`

```
00402740   /$   6A FF             push -1
00402742   |.   68 63EC4200        push Registry.0042EC63                     ;  SE handler installation
00402747   |.   64:A1 00000000     mov eax,dword ptr fs:[0]
0040274D   |.   50                 push eax
0040274E   |.   64:8925 00000000   mov dword ptr fs:[0],esp
00402755   |.   83EC 54            sub esp,54
00402758   |.   53                 push ebx
00402759   |.   55                 push ebp
0040275A   |.   56                 push esi
0040275B   |.   8BE9              mov ebp,ecx
0040275D   |.   57                 push edi
0040275E   |.   896C24 1C         mov dword ptr ss:[esp+1C],ebp
00402762   |.   E8 09090000        call Registry.00403070
00402767   |.   33C0              xor eax,eax
00402769   |.   C745 44 44174300   mov dword ptr ss:[ebp+44],Registry.00431744  ;  ASCII "alB"
00402770   |.   894424 6C         mov dword ptr ss:[esp+6C],eax
00402774   |.   8945 48           mov dword ptr ss:[ebp+48],eax
00402777   |.   C745 4C 11000000   mov dword ptr ss:[ebp+4C],11
0040277E   |.   8945 50           mov dword ptr ss:[ebp+50],eax
00402781   |.   8945 54           mov dword ptr ss:[ebp+54],eax
00402784   |.   8945 58           mov dword ptr ss:[ebp+58],eax
00402787   |.   C745 5C 0A000000   mov dword ptr ss:[ebp+5C],0A
0040278E   |.   B9 10000000        mov ecx,10
00402793   |.   BE 30C14300        mov esi,Registry.0043C130                   ;  ASCII "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
00402798   |.   8D7C24 20          lea edi,dword ptr ss:[esp+20]
0040279C   |.   C745 00 3C174300   mov dword ptr ss:[ebp],Registry.0043173C
004027A3   |.   F3:A5             rep movs dword ptr es:[edi],dword ptr ds:[esi]
004027A5   |.   894424 14         mov dword ptr ss:[esp+14],eax
004027A9   |.   8BC5              mov eax,ebp
004027AB   |.   8D4C24 20          lea ecx,dword ptr ss:[esp+20]
004027AF   |.   C64424 6C 01       mov byte ptr ss:[esp+6C],1
004027B4   |.   2BC1              sub eax,ecx
```



## Disassembly Listing

```
004027B6   |.  83C0 04            add eax,4  

004027B9   |.  A4                 movs byte ptr es:[edi],byte ptr ds:[esi]  

004027BA   |.  894424 18          mov dword ptr ss:[esp+18],eax  

004027BE   |>  8B5424 14          /mov edx,dword ptr ss:[esp+14]  

004027C2   |.  8A4C14 20          |mov cl,byte ptr ss:[esp+edx+20]  

004027C6   |.  8D4414 20          |lea eax,dword ptr ss:[esp+edx+20]  

004027CA   |.  8B5424 18          |mov edx,dword ptr ss:[esp+18]  

004027CE   |.  884C24 10          |mov byte ptr ss:[esp+10],cl  

004027D2   |.  880C02             |mov byte ptr ds:[edx+eax],cl  

004027D5   |.  8B4424 10          |mov eax,dword ptr ss:[esp+10]  

004027D9   |.  8B75 4C            |mov esi,dword ptr ss:[ebp+4C]  

004027DC   |.  25 FF000000        |and eax,0FF  

004027E1   |.  C1E8 04            |shr eax,4  

004027E4   |.  33D2               |xor edx,edx  

004027E6   |.  8B4D 48            |mov ecx,dword ptr ss:[ebp+48]  

004027E9   |.  F7F6               |div esi  

004027EB   |.  85C9               |test ecx,ecx  

004027ED   |.  8BDA               |mov ebx,edx  

004027EF   |.  74 26              |je short Registry.00402817  

004027F1   |.  8B0499             |mov eax,dword ptr ds:[ecx+ebx*4]  

004027F4   |.  85C0               |test eax,eax  

004027F6   |.  74 1B              |je short Registry.00402813  

004027F8   |>  8A50 08            |/mov dl,byte ptr ds:[eax+8]  

004027FB   |.  3A5424 10          ||cmp dl,byte ptr ss:[esp+10]  

004027FF   |.  74 08              ||je short Registry.00402809  

00402801   |.  8B00               ||mov eax,dword ptr ds:[eax]  

00402803   |.  85C0               ||test eax,eax  

00402805   |.^ 75 F1              |\jnz short Registry.004027F8  

00402807   |.  EB 0A              |jmp short Registry.00402813  

00402809   |>  85C0               |test eax,eax  

0040280B   |.  8BF8               |mov edi,eax  

0040280D   |.  0F85 98000000     |jnz Registry.004028AB  

00402813   |>  85C9               |test ecx,ecx  

00402815   |.  75 2C              |jnz short Registry.00402843  

00402817   |>  8D3CB5 00000000    |lea edi,dword ptr ds:[esi*4]  

0040281E   |.  57                 |push edi  

0040281F   |.  E8 213D0200        |call Registry.00426545  

00402824   |.  8BD0               |mov edx,eax  

00402826   |.  8BCF               |mov ecx,edi  

00402828   |.  8955 48            |mov dword ptr ss:[ebp+48],edx  

0040282B   |.  8BFA               |mov edi,edx  

0040282D   |.  8BD1               |mov edx,ecx  

0040282F   |.  33C0               |xor eax,eax  

00402831   |.  C1E9 02            |shr ecx,2  

00402834   |.  F3:AB              |rep stos dword ptr es:[edi]  

00402836   |.  8BCA               |mov ecx,edx  

00402838   |.  83C4 04            |add esp,4  

0040283B   |.  83E1 03            |and ecx,3  

0040283E   |.  F3:AA              |rep stos byte ptr es:[edi]  

00402840   |.  8975 4C            |mov dword ptr ss:[ebp+4C],esi  

00402843   |>  8B45 54            |mov eax,dword ptr ss:[ebp+54]  

00402846   |.  85C0               |test eax,eax  

00402848   |.  75 30              |jnz short Registry.0040287A  

0040284A   |.  8B45 5C            |mov eax,dword ptr ss:[ebp+5C]  

0040284D   |.  6A 0C              |push 0C  

0040284F   |.  8D4D 58            |lea ecx,dword ptr ss:[ebp+58]  

00402852   |.  50                 |push eax  

00402853   |.  51                 |push ecx  

00402854   |.  E8 90000200        |call Registry.004228E9  

00402859   |.  8B4D 5C            |mov ecx,dword ptr ss:[ebp+5C]  

0040285C   |.  83C0 04            |add eax,4  

0040285F   |.  8D1449             |lea edx,dword ptr ds:[ecx+ecx*2]  

00402862   |.  49                 |dec ecx  

00402863   |.  85C9               |test ecx,ecx  

00402865   |.  8D4490 F4          |lea eax,dword ptr ds:[eax+edx*4-C]  

00402869   |.  7C 0F              |jl short Registry.0040287A  

0040286B   |.  41                 |inc ecx  

0040286C   |>  8B55 54            |/mov edx,dword ptr ss:[ebp+54]  

0040286F   |.  8910               ||mov dword ptr ds:[eax],edx  

00402871   |.  8945 54            ||mov dword ptr ss:[ebp+54],eax  

00402874   |.  83E8 0C            ||sub eax,0C  

00402877   |.  49                 ||dec ecx  

00402878   |.^ 75 F2              |\jnz short Registry.0040286C  

0040287A   |>  8B45 54            |mov eax,dword ptr ss:[ebp+54]  

0040287D   |.  8A5424 10          |mov dl,byte ptr ss:[esp+10]  

00402881   |.  8BF8               |mov edi,eax  

00402883   |.  8B08               |mov ecx,dword ptr ds:[eax]  

00402885   |.  894D 54            |mov dword ptr ss:[ebp+54],ecx  

00402888   |.  8B4D 50            |mov ecx,dword ptr ss:[ebp+50]  

0040288B   |.  41                 |inc ecx  

0040288C   |.  894D 50            |mov dword ptr ss:[ebp+50],ecx  

0040288F   |.  C640 08 00         |mov byte ptr ds:[eax+8],0  

00402893   |.  C640 09 00         |mov byte ptr ds:[eax+9],0  

00402897   |.  8958 04            |mov dword ptr ds:[eax+4],ebx  

0040289A   |.  8850 08            |mov byte ptr ds:[eax+8],dl  

0040289D   |.  8B4D 48            |mov ecx,dword ptr ss:[ebp+48]  

004028A0   |.  8B1499             |mov edx,dword ptr ds:[ecx+ebx*4]  

004028A3   |.  8910               |mov dword ptr ds:[eax],edx  

004028A5   |.  8B4D 48            |mov ecx,dword ptr ss:[ebp+48]  

004028A8   |.  890499             |mov dword ptr ds:[ecx+ebx*4],eax  

004028AB   |>  8B4424 14          |mov eax,dword ptr ss:[esp+14]  

004028AF   |.  8847 09            |mov byte ptr ds:[edi+9],al  

004028B2   |.  40                 |inc eax  

004028B3   |.  83F8 40            |cmp eax,40  

004028B6   |.  894424 14          |mov dword ptr ss:[esp+14],eax  

004028BA   |.^ 0F8C FEFEFFFF     \jl Registry.004027BE  

004028C0   |.  8B4C24 64          mov ecx,dword ptr ss:[esp+64]  

004028C4   |.  5F                 pop edi  

004028C5   |.  8BC5               mov eax,ebp  

004028C7   |.  5E                 pop esi  

004028C8   |.  5D                 pop ebp  

004028C9   |.  5B                 pop ebx  

004028CA   |.  64:890D 00000000   mov dword ptr fs:[0],ecx  

004028D1   |.  83C4 60            add esp,60  
```

```
///////////////////////////////////////////////////////////////////////////  

00409A8E   |.  8D4C24 18          lea ecx,dword ptr ss:[esp+18]  

00409A92   |.  C64424 3C 04      mov byte ptr ss:[esp+3C],4  

00409A97   |.  E8 5499FFFF        call Registry.004033F0  

00409A9C   |.  50                 push eax  

00409A9D   |.  8D4C24 08          lea ecx,dword ptr ss:[esp+8]  

00409AA1   |.  E8 A3CE0100        call Registry.00426949  

00409AA6   |.  6A 3D              push 3D  

00409AA8   |.  8D4C24 08          lea ecx,dword ptr ss:[esp+8]  

00409AAC   |.  E8 70840100        call Registry.00421F21  

00409AB1   |.  6A 2B              push 2B  

00409AB3   |.  8D4C24 08          lea ecx,dword ptr ss:[esp+8]  

00409AB7   |.  E8 65840100        call Registry.00421F21  

00409ABC   |.  6A 2F              push 2F  

00409ABE   |.  8D4C24 08          lea ecx,dword ptr ss:[esp+8]  

00409AC2   |.  E8 5A840100        call Registry.00421F21  

00409AC7   |.  51                 push ecx  

00409AC8   |.  8D4424 08          lea eax,dword ptr ss:[esp+8]  

00409ACC   |.  8BCC               mov ecx,esp  

00409ACE   |.  896424 18          mov dword ptr ss:[esp+18],esp  

00409AD2   |.  50                 push eax  

00409AD3   |.  E8 A1CA0100        call Registry.00426579  

00409AD8   |.  8D4C24 0C          lea ecx,dword ptr ss:[esp+C]                 ; |  

00409ADC   |.  51                 push ecx                                   ; |Arg1  

00409ADD   |.  8BCE               mov ecx,esi                                ; |  

00409ADF   |.  E8 8C020000        call Registry.00409D70                     ; \f7 关键算法  
```



# 逆向分析与注册机源码参考

## 汇编代码段

```
/////////////////////////////////////////////////////////////////////////////////  

00409D70   /$  6A FF                push -1  

00409D72   |.  68 57F34200          push Registry.0042F357                     ;  SE handler installation  

00409D77   |.  64:A1 00000000       mov eax,dword ptr fs:\[0\]  

00409D7D   |.  50                   push eax  

00409D7E   |.  64:8925 00000000     mov dword ptr fs:\[0\],esp  

00409D85   |.  83EC 08             sub esp,8  

00409D88   |.  55                   push ebp  

00409D89   |.  56                   push esi  

00409D8A   |.  57                   push edi  

00409D8B   |.  C74424 10 00000000 mov dword ptr ss:\[esp+10\],0  

00409D93   |.  A1 F0CB4300         mov eax,dword ptr ds:\[43CBF0\]  

00409D98   |.  C74424 1C 01000000 mov dword ptr ss:\[esp+1C\],1  

00409DA0   |.  894424 0C           mov dword ptr ss:\[esp+C\],eax  

00409DA4   |.  8B4C24 28          mov ecx,dword ptr ss:\[esp+28\]  

00409DA8   |.  C64424 1C 02       mov byte ptr ss:\[esp+1C\],2  

00409DAD   |.  8B69 F8           mov ebp,dword ptr ds:\[ecx-8\]  

00409DB0   |.  8D7D 01           lea edi,dword ptr ss:\[ebp+1\]  

00409DB3   |.  57                 push edi  

00409DB4   |.  E8 8CC70100        call Registry.00426545  

00409DB9   |.  8BCF              mov ecx,edi  

00409DBB   |.  8BF0              mov esi,eax  

00409DBD   |.  8BD1              mov edx,ecx  

00409DBF   |.  33C0              xor eax,eax  

00409DC1   |.  8BFE              mov edi,esi  

00409DC3   |.  83C4 04          add esp,4  

00409DC6   |.  C1E9 02          shr ecx,2  

00409DC9   |.  F3:AB             rep stos dword ptr es:\[edi\]  

00409DCB   |.  8BCA              mov ecx,edx  

00409DCD   |.  83E1 03          and ecx,3  

00409DD0   |.  F3:AA             rep stos byte ptr es:\[edi\]  

00409DD2   |.  33C9              xor ecx,ecx  

00409DD4   |.  85ED              test ebp,ebp  

00409DD6   |.  7E 37              jle short Registry.00409E0F  

00409DD8   |>  8B4424 28          /mov eax,dword ptr ss:\[esp+28\]  

00409DDC   |.  8A0401             |mov al,byte ptr ds:\[ecx+eax\]  

00409DDF   |.  3C 41              |cmp al,41  

00409DE1   |.  74 22              |je short Registry.00409E05                ;  如果 RES\[I\]是 41=A 或 61=a 就 RES\[I\]+=1  

00409DE3   |.  3C 61              |cmp al,61  

00409DE5   |.  74 1E              |je short Registry.00409E05  

00409DE7   |.  3C 5A              |cmp al,5A                                 ;  如果 RES\[I\]是 5A=Z 或 7A=z 就 RES\[I\]-=1  

00409DE9   |.  74 16              |je short Registry.00409E01  

00409DEB   |.  3C 7A              |cmp al,7A  

00409DED   |.  74 12              |je short Registry.00409E01  

00409DEF   |.  0FBED0             |movsx edx,al  

00409DF2   |.  81E2 01000080      |and edx,80000001                         ;  res\[i\] and 80000001  

00409DF8   |.  79 05              |jns short Registry.00409DFF  

00409DFA   |.  4A                  |dec edx  

00409DFB   |.  83CA FE            |or edx,FFFFFFFE  

00409DFE   |.  42                  |inc edx  

00409DFF   |>  74 04              |je short Registry.00409E05                ;  res\[i\] and 80000001 等于 0 就 res\[i\]+1  

00409E01   |>  FEC8               |dec al                                   ;  res\[i\] and 80000001 不等于 0 就 res\[i\]-1  

00409E03   |.  EB 02              |jmp short Registry.00409E07  

00409E05   |>  FEC0               |inc al  

00409E07   |>  880431             |mov byte ptr ds:\[ecx+esi\],al           ;  形成真码  

00409E0A   |.  41                  |inc ecx  

00409E0B   |.  3BCD               |cmp ecx,ebp  

00409E0D   |.^ 7C C9              \\jl short Registry.00409DD8  

00409E0F   |>  56                  push esi  

00409E10   |.  8D4C24 10          lea ecx,dword ptr ss:\[esp+10\]  

00409E14   |.  E8 30CB0100        call Registry.00426949  

00409E19   |.  56                  push esi  

00409E1A   |.  E8 4FC70100        call Registry.0042656E  

00409E1F   |.  8B7424 28          mov esi,dword ptr ss:\[esp+28\]  

00409E23   |.  83C4 04          add esp,4  

00409E26   |.  8D4424 0C          lea eax,dword ptr ss:\[esp+C\]  

00409E2A   |.  8BCE               mov ecx,esi  

////////////////////////////////////////////////////////////////////////////////////////////////////  
```

## 总结

注册方式为对机器码进行 `BASE64` 加密后对其结果进行了一些处理形成注册码，详见注册机 `C` 源码。

`BASE64C` 代码参考了网上不知哪位大虾编的。

## C 源码参考

```c
#include  <stdio.h>   

#include  <stdlib.h>   

#include  <string.h>   

#include  <conio.h>  /*  Windows  special  */   

char*  alphabet  =  "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";   

char*  encode(char  data[],  int  length)   

{   

  int  i,  index,  val;   

  int  quad,  trip;    /*  bool  type:  0  or  1  */   

   char*  out;   

    //out = (char*)malloc((((length+2)/3)\*4)\*sizeof(char));   

    out=new char\[(((length+2)/3)\*4)\*sizeof(char)\];  

  for  (i=0,  index=0;  i<length;  i+=3,  index+=4)   

  {   

               quad  =  0;   

               trip  =  0;                  

        val=(0xFF  &  (int)  data\[i\]);   

               val<<=8;   

               if((i+1)<length)   

        {   

                       val|=(0xFF&(int)data\[i+1\]);   

                       trip  = 1;   

               }   

               val<<=8;   

               if  ((i+2)<length)   

        {   

                       val|=(0xFF&(int)data\[i+2\]);   

                       quad  = 1;   

               }   

               out\[index+3\]  =  alphabet\[(quad  ?(val&0x3F):64)\];   

               val>>=6;   

               out\[index+2\]  =  alphabet\[(trip?(val&0x3F):64)\];   

               val>>=6;   

               out\[index+1\]  =  alphabet\[val&0x3F\];   

               val>>=6;   

               out\[index+0\]  =  alphabet\[val&0x3F\];   

       }   

  out\[index\]  =  '\\0';   

       return  out;   

}   

void  init_codes(char  codes[])   

{   

 int  i;   

 for  (i =0; i< 256; i++)   

     codes\[i\]  = -1;   

       for  (i = 'A'; i <= 'Z'; i++)   

       codes\[i\] = i -'A';   

       for  (i='a'; i<='z';i++)   

       codes\[i\] = 26  + i- 'a';   

       for  (i='0';i<='9'; i++)   

       codes\[i\]= 52+i-'0';   

       codes\['+'\] = 62;   

       codes\['/'\] = 63;   

}   

char*  decode(char  data[],int  length)   

{   

  int  value,  ix;   

       int  shift  =  0;      /*  #  of  excess  bits  stored  in  accum  */   

       int  accum  =  0;      /*  excess  bits  */   

       int  index  =  0;   

       int  len;   

     char  codes\[256\];   

     char*  out;   

     len=((length  + 3)/4)\*3;   

       if(length>0 && data\[length-1\] =='=')   

       --len;   

       if  (length>1&&data\[length-2\] =='=')   

       --len;   

     printf("%d\\n",sizeof(char));   

     //out  =  (char*)malloc(len  *  sizeof(char));   

        out=new char\[len\*sizeof(char)\];  

     init_codes(codes);   

       for  (ix=0;  ix<length;  ix++)   

     {   

               value = codes\[data\[ix\] & 0xFF\];  /*  ignore  high  byte  of  char  */   

               if  (  value  >=  0  )   

         {          /*  skip  over  non-code  */   

                       accum  <<= 6;                    /*  bits  shift  up  by  6  each  time  thru  */   

                       shift  +=  6;                   /*  loop,  with  new  bits  being  put  in  */   

                       accum  |=  value;               /*  at  the  bottom.  */   

                       if  (  shift  >=  8  )   

         {          /*  whenever  there  are  8  or  more  shifted  in,  */   

                       shift  -=  8;                  /*  write  them  out  (from  the  top,  leaving  any  */   
```



```
    out[index++]  =  ((accum  >>  shift)  &  0xff);      /*  excess  at  the  bottom  for  next  iteration.  */  
    }  
  }  
 }  
    out[index]  =  '\0';  
    if  (index  !=  len)  
      printf("miscalculated  data  length!\n");  
 return  out;  
}  
char*  fu1(char  *data,  int  length)  
{  
  int    index;  
   char*  out1;  
    out1=new char[length*sizeof(char)];  
   strcpy(out1,data);  
  for  (index=0;  index<length; index++)  
  {  
    if(out1[index]=='a'||out1[index]=='A')  
        out1[index]=out1[index]+1;  
    else if(out1[index]=='Z'||out1[index]=='z')  
            out1[index]=out1[index]-1;  
    else if((out1[index]&0x80000001)==0)  
            out1[index]=out1[index]+1;  
    else out1[index]=out1[index]-1;  
      }  
  out1[index-1]  =  '\0';  
      return  out1;  
}  
void   main()  
{  
  char data1[255]={0},data2[255]={0};  
printf("请输入机器码： \n");  
  scanf( "%s",  &data2 );  
 char*  out1;  
 char*  out2;  
 char*  res;  
 out2  =  encode(data2,strlen(data2)); //加密  
 res   =  fu1(out2,strlen(out2));      //对加密结果简单处理形成真正注册码  
 out1  =  decode(out2,strlen(out2));  //解密  
 printf("%s\n%s\n",out2,out1);  
 printf("您的注册码是：\n%s\n",res);  
 system("pause");  /*  Windows  special  */  
}  
```

注册后虽然出现注册成功，但是，还是没有真正注册上。可能还有什么暗桩 但是，没有发现。

不过通过破解学习了 `BASE64` 加密解密算法，要不然真难以去理解枯燥的算法。

失误或得罪之处多多包涵。

后记：

注册后在 `WINDOWS` 目录下生成 `REGSUCCESS.ISD` 文件

具体格式如下：

`Copyright by He.Yu.Peng`

机器码

注册码

机器码之和加取注册码第 `LEN`（机器码）位*`LEN`（注册码）

继续跟踪了根目录下的 `ISEncrypter.exe` 未注册只能加 `5KB` 以下的软件，但是跟踪发现注册成功与否只能加壳 `5KB` 以下的软件

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
00401517    .  6A 04                 push 4
00401519    .  51                   push ecx
0040151A    .  8BCF                 mov ecx,edi
0040151C    .  E8 10950200          call ISEncryp.0042AA31
00401521    .  8B5424 10            mov edx,dword ptr ss:[esp+10]
00401525    .  68 3C824400          push ISEncryp.0044823C                       ;  ASCII ".EXE"
0040152A    .  52                   push edx
0040152B    .  C78424 48020000 00>mov dword ptr ss:[esp+248],0
00401536    .  E8 697C0100          call ISEncryp.004191A4
0040153B    .  83C4 08              add esp,8
0040153E    .  85C0                 test eax,eax
00401540    .  74 13                 je short ISEncryp.00401555
00401542    .  6A 00                 push 0                                     ; /Arg3 = 00000000
00401544    .  6A 00                 push 0                                     ; |Arg2 = 00000000
00401546    .  68 1C824400           push ISEncryp.0044821C                     ; |Arg1 = 0044821C
0040154B    .  E8 632A0300           call ISEncryp.00433FB3                     ; \ISEncryp.00433FB3
00401550    .  E9 6C010000           jmp ISEncryp.004016C1
00401555    >  A1 948C4400           mov eax,dword ptr ds:[448C94]
0040155A    .  894424 14             mov dword ptr ss:[esp+14],eax
0040155E    .  8D4C24 1C             lea ecx,dword ptr ss:[esp+1C]
00401562    .  C68424 40020000 01    mov byte ptr ss:[esp+240],1
0040156A    .  51                     push ecx                                 ; /Arg1
0040156B    .  8BCE                   mov ecx,esi                              ; |
0040156D    .  E8 8E010000            call ISEncryp.00401700                    ; \ISEncryp.00401700
00401572    .  8D5424 1C             lea edx,dword ptr ss:[esp+1C]
00401576    .  68 10824400           push ISEncryp.00448210                     ;  ASCII "ISHP.DLL"
0040157B    .  52                     push edx
0040157C    .  8D4424 1C             lea eax,dword ptr ss:[esp+1C]
00401580    .  68 08824400           push ISEncryp.00448208                     ;  ASCII "%s%s"
00401585    .  50                     push eax
00401586    .  E8 A2980200           call ISEncryp.0042AE2D
0040158B    .  83C4 10               add esp,10
0040158E    .  8D4C24 0C             lea ecx,dword ptr ss:[esp+C]
00401592    .  57                     push edi                                 ; /Arg2
00401593    .  51                     push ecx                                 ; |Arg1
00401594    .  8BCE                   mov ecx,esi                              ; |
00401596    .  E8 55020000            call ISEncryp.004017F0                    ; \ISEncryp.004017F0
0040159B    .  8D5424 0C             lea edx,dword ptr ss:[esp+C]
0040159F    .  68 FC814400           push ISEncryp.004481FC                     ;  ASCII "_New.exe"
004015A4    .  8D4424 1C             lea eax,dword ptr ss:[esp+1C]
004015A8    .  52                     push edx
004015A9    .  50                     push eax
004015AA    .  C68424 4C020000 02    mov byte ptr ss:[esp+24C],2
004015B2    .  E8 09EE0200           call ISEncryp.004303C0
004015B7    .  8B08                   mov ecx,dword ptr ds:[eax]
004015B9    .  8D5424 0C             lea edx,dword ptr ss:[esp+C]
004015BD    .  51                     push ecx
004015BE    .  68 F8814400           push ISEncryp.004481F8                     ;  ASCII "%s"
004015C3    .  52                     push edx
004015C4    .  C68424 4C020000 03    mov byte ptr ss:[esp+24C],3
004015CC    .  E8 5C980200           call ISEncryp.0042AE2D
004015D1    .  83C4 0C               add esp,0C
004015D4    .  8D4C24 18             lea ecx,dword ptr ss:[esp+18]
004015D8    .  C68424 40020000 02    mov byte ptr ss:[esp+240],2
004015E0    .  E8 46EB0200           call ISEncryp.0043012B
004015E5    .  68 E4814400           push ISEncryp.004481E4
004015EA    .  68 10040000           push 410
004015EF    .  8BCE                   mov ecx,esi
004015F1    .  E8 A7E00200           call ISEncryp.0042F69D
004015F6    .  8BC8                   mov ecx,eax
004015F8    .  E8 90E10200           call ISEncryp.0042F78D
004015FD    .  8BCE                   mov ecx,esi
004015FF    .  E8 BC0B0000           call ISEncryp.004021C0                     ;  关键算法
```

/////////////////////////////////////////////////////////////////////////////////////////////////////////////

```
004021C0   /$  6A FF                   push -1
004021C2   |.  68 0B874300            push ISEncryp.0043870B                     ;  SE  handler  installation
004021C7   |.  64:A1 00000000        mov eax,dword ptr fs:[0]
004021CD   |.  50                     push eax
004021CE   |.  64:8925 00000000       mov dword ptr fs:[0],esp
004021D5   |.  81EC 98020000          sub esp,298
004021DB   |.  53                     push ebx
004021DC   |.  55                     push ebp
004021DD   |.  56                     push esi
004021DE   |.  57                     push edi
004021DF   |.  B9 61000000            mov ecx,61
004021E4   |.  33C0                   xor eax,eax
004021E6   |.  8D7C24 20             lea edi,dword ptr ss:[esp+20]
004021EA   |.  68 04010000            push 104                                   ; /BufSize = 104 (260.)
004021EF   |.  F3:AB                  rep  stos dword ptr es:[edi]               ; |
004021F1   |.  8D8424 A8010000        lea eax,dword ptr ss:[esp+1A8]              ; |查找系统目录是否存在 REGSUCCESS.ISD
004021F8   |.  33ED                   xor ebp,ebp                                ; |
004021FA   |.  50                     push eax                                   ; |Buffer
004021FB   |.  33DB                   xor ebx,ebx                                ; |EBX=0
004021FD   |.  FF15 FCB14300          call dword ptr ds:[<&KERNEL32.GetWindowsDirect>;  \GetWindowsDirectoryA
00402203   |.  BF B8824400            mov edi,ISEncryp.004482B8                    ;  ASCII "\REGSUCCESS.ISD"
00402208   |.  83C9 FF                or ecx,FFFFFFFF
```



```asm
0040220B   |.  33C0                 xor eax,eax

0040220D   |.  8D9424 A4010000      lea edx,dword ptr ss:[esp+1A4]

00402214   |.  F2:AE                repne scas byte ptr es:[edi]

00402216   |.  F7D1                 not ecx

00402218   |.  2BF9                 sub edi,ecx

0040221A   |.  8BF7                 mov esi,edi

0040221C   |.  8BFA                 mov edi,edx

0040221E   |.  8BD1                 mov edx,ecx

00402220   |.  83C9 FF              or ecx,FFFFFFFF

00402223   |.  F2:AE                repne scas byte ptr es:[edi]

00402225   |.  8BCA                 mov ecx,edx

00402227   |.  4F                   dec edi

00402228   |.  C1E9 02              shr ecx,2

0040222B   |.  F3:A5                rep movs dword ptr es:[edi],dword ptr ds:[esi]

0040222D   |.  8BCA                 mov ecx,edx

0040222F   |.  83E1 03              and ecx,3

00402232   |.  F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]

00402234   |.  8D4C24 10            lea ecx,dword ptr ss:[esp+10]

00402238   |.  E8 F8E60200          call ISEncryp.00430935

0040223D   |.  33F6                 xor esi,esi

0040223F   |.  8D8424 A4010000      lea eax,dword ptr ss:[esp+1A4]

00402246   |.  56                   push esi                               ; /Arg3 => 00000000

00402247   |.  68 00800000          push 8000                              ; |Arg2 = 00008000

0040224C   |.  50                   push eax                               ; |Arg1

0040224D   |.  8D4C24 1C            lea ecx,dword ptr ss:[esp+1C]          ; |

00402251   |.  89B424 BC020000      mov dword ptr ss:[esp+2BC],esi         ; |

00402258   |.  E8 EAE70200          call ISEncryp.00430A47                 ; \\ISEncryp.00430A47

0040225D   |.  85C0                 test eax,eax

0040225F   |.  0F84 92000000        je ISEncryp.004022F7

00402265   |.  8D4C24 20            lea ecx,dword ptr ss:[esp+20]

00402269   |.  68 84010000          push 184                               ; /Arg2 = 00000184

0040226E   |.  51                   push ecx                               ; |Arg1

0040226F   |.  8D4C24 18            lea ecx,dword ptr ss:[esp+18]          ; |

00402273   |.  E8 ECE80200          call ISEncryp.00430B64                 ; \\ISEncryp.00430B64

00402278   |.  8DBC24 A0000000      lea edi,dword ptr ss:[esp+A0]          ;  机器码

0040227F   |.  83C9 FF              or ecx,FFFFFFFF

00402282   |.  33C0                 xor eax,eax

00402284   |.  33D2                 xor edx,edx

00402286   |.  F2:AE                repne scas byte ptr es:[edi]

00402288   |.  F7D1                 not ecx

0040228A   |.  49                   dec ecx                                ;  ecx=机器码长度

0040228B   |.  74 20                je short ISEncryp.004022AD

0040228D   |>  0FBE8414 A0000000    /movsx eax,byte ptr ss:[esp+edx+A0]

00402295   |.  03D8                 |add ebx,eax                          ;  机器码按位相加

00402297   |.  8DBC24 A0000000      |lea edi,dword ptr ss:[esp+A0]

0040229E   |.  83C9 FF              |or ecx,FFFFFFFF

004022A1   |.  33C0                 |xor eax,eax

004022A3   |.  42                   |inc edx

004022A4   |.  F2:AE                |repne scas byte ptr es:[edi]

004022A6   |.  F7D1                 |not ecx

004022A8   |.  49                   |dec ecx

004022A9   |.  3BD1                 |cmp edx,ecx

004022AB   |.^ 72 E0                \\jb short ISEncryp.0040228D

004022AD   |>  8DBC24 20010000      lea edi,dword ptr ss:[esp+120]         ;  注册码

004022B4   |.  83C9 FF              or ecx,FFFFFFFF

004022B7   |.  33C0                 xor eax,eax

004022B9   |.  F2:AE                repne scas byte ptr es:[edi]

004022BB   |.  F7D1                 not ecx

004022BD   |.  49                   dec ecx                                ;  注册码长度

004022BE   |.  74 20                je short ISEncryp.004022E0

004022C0   |.  0FBE9414 20010000    movsx edx,byte ptr ss:[esp+edx+120]    ;  EDX=取注册码第 LEN（机器码）位

004022C8   |>  03DA                 /add ebx,edx                          ;  相加

004022CA   |.  8DBC24 20010000      |lea edi,dword ptr ss:[esp+120]

004022D1   |.  83C9 FF              |or ecx,FFFFFFFF

004022D4   |.  33C0                 |xor eax,eax

004022D6   |.  46                   |inc esi

004022D7   |.  F2:AE                |repne scas byte ptr es:[edi]

004022D9   |.  F7D1                 |not ecx

004022DB   |.  49                   |dec ecx

004022DC   |.  3BF1                 |cmp esi,ecx

004022DE   |.^ 72 E8                \\jb short ISEncryp.004022C8

004022E0   |>  8D4C24 10            lea ecx,dword ptr ss:[esp+10]

004022E4   |.  E8 79E90200          call ISEncryp.00430C62               ;  CloseHandle

004022E9   |.  3B9C24 A0010000      cmp ebx,dword ptr ss:[esp+1A0]         ;  结果是否相等

004022F0   |.  75 05                jnz short ISEncryp.004022F7

004022F2   |.  BD 01000000          mov ebp,1

004022F7   |>  8D4C24 10            lea ecx,dword ptr ss:[esp+10]

004022FB   |.  C78424 B0020000 FF>  mov dword ptr ss:[esp+2B0],-1

00402306   |.  E8 7FE60200          call ISEncryp.0043098A

0040230B   |.  8B8C24 A8020000      mov ecx,dword ptr ss:[esp+2A8]

00402312   |.  5F                   pop edi

00402313   |.  8BC5                 mov eax,ebp

00402315   |.  5E                   pop esi

00402316   |.  5D                   pop ebp

00402317   |.  5B                   pop ebx

00402318   |.  64:890D 00000000     mov dword ptr fs:[0],ecx

0040231F   |.  81C4 A4020000        add esp,2A4

00402325   \\.  C3                  ret

////////////////////////////////////////////////////////////////////////////////////////////////////////////

00401604     .  85C0                 test eax,eax

00401606     .  75 32                jnz short ISEncryp.0040163A          ;  关键跳转吗   ***TMD 往哪里跳都是个死胡同

00401608     .  8B07                 mov eax,dword ptr ds:[edi]

0040160A     .  8D8C24 20010000      lea ecx,dword ptr ss:[esp+120]

00401611     .  51                   push ecx                             ; /Arg2

00401612     .  50                   push eax                             ; |Arg1

00401613     .  E8 6BF20200          call ISEncryp.00430883               ; \\ISEncryp.00430883

00401618     .  81BC24 2C010000 00>  cmp dword ptr ss:[esp+12C],1400

00401623     .  7E 15                jle short ISEncryp.0040163A

00401625     .  6A 00                push 0

00401627     .  68 DC814400          push ISEncryp.004481DC

0040162C     .  68 A8814400          push ISEncryp.004481A8

00401631     .  8BCE                 mov ecx,esi

00401633     .  E8 35CF0200          call ISEncryp.0042E56D               ;  未注册版不能加 5KB 以上的软件

00401638     .  EB 65                jmp short ISEncryp.0040169F

0040163A     >  6A 01                push 1

0040163C     .  51                   push ecx

0040163D     .  8BCC                 mov ecx,esp

0040163F     .  896424 20            mov dword ptr ss:[esp+20],esp

00401643     .  55                   push ebp

00401644     .  E8 57E80200          call ISEncryp.0042FEA0

00401649     .  8D5424 14            lea edx,dword ptr ss:[esp+14]         ; |

0040164D     .  8D4424 1C            lea eax,dword ptr ss:[esp+1C]         ; |

00401651     .  52                   push edx                             ; |Arg3

00401652     .  57                   push edi                             ; |Arg2

00401653     .  50                   push eax                             ; |Arg1

00401654     .  8D8E 2C060000        lea ecx,dword ptr ds:[esi+62C]        ; |

0040165A     .  E8 91BD0000          call ISEncryp.0040D3F0               ; \\未注册版不能加 5KB 以上的软件

0040165F     .  85C0                 test eax,eax
```



`00401661    .   74 0B           je short ISEncryp.0040166E`

`00401663    .   6A 00          push 0`

`////////////////////////////////////////////////////////////////////////////////////////////////`

作者好象根本就没有提供 5KB 以上加壳算法。

`4dd466d0e1e0cc123afbe504da2462ca`
