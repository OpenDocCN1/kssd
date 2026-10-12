
# MD5 加密算法破解--莱鸟入门破文之三【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-764.htm`](https://www.pediy.com/kssd/pediy07/pediy7-764.htm)

MD5 加密算法破解--莱鸟入门破文之三

日期：2005 年 12 月 15 日 破解人：林海雪原

———————————————————————————————————————————

【软件名称】：`Netpise` 迅派网上办公室 `1.2`

【软件大小】： KB

【下载地址】：`google`

【软件限制】：`30` 天试用

【破解声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`win2K`, `od`, `dede`

———————————————————————————————————————————

## 破解过程

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

注册码格式：

注册码 1-注册码 2-认证号

`XXXXXX`-`XXXXXX`-`XXX`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

试练信息：

用户名：林海雪原

注册码：`123456-789012-123`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

今天，主任要我给他弄下软件，不知是谁的不是，一不小心，`10` 分钟弄破了 `md5`（后经主任批复：是作者之过。既用了高档算法，为什么还要明码比较？![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) ）。拿到它时，用 `peid` 查了下: `delphi6.0-7.0`,无壳？![](img/b16bec246e75299c87cf57e1fb808ac9.jpg) 心里一阵窃喜....N 久没有遇到软柿子了，今日可别错过.....

