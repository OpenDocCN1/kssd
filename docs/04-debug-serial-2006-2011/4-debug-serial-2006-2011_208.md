
# 双色球大赢家 2.04

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-450.htm`](https://www.pediy.com/kssd/pediy08/pediy8-450.htm)

【文章标题】: XX 大赢家 2.04

【文章作者】: rdsnow\[BCG\]\[PYG\]\[D.4s\]

【作者邮箱】: rdsnow@163.com

【作者主页】: [`rdsnow.ys168.com`](http://rdsnow.ys168.com)

【作者 QQ 号】: 83757177

【软件名称】: 双色球大赢家 2.04

【下载地址】: [`down.zcdyj.com/down/ssqdyj/SSQDYJ2000.exe`](http://down.zcdyj.com/down/ssqdyj/SSQDYJ2000.exe)

【保护方式】: 序列号、功能限制

【使用工具】: ODbyDYK v1.10\[05.09\]

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

\--------------------------------------------------------------------------------

【文章简介】

真后悔本次世界杯没有买足球彩票，16 强全中，汗，马上世界杯要 Over 了，找个玩双色球的软件玩玩，安慰下自己。

程序采用了如果 `F1( 机器码 ) ==F2( 注册码 )` 则注册成功的非明码比较，过程不是很难，所以高手可以略过。

校验时作者没有进行字符串比较，而是将 `F1( 注册码 )` 的运算结果插入到某个字符串的随机位置，比较时，返回在其中搜索出 `F2( 机器码 )` 结果的位置，能搜索到则注册成功。

【注册过程】

    ----------------------------------------------------------

    dephi 的程序，DEDE 给出了相当详细的注释，大家一看就知道了：

    本文注释中 szMac 是机器码，szReg 是输入的假码

    我的机器码：szMac = "61326210431"

    输入的假码：szReg = "987a-654b-987c-654d"

    ----------------------------------------------------------

```
006070A0  |> \8D55 E0         lea edx,dword ptr ss:[ebp-20]

006070A3  |.   8B83 1C040000  mov eax,dword ptr ds:[ebx+41C]    ;   edtReg1:N.A.( 读取 szReg1 )

006070A9  |.   E8 DA2DE5FF    call SSQDYJ.00459E88               ;   Controls.TControl.GetText(TControl):TCaption;

006070AE  |.   FF75 E0        push dword ptr ss:[ebp-20]

006070B1  |.   8D55 DC        lea edx,dword ptr ss:[ebp-24]

006070B4  |.   8B83 20040000  mov eax,dword ptr ds:[ebx+420]    ;   edtReg2:N.A.( 读取 szReg2 )

006070BA  |.   E8 C92DE5FF    call SSQDYJ.00459E88               ;   Controls.TControl.GetText(TControl):TCaption;

006070BF  |.   FF75 DC        push dword ptr ss:[ebp-24]

006070C2  |.   8D55 D8        lea edx,dword ptr ss:[ebp-28]

006070C5  |.   8B83 24040000  mov eax,dword ptr ds:[ebx+424]    ;   edtReg3:N.A.( 读取 szReg3 )

006070CB  |.   E8 B82DE5FF    call SSQDYJ.00459E88               ;   Controls.TControl.GetText(TControl):TCaption;

006070D0  |.   FF75 D8        push dword ptr ss:[ebp-28]

006070D3  |.   8D55 D4        lea edx,dword ptr ss:[ebp-2C]

006070D6  |.   8B83 28040000  mov eax,dword ptr ds:[ebx+428]    ;   edtReg4:N.A.( 读取 szReg4 )

006070DC  |.   E8 A72DE5FF    call SSQDYJ.00459E88               ;   Controls.TControl.GetText(TControl):TCaption;

006070E1  |.   FF75 D4        push dword ptr ss:[ebp-2C]

006070E4  |.   8D45 F8        lea eax,dword ptr ss:[ebp-8]

006070E7  |.   BA 04000000    mov edx,4

006070EC  |.   E8 BFDEDFFF    call SSQDYJ.00404FB0               ;   System.@LStrCatN;( 连接四段得到 szReg )

006070F1  |.   8D55 D0        lea edx,dword ptr ss:[ebp-30]

006070F4  |.   8B45 F8        mov eax,dword ptr ss:[ebp-8]      ;   ( szReg 小写转大写 )

006070F7  |.   E8 8426E0FF    call SSQDYJ.00409780               ;   SysUtils.UpperCase(AnsiString):AnsiString;

006070FC  |.   8B55 D0        mov edx,dword ptr ss:[ebp-30]

006070FF  |.   8D45 F8        lea eax,dword ptr ss:[ebp-8]

00607102  |.   E8 C9DBDFFF    call SSQDYJ.00404CD0               ;   System.@LStrLAsg(void;void;void;void);

00607107  |.   8B55 F8        mov edx,dword ptr ss:[ebp-8]

0060710A  |.   8BC3           mov eax,ebx

