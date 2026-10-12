
# 国外也有简单的--PDF to Word v1.4 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-196.htm`](https://www.pediy.com/kssd/pediy07/pediy7-196.htm)

`PDF to Word v1.4` 注册算法分析

日期：2005 年 4 月 30 日  破解人：`Baby2008`

-----------------------------------------------------------

『软件下载地址』: `http://www.verypdf.com/pdf2word/index.html`

`OD` 载入主程序，输入注册信息 `Email:jw6y8@21cn.com`，`Serial:1234567890`，下 `bp GetDlgItemTextA` 中断，确定，`OD` 中断在：

```
77D6AC06 >   8BFF            mov edi,edi                                           ; pdf2rtf.0046A7C4
77D6AC08    55              push ebp
77D6AC09    8BEC            mov ebp,esp
77D6AC0B    FF75 0C         push dword ptr ss:[ebp+C]
77D6AC0E    FF75 08         push dword ptr ss:[ebp+8]
77D6AC11    E8 8EA6FBFF     call USER32.GetDlgItem
77D6AC16    85C0            test eax,eax
Alt+F9 返回
00429B94    8BB424 E8000000 mov esi,dword ptr ss:[esp+E8]
00429B9B    68 C8000000     push 0C8
00429BA0    B9 32000000     mov ecx,32
00429BA5    33C0            xor eax,eax
00429BA7    BF FCA64600     mov edi,pdf2rtf.0046A6FC                                ; ASCII "1234567890123456"
00429BAC    68 FCA64600     push pdf2rtf.0046A6FC                                    ; ASCII "1234567890123456"
00429BB1    68 FB030000     push 3FB
00429BB6    56              push esi
00429BB7    F3:AB           rep stos dword ptr es:[edi]
00429BB9    FF15 18874400   call dword ptr ds:[<&USER32.GetDlgItemTextA>]            ; USER32.GetDlgItemTextA
00429BBF    68 FCA64600     push pdf2rtf.0046A6FC                                    ; 返回到这里
00429BC4    E8 F7F7FFFF     call pdf2rtf.004293C0       //关键函数
00429BC9    83C4 04         add esp,4
00429BCC    85C0            test eax,eax
00429BCE    74 39           je short pdf2rtf.00429C09                               ; 注册验证爆破
00429BD0    6A 40           push 40
00429BD2    68 38744600     push pdf2rtf.00467438                                    ; ASCII "Thank you registered"
00429BD7    68 14744600     push pdf2rtf.00467414                                    ; ASCII "Thank you registered pdf2word v1.4."
00429BDC    56              push esi
00429BDD    FF15 20874400   call dword ptr ds:[<&USER32.MessageBoxA>]                ; USER32.MessageBoxA
00429BE3    68 FCA64600     push pdf2rtf.0046A6FC                                    ; ASCII "1234567890123456"
00429BE8    56              push esi
00429BE9    E8 42F9FFFF     call pdf2rtf.00429530
00429BEE    83C4 08         add esp,8
00429BF1    C705 C4A74600 0>mov dword ptr ds:[46A7C4],1
00429BFB    6A 01           push 1
00429BFD    56              push esi
00429BFE    FF15 1C874400   call dword ptr ds:[<&USER32.EndDialog>]                 ; USER32.EndDialog
00429C04    E9 F5010000     jmp pdf2rtf.00429DFE
```

更进关键函数

-----------------------------------------------------------

