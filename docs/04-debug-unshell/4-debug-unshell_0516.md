
# 菜鸟学习脱壳之 ASProtect 2.1x SKE 简单加壳的 98NOTEPAD(MC++)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-568.htm`](https://www.pediy.com/kssd/pediy08/pediy8-568.htm)

【破解作者】 windycandy

【使用工具】 OD, PEID0.94, LordPE.ImportREC1.6F

【破解平台】 Win9x/NT/2000/XP

【软件名称】 菜鸟学习脱壳之 ASProtect 2.1x SKE 简单加壳的 98NOTEPAD(MC++)

【软件下载】 `http://bbs.chinapyg.com/viewthread.php?tid=4436&extra=page%3D1`

【软件简介】 ASProtect SKE 2.1x build 03.13 在看雪工具下载区下载的
对 98notepad 加壳时我只用了比较简单的 resources protection 的两个选项,
没有 stolen code

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

----------------------------------------

## 1. 声明

本文是学习 lchhome 的<ASProtect 2.1x SKE 之木马克星 5.50 build 2403 脱壳分析>后照搬方法过来完成的

## 2. 脱壳过程

先用 `shift+f9` 到最后一次异常+code 段内存断点法到达 OEP

```
004010CC     55                     push ebp
004010CD     8BEC                   mov ebp,esp
004010CF     83EC 44                sub esp,44
004010D2     56                     push esi
004010D3     E8 28EF8F00            call 00D00000------------------------------注意这里,00D00000 重要数据①
004010D8     308B F08A003C          xor byte ptr ds:[ebx+3C008AF0],cl
004010DE     2275 1B                and dh,byte ptr ss:[ebp+1B]
004010E1     56                     push esi
004010E2     FF15 F4644000          call dword ptr ds:[4064F4]               ; USER32.CharNextA-------随便找个 IAT,就这个吧,得 IAT 所在段 406400
004010E8     8BF0                   mov esi,eax
004010EA     8A00                   mov al,byte ptr ds:[eax]
004010EC     84C0                   test al,al
004010EE     74 04                  je short NOTEPAD.004010F4
004010F0     3C 22                  cmp al,22
004010F2     75 ED                  jnz short NOTEPAD.004010E1
004010F4     803E 22                cmp byte ptr ds:[esi],22
004010F7     75 15                  jnz short NOTEPAD.0040110E
004010F9     46                     inc esi
004010FA     EB 12                  jmp short NOTEPAD.0040110E
004010FC     3C 20                  cmp al,20
004010FE     7E 0E                  jle short NOTEPAD.0040110E
00401100     56                     push esi
00401101     FF15 F4644000          call dword ptr ds:[4064F4]               ; USER32.CharNextA
```

在命令行下 `DD 406400`,向上找到 IAT 的起始 `4062E4`(重要数据②)

OK, `CTRL+F2` 重新载入,命令行下 `DD 4062E4`,看数据框数值

```
004062E4   4B4D681E-----------------开始时的数值
004062E8   CC64D787
004062EC   DD66FC7B
004062F0   3069C122
004062F4   094AC5BE
004062F8   3A54002D
004062FC   7844F6D2
00406300   004FD9D9  OLE32.004FD9D9
```

现在开始按 `shift+f9` 运行,大概 13 次左右,注意数据框的数值第一次发生变化,变成这样

```
004062E4   EDAD82FA--------------第一次变化
004062E8   644A3619
004062EC   E48F601C
004062F0   5FCCB218
004062F4   339150CC
004062F8   EFEEE474
004062FC   969653D6
00406300   AB6EEF48
```

在继续 `shift+f9` 运行,大概 3 次左右,数据框的数值第二次发生变化

```
004062E4   8A14F25D--------------第二次变化
004062E8   F4312E14
004062EC   E2DDA7FA
004062F0   50544050
004062F4   CDDCE005
004062F8   00000000
004062FC   1930CEFE
```

发生第二次变化后,再按一次或两次 `shift+f9`,就会中断在

```
00ACEA55     C601 B6                  mov byte ptr ds:[ecx],0B6----------中断在这里
00ACEA58     6323                     arpl word ptr ds:[ebx],sp
00ACEA5A     DEAA B86B9C6B            fisubr word ptr ds:[edx+6B9C6BB8]
00ACEA60     75 11                    jnz short 00ACEA73
00ACEA62     E5 8F                    in eax,8F
00ACEA64     FE                       ???                                 ; 未知命令
00ACEA65     67:64:8F06 0000          pop dword ptr fs:[0]
00ACEA6B     EB 01                    jmp short 00ACEA6E
00ACEA6D     C783 C40483E9 5FEB0169  mov dword ptr ds:[ebx+E98304C4],6901E>
00ACEA77     C1D9 A1                  rcr ecx,0A1
00ACEA7A     59                       pop ecx
00ACEA7B     A1 1C37AD00              mov eax,dword ptr ds:[AD371C]
```