0060710C  |.   E8 F3FDFFFF    call SSQDYJ.00606F04               ;   TfrmRegWizard.\_PROC\_00606F04() - 需要跟进

00607111  |.   84C0           test al,al

00607113  |.   74 23          je short SSQDYJ.00607138           ;   程序中验证的地方很多，如果爆破需要找出其他验证处一起爆破

……………………

0060714F  |.   8BD0           mov edx,eax

00607151  |.   B9 DC716000    mov ecx,SSQDYJ.006071DC

00607156  |.   A1 A4506900    mov eax,dword ptr ds:[6950A4]

0060715B  |.   8B00           mov eax,dword ptr ds:[eax]        ;   注册码错误的对话框

0060715D  |.   E8 C647E7FF    call SSQDYJ.0047B928               ;   Forms.TApplication.MessageBox(TApplication;PChar;PChar;Longint):Integer;
```

跟进 
------------------------------------------------------------------------------------------

    0060710C  |.   E8 F3FDFFFF    call SSQDYJ.00606F04               ;   TfrmRegWizard.\_PROC\_00606F04() - 需要跟进

来到 
------------------------------------------------------------------------------------------

```
00606F1D  |.   55             push ebp

00606F1E  |.   68 9E6F6000    push SSQDYJ.00606F9E

00606F23  |.   64:FF30        push dword ptr fs:[eax]

00606F26  |.   64:8920        mov dword ptr fs:[eax],esp

00606F29  |.   8D55 F4        lea edx,dword ptr ss:[ebp-C]

00606F2C  |.   8B45 FC        mov eax,dword ptr ss:[ebp-4]

00606F2F  |.   E8 9C2AE0FF    call SSQDYJ.004099D0               ;   去掉 szReg 前后的空格

00606F34  |.   8B55 F4        mov edx,dword ptr ss:[ebp-C]

00606F37  |.   8D45 FC        lea eax,dword ptr ss:[ebp-4]

00606F3A  |.   E8 91DDDFFF    call SSQDYJ.00404CD0               ;   ...

00606F3F  |.   8D55 F0        lea edx,dword ptr ss:[ebp-10]

00606F42  |.   8B83 18040000  mov eax,dword ptr ds:[ebx+418]

00606F48  |.   E8 3B2FE5FF    call SSQDYJ.00459E88               ;   读取机器码 szMac

00606F4D  |.   8B45 F0        mov eax,dword ptr ss:[ebp-10]

00606F50  |.   8D55 F8        lea edx,dword ptr ss:[ebp-8]

00606F53  |.   E8 782AE0FF    call SSQDYJ.004099D0               ;   去掉 szMac 前后的空格

00606F58  |.   8B4D FC        mov ecx,dword ptr ss:[ebp-4]

00606F5B  |.   8B55 F8        mov edx,dword ptr ss:[ebp-8]

00606F5E  |.   A1 60A05F00    mov eax,dword ptr ds:[5FA060]

00606F63  |.   E8 C434FFFF    call SSQDYJ.005FA42C               ;   需要跟进的关键调用

00606F68  |.   8BD8           mov ebx,eax

00606F6A  |.   84DB           test bl,bl

00606F6C  |.   74 0D          je short SSQDYJ.00606F7B

00606F6E  |.   8B55 FC        mov edx,dword ptr ss:[ebp-4]

00606F71  |.   A1 60A05F00    mov eax,dword ptr ds:[5FA060]

00606F76  |.   E8 CD3BFFFF    call SSQDYJ.005FAB48

00606F7B  |>   33C0           xor eax,eax                           ;   return false ( 注册失败 )

……………………

00606F98  |.   E8 BFDCDFFF    call SSQDYJ.00404C5C

00606F9D  \\.   C3             retn

00606F9E   .^ E9 35D5DFFF     jmp SSQDYJ.004044D8

00606FA3   .^ EB E3          jmp short SSQDYJ.00606F88

00606FA5   .  8BC3           mov eax,ebx                           ;   return ture ( 注册成功 )

00606FA7   .  5B             pop ebx

00606FA8   .  8BE5           mov esp,ebp

00606FAA   .  5D             pop ebp

00606FAB   .  C3             retn
```

跟进 
-----------------------------------------------------------------------------------

    00606F63  |.   E8 C434FFFF    call SSQDYJ.005FA42C               ;   需要跟进的关键调用

来到 
-----------------------------------------------------------------------------------

```
005FA45D   .  55             push ebp

005FA45E   .  68 C4A65F00     push SSQDYJ.005FA6C4

005FA463   .  64:FF30         push dword ptr fs:[eax]

005FA466   .  64:8920         mov dword ptr fs:[eax],esp

005FA469   .  C645 F7 00      mov byte ptr ss:[ebp-9],0

005FA46D   .  8B45 F8         mov eax,dword ptr ss:[ebp-8]

005FA470   .  E8 7BAAE0FF     call SSQDYJ.00404EF0               ;   strlen( szReg )

005FA475   .  8BD8            mov ebx,eax

005FA477   .  83FB 01         cmp ebx,1