```
004293C0    83EC 30         sub esp,30
004293C3    55              push ebp
004293C4    56              push esi
004293C5    8B7424 3C       mov esi,dword ptr ss:[esp+3C]                   ; 注册码，记为 Serial
004293C9    57              push edi
004293CA    8A06            mov al,byte ptr ds:[esi]                         ; al=Serial[1]
004293CC    8A4E 01         mov cl,byte ptr ds:[esi+1]                       ; cl=Serial[2]
004293CF    8A56 0E         mov dl,byte ptr ds:[esi+E]                       ; dl=Serial[15]
004293D2    884424 18       mov byte ptr ss:[esp+18],al
004293D6    32C0            xor al,al
004293D8    884C24 30       mov byte ptr ss:[esp+30],cl
004293DC    8A4E 0F         mov cl,byte ptr ds:[esi+F]                       ; Serial[16]
004293DF    884424 19       mov byte ptr ss:[esp+19],al                      ; 0
004293E3    884424 31       mov byte ptr ss:[esp+31],al                      ; 0
004293E7    884424 25       mov byte ptr ss:[esp+25],al                      ; 0
004293EB    884424 0D       mov byte ptr ss:[esp+D],al                       ; 0
004293EF    8A46 02         mov al,byte ptr ds:[esi+2]                       ; Serial[3]
004293F2    3C 24           cmp al,24                                       ; '$'
004293F4    885424 24       mov byte ptr ss:[esp+24],dl                     ; dl=Serial[15]
004293F8    884C24 0C       mov byte ptr ss:[esp+C],cl                      ; cl=Serial[16]
004293FC    74 52           je short pdf2rtf.00429450                       ; 不能跳走,则要求 Serial[3]='$'
004293FE    8B3D 2C864400   mov edi,dword ptr ds:[<&MSVCRT.atoi>]           ; msvcrt.atoi
00429404    8D5424 0C       lea edx,dword ptr ss:[esp+C]
00429408    52              push edx
00429409    FFD7            call edi
0042940B    8BE8            mov ebp,eax                                     ; EAX=atoi(Serial[16])
0042940D    8D4424 1C       lea eax,dword ptr ss:[esp+1C]                   ; Serial[1]
00429411    50              push eax
00429412    FFD7            call edi
00429414    03E8            add ebp,eax                                     ; atoi(Serial[16])+atoi(Serial[1])
00429416    83C4 08         add esp,8
00429419    83FD 0A         cmp ebp,0A
0042941C    74 32           je short pdf2rtf.00429450                       ; atoi(Serial[16])+atoi(Serial[1])=0A
0042941E    8D4C24 24       lea ecx,dword ptr ss:[esp+24]                   ; Serial[15]
00429422    51              push ecx
00429423    FFD7            call edi
00429425    8D5424 34       lea edx,dword ptr ss:[esp+34]                   ; Serial[2]
00429429    8BE8            mov ebp,eax
0042942B    52              push edx
0042942C    FFD7            call edi
0042942E    03E8            add ebp,eax
00429430    83C4 08         add esp,8
00429433    83FD 0A         cmp ebp,0A
00429436    75 18           jnz short pdf2rtf.00429450                      ; atoi(Serial[15])+atoi(Serial[2])=0A
00429438    807E 03 24      cmp byte ptr ds:[esi+3],24                      ; Serial[4]='$'
0042943C    75 12           jnz short pdf2rtf.00429450
0042943E    8A4E 05         mov cl,byte ptr ds:[esi+5]                      ; Serial[6]
00429441    33C0            xor eax,eax
00429443    80F9 23         cmp cl,23                                       ; '#'
00429446    5F              pop edi
00429447    5E              pop esi
00429448    5D              pop ebp
00429449    0F94C0          sete al
0042944C    83C4 30         add esp,30
0042944F    C3              retn
```

-----------------------------------------------------------

『算法总结』：

1、注册码长度至少 16 位，记为 `Serial`；

2、在 `004293FC` 处 要求 `Serial[3]='$'`；

3、在 `0042941C` 处 要求 `atoi(Serial[16])+atoi(Serial[1])=$0A`；

4、在 `00429436` 处 要求 `atoi(Serial[15])+atoi(Serial[2])=$0A`；

5、在 `0042943C` 处 要求 `Serial[4]='$'`；

6、在 `00429449` 处 要求 `Serial[6]='#'`；

`Email` 未参加计算，仅仅要求注册码符合一定规律即可注册成功。