试注册一下，有出错提示....（这个柿子可真是熟透了呵，还没捏就烂了![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ）....`delphi` 的东东，入口太熟了，就不贴了。

`OD` 载入后，直接 `go:00429420`,下断：

```
00429420   /.  55            push  ebp
...............
00429450   |.  E8 9391FDFF   call  <jmp.&vcl70.Controls::TControl::GetText>  ；断点设于此！
00429455   |.  8B45 E8       mov  eax,[dword ss:ebp-18]
00429458   |.  8D55 EC       lea  edx,[dword ss:ebp-14]
```

为什么不在首行断？这个自然是请 `DeDe` 帮搞了下的，我可真敬慕作者人编程习惯，到了 `DeDe` 里面，那个模块是些行么功能，那简直是一目了然！你看看它那些控件响应事件，每次按键都有相应动作，首行断只会给自己找麻烦。先别管他，运行，注册，断下了！（输入试练信息后，在激活断点，运行）

```
00429450   |.  E8 9391FDFF   call  <jmp.&vcl70.Controls::TControl::Ge>
00429455   |.  8B45 E8       mov  eax,[dword ss:ebp-18]                        ;  林海雪原
00429458   |.  8D55 EC       lea  edx,[dword ss:ebp-14]
0042945B   |.  E8 4483FDFF   call  <jmp.&rtl70.Sysutils::Trim>
00429460   |.  837D EC 00    cmp  [dword ss:ebp-14],0
00429464   |.  75 28         jnz  short  NetPise.0042948E                       ;  用户名不为空！
00429466   |.  8D55 E4       lea  edx,[dword ss:ebp-1C]
00429469   |.  A1 E4D44400   mov  eax,[dword ds:44D4E4]
0042946E   |.  E8 0D7FFDFF   call  <jmp.&rtl70.System::LoadResString>
00429473   |.  8B45 E4       mov  eax,[dword ss:ebp-1C]
00429476   |.  E8 E1AAFDFF   call  NetPise.00403F5C
0042947B   |.  8B83 08030000 mov  eax,[dword ds:ebx+308]
00429481   |.  8B10          mov  edx,[dword ds:eax]
00429483   |.  FF92 C4000000 call  [dword ds:edx+C4]
00429489   |.  E9 FF010000   jmp  NetPise.0042968D
0042948E   |>  8D55 E0       lea  edx,[dword ss:ebp-20]
00429491   |.  8B83 0C030000 mov  eax,[dword ds:ebx+30C]
00429497   |.  E8 4C91FDFF   call  <jmp.&vcl70.Controls::TControl::Ge>
0042949C   |.  8B45 E0       mov  eax,[dword ss:ebp-20]                        ;  注册码 1=123456
0042949F   |.  8D55 F8       lea  edx,[dword ss:ebp-8]
004294A2   |.  E8 FD82FDFF   call  <jmp.&rtl70.Sysutils::Trim>
004294A7   |.  837D F8 00    cmp  [dword ss:ebp-8],0
004294AB   |.  74 0D         je   short  NetPise.004294BA                       ;  不为空！
004294AD   |.  8B45 F8       mov  eax,[dword ss:ebp-8]
004294B0   |.  E8 537DFDFF   call  <jmp.&rtl70.System::LStrLen>
004294B5   |.  83F8 06       cmp  eax,6                                         ;  6 位!
004294B8   |.  74 28         je   short  NetPise.004294E2
004294BA   |>  8D55 DC       lea  edx,[dword ss:ebp-24]
004294BD   |.  A1 60D44400   mov  eax,[dword ds:44D460]
004294C2   |.  E8 B97EFDFF   call  <jmp.&rtl70.System::LoadResString>
004294C7   |.  8B45 DC       mov  eax,[dword ss:ebp-24]
004294CA   |.  E8 8DAAFDFF   call  NetPise.00403F5C
004294CF   |.  8B83 0C030000 mov  eax,[dword ds:ebx+30C]
004294D5   |.  8B10          mov  edx,[dword ds:eax]
004294D7   |.  FF92 C4000000 call  [dword ds:edx+C4]
004294DD   |.  E9 AB010000   jmp  NetPise.0042968D
004294E2   |>  8D45 F4       lea  eax,[dword ss:ebp-C]
004294E5   |.  8B55 F8       mov  edx,[dword ss:ebp-8]
004294E8   |.  E8 DB7CFDFF   call  <jmp.&rtl70.System::LStrLAsg>
004294ED   |.  8D55 D8       lea  edx,[dword ss:ebp-28]
004294F0   |.  8B83 14030000 mov  eax,[dword ds:ebx+314]
004294F6   |.  E8 ED90FDFF   call  <jmp.&vcl70.Controls::TControl::Ge>
004294FB   |.  8B45 D8       mov  eax,[dword ss:ebp-28]                        ;  注册码 2=789012
004294FE   |.  8D55 F8       lea  edx,[dword ss:ebp-8]
00429501   |.  E8 9E82FDFF   call  <jmp.&rtl70.Sysutils::Trim>
00429506   |.  837D F8 00    cmp  [dword ss:ebp-8],0
0042950A   |.  74 0D         je   short  NetPise.00429519                       ;  不为空！
0042950C   |.  8B45 F8       mov  eax,[dword ss:ebp-8]
0042950F   |.  E8 F47CFDFF   call  <jmp.&rtl70.System::LStrLen>
00429514   |.  83F8 06       cmp  eax,6
00429517   |.  74 28         je   short  NetPise.00429541
00429519   |>  8D55 D4       lea  edx,[dword ss:ebp-2C]
0042951C   |.  A1 60D44400   mov  eax,[dword ds:44D460]
00429521   |.  E8 5A7EFDFF   call  <jmp.&rtl70.System::LoadResString>
00429526   |.  8B45 D4       mov  eax,[dword ss:ebp-2C]
00429529   |.  E8 2EAAFDFF   call  NetPise.00403F5C
0042952E   |.  8B83 14030000 mov  eax,[dword ds:ebx+314]
00429534   |.  8B10          mov  edx,[dword ds:eax]
00429536   |.  FF92 C4000000 call  [dword ds:edx+C4]
0042953C   |.  E9 4C010000   jmp  NetPise.0042968D
00429541   |>  8D45 F4       lea  eax,[dword ss:ebp-C]
00429544   |.  8B55 F8       mov  edx,[dword ss:ebp-8]
00429547   |.  E8 C47CFDFF   call  <jmp.&rtl70.System::LStrCat>
0042954C   |.  8D55 D0       lea  edx,[dword ss:ebp-30]
0042954F   |.  8B83 1C030000 mov  eax,[dword ds:ebx+31C]
00429555   |.  E8 8E90FDFF   call  <jmp.&vcl70.Controls::TControl::Ge>
0042955A   |.  8B45 D0       mov  eax,[dword ss:ebp-30]                        ;  认证号=123***以上是处理假码！
0042955D   |.  8D55 F8       lea  edx,[dword ss:ebp-8]
00429560   |.  E8 3F82FDFF   call  <jmp.&rtl70.Sysutils::Trim>
00429565   |.  837D F8 00    cmp  [dword ss:ebp-8],0
00429569   |.  75 28         jnz  short  NetPise.00429593
0042956B   |.  8D55 CC       lea  edx,[dword ss:ebp-34]
0042956E   |.  A1 60D44400   mov  eax,[dword ds:44D460]
```

我上次破 `VB` 时,感到 `CALL` 的函数可通过名称推测,这次也一样啊。后面的我不多说了，它把字符连成如下字串：专业版： 用户名+认证号+`myfirstlicense` 或 个人版：用户名+认证号+`MyPersonalLicense` ；继续，我们看见了著名的 `MD5` 运算：

```
0040BDAE   |.  50          push  eax
0040BDAF   |.  B9 06000000 mov   ecx,6
0040BDB4   |.  BA 07000000 mov   edx,7
0040BDB9   |.  8BC3        mov   eax,ebx
0040BDBB   |.  E8 D863FFFF call  <jmp.&rtl70.Strutils::MidStr>  ;截取字串！
0040BDC0   |.  8B45 D4     mov   eax,[dword ss:ebp-2C]
0040BDC3   |.  8D55 D8     lea   edx,[dword ss:ebp-28]
0040BDC6   |.  E8 C159FFFF call  <jmp.&rtl70.Sysutils::UpperCase>
0040BDCB   |.  8B55 D8     mov   edx,[dword ss:ebp-28]
0040BDCE   |.  8D45 F4     lea   eax,[dword ss:ebp-C]
0040BDD1   |.  E8 F253FFFF call  <jmp.&rtl70.System::LStrLAsg>
0040BDD6   |.  8D45 D0     lea   eax,[dword ss:ebp-30]
0040BDD9   |.  50          push  eax
0040BDDA   |.  8BC3        mov   eax,ebx
0040BDDC   |.  E8 2754FFFF call  <jmp.&rtl70.System::LStrLen>
0040BDE1   |.  8BC8        mov   ecx,eax
0040BDE3   |.  83E9 0C     sub   ecx,0C
0040BDE6   |.  BA 0D000000 mov   edx,0D
0040BDEB   |.  8BC3        mov   eax,ebx
0040BDED   |.  E8 A663FFFF call  <jmp.&rtl70.Strutils::MidStr>
0040BDF2   |.  8B45 D0     mov   eax,[dword ss:ebp-30]
0040BDF5   |.  E8 D259FFFF call  <jmp.&rtl70.Sysutils::StrToInt>
0040BDFA   |.  8945 F8     mov   [dword ss:ebp-8],eax
0040BDFD   |.  8D45 E4     lea   eax,[dword ss:ebp-1C]
0040BE00   |.  50          push  eax
0040BE01   |.  33C9        xor   ecx,ecx
```



### 汇编代码分析

```
0040BE03   |.  8B55 F8       mov edx,[dword ss:ebp-8]  

0040BE06   |.  8BC6          mov eax,esi  

0040BE08   |.  E8 B7000000   call NetPise.0040BEC4  
```

以上没有 z 细跟，总的是计算：`md5("林海雪原 123myfirstlicense")=6233F0F0BF688CA97FDEE054EFE3557C`

跟入 `0040BE08  call NetPise.0040BEC4` 之中

```
0040BF3E   |.  E8 CD7CFFFF   call NetPise.00403C10.....进入！  

..........  

00403C51   |.  E8 C6FEFFFF   call NetPise.00403B1C.....进入！  
```

到这！

```
00403B1C   /$  53            push ebx                          ;EDX 020432C8 ASCII "林海雪原 123myfirstlicense"  

00403B1D   |.  56            push esi  

00403B1E   |.  57            push edi  

00403B1F   |.  55            push ebp  

00403B20   |.  8BF9          mov edi,ecx                       ;  ecx=8  

00403B22   |.  8BEA          mov ebp,edx                       ;  edx=c8  

00403B24   |.  8BF0          mov esi,eax                       ;  0123456789abcdeffedcba9876543210  

00403B26   |.  8B46 10       mov eax,[dword ds:esi+10]         ;  "c001";01c0  

00403B29   |.  C1E8 03       shr eax,3                         ;  ...eax=38  

00403B2C   |.  83E0 3F       and eax,3F                        ;  38  

00403B2F   |.  8BD7          mov edx,edi                       ;  edi=8  

00403B31   |.  C1E2 03       shl edx,3                         ;  .....edx=40  

00403B34   |.  0156 10       add [dword ds:esi+10],edx         ;  1c0 and 40 = 200  

00403B37   |.  3B56 10       cmp edx,[dword ds:esi+10]  

00403B3A   |.  76 03         jbe short NetPise.00403B3F        ;  edx<200,跳  

00403B3C   |.  FF46 14       inc [dword ds:esi+14]  

00403B3F   |>  8BD7          mov edx,edi  

00403B41   |.  C1EA 1D       shr edx,1D  

00403B44   |.  0156 14       add [dword ds:esi+14],edx  

00403B47   |.  BB 40000000   mov ebx,40  

00403B4C   |.  2BD8          sub ebx,eax  

00403B4E   |.  3BDF          cmp ebx,edi  

00403B50   |.  77 32         ja short NetPise.00403B84  

00403B52   |.  8D4406 18     lea eax,[dword ds:esi+eax+18]  

00403B56   |.  8BCB          mov ecx,ebx  

00403B58   |.  8BD5          mov edx,ebp  

00403B5A   |.  E8 35DBFFFF   call NetPise.00401694  

00403B5F   |.  8BD6          mov edx,esi  

00403B61   |.  8D46 18       lea eax,[dword ds:esi+18]  

00403B64   |.  E8 4FF8FFFF   call NetPise.004033B8             ;  计算！跟入  

00403B69   |.  EB 0E         jmp short NetPise.00403B79  

00403B6B   |>  8BD6          /mov edx,esi  

00403B6D   |.  8D441D 00     |lea eax,[dword ss:ebp+ebx]  
```

#### 计算开始！16\*4=64 轮！

```
004033B8   /$  53           push ebx  

004033B9   |.  56           push esi  

004033BA   |.  57           push edi  

004033BB   |.  55           push ebp  

...................................................  

   第一轮  

   0040340E   |.  50           push eax                         ; /Arg4 = 10325476  

   0040340F   |.  8B4424 1C    mov eax,[dword ss:esp+1C]        ; |  

   00403413   |.  50           push eax                         ; |Arg3 = A3BAD6C1  

   00403414   |.  6A 07        push 7                           ; |Arg2 = 00000007  

   00403416   |.  68 78A46AD7  push D76AA478                   ; |Arg1 = D76AA478  

   0040341B   |.  8BC3         mov eax,ebx                      ; |"0123456789abcdeffedcba9876543210"  

   0040341D   |.  8B0F         mov ecx,[dword ds:edi]           ; |[ds:0012FC2C]=98BADCFE：89abcdef  

   0040341F   |.  8B16         mov edx,[dword ds:esi]           ; |........................fedcba98  

   00403421   |.  E8 4EFEFFFF  call NetPise.00403274            ; \NetPise.00403274  

   00403426   |.  8B07         mov eax,[dword ds:edi]  

   00403428   |.  50           push eax                         ; /Arg4  

   00403429   |.  8B4424 20    mov eax,[dword ss:esp+20]        ; |  

   0040342D   |.  50           push eax                         ; |Arg3  

   0040342E   |.  6A 0C        push 0C                          ; |Arg2 = 0000000C  

   00403430   |.  68 56B7C7E8  push E8C7B756                    ; |Arg1 = E8C7B756  

   00403435   |.  8BC5         mov eax,ebp                      ; |  

   00403437   |.  8B0E         mov ecx,[dword ds:esi]           ; |  

   00403439   |.  8B13         mov edx,[dword ds:ebx]           ; |  

   0040343B   |.  E8 34FEFFFF  call NetPise.00403274            ; \NetPise.00403274  

    .......  共 16 次  

   ........  

..................................................................  

  第二轮：  

  004035BA   |.  50           push eax                         ; /Arg4  

  004035BB   |.  8B4424 20    mov eax,[dword ss:esp+20]        ; |  

  004035BF   |.  50           push eax                         ; |Arg3  

  004035C0   |.  6A 05        push 5                           ; |Arg2 = 00000005  

  004035C2   |.  68 62251EF6  push F61E2562                    ; |Arg1 = F61E2562  

  004035C7   |.  8BC3         mov eax,ebx                      ; |  

  004035C9   |.  8B0F         mov ecx,[dword ds:edi]           ; |  

  004035CB   |.  8B16         mov edx,[dword ds:esi]           ; |  

  004035CD   |.  E8 D6FCFFFF  call NetPise.004032A8            ; \NetPise.004032A8  

   .......  共 16 次  

   ........  

..................................................................  

  第三轮：  

  00403763   |.  8B45 00      mov eax,[dword ss:ebp]  

  00403766   |.  50           push eax                         ; /Arg4  

  00403767   |.  8B4424 30    mov eax,[dword ss:esp+30]        ; |  

  0040376B   |.  50           push eax                         ; |Arg3  

  0040376C   |.  6A 04        push 4                           ; |Arg2 = 00000004  

  0040376E   |.  68 4239FAFF  push FFFA3942                    ; |Arg1 = FFFA3942  

  00403773   |.  8BC3         mov eax,ebx                      ; |  

  00403775   |.  8B0F         mov ecx,[dword ds:edi]           ; |  

  00403777   |.  8B16         mov edx,[dword ds:esi]           ; |  

  00403779   |.  E8 5EFBFFFF  call NetPise.004032DC            ; \NetPise.004032DC  

  0040377E   |.  8B07         mov eax,[dword ds:edi]  

   .......  共 16 次  

   ........  

..................................................................  

  第四轮：  

  0040379A   |.  50           push eax                         ; /Arg4  

  0040379B   |.  8B4424 48    mov eax,[dword ss:esp+48]        ; |  

  0040379F   |.  50           push eax                         ; |Arg3  

  004037A0   |.  6A 10        push 10                          ; |Arg2 = 00000010  

  004037A2   |.  68 22619D6D  push 6D9D6122                   ; |Arg1 = 6D9D6122  

  004037A7   |.  8BC7         mov eax,edi                      ; |  

  004037A9   |.  8B0B         mov ecx,[dword ds:ebx]           ; |  

  004037AB   |.  8B55 00      mov edx,[dword ss:ebp]           ; |  

  004037AE   |.  E8 29FBFFFF  call NetPise.004032DC            ; \NetPise.004032DC  

  .......  共 16 次  

   ........  

..................................................................................  

00403AE4   |.  5E           pop esi  

00403AE5   |.  5B           pop ebx  

00403AE6   \\.  C3           retn                            ;MD5 运算结束！  
```

这时你到看雪学院主页的解密教学中看看 `garfield` 整理的 `MD5` 算法吧。不要惊奇！真是一模一样！我就不献丑了。

我们继续跟下去，在这里边，你更高兴了：

```
0040BE0D   |.  8B45 F0       mov eax,[dword ss:ebp-10]         ;  真码 1=6233F0  

0040BE10   |.  8B55 E4       mov edx,[dword ss:ebp-1C]         ;  假码 1=123456  

0040BE13   |.  E8 1054FFFF   call <jmp.&rtl70.System::LStrCmp>  

0040BE18   |.  75 26         jnz short NetPise.0040BE40        ;  注册码 1 不等，跳！  

0040BE1A   |.  8B45 F4       mov eax,[dword ss:ebp-C]          ;  真码 2=E3557C  

0040BE1D   |.  8B55 E8       mov edx,[dword ss:ebp-18]         ;  假码 2=789012  

0040BE20   |.  E8 0354FFFF   call <jmp.&rtl70.System::LStrCmp>  

0040BE25   |.  75 19         jnz short NetPise.0040BE40        ;  注册码 2 不等，跳！  

0040BE27   |.  8B45 F8       mov eax,[dword ss:ebp-8]  

0040BE2A   |.  3B45 EC       cmp eax,[dword ss:ebp-14]  
```



### 代码片段

```
0040BE2D   |.  75 11            jnz short NetPise.0040BE40  
0040BE2F   |.  8B45 F8          mov eax,[dword ss:ebp-8]  
```

> 引用：“主任批复：是作者之过。既用了高档算法，为什么还要明码比较？”  
> ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)  
> ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)  
> ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)

