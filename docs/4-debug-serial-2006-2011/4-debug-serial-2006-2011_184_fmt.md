
# 金秘书家庭理财 1.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-660.htm`](https://www.pediy.com/kssd/pediy08/pediy8-660.htm)

## 破文信息

- **【破文标题】**：最简单的金秘书家庭理财 1.2——MD5 算法
- **【破文作者】**：XXNB
- **【作者邮箱】**：
- **【作者主页】**：[`free.ys168.com/?binbinbin7456`](http://free.ys168.com/?binbinbin7456)
- **【破解工具】**：OD
- **【破解平台】**：xpsp2
- **【软件名称】**：金秘书家庭理财 1.2
- **【软件大小】**：1373KB
- **【原版下载】**：[`www.onlinedown.net/soft/52975.htm`](http://www.onlinedown.net/soft/52975.htm)
- **【保护方式】**：
- **【软件简介】**：金秘书家庭理财是帮助个人、家庭、小型企业理财的好帮手，能很方便的知道客户消费了多少钱，收入多少钱，现有现金多少，存款多少。能快速的满足客户的各种统计需求！
- **【破解声明】**：向大侠们学习！！！

---

## 破解过程

发现了本世纪最简单的注册算法（作者比较懒）。一开始我用 `PEID` 的插件 `Krypto ANALyzer` 就看到有 `MD5` 算法。没想到还是明码比较。

字符串搜索“您已经成为我们的正式用户!感谢您的注册,您将可以免费升级和获得我们的售后服务!” 可以来到下面下断：

```
00452870   .  6A FF         push    -1
00452872   .  68 7C674600   push    0046677C                   ; 葛 mg; SE 处理程序安装
00452877   .  64:A1 0000000>mov     eax, fs:[0]
0045287D   .  50           push    eax
0045287E   .  64:8925 00000>mov     fs:[0], esp
00452885   .  81EC 0C010000 sub     esp, 10C
0045288B   .  53           push    ebx
0045288C   .  55           push    ebp
0045288D   .  56           push    esi
0045288E   .  8BF1         mov     esi, ecx
00452890   .  57           push    edi
00452891   .  8D4C24 20    lea     ecx, [esp+20]
00452895   .  E8 108A0000  call    <jmp.&MFC42.#540>
0045289A   .  8D4C24 14    lea     ecx, [esp+14]
0045289E   .  C78424 24010>mov     dword ptr [esp+124], 0
004528A9   .  E8 FC890000  call    <jmp.&MFC42.#540>
004528AE   .  8D4C24 1C    lea     ecx, [esp+1C]
004528B2   .  C68424 24010>mov     byte ptr [esp+124], 1
004528BA   .  E8 EB890000  call    <jmp.&MFC42.#540>
004528BF   .  8D4C24 10    lea     ecx, [esp+10]
004528C3   .  C68424 24010>mov     byte ptr [esp+124], 2
004528CB   .  E8 DA890000  call    <jmp.&MFC42.#540>
004528D0   .  8D4C24 18    lea     ecx, [esp+18]
004528D4   .  C68424 24010>mov     byte ptr [esp+124], 3
004528DC   .  E8 C9890000  call    <jmp.&MFC42.#540>
004528E1   .  8D4424 20    lea     eax, [esp+20]
004528E5   .  8BCE         mov     ecx, esi
004528E7   .  50           push    eax
004528E8   .  68 2B040000  push    42B
004528ED   .  C68424 2C010>mov     byte ptr [esp+12C], 4
004528F5   .  E8 2A8B0000  call    <jmp.&MFC42.#3097>
004528FA   .  8D4C24 14    lea     ecx, [esp+14]                 ; 用户名
004528FE   .  51           push    ecx
004528FF   .  68 2D040000  push    42D
00452904   .  8BCE         mov     ecx, esi
00452906   .  E8 198B0000  call    <jmp.&MFC42.#3097>
0045290B   .  8D5424 1C    lea     edx, [esp+1C]                 ; 邮箱
0045290F   .  8BCE         mov     ecx, esi
00452911   .  52           push    edx
00452912   .  68 2E040000  push    42E
00452917   .  E8 088B0000  call    <jmp.&MFC42.#3097>
0045291C   .  8D4424 10    lea     eax, [esp+10]                 ; 公司名字
00452920   .  8BCE         mov     ecx, esi
00452922   .  50           push    eax
00452923   .  68 2F040000  push    42F
00452928   .  E8 F78A0000  call    <jmp.&MFC42.#3097>
0045292D   .  8D4C24 18    lea     ecx, [esp+18]                 ; 注册日期
00452931   .  51           push    ecx
00452932   .  68 30040000  push    430
00452937   .  8BCE         mov     ecx, esi
00452939   .  E8 E68A0000  call    <jmp.&MFC42.#3097>
0045293E   .  8B5424 20    mov     edx, [esp+20]                ; 用户名
00452942   .  8B3D 74894600 mov     edi, [<&MSVCRT._mbscmp>]    ; msvcrt._mbscmp
00452948   .  68 C0E14700  push    0047E1C0                     ; /s2 = ""
0045294D   .  52           push    edx                            ; |s1
0045294E   .  FFD7         call    edi                            ; \\_mbscmp
00452950   .  83C4 08      add     esp, 8
00452953   .  85C0         test    eax, eax
00452955   .  74 4C        je      short 004529A3                 ; 用户名是否为空
00452957   .  8B4424 14    mov     eax, [esp+14]
0045295B   .  68 C0E14700  push    0047E1C0
00452960   .  50           push    eax
00452961   .  FFD7         call    edi
00452963   .  83C4 08      add     esp, 8                         ; aG7
00452966   .  85C0         test    eax, eax
00452968   .  74 39        je      short 004529A3                 ; 邮箱是否为空
0045296A   .  8B4C24 1C    mov     ecx, [esp+1C]
0045296E   .  68 C0E14700  push    0047E1C0
00452973   .  51           push    ecx
00452974   .  FFD7         call    edi
00452976   .  83C4 08      add     esp, 8
00452979   .  85C0         test    eax, eax
0045297B   .  74 26        je      short 004529A3                 ; 公司名称是否为空
0045297D   .  8B5424 10    mov     edx, [esp+10]
00452981   .  68 C0E14700  push    0047E1C0
00452986   .  52           push    edx
00452987   .  FFD7         call    edi
00452989   .  83C4 08      add     esp, 8
0045298C   .  85C0         test    eax, eax
0045298E   .  74 13        je      short 004529A3                 ; 注册日期是否为空
00452990   .  8B4424 18    mov     eax, [esp+18]
00452994   .  68 C0E14700  push    0047E1C0
00452999   .  50           push    eax
0045299A   .  FFD7         call    edi                            ; PH7
0045299C   .  83C4 08      add     esp, 8
0045299F   .  85C0         test    eax, eax
004529A1   .  75 10        jnz     short 004529B3                 ; 注册码是否为空
004529A3   >  6A 00        push    0
004529A5   .  6A 00        push    0
004529A7   .  68 38DE4700  push    0047DE38                   ; 尊敬的客户!你需要在右边的网站上注册后,正确的填写用户
                                                                   ; 、邮箱、地区、注册日期和取得的密码就可以注册啦!
004529AC   .  8BCE         mov     ecx, esi
004529AE   .  E8 8D890000  call    <jmp.&MFC42.#4224>
004529B3   >  8D4C24 10    lea     ecx, [esp+10]
004529B7   .  E8 48890000  call    <jmp.&MFC42.#6282>
004529BC   .  8D4C24 10    lea     ecx, [esp+10]                ; 20061105
004529C0   .  E8 39890000  call    <jmp.&MFC42.#6283>
004529C5   .  8D4C24 14    lea     ecx, [esp+14]
004529C9   .  E8 36890000  call    <jmp.&MFC42.#6282>
004529CE   .  8D4C24 14    lea     ecx, [esp+14]                ; lj@163.com
004529D2   .  E8 27890000  call    <jmp.&MFC42.#6283>
004529D7   .  B9 10000000  mov     ecx, 10
004529DC   .  33C0         xor     eax, eax
004529DE   .  8D7C24 55    lea     edi, [esp+55]
004529E2   .  C64424 54 00  mov     byte ptr [esp+54], 0
004529E7   .  F3:AB        rep     stos dword ptr es:[edi]
004529E9   .  8D4C24 2C    lea     ecx, [esp+2C]
004529ED   .  8D6E 60      lea     ebp, [esi+60]
004529F0   .  E8 B5880000  call    <jmp.&MFC42.#540>
004529F5   .  8D4C24 24    lea     ecx, [esp+24]
004529F9   .  C68424 24010>mov     byte ptr [esp+124], 5
00452A01   .  E8 A4880000  call    <jmp.&MFC42.#540>
00452A06   .  8D4C24 10    lea     ecx, [esp+10]
00452A0A   .  68 30DE4700  push    0047DE30                   ; fuck 固定字符串 作者真搞笑
00452A0F   .  8D5424 2C    lea     edx, [esp+2C]
00452A13   .  B3 06        mov     bl, 6
00452A15   .  51           push    ecx
00452A16   .  52           push    edx
00452A17   .  889C24 30010>mov     [esp+130], bl
00452A1E   .  E8 6D8A0000  call    <jmp.&MFC42.#924>
00452A23   .  8D4C24 14    lea     ecx, [esp+14]
00452A27   .  8D5424 34    lea     edx, [esp+34]
00452A2B   .  51           push    ecx
00452A2C   .  50           push    eax
00452A2D   .  52           push    edx
00452A2E   .  C68424 30010>mov     byte ptr [esp+130], 7
00452A36   .  E8 17890000  call    <jmp.&MFC42.#922>
```



```
00452A3B   .  50               push    eax  

00452A3C   .  8D4C24 30        lea     ecx, [esp+30]  

00452A40   .  C68424 280100>   mov     byte ptr [esp+128], 8  

00452A48   .  E8 D5880000      call    <jmp.&MFC42.#858>  

00452A4D   .  8D4C24 34        lea     ecx, [esp+34]  

00452A51   .  C68424 240100>   mov     byte ptr [esp+124], 7  

00452A59   .  E8 1C880000      call    <jmp.&MFC42.#800>  

00452A5E   .  8D4C24 28        lea     ecx, [esp+28]  

00452A62   .  889C24 240100>   mov     [esp+124], bl  

00452A69   .  E8 0C880000      call    <jmp.&MFC42.#800>  

00452A6E   .  8B45 00          mov     eax, [ebp]  

00452A71   .  8BCD             mov     ecx, ebp  

00452A73   .  FF50 0C          call    [eax+C]                    ;  连接连接  

00452A76   .  8B4424 2C        mov     eax, [esp+2C]              ;  "20061105fucklj@163.com"  

00452A7A   .  8B55 00          mov     edx, [ebp]  

00452A7D   .  8B48 F8          mov     ecx, [eax-8]               ；MD5（20061105fucklj@163.com）就是真正的注册码。  

00452A80   .  51               push    ecx  

00452A81   .  50               push    eax  

00452A82   .  8BCD             mov     ecx, ebp  

00452A84   .  FF52 04          call    [edx+4]  

00452A87   .  8B45 00          mov     eax, [ebp]  

00452A8A   .  8D4C24 54        lea     ecx, [esp+54]  

00452A8E   .  51               push    ecx  

00452A8F   .  8BCD             mov     ecx, ebp  

00452A91   .  FF50 08          call    [eax+8]  

00452A94   .  B9 20000000      mov     ecx, 20  

00452A99   .  33C0             xor     eax, eax  

00452A9B   .  8DBC24 990000>   lea     edi, [esp+99]  

00452AA2   .  C68424 980000>   mov     byte ptr [esp+98], 0  

00452AAA   .  8D9424 980000>   lea     edx, [esp+98]  

00452AB1   .  F3:AB            rep     stos dword ptr es:[edi]  

00452AB3   .  52               push    edx  

00452AB4   .  8D4424 58        lea     eax, [esp+58]  

00452AB8   .  6A 10            push    10  

00452ABA   .  50               push    eax  

00452ABB   .  E8 40FDFFFF      call    00452800                   ;  这个是关键算法 call。使用的居然是标准的 MD5 算法  

00452AC0   .  8B5424 24        mov     edx, [esp+24]              ;  54  

00452AC4   .  8D8C24 A40000>   lea     ecx, [esp+A4]              ;  "F33F6E66EBEEC6884608F0CDA34CCA54"  

00452ACB   .  51               push    ecx                        ; /s2  

00452ACC   .  52               push    edx                        ; |假码  

00452ACD   .  FF15 74894600    call    [<&MSVCRT._mbscmp>]       ; \(msvcrt._mbscmp) 关键比较。  

00452AD3   .  83C4 14          add     esp, 14                    ;  QH7  

00452AD6   .  85C0             test    eax, eax  

00452AD8   .  0F85 F0020000    jnz     00452DCE                   ;  关键跳转  

00452ADE   .  8D4424 20        lea     eax, [esp+20]  

00452AE2   .  8D4C24 30        lea     ecx, [esp+30]  

00452AE6   .  50               push    eax  

00452AE7   .  68 14DE4700      push    0047DE14                   ;  update regsoft set rname ='  

00452AEC   .  51               push    ecx  

00452AED   .  E8 5E8A0000      call    <jmp.&MFC42.#926>  

00452AF2   .  68 08DE4700      push    0047DE08                   ;  ', remail='
```

整个算法只和你输入的注册日期和邮箱有关系。其它的信息只要不为空就行。

`注册码＝MD5（注册日期 fuck 邮箱地址）`

`4dd466d0e1e0cc123afbe504da2462ca`