在汇编窗口单击右键------查找-------所有参考文本串,找到以下文本串

```
00ACE243   push 0ACE260                    ASCII "151
00ACE33F   push 0ACE5FC                    ASCII "61
00ACE66E   push 0ACE683                    ASCII "W 鑿"
00ACEA55   mov byte ptr ds:[ecx],0B6       （初始 CPU 选择）
00ACEBF8   push 0ACF4E8                    ASCII "85-----------找到这里"85,特征码,双击,来到反汇编窗口
00ACEC12   push 0ACF4E8                    ASCII "85
00ACF1C5   mov ecx,47E7A2                  ASCII "h?"
```

来到这里

```
00ACEBF8     68 E8F4AC00        push 0ACF4E8                       ; ASCII "85
00ACEBFD     E8 2A62FEFF        call 00AB4E2C
00ACEC02     A1 1C37AD00        mov eax,dword ptr ds:[AD371C]
00ACEC07     8B00               mov eax,dword ptr ds:[eax]
00ACEC09     E8 0A8CFFFF        call 00AC7818--------------------------在这里下断
00ACEC0E     84C0               test al,al
00ACEC10     75 0A              jnz short 00ACEC1C
00ACEC12     68 E8F4AC00        push 0ACF4E8                       ; ASCII "85
00ACEC17     E8 1062FEFF        call 00AB4E2C
```

在 `00ACEC09` 处 `F2` 下断点,`shift+f9` 中断在 `00ACEC09`,取消断点,`F7` 进入

```
00AC7818     53                     push ebx
00AC7819     56                     push esi
00AC781A     57                     push edi
00AC781B     55                     push ebp
00AC781C     83C4 F4                add esp,-0C
00AC781F     8BF0                   mov esi,eax
00AC7821     C60424 01              mov byte ptr ss:[esp],1
00AC7825     EB 01                  jmp short 00AC7828
```

向下找到这里

```
00AC7954     8D4424 10              lea eax,dword ptr ss:[esp+10]
00AC7958     50                     push eax
00AC7959     56                     push esi
00AC795A     E8 59FCFFFF            call 00AC75B8---------------这个 CALL 是进行 IAT 处理，
00AC795F     0FB707                 movzx eax,word ptr ds:[edi]
00AC7962     83C0 02                add eax,2
00AC7965     03F8                   add edi,eax
00AC7967     8A1F                   mov bl,byte ptr ds:[edi]
00AC7969     47                     inc edi
00AC796A     3A5E 34                cmp bl,byte ptr ds:[esi+34]
00AC796D     0F85 77FFFFFF          jnz 00AC78EA-----------------每个 DLL 的函数是否处理
00AC7973     8BDF                   mov ebx,edi
00AC7975     8B03                   mov eax,dword ptr ds:[ebx]
00AC7977     85C0                   test eax,eax
00AC7979     0F85 0AFFFFFF          jnz 00AC7889-----------------比较 DLL 是否处理完
00AC797F     8A0424                 mov al,byte ptr ss:[esp]
00AC7982     83C4 0C                add esp,0C
00AC7985     5D                     pop ebp
00AC7986     5F                     pop edi
00AC7987     5E                     pop esi
00AC7988     5B                     pop ebx
00AC7989     C3                     retn
```

先在 `00AC7989` 上 `F2` 下断,再直接 `F4` 到 `00AC795A`,`F7` 进入 Call,并向下找到

```
00AC75D3     83E8 02                sub eax,2
00AC75D6     0FB630                 movzx esi,byte ptr ds:[eax]
00AC75D9     8B45 10                mov eax,dword ptr ss:[ebp+10]
00AC75DC     83E8 02                sub eax,2
00AC75DF     0FB600                 movzx eax,byte ptr ds:[eax]
00AC75E2     3B43 2C                cmp eax,dword ptr ds:[ebx+2C]
00AC75E5     76 06                  jbe short 00AC75ED
00AC75E7     8943 2C                mov dword ptr ds:[ebx+2C],eax
00AC75EA     EB 01                  jmp short 00AC75ED
00AC75EC     6933 C08A433B          imul esi,dword ptr ds:[ebx],3B438AC0
00AC75F2     3BF0                   cmp esi,eax----------------在这里下 F2 断点,F9 运行,观察 esi 的值,并注意数据框 IAT
00AC75F4     75 5E                  jnz short 00AC7654---------这里也下断,看不同数值,跳转是否是成功
00AC75F6     EB 01                  jmp short 00AC75F9
```



# 机器码逆向与IAT修复记录

