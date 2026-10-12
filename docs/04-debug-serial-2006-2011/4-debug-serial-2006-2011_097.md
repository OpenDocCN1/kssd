
# Open Video Capture 1.24.304

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-441.htm`](https://www.pediy.com/kssd/pediy08/pediy8-441.htm)

Open Video Capture 1.24.304 简单算法分析-菜鸟篇

【文章作者】: `tzl`

【作者邮箱】: 无

【软件名称】: `Open Video Capture 1.24.304`

【软件大小】: 641KB

【下载地址】: [http://nj.onlinedown.net/soft/46986.htm](http://nj.onlinedown.net/soft/46986.htm)

【加壳方式】: 无

【保护方式】: 注册码

【编写语言】: `Microsoft Visual C++ 7.0`

【使用工具】: `OD` `PEID`

【操作平台】: `XP SP2`

【软件介绍】:

`Open Video Converter` 是一款易于使用的视频转换，分割和编辑工具。它能转换多个视频格式如 `MPG`，`AVI`，`ASF`，`WMV` 到 `AVI` 文件。它能改变帧尺寸，帧频，视频和音频压缩编码。主要功能有：-转换 `MPEG`，`WMV`，`ASF`，`MPG`，`VCD`，`OGM`，`DAT`，`SVCD` 为 `AVI`。

适合菜鸟学习的好软件，这里与大家分享，菜鸟共同进步。晚上喝了点酒,不知不觉看一下时间到快凌晨 1 点了...搞这个时间过的真是快![](img/45777b5dbd6b901752ab7df42fce6892.jpg)

## 一、查壳无

## 二、根据字符串相关信息，我们可以在这里下断开始分析

注册名：`tigerisme` 试练码：`123456789`

```
00402AD5   > \8B4424 24     mov eax,dword ptr ss:[esp+24]     ;  Case 111 of switch 00402AB7

00402AD9   .  48           dec eax                             ;  Switch (cases 1..3EA)

00402ADA   .  74 51        je short openvcap.00402B2D

00402ADC   .  48           dec eax

00402ADD   .  74 32        je short openvcap.00402B11

00402ADF   .  2D E8030000  sub eax,3E8

00402AE4   .  0F85 CF020000 jnz openvcap.00402DB9

00402AEA   .  6A 01        push 1                              ; /IsShown = 1; Case 3EA of switch 00402AD9

00402AEC   .  6A 00        push 0                              ; |DefDir = NULL

00402AEE   .  6A 00        push 0                              ; |Parameters = NULL

00402AF0   .  68 C0B74100  push openvcap.0041B7C0             ; |FileName = "http://www.008soft.com/products/video-capture.htm"

00402AF5   .  68 B8B74100  push openvcap.0041B7B8             ; |Operation = "open"

00402AFA   .  6A 00        push 0                              ; |hWnd = NULL

