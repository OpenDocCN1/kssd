
# Turbo Photo V4.8

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-70.htm`](https://www.pediy.com/kssd/pediy08/pediy8-70.htm)

## 基本信息

- **【破文作者】** rdsnow\[BCG\]\[PYG\]\[D.4s\]
- **【作者主页】** [`rdsnow.ys168.com`](http://rdsnow.ys168.com)
- **【 E-mail 】** rdsnow@163.com
- **【 作者 QQ 】** 83757177
- **【文章题目】** Turbo Photo V4.8 的注册
- **【软件名称】** Turbo Photo
- **【软件版本】** V4.8
- **【下载地址】** [`nj.onlinedown.net/soft/5620.htm`](http://nj.onlinedown.net/soft/5620.htm)

## 技术细节

- **【加密方式】** 序列号
- **【破解工具】** ODbyDYK v1.10\[05.09\]
- **【软件限制】** 时间限制
- **【破解平台】** Microsoft Windows XP Professional
- **【平台版本】** 5.1.2600 Service Pack 2 内部版本号 2600

## 软件简介

Turbo Photo 是一个以数码影像为背景，面向数码相机普通用户和准专业用户而设计的一套集图片管理，浏览，处理，输出为一身的软件系统。她包括两个部分:`Turbo Photo 相册`和 `Turbo Photo 编辑器`。

- `Turbo Photo 相册` 是一个照片管理、浏览与批处理为一体的软件。
- `Turbo Photo 编辑器` 的所有功能均围绕如何让您的照片更出色这样一个主题而设计。每个功能都针对了数码相机本身的特点和最常见的问题。

## 文章简介

网友给的一个程序，算法不是太难，不过程序中的随机验证的思路倒是可以让我们学习一下。

## 破解过程

注册码跟用户名无关，输入 `98765-54321-123456-56789` 下断：`BP MessageBoxA` 很容易找到解锁按钮的响应代码：

```asm
0045B590   .  53            PUSH EBX
0045B591   .  55            PUSH EBP
0045B592   .  56            PUSH ESI
0045B593   .  57            PUSH EDI
0045B594   .  8BD9          MOV EBX,ECX
0045B596   .  6A 01         PUSH 1
0045B598   .  E8 45E20300  CALL <JMP.&MFC42.#6334_CWnd::UpdateData>    ; UpdateData(true)
0045B59D   .  8DAB A0000000 LEA EBP,DWORD PTR [EBX+A0]
0045B5A3   .  8DBB A8000000 LEA EDI,DWORD PTR [EBX+A8]
0045B5A9   .  8BCD          MOV ECX,EBP
0045B5AB   .  8DB3 B4000000 LEA ESI,DWORD PTR [EBX+B4]
0045B5B1   .  E8 F2E50300  CALL <JMP.&MFC42.#4171_CString::LockBuffer> ; S3
0045B5B6   .  50            PUSH EAX
0045B5B7   .  8BCF          MOV ECX,EDI
0045B5B9   .  E8 EAE50300  CALL <JMP.&MFC42.#4171_CString::LockBuffer> ; S2
0045B5BE   .  50            PUSH EAX
0045B5BF   .  8D8B B0000000 LEA ECX,DWORD PTR [EBX+B0]
0045B5C5   .  E8 DEE50300  CALL <JMP.&MFC42.#4171_CString::LockBuffer> ; S1
0045B5CA   .  50            PUSH EAX
0045B5CB   .  8BCE          MOV ECX,ESI
0045B5CD   .  E8 D6E50300  CALL <JMP.&MFC42.#4171_CString::LockBuffer> ; S0
0045B5D2   .  50            PUSH EAX
0045B5D3   .  B9 8CA84C00  MOV ECX,TPhoto.004CA88C
0045B5D8   .  E8 53050000  CALL TPhoto.0045BB30                   ; 关键 Call，要跟进
0045B5DD   .  85C0          TEST EAX,EAX
0045B5DF   .  75 1B         JNZ SHORT TPhoto.0045B5FC             ; 关键跳转
0045B5E1   .  50            PUSH EAX
0045B5E2   .  50            PUSH EAX
0045B5E3   .  6A 47         PUSH 47
0045B5E5   .  B9 689A4C00  MOV ECX,TPhoto.004C9A68
0045B5EA   .  E8 C17D0000  CALL TPhoto.004633B0
0045B5EF   .  50            PUSH EAX
0045B5F0   .  8BCB          MOV ECX,EBX
0045B5F2   .  E8 4DE30300  CALL <JMP.&MFC42.#4224_CWnd::MessageBoxA> ; 错误的序列号
0045B5F7   .  5F            POP EDI
0045B5F8   .  5E            POP ESI
0045B5F9   .  5D            POP EBP
0045B5FA   .  5B            POP EBX
0045B5FB   .  C3            RETN
```

跟进后，走过一个 `S0` 跟 `"DDGTM"`(这个是不能注册成功的) 比较的循环后来到这里

```asm
0045BC39  |.  50            PUSH EAX                   ; /Arg4
0045BC3A  |.  51            PUSH ECX                   ; |Arg3
0045BC3B  |.  52            PUSH EDX                   ; |Arg2
0045BC3C  |.  55            PUSH EBP                   ; |Arg1
0045BC3D  |.  8D8B 04020000 LEA ECX,DWORD PTR [EBX+204]   ; |
0045BC43  |.  E8 08D40300  CALL TPhoto.00499050          ; \TPhoto.00499050
0045BC48  |.  85C0          TEST EAX,EAX
0045BC4A  |.  75 07         JNZ SHORT TPhoto.0045BC53     ; 把这里作为爆破点
0045BC4C  |.  5F            POP EDI
0045BC4D  |.  5E            POP ESI
0045BC4E  |.  5D            POP EBP
0045BC4F  |.  5B            POP EBX
0045BC50  |.  C2 1000       RETN 10
```

继续跟进 `0045BC43 CALL TPhoto.00499050` 来到：

```asm
00499050  /$  83EC 64       SUB ESP,64
00499053  |.  53            PUSH EBX
00499054  |.  55            PUSH EBP
00499055  |.  56            PUSH ESI
00499056  |.  57            PUSH EDI
00499057  |.  8BF1          MOV ESI,ECX
00499059  |.  E8 62040000   CALL TPhoto.004994C0          ;  (Sum(用户帐户名)+1)%10
0049905E  |.  8BCE          MOV ECX,ESI
00499060  |.  8BE8          MOV EBP,EAX
00499062  |.  E8 59040000   CALL TPhoto.004994C0          ;  (Sum(用户帐户名)+1)%10
00499067  |.  8AC8          MOV CL,AL
00499069  |.  8D7C24 10     LEA EDI,DWORD PTR [ESP+10]
0049906D  |.  8AE9          MOV CH,CL
0049906F  |.  8D546D 00     LEA EDX,DWORD PTR [EBP+EBP*2]
00499073  |.  8BC1          MOV EAX,ECX
00499075  |.  C1E0 10       SHL EAX,10
00499078  |.  66:8BC1       MOV AX,CX
0049907B  |.  B9 19000000   MOV ECX,19
00499080  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]
00499082  |.  33DB          XOR EBX,EBX
00499084  |.  8D7C96 40     LEA EDI,DWORD PTR [ESI+EDX*4+40]   ;  根据 (Sum(用户帐户名)+1)%10 取得验证地址的偏移表
00499088  |>  FF15 D0684A00 /CALL DWORD PTR [<&MSVCRT.rand>]   ; [rand
0049908E  |.  99            |CDQ
0049908F  |.  B9 64000000   |MOV ECX,64
00499094  |.  F7F9          |IDIV ECX                 ;  rand() % 100
00499096  |.  0FBE5414 10  |MOVSX EDX,BYTE PTR [ESP+EDX+10]
0049909B  |.  3BEA          |CMP EBP,EDX
0049909D  |.  74 09         |JE SHORT TPhoto.004990A8
0049909F  |.  8B07          |MOV EAX,DWORD PTR [EDI]
004990A1  |.  C70486 000000>|MOV DWORD PTR [ESI+EAX*4],0
004990A8  |>  8BCE          |MOV ECX,ESI
004990AA  |.  E8 31FFFFFF   |CALL TPhoto.00498FE0       ;  第一次 GetTickCount
004990AF  |.  8B9424 840000>|MOV EDX,DWORD PTR [ESP+84]
004990B6  |.  8B8424 800000>|MOV EAX,DWORD PTR [ESP+80]
004990BD  |.  8B0F          |MOV ECX,DWORD PTR [EDI]
004990BF  |.  52            |PUSH EDX
004990C0  |.  8B9424 800000>|MOV EDX,DWORD PTR [ESP+80]
004990C7  |.  50            |PUSH EAX
004990C8  |.  8B8424 800000>|MOV EAX,DWORD PTR [ESP+80]
004990CF  |.  52            |PUSH EDX
004990D0  |.  50            |PUSH EAX
004990D1  |.  FF148E        |CALL DWORD PTR [ESI+ECX*4]   ;  [ESI+ECX*4]得到验证地址
004990D4  |.  83C4 10       |ADD ESP,10
004990D7  |.  85C0          |TEST EAX,EAX
004990D9  |.  74 1F         |JE SHORT TPhoto.004990FA
004990DB  |.  8BCE          |MOV ECX,ESI
004990DD  |.  E8 1EFFFFFF   |CALL TPhoto.00499000       ;  第二次 GetTickCount，两次结果相减判断程序有没有被单步跟踪
004990E2  |.  43            |INC EBX
004990E3  |.  83C7 04       |ADD EDI,4
004990E6  |.  83FB 03       |CMP EBX,3
004990E9  |.^ 7C 9D         \JL SHORT TPhoto.00499088
004990EB  |.  5F            POP EDI
004990EC  |.  5E            POP ESI
004990ED  |.  5D            POP EBP
004990EE  |.  B8 01000000   MOV EAX,1
004990F3  |.  5B            POP EBX
004990F4  |.  83C4 64       ADD ESP,64
004990F7  |.  C2 1000       RETN 10
```

这一段代码中放了 2 个暗桩：

### 1、第一个暗桩

两次 `GetTickCount` ，其中第二次将返回值相减，得到结果如果太大会清空验证地址表，若要顺利调试，要跟进 `CALL TPhoto.00499000`

```asm
00499000  /$  56            PUSH ESI                   ;  <TPhoto.验证地址>
00499001  |.  8BF1          MOV ESI,ECX
00499003  |.  8B86 5C010000 MOV EAX,DWORD PTR [ESI+15C]
00499009  |.  85C0          TEST EAX,EAX
0049900B  |.  74 28         JE SHORT TPhoto.00499035
0049900D  |.  FF15 6C614A00 CALL DWORD PTR [<&KERNEL32.GetTick>; [GetTickCount(第二次)
00499013  |.  2B86 58010000 SUB EAX,DWORD PTR [ESI+158]
00499019  |.  3D F4010000   CMP EAX,1F4
0049901E  |.  76 15         JBE SHORT TPhoto.00499035     ; 这里改为 JMP，跳过清空
00499020  |.  57            PUSH EDI
00499021  |.  B9 10000000   MOV ECX,10
```



```
00499026  |.  33C0           XOR EAX,EAX  
00499028  |.  8BFE           MOV EDI,ESI  
0049902A  |.  F3:AB          REP STOS DWORD PTR ES:[EDI]        ; 清空验证地址表，导致接下来的跟踪出错  
0049902C  |.  8986 5C010000 MOV DWORD PTR [ESI+15C],EAX  
00499032  |.  5F             POP EDI  
00499033  |.  5E             POP ESI  
00499034  |.  C3             RETN  
00499035  |>  C786 5C010000>MOV DWORD PTR [ESI+15C],0           ; 跳到这里来都过程序清空验证地址表  
0049903F  |.  5E             POP ESI  
00499040  \\.  C3             RETN  
```

2、第二个暗桩：  

来看一下验证地址表和偏移表  

验证地址表存放了 16 个验证 `CALL` 的地址：  

```
004CAA90 >00 00 00 00  00 00 00 00  20 89 49 00  00 00 00 00  ........ 塈.....  
004CAAA0  A0 88 49 00  00 00 00 00  00 00 00 00  00 00 00 00  \_ 圛.............  
004CAAB0  00 00 00 00  B0 8F 49 00  F0 8B 49 00  90 8B 49 00  ....皬 I.饗 I.悑 I.  
004CAAC0  40 8B 49 00  50 8C 49 00  F0 8A 49 00  B0 8C 49 00  @婭.P 孖.饖 I.皩 I.  
```

存放了 3 个验证地址的偏移(这是我的电脑上取得的其他的可能不同)：  

```
004CAB30 >           2                4                9  
```

上面循环只有三轮，每一轮查验证地址的偏移表取得一个验证地址，然后 `CALL` 进这个地址验证，其实作者在程序中放了很多的验证代码，然后根据 `(Sum(用户帐户名)+1)%10` 的计算结果取得不同的验证地址的偏移表，也就是说如果用不同的用户帐号名启动程序，会进入不同的地址验证，这就使得在某个帐号名下能通过的注册码在另外一个账号下不能通过。  

我们的任务就是取得程序中所有的验证地址，然后跟进，找到能通用的注册码，于是在 `004CAA90` 到 `004CAAD0` 中下内存访问断点，重新启动程序断在这里：  

```
004992FE  |.  C703 308A4900   MOV DWORD PTR [EBX],00498A30  
00499304  |.  C743 04 A0894>  MOV DWORD PTR [EBX+4],004989A0  
0049930B  |.  C743 08 20894>  MOV DWORD PTR [EBX+8],00498920  
00499312  |.  C743 0C 108A4>  MOV DWORD PTR [EBX+C],00498A10  
00499319  |.  C743 10 A0884>  MOV DWORD PTR [EBX+10],004988A0  
00499320  |.  C743 14 F08C4>  MOV DWORD PTR [EBX+14],00498CF0  
00499327  |.  C743 18 B08D4>  MOV DWORD PTR [EBX+18],00498DB0  
0049932E  |.  C743 1C 708E4>  MOV DWORD PTR [EBX+1C],00498E70  
00499335  |.  C743 20 208F4>  MOV DWORD PTR [EBX+20],00498F20  
0049933C  |.  C743 24 B08F4>  MOV DWORD PTR [EBX+24],00498FB0  
00499343  |.  C743 28 F08B4>  MOV DWORD PTR [EBX+28],00498BF0  
0049934A  |.  C743 2C 908B4>  MOV DWORD PTR [EBX+2C],00498B90  
00499351  |.  C743 30 408B4>  MOV DWORD PTR [EBX+30],00498B40  
00499358  |.  C743 34 508C4>  MOV DWORD PTR [EBX+34],00498C50  
0049935F  |.  C743 38 F08A4>  MOV DWORD PTR [EBX+38],00498AF0  
00499366  |.  C743 3C B08C4>  MOV DWORD PTR [EBX+3C],00498CB0  
```

…………下面是传递偏移表，并根据 `(Sum(用户帐户名)+1)%10` 清空地址表中不用的一些验证地址  

^\_^，一排地址，走过之后，再看看验证地址表（这才是完整的存放了 16 个地址的表）：  

```
004CAA90 >30 8A 49 00  A0 89 49 00  20 89 49 00  10 8A 49 00  訰 I 覠 I 褠 I 訐 I  
004CAAA0  A0 88 49 00  F0 8C 49 00  B0 8D 49 00  70 8E 49 00  袠 I 賰 I 趰 I 蹰 I  
004CAAB0  20 8F 49 00  B0 8F 49 00  F0 8B 49 00  90 8B 49 00  ?#123;I 辰 I 诰 I 讐 I  
004CAAC0  40 8B 49 00  50 8C 49 00  F0 8A 49 00  B0 8C 49 00  譀 I 豐 I 諰 I 貰 I  
```

将这个表格复制一份，因为程序马上会根据 `(Sum(用户帐户名)+1)%10` 删掉验证地址表中的一些不用的验证地址。  

好了现在将上面的 3 轮循环改为 16 轮循环，并在走到 `004990D1` `CALL DWORD PTR [ESI+ECX*4]` ; `[ESI+ECX*4]`得到验证地址  

时通过修改 `ECX` 的值进入分别各个验证地址（总共有 16 个）  

`/////////////////////////////////////   第 1 个   //////////////////////////////////////`  

```
00498A30   .  8B4424 04     MOV EAX,DWORD PTR [ESP+4]          ;  TPhoto.004CAC3C  
00498A34   .  53             PUSH EBX  
00498A35   .  56             PUSH ESI  
00498A36   .  57             PUSH EDI  
00498A37   .  8A58 04        MOV BL,BYTE PTR [EAX+4]           ;  取出 S0[4]  
00498A3A   .  FF15 68614A00 CALL DWORD PTR [<&KERNEL32.GetSys>; [GetSystemDefaultLangID  
00498A40   .  8BF8          MOV EDI,EAX  
00498A42   .  FF15 D0684A00 CALL DWORD PTR [<&MSVCRT.rand>]    ; [rand  
00498A48   .  8BF0          MOV ESI,EAX  
00498A4A   .  81E6 1F000080 AND ESI,8000001F  
00498A50   .  79 05         JNS SHORT 00498A57  
00498A52   .  4E             DEC ESI  
00498A53   .  83CE E0        OR ESI,FFFFFFE0  
00498A56   .  46             INC ESI  
00498A57   >  FF15 F4684A00 CALL DWORD PTR [<&MSVCRT.clock>]   ;  msvcrt.clock  
00498A5D   .  0FAFF7         IMUL ESI,EDI  
00498A60   .  8BC8          MOV ECX,EAX  
00498A62   .  B8 1F85EB51   MOV EAX,51EB851F  
00498A67   .  F7E9          IMUL ECX  
00498A69   .  8BC2          MOV EAX,EDX  
00498A6B   .  C1F8 05       SAR EAX,5  
00498A6E   .  8BC8          MOV ECX,EAX  
00498A70   .  C1E9 1F       SHR ECX,1F  
00498A73   .  03C1          ADD EAX,ECX  
00498A75   .  B9 0A000000   MOV ECX,0A  
00498A7A   .  03C6          ADD EAX,ESI  
00498A7C   .  25 FFFF0000   AND EAX,0FFFF  
00498A81   .  99             CDQ  
00498A82   .  F7F9          IDIV ECX  
00498A84   .  66:83FA 05    CMP DX,5                         ;  以上代码生成的随机结果，随机参与下列验证  
00498A88   .  76 13         JBE SHORT 00498A9D  
00498A8A   .  80FB 41       CMP BL,41  
00498A8D   .  7E 40         JLE SHORT 00498ACF  
00498A8F   .  80FB 7A       CMP BL,7A                         ;  0x41 < S0[4] < 0x7A 时通过  
00498A92   .  7D 3B         JGE SHORT 00498ACF  
00498A94   .  5F             POP EDI  
00498A95   .  5E             POP ESI  
00498A96   .  B8 01000000   MOV EAX,1  
00498A9B   .  5B             POP EBX  
00498A9C   .  C3             RETN  
00498A9D   >  66:83FA 03    CMP DX,3  
00498AA1   .  76 13         JBE SHORT 00498AB6  
00498AA3   .  80FB 41       CMP BL,41  
00498AA6   .  7E 27         JLE SHORT 00498ACF  
00498AA8   .  80FB 5A       CMP BL,5A                         ;  0x41 < S0[4] < 0x5A 时通过  
00498AAB   .  7D 22         JGE SHORT 00498ACF  
00498AAD   .  5F             POP EDI  
00498AAE   .  5E             POP ESI  
00498AAF   .  B8 01000000   MOV EAX,1  
00498AB4   .  5B             POP EBX  
00498AB5   .  C3             RETN  
00498AB6   >  66:83FA 01    CMP DX,1  
00498ABA   .  76 19         JBE SHORT 00498AD5  
00498ABC   .  80FB 4C       CMP BL,4C  
00498ABF   .  7C 0E         JL SHORT 00498ACF  
00498AC1   .  80FB 4D       CMP BL,4D                         ;  0x4C <= S0[4] <= 0x4D 时通过  
00498AC4   .  7F 09         JG SHORT 00498ACF  
00498AC6   .  5F             POP EDI  
00498AC7   .  5E             POP ESI  
00498AC8   .  B8 01000000   MOV EAX,1  
00498ACD   .  5B             POP EBX  
00498ACE   .  C3             RETN  
00498ACF   >  5F             POP EDI  
00498AD0   .  5E             POP ESI  
00498AD1   .  33C0          XOR EAX,EAX                       ;  这里就要返回 0 了  
00498AD3   .  5B             POP EBX  
00498AD4   .  C3             RETN  
```

★ 第一个条件： `0x4C <= S0[4] <= 0x4D`  

`/////////////////////////////////////   第 2 个   //////////////////////////////////////`  

```
004989A0   .  FF15 D0684A00 CALL DWORD PTR [<&MSVCRT.rand>]    ; [rand  
004989A6   .  25 1F000080   AND EAX,8000001F  
004989AB   .  79 05         JNS SHORT 004989B2  
004989AD   .  48             DEC EAX  
004989AE   .  83C8 E0        OR EAX,FFFFFFE0  
004989B1   .  40             INC EAX  
004989B2   >  83F8 14       CMP EAX,14  
004989B5   .  7E 06         JLE SHORT 004989BD  
004989B7   .  B8 01000000   MOV EAX,1  
004989BC   .  C3             RETN  
004989BD   >  83F8 0A       CMP EAX,0A  
004989C0   .  7E 15         JLE SHORT 004989D7  
004989C2   .  8B4424 04     MOV EAX,DWORD PTR [ESP+4]  
004989C6   .  8A40 02       MOV AL,BYTE PTR [EAX+2]           ;  取 S0[2]  
004989C9   .  3C 41         CMP AL,41  
004989CB   .  7E 39         JLE SHORT 00498A06  
004989CD   .  3C 5A         CMP AL,5A  
004989CF   .  7D 35         JGE SHORT 00498A06                ;  0x41 < S0[2] < 0x5A  
004989D1   .  B8 01000000   MOV EAX,1  
004989D6   .  C3             RETN  
004989D7   >  83F8 05       CMP EAX,5  
004989DA   .  7E 15         JLE SHORT 004989F1  
004989DC   .  8B4C24 04     MOV ECX,DWORD PTR [ESP+4]  
004989E0   .  8A41 02       MOV AL,BYTE PTR [ECX+2]           ;  取 S0[2]  
004989E3   .  3C 43         CMP AL,43  
004989E5   .  7C 1F         JL SHORT 00498A06  
004989E7   .  3C 5A         CMP AL,5A  
004989E9   .  7D 1B         JGE SHORT 00498A06                ;  0x43 <= S0[2] < 0x5A  
004989EB   .  B8 01000000   MOV EAX,1  
004989F0   .  C3             RETN  
```



```
004989F1   >  8B5424 04      MOV EDX,DWORD PTR [ESP+4]           ;  TPhoto.004CAC3C

004989F5   .  8A42 02        MOV AL,BYTE PTR [EDX+2]             ;  取 S0[2]

004989F8   .  3C 43         CMP AL,43

004989FA   .  7C 0A         JL SHORT 00498A06

004989FC   .  3C 4C         CMP AL,4C

004989FE   .  7F 06         JG SHORT 00498A06                     ;  0x43 <= S0[2] <= 0x4C

00498A00   .  B8 01000000   MOV EAX,1

00498A05   .  C3            RETN

00498A06   >  33C0          XOR EAX,EAX                           ;  到这里就返回 0 了

00498A08   .  C3            RETN
```

我们发现即使进入验证地址，他的验证仍然采用了随机验证，每次条件不同，我们要找到符合所有条件的

★ 第二个条件 ：`0x43 <= S0[2] <= 0x4C`

`/////////////////////////////////////   第 3 个   /////////////////////////////////////`

有很多产生随机数，并根据随机数选择不同的条件，下面省略这些代码：

```
0049895C   .  8D4C24 04     LEA ECX,DWORD PTR [ESP+4]

00498960   .  8BC4          MOV EAX,ESP

00498962   .  C700 05000000 MOV DWORD PTR [EAX],5

00498968   .  E8 B30B0000   CALL 00499520

0049896D   .  85C0          TEST EAX,EAX

0049896F   .  74 11         JE SHORT 00498982

00498971   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]

00498975   .  33C0          XOR EAX,EAX

00498977   .  8079 01 41    CMP BYTE PTR [ECX+1],41               ;  S0[1] >= 0x41

0049897B   .  0F9DC0        SETGE AL

0049897E   .  83C4 08       ADD ESP,8

00498981   .  C3            RETN

00498982   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]

00498986   .  8A42 01       MOV AL,BYTE PTR [EDX+1]

00498989   .  3C 41         CMP AL,41

0049898B   .  7C 0D         JL SHORT 0049899A

0049898D   .  3C 46         CMP AL,46

0049898F   .  7D 09         JGE SHORT 0049899A                    ;  0x41 <= S0[1] < 0x46

00498991   .  B8 01000000   MOV EAX,1

00498996   .  83C4 08       ADD ESP,8

00498999   .  C3            RETN

0049899A   >  33C0          XOR EAX,EAX                           ;  到这里就返回 0 了

0049899C   .  83C4 08       ADD ESP,8

0049899F   .  C3            RETN
```

★ 第三个条件 ：`0x41 <= S0[1] < 0x46`

`/////////////////////////////////////   第 4 个   /////////////////////////////////////`

```
00498A10   .  8B4424 04     MOV EAX,DWORD PTR [ESP+4]

00498A14   .  8A40 03       MOV AL,BYTE PTR [EAX+3]               ;  取 S0[3]

00498A17   .  3C 54         CMP AL,54

00498A19   .  7C 0A         JL SHORT 00498A25                     ;  S0[3] >= 0x54

00498A1B   .  3C 5A         CMP AL,5A

00498A1D   .  7D 06         JGE SHORT 00498A25                    ;  S0[3] < 0x5A

00498A1F   .  B8 01000000   MOV EAX,1

00498A24   .  C3            RETN

00498A25   >  33C0          XOR EAX,EAX                           ;  到这里就返回 0 了

00498A27   .  C3            RETN
```

★ 第四个条件：`0x54 <= S0[3] < 0x5A`

`/////////////////////////////////////   第 5 个   /////////////////////////////////////`

```
004988CB   >  \83F8 03      CMP EAX,3                             ;  Switch (cases 0..2)

004988CE   .  77 34         JA SHORT 00498904

004988D0   .  FF2485 0C8949>JMP DWORD PTR [EAX*4+49890C]

004988D7   >  8B4C24 04     MOV ECX,DWORD PTR [ESP+4]             ;  Case 0 of switch 004988CB

004988DB   .  8A01          MOV AL,BYTE PTR [ECX]                 ;  取 S0[0]

004988DD   .  3C 41         CMP AL,41

004988DF   .  7E 20         JLE SHORT 00498901

004988E1   .  3C 7A         CMP AL,7A                               ;  0x41 < S0[0] < 0x7A

004988E3   .  EB 1A         JMP SHORT 004988FF

004988E5   >  8B5424 04     MOV EDX,DWORD PTR [ESP+4]             ;  Case 1 of switch 004988CB

004988E9   .  8A02          MOV AL,BYTE PTR [EDX]

004988EB   .  3C 41         CMP AL,41

004988ED   .  7E 12         JLE SHORT 00498901

004988EF   .  3C 5A         CMP AL,5A                               ;  0x41 < S0[0] < 0x5A

004988F1   .  EB 0C         JMP SHORT 004988FF

004988F3   >  8B4424 04     MOV EAX,DWORD PTR [ESP+4]             ;  Case 2 of switch 004988CB

004988F7   .  8A00          MOV AL,BYTE PTR [EAX]

004988F9   .  3C 44         CMP AL,44

004988FB   .  7C 04         JL SHORT 00498901

004988FD   .  3C 45         CMP AL,45

004988FF   >  7C 03         JL SHORT 00498904                     ;  0x44 <= S0[0] < 0x45

00498901   >  33C0          XOR EAX,EAX                           ;  到这里就返回 0 了

00498903   .  C3            RETN

00498904   >  B8 01000000   MOV EAX,1                             ;  Default case of switch 004988CB

00498909   .  C3            RETN
```

★ 第五个条件：`S0[0]==0x44` ，即 `'D'`

`/////////////////////////////////////   第 6 个   /////////////////////////////////////`

```
00498D16   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]

00498D1A   .  56            PUSH ESI

00498D1B   .  BE 07000000   MOV ESI,7

00498D20   .  57            PUSH EDI

00498D21   .  0FBE41 04     MOVSX EAX,BYTE PTR [ECX+4]            ;  取 S0[4]

00498D25   .  99            CDQ

00498D26   .  F7FE          IDIV ESI                                ;  S0[0] % 7

00498D28   .  0FBE41 02     MOVSX EAX,BYTE PTR [ECX+2]

00498D2C   .  BF 05000000   MOV EDI,5

00498D31   .  8BF2          MOV ESI,EDX

00498D33   .  99            CDQ

00498D34   .  F7FF          IDIV EDI                                ;  S0[2] % 5

00498D36   .  5F            POP EDI

00498D37   .  03F2          ADD ESI,EDX                             ;  两个余数相加

00498D39   .  0FBE51 01     MOVSX EDX,BYTE PTR [ECX+1]

00498D3D   .  0FBE09        MOVSX ECX,BYTE PTR [ECX]

00498D40   .  8D0456        LEA EAX,DWORD PTR [ESI+EDX*2]         ;  结果 + S0[1] * 2

00498D43   .  5E            POP ESI

00498D44   .  03C1          ADD EAX,ECX                             ;  结果 + S0[0]

00498D46   .  B9 1A000000   MOV ECX,1A

00498D4B   .  99            CDQ

00498D4C   .  F7F9          IDIV ECX                                ;  结果 % 26

00498D4E   .  8B4424 18     MOV EAX,DWORD PTR [ESP+18]

00498D52   .  0FBE08        MOVSX ECX,BYTE PTR [EAX]

00498D55   .  33C0          XOR EAX,EAX

00498D57   .  80C2 41       ADD DL,41                               ;  余数 + 0x41

00498D5A   .  81E2 FF000000 AND EDX,0FF

00498D60   .  3BCA          CMP ECX,EDX                             ;  跟 S3[0] 比较是否一样

00498D62   .  0F94C0        SETE AL

00498D65   .  83C4 08       ADD ESP,8

00498D68   .  C3            RETN

00498D69   >  83F8 14       CMP EAX,14

00498D6C   .  7E 17         JLE SHORT 00498D85

00498D6E   .  8B5424 18     MOV EDX,DWORD PTR [ESP+18]

00498D72   .  8A02          MOV AL,BYTE PTR [EDX]                   ;  取得 S3[0]

00498D74   .  3C 41         CMP AL,41

00498D76   .  7C 24         JL SHORT 00498D9C

00498D78   .  3C 64         CMP AL,64

00498D7A   .  7D 20         JGE SHORT 00498D9C                    ;  0x41 <= S3[0] < 0x64

00498D7C   .  B8 01000000   MOV EAX,1

00498D81   .  83C4 08       ADD ESP,8

00498D84   .  C3            RETN

00498D85   >  8B4424 18     MOV EAX,DWORD PTR [ESP+18]

00498D89   .  8A00          MOV AL,BYTE PTR [EAX]                   ;  取得 S3[0]

00498D8B   .  3C 20         CMP AL,20

00498D8D   .  7E 0D         JLE SHORT 00498D9C

00498D8F   .  3C 72         CMP AL,72

00498D91   .  7D 09         JGE SHORT 00498D9C                    ;  0x20 < S3[0] < 0x72

00498D93   .  B8 01000000   MOV EAX,1

00498D98   .  83C4 08       ADD ESP,8

00498D9B   .  C3            RETN

00498D9C   >  33C0          XOR EAX,EAX                           ;  到这里就返回 0 了

00498D9E   .  83C4 08       ADD ESP,8

00498DA1   .  C3            RETN
```

★ 第六个条件：`S3[0] = ( S0[4]%7 + S0[2]%5 + S0[1]*2 + S0[0] )%26 + 0x41`

`/////////////////////////////////////   第 7 个   /////////////////////////////////////`

```
00498DD7   .  8B4C24 10     MOV ECX,DWORD PTR [ESP+10]

00498DDB   .  56            PUSH ESI

00498DDC   .  BE 03000000   MOV ESI,3

00498DE1   .  0FBE41 03     MOVSX EAX,BYTE PTR [ECX+3]            ;  取 S1[3]

00498DE5   .  99            CDQ

00498DE6   .  F7FE          IDIV ESI                                ;  S1[3] % 3
```



## 第七个条件

```
00498DE8   .  0FBE41 04      MOVSX EAX,BYTE PTR [ECX+4]
00498DEC   .  5E             POP ESI
00498DED   .  8D0480         LEA EAX,DWORD PTR [EAX+EAX*4]     ;  S1[4] * 5
00498DF0   .  03D0           ADD EDX,EAX                   ;  两结果相加
00498DF2   .  0FBE01         MOVSX EAX,BYTE PTR [ECX]
00498DF5   .  8D0442         LEA EAX,DWORD PTR [EDX+EAX*2]     ;  结果 + S1[0] * 2
00498DF8   .  0FBE51 02      MOVSX EDX,BYTE PTR [ECX+2]
00498DFC   .  0FBE49 01      MOVSX ECX,BYTE PTR [ECX+1]
00498E00   .  03C2           ADD EAX,EDX                   ;  结果 + S1[2]
00498E02   .  03C1           ADD EAX,ECX                   ;  结果 + S1[1]
00498E04   .  B9 1A000000    MOV ECX,1A
00498E09   .  99             CDQ
00498E0A   .  F7F9           IDIV ECX                     ;  结果 % 26
00498E0C   .  8B4424 18      MOV EAX,DWORD PTR [ESP+18]
00498E10   .  0FBE48 01      MOVSX ECX,BYTE PTR [EAX+1]
00498E14   .  33C0           XOR EAX,EAX
00498E16   .  80C2 41        ADD DL,41                    ;  结果 + 0x41
00498E19   .  81E2 FF000000  AND EDX,0FF
00498E1F   .  3BCA           CMP ECX,EDX                   ;  结果跟 S3[1] 是否一样
00498E21   .  0F94C0         SETE AL
00498E24   .  83C4 08        ADD ESP,8
00498E27   .  C3             RETN

00498E28   >  83F8 02        CMP EAX,2
00498E2B   .  7E 18          JLE SHORT 00498E45
00498E2D   .  8B5424 18      MOV EDX,DWORD PTR [ESP+18]
00498E31   .  8A42 01        MOV AL,BYTE PTR [EDX+1]       ;  取 S3[1]
00498E34   .  3C 41          CMP AL,41
00498E36   .  7C 25          JL SHORT 00498E5D
00498E38   .  3C 61          CMP AL,61
00498E3A   .  7D 21          JGE SHORT 00498E5D             ;  0x41 <= S3[1] <= 0x61
00498E3C   .  B8 01000000    MOV EAX,1
00498E41   .  83C4 08        ADD ESP,8
00498E44   .  C3             RETN

00498E45   >  8B4424 18      MOV EAX,DWORD PTR [ESP+18]
00498E49   .  8A40 01        MOV AL,BYTE PTR [EAX+1]       ;  取 S3[1]
00498E4C   .  3C 20          CMP AL,20
00498E4E   .  7E 0D          JLE SHORT 00498E5D
00498E50   .  3C 6B          CMP AL,6B
00498E52   .  7D 09          JGE SHORT 00498E5D             ;  0x20 < S3[1] < 0x6B
00498E54   .  B8 01000000    MOV EAX,1
00498E59   .  83C4 08        ADD ESP,8
00498E5C   .  C3             RETN

00498E5D   >  33C0           XOR EAX,EAX
00498E5F   .  83C4 08        ADD ESP,8
00498E62   .  C3             RETN
```

★ 第七个条件：`S3[1] = ( S1[3]%3 + S1[4]*5 + S1[0]*2 + S1[2] + S1[1] )%26 + 0x41`

## 第 8 个

```
00498EA1   .  8B5424 14      MOV EDX,DWORD PTR [ESP+14]
00498EA5   .  0FBE4A 03      MOVSX ECX,BYTE PTR [EDX+3]     ;  取 S2[3]
00498EA9   .  8D0449         LEA EAX,DWORD PTR [ECX+ECX*2]     ;  S2[3] * 3
00498EAC   .  C1E0 03        SHL EAX,3                  ;  结果 * 8
00498EAF   .  2BC1           SUB EAX,ECX                  ;  结果 - S2[3] 以上即 S2[3] * 23
00498EB1   .  0FBE4A 05      MOVSX ECX,BYTE PTR [EDX+5]
00498EB5   .  0FBE12         MOVSX EDX,BYTE PTR [EDX]
00498EB8   .  03C1           ADD EAX,ECX                  ;  结果 + S2[5]
00498EBA   .  B9 1A000000    MOV ECX,1A
00498EBF   .  03C2           ADD EAX,EDX                  ;  结果 + S2[0]
00498EC1   .  99             CDQ
00498EC2   .  F7F9           IDIV ECX                     ;  结果 % 26
00498EC4   .  8B4424 18      MOV EAX,DWORD PTR [ESP+18]
00498EC8   .  0FBE48 02      MOVSX ECX,BYTE PTR [EAX+2]
00498ECC   .  33C0           XOR EAX,EAX
00498ECE   .  80C2 41        ADD DL,41                    ;  结果 + 0x41
00498ED1   .  81E2 FF000000  AND EDX,0FF
00498ED7   .  3BCA           CMP ECX,EDX                  ;  结果跟 S3[2] 是否一样
00498ED9   .  0F94C0         SETE AL
00498EDC   .  83C4 08        ADD ESP,8
00498EDF   .  C3             RETN

00498EE0   >  83F8 02        CMP EAX,2
00498EE3   .  7E 18          JLE SHORT 00498EFD
00498EE5   .  8B5424 18      MOV EDX,DWORD PTR [ESP+18]
00498EE9   .  8A42 02        MOV AL,BYTE PTR [EDX+2]       ;  取 S3[2]
00498EEC   .  3C 41          CMP AL,41
00498EEE   .  7C 25          JL SHORT 00498F15
00498EF0   .  3C 62          CMP AL,62
00498EF2   .  7D 21          JGE SHORT 00498F15             ;  0x41 <= S3[2] < 0x62
00498EF4   .  B8 01000000    MOV EAX,1
00498EF9   .  83C4 08        ADD ESP,8
00498EFC   .  C3             RETN

00498EFD   >  8B4424 18      MOV EAX,DWORD PTR [ESP+18]
00498F01   .  8A40 02        MOV AL,BYTE PTR [EAX+2]       ;  取 S3[2]
00498F04   .  3C 20          CMP AL,20
00498F06   .  7E 0D          JLE SHORT 00498F15
00498F08   .  3C 68          CMP AL,68
00498F0A   .  7D 09          JGE SHORT 00498F15             ;  0x20 < S3[2] < 0x68
00498F0C   .  B8 01000000    MOV EAX,1
00498F11   .  83C4 08        ADD ESP,8
00498F14   .  C3             RETN

00498F15   >  33C0           XOR EAX,EAX                  ;  到这里就返回 0 了
00498F17   .  83C4 08        ADD ESP,8
00498F1A   .  C3             RETN
```

★ 第八个条件是：`S3[2] = ( S2[3]*23 + S2[5] + S2[0] )%26 + 0x41`

## 第 9 个

```
00498F2A   .  8B5424 0C      MOV EDX,DWORD PTR [ESP+C]
00498F2E   .  0FBE4A 01      MOVSX ECX,BYTE PTR [EDX+1]     ;  取 S2[1]
00498F32   .  0FBE52 02      MOVSX EDX,BYTE PTR [EDX+2]     ;  取 S2[2]
00498F36   .  8D04CD 000000>LEA EAX,DWORD PTR [ECX*8]       ;  S2[1] * 8
00498F3D   .  2BC1           SUB EAX,ECX                  ;  结果 - S2[1] 以上即 S2[1] * 7
00498F3F   .  8B4C24 04      MOV ECX,DWORD PTR [ESP+4]
00498F43   .  0FBE49 03      MOVSX ECX,BYTE PTR [ECX+3]
00498F47   .  03C1           ADD EAX,ECX                  ;  结果 + S0[3]
00498F49   .  8B4C24 08      MOV ECX,DWORD PTR [ESP+8]
00498F4D   .  0FBE49 02      MOVSX ECX,BYTE PTR [ECX+2]
00498F51   .  03C1           ADD EAX,ECX                  ;  结果 + S1[2]
00498F53   .  B9 1A000000    MOV ECX,1A
00498F58   .  03C2           ADD EAX,EDX                  ;  结果 + S2[2]
00498F5A   .  99             CDQ
00498F5B   .  F7F9           IDIV ECX                     ;  结果 % 26
00498F5D   .  8B4424 10      MOV EAX,DWORD PTR [ESP+10]
00498F61   .  0FBE48 03      MOVSX ECX,BYTE PTR [EAX+3]
00498F65   .  33C0           XOR EAX,EAX
00498F67   .  80C2 41        ADD DL,41                    ;  结果 + 0x41
00498F6A   .  81E2 FF000000  AND EDX,0FF
00498F70   .  3BCA           CMP ECX,EDX                  ;  结果跟 S3[3] 是否一样
00498F72   .  0F94C0         SETE AL
00498F75   .  C3             RETN

00498F76   >  83F8 02        CMP EAX,2
00498F79   .  7E 15          JLE SHORT 00498F90
00498F7B   .  8B5424 10      MOV EDX,DWORD PTR [ESP+10]
00498F7F   .  8A42 03        MOV AL,BYTE PTR [EDX+3]       ;  取 S3[3]
00498F82   .  3C 41          CMP AL,41
00498F84   .  7C 1F          JL SHORT 00498FA5
00498F86   .  3C 64          CMP AL,64
00498F88   .  7D 1B          JGE SHORT 00498FA5             ;  0x41 <= S3[3] < 0x64
00498F8A   .  B8 01000000    MOV EAX,1
00498F8F   .  C3             RETN

00498F90   >  8B4424 10      MOV EAX,DWORD PTR [ESP+10]
00498F94   .  8A40 03        MOV AL,BYTE PTR [EAX+3]       ;  取 S3[3]
00498F97   .  3C 20          CMP AL,20
00498F99   .  7E 0A          JLE SHORT 00498FA5
00498F9B   .  3C 64          CMP AL,64
00498F9D   .  7D 06          JGE SHORT 00498FA5             ;  0x20 < S3[3] < 0x64
00498F9F   .  B8 01000000    MOV EAX,1
00498FA4   .  C3             RETN

00498FA5   >  33C0           XOR EAX,EAX                  ;  到这里就返回 0 了
00498FA7   .  C3             RETN
```

★ 第九个条件是：`S3[3] = ( S2[1]*7 + S0[3] + S1[2] + S2[2] )%26 + 0x41`



## 第 10 个

```
00498FB0   .  B8 01000000   MOV EAX,1
00498FB5   .  C3            RETN
```

★ 第十个条件：无条件

## 第 11 个

```
00498BFA   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498BFE   .  8A40 03       MOV AL,BYTE PTR [EAX+3]          ;  取 S2[3]
00498C01   .  3C 49         CMP AL,49
00498C03   .  7C 39         JL SHORT 00498C3E
00498C05   .  3C 52         CMP AL,52
00498C07   .  7F 35         JG SHORT 00498C3E                ;  0x49 <= S2[3] <= 0x52
00498C09   .  B8 01000000   MOV EAX,1
00498C0E   .  C3            RETN
00498C0F   >  83F8 05       CMP EAX,5
00498C12   .  7E 15         JLE SHORT 00498C29
00498C14   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498C18   .  8A41 03       MOV AL,BYTE PTR [ECX+3]          ;  取 S2[3]
00498C1B   .  3C 41         CMP AL,41
00498C1D   .  7E 1F         JLE SHORT 00498C3E
00498C1F   .  3C 52         CMP AL,52
00498C21   .  7F 1B         JG SHORT 00498C3E                ;  0x41 < S2[3] <= 0x52
00498C23   .  B8 01000000   MOV EAX,1
00498C28   .  C3            RETN
00498C29   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]
00498C2D   .  8A42 03       MOV AL,BYTE PTR [EDX+3]
00498C30   .  3C 41         CMP AL,41
00498C32   .  7C 0A         JL SHORT 00498C3E
00498C34   .  3C 5A         CMP AL,5A
00498C36   .  7F 06         JG SHORT 00498C3E
00498C38   .  B8 01000000   MOV EAX,1
00498C3D   .  C3            RETN
00498C3E   >  33C0          XOR EAX,EAX                      ;  到这里就返回 0 了
00498C40   .  C3            RETN
```

★ 第十一个条件：`0x49` <= `S2[3]` <= `0x52`

## 第 12 个

```
00498B9A   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498B9E   .  8A40 02       MOV AL,BYTE PTR [EAX+2]          ;  取 S2[2]
00498BA1   .  3C 42         CMP AL,42
00498BA3   .  7C 39         JL SHORT 00498BDE
00498BA5   .  3C 4B         CMP AL,4B
00498BA7   .  7F 35         JG SHORT 00498BDE                ;  0x42 <= S2[2] <= 0x4B
00498BA9   .  B8 01000000   MOV EAX,1
00498BAE   .  C3            RETN
00498BAF   >  83F8 05       CMP EAX,5
00498BB2   .  7E 15         JLE SHORT 00498BC9
00498BB4   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498BB8   .  8A41 02       MOV AL,BYTE PTR [ECX+2]          ;  取 S2[2]
00498BBB   .  3C 20         CMP AL,20
00498BBD   .  7E 1F         JLE SHORT 00498BDE
00498BBF   .  3C 4B         CMP AL,4B
00498BC1   .  7F 1B         JG SHORT 00498BDE                ;  0x20 < S2[2] <= 0x4B
00498BC3   .  B8 01000000   MOV EAX,1
00498BC8   .  C3            RETN
00498BC9   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]
00498BCD   .  8A42 02       MOV AL,BYTE PTR [EDX+2]
00498BD0   .  3C 41         CMP AL,41
00498BD2   .  7C 0A         JL SHORT 00498BDE
00498BD4   .  3C 5A         CMP AL,5A
00498BD6   .  7F 06         JG SHORT 00498BDE                ;  0x41 <= S2[2] <= 0x5A
00498BD8   .  B8 01000000   MOV EAX,1
00498BDD   .  C3            RETN
00498BDE   >  33C0          XOR EAX,EAX                      ;  到这里就返回 0 了
00498BE0   .  C3            RETN
```

★ 第十二个条件是：`0x42` <= `S2[2]` <= `0x4B`

## 第 13 个

```
00498B4A   .  8B4424 08     MOV EAX,DWORD PTR [ESP+8]
00498B4E   .  8A40 01       MOV AL,BYTE PTR [EAX+1]          ;  取 S1[1]
00498B51   .  3C 41         CMP AL,41
00498B53   .  7E 04         JLE SHORT 00498B59
00498B55   .  3C 5A         CMP AL,5A
00498B57   .  7C 29         JL SHORT 00498B82                ;  0x41 < S1[1] < 0x5A
00498B59   >  33C0          XOR EAX,EAX                      ;  到这里就返回 0 了
00498B5B   .  C3            RETN
00498B5C   >  83F8 04       CMP EAX,4
00498B5F   .  7C 0E         JL SHORT 00498B6F
00498B61   .  8B4C24 08     MOV ECX,DWORD PTR [ESP+8]
00498B65   .  33C0          XOR EAX,EAX
00498B67   .  8079 01 41    CMP BYTE PTR [ECX+1],41          ;  S1[1] > 0x41
00498B6B   .  0F9FC0        SETG AL
00498B6E   .  C3            RETN
00498B6F   >  83F8 02       CMP EAX,2
00498B72   .  7C 0E         JL SHORT 00498B82
00498B74   .  8B5424 08     MOV EDX,DWORD PTR [ESP+8]
00498B78   .  33C0          XOR EAX,EAX
00498B7A   .  807A 01 7A    CMP BYTE PTR [EDX+1],7A          ;  S1[1] < 0x7A
00498B7E   .  0F9CC0        SETL AL
00498B81   .  C3            RETN
00498B82   >  B8 01000000   MOV EAX,1                        ;  到这里就返回 1 了
00498B87   .  C3            RETN
```

★ 第十三个条件：`0x41` < `S1[1]` < `0x5A`

## 第 14 个

```
00498C5A   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498C5E   .  8A40 04       MOV AL,BYTE PTR [EAX+4]          ;  取 S2[4]
00498C61   .  3C 4B         CMP AL,4B
00498C63   .  7C 39         JL SHORT 00498C9E
00498C65   .  3C 54         CMP AL,54
00498C67   .  7F 35         JG SHORT 00498C9E                ;  0x4B <= S2[4] <= 0x54
00498C69   .  B8 01000000   MOV EAX,1
00498C6E   .  C3            RETN
00498C6F   >  83F8 02       CMP EAX,2
00498C72   .  7E 15         JLE SHORT 00498C89
00498C74   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498C78   .  8A41 04       MOV AL,BYTE PTR [ECX+4]          ;  取 S2[4]
00498C7B   .  3C 41         CMP AL,41
00498C7D   .  7E 1F         JLE SHORT 00498C9E
00498C7F   .  3C 54         CMP AL,54
00498C81   .  7F 1B         JG SHORT 00498C9E                ;  0x41 < S2[4] <= 0x54
00498C83   .  B8 01000000   MOV EAX,1
00498C88   .  C3            RETN
00498C89   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]
00498C8D   .  8A42 04       MOV AL,BYTE PTR [EDX+4]
00498C90   .  3C 41         CMP AL,41
00498C92   .  7C 0A         JL SHORT 00498C9E
00498C94   .  3C 5A         CMP AL,5A
00498C96   .  7F 06         JG SHORT 00498C9E                ;  0x41 <= S2[4] <= 0x5A
00498C98   .  B8 01000000   MOV EAX,1
00498C9D   .  C3            RETN
00498C9E   >  33C0          XOR EAX,EAX                      ;  到这里就返回 0 了
00498CA0   .  C3            RETN
```

★ 第十四个条件：`0x4B` <= `S2[4]` <= `0x54`

## 第 15 个

```
00498AFA   .  8B4424 08     MOV EAX,DWORD PTR [ESP+8]
00498AFE   .  8A00          MOV AL,BYTE PTR [EAX]            ;  取 S1[0]
00498B00   .  3C 41         CMP AL,41
00498B02   .  7E 04         JLE SHORT 00498B08
00498B04   .  3C 5A         CMP AL,5A                         ;  0x41 < S1[0] < 0x5A
00498B06   .  7C 27         JL SHORT 00498B2F
00498B08   >  33C0          XOR EAX,EAX                      ;  到这里就返回 0 了
00498B0A   .  C3            RETN
00498B0B   >  83F8 04       CMP EAX,4
00498B0E   .  7C 0D         JL SHORT 00498B1D
00498B10   .  8B4C24 08     MOV ECX,DWORD PTR [ESP+8]
00498B14   .  33C0          XOR EAX,EAX
00498B16   .  8039 41       CMP BYTE PTR [ECX],41            ;  S1[0] > 0x41
00498B19   .  0F9FC0        SETG AL
00498B1C   .  C3            RETN
00498B1D   >  83F8 02       CMP EAX,2
00498B20   .  7C 0D         JL SHORT 00498B2F
00498B22   .  8B5424 08     MOV EDX,DWORD PTR [ESP+8]
00498B26   .  33C0          XOR EAX,EAX
00498B28   .  803A 7A       CMP BYTE PTR [EDX],7A            ;  S1[0] < 0x7A
00498B2B   .  0F9CC0        SETL AL
00498B2E   .  C3            RETN
00498B2F   >  B8 01000000   MOV EAX,1                        ;  到这里就返回 1 了
00498B34   .  C3            RETN
```

★ 第十五个条件：`0x41` < `S1[0]` < `0x5A`

## 第 16 个



```
00498CBA   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498CBE   .  8A40 05       MOV AL,BYTE PTR [EAX+5]         ;  取 S2[5]
00498CC1   .  3C 4A         CMP AL,4A
00498CC3   .  7C 1F         JL SHORT 00498CE4
00498CC5   .  3C 53         CMP AL,53
00498CC7   .  7F 1B         JG SHORT 00498CE4               ;  0x4A <= S2[5] <= 0x53
00498CC9   .  B8 01000000  MOV EAX,1
00498CCE   .  C3           RETN
00498CCF   >  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498CD3   .  8A41 05       MOV AL,BYTE PTR [ECX+5]         ;  取 S2[5]
00498CD6   .  3C 41         CMP AL,41
00498CD8   .  7C 0A         JL SHORT 00498CE4
00498CDA   .  3C 5A         CMP AL,5A
00498CDC   .  7F 06         JG SHORT 00498CE4               ;  0x41 <= S2[5] <= 0x5A
00498CDE   .  B8 01000000  MOV EAX,1
00498CE3   .  C3           RETN
00498CE4   >  33C0         XOR EAX,EAX                       ;  到这里就返回 0 了
00498CE6   .  C3           RETN
```

★ 第十六个条件：`0x4A <= S2[5] <= 0x53`

----------------------------------------------------------------------------------------

【破解心得】

把十六个条件整理一下：就可以制作注册机了，另外程序中又很多方法生成随机数，也值得借鉴一下(文中已经略去，可以自己跟一下)

个人感觉作者使用了消磨 `Cracker` 耐力的办法了阻挡破解，也有一定效果，如果程序被做出 `Keygen`，作者只要稍许修改几个数据，就会使得 `Cracker` 又要花上很多的精力去跟程序。

但是这种方法也有缺陷：

因为注册码跟硬件无关，作者通过频繁更换注册方法阻止算法注册机的出现，但是新版本注册算法的变化会导致正版用户不得不不断的更换注册码来享受新版本。

另外程序在防止暴力破解上做的不够。

建议：

某些位置的注册码跟硬件有关，这样特定的电脑上只能使用某些注册码

可以将 `16` 个验证放在程序的不同位置，不一定非要放在按钮的响应代码中，比如程序启动时或使用到某个功能时随机选择不同的代码验证。

----------------------------------------------------------------------------------------

【注册机源码】

```
void CKeygenDlg::OnOK()

{
  // TODO: Add extra validation here
  char S0[7],S1[7],S2[8],S3[6];

  //S1
  S0[0]='D';
  S0[5]='-';
  S0[6]=0;
  do{
  S0[1]='A'+ (rand()%5);
  S0[2]='C'+ (rand()%10);
  S0[3]='T'+ (rand()%6);
  S0[4]='L'+ (rand()%2);}
  while ( !strcmp(S0,"DDGTM-") );

  //S2
  S1[0]='B'+ (rand()%24);
  S1[1]='B'+ (rand()%24);
  S1[2]='A'+ (rand()%26);
  S1[3]='A'+ (rand()%26);
  S1[4]='A'+ (rand()%26);
  S1[5]='-';
  S1[6]=0;

  //S3
  S2[0]='A'+ (rand()%26);
  S2[1]='A'+ (rand()%26);
  S2[2]='B'+ (rand()%10);
  S2[3]='I'+ (rand()%10);
  S2[4]='K'+ (rand()%10);
  S2[5]='J'+ (rand()%10);
  S2[6]='-';
  S2[7]=0;

  //S4
  S3[0]=(S0[4]%7+S0[2]%5+(S0[1]<<1)+S0[0])%26+0x41;
  S3[1]=(S1[3]%3+S1[4]*5+(S1[0]<<1)+S1[2]+S1[1])%26 + 0x41;
  S3[2]=(S2[3]*23+S2[5]+S2[0])%26+0x41;
  S3[3]=(S2[1]*7+S0[3]+S1[2]+S2[2])%26+0x41;
  S3[4]='A'+ (rand()%26);
  S3[5]=0;

  m_Edit1 = S0;
  m_Edit1 += S1;
  m_Edit1 += S2;
  m_Edit1 += S3;

  UpdateData(false);

}
```

----------------------------------------------------------------------------------------

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享:)

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

----------------------------------------------------------------------------------------

文章写于 2006-2-12 16:53:49

`4dd466d0e1e0cc123afbe504da2462ca`