每人的机子不一样，数值也不一样。我这里的 `esi` 有三个值 `96`、`77`、`89`（这几个数值好像每天都不一样）。其中 `esi` 的值为 `77`、`89` 时 `00AC75F4` 处的跳转成立，但两个数中只有当 `esi=77` 时才会对 IAT 进行加密，因此只要将 `77` 改为 `89` 就可以避开 IAT 加密了。

## 修改 00AC75F4 处代码

好，就在 `00AC75F4` 进行修改。先用 OD 插件 `memory manage` 申请一个内存空间，`lchhome` 用的是 `01640000`，为了方便我也用 `01640000`。

把 `00AC75F4` 改为：

```
00AC75F4 75F4  - E9 078A3500      JMP 01640000
```

然后下 `F2` 断点，`F9` 运行中断后，`F7` 跟进。

把修改代码写进去：

```
01640000   - 0F84 F37548FF          je 00AC75F9 ------ 这个是 00AC75F6 处 jmp 的地址
01640006      81FE 89000000          cmp esi,89
0164000C   - 0F84 427648FF          je 00AC7654
01640012      BE 89000000           mov esi,89
01640017   - E9 387648FF           jmp 00AC7654
```

写好代码后，取消 `00AC75F2` 及 `00AC75F4` 两处的断点，`F9` 运行中断在 `00AC7989`，中断后取消断点。

看看数据窗口，是不是得到所有的 IAT 了？起始 `4062E4`，终点 `406520`。

```
004062E4  77DA2410  ADVAPI32.RegQueryValueExA
004062E8  77DA17D8  ADVAPI32.RegCloseKey
004062EC  77DB63B1  ADVAPI32.RegSetValueExA
004062F0  77DA23D9  ADVAPI32.RegOpenKeyA
004062F4  77DA28BB  ADVAPI32.RegCreateKeyA
004062F8  00000000
004062FC  77C4513D  GDI32.GetObjectA
00406300  77C44B71  GDI32.GetDeviceCaps
00406304  77C4889D  GDI32.CreateFontIndirectA
00406308  77C41B98  ASCII "U 嬱 Q 僥?
0040630C  77C6DCB4  GDI32.AbortDoc
00406310  77C5472B  GDI32.EndDoc
00406314  77C43607  GDI32.DeleteDC
00406318  77C5483B  GDI32.StartPage
0040631C  77C54D16  GDI32.StartDocA
00406320  77C5460B  GDI32.EndPage
00406324  77C48333  GDI32.GetTextExtentPointA
00406328  77C53BB4  GDI32.CreateFontA
0040632C  77C543A2  GDI32.SetAbortProc
00406330  77C41EEB  GDI32.SetBkMode
00406334  77C45011  GDI32.SetMapMode
00406338  77C483C6  ASCII "U 嬱冹 H 僥?
0040633C  77C4A9F8  ASCII "U 嬱 Q 僥?
00406340  77C4AAF9  GDI32.SetViewportExtEx
00406344  77C48D75  GDI32.LPtoDP
00406348  77C47080  GDI32.CreateDCA
```

将 `1640000` 处的修改代码及 `00AC75F4` `75F4` `-` `E9` `078A3500` `JMP` `01640000` 的修改全部取消。然后 `shift+F9` 到最后一次异常并用 `CODE` 断点法到达 `OEP` 处。

## 修改 00AC7190 处代码

到达 `OEP` 后，搜索一下 `89,45,F0,B8,00,07,00,00` 到达这里（也可以直接 `CTRL+G` `00AC7190`，前面 `00AC` 每个机子可能是不同的，但后面的 `7190` 是一样的）：

```
00AC7188      8945 F0             mov dword ptr ss:\[ebp-10\],eax
00AC718B      B8 00070000         mov eax,700
00AC7190      E8 B7B3FDFF         call 00AA254C---------------这里要修改
00AC7195      8945 E4             mov dword ptr ss:\[ebp-1C\],eax
00AC7198      C645 EF 00          mov byte ptr ss:\[ebp-11\],0
00AC719C      8B7D F8             mov edi,dword ptr ss:\[ebp-8\]
```

就用刚才申请的内存空间 `01640000`，`00AC7190` 句修改为：

```
00AC7190  - E9 A28E3500     JMP 01640037
```

然后写下如下代码（抄 `lchhome` 的）：

