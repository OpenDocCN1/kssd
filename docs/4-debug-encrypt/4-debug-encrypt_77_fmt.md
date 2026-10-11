
# NetWatch 网络监控与入侵检测系统专业版 2.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6888.htm`](https://www.pediy.com/kssd/pediy06/pediy6888.htm)

## 基本信息

【破文作者】 落魄浪子

【文章题目】 失败的 MD5 算法

【软件名称】 XXXX 网络软件 V2.01

【下载地址】 [`mujweb.cz/www/netsoft/`](http://mujweb.cz/www/netsoft/)

【加密方式】 MD5

【破解工具】 FLY 大侠的 OD1.1

【破解平台】 XP SP2

## 简介

【软件简介】
这个软件具有网络监控和实时检测功能，可以实时监控网络流量信息和连接信息

【文章简介】
软件没有壳，现在这种不加壳的软件真是少，呵呵，我这样的菜菜就喜欢没壳的，不过有壳也没关系，呵呵。＠！＃＠！＃＃￥＠

## 破解过程

注册码不对时有出错提示“序列号校验失败，请重新输入序列号”，呵呵，有门，OD 载入，查找字符串，找到两处下断，向上找看看哪里跳过来的然后下断，载入后运行。

```
004A6E99     .  E8 46270000  call <jmp.&MFC42.#3097>  OD 中断在这里
004A6E9E     .  8B4C24 08    mov ecx,dword ptr ss:[esp+8]
004A6EA2     .  8379 F8 17   cmp dword ptr ds:[ecx-8],17   比较注册码是否等于 17 即 23 位
004A6EA6     .  75 24        jnz short NetWatch.004A6ECC   不对则出错
。。。。。。。。。。。。。。。。。。。。。。。。。
004A6EB6     .  E8 BD260000  call <jmp.&MFC42.#3098>
004A6EBB     .  8D4424 18    lea eax,dword ptr ss:[esp+18] 注册码入 EAX
004A6EBF     .  50           push eax
004A6EC0     .  E8 2BEEFFFF  call NetWatch.004A5CF0    注册算法 CALL，进入
004A6EC5     .  83C4 04      add esp,4
004A6EC8     .  85C0         test eax,eax
004A6ECA     .  75 1B        jnz short NetWatch.004A6EE7   不跳则出错
004A6ECC     >  68 00100000  push 1000
004A6ED1     .  68 105B4D00  push NetWatch.004D5B10
004A6ED6     .  68 70B64D00  push NetWatch.004DB670
004A6EDB     .  8BCE         mov ecx,esi
004A6EDD     .  E8 00260000  call <jmp.&MFC42.#4224>    出错信息
004A5CF0     /$  81EC 9C00000>sub esp,9C             进入后来到这里
004A5CF6     |.  83C9 FF      or ecx,FFFFFFFF
004A5CF9     |.  33C0         xor eax,eax
004A5CFB     |.  56           push esi
004A5CFC     |.  8BB424 A400>mov esi,dword ptr ss:[esp+A4]
004A5D03     |.  57           push edi
004A5D04     |.  8BFE         mov edi,esi
004A5D06     |.  F2:AE        repne scas byte ptr es:[edi]
004A5D08     |.  F7D1         not ecx
004A5D0A     |.  49           dec ecx
004A5D0B     |.  83F9 17      cmp ecx,17
004A5D0E     |.  0F85 9201000>jnz NetWatch.004A5EA6  再次比较注册码的位数，不对则出错
004A5D14     |.  8A4E 05      mov cl,byte ptr ds:[esi+5]比较注册码第六位是否等于 2D 即“-”符号
004A5D17     |.  B0 2D        mov al,2D
004A5D19     |.  3AC8         cmp cl,al
004A5D1B     |.  0F85 8501000>jnz NetWatch.004A5EA6      不对则出错
004A5D21     |.  3846 0B      cmp byte ptr ds:[esi+B],al 比较注册码第 12 位是否等于 2D 即“-”符号
004A5D24     |.  0F85 7C01000>jnz NetWatch.004A5EA6       不对则出错
004A5D2A     |.  3846 11      cmp byte ptr ds:[esi+11],al 比较注册码第 18 位是否等于 2D 即“-”符号
004A5D2D     |.  0F85 7301000>jnz NetWatch.004A5EA6       不对则出错
004A5D33     |.  B9 08000000  mov ecx,8
004A5D38     |.  33C0         xor eax,eax
004A5D3A     |.  8D7C24 2C    lea edi,dword ptr ss:[esp+2C]
004A5D3E     |.  6A 05        push 5                           ; /maxlen = 5
004A5D40     |.  F3:AB        rep stos dword ptr es:[edi]           ; |
004A5D42     |.  8D4424 30    lea eax,dword ptr ss:[esp+30]          ; |
004A5D46     |.  56           push esi                          ; |src
004A5D47     |.  50           push eax                          ; |dest
004A5D48     |.  FF15 B85C4B0>call dword ptr ds:[<&MSVCRT.strncpy>] ; \strncpy
004A5D4E     |.  8B3D 385C4B0>mov edi,dword ptr ds:[<&MSVCRT.strnca>;   msvcrt.strncat
004A5D54     |.  8D4E 06      lea ecx,dword ptr ds:[esi+6]
004A5D57     |.  6A 05        push 5                           ; /maxlen = 5
004A5D59     |.  8D5424 3C    lea edx,dword ptr ss:[esp+3C]          ; |
004A5D5D     |.  51           push ecx                          ; |src
004A5D5E     |.  52           push edx                          ; |dest
004A5D5F     |.  FFD7         call edi                          ; \strncat
004A5D61     |.  8D46 0C      lea eax,dword ptr ds:[esi+C]
004A5D64     |.  6A 05        push 5
004A5D66     |.  8D4C24 48    lea ecx,dword ptr ss:[esp+48]
004A5D6A     |.  50           push eax
004A5D6B     |.  51           push ecx
004A5D6C     |.  FFD7         call edi
004A5D6E     |.  83C6 12      add esi,12
004A5D71     |.  6A 05        push 5
004A5D73     |.  8D5424 54    lea edx,dword ptr ss:[esp+54]
004A5D77     |.  56           push esi
004A5D78     |.  52           push edx
004A5D79     |.  FFD7         call edi             以上是把注册码的“-”符号去掉后连起来
004A5D7B     |.  83C4 30      add esp,30
004A5D7E     |.  33C0         xor eax,eax
004A5D80     |>  8A4C04 2C    /mov cl,byte ptr ss:[esp+eax+2C]
004A5D84     |.  80F9 41      |cmp cl,41
004A5D87     |.  0F8C 1901000>|jl NetWatch.004A5EA6
004A5D8D     |.  80F9 5A      |cmp cl,5A
004A5D90     |.  0F8F 1001000>|jg NetWatch.004A5EA6
004A5D96     |.  40           |inc eax
004A5D97     |.  83F8 14      |cmp eax,14
004A5D9A     |.^ 7C E4        \jl short NetWatch.004A5D80   查找字符串是否是 A~Z 的字母，否则出错
004A5D9C     |.  8A4424 2C    mov al,byte ptr ss:[esp+2C]   第一位注册码入 AL  记为 N1
004A5DA0     |.  2C 48        sub al,48
004A5DA2     |.  3C 01        cmp al,1
004A5DA4     |.  0F85 FC00000>jnz NetWatch.004A5EA6   比较第一位注册码减 48 是否等于 1，不等则出错
004A5DAA     |.  66:0FBE4424 >movsx ax,byte ptr ss:[esp+2D]第 2 位注册码入 AX 记为 N2
004A5DB0     |.  66:0FBE5424 >movsx dx,byte ptr ss:[esp+2E]第 3 位注册码入 DX 记为 N3
004A5DB6     |.  8D0C80       lea ecx,dword ptr ds:[eax+eax*4] N2+N2*4 结果记为 R1
004A5DB9     |.  8D844A 92FCF>lea eax,dword ptr ds:[edx+ecx*2-36E]
004A5DC0     |.  66:3D 0200   cmp ax,2
004A5DC4     |.  0F85 DC00000>jnz NetWatch.004A5EA6   比较 N3+R1*2-36E＝2  不等于 2 则出错
004A5DCA     |.  8D4C24 4C    lea ecx,dword ptr ss:[esp+4C]
004A5DCE     |.  51           push ecx
004A5DCF     |.  E8 7C29FCFF  call NetWatch.00468750  进入
00468750     /$  8B4424 04    mov eax,dword ptr ss:[esp+4]   进入后来到这里
00468754     |.  33C9         xor ecx,ecx
00468756     |.  8948 14      mov dword ptr ds:[eax+14],ecx
00468759     |.  8948 10      mov dword ptr ds:[eax+10],ecx
0046875C     |.  C700 0123456>mov dword ptr ds:[eax],67452301
00468762     |.  C740 04 89AB>mov dword ptr ds:[eax+4],EFCDAB89
00468769     |.  C740 08 FEDC>mov dword ptr ds:[eax+8],98BADCFE
00468770     |.  C740 0C 7654>mov dword ptr ds:[eax+C],10325476   呵呵，这不会是 MD5 的四个变量吧？？？
00468777     \.  C3           retn                           返回
004A5DD4     |.  8D5424 30    lea edx,dword ptr ss:[esp+30] 返回到这里，注册码放入 EDX
004A5DD8     |.  6A 0C        push 0C
004A5DDA     |.  8D4424 54    lea eax,dword ptr ss:[esp+54]
004A5DDE     |.  52           push edx
004A5DDF     |.  50           push eax
004A5DE0     |.  E8 9B29FCFF  call NetWatch.00468780
004A5DE5     |.  6A 26        push 26
004A5DE7     |.  8D4C24 60    lea ecx,dword ptr ss:[esp+60]
004A5DEB     |.  68 F8B44D00  push NetWatch.004DB4F8
004A5DF0     |.  51           push ecx
004A5DF1     |.  E8 8A29FCFF  call NetWatch.00468780
004A5DF6     |.  8D5424 68    lea edx,dword ptr ss:[esp+68]  四个变量
004A5DFA     |.  8D4424 28    lea eax,dword ptr ss:[esp+28]
004A5DFE     |.  52           push edx
004A5DFF     |.  50           push eax
004A5E00     |.  E8 2B2AFCFF  call NetWatch.00468830   进入看看
00468830     /$  83EC 08      sub esp,8                进入后来到这里
00468833     |.  8D4424 00    lea eax,dword ptr ss:[esp]
00468837     |.  56           push esi
00468838     |.  57           push edi
。。。。。。。。。。。。。。。。。。。略过代码
00468871     |.  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
00468875     |.  6A 08        push 8
00468877     |.  51           push ecx
00468878     |.  57           push edi
```



00468879 `|. E8 02FFFFFF call NetWatch.00468780` 进入看看  

00468780 `/$ 53 push ebx` 进入后来到这里  

00468781 `|. 55 push ebp`  

00468782 `|. 56 push esi`  

00468783 `|. 8B7424 10 mov esi,dword ptr ss:[esp+10]`  

`。。。。。。。。。。。。。。。。。。。略过代码`  

004687D3 `|. 8D4E 18 lea ecx,dword ptr ds:[esi+18]` 注册码放入 `ECX`  

004687D6 `|. 51 push ecx`  

004687D7 `|. 56 push esi`  

004687D8 `|. E8 C3000000 call NetWatch.004688A0` 进入看看  

004688A0 `/$ 83EC 40 sub esp,40` 进入后来到这里  

004688A3 `|. 8B4C24 48 mov ecx,dword ptr ss:[esp+48]`  

004688A7 `|. 53 push ebx`  

004688A8 `|. 55 push ebp`  

004688A9 `|. 56 push esi`  

004688AA `|. 8B7424 50 mov esi,dword ptr ss:[esp+50]`  

004688AE `|. 57 push edi`  

004688AF `|. 6A 40 push 40`  

004688B1 `|. 8D5424 14 lea edx,dword ptr ss:[esp+14]`  

004688B5 `|. 8B06 mov eax,dword ptr ds:[esi]`  

004688B7 `|. 8B7E 04 mov edi,dword ptr ds:[esi+4]`  

004688BA `|. 8B5E 08 mov ebx,dword ptr ds:[esi+8]`  

004688BD `|. 8B6E 0C mov ebp,dword ptr ds:[esi+C]` 四个变量  

004688C0 `|. 51 push ecx`  

004688C1 `|. 52 push edx`  

004688C2 `|. 894424 60 mov dword ptr ss:[esp+60],eax`  

004688C6 `|. E8 75090000 call NetWatch.00469240`  

`****************************************************************************************`  

004688CB `|. 8BC7 mov eax,edi` MD5 计算开始  

004688CD `|. 8B5424 60 mov edx,dword ptr ss:[esp+60]`  

004688D1 `|. F7D0 not eax`  

004688D3 `|. 8BCB mov ecx,ebx`  

004688D5 `|. 23C5 and eax,ebp`  

004688D7 `|. 23CF and ecx,edi`  

004688D9 `|. 0BC1 or eax,ecx`  

004688DB `|. 8B4C24 1C mov ecx,dword ptr ss:[esp+1C]` 第一轮  

004688DF `|. 03C1 add eax,ecx`  

004688E1 `|. 8D8C02 78A46 lea ecx,dword ptr ds:[edx+eax+D76AA478]`  

`。。。。。。。。。。。。。。略过代码 。。。。。。。。。。。。。。。。。`  

00468BC6 `|. 0BFB or edi,ebx`  

00468BC8 `|. 8B5C24 1C mov ebx,dword ptr ss:[esp+1C]` 第二轮  

00468BCC `|. 03FB add edi,ebx`  

00468BCE `|. 8B5C24 60 mov ebx,dword ptr ss:[esp+60]`  

00468BD2 `|. 8D9C3B AAC7B lea ebx,dword ptr ds:[ebx+edi+E9B6C7AA]`  

`。。。。。。。。。。。。。。略过代码 。。。。。。。。。。。。。。。。。。。`  

00468ED7 `|. 33F9 xor edi,ecx`  

00468ED9 `|. 03FD add edi,ebp`  

00468EDB `|. 8B6C24 1C mov ebp,dword ptr ss:[esp+1C]` 第三轮  

00468EDF `|. 8DBC38 C67E9 lea edi,dword ptr ds:[eax+edi+289B7EC6]`  

`。。。。。。。。。。。。。。略过代码 。。。。。。。。。。。。。。。。。。。`  

00468FC1 `|. 0BCB or ecx,ebx`  

00468FC3 `|. 33CF xor ecx,edi`  

00468FC5 `|. 034C24 1C add ecx,dword ptr ss:[esp+1C]` 第四轮  

00468FC9 `|. 8D8408 44222 lea eax,dword ptr ds:[eax+ecx+F4292244]`  

`。。。。。。。。。。。。。。略过代码 。。。。。。。。。。。。。。。。。。。`  

004691C0 `|. 03DF add ebx,edi`  

004691C2 `|. 8946 04 mov dword ptr ds:[esi+4],eax` `*A`  

004691C5 `|. 8D4424 24 lea eax,dword ptr ss:[esp+24]`  

004691C9 `|. 50 push eax`  

004691CA `|. 890E mov dword ptr ds:[esi],ecx` `*B`  

004691CC `|. 896E 08 mov dword ptr ds:[esi+8],ebp` `*C`  

004691CF `|. 895E 0C mov dword ptr ds:[esi+C],ebx` `*D`  

004691D2 `|. E8 D9000000 call NetWatch.004692B0`  

004691D7 `|. 83C4 18 add esp,18`  

004691DA `|. 5F pop edi`  

004691DB `|. 5E pop esi`  

004691DC `|. 5D pop ebp`  

004691DD `|. 5B pop ebx`  

004691DE `|. 83C4 40 add esp,40`  

004691E1 `\. C3 retn` 返回  

以上代码是把注册码的前 12 位和本软件的名字 `+991235` 进行 `MD5` 运算，  

结果记为十六进制：`AA,BB,CC,DD,EE,FF,GG,HH,JJ,KK,MM,NN,RR,UU,TT,PP`  

004A5E05 `|. 83C4 24 add esp,24` 经过几个 `CALL` 返回之后来到这里  

004A5E08 `|. 33C0 xor eax,eax`  

004A5E0A `|> 8A4C04 0C /mov cl,byte ptr ss:[esp+eax+C]`  

004A5E0E `|. 80F1 25 |xor cl,25`  

004A5E11 `|. 884C04 0C |mov byte ptr ss:[esp+eax+C],cl`  

004A5E15 `|. 40 |inc eax`  

004A5E16 `|. 83F8 10 |cmp eax,10`  

004A5E19 `|.^ 7C EF \\jl short NetWatch.004A5E0A` 取 `AA~PP` 与 `25` 做异或 连起来记为 `N4`  

004A5E1B `|. 33C0 xor eax,eax`  

004A5E1D `|> 8A4C04 14 /mov cl,byte ptr ss:[esp+eax+14]`  

004A5E21 `|. 8A5404 0C |mov dl,byte ptr ss:[esp+eax+C]`  

004A5E25 `|. 32CA |xor cl,dl`  

004A5E27 `|. 884C04 08 |mov byte ptr ss:[esp+eax+8],cl`  

004A5E2B `|. 40 |inc eax`  

004A5E2C `|. 83F8 08 |cmp eax,8`  

004A5E2F `|.^ 7C EC \\jl short NetWatch.004A5E1D` `N4` 分成 2 组分别进行异或运算，结果记为 `N5`，`N5` 和 `N4` 的后 3 组连起来记为 `N6`  

004A5E31 `|. 53 push ebx`  

004A5E32 `|. B9 64000000 mov ecx,64`  

004A5E37 `|> 33C0 /xor eax,eax`  

004A5E39 `|> 8A5404 10 /mov dl,byte ptr ss:[esp+eax+10]`  

004A5E3D `|. 8A5C04 0C ||mov bl,byte ptr ss:[esp+eax+C]`  

004A5E41 `|. 32DA ||xor bl,dl`  

004A5E43 `|. 885C04 0C ||mov byte ptr ss:[esp+eax+C],bl`  

004A5E47 `|. 40 ||inc eax`  

004A5E48 `|. 83F8 08 ||cmp eax,8`  

004A5E4B `|.^ 7C EC |\\jl short NetWatch.004A5E39` `N6` 分成 5 组，每组 8 位，取第二组分别与第一组和第三组做异或运算，结果记为 `N7`，`N7` 和 `N6` 的后 3 组连起来记为 `N8` 这样运算共进行 64 次 记为 `N9`  

004A5E4D `|. 49 |dec ecx`  

004A5E4E `|.^ 75 E7 \\jnz short NetWatch.004A5E37`  

004A5E50 `|. 33C9 xor ecx,ecx`  

004A5E52 `|. 8D7424 11 lea esi,dword ptr ss:[esp+11]` 注册码的最后五位入 `ESI`  

004A5E56 `|> 8A440C 0C /mov al,byte ptr ss:[esp+ecx+C]`  

004A5E5A `|. 83C6 02 |add esi,2`  

004A5E5D `|. 8AD0 |mov dl,al`  

004A5E5F `|. 24 0F |and al,0F`  

004A5E61 `|. C0EA 04 |shr dl,4`  

004A5E64 `|. 8856 FD |mov byte ptr ds:[esi-3],dl`  

004A5E67 `|. 8846 FE |mov byte ptr ds:[esi-2],al`  

004A5E6A `|. 41 |inc ecx`  

004A5E6B `|. 83F9 08 |cmp ecx,8`  

004A5E6E `|.^ 7C E6 \\jl short NetWatch.004A5E56`  

`N9` 分成 5 组，取第一组的每两位与 `0F` 做与运算和右移 4 次，结果依次连起来分别替换第二组和第三组记为 `NN1`  

`NN1` 分成 5 组，取第二组的每两位与 `0F` 做与运算和右移 4 次，结果依次连起来分别替换 `NN1` 的第四组和第五组记为 `NN2`  

004A5E70 `|. 33C0 xor eax,eax`  

004A5E72 `|> 8A5C04 10 /mov bl,byte ptr ss:[esp+eax+10]`  

004A5E76 `|. 80C3 44 |add bl,44`  

004A5E79 `|. 885C04 10 |mov byte ptr ss:[esp+eax+10],bl`  

004A5E7D `|. 40 |inc eax`  

004A5E7E `|. 83F8 08 |cmp eax,8`  

004A5E81 `|.^ 7C EF \\jl short NetWatch.004A5E72`  

`NN2` 分成 5 组，取第二组和第三组的每两位与 `44` 相加，结果依次替换第二组和第三组记为 `NN3`  

004A5E83 `|. B9 02000000 mov ecx,2`  

004A5E88 `|. 8D7C24 3C lea edi,dword ptr ss:[esp+3C]` 输入的注册码的最后 8 位  

004A5E8C `|. 8D7424 10 lea esi,dword ptr ss:[esp+10]` 上面计算的结果 `NN3`  

004A5E90 `|. 33C0 xor eax,eax`  

004A5E92 `|. C64424 18 00 mov byte ptr ss:[esp+18],0`  

004A5E97 `|. 5B pop ebx`  

004A5E98 `|. F3:A7 repe cmps dword ptr es:[edi],dword ptr ds:[esi]` 晕，最后竟然进行明码比较  

004A5E9A `|. 5F pop edi`  

004A5E9B `|. 5E pop esi`  

004A5E9C `|. 0F94C0 sete al` 置注册标志位，正确注册 `AL` 为 1  

004A5E9F `|. 81C4 9C000000 add esp,9C`  

004A5EA5 `|. C3 retn` 返回  

004A6EC5 `. 83C4 04 add esp,4` 来到这里  

004A6EC8 `. 85C0 test eax,eax`  

004A6ECA `. 75 1B jnz short NetWatch.004A6EE7` 不对则出错  

004A6ECC `> 68 00100000 push 1000`  

004A6ED1 `. 68 105B4D00 push NetWatch.004D5B10`  

004A6ED6 `. 68 70B64D00 push NetWatch.004DB670`  



`004A6EDB     .  8BCE         mov ecx,esi`  

`004A6EDD     .  E8 00260000  call <jmp.&MFC42.#4224>    出错`  

到这里已经全部完成，呵呵，本人文字表达能力差，写的很乱也很臭，以上分组只是我为了好表达  

才这样，希望各位能看的明白，不能我就白忙了。。。。。  

----------------------------------------------------------------------------------------------  

【破解心得】  

第一位注册码减 `48` 等于 `1`，第 `3` 位注册码+（第 `2` 位注册码+第 `2` 位注册码\*4）\*2-`36E` 等于 `2`，  

注册码的前 `12` 位和本软件的名字+`991235` 进行 `MD5` 运算后的结果再进行一些运算得到最后 `8` 位注册码。  

这个软件的加密实在是。。。唉，还用什么 `MD5` 嘛，最后还进行明码比较，写注册机也非常  

容易，网上 `MD5` 算法库到处都有。  

----------------------------------------------------------------------------------------------  

【破解声明】   我是一只小菜鸟，偶得一点心得，愿与大家分享:)  

【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  

----------------------------------------------------------------------------------------------

文章写于 `2005-12-29 10:09:04`

`4dd466d0e1e0cc123afbe504da2462ca`