005FA47A   .  7C 21          jl short SSQDYJ.005FA49D           ;   strlen( szReg ) > 0

005FA47C   >  8B45 F8         mov eax,dword ptr ss:[ebp-8]

005FA47F   .  8A4418 FF      mov al,byte ptr ds:[eax+ebx-1]    ;   szReg 从后向前取一个字符 szReg[15-i]

005FA483   .  04 D0          add al,0D0

005FA485   .  2C 0A          sub al,0A                         ;   szReg[15-i] - 0x40 ( 这里判断 szReg[15-i] 是否是字母 )

005FA487   .  72 0F          jb short SSQDYJ.005FA498

005FA489   .  8D45 F8        lea eax,dword ptr ss:[ebp-8]      ;   是字符到这里

005FA48C   .  B9 01000000    mov ecx,1

005FA491   .  8BD3            mov edx,ebx

005FA493   .  E8 F0ACE0FF     call SSQDYJ.00405188               ;   szReg[15] 是字符则将其从字符串中去除

005FA498   >  4B              dec ebx                           ;   不是字符到这里

005FA499   .  85DB            test ebx,ebx
```



### 汇编代码片段

```
005FA49B   .^ 75 DF         jnz short SSQDYJ.005FA47C         ;  循环 16 (字符串长度) 轮  

005FA49D   >  8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  

005FA4A0   .  E8 4BAAE0FF   call SSQDYJ.00404EF0              ;  strlen( szMac )  

005FA4A5   .  83F8 05       cmp eax,5  

005FA4A8   .  0F8C D9010000 jl SSQDYJ.005FA687                ;  strlen ( szMac ) 不小于 5  

005FA4AE   .  8B45 F8       mov eax,dword ptr ss:\[ebp-8\]  

005FA4B1   .  E8 3AAAE0FF   call SSQDYJ.00404EF0              ;  strlen( szReg )  

005FA4B6   .  83F8 08       cmp eax,8  

005FA4B9   .  0F8C C8010000 jl SSQDYJ.005FA687                ;  strlen ( szReg ) 不小于 8  

005FA4BF   .  E8 F487E0FF   call SSQDYJ.00402CB8              ;  GetSystemTime  

005FA4C4   .  33C0          xor eax,eax  

005FA4C6   .  55           push ebp  

005FA4C7   .  68 79A65F00  push SSQDYJ.005FA679  

005FA4CC   .  64:FF30      push dword ptr fs:\[eax\]  

005FA4CF   .  64:8920      mov dword ptr fs:\[eax\],esp  

005FA4D2   .  B8 0A000000  mov eax,0A  

005FA4D7   .  E8 788EE0FF  call SSQDYJ.00403354  

005FA4DC   .  85C0         test eax,eax  

005FA4DE   .  7D 29        jge short SSQDYJ.005FA509  

005FA4E0   .  8D45 E4      lea eax,dword ptr ss:\[ebp-1C\]  

005FA4E3   .  8B55 F8      mov edx,dword ptr ss:\[ebp-8\]  

005FA4E6   .  E8 EDAFE0FF  call SSQDYJ.004054D8  

005FA4EB   .  8B55 E4      mov edx,dword ptr ss:\[ebp-1C\]  

005FA4EE   .  B9 01000000  mov ecx,1  

005FA4F3   .  8BC6         mov eax,esi  

005FA4F5   .  E8 5EFEFFFF  call SSQDYJ.005FA358  

005FA4FA   .  66:83F8 01   cmp ax,1  

005FA4FE   .  1BC0         sbb eax,eax  

005FA500   .  40           inc eax  

005FA501   .  8845 F7      mov byte ptr ss:\[ebp-9\],al  

005FA504   .  E9 66010000  jmp SSQDYJ.005FA66F  

005FA509   >  8D45 F0      lea eax,dword ptr ss:\[ebp-10\]  

005FA50C   .  50           push eax  

005FA50D   .  8B45 F8      mov eax,dword ptr ss:\[ebp-8\]  

005FA510   .  E8 DBA9E0FF  call SSQDYJ.00404EF0              ;  strlen( szReg ) ( 此时 szReg 中的字母已经被去除 )  

005FA515   .  8BC8         mov ecx,eax  

005FA517   .  49           dec ecx  

005FA518   .  BA 01000000  mov edx,1  

005FA51D   .  8B45 F8      mov eax,dword ptr ss:\[ebp-8\]  

005FA520   .  E8 23ACE0FF  call SSQDYJ.00405148              ;  取 szReg 在最后一位字符"前面"的字符  

005FA525   .  8B45 F0      mov eax,dword ptr ss:\[ebp-10\]  

005FA528   .  E8 9FFAE0FF  call SSQDYJ.00409FCC              ;  转换为 64 位整数 __int64  

005FA52D   .  52           push edx                            ; /Arg2  

005FA52E   .  50           push eax                            ; |Arg1  

005FA52F   .  8D55 E0      lea edx,dword ptr ss:\[ebp-20\]     ; |  