将注册试炼码: `1234567890123456`

按照要求修改一下：`12$$5#1234567889`

`$39.95USD` 就采用这样的保护？郁闷中 ing……

该系列软件采用的保护方法基本差不多，下面是

`VeryPDF PDF Editor v1.1` `$89.95USD` 中的注册验证代码：

```
00407800    83EC 30         sub esp,30
00407803    56              push esi
00407804    8B7424 38       mov esi,dword ptr ss:[esp+38]
00407808    57              push edi
00407809    8A06            mov al,byte ptr ds:[esi]
0040780B    8A4E 01         mov cl,byte ptr ds:[esi+1]
0040780E    8A56 0E         mov dl,byte ptr ds:[esi+E]
00407811    884424 14       mov byte ptr ss:[esp+14],al
00407815    32C0            xor al,al
00407817    884C24 2C       mov byte ptr ss:[esp+2C],cl
0040781B    8A4E 0F         mov cl,byte ptr ds:[esi+F]
0040781E    884424 15       mov byte ptr ss:[esp+15],al
00407822    884424 2D       mov byte ptr ss:[esp+2D],al
00407826    884424 21       mov byte ptr ss:[esp+21],al
```



```
0040782A    884424 09        mov byte ptr ss:[esp+9],al  

0040782E    8A46 02         mov al,byte ptr ds:[esi+2]  

00407831    3C 50         cmp al,50  

00407833    885424 20     mov byte ptr ss:[esp+20],dl  

00407837    884C24 08     mov byte ptr ss:[esp+8],cl  

0040783B    75 57         jnz short pdfedito.00407894  

0040783D    8D5424 08     lea edx,dword ptr ss:[esp+8]  

00407841    52            push edx  

00407842    E8 88650100   call pdfedito.0041DDCF  

00407847    8BF8          mov edi,eax  

00407849    8D4424 18     lea eax,dword ptr ss:[esp+18]  

0040784D    50            push eax  

0040784E    E8 7C650100   call pdfedito.0041DDCF  

00407853    03F8          add edi,eax  

00407855    83C4 08       add esp,8  

00407858    83FF 0A       cmp edi,0A  

0040785B    75 37         jnz short pdfedito.00407894  

0040785D    8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

00407861    51            push ecx  

00407862    E8 68650100   call pdfedito.0041DDCF  

00407867    8D5424 30     lea edx,dword ptr ss:[esp+30]  

0040786B    8BF8          mov edi,eax  

0040786D    52            push edx  

0040786E    E8 5C650100   call pdfedito.0041DDCF  

00407873    03F8          add edi,eax  

00407875    83C4 08       add esp,8  

00407878    83FF 0A       cmp edi,0A  

0040787B    75 17         jnz short pdfedito.00407894  

0040787D    807E 03 4D    cmp byte ptr ds:[esi+3],4D  

00407881    75 11         jnz short pdfedito.00407894  

00407883    8A4E 05       mov cl,byte ptr ds:[esi+5]  

00407886    33C0          xor eax,eax  

00407888    80F9 57       cmp cl,57  

0040788B    5F            pop edi  

0040788C    0F94C0        sete al  

0040788F    5E            pop esi  

00407890    83C4 30       add esp,30  

00407893    C3            retn  

00407894    5F            pop edi  

00407895    33C0          xor eax,eax  

00407897    5E            pop esi  

00407898    83C4 30       add esp,30  

0040789B    C3            retn
```

`PDF2TXT(PDF To TEXT) v3.1` `$38USD`

