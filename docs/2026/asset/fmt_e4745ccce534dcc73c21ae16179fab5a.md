
```asm
0040E8B7    C2 0800         retn 8                                     ; 用户名取位合并结束，返回

........

========================= 跟进 0040DF16    E8 65CFFFFF     call BoxMan.0040AE80 =========================

0040AE80    6A FF           push -1                                               ; 跟进来到这里

0040AE82    68 C8464300     push BoxMan.004346C8

0040AE87    64:A1 00000000  mov eax,dword ptr fs:[0]                              ; 取出计算名，ASCII "KUasfwfawefaefawfawfaf"

0040AE8D    50              push eax

0040AE8E    64:8925 0000000>mov dword ptr fs:[0],esp

0040AE95    83EC 60         sub esp,60

0040AE98    56              push esi

0040AE99    8B7424 7C       mov esi,dword ptr ss:[esp+7C]           ; 得到计算名位数

0040AE9D    57              push edi

0040AE9E    8B7C24 7C       mov edi,dword ptr ss:[esp+7C]           ; 得到计算名

0040AEA2    6A 00           push 0

0040AEA4    56              push esi                                ; esi=00000016（22位）

0040AEA5    57              push edi                                ; edi=00BEBD98, (ASCII "KUasfwfawefaefawfawfaf")

0040AEA6    C74424 14 00000>mov dword ptr ss:[esp+14],0

0040AEAE    E8 B93B0100     call BoxMan.0041EA6C

0040AEB3    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]

0040AEB7    E8 A40A0000     call BoxMan.0040B960                                  ; ★调用MD5标准算法常数★F7跟进

0040AEBC    56              push esi

0040AEBD    57              push edi

0040AEBE    8D4C24 14       lea ecx,dword ptr ss:[esp+14]

0040AEC2    C74424 78 00000>mov dword ptr ss:[esp+78],0

0040AECA    E8 810C0000     call BoxMan.0040BB50                                  ; ★调用MD5标准算法机制★F7跟进

0040AECF    8B7424 78       mov esi,dword ptr ss:[esp+78]

0040AED3    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]

0040AED7    56              push esi

0040AED8    E8 130B0000     call BoxMan.0040B9F0                                  ; ★使用MD5标准算法，转换计算名★

0040AEDD    8B4C24 68       mov ecx,dword ptr ss:[esp+68]

0040AEE1    8BC6            mov eax,esi

0040AEE3    5F              pop edi

0040AEE4    5E              pop esi

0040AEE5    64:890D 0000000>mov dword ptr fs:[0],ecx

0040AEEC    83C4 6C         add esp,6C

0040AEEF    C3              retn                                                  ; 计算完毕，返回程序

........

========================= 跟进 0040AEB7    E8 A40A0000     call BoxMan.0040B960 =========================

0040B960    8BD1            mov edx,ecx                                ; 下面是MD5算法的标准常数

0040B962    57              push edi

0040B963    B9 10000000     mov ecx,10

0040B968    33C0            xor eax,eax

0040B96A    8D7A 04         lea edi,dword ptr ds:[edx+4]

0040B96D    C702 88744300   mov dword ptr ds:[edx],BoxMan.00437488

0040B973    F3:AB           rep stos dword ptr es:[edi]

0040B975    8942 48         mov dword ptr ds:[edx+48],eax

0040B978    8942 44         mov dword ptr ds:[edx+44],eax

0040B97B    C742 4C 0123456>mov dword ptr ds:[edx+4C],67452301

0040B982    C742 50 89ABCDE>mov dword ptr ds:[edx+50],EFCDAB89

0040B989    C742 54 FEDCBA9>mov dword ptr ds:[edx+54],98BADCFE

0040B990    C742 58 7654321>mov dword ptr ds:[edx+58],10325476

0040B997    8BC2            mov eax,edx

0040B999    5F              pop edi

0040B99A    C3              retn

........

========================= 跟进 0040AECA    E8 810C0000     call BoxMan.0040BB50 =========================

0040BB50    53              push ebx                                              ; 以下是MD5算法的标准变换运算机制

0040BB51    8BD9            mov ebx,ecx

0040BB53    8B4C24 0C       mov ecx,dword ptr ss:[esp+C]

0040BB57    55              push ebp

0040BB58    8B53 44         mov edx,dword ptr ds:[ebx+44]

0040BB5B    56              push esi

0040BB5C    8BC2            mov eax,edx

0040BB5E    8D34CD 00000000 lea esi,dword ptr ds:[ecx\*8]

0040BB65    C1E8 03         shr eax,3

0040BB68    8D14CA          lea edx,dword ptr ds:[edx+ecx\*8]

0040BB6B    83E0 3F         and eax,3F

0040BB6E    3BD6            cmp edx,esi

0040BB70    57              push edi

0040BB71    8953 44         mov dword ptr ds:[ebx+44],edx

0040BB74    73 03           jnb short BoxMan.0040BB79

0040BB76    FF43 48         inc dword ptr ds:[ebx+48]

0040BB79    8B7B 48         mov edi,dword ptr ds:[ebx+48]

0040BB7C    8BD1            mov edx,ecx

0040BB7E    BD 40000000     mov ebp,40

0040BB83    C1EA 1D         shr edx,1D

0040BB86    2BE8            sub ebp,eax

0040BB88    03FA            add edi,edx

0040BB8A    3BCD            cmp ecx,ebp

0040BB8C    897B 48         mov dword ptr ds:[ebx+48],edi

0040BB8F    72 52           jb short BoxMan.0040BBE3

0040BB91    8B7424 14       mov esi,dword ptr ss:[esp+14]

0040BB95    8BCD            mov ecx,ebp

0040BB97    8D7C18 04       lea edi,dword ptr ds:[eax+ebx+4]

0040BB9B    8BC1            mov eax,ecx

0040BB9D    C1E9 02         shr ecx,2

0040BBA0    F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]

0040BBA2    8BC8            mov ecx,eax

0040BBA4    83E1 03         and ecx,3

0040BBA7    F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]

0040BBA9    8D4B 04         lea ecx,dword ptr ds:[ebx+4]

0040BBAC    51              push ecx

0040BBAD    8BCB            mov ecx,ebx

0040BBAF    E8 8CF3FFFF     call BoxMan.0040AF40

0040BBB4    8BFD            mov edi,ebp

0040BBB6    8D75 3F         lea esi,dword ptr ss:[ebp+3F]

0040BBB9    8B6C24 18       mov ebp,dword ptr ss:[esp+18]

0040BBBD    3BF5            cmp esi,ebp

0040BBBF    73 1A           jnb short BoxMan.0040BBDB

0040BBC1    8B5424 14       mov edx,dword ptr ss:[esp+14]

0040BBC5    8BCB            mov ecx,ebx

0040BBC7    8D4432 C1       lea eax,dword ptr ds:[edx+esi-3F]

0040BBCB    50              push eax

0040BBCC    E8 6FF3FFFF     call BoxMan.0040AF40

0040BBD1    83C6 40         add esi,40

0040BBD4    83C7 40         add edi,40

0040BBD7    3BF5            cmp esi,ebp

0040BBD9  ^ 72 E6           jb short BoxMan.0040BBC1

0040BBDB    8B4C24 18       mov ecx,dword ptr ss:[esp+18]

0040BBDF    33C0            xor eax,eax

0040BBE1    EB 02           jmp short BoxMan.0040BBE5

0040BBE3    33FF            xor edi,edi

0040BBE5    8B5424 14       mov edx,dword ptr ss:[esp+14]

0040BBE9    2BCF            sub ecx,edi

0040BBEB    8D3417          lea esi,dword ptr ds:[edi+edx]

0040BBEE    8D7C18 04       lea edi,dword ptr ds:[eax+ebx+4]

0040BBF2    8BC1            mov eax,ecx

0040BBF4    C1E9 02         shr ecx,2

0040BBF7    F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]

0040BBF9    8BC8            mov ecx,eax

0040BBFB    83E1 03         and ecx,3

0040BBFE    F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]

0040BC00    5F              pop edi

0040BC01    5E              pop esi

0040BC02    5D              pop ebp

0040BC03    5B              pop ebx

0040BC04    C2 0800         retn 8

........

```

---

## 算法总结

1. 注册码固定为16位。
2. 把用户名所有字符由小写转为大写,结果记为`N1`。
3. `N1`前两位与密钥“`asfwfawefaefawfawfaf`”连接组合成计算名,结果记为`N2`。

注意：用户名位数小于或等于2位，则调用程序固定用户名“`AA`”来作为用户名。密钥变为：`rfwefawefawefaefawefa`）

5. 将`N2`进行标准MD5运算转换,结果记为`KEY`。
6. 取`KEY`前16位转换为小写输出，则为注册码，结果记为`SN`。

---

## 注册机

算法已经很清楚了，注册机我就不写了，自己试一试，很简单的哦。。。

---

## 注册信息

Registration Name：KuNgBiM
Enter your registered Code：89ec-c085-789c-d0cc

---

版权所有（C）2005 KuNgBiM[DFCG]         Copyright (C) 2005 KuNgBiM[DFCG]

---

Cracked By KuNgBiM[DFCG]

2005-11-06

03:23:33 AM

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