005FA532   .  B8 0C000000  mov eax,0C                          ; |转换结果的长度是 12 位  

005FA537   .  E8 DCF9E0FF  call SSQDYJ.00409F18                ; \\将 __int64 转换成 16 进制字符串  

    -------------------------------------------------------------------------------------  

    以上代码是去掉注册码 szReg 中的字母，并且去掉最后一个数值，留下来的数字继续向下验证  

    我输入的假码是 987a654b987c654d，经过处理后，szReg = "98765498765"  

    98765498765 转为 16 进制文本得到 "00A6E1236B9AF7" 后面会用到，也是 F2(注册码)的部分代码  

    -------------------------------------------------------------------------------------  

005FA53C   .  8B55 E0      mov edx,dword ptr ss:\[ebp-20\]  

005FA53F   .  8D45 F0      lea eax,dword ptr ss:\[ebp-10\]  

005FA542   .  E8 89A7E0FF  call SSQDYJ.00404CD0                ;  ...  

005FA547   .  8D4D E8      lea ecx,dword ptr ss:\[ebp-18\]  

005FA54A   .  33D2         xor edx,edx  

005FA54C   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

005FA54F   .  E8 2CE4FFFF  call SSQDYJ.005F8980                ;  机器码通过异或运算运算得到文本 szKey = "6962>:514;4",跟进 1  

005FA554   .  8D4D EC      lea ecx,dword ptr ss:\[ebp-14\]  

005FA557   .  8B55 E8      mov edx,dword ptr ss:\[ebp-18\]  

005FA55A   .  8B45 F0      mov eax,dword ptr ss:\[ebp-10\]  

005FA55D   .  E8 16E2FFFF  call SSQDYJ.005F8778                ;  需要跟进的关键调用，得到字符串 str1，跟进 2  

005FA562   .  8D45 D8      lea eax,dword ptr ss:\[ebp-28\]  

005FA565   .  50           push eax  

005FA566   .  B9 06000000  mov ecx,6  

005FA56B   .  BA 01000000  mov edx,1  

005FA570   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

005FA573   .  E8 D0ABE0FF  call SSQDYJ.00405148                ;  szMac 的 1 到 6 位 "613262"  

005FA578   .  8B45 D8      mov eax,dword ptr ss:\[ebp-28\]  

005FA57B   .  E8 D8F9E0FF  call SSQDYJ.00409F58                ;  十进制文本转换为数值 1  

005FA580   .  8BD8         mov ebx,eax  

005FA582   .  8D45 D4      lea eax,dword ptr ss:\[ebp-2C\]  

005FA585   .  50           push eax  

005FA586   .  B9 06000000  mov ecx,6  

005FA58B   .  BA 07000000  mov edx,7  

005FA590   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

005FA593   .  E8 B0ABE0FF  call SSQDYJ.00405148                ;  机器码的 7 到 12 位 "10431"  

005FA598   .  8B45 D4      mov eax,dword ptr ss:\[ebp-2C\]  

005FA59B   .  E8 B8F9E0FF  call SSQDYJ.00409F58                ;  十进制文本转换为数值 2  

005FA5A0   .  03D8         add ebx,eax                         ;  数值 1 + 数值 2 即 613262 + 10431 = 623693  

005FA5A2   .  8BC3         mov eax,ebx  

005FA5A4   .  8D55 DC      lea edx,dword ptr ss:\[ebp-24\]  

005FA5A7   .  E8 CCF8E0FF  call SSQDYJ.00409E78                ;  相加结果转换回十进制文本 str2 = "623693"  

005FA5AC   .  8B55 DC      mov edx,dword ptr ss:\[ebp-24\]  

005FA5AF   .  8D45 FC      lea eax,dword ptr ss:\[ebp-4\]  

005FA5B2   .  E8 19A7E0FF  call SSQDYJ.00404CD0                ;  ...  

005FA5B7   .  8D45 FC      lea eax,dword ptr ss:\[ebp-4\]  

005FA5BA   .  50           push eax  

005FA5BB   .  B9 06000000  mov ecx,6  

005FA5C0   .  BA 01000000  mov edx,1  

005FA5C5   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

005FA5C8   .  E8 7BABE0FF  call SSQDYJ.00405148                ;  取 str2 的 1 到 6 位 "623693"  

005FA5CD   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

005FA5D0   .  E8 1BA9E0FF  call SSQDYJ.00404EF0                ;  strlen( str2 ) 这里应该等于 6  

005FA5D5   .  8BD8         mov ebx,eax  

005FA5D7   .  43           inc ebx  

005FA5D8   .  83FB 06      cmp ebx,6  

005FA5DB   .  7F 16        jg short SSQDYJ.005FA5F3  

005FA5DD   >  8D45 FC      lea eax,dword ptr ss:\[ebp-4\]  

005FA5E0   .  8B4D FC      mov ecx,dword ptr ss:\[ebp-4\]  

005FA5E3   .  BA E0A65F00  mov edx,SSQDYJ.005FA6E0  