### 版本判断代码

另外，专业版或个人版的判断在这：

```
0040BF10   |.  59             pop ecx  
0040BF11   |.  E8 0253FFFF    call <jmp.&rtl70.System::LStrCat3>     ;在这跟入分析！我没分析，有愿意的可看一下。  
0040BF16   |.  807D FF 00     cmp [byte ss:ebp-1],0  
0040BF1A   |.  74 0F          je short NetPise.0040BF2B  
0040BF1C   |.  8D45 F8        lea eax,[dword ss:ebp-8]  
0040BF1F   |.  BA E8BF4000    mov edx,NetPise.0040BFE8             ;  ASCII "MyPersonalLicense"  
0040BF24   |.  E8 E752FFFF    call <jmp.&rtl70.System::LStrCat>  
0040BF29   |.  EB 0D          jmp short NetPise.0040BF38  
0040BF2B   |>  8D45 F8        lea eax,[dword ss:ebp-8]  
0040BF2E   |.  BA 04C04000    mov edx,NetPise.0040C004             ;  ASCII "myfirstlicense"  
0040BF33   |.  E8 D852FFFF    call <jmp.&rtl70.System::LStrCat>  
```

***

### 算法说明

算法: `f(用户名)=注册码` ====》明码是也！

1. 计算 `md5` 值：
   - 专业版 `md5(用户名+认证号+myfirstlicense)`
   - 或 个人版 `md5(用户名+认证号+myProce...license)`
   - `md5(林海雪原 123myfirstlicense)=6233F0F0BF688CA97FDEE054EFE3557C`
