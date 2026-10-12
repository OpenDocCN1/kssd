
# 超级通讯王 1.83

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-207.htm`](https://www.pediy.com/kssd/pediy07/pediy7-207.htm)

日期：2005 年 5 月 6 日
破解人：`yijun[PYG]`

【软件名称】：超级通讯王
【软件版本】：1.83
【软件大小】：2.45M
【软件限制】：30 次使用限制
【破解声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！
【破解工具】：`OD`, `peid`，`c32`

【破解过程】：

`peid` 查知道该软件无壳，`Microsoft Visual C++ 7.0` 编写。用 `C32` 可查到 4 个错误提示和 2 个正确提示，错误分别是：`0041fc24` `0045ac34` `0045b58f` `004a9c53`，正确分别是：`0041fb99` `004c0793`。`OD` 载入，把 4 个错误都下断点，在 `0041fc24` 断下，往上一直到以下地方：

```asm
0041FB00    6A FF            push -1  //在此下断
0041FB02    68 A8BE4D00      push SuperPIM.004DBEA8
0041FB07    64:A1 00000000   mov eax,dword ptr fs:[0]
0041FB0D    50               push eax
0041FB0E    64:8925 0000000>mov dword ptr fs:[0],esp
0041FB15    83EC 08          sub esp,8
0041FB18    53               push ebx
0041FB19    56               push esi
0041FB1A    57               push edi
0041FB1B    6A 01            push 1
0041FB1D    8BF1             mov esi,ecx
0041FB1F    E8 DC530800      call SuperPIM.004A4F00          ; 跟进
0041FB24    8D4424 0C        lea eax,dword ptr ss:[esp+C]     ; 跳回来
0041FB28    50               push eax
0041FB29    E8 322A0400      call SuperPIM.00462560          ; 处理机器码，我的是机器码是 U2L2T26FBW01N25，跟进~~~~~~~
0041FB2E    83C4 04          add esp,4                        ; ESP+4 送 ESP
0041FB31    8B4E 74          mov ecx,dword ptr ds:[esi+74]    ; 假码到 ECX
0041FB34    8B41 F4          mov eax,dword ptr ds:[ecx-C]      ; 假码位数送 EAX
0041FB37    85C0             test eax,eax
0041FB39    8D7E 74          lea edi,dword ptr ds:[esi+74]    ; 假码地址送 EDI
0041FB3C    C74424 1C 00000>mov dword ptr ss:[esp+1C],0
0041FB44    0F84 D6000000    je SuperPIM.0041FC20              ; 相等就跳
0041FB4A    6A 28            push 28
0041FB4C    8D5424 14        lea edx,dword ptr ss:[esp+14]
0041FB50    57               push edi
0041FB51    52               push edx
0041FB52    E8 B97E0300      call SuperPIM.00457A10          ; 处理注册码，和处理机器码的是差不多的~~~~~~~~
0041FB57    8B4C24 18        mov ecx,dword ptr ss:[esp+18]    ; 串 1 送 ECX
0041FB5B    8B00             mov eax,dword ptr ds:[eax]        ; 串 2 送 EAX
0041FB5D    51               push ecx
0041FB5E    50               push eax
0041FB5F    E8 790D0700      call SuperPIM.004908DD          ; 关键 CALL
0041FB64    8BD8             mov ebx,eax
0041FB66    8B4424 24        mov eax,dword ptr ss:[esp+24]    ; 串 2 送 EAX
0041FB6A    83C4 14          add esp,14                       ; ESP+14 送 ESP
0041FB6D    F7DB             neg ebx                            ; EBX 取反
0041FB6F    1ADB             sbb bl,bl                          ; 带借位减
0041FB71    83C0 F0          add eax,-10                        ; EAX-10 送 EAX
0041FB74    FEC3             inc bl                              ; BL+1
0041FB76    8D50 0C          lea edx,dword ptr ds:[eax+C]
0041FB79    83C9 FF          or ecx,FFFFFFFF
0041FB7C    F0:0FC10A        lock xadd dword ptr ds:[edx],ecx
0041FB80    49               dec ecx
0041FB81    85C9             test ecx,ecx
0041FB83    7F 08            jg short SuperPIM.0041FB8D
0041FB85    8B08             mov ecx,dword ptr ds:[eax]
0041FB87    8B11             mov edx,dword ptr ds:[ecx]
0041FB89    50               push eax
0041FB8A    FF52 04          call dword ptr ds:[edx+4]
0041FB8D    84DB             test bl,bl                          ; BL 是否为 0
0041FB8F    0F84 8B000000    je SuperPIM.0041FC20              ; 关键跳，跳就 OVER
0041FB95    8D4424 10        lea eax,dword ptr ss:[esp+10]
0041FB99    6A 71            push 71
0041FB9B    50               push eax
0041FB9C    E8 AF270400      call SuperPIM.00462350
0041FBA1    83C4 08          add esp,8
0041FBA4    8B00             mov eax,dword ptr ds:[eax]
0041FBA6    6A 00            push 0
0041FBA8    6A 00            push 0
0041FBAA    50               push eax
0041FBAB    8BCE             mov ecx,esi
0041FBAD    C64424 28 01     mov byte ptr ss:[esp+28],1
0041FBB2    E8 0A500800      call SuperPIM.004A4BC1
0041FBB7    8B4424 10        mov eax,dword ptr ss:[esp+10]
0041FBBB    83C0 F0          add eax,-10
0041FBBE    C64424 1C 00     mov byte ptr ss:[esp+1C],0
0041FBC3    8D48 0C          lea ecx,dword ptr ds:[eax+C]
0041FBC6    83CA FF          or edx,FFFFFFFF
0041FBC9    F0:0FC111        lock xadd dword ptr ds:[ecx],edx
0041FBCD    4A               dec edx
0041FBCE    85D2             test edx,edx
0041FBD0    7F 08            jg short SuperPIM.0041FBDA
0041FBD2    8B08             mov ecx,dword ptr ds:[eax]
0041FBD4    8B11             mov edx,dword ptr ds:[ecx]
0041FBD6    50               push eax
0041FBD7    FF52 04          call dword ptr ds:[edx+4]
0041FBDA    E8 CF470A00      call SuperPIM.004C43AE
0041FBDF    8B40 04          mov eax,dword ptr ds:[eax+4]
0041FBE2    6A 01            push 1
0041FBE4    57               push edi
0041FBE5    C780 28040000 0>mov dword ptr ds:[eax+428],1
0041FBEF    E8 8C7D0300      call SuperPIM.00457980
0041FBF4    8B3F             mov edi,dword ptr ds:[edi]
0041FBF6    8B5F F4          mov ebx,dword ptr ds:[edi-C]
0041FBF9    83C4 08          add esp,8
0041FBFC    E8 AD470A00      call SuperPIM.004C43AE
0041FC01    8B40 04          mov eax,dword ptr ds:[eax+4]
0041FC04    53               push ebx
0041FC05    57               push edi
0041FC06    68 28D74E00      push SuperPIM.004ED728          ; ASCII "rc"
0041FC0B    68 926C4E00      push SuperPIM.004E6C92
0041FC10    8BC8             mov ecx,eax
0041FC12    E8 B61E0900      call SuperPIM.004B1ACD
0041FC17    8BCE             mov ecx,esi
0041FC19    E8 7C850800      call SuperPIM.004A819A
0041FC1E    EB 45            jmp short SuperPIM.0041FC65
0041FC20    8D4424 10        lea eax,dword ptr ss:[esp+10]
0041FC24    6A 72            push 72     //从此往上~~~~~~~~~
0041FC26    50               push eax
0041FC27    E8 24270400      call SuperPIM.00462350
0041FC2C    83C4 08          add esp,8
0041FC2F    8B00             mov eax,dword ptr ds:[eax]
0041FC31    6A 00            push 0
0041FC33    6A 00            push 0
0041FC35    50               push eax
0041FC36    8BCE             mov ecx,esi
0041FC38    C64424 28 02     mov byte ptr ss:[esp+28],2
0041FC3D    E8 7F4F0800      call SuperPIM.004A4BC1
0041FC42    8B4424 10        mov eax,dword ptr ss:[esp+10]
0041FC46    83C0 F0          add eax,-10
0041FC49    C64424 1C 00     mov byte ptr ss:[esp+1C],0
0041FC4E    8D48 0C          lea ecx,dword ptr ds:[eax+C]
0041FC51    83CA FF          or edx,FFFFFFFF
0041FC54    F0:0FC111        lock xadd dword ptr ds:[ecx],edx
0041FC58    4A               dec edx
0041FC59    85D2             test edx,edx
0041FC5B    7F 08            jg short SuperPIM.0041FC65
```

跟进 `0041FB1F` 处 `CALL` 来到：

```asm
004A4F00    B8 2F364E00      mov eax,SuperPIM.004E362F
004A4F05    E8 E6DBFEFF      call SuperPIM.00492AF0
004A4F0A    83EC 24          sub esp,24
004A4F0D    8B45 08          mov eax,dword ptr ss:[ebp+8]
004A4F10    53               push ebx
004A4F11    56               push esi
004A4F12    57               push edi
004A4F13    8BF1             mov esi,ecx
004A4F15    33DB             xor ebx,ebx
004A4F17    8965 F0          mov dword ptr ss:[ebp-10],esp
004A4F1A    8945 D4          mov dword ptr ss:[ebp-2C],eax
004A4F1D    8975 D8          mov dword ptr ss:[ebp-28],esi
```



# 逆向分析记录

## 片段 1

```
004A4F20    895D DC         mov dword ptr ss:[ebp-24],ebx  
004A4F23    E8 76F40100     call SuperPIM.004C439E  
004A4F28    8BF8            mov edi,eax  
004A4F2A    8D87 DC000000   lea eax,dword ptr ds:[edi+DC]  
004A4F30    8B08            mov ecx,dword ptr ds:[eax]  
004A4F32    894D E8         mov dword ptr ss:[ebp-18],ecx  
004A4F35    8B4E 1C         mov ecx,dword ptr ds:[esi+1C]  
004A4F38    8908            mov dword ptr ds:[eax],ecx  
004A4F3A    8B06            mov eax,dword ptr ds:[esi]  
004A4F3C    8D4D D4         lea ecx,dword ptr ss:[ebp-2C]  
004A4F3F    51              push ecx  
004A4F40    8BCE            mov ecx,esi  
004A4F42    897D EC         mov dword ptr ss:[ebp-14],edi  
004A4F45    895D 08         mov dword ptr ss:[ebp+8],ebx  
004A4F48    895D FC         mov dword ptr ss:[ebp-4],ebx  
004A4F4B    FF90 F4000000   call dword ptr ds:[eax+F4]               ; 跟进  
004A4F51    C745 08 0100000>mov dword ptr ss:[ebp+8],1  
004A4F58    EB 27           jmp short SuperPIM.004A4F81  
004A4F5A    B8 7E4F4A00     mov eax,SuperPIM.004A4F7E  
004A4F5F    C3              retn  
```

---

## 跟进 `004A4F4B` 处 `CALL` 来到

```
0041FA50    56              push esi  
0041FA51    57              push edi  
0041FA52    8B7C24 0C       mov edi,dword ptr ss:[esp+C]  
0041FA56    8BF1            mov esi,ecx  
0041FA58    8D46 70         lea eax,dword ptr ds:[esi+70]  
0041FA5B    50              push eax  
0041FA5C    68 EE030000     push 3EE  
0041FA61    57              push edi  
0041FA62    E8 6D490900     call SuperPIM.004B43D4                 ; 取机器码，跟进~~~~~~~~  
0041FA67    83C6 74         add esi,74  
0041FA6A    56              push esi  
0041FA6B    68 EF030000     push 3EF  
0041FA70    57              push edi  
0041FA71    E8 5E490900     call SuperPIM.004B43D4                 ; 取假码，和上面是同一个 CALL  
0041FA76    5F              pop edi  
0041FA77    5E              pop esi  
0041FA78    C2 0400         retn 4  
```

---

## 跟进 `0041FA62` 处 `CALL` 来到

```
004B43D4    55              push ebp  
004B43D5    8BEC            mov ebp,esp  
004B43D7    56              push esi  
004B43D8    57              push edi  
004B43D9    8B7D 08         mov edi,dword ptr ss:[ebp+8]  
004B43DC    FF75 0C         push dword ptr ss:[ebp+C]  
004B43DF    8BCF            mov ecx,edi  
004B43E1    E8 FDFdFFFF     call SuperPIM.004B41E3  
004B43E6    833F 00         cmp dword ptr ds:[edi],0  
004B43E9    8BF0            mov esi,eax  
004B43EB    74 28           je short SuperPIM.004B4415  
004B43ED    56              push esi  
004B43EE    FF15 88664E00   call dword ptr ds:[<&USER32.GetWindowTex>; USER32.GetWindowTextLengthA  
004B43F4    8D48 01         lea ecx,dword ptr ds:[eax+1]  
004B43F7    51              push ecx  
004B43F8    8B4D 10         mov ecx,dword ptr ss:[ebp+10]  
004B43FB    50              push eax  
004B43FC    E8 6FF9FBFF     call SuperPIM.00473D70  
004B4401    50              push eax  
004B4402    56              push esi  
004B4403    FF15 8C664E00   call dword ptr ds:[<&USER32.GetWindowTex>; 测试假码长度送 EAX  
004B4409    8B4D 10         mov ecx,dword ptr ss:[ebp+10]  
004B440C    6A FF           push -1  
004B440E    E8 3DE1F4FF     call SuperPIM.00402550                 ; 处理假码，跟进看看~~~~~~  
004B4413    EB 0B           jmp short SuperPIM.004B4420  
004B4415    8B45 10         mov eax,dword ptr ss:[ebp+10]  
004B4418    FF30            push dword ptr ds:[eax]  
004B441A    56              push esi  
004B441B    E8 C3C8FFFF     call SuperPIM.004B0CE3  
004B4420    5F              pop edi  
004B4421    5E              pop esi  
004B4422    5D              pop ebp  
004B4423    C2 0C00         ret 0C  
```

---

## 跟进 `004B440E` 处 `CALL` 来到

```
00402550    8B4424 04       mov eax,dword ptr ss:[esp+4]  
00402554    83F8 FF         cmp eax,-1  
00402557    75 21           jnz short SuperPIM.0040257A  
00402559    8B01            mov eax,dword ptr ds:[ecx]             ; 假码/机器码送 EAX  
0040255B    85C0            test eax,eax                             ; 是否为空  
0040255D    75 0D           jnz short SuperPIM.0040256C             ; 不是就跳  
0040255F    8BD0            mov edx,eax  
00402561    8942 F4         mov dword ptr ds:[edx-C],eax  
00402564    8B09            mov ecx,dword ptr ds:[ecx]  
00402566    880408          mov byte ptr ds:[eax+ecx],al  
00402569    C2 0400         ret 4  
0040256C    56              push esi  
0040256D    8D70 01         lea esi,dword ptr ds:[eax+1]           ; 从[EAX+1]送 ESI，即舍掉假码第一位~~  
00402570    8A10            mov dl,byte ptr ds:[eax]               ; 假码逐位送 DL  
00402572    40              inc eax                                 ; EAX+1 送 EAX  
00402573    84D2            test dl,dl                              ; DL 是否为 0  
00402575  ^ 75 F9          jnz short SuperPIM.00402570             ; 不是就继续,直到取完  
00402577    2BC6            sub eax,esi                             ; EAX-ESI 送 EAX  
00402579    5E              pop esi  
0040257A    8B11            mov edx,dword ptr ds:[ecx]              ; 假码送 EDX  
0040257C    8942 F4         mov dword ptr ds:[edx-C],eax  
0040257F    8B09            mov ecx,dword ptr ds:[ecx]              ; 假码送 ECX  
00402581    C60408 00       mov byte ptr ds:[eax+ecx],0  
00402585    C2 0400         ret 4                                     //跳出去~~~~~~~~  
```

---

## 跟进 `0041FB29` 处 `CALL` 来到

```
00462560    6A FF           push -1  
00462562    68 000B4E00     push SuperPIM.004E0B00  
00462567    64:A1 00000000  mov eax,dword ptr fs:[0]  
0046256D    50              push eax  
0046256E    64:8925 0000000>mov dword ptr fs:[0],esp  
00462575    83EC 08         sub esp,8  
00462578    8D4424 04       lea eax,dword ptr ss:[esp+4]  
0046257C    56              push esi  
0046257D    50              push eax  
0046257E    C74424 0C 00000>mov dword ptr ss:[esp+C],0  
00462586    E8 65FEFFFF     call SuperPIM.004623F0  
0046258B    6A 14           push 14  
0046258D    8D4C24 10       lea ecx,dword ptr ss:[esp+10]  
00462591    51              push ecx  
00462592    8D5424 10       lea edx,dword ptr ss:[esp+10]  
00462596    52              push edx  
00462597    C74424 24 00000>mov dword ptr ss:[esp+24],0  
0046259F    E8 6C54FFFF     call SuperPIM.00457A10                  //该 CALL 由机器码计算出一个字符串，我的是"R15LE8CX7EXAB3LQ5I2H"，跟进  
004625A4    8B7424 2C       mov esi,dword ptr ss:[esp+2C]  
004625A8    6A 28           push 28  
004625AA    8D4424 18       lea eax,dword ptr ss:[esp+18]  
004625AE    50              push eax  
004625AF    56              push esi  
004625B0    C64424 30 01    mov byte ptr ss:[esp+30],1  
004625B5    E8 5654FFFF     call SuperPIM.00457A10                  ; 计算串 1，和上面的是同一个 CALL  
004625BA    8B4424 20       mov eax,dword ptr ss:[esp+20]           ; "R15LE8CX7EXAB3LQ5I2H"送 EAX  
004625BE    83C0 F0         add eax,-10                              ; EAX-10 送 EAX  
004625C1    83C4 1C         add esp,1C                               ; ESP+1C 送 ESP  
004625C4    C64424 14 00    mov byte ptr ss:[esp+14],0  
004625C9    8D48 0C         lea ecx,dword ptr ds:[eax+C]  
004625CC    83CA FF         or edx,FFFFFFFF  
004625CF    F0:0FC111       lock xadd dword ptr ds:[ecx],edx  
004625D3    4A              dec edx  
004625D4    85D2            test edx,edx  
004625D6    7F 08           jg short SuperPIM.004625E0  
```



```
004625D8    8B08            mov ecx,dword ptr ds:[eax]

004625DA    8B11            mov edx,dword ptr ds:[ecx]

004625DC    50              push eax

004625DD    FF52 04         call dword ptr ds:[edx+4]

004625E0    8B4424 08       mov eax,dword ptr ss:[esp+8]          ; 机器码送 EAX

004625E4    83C0 F0         add eax,-10                             ; EAX-10 送 EAX

004625E7    C74424 14 FFFFF>mov dword ptr ss:[esp+14],-1

004625EF    8D48 0C         lea ecx,dword ptr ds:[eax+C]

004625F2    83CA FF         or edx,FFFFFFFF

004625F5    F0:0FC111       lock xadd dword ptr ds:[ecx],edx

004625F9    4A              dec edx

004625FA    85D2            test edx,edx

004625FC    7F 08           jg short SuperPIM.00462606

004625FE    8B08            mov ecx,dword ptr ds:[eax]

00462600    8B11            mov edx,dword ptr ds:[ecx]

00462602    50              push eax

00462603    FF52 04         call dword ptr ds:[edx+4]

00462606    8B4C24 0C       mov ecx,dword ptr ss:[esp+C]

0046260A    8BC6            mov eax,esi

0046260C    5E              pop esi

0046260D    64:890D 0000000>mov dword ptr fs:[0],ecx

00462614    83C4 14         add esp,14

00462617    C3              retn
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

跟进 `0046259F` 处 `CALL` 来到：

```
00457A10    6A FF           push -1

00457A12    68 88FE4D00     push SuperPIM.004DFE88

00457A17    64:A1 00000000  mov eax,dword ptr fs:[0]

00457A1D    50              push eax

00457A1E    64:8925 0000000>mov dword ptr fs:[0],esp

00457A25    83EC 14         sub esp,14

00457A28    55              push ebp

00457A29    56              push esi

00457A2A    33ED            xor ebp,ebp

00457A2C    57              push edi

00457A2D    896C24 1C       mov dword ptr ss:[esp+1C],ebp

00457A31    E8 A4C00400     call SuperPIM.004A3ADA

00457A36    8B10            mov edx,dword ptr ds:[eax]

00457A38    8BC8            mov ecx,eax

00457A3A    FF52 0C         call dword ptr ds:[edx+C]

00457A3D    8D70 10         lea esi,dword ptr ds:[eax+10]

00457A40    897424 0C       mov dword ptr ss:[esp+C],esi

00457A44    8B4424 34       mov eax,dword ptr ss:[esp+34]

00457A48    8B08            mov ecx,dword ptr ds:[eax]          ; 机器码/"R15LE8CX7EXAB3LQ5I2H"/假码送 ECX

00457A4A    8B49 F4         mov ecx,dword ptr ds:[ecx-C]       ; 该串长度送 ECX

00457A4D    3BCD            cmp ecx,ebp

00457A4F    896C24 28       mov dword ptr ss:[esp+28],ebp

00457A53    894C24 14       mov dword ptr ss:[esp+14],ecx

00457A57    75 48           jnz short SuperPIM.00457AA1

00457A59    83C6 F0         add esi,-10

00457A5C    56              push esi

00457A5D    E8 0EACFAFF     call SuperPIM.00402670

00457A62    8B7C24 34       mov edi,dword ptr ss:[esp+34]

00457A66    83C0 10         add eax,10

00457A69    8907            mov dword ptr ds:[edi],eax

00457A6B    83C4 04         add esp,4

00457A6E    C74424 28 FFFFF>mov dword ptr ss:[esp+28],-1

00457A76    8D56 0C         lea edx,dword ptr ds:[esi+C]

00457A79    83C8 FF         or eax,FFFFFFFF

00457A7C    F0:0FC102       lock xadd dword ptr ds:[edx],eax

00457A80    48              dec eax

00457A81    85C0            test eax,eax

00457A83    7F 08           jg short SuperPIM.00457A8D

00457A85    8B0E            mov ecx,dword ptr ds:[esi]

00457A87    8B11            mov edx,dword ptr ds:[ecx]

00457A89    56              push esi

00457A8A    FF52 04         call dword ptr ds:[edx+4]

00457A8D    8BC7            mov eax,edi

00457A8F    5F              pop edi

00457A90    5E              pop esi

00457A91    5D              pop ebp

00457A92    8B4C24 14       mov ecx,dword ptr ss:[esp+14]

00457A96    64:890D 0000000>mov dword ptr fs:[0],ecx

00457A9D    83C4 20         add esp,20

00457AA0    C3              retn

00457AA1    8B7C24 38       mov edi,dword ptr ss:[esp+38]

00457AA5    3BF9            cmp edi,ecx

00457AA7    897C24 10       mov dword ptr ss:[esp+10],edi

00457AAB    7F 04           jg short SuperPIM.00457AB1

00457AAD    894C24 10       mov dword ptr ss:[esp+10],ecx

00457AB1    396C24 10       cmp dword ptr ss:[esp+10],ebp

00457AB5    53              push ebx

00457AB6    0F8E E8000000   jle SuperPIM.00457BA4

00457ABC    EB 08           jmp short SuperPIM.00457AC6

00457ABE    8B4C24 18       mov ecx,dword ptr ss:[esp+18]

00457AC2    8B7C24 3C       mov edi,dword ptr ss:[esp+3C]

00457AC6    8BC5            mov eax,ebp

00457AC8    99              cdq                                 ; 清 EDX

00457AC9    F7F9            idiv ecx

00457ACB    8B4424 38       mov eax,dword ptr ss:[esp+38]

00457ACF    8B08            mov ecx,dword ptr ds:[eax]          ; 机器码/"R15LE8CX7EXAB3LQ5I2H"/假码送 ECX

00457AD1    55              push ebp

00457AD2    8A140A          mov dl,byte ptr ds:[edx+ecx]        ; 逐位取该串到 DL

00457AD5    885424 20       mov byte ptr ss:[esp+20],dl

00457AD9    8B4424 20       mov eax,dword ptr ss:[esp+20]       ; [ESP+20]送 EAX

00457ADD    50              push eax

00457ADE    E8 8DFDFFFF     call SuperPIM.00457870

00457AE3    33D2            xor edx,edx                           ; EDX 清 0

00457AE5    B9 2B000000     mov ecx,2B                            ; 2B 送 ECX

00457AEA    F7F1            div ecx                               ; 除以 ECX

00457AEC    83C4 08         add esp,8

00457AEF    8BDA            mov ebx,edx                             ; EDX 送 EBX

00457AF1    80C3 30         add bl,30                               ; BL+30 送 BL

00457AF4    80FB 39         cmp bl,39                               ; 和 39 比较

00457AF7    7E 08           jle short SuperPIM.00457B01           ; 小于就跳

00457AF9    80FB 41         cmp bl,41                               ; 和 41 比较

00457AFC    7D 03           jge short SuperPIM.00457B01           ; 大于就跳

00457AFE    80C3 F6         add bl,0F6                              ; BL+F6 送 BL

00457B01    3BEF            cmp ebp,edi                             ; 和 EDI 比较

00457B03    7D 3B           jge short SuperPIM.00457B40           ; 大于等于就跳

00457B05    8B46 FC         mov eax,dword ptr ds:[esi-4]          ; [ESI-4]送 EAX

00457B08    8B56 F4         mov edx,dword ptr ds:[esi-C]          ; [ESI-C]送 EDX

00457B0B    B9 01000000     mov ecx,1

00457B10    2BC8            sub ecx,eax                           ; ECX-EAX 送 ECX

00457B12    8B46 F8         mov eax,dword ptr ds:[esi-8]          ; [ESI-8]送 EAX

00457B15    8D7A 01         lea edi,dword ptr ds:[edx+1]          ; [EDX+1]的地址送 EDI

00457B18    2BC7            sub eax,edi                           ; EAX-EDI 送 EAX

00457B1A    0BC1            or eax,ecx                            ; EAX 与 ECX 或送 EAX

00457B1C    895424 20       mov dword ptr ss:[esp+20],edx

00457B20    7D 12           jge short SuperPIM.00457B34

00457B22    57              push edi

00457B23    8D4C24 14       lea ecx,dword ptr ss:[esp+14]

00457B27    E8 F4AAFAFF     call SuperPIM.00402620

00457B2C    8B7424 10       mov esi,dword ptr ss:[esp+10]

00457B30    8B5424 20       mov edx,dword ptr ss:[esp+20]

00457B34    881C32          mov byte ptr ds:[edx+esi],bl          ; BL 送[edx+esi]

00457B37    897E F4         mov dword ptr ds:[esi-C],edi

00457B3A    C60437 00       mov byte ptr ds:[edi+esi],0

00457B3E    EB 57           jmp short SuperPIM.00457B97

00457B40    8BC5            mov eax,ebp

00457B42    99              cdq

00457B43    F7FF            idiv edi

00457B45    0FBEC3          movsx eax,bl

00457B48    B9 2B000000     mov ecx,2B
```



## 汇编代码段 1

```asm
00457B4D    8BFA                 mov edi,edx  

00457B4F    0FBE1437             movsx edx,byte ptr ds:[edi+esi]  

00457B53    8D4402 A0            lea eax,dword ptr ds:[edx+eax-60]  

00457B57    99                   cdq  

00457B58    F7F9                 idiv ecx  

00457B5A    8BDA                 mov ebx,edx  

00457B5C    80C3 30              add bl,30  

00457B5F    80FB 39              cmp bl,39  

00457B62    7E 08                jle short SuperPIM.00457B6C  

00457B64    80FB 41              cmp bl,41  

00457B67    7D 03                jge short SuperPIM.00457B6C  

00457B69    80C3 F6              add bl,0F6  

00457B6C    837E FC 01           cmp dword ptr ds:[esi-4],1  

00457B70    8B46 F4              mov eax,dword ptr ds:[esi-C]  

00457B73    894424 20            mov dword ptr ss:[esp+20],eax  

00457B77    7E 14                jle short SuperPIM.00457B8D  

00457B79    8BD0                 mov edx,eax  

00457B7B    52                   push edx  

00457B7C    8D4C24 14            lea ecx,dword ptr ss:[esp+14]  

00457B80    E8 CBA3FAFF          call SuperPIM.00401F50  

00457B85    8B7424 10            mov esi,dword ptr ss:[esp+10]  

00457B89    8B4424 20            mov eax,dword ptr ss:[esp+20]  

00457B8D    881C37               mov byte ptr ds:[edi+esi],bl  

00457B90    8946 F4              mov dword ptr ds:[esi-C],eax  

00457B93    C60430 00            mov byte ptr ds:[eax+esi],0  

00457B97    8B4424 14            mov eax,dword ptr ss:[esp+14]                ; 给 EAX 赋值 28（16 进制）  

00457B9B    45                   inc ebp                                        ; EBP+1 送 EBP  

00457B9C    3BE8                 cmp ebp,eax                                    ; 和 EAX 比较  

00457B9E    0F8C 1AFFFFFF        jl SuperPIM.00457ABE                           ; 小于就跳，算得字符串保存在 ESI  

00457BA4    8B4E F0              mov ecx,dword ptr ds:[esi-10]                 ; 计算机器码最终保存到 ESI，计算机器码得 R15LE8CX7EXAB3LQ5I2H，计算 R15LE8CX7EXAB3LQ5I2H 得 ESI=1QWSVWY80NAWU4BP062PB91VEJS7TXX3U2C9145V，记为串 1，计算假码结果为 QQB2PW8FSAM6VCN515YSBP261Z4UK36CJUGU10WE，机为串 2  

00457BA7    8B01                 mov eax,dword ptr ds:[ecx]                    

00457BA9    8D5E F0              lea ebx,dword ptr ds:[esi-10]                 

00457BAC    FF50 10              call dword ptr ds:[eax+10]  

00457BAF    8B53 0C              mov edx,dword ptr ds:[ebx+C]  

00457BB2    85D2                 test edx,edx  

00457BB4    8D4B 0C              lea ecx,dword ptr ds:[ebx+C]  

00457BB7    7C 11                jl short SuperPIM.00457BCA  

00457BB9    3B03                 cmp eax,dword ptr ds:[ebx]  

00457BBB    75 0D                jnz short SuperPIM.00457BCA  

00457BBD    8BC3                 mov eax,ebx  

00457BBF    BA 01000000          mov edx,1  

00457BC4    F0:0FC111            lock xadd dword ptr ds:[ecx],edx  

00457BC8    EB 33                jmp short SuperPIM.00457BFD  

00457BCA    8B4B 04              mov ecx,dword ptr ds:[ebx+4]  

00457BCD    8B10                 mov edx,dword ptr ds:[eax]  

00457BCF    6A 01                push 1  

00457BD1    51                   push ecx  

00457BD2    8BC8                 mov ecx,eax  

00457BD4    FF12                 call dword ptr ds:[edx]  

00457BD6    85C0                 test eax,eax  

00457BD8    75 05                jnz short SuperPIM.00457BDF  

00457BDA    E9 61A3FAFF          jmp SuperPIM.00401F40  

00457BDF    8B53 04              mov edx,dword ptr ds:[ebx+4]  

00457BE2    8950 04              mov dword ptr ds:[eax+4],edx  

00457BE5    8B4B 04              mov ecx,dword ptr ds:[ebx+4]  

00457BE8    41                   inc ecx  

00457BE9    8BD1                 mov edx,ecx  

00457BEB    C1E9 02              shr ecx,2  

00457BEE    8D73 10              lea esi,dword ptr ds:[ebx+10]  

00457BF1    8D78 10              lea edi,dword ptr ds:[eax+10]  

00457BF4    F3:A5                rep movs dword ptr es:[edi],dword ptr ds>  

00457BF6    8BCA                 mov ecx,edx  

00457BF8    83E1 03              and ecx,3  

00457BFB    F3:A4                rep movs byte ptr es:[edi],byte ptr ds:>  

00457BFD    8B7424 34            mov esi,dword ptr ss:[esp+34]  

00457C01    83C0 10              add eax,10  

00457C04    8906                 mov dword ptr ds:[esi],eax  

00457C06    C74424 2C FFFFFFFF   mov dword ptr ss:[esp+2C],-1  

00457C0E    8D43 0C              lea eax,dword ptr ds:[ebx+C]  

00457C11    83C9 FF              or ecx,FFFFFFFF  

00457C14    F0:0FC108            lock xadd dword ptr ds:[eax],ecx  

00457C18    49                   dec ecx  

00457C19    85C9                 test ecx,ecx  

00457C1B    7F 08                jg short SuperPIM.00457C25  

00457C1D    8B0B                 mov ecx,dword ptr ds:[ebx]  

00457C1F    8B11                 mov edx,dword ptr ds:[ecx]  

00457C21    53                   push ebx  

00457C22    FF52 04              call dword ptr ds:[edx+4]  //处理机器码后 EBX=111                ;  

00457C25    8B4C24 24            mov ecx,dword ptr ss:[esp+24]  

00457C29    5B                   pop ebx  

00457C2A    5F                   pop edi  

00457C2B    8BC6                 mov eax,esi  

00457C2D    5E                   pop esi  

00457C2E    5D                   pop ebp  

00457C2F    64:890D 00000000     mov dword ptr fs:[0],ecx  

00457C36    83C4 20              add esp,20  

00457C39    C3                   retn   //跳回去~~~~  

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
```

## 汇编代码段 2

跟进 `0041FB5F` 处 `CALL` 来到：

```asm
004908DD    55                   push ebp  

004908DE    8BEC                 mov ebp,esp  

004908E0    56                   push esi  

004908E1    E8 22540000          call SuperPIM.00495D08  

004908E6    8B70 60              mov esi,dword ptr ds:[eax+60]  

004908E9    3B35 38F75200        cmp esi,dword ptr ds:[52F738]  

004908EF    74 07                je short SuperPIM.004908F8  

004908F1    E8 713C0000          call SuperPIM.00494567  

004908F6    8BF0                 mov esi,eax  

004908F8    837E 08 00           cmp dword ptr ds:[esi+8],0  

004908FC    75 07                jnz short SuperPIM.00490905  

004908FE    5E                   pop esi  

004908FF    5D                   pop ebp  

00490900    E9 3B550000          jmp SuperPIM.00495E40  

00490905    53                   push ebx  

00490906    57                   push edi  

00490907    8B7D 0C              mov edi,dword ptr ss:[ebp+C]                ; 串 1 送 EDI  

0049090A    8B45 08              mov eax,dword ptr ss:[ebp+8]                ; 串 2 送 EAX  

0049090D    66:0FB600            movzx ax,byte ptr ds:[eax]                 ; 逐位取串 2 到 AL  

00490911    FF45 08              inc dword ptr ss:[ebp+8]  

00490914    0FB6C8               movzx ecx,al                               ; AL 送 ECX  

00490917    F64431 1D 04         test byte ptr ds:[ecx+esi+1D],4            ; [ecx+esi+1D]和 4 与  

0049091C    74 18                je short SuperPIM.00490936                 ; 为 0 就跳  

0049091E    8B4D 08              mov ecx,dword ptr ss:[ebp+8]                ; 否则串 2 从本位到完的字符送到 ECX  

00490921    8A09                 mov cl,byte ptr ds:[ecx]                   ; 取串 2 第一位送 CL  

00490923    84C9                 test cl,cl                                 ; 是否为 0  

00490925    75 04                jnz short SuperPIM.0049092B                ; 不为 0 就跳  

00490927    33C0                 xor eax,eax                                ; 否则 EAX 清 0  

00490929    EB 0B                jmp short SuperPIM.00490936  

0049092B    33D2                 xor edx,edx                                ; EDX 清 0  

0049092D    FF45 08              inc dword ptr ss:[ebp+8]  

00490930    8AF0                 mov dh,al                                  ; AL 送 DH  

00490932    8AD1                 mov dl,cl                                  ; CL 送 DL  

00490934    8BC2                 mov eax,edx                                ; EDX 送 EAX  

00490936    66:0FB60F            movzx cx,byte ptr ds:[edi]                ; 逐位取串 1 到 CL  

0049093A    0FB6D1               movzx edx,cl                               ; CL 送 EDX  

0049093D    47                   inc edi                                      ; EDI+1  

0049093E    F64432 1D 04         test byte ptr ds:[edx+esi+1D],4            ; [edx+esi+1D]和 4 与  

00490943    74 13                je short SuperPIM.00490958                 ; 为 0 就跳  

00490945    8A17                 mov dl,byte ptr ds:[edi]                   ; 否则取[edi]送 DL  
```



```
00490947    84D2             test dl,dl                           ; 是否为 0
00490949    75 04            jnz short SuperPIM.0049094F          ; 不则跳
0049094B    33C9             xor ecx,ecx                          ; 否则 ECX
0049094D   /EB 09            jmp short SuperPIM.00490958
0049094F   |33DB             xor ebx,ebx                          ; EBX 清 0
00490951   |8AF9             mov bh,cl                            ; 取串 1 第一位送 CL
00490953   |47               inc edi                              ; EDI+1
00490954   |8ADA             mov bl,dl                            ; DL 送 BL
00490956   |8BCB             mov ecx,ebx                          ; EBX 送 ECX
00490958   \\66:3BC8         cmp cx,ax                            ; CX 和 AX 比较
0049095B    75 0C            jnz short SuperPIM.00490969          ; 不等就跳出去
0049095D    66:85C0          test ax,ax
00490960  ^ 75 A8            jnz short SuperPIM.0049090A          ; 等就跳回去继续~~~~~
00490962    33C0             xor eax,eax
00490964    5F               pop edi
00490965    5B               pop ebx
00490966    5E               pop esi
00490967    5D               pop ebp
00490968    C3               retn
00490969    1BC0             sbb eax,eax                          ; EAX 带借位减法
0049096B    83E0 02          and eax,2                            ; EAX 和 2 与
0049096E    48               dec eax                              ; 再减一
0049096F  ^ EB F3            jmp short SuperPIM.00490964
```

该程序我就分析到这里，如果要推出注册码不是小弟力所能及的啦，等待各位大侠啦~~~~~~~~顺便说一句好象爆破不行哟^-^

`4dd466d0e1e0cc123afbe504da2462ca`