005FA5E8   .  E8 4FA9E0FF  call SSQDYJ.00404F3C                ;  str2 没有 6 位则在前面添加 0  

005FA5ED   .  43           inc ebx  

005FA5EE   .  83FB 07      cmp ebx,7  

005FA5F1   .  ^ 75 EA      jnz short SSQDYJ.005FA5DD  

005FA5F3     8B55 EC      mov edx,dword ptr ss:\[ebp-14\]     ;  str1( 由于插入了随机位置，所以每次跟踪会得到不同的结果，详见后面 )  

005FA5F6   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]      ;  str2 = "623693"  

005FA5F9   .  E8 2EACE0FF  call SSQDYJ.0040522C                ;  返回 str1 中 str2 的位置（失败返回 0）  

005FA5FE   .  8BD8         mov ebx,eax  

005FA600   .  85DB         test ebx,ebx  

005FA602   .  7E 6B        jle short SSQDYJ.005FA66F          ;  关键跳  

005FA604   .  8B45 FC      mov eax,dword ptr ss:\[ebp-4\]  

    ---------------------------------------------------------------------------------------------------------------------------  

    上面代码有两行需要跟进：  

    005FA54F   .  E8 2CE4FFFF   call SSQDYJ.005F8980              ;  机器码通过异或运算运算得到文本 szKey = "6962>:514;4",跟进 1  

    005FA55D   .  E8 16E2FFFF   call SSQDYJ.005F8778              ;  需要跟进的关键调用，得到字符串 str1，跟进 2  

    等下说说这两个跟进，看看是如何得到 str1 的。  

    继续向下看，看到了 F1( 机器码 ) 的处理：  

    大致过程这样：我的机器码 "61326210431"  

    机器码的前 6 位 + 接下来的 6 位 即 613236 + 10431 = 623693，然后再取前 6 位得到 str2 = '623693'  

    ---------------------------------------------------------------------------------------------------------------------------  
```

### 跟进分析

下面看下两个跟进，看看如何得到 `str1`：

```
    ---------------------------------------------------------------------------------------------------------------------------