```
00371170    83EC 18       sub esp,18  

00371173    53            push ebx  

00371174    56            push esi  

00371175    8B7424 24     mov esi,dword ptr ss:[esp+24]                 ; 注册码  

00371179    8D5424 08     lea edx,dword ptr ss:[esp+8]  

0037117D    57            push edi  

0037117E    32DB          xor bl,bl  

00371180    8A46 0E       mov al,byte ptr ds:[esi+E]  

00371183    8A4E 0F       mov cl,byte ptr ds:[esi+F]  

00371186    52            push edx  

00371187    884424 1C     mov byte ptr ss:[esp+1C],al  

0037118B    885C24 1D     mov byte ptr ss:[esp+1D],bl  

0037118F    884C24 10     mov byte ptr ss:[esp+10],cl  

00371193    885C24 11     mov byte ptr ss:[esp+11],bl  

00371197    E8 35160000   call verypdf.003727D1  

0037119C    8BF8          mov edi,eax  

0037119E    8D4424 1C     lea eax,dword ptr ss:[esp+1C]  

003711A2    50            push eax  

003711A3    E8 29160000   call verypdf.003727D1  

003711A8    03F8          add edi,eax  

003711AA    83C4 08       add esp,8  

003711AD    83FF 0B       cmp edi,0B                                       ; 15+16=11  

003711B0    74 09         je short verypdf.003711BB  

003711B2    5F            pop edi  

003711B3    5E            pop esi  

003711B4    33C0          xor eax,eax  

003711B6    5B            pop ebx  

003711B7    83C4 18       add esp,18  

003711BA    C3            retn  

003711BB    8A0E          mov cl,byte ptr ds:[esi]                           ; 1  

003711BD    8A56 01       mov dl,byte ptr ds:[esi+1]                         ; 2  

003711C0    8D4424 0C     lea eax,dword ptr ss:[esp+C]  

003711C4    884C24 18     mov byte ptr ss:[esp+18],cl  

003711C8    50            push eax  

003711C9    885C24 1D     mov byte ptr ss:[esp+1D],bl  

003711CD    885424 10     mov byte ptr ss:[esp+10],dl  

003711D1    885C24 11     mov byte ptr ss:[esp+11],bl  

003711D5    E8 F7150000   call verypdf.003727D1  

003711DA    8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  

003711DE    8BF8          mov edi,eax  

003711E0    51            push ecx  

003711E1    E8 EB150000   call verypdf.003727D1  

003711E6    03F8          add edi,eax  

003711E8    83C4 08       add esp,8  

003711EB    83FF 08       cmp edi,8                                        ; 1+2=8  

003711EE    74 09         je short verypdf.003711F9  

003711F0    5F            pop edi  

003711F1    5E            pop esi  

003711F2    33C0          xor eax,eax  

003711F4    5B            pop ebx  

003711F5    83C4 18       add esp,18  

003711F8    C3            retn  

003711F9    8A4E 05       mov cl,byte ptr ds:[esi+5]                         ; 6  

003711FC    B0 24         mov al,24                                        ; $  

003711FE    3AC8          cmp cl,al  

00371200    74 09         je short verypdf.0037120B                        ; 6=$  

00371202    5F            pop edi  

00371203    5E            pop esi  

00371204    33C0          xor eax,eax  

00371206    5B            pop ebx  

00371207    83C4 18       add esp,18  

0037120A    C3            retn  

0037120B    3846 06       cmp byte ptr ds:[esi+6],al                       ; 7  

0037120E    74 09         je short verypdf.00371219                        ; 7=$  

00371210    5F            pop edi  

00371211    5E            pop esi  

00371212    33C0          xor eax,eax  

00371214    5B            pop ebx  

00371215    83C4 18       add esp,18  

00371218    C3            retn  

00371219    8A4E 07       mov cl,byte ptr ds:[esi+7]                       ; 8  

0037121C    33C0          xor eax,eax  

0037121E    80F9 40       cmp cl,40                                        ; @  

00371221    5F            pop edi  

00371222    5E            pop esi  

00371223    5B            pop ebx  

00371224    0F94C0        sete al                                          ; 8=@  

00371227    83C4 18       add esp,18  

0037122A    C3            retn
```

`4dd466d0e1e0cc123afbe504da2462ca`