2. 取 `md5` 串首 6 位+未 6 位+认证号：
   - `6233F0-E3557C-123`

***

### 注册机

你的 `MD5` 计算器哪去了?...略去...

> 引用:
>
> * * *
>
> 这时你到看雪学院主页的解密教学中看看 `garfield` 整理的 `MD5` 算法吧。不要惊奇！真是一模一样！
>
> * * *

其实这个算法是错误的，这不是 `MD5` 算法， `Rivest` 他老人家并不是这样设置的 `MD5`，但我发现的部分密码书中都是这样描述的 `MD5`， `王晓云` 也是依照这个错误的算法基础上修改了，`王晓云` 编写了一个现在只能称作类 `MD5` 的算法步骤进行了她的碰撞，同样是由 `WORD` 作为最小的运算单位，每个 `WORD` 用 4 个字节构成。每个字节用两个 16 进制字母表示，一个字节中两个 16 进制字母最高位排在左边，这是 `王晓云` 研究的“碰撞”`MD5` 数据结构，我看 `王` 的这个结构只能称其为 “类 `MD5`”，真正的 `Rivest` 写的 `MD5` 并不是这样的，如果按照正版 `MD5`，`王` 的碰撞条件就先天不足了，还需要补充一些碰撞条件，我在 `bcg` 论坛帖过碰撞条件的补充；

> 引用:
>
> * * *
>
> 引用：“主任批复：是作者之过。既用了高档算法，为什么还要明码比较？”
>
> * * *

明码比较是 `MD5` 类的破坏性加密算法在软件应用中的死穴， `MD5` 一些的加密算法之所以在软件中不流行主要原因就是这点不好处理，比较吧，肯定是明码，不比较，一个钥匙就可通行天下了，所以不是很好处理，因此不推荐将 `MD5` 算法作为运算注册码的核心部分，可以辅助的用，那样应用效果会好一些。

`4dd466d0e1e0cc123afbe504da2462ca`