```



005FA54F . E8 2CE4FFFF call `SSQDYJ.005F8980` ; 机器码通过异或运算运算得到文本 `szKey` = "6962>:514;4",跟进 1

---------------------------------------------------------------------------------------------------------------------------

```
005F89CC . 8D45 F8 lea eax,dword ptr ss:[ebp-8]
005F89CF . BA 9C8A5F00 mov edx,SSQDYJ.005F8A9C ; 取一个字符串 `szStr` = "yxb_200@"
005F89D4 . E8 F7C2E0FF call SSQDYJ.00404CD0 ; ...
005F89D9 |> 837D FC 00 cmp dword ptr ss:[ebp-4],0
005F89DD . 75 0A jnz short SSQDYJ.005F89E9
005F89DF . 8B45 F4 mov eax,dword ptr ss:[ebp-C]
005F89E2 . E8 51C2E0FF call SSQDYJ.00404C38 ; ...
005F89E7 . EB 6C jmp short SSQDYJ.005F8A55
005F89E9 |> 8D45 F0 lea eax,dword ptr ss:[ebp-10]
005F89EC . E8 47C2E0FF call SSQDYJ.00404C38 ; ...
005F89F1 . 8B45 F8 mov eax,dword ptr ss:[ebp-8]
005F89F4 . E8 F7C4E0FF call SSQDYJ.00404EF0 ; `strlen(`yxb_200@`)` = 8
005F89F9 . 8945 EC mov dword ptr ss:[ebp-14],eax
005F89FC . 8B45 FC mov eax,dword ptr ss:[ebp-4]
005F89FF . E8 ECC4E0FF call SSQDYJ.00404EF0 ; `strlen(`szMac`)` = 11
005F8A04 . 8BF0 mov esi,eax
005F8A06 . 85F6 test esi,esi
005F8A08 . 7E 40 jle short SSQDYJ.005F8A4A
005F8A0A . BB 01000000 mov ebx,1 ; `i` = 1 ( 以下循环 `i` 从 1 开始 )
005F8A0F |> /mov eax,ebx
005F8A11 |. 99 |cdq
005F8A12 |. F77D EC |idiv dword ptr ss:[ebp-14] ; `i` % 8
005F8A15 |. 8BFA |mov edi,edx
005F8A17 |. 47 |inc edi ; `i` % 8 + 1
005F8A18 |. 8B45 F8 |mov eax,dword ptr ss:[ebp-8]
005F8A1B |. 0FB64438 FF |movzx eax,byte ptr ds:[eax+edi-1>; `szStr[`i` % 8]`
005F8A20 |. B9 0A000000 |mov ecx,0A
005F8A25 |. 33D2 |xor edx,edx
005F8A27 |. F7F1 |div ecx ; `szStr[`i` % 8]` % 10
005F8A29 |. 8B45 FC |mov eax,dword ptr ss:[ebp-4]
005F8A2C |. 0FB64418 FF |movzx eax,byte ptr ds:[eax+ebx-1>; `szMac[`i`-1]`
005F8A31 |. 33D0 |xor edx,eax ; `szMac[`i`-1]` ^ ( `szStr[`i` % 8]` % 10 )
005F8A33 |. 8D45 E4 |lea eax,dword ptr ss:[ebp-1C]
005F8A36 |. E8 DDC3E0FF |call SSQDYJ.00404E18 ; 转换成字符类型
005F8A3B |. 8B55 E4 |mov edx,dword ptr ss:[ebp-1C]
005F8A3E |. 8D45 F0 |lea eax,dword ptr ss:[ebp-10]
005F8A41 |. E8 B2C4E0FF |call SSQDYJ.00404EF8 ; 每轮计算结果连接
005F8A46 |. 43 |inc ebx
005F8A47 |. 4E |dec esi
005F8A48 |.^ 75 C5 \jnz short SSQDYJ.005F8A0F ; 处理完机器码所有的字符退出循环
005F8A4A |> 8B45 F4 mov eax,dword ptr ss:[ebp-C]
005F8A4D . 8B55 F0 mov edx,dword ptr ss:[ebp-10]
005F8A50 . E8 37C2E0FF call SSQDYJ.00404C8C
005F8A55 |> 33C0 xor eax,eax
005F8A57 . 5A pop edx
005F8A58 . 59 pop ecx
```

-----------------------------------------------------------------------------------------------------------

005FA55D . E8 16E2FFFF call `SSQDYJ.005F8778` ; 需要跟进的关键调用，得到字符串 `str1`，跟进 2

-----------------------------------------------------------------------------------------------------------

```
005F87AD . 55 push ebp
005F87AE . 68 2B895F00 push SSQDYJ.005F892B
005F87B3 . 64:FF30 push dword ptr fs:[eax]
005F87B6 . 64:8920 mov dword ptr fs:[eax],esp
005F87B9 . 8B45 F8 mov eax,dword ptr ss:[ebp-8]
005F87BC . E8 2FC7E0FF call SSQDYJ.00404EF0 ; `strlen(`szKey`)` = 11
005F87C1 . 8945 F0 mov dword ptr ss:[ebp-10],eax
005F87C4 . 837D F0 00 cmp dword ptr ss:[ebp-10],0
005F87C8 . 75 0D jnz short SSQDYJ.005F87D7
005F87CA . 8D45 F8 lea eax,dword ptr ss:[ebp-8]
005F87CD . BA 44895F00 mov edx,SSQDYJ.005F8944 ; ASCII "yxb_200@"
005F87D2 . E8 F9C4E0FF call SSQDYJ.00404CD0 ; ...
005F87D7 > 33F6 xor esi,esi
005F87D9 . 33C0 xor eax,eax
005F87DB . 55 push ebp
005F87DC . 68 97885F00 push SSQDYJ.005F8897
005F87E1 . 64:FF30 push dword ptr fs:[eax]
005F87E4 . 64:8920 mov dword ptr fs:[eax],esp
005F87E7 . 8B45 F8 mov eax,dword ptr ss:[ebp-8]
005F87EA . E8 25FFFFFF call SSQDYJ.005F8714 ; `szKey` = "6962>:514;4"各字符求和 得到 600
005F87EF . B9 FF000000 mov ecx,0FF
005F87F4 . 99 cdq
005F87F5 . F7F9 idiv ecx ; 600 % 0xFF = 90
005F87F7 . 8BFA mov edi,edx
005F87F9 . 8D45 EC lea eax,dword ptr ss:[ebp-14]
005F87FC . E8 37C4E0FF call SSQDYJ.00404C38 ; ...
005F8801 . C745 E8 01000>mov dword ptr ss:[ebp-18],1 ; 下面循环 `i` 初值是 1 ( 不是 0 )
005F8808 > 8D45 DC lea eax,dword ptr ss:[ebp-24]
005F880B . 50 push eax
005F880C . B9 02000000 mov ecx,2
005F8811 . 8B55 E8 mov edx,dword ptr ss:[ebp-18]
005F8814 . 8B45 FC mov eax,dword ptr ss:[ebp-4]
005F8817 . E8 2CC9E0FF call SSQDYJ.00405148 ; 循环取 "00A6E1236B9AF7" 中的两个字符
005F881C . 8B4D DC mov ecx,dword ptr ss:[ebp-24]
005F881F . 8D45 E0 lea eax,dword ptr ss:[ebp-20]
005F8822 . BA 58895F00 mov edx,SSQDYJ.005F8958
005F8827 . E8 10C7E0FF call SSQDYJ.00404F3C ; 字符串前 + '$'
005F882C . 8B45 E0 mov eax,dword ptr ss:[ebp-20]
005F882F . E8 2417E1FF call SSQDYJ.00409F58 ; 文本转换为数值
005F8834 . 8945 E4 mov dword ptr ss:[ebp-1C],eax
005F8837 . 3B75 F0 cmp esi,dword ptr ss:[ebp-10]
005F883A . 7D 03 jge short SSQDYJ.005F883F
005F883C . 46 inc esi ; 这里让 ESI 从 1 到 11 ( `szKey` 的长度) 循环
005F883D . EB 05 jmp short SSQDYJ.005F8844
005F883F > BE 01000000 mov esi,1
005F8844 > 8B45 F8 mov eax,dword ptr ss:[ebp-8]
005F8847 . 33DB xor ebx,ebx
005F8849 . 8A5C30 FF mov bl,byte ptr ds:[eax+esi-1]
005F884D . 335D E4 xor ebx,dword ptr ss:[ebp-1C] ; 结果 ^ `szKey[`ESI` - 1]`
005F8850 . 3BFB cmp edi,ebx
005F8852 . 7C 0A jl short SSQDYJ.005F885E
005F8854 . 81C3 FF000000 add ebx,0FF ; 异或结果 - 上轮结果 ( 不够减就先加上 255 再减 )
005F885A . 2BDF sub ebx,edi
005F885C . EB 02 jmp short SSQDYJ.005F8860
005F885E > 2BDF sub ebx,edi ; 异或结果 - 上轮结果 ( 够减就直接减 )
005F8860 > 8D45 D8 lea eax,dword ptr ss:[ebp-28]
005F8863 . 8BD3 mov edx,ebx
005F8865 . E8 AEC5E0FF call SSQDYJ.00404E18
005F886A . 8B55 D8 mov edx,dword ptr ss:[ebp-28]
005F886D . 8D45 EC lea eax,dword ptr ss:[ebp-14]
005F8870 . E8 83C6E0FF call SSQDYJ.00404EF8 ; 连接每轮计算结果
005F8875 . 8B7D E4 mov edi,dword ptr ss:[ebp-1C]
005F8878 . 8345 E8 02 add dword ptr ss:[ebp-18],2 ; `i` = `i` + 2
005F887C . 8B45 FC mov eax,dword ptr ss:[ebp-4]
005F887F . E8 6CC6E0FF call SSQDYJ.00404EF0 ; 读取字符串的 `strlen( )`
005F8884 . 3B45 E8 cmp eax,dword ptr ss:[ebp-18]
005F8887 .^. 0F8F 7BFFFFFF jg SSQDYJ.005F8808 ; 是否处理完所有的字符串
005F888D . 33C0 xor eax,eax
005F888F . 5A pop edx
005F8890 . 59 pop ecx
005F8891 . 59 pop ecx
005F8892 . 64:8910 mov dword ptr fs:[eax],edx
```



```
005F8895   .  EB 12         jmp short SSQDYJ.005F88A9  

005F8897   .^ E9 88B9E0FF   jmp SSQDYJ.00404224  

005F889C   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]  

005F889F   .  E8 94C3E0FF   call SSQDYJ.00404C38              ;  ...  

005F88A4   .  E8 A7BDE0FF   call SSQDYJ.00404650  

005F88A9   >  8B4D F4       mov ecx,dword ptr ss:[ebp-C]  

005F88AC   .  BA 64895F00   mov edx,SSQDYJ.005F8964           ;  ASCII "hazizi"  

005F88B1   .  B8 74895F00   mov eax,SSQDYJ.005F8974           ;  ASCII "zizi#826_zc"  

005F88B6   .  E8 45FCFFFF   call SSQDYJ.005F8500              ;  得到字符串 szStr = "0D430B99C05B5EB54383725D3255CA56"  

005F88BB   .  8D55 D4       lea edx,dword ptr ss:[ebp-2C]     ;  上面的这个字符串跟机器无关，无须跟进  

005F88BE   .  8B45 EC       mov eax,dword ptr ss:[ebp-14]  

005F88C1   .  E8 0A11E1FF   call SSQDYJ.004099D0              ;  去掉字符串前后的空格  

005F88C6   .  837D D4 00    cmp dword ptr ss:[ebp-2C],0  

005F88CA   .  75 0D         jnz short SSQDYJ.005F88D9  

005F88CC   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]  

