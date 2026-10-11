
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