```
01640000      BA 00104000          mov edx,401000----写好代码后在这里新建 EIP
01640005      803A E8              cmp byte ptr ds:\[edx\],0E8
01640008      75 12               jnz short 0164001C
0164000A      8B42 01              mov eax,dword ptr ds:\[edx+1\]
0164000D      03C2                 add eax,edx
0164000F      83C0 05              add eax,5
01640012      3D 0000D000          cmp eax,0D00000----------------重要数据①
01640017      75 03               jnz short 0164001C
01640019      EB 0C               jmp short 01640027
0164001B      90                   nop
0164001C      42                   inc edx
0164001D      81FA D04F4000        cmp edx,404FD0 ---------------确保所有的 call 0D00000 被修改过来
01640023    ^ 72 E0               jb short 01640005
01640025    - EB FE               jmp short 01640025
01640027      8915 00016401        mov dword ptr ds:\[1640100\],edx
0164002D      60                   pushad
0164002E      FFE2                 jmp edx
01640030      90                   nop
01640031      90                   nop
01640032      90                   nop
01640033      90                   nop
01640034      90                   nop
01640035      90                   nop
01640036      90                   nop
01640037      60                   pushad
01640038      B8 E4624000          mov eax,4062E4----------------------重要数据②
0164003D      90                   nop
0164003E      3910                 cmp dword ptr ds:\[eax\],edx
01640040      75 20               jnz short 01640062
01640042      8B0D 00016401        mov ecx,dword ptr ds:\[1640100\]
01640048      C701 FF150000        mov dword ptr ds:\[ecx\],15FF--------------是 delphi 语言程序的用 25FF,其他语言的用 15FF
0164004E      8941 02              mov dword ptr ds:\[ecx+2\],eax
01640051      61                   popad
01640052      90                   nop
01640053      8B15 00016401        mov edx,dword ptr ds:\[1640100\]
01640059      90                   nop
0164005A      90                   nop
0164005B      90                   nop
0164005C    ^ EB BE               jmp short 0164001C
0164005E      90                   nop
0164005F      90                   nop
01640060      90                   nop
01640061      90                   nop
01640062      83C0 04              add eax,4
01640065      3D 60654000          cmp eax,406520 ---------------------IAT 的终点
0164006A    ^ 7E D2               jle short 0164003E
0164006C    ^ EB E3               jmp short 01640051

BA 00 10 40 00 80 3A E8 75 12 8B 42 01 03 C2 83 C0 05 3D 00 00 D0 00 75 03 EB 0C 90 42 81 FA 00
20 40 00 72 E0 EB FE 89 15 00 01 64 01 60 FF E2 90 90 90 90 90 90 90 60 B8 E4 62 40 00 90 39 10
75 20 8B 0D 00 01 64 01 C7 01 FF 25 00 00 89 41 02 61 90 8B 15 00 01 64 01 90 90 90 EB BE 90 90
90 90 83 C0 04 3D 60 65 40 00 7E D2 EB E3
```

然后 `F9` 运行一会，用 `LordPe` 纠正大小后 `dump`，用 `ImportREC 1.6F` 修复，填上 `OEP=10CC`，点 `IAT` 自动搜索，获得输入信息，全部为有效指针。

修复抓取文件。运行脱壳文件，能正常运行。收工。

## 后记与补充

> 最初由 `wangcai` 发布

楼主我发现个问题 你脱壳的 `NOTEPAD.EXE` 打开后点->文件->打开 后出错.............

看到了，可以 `OD` 载入脱壳后的 `dumped_`，`F9` 运行，打开后点->文件->打开，停在以下代码处，

```
00404FAA    $- FF25 1C654000        jmp dword ptr ds:\[<&comdlg32.CommDlgE>;  comdlg32.CommDlgExtendedError
00404FB0       FF15 18654000        call dword ptr ds:\[<&comdlg32.GetSave>;  comdlg32.GetSaveFileNameA
00404FB6       FF15 14654000        call dword ptr ds:\[<&comdlg32.PageSet>;  comdlg32.PageSetupDlgA
00404FBC    $- FF25 10654000        jmp dword ptr ds:\[<&comdlg32.FindText>;  comdlg32.FindTextA
00404FC2       FF15 0C654000        call dword ptr ds:\[<&comdlg32.ChooseF>;  comdlg32.ChooseFontA
00404FC8       FF15 08654000        call dword ptr ds:\[<&comdlg32.GetOpen>;  comdlg32.GetOpenFileNameA
00404FCE       FF15 20654000        call dword ptr ds:\[<&comdlg32.GetFile>;  comdlg32.GetFileTitleA
00404FD4    .  0000                 add byte ptr ds:\[eax\],al----------停在这里
```

对比没有加壳的 `NOTEPAD` 发现从 `00404AA` 到 `00404FCE` 处应该全部是 `jmp`，但是脱壳后的 `dumped_` 有的 `jmp` 已经变成 `call`，将 `call` 修改为 `jmp`，程序正常运行。应该是到 `OEP` 后 patch 中:

```
0164001D      81FA D04F4000         cmp edx,404FD0 -------- 404FD0 这个数值太大的原因.应该是多少,还在寻找
```

配套脱壳动画在 `bbs.chinapyg.com` 视频区

\--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