005F88CF   .  BA 64895F00   mov edx,SSQDYJ.005F8964           ;  ASCII "hazizi"  

005F88D4   .  E8 F7C3E0FF   call SSQDYJ.00404CD0              ;  ...  

005F88D9   >  B8 06000000   mov eax,6  

005F88DE   .  E8 71AAE0FF   call SSQDYJ.00403354              ;  生成一个随机数  

005F88E3   .  8BF0          mov esi,eax  

005F88E5   .  85F6         test esi,esi  

005F88E7   .  7D 05        jge short SSQDYJ.005F88EE  

005F88E9   .  BE 01000000  mov esi,1  

005F88EE   >  8B55 F4       mov edx,dword ptr ss:[ebp-C]  

005F88F1   .  8BCE          mov ecx,esi  

005F88F3   .  8B45 EC       mov eax,dword ptr ss:[ebp-14]  

005F88F6   .  E8 D5C8E0FF   call SSQDYJ.004051D0              ;  将上面循环结果插入到 szStr 的随机位置  

005F88FB   .  33C0          xor eax,eax  

005F88FD   .  5A            pop edx  
```

先把前面得到的 `"00A6E1236B9AF7"` 分成 6 个 byte 得到 `byte bHex[6] = {0x00,0xA6,0xE1,0x23,0x6B,0x9A,0xF7}`

然后循环处理：`bHex[i] ^ szKey[i] - bHex[i-1]`

说明：不够减则加上 `0xFF` 再减，另外 `i = 0` 时 ，`bHex[i-1]` 另外单独算，大概是 `sum( szKey ) % 255`

最后我们看到程序解密出一个字符串 `"0D430B99C05B5EB54383725D3255CA56"`，然后将循环结果插入随机位置，为后面校验准备

实际，我们并不关心 `"0D430B99C05B5EB54383725D3255CA56"` 这个字符串，只要让 `str1 = str2` 即可通过注册

【破解小结】

上面过程写得太乱了，都不知道如何总结了，瞎写吧：

采用 `F1( 机器码 ) = F2( 注册码 )` 的方式验证：

`F1( 机器码 )` 的过程是这样的：

机器码的前 6 位 + 接下来 6 位 613236 + 10431 = 623693，然后再取前 6 位得到 `"623693"`

`F2( 注册码 )` 的过程是这样的：

先去除字母和最后一个数字得到 `"98765498765"` 转成 16 进制的 `__int64` 得到 `"00A6E1236B9AF7"`

然后分成 6 个 byte 得到 `byte bHex[6] = {0x00,0xA6,0xE1,0x23,0x6B,0x9A,0xF7}`

同时将机器码经过循环异或得到 `szKey = "6962>:514;4"`

然后循环处理：`bHex[i] ^ szKey[i] - bHex[i-1]`

说明：不够减则加上 `0xFF` 再减，另外 `i = 0` 时 ，`bHex[i-1]` 另外单独算，大概是 `sum( szKey ) % 255`

计算结果是：

```
    00C3D85C  DB 2F B2 D4 F5 CD                  ?苍跬..&