00402AFC   .  FF15 28B24100 call dword ptr ds:[<&SHELL32.Shel>; \ShellExecuteA

00402B02   .  5F           pop edi

00402B03   .  5E           pop esi

00402B04   .  5D           pop ebp

00402B05   .  B8 01000000  mov eax,1

00402B0A   .  5B           pop ebx

00402B0B   .  83C4 08      add esp,8

00402B0E   .  C2 1000      retn 10

00402B11   >  8B4424 1C    mov eax,dword ptr ss:[esp+1C]     ;  Case 2 of switch 00402AD9

00402B15   .  6A 00        push 0                              ; /Result = 0

00402B17   .  50           push eax                            ; |hWnd

00402B18   .  FF15 18B34100 call dword ptr ds:[<&USER32.EndDi>; \EndDialog

00402B1E   .  5F           pop edi

00402B1F   .  5E           pop esi

00402B20   .  5D           pop ebp

00402B21   .  B8 01000000  mov eax,1

00402B26   .  5B           pop ebx

00402B27   .  83C4 08      add esp,8

00402B2A   .  C2 1000      retn 10

00402B2D   >  8B7C24 1C    mov edi,dword ptr ss:[esp+1C]     ;  Case 1 of switch 00402AD9

00402B31   .  8B35 04B34100 mov esi,dword ptr ds:[<&USER32.Ge>;  USER32.GetDlgItemTextA

00402B37   .  68 00010000  push 100                            ; /Count = 100 (256.)

00402B3C   .  68 B0104200  push openvcap.004210B0             ; |Buffer = openvcap.004210B0

00402B41   .  68 E8030000  push 3E8                            ; |ControlID = 3E8 (1000.)

00402B46   .  57           push edi                            ; |hWnd

00402B47   .  FFD6         call esi                            ; \GetDlgItemTextA

00402B49   .  68 00010000  push 100                            ; /Count = 100 (256.)

00402B4E   .  68 20164200  push openvcap.00421620             ; |Buffer = openvcap.00421620

00402B53   .  68 E9030000  push 3E9                            ; |ControlID = 3E9 (1001.)

00402B58   .  57           push edi                            ; |hWnd

00402B59   .  FFD6         call esi                            ; \GetDlgItemTextA

00402B5B   .  B8 B0104200  mov eax,openvcap.004210B0          ;  ASCII "tigerisme"

00402B60   .  8D50 01      lea edx,dword ptr ds:[eax+1]

00402B63   >  8A08         mov cl,byte ptr ds:[eax]            ;  注册名 ascii 码逐个送 cl

00402B65   .  40           inc eax                               ;  eax+1

00402B66   .  84C9         test cl,cl

00402B68   . ^ 75 F9       jnz short openvcap.00402B63          ;  运算后 eax=004210BA

00402B6A   .  2BC2         sub eax,edx                           ;  eax-edx

00402B6C   .  83F8 02      cmp eax,2                             ;  eax（9）与 2 比较，注册名位数须大于等于 2

00402B6F   .  73 22        jnb short openvcap.00402B93

00402B71   .  6A 00        push 0                              ; /Style = MB_OK|MB_APPLMODAL

00402B73   .  68 B0B74100  push openvcap.0041B7B0             ; |Title = "Error"

00402B78   .  68 90B74100  push openvcap.0041B790             ; |Text = "Please input correct User Name!"

00402B7D   .  57           push edi                            ; |hOwner

00402B7E   .  FF15 68B24100 call dword ptr ds:[<&USER32.Messa>; \MessageBoxA

00402B84   .  5F           pop edi

00402B85   .  5E           pop esi

00402B86   .  5D           pop ebp

00402B87   .  B8 01000000  mov eax,1

00402B8C   .  5B           pop ebx

00402B8D   .  83C4 08      add esp,8

00402B90   .  C2 1000      retn 10

00402B93   >  B8 20164200  mov eax,openvcap.00421620          ;  ASCII "123456789"

00402B98   .  8D50 01      lea edx,dword ptr ds:[eax+1]

00402B9B   .  EB 03        jmp short openvcap.00402BA0

00402B9D     8D49 00      lea ecx,dword ptr ds:[ecx]

00402BA0   >  8A08         mov cl,byte ptr ds:[eax]            ;  试练码逐个送 cl

00402BA2   .  40           inc eax                               ;  eax+1

00402BA3   .  84C9         test cl,cl

00402BA5   . ^ 75 F9       jnz short openvcap.00402BA0          ;  运算后 eax=0042162A

00402BA7   .  2BC2         sub eax,edx                           ;  eax-edx

00402BA9   .  83F8 08      cmp eax,8                             ;  eax（9）与 8 比较，试练码位数须大于等于 8

00402BAC   .  73 22        jnb short openvcap.00402BD0

00402BAE   .  6A 00        push 0                              ; /Style = MB_OK|MB_APPLMODAL

00402BB0   .  68 B0B74100  push openvcap.0041B7B0             ; |Title = "Error"

00402BB5   .  68 68B74100  push openvcap.0041B768             ; |Text = "Please input correct Registration Code!"

00402BBA   .  57           push edi                            ; |hOwner

00402BBB   .  FF15 68B24100 call dword ptr ds:[<&USER32.Messa>; \MessageBoxA

00402BC1   .  5F           pop edi

00402BC2   .  5E           pop esi

00402BC3   .  5D           pop ebp

00402BC4   .  B8 01000000  mov eax,1

00402BC9   .  5B           pop ebx

00402BCA   .  83C4 08      add esp,8

00402BCD   .  C2 1000      retn 10

00402BD0   >  0FB60D B01042 >movzx ecx,byte ptr ds:[4210B0]    ;  ds:[4210B0]=“t”ascii 码 74 送 ecx

00402BD7   .  8BC1         mov eax,ecx                           ;  ecx=74，送 eax

00402BD9   .  83C8 57      or eax,57                             ;  eax=74 与 57 进行 or 运算

00402BDC   .  99           cdq                                   ;  eax=77

00402BDD   .  BE 0A000000  mov esi,0A                            ;  0A 送 esi

00402BE2   .  F7FE         idiv esi                              ;  eax 与 A 进行 idiv 运算，结果为 0000000B r 00000009，余数 9 放在 edx 中

00402BE4     0FB635 B11042 >movzx esi,byte ptr ds:[4210B1]    ;  ds:[4210B1]="i"ascii 码 69 送 esi

00402BEB   .  8BC6         mov eax,esi                           ;  69 送 eax

00402BED   .  83C8 45      or eax,45                             ;  eax（69）与 45 进行 or 运算，结果为 6D

00402BF0   .  BF 0A000000  mov edi,0A                            ;  eax=6D

00402BF5   .  33ED         xor ebp,ebp

00402BF7   .  885424 20    mov byte ptr ss:[esp+20],dl       ;  dl=09 送 ss:[esp+20]

00402BFB   .  99           cdq

00402BFC   .  F7FF         idiv edi                              ;  eax 与 A 进行 idiv 运算，结果 0000000A r 00000009，余数 9 放在 edx 中

00402BFE   .  8BC1         mov eax,ecx                           ;  ecx=74 送 eax
```



```
00402C00   .  83C8 42       or eax,42                    ;  eax or 42=76，eax=76  
00402C03   .  8BCF         mov ecx,edi                  ;  edi=A，送 ecx  
00402C05   .  885424 24    mov byte ptr ss:[esp+24],dl  ;  dl=09 送 ss:[esp+24]  
00402C09   .  99           cdq  
00402C0A   .  F7F9         idiv ecx                     ;  eax 与 A 进行 idiv 运算，0000000B r 00000008，余数 8 放在 edx 中  
00402C0C   .  8BC6         mov eax,esi                  ;  esi=69，送 eax  
00402C0E   .  83C8 43      or eax,43                    ;  eax（69） or 43=6B  
00402C11   .  885424 12    mov byte ptr ss:[esp+12],dl  ;  dl=8，送 ss:[esp+12]  
00402C15   .  99           cdq  
00402C16   .  F7F9         idiv ecx                     ;  eax（6B）与 A 进行 idiv 运算，结果为 0000000A r 00000007，余数 7 放在 edx 中  
00402C18   .  B9 B0104200  mov ecx,openvcap.004210B0    ;  注册名 tigerisme 送 ecx  
00402C1D   .  33F6         xor esi,esi                  ;  esi 清零  
00402C1F   .  8D79 01      lea edi,dword ptr ds:[ecx+1]  
00402C22      885424 13    mov byte ptr ss:[esp+13],dl  ;  dl=7，送 ss:[esp+13]  
00402C26   >  8A01         mov al,byte ptr ds:[ecx]      ;  74“t”送 al  
00402C28   .  41           inc ecx                      ;  ecx+1  
00402C29   .  84C0         test al,al  
00402C2B   .^ 75 F9       jnz short openvcap.00402C26    ;  循环后得 ecx=004210BA  
00402C2D   .  2BCF         sub ecx,edi                  ;  ecx-edi=9  
00402C2F   .  894C24 14    mov dword ptr ss:[esp+14],ecx ;  ecx=9，送 ss:[esp+14]  
00402C33   .  74 2A       je short openvcap.00402C5F  
00402C35   .  EB 09       jmp short openvcap.00402C40  
00402C37   .  8DA424 000000>lea esp,dword ptr ss:[esp]  
00402C3E   .  8BFF         mov edi,edi  
00402C40   >  0FB696 B01042>movzx edx,byte ptr ds:[esi+4210B0>;  进入循环，注册名逐位 ascii 码送 edx  
00402C47   .  B9 B0104200  mov ecx,openvcap.004210B0    ;  ASCII "tigerisme"  
00402C4C   .  03EA         add ebp,edx                  ;  ebp+edx  
00402C4E   .  46           inc esi                      ;  esi+1  
00402C4F   .  8D79 01      lea edi,dword ptr ds:[ecx+1]  
00402C52   >  8A01         mov al,byte ptr ds:[ecx]      ;  注册名逐位 ascii 码送 al，进入循环  
00402C54   .  41           inc ecx                      ;  ecx+1  
00402C55   .  84C0         test al,al  
00402C57   .^ 75 F9       jnz short openvcap.00402C52    ;  循环后得 ecx=004210BA  
00402C59   .  2BCF         sub ecx,edi                  ;  ecx-edi  
00402C5B   .  3BF1         cmp esi,ecx                  ;  esi 与 ecx 比较  
00402C5D   .^ 72 E1       jb short openvcap.00402C40  
00402C5F   >  8A0D 20164200 mov cl,byte ptr ds:[421620]  ;  ebp=3C9，cl=9，ds:[421620]=31 送 cl  
00402C65   .  0FB67C24 20  movzx edi,byte ptr ss:[esp+20] ;  ss:[esp+20]=09 送 edi  
00402C6A   .  8A1D 21164200 mov bl,byte ptr ds:[421621] ;  32 送 bl  
00402C70   .  A0 22164200  mov al,byte ptr ds:[421622]  ;  33 送 al  
00402C75   .  8A15 23164200 mov dl,byte ptr ds:[421623]  ;  34 送 dl  
00402C7B   .  0FB6F1       movzx esi,cl                 ;  cl=39 送 esi  
00402C7E   .  83EE 30      sub esi,30                   ;  esi-30，esi=09  
00402C81   .  3BFE         cmp edi,esi                  ;  edi（3）与 esi（9）比较，即第一位必须为 9  
00402C83   .  75 48       jnz short openvcap.00402CCD  
00402C85   .  0FB67C24 24  movzx edi,byte ptr ss:[esp+24] ;  ss:[esp+24]=09  
00402C8A   .  0FB6F3       movzx esi,bl                 ;  bl 送 esi  
00402C8D   .  83EE 30      sub esi,30  
00402C90   .  3BFE         cmp edi,esi                  ;  edi（2）与 esi（9）比较，即第二位必须为 9  
00402C92   .  75 39       jnz short openvcap.00402CCD  
00402C94   .  0FB67424 12  movzx esi,byte ptr ss:[esp+12] ;  ss:[esp+12]=08  
00402C99   .  0FB6C0       movzx eax,al  
00402C9C   .  83E8 30      sub eax,30  
00402C9F   .  3BF0         cmp esi,eax                  ;  eax（3）与 esi（8）比较，即第三位必须为 8  
00402CA1   .  75 2A       jnz short openvcap.00402CCD  
00402CA3   .  0FB64424 13  movzx eax,byte ptr ss:[esp+13] ;  ss:[esp+13]=07  
00402CA8   .  0FB6D2       movzx edx,dl  
00402CAB   .  83EA 30      sub edx,30  
00402CAE   .  3BC2         cmp eax,edx                  ;  edx（4）与 eax（7）比较，即第四位必须为 7  
00402CB0   .  75 1B       jnz short openvcap.00402CCD  
00402CB2   .  8BC5         mov eax,ebp                  ;  ebp=3C9，送 eax  
00402CB4   .  99           cdq  
00402CB5   .  BE 0A000000  mov esi,0A  
00402CBA   .  F7FE         idiv esi                     ;  eax（3C9）与 A 进行 idiv 运算，结果 00000060 r 00000009，余数 9 放 edx  
00402CBC   .  0FB605 241642>movzx eax,byte ptr ds:[421624] ;  试练码第五位 ds:[421624]=35“5”，送 eax  
00402CC3   .  83E8 30      sub eax,30                   ;  eax-30，eax=05  
00402CC6   .  0FB6D2       movzx edx,dl                 ;  dl=9，送 edx  
00402CC9   .  3BD0         cmp edx,eax                  ;  eax（5）与 edx（9）比较，即第五位必须为 9  
00402CCB   .  74 4B       je short openvcap.00402D18    ;  前面 5 位正确则注册成功，与后面的注册码无关  
00402CCD   >  80F9 32      cmp cl,32                    ;  这里是一组通用注册码的比较判断，2  
00402CD0   .  0F85 99000000 jnz openvcap.00402D6F       
00402CD6   .  80FB 33      cmp bl,33                    ;  3  
00402CD9   .  0F85 90000000 jnz openvcap.00402D6F  
00402CDF   .  803D 22164200>cmp byte ptr ds:[421622],39  ;  9  
00402CE6   .  0F85 83000000 jnz openvcap.00402D6F  
00402CEC   .  803D 23164200>cmp byte ptr ds:[421623],31  ;  1  
00402CF3   .  75 7A       jnz short openvcap.00402D6F  
00402CF5   .  381D 24164200 cmp byte ptr ds:[421624],bl  ;  3  
00402CFB   .  75 72       jnz short openvcap.00402D6F  
00402CFD   .  803D 25164200>cmp byte ptr ds:[421625],31  ;  1  
00402D04   .  75 69       jnz short openvcap.00402D6F  
00402D06   .  803D 26164200>cmp byte ptr ds:[421626],34  ;  4  
00402D0D   .  75 60       jnz short openvcap.00402D6F  
00402D0F   .  803D 27164200>cmp byte ptr ds:[421627],36  ;  6  
00402D16   .  75 57       jnz short openvcap.00402D6F  
00402D18   >  8B7C24 1C    mov edi,dword ptr ss:[esp+1C]  
00402D1C   .  6A 00        push 0                       ;  /Style = MB_OK|MB_APPLMODAL  
00402D1E   .  68 60B74100  push openvcap.0041B760      ;  |Title = "Message"  
00402D23   .  68 44B74100  push openvcap.0041B744      ;  |registration has succeeded!  
00402D28   .  57          push edi                     ;  |hOwner  
00402D29   .  FF15 68B24100 call dword ptr ds:[<&USER32.Messa>;  \MessageBoxA  
00402D2F   .  8B35 A0B04100 mov esi,dword ptr ds:[<&KERNEL32.>;  kernel32.WriteProfileStringA  
00402D35   .  68 B0104200  push openvcap.004210B0      ;  /String = "tigerisme"  
00402D3A   .  68 38B74100  push openvcap.0041B738      ;  |username  
00402D3F   .  68 ECB64100  push openvcap.0041B6EC      ;  |openvideocapture  
00402D44   .  FFD6        call esi                     ;  \WriteProfileStringA  
00402D46   .  68 20164200  push openvcap.00421620      ;  /String = "123456789"  
00402D4B   .  68 24B74100  push openvcap.0041B724      ;  |registration_code  
00402D50   .  68 ECB64100  push openvcap.0041B6EC      ;  |openvideocapture  
00402D55   .  FFD6        call esi                     ;  \WriteProfileStringA  
00402D57   .  6A 01       push 1                       ;  /Result = 1  
00402D59   .  57          push edi                     ;  |hWnd  
00402D5A   .  FF15 18B34100 call dword ptr ds:[<&USER32.EndDi>;  \EndDialog  
00402D60   .  5F          pop edi  
00402D61   .  5E          pop esi  
00402D62   .  5D          pop ebp  
00402D63   .  B8 01000000  mov eax,1  
00402D68   .  5B          pop ebx  
```



```
00402D69   .  83C4 08       add esp,8  
00402D6C   .  C2 1000       retn 10  
00402D6F   >  8B4C24 1C     mov ecx,dword ptr ss:[esp+1C]  
00402D73   .  6A 00         push 0                     ; /Style = MB_OK|MB_APPLMODAL  
00402D75   .  68 B0B74100   push openvcap.0041B7B0      ; |error  
00402D7A   .  68 0CB74100   push openvcap.0041B70C      ; |registration failed!  
00402D7F   .  51            push ecx                   ; |hOwner  
00402D80   .  FF15 68B24100 call dword ptr ds:<&USER32.Messa>; \\MessageBoxA  
00402D86   .  5F            pop edi  
00402D87   .  5E            pop esi  
00402D88   .  5D            pop ebp  
00402D89   .  B8 01000000   mov eax,1  
00402D8E   .  5B            pop ebx  
00402D8F   .  83C4 08       add esp,8  
00402D92   .  C2 1000       retn 10  
```

**算法总结**

软件采算法比较简单，注册名须不小于两位，注册码位数为 8 位以上，主要思路如下：

1. 注册名第一位的 `ascii` 码与 `57` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第一位；
2. 注册名第二位的 `ascii` 码与 `45` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第二位；
3. 注册名第一位的 `ascii` 码与 `42` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `8` 为注册码第三位；
4. 注册名第二位的 `ascii` 码与 `43` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `7` 为注册码第四位；

此时 `ebp=3C9`

5. 将 `3C9` 与 `43` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第五位；
6. 第六位以后任意

合起来，即注册名：`tigerisme` 注册码为：`99879*****`，这里还有一组通用注册码 `23913146`。

**特别说明**

本文仅是一些破解的心得和思路，完全是个人对程序的研究，无其他目的。

`4dd466d0e1e0cc123afbe504da2462ca`