```

【注册机源码】

```c
void CKeygenDlg::OnOK() 

{ 

  // TODO: Add extra validation here 

  UpdateData(true); 

  char szKey[64],szReg[32]={0},szStr[32]={"yxb_200@"}; 

  //我的机器码 

  //m_Edit1 = "61326210431"; 

  //检查机器码的长度不小于 5 

  int iLength = m_Edit1.GetLength (); 

  if ( iLength < 5 ){ 

    m_Edit2 = "机器码应该至少 5 个数字"; 

    UpdateData(false); 

    return; 

  } 

  //机器码的前 6 位 + 接下来的 6 位 

  //即 613262 + 10431 = 623693 

  int iMac= atoi(m_Edit1.Mid (0,6)) + (iLength>6?atoi(m_Edit1.Mid (6,6)):0); 

  //机器码中的字符跟（szStr 中的字符对 10 的余数）循环异或 

  //得到 szKey = "6962>:514:4" 

  strcpy(szKey,m_Edit1); 

  for ( byte i=0; i<iLength; i++ ) szKey[i] ^= (szStr[(i+1)%8]%10); 

  //取机器码相加结果 "623693" 的前 6 位 

  sprintf(szStr,"%06d",iMac);szStr[6]=0; 

  //szKey 的各个字符 ASC 求和并对 255 取余得到 90，作为 iSum 的初值 

  int iSum = 0; 

  for ( i=0; i<iLength; i++ ) iSum += szKey[i]; 

  iSum %= 255; 

  //经过下面循环 iReg 转为十进制文本就是注册码了 

  __int64 iReg = 0; 

  iLength = strlen(szStr); 

  for( i=0; i<iLength; i++ ){ 

    iSum += szStr[i]; 

    if( iSum > 255 ) iSum -= 255; 

    iSum ^= szKey[i]; 

    iReg *= 256; 

    iReg += iSum; 

  } 

  //最后对注册码处理 

  _i64toa(iReg,szReg,10); 

  m_Edit2 = szReg; 

  //如果注册码没有 16 位，可以在注册码的随机位置插入随机字母 

  //这些字母在校验时会被去除 

  iLength = 15 - m_Edit2.GetLength (); 

  for ( i=0; i<iLength; i++ ){ 

    char cRand = 'A' + rand() % 26 ; 

    byte bIndex = rand() % iLength; 

    m_Edit2.Insert (bIndex,cRand); 

  } 

  //注册码的最后一位插入一个随机数字 

  m_Edit2.Insert (15,('0'+rand()%10)); 

  //用 '-' 将注册码分成四段 

  m_Edit2.Insert (4,'-'); 

  m_Edit2.Insert (9,'-'); 

  m_Edit2.Insert (14,'-'); 

  UpdateData(false); 

}
```

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 07 月 06 日 19:31:22

`4dd466d0e1e0cc123afbe504da2462ca`
