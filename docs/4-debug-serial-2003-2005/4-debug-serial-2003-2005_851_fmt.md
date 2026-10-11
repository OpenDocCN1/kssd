
# 柿子挑软的捡——多功能时钟日历 V2.5 完全算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-140.htm`](https://www.pediy.com/kssd/pediy07/pediy7-140.htm)

## 基本信息

日期：2005 年 3 月 22 日
破解人：`Baby2008`

## 软件信息

- **软件名称**：多功能时钟日历 V2.5
- **软件大小**：4.16M
- **下载地址**：`http://www.magic2008.com/`
- **软件限制**：`Nag` 试用提示限制，随着试用次数的增加 `Nag` 提示越长，试用 1 次提示 100ms。
- **保护方式**：
    1. 注册码保护，注册码不正确，注册按钮为灰色；
    2. 重启验证。

## 破解声明

初学 Crack，只是感兴趣，加上和女朋友吵架了，无聊中看见坛子里有两篇文章提及这个软件的部分破解，顺便下载来学习一下，失误之处敬请诸位大侠赐教！

## 破解工具

- `DeDe3.50.04` Fix 超强版
- `Ollydbg` V1.10 2005.3 汉化修正版+最新最全插件
- `Quick Unpack` 0.6 脱壳工具
- `PeID` 0.93

## 破解过程

先用 `PeID` 0.93 汉化增强版查壳，`ASPack` 2.12 -> `Alexey Solodovnikov`，用 `Quick Unpack` 0.6 轻松搞定默认另存为 `unpacked.ExE`。脱壳后再用 `PeID` 0.93 汉化增强版查壳，`Borland Delphi` 4.0 - 5.0，程序没有自校验，脱壳后能直接运行^\_^ (是工具好使还是作者善良？)

按照习惯，请出 Delphi 的“朋友”`DeDe3.50`，发现程序有反 `DeDe` 功能，载入文件就提示退出 `DeDe`，幸好现在有 `DeDe3.50.04` Fix 超强版！！

载入 `unpacked.ExE`，分析 `Procedures` 得知 `TFormtest2` 为注册界面。按照我个人编程习惯，激活“灰色注册”按钮是通过 `edit` 控件的 `onkeypress` 事件实现，但在这个注册界面上没有此事件，而是放了个 `Timer` 检测注册码的正确性。

### 注册算法第一部分分析

记下 `Timer1Timer` 事件的地址：`005615A0`，`Button1Click` 事件的地址：`0056167C` 备用。初略分析后用 `OD` 载入 `unpacked.ExE`，`F9` 运行，程序在自动忽略几次异常后出现 `Nag` 界面，点击输入注册码，出现注册界面。用户名：`Baby2008`，试炼码：`1234567890`。根据前面初步分析的结果，注册码由 `Timer1Timer` 检验，因此切换到 `OD` 窗口，下断点 `bp 005615A0`，`OD` 中断在：

```
005615A0  <>/.   55                   push ebp                           ;   <-TFormTest2@Timer1Timer
005615A1   |.   8BEC                 mov ebp,esp
005615A3   |.   33C9                 xor ecx,ecx
005615A5   |.   51                   push ecx
005615A6   |.   51                   push ecx
005615A7   |.   51                   push ecx
005615A8   |.   51                   push ecx
005615A9   |.   53                   push ebx
005615AA   |.   8BD8                 mov ebx,eax
005615AC   |.   33C0                 xor eax,eax
005615AE   |.   55                   push ebp
005615AF   |.   68 55165600          push <unpacked.->system.@HandleFinal>
005615B4   |.   64:FF30              push dword ptr fs: [eax]
005615B7   |.   64:8920              mov dword ptr fs: [eax],esp
005615BA   |.   8D45 FC             lea eax, [local.1]
005615BD   |.   50                   push eax
005615BE   |.   8D55 F8             lea edx, [local.2]
005615C1 <>|.   8B83 EC020000        mov eax,dword ptr ds: [ebx+2EC]       ;   *Edit2:N.A.
005615C7 <>|.   E8 605AEDFF          call unpacked.0043702C               ;   ->controls.TControl.GetText(TControl):TCaption;
005615CC   |.   8B45 F8             mov eax, [local.2]                     ;   保存试炼码到 EAX
005615CF   |.   B9 06000000          mov ecx,6
005615D4   |.   BA 01000000          mov edx,1
005615D9 <>|.   E8 662DEAFF          call unpacked.00404344               ;   ->system.@LStrCopy;
005615DE   |.   8B45 FC             mov eax, [local.1]                     ;   取试炼码的 1-6 位
005615E1   |.   50                   push eax
005615E2   |.   8D55 F4             lea edx, [local.3]
005615E5   |.   B8 6C165600          mov eax,unpacked.0056166C            ;   固定字符串 "A4AFA8A4AFA8"
005615EA   |.   E8 ED5CF5FF          call unpacked.004B72DC               ;   转换函数，比较简单，详细分析略过
005615EF   |.   8B55 F4             mov edx, [local.3]                     ;   将上面固定字符串转为字符串'BINBIN'
005615F2   |.   58                   pop eax
005615F3 <>|.   E8 542CEAFF          call unpacked.0040424C               ;   ->system.@LStrCmp;试炼码 1-6 位与'BINBIN'比较
005615F8       75 23                jnz short unpacked.0056161D           ;   不等 Over,相等激活注册按钮
005615FA   |.   8D55 F0             lea edx, [local.4]
005615FD <>|.   8B83 E4020000        mov eax,dword ptr ds: [ebx+2E4]       ;   *Edit1:N.A.
00561603 <>|.   E8 245AEDFF          call unpacked.0043702C               ;   ->controls.TControl.GetText(TControl):TCaption;
00561608   |.   837D F0 00           cmp [local.4],0
0056160C   |.   74 0F                je short unpacked.0056161D
0056160E   |.   B2 01                mov dl,1
00561610 <>|.   8B83 F0020000        mov eax,dword ptr ds: [ebx+2F0]       ;   *Button1:N.A.
00561616   |.   8B08                 mov ecx,dword ptr ds: [eax]
00561618   |.   FF51 5C              call dword ptr ds: [ecx+5C]
0056161B   |.   EB 0D                jmp short unpacked.0056162A
0056161D   |>  33D2                 xor edx,edx
0056161F <>|.   8B83 F0020000        mov eax,dword ptr ds: [ebx+2F0]       ;   *Button1:N.A.
00561625   |.   8B08                 mov ecx,dword ptr ds: [eax]
00561627   |.   FF51 5C              call dword ptr ds: [ecx+5C]
0056162A   |>  33C0                 xor eax,eax
0056162C   |.   5A                   pop edx
0056162D   |.   59                   pop ecx
0056162E   |.   59                   pop ecx
0056162F   |.   64:8910              mov dword ptr fs: [eax],edx
00561632   |.   68 5C165600          push unpacked.0056165C
00561637   |>  8D45 F0              lea eax, [local.4]
0056163A <>|.   E8 6D28EAFF          call unpacked.00403EAC               ;   ->system.@LStrClr(String;String);
0056163F   |.   8D45 F4              lea eax, [local.3]
00561642 <>|.   E8 6528EAFF          call unpacked.00403EAC               ;   ->system.@LStrClr(String;String);
00561647   |.   8D45 F8              lea eax, [local.2]
0056164A   |.   BA 02000000          mov edx,2
0056164F <>|.   E8 7C28EAFF          call unpacked.00403ED0               ;   ->system.@LStrArrayClr;
00561654   \\.   C3                   retn
```

由此可得：注册码的 1－6 位必须为“`BINBIN`”，在 `about` 窗口中作者特别感谢的一个人就叫“斌斌”，不知是什么人物，连注册码都要以他名字开头，废话少说，继续……

### 注册算法第二部分分析

取消断点 `005615A0`（如果不取消，程序重新载入后立即中断在 `005615A0` 处，100ms 的时间够你输入注册用户名及试炼码吗？），`F2` 重新载入程序，`F9` 运行。在注册界面输入用户名：`Baby2008`，试炼码：`BINBIN1234567890`（注意大写），注册按钮立即激活，由此可知第一步分析的是正确的。

切换到 `OD` 窗口下断点 `bp 0056167C`，`OD` 中断在：

```
0056167C <>  .   55                   push ebp                           ;   <-TFormTest2@Button1Click
0056167D    .   8BEC                 mov ebp,esp                          ;   注册按钮
0056167F    .   B9 04000000          mov ecx,4
00561684    >   6A 00                push 0
00561686    .   6A 00                push 0
00561688    .   49                   dec ecx
00561689    .^  75 F9                jnz short unpacked.00561684
0056168B    .   51                   push ecx
0056168C    .   53                   push ebx
0056168D    .   8BD8                 mov ebx,eax
0056168F    .   33C0                 xor eax,eax
00561691    .   55                   push ebp
00561692    .   68 ED175600          push <unpacked.->system.@HandleFinal>
00561697    .   64:FF30              push dword ptr fs: [eax]
0056169A    .   64:8920              mov dword ptr fs: [eax],esp
0056169D    .   8D55 F4              lea edx,dword ptr ss: [ebp-C]
005616A0 <> .   8B83 E4020000        mov eax,dword ptr ds: [ebx+2E4]       ;   *Edit1:N.A.
005616A6 <> .   E8 8159EDFF          call unpacked.0043702C               ;   ->controls.TControl.GetText(TControl):TCaption;
005616AB    .   8B45 F4              mov eax,dword ptr ss: [ebp-C]         ;   用户名
005616AE    .   8D55 F8              lea edx,dword ptr ss: [ebp-8]
005616B1 <> .   E8 0286EAFF          call unpacked.00409CB8               ;   ->sysutils.Trim(AnsiString):AnsiString;
005616B6    .   837D F8 00           cmp dword ptr ss: [ebp-8],0           ;   去前后空格后不能为空
005616BA    .   0F84 DF000000        je unpacked.0056179F                  ;   为空则 Over!
005616C0    .   8D55 EC              lea edx,dword ptr ss: [ebp-14]
005616C3 <> .   8B83 EC020000        mov eax,dword ptr ds: [ebx+2EC]       ;   *Edit2:N.A.
005616C9 <> .   E8 5E59EDFF          call unpacked.0043702C               ;   ->controls.TControl.GetText(TControl):TCaption;
005616CE    .   8B45 EC              mov eax,dword ptr ss: [ebp-14]        ;   注册码
```



```
005616D1    .   8D55 F0               lea edx,dword ptr ss:[ebp-10]  
005616D4 <> .   E8 DF85EAFF          call unpacked.00409CB8             ;  ->sysutils.Trim(AnsiString):AnsiString;  
005616D9    .   837D F0 00          cmp dword ptr ss:[ebp-10],0       ;  注册码不能为空  
005616DD    .   0F84 BC000000       je unpacked.0056179F  
005616E3    .   B2 01               mov dl,1  
005616E5    .   A1 A4ED4A00         mov eax,dword ptr ds:[4AEDA4]  
005616EA <> .   E8 21D8F4FF          call unpacked.004AEF10             ;   
->registry.TRegistry.Create(TRegistry;boolean);overload;  
005616EF    .   8945 FC             mov dword ptr ss:[ebp-4],eax  
005616F2    .   BA 02000080         mov edx,80000002  
005616F7    .   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
005616FA <> .   E8 EDD8F4FF          call unpacked.004AEFEC             ;  ->registry.TRegistry.SetRootKey(TRegistry;HKEY);  
005616FF    .   33C0                xor eax,eax  
00561701    .   55                  push ebp  
00561702    .   68 8C175600         push <unpacked.->system.@HandleFinal>  
00561707    .   64:FF30             push dword ptr fs:[eax]  
0056170A    .   64:8920             mov dword ptr fs:[eax],esp  
0056170D    .   B1 01               mov cl,1  
0056170F    .   BA 04185600         mov edx,unpacked.00561804          ;  ASCII "SOFTWARE\\MGC\\RL"  
00561714    .   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
00561717 <> .   E8 14DAF4FF          call unpacked.004AF130             ;   
->registry.TRegistry.OpenKey(TRegistry;AnsiString;Boolean):Boolean;  
0056171C    .   8D55 E4             lea edx,dword ptr ss:[ebp-1C]  
0056171F <> .   8B83 E4020000        mov eax,dword ptr ds:[ebx+2E4]     ;  \*Edit1:N.A.  
00561725 <> .   E8 0259EDFF          call unpacked.0043702C             ;  ->controls.TControl.GetText(TControl):TCaption;  
0056172A    .   8B45 E4             mov eax,dword ptr ss:[ebp-1C]  
0056172D    .   8D55 E8             lea edx,dword ptr ss:[ebp-18]  
00561730 <> .   E8 8385EAFF          call unpacked.00409CB8             ;  ->sysutils.Trim(AnsiString):AnsiString;  
00561735    .   8B4D E8             mov ecx,dword ptr ss:[ebp-18]  
00561738    .   BA 1C185600         mov edx,unpacked.0056181C          ;  ASCII "NAME"  
0056173D    .   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
00561740 <> .   E8 A7DBF4FF          call unpacked.004AF2EC             ;   
->registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);  
00561745    .   8D55 DC             lea edx,dword ptr ss:[ebp-24]  
00561748 <> .   8B83 EC020000        mov eax,dword ptr ds:[ebx+2EC]     ;  \*Edit2:N.A.  
0056174E <> .   E8 D958EDFF          call unpacked.0043702C             ;  ->controls.TControl.GetText(TControl):TCaption;  
00561753    .   8B45 DC             mov eax,dword ptr ss:[ebp-24]  
00561756    .   8D55 E0             lea edx,dword ptr ss:[ebp-20]  
00561759 <> .   E8 5A85EAFF          call unpacked.00409CB8             ;  ->sysutils.Trim(AnsiString):AnsiString;  
0056175E    .   8B4D E0             mov ecx,dword ptr ss:[ebp-20]  
00561761    .   BA 2C185600         mov edx,unpacked.0056182C          ;  ASCII "CODE"  
00561766    .   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
00561769 <> .   E8 7EDBF4FF          call unpacked.004AF2EC             ;   
->registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);  
0056176E    .   33C0                xor eax,eax  
00561770    .   5A                  pop edx  
00561771    .   59                  pop ecx  
00561772    .   59                  pop ecx  
00561773    .   64:8910             mov dword ptr fs:[eax],edx  
00561776    .   68 93175600         push unpacked.00561793  
0056177B    >   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
0056177E <> .   E8 39D8F4FF          call unpacked.004AEFBC             ;  ->registry.TRegistry.CloseKey(TRegistry);  
00561783    .   8B45 FC             mov eax,dword ptr ss:[ebp-4]  
00561786 <> .   E8 3D1AEAFF          call unpacked.004031C8             ;  ->system.TObject.Free(TObject);  
0056178B    .   C3                  retn  
0056178C <> .^  E9 9721EAFF          jmp unpacked.00403928              ;  ->system.@HandleFinally;  
00561791    .^ EB E8                jmp short unpacked.0056177B  
00561793    .   A1 70245900         mov eax,dword ptr ds:[592470]  
00561798    .   8B00                mov eax,dword ptr ds:[eax]  
0056179A <> .   E8 3D0BEFFF          call unpacked.004522DC             ;  ->forms.TCustomForm.Close(TCustomForm);  
0056179F    >   33C0                xor eax,eax  
005617A1    .   5A                  pop edx  
005617A2    .   59                  pop ecx  
005617A3    .   59                  pop ecx  
005617A4    .   64:8910             mov dword ptr fs:[eax],edx  
005617A7    .   68 F4175600         push unpacked.005617F4  
005617AC    >   8D45 DC             lea eax,dword ptr ss:[ebp-24]  
005617AF <> .   E8 F826EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617B4    .   8D45 E0             lea eax,dword ptr ss:[ebp-20]  
005617B7 <> .   E8 F026EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617BC    .   8D45 E4             lea eax,dword ptr ss:[ebp-1C]  
005617BF <> .   E8 E826EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617C4    .   8D45 E8             lea eax,dword ptr ss:[ebp-18]  
005617C7 <> .   E8 E026EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617CC    .   8D45 EC             lea eax,dword ptr ss:[ebp-14]  
005617CF <> .   E8 D826EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617D4    .   8D45 F0             lea eax,dword ptr ss:[ebp-10]  
005617D7 <> .   E8 D026EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617DC    .   8D45 F4             lea eax,dword ptr ss:[ebp-C]  
005617DF <> .   E8 C826EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617E4    .   8D45 F8             lea eax,dword ptr ss:[ebp-8]  
005617E7 <> .   E8 C026EAFF          call unpacked.00403EAC             ;  ->system.@LStrClr(String;String);  
005617EC    .   C3                  retn  
005617ED <> .^  E9 3621EAFF          jmp unpacked.00403928              ;  ->system.@HandleFinally;  
005617F2    .^ EB B8                jmp short unpacked.005617AC  
005617F4    .   5B                  pop ebx  
005617F5    .   8BE5                mov esp,ebp  
005617F7    .   5D                  pop ebp  
005617F8    .   C3                  retn  
```

分析得知：注册按钮事件仅仅简单校验用户名的完整性，不为空即保存注册信息到注册表 `SOFTWARE\MGC\RL` 的分支中，是典型的重起验证。

使用 `Ultra String Reference` 插件查找 `SOFTWARE\MGC\RL`，仅两处在 `0056170F`、`00581715`，分别下断。`F2` 重新载入，`F9` 运行。`OD` 中断在 `00581715` 处：

```
00581708    .   68 AA175800         push <unpacked.->system.@HandleFinal>  
0058170D    .   64:FF30             push dword ptr fs:[eax]  
00581710    .   64:8920             mov dword ptr fs:[eax],esp  
00581713    .   B1 01               mov cl,1  
00581715    .   BA D0195800         mov edx,unpacked.005819D0          ;  ASCII "SOFTWARE\\MGC\\RL"  
0058171A    .   8B45 F4             mov eax,dword ptr ss:[ebp-C]  
0058171D <> .   E8 0EDAF2FF          call unpacked.004AF130             ;   
->registry.TRegistry.OpenKey(TRegistry;AnsiString;Boolean):Boolean;  
00581722    .   BA E8195800         mov edx,unpacked.005819E8          ;  ASCII "NAME"  
00581727    .   8B45 F4             mov eax,dword ptr ss:[ebp-C]  
0058172A <> .   E8 81DDF2FF          call unpacked.004AF4B0             ;   
```



`->registry.TRegistry.ValueExists(TRegistry;AnsiString):Boolean;`

```
0058172F    .  84C0                   test al,al
00581731    .  75 07                  jnz short unpacked.0058173A
00581733 <> .  E8 C822E8FF            call unpacked.00403A00               ; ->system.@TryFinallyExit;
00581738    .  EB 77                  jmp short unpacked.005817B1
0058173A    >  BA F8195800            mov edx,unpacked.005819F8           ; ASCII "CODE"
0058173F    .  8B45 F4               mov eax,dword ptr ss:[ebp-C]
00581742 <> .  E8 69DDF2FF            call unpacked.004AF4B0               ;
```

`->registry.TRegistry.ValueExists(TRegistry;AnsiString):Boolean;`

```
00581747    .  84C0                   test al,al
00581749    .  75 07                  jnz short unpacked.00581752
0058174B <> .  E8 B022E8FF            call unpacked.00403A00               ; ->system.@TryFinallyExit;
00581750    .  EB 5F                  jmp short unpacked.005817B1
00581752    >  8D4D C0               lea ecx,dword ptr ss:[ebp-40]
00581755    .  BA E8195800           mov edx,unpacked.005819E8           ; ASCII "NAME"
0058175A    .  8B45 F4               mov eax,dword ptr ss:[ebp-C]
0058175D <> .  E8 B6DBF2FF            call unpacked.004AF318               ;
```

`->registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;`

```
00581762    .  8B55 C0               mov edx,dword ptr ss:[ebp-40]       ; 用户名
00581765    .  A1 6C225900           mov eax,dword ptr ds:[59226C]
0058176A <> .  E8 9127E8FF            call unpacked.00403F00               ; ->system.@LStrAsg;
0058176F    .  8D4D BC               lea ecx,dword ptr ss:[ebp-44]
00581772    .  BA F8195800           mov edx,unpacked.005819F8           ; ASCII "CODE"
00581777    .  8B45 F4               mov eax,dword ptr ss:[ebp-C]
0058177A <> .  E8 99DBF2FF            call unpacked.004AF318               ;
```

`->registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;`

```
0058177F    .  8B55 BC               mov edx,dword ptr ss:[ebp-44]       ; 试炼码
00581782    .  A1 4C1E5900           mov eax,dword ptr ds:[591E4C]
00581787 <> .  E8 7427E8FF            call unpacked.00403F00               ; ->system.@LStrAsg;
0058178C    .  33C0                   xor eax,eax
0058178E    .  5A                     pop edx
0058178F    .  59                     pop ecx
00581790    .  59                     pop ecx
00581791    .  64:8910                mov dword ptr fs:[eax],edx
00581794    .  68 B1175800            push unpacked.005817B1
00581799    >  8B45 F4               mov eax,dword ptr ss:[ebp-C]
0058179C <> .  E8 1BD8F2FF            call unpacked.004AEFBC               ; ->registry.TRegistry.CloseKey(TRegistry);
005817A1    .  8B45 F4               mov eax,dword ptr ss:[ebp-C]
005817A4 <> .  E8 1F1AE8FF            call unpacked.004031C8               ; ->system.TObject.Free(TObject);
005817A9    .  C3                     retn
```

分析得知程序仅从注册表读取注册信息，没有验证。`F8`单步调试到`00581762`时，堆栈出现`Stack ss:[0012FDDC]=00B23604, (ASCII "baby2008")`，用户名你总可能要参加注册计算的吧?在`00B23604`处下访问断点，`F9`运行，`OD`断在`00402ACC`处：

```
00402ACC   |.  FC                     cld
```

堆栈友好提示---返回到`unpacked.004B75FA`：

```
0012F7AC    00B763DC
0012F7B0    00B23604     ASCII "baby2008"
0012F7B4    00403FB5     返回到 unpacked.00403FB5 来自 unpacked.00402A90
0012F7B8    0012F848
0012F7BC    00000000
0012F7C0    00000008
0012F7C4    0040436C     返回到 unpacked.0040436C 来自 unpacked.00403F94
0012F7C8    00564CAA     <unpacked.->system.@DoneExcept;>
0012F7CC    004B75FA     返回到 unpacked.004B75FA 来自 unpacked.00404344
0012F7D0    0012F818
0012F7D4    0012F854     指针到下一个 SEH 记录
0012F7D8    004B78F2     SE 句柄
```

`Ctrl+G 004B75FA`，`F2`下断，删除`00B23604`断点，`F9`运行，`OD`中断在`004B75FA`:

```
004B7548    $  55                     push ebp                             ; 注册码第二部分验证
004B7549    .  8BEC                   mov ebp,esp
004B754B    .  B9 0C000000            mov ecx,0C
004B7550    >  6A 00                  push 0
004B7552    .  6A 00                  push 0
004B7554    .  49                     dec ecx
004B7555    .^ 75 F9                  jnz short unpacked.004B7550
004B7557    .  53                     push ebx
004B7558    .  56                     push esi
004B7559    .  57                     push edi
004B755A    .  8955 F8               mov dword ptr ss:[ebp-8],edx         ; EDX:=试炼码
004B755D         8945 FC             mov dword ptr ss:[ebp-4],eax         ; EAX:=用户名
004B7560    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]         ; 用户名
004B7563    .  E8 88CDF4FF            call unpacked.004042F0
004B7568    .  8B45 F8               mov eax,dword ptr ss:[ebp-8]         ; 试炼码
004B756B    .  E8 80CDF4FF            call unpacked.004042F0
004B7570    .  33C0                   xor eax,eax
004B7572    .  55                     push ebp
004B7573    .  68 F2784B00            push unpacked.004B78F2
004B7578    .  64:FF30               push dword ptr fs:[eax]
004B757B    .  64:8920                mov dword ptr fs:[eax],esp
004B757E    .  C745 F4 D4070000       mov dword ptr ss:[ebp-C],7D4
004B7585    .  8D45 F0               lea eax,dword ptr ss:[ebp-10]
004B7588    .  BA 0C794B00            mov edx,unpacked.004B790C           ; ASCII "        "
004B758D    .  E8 B2C9F4FF            call unpacked.00403F44
004B7592    .  837D FC 00             cmp dword ptr ss:[ebp-4],0           ; 用户名为空则 Over!
004B7596    .  0F84 26030000          je unpacked.004B78C2
004B759C    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]
004B759F    .  E8 98CBF4FF            call unpacked.0040413C               ; 取用户名长度
004B75A4    .  83F8 08                cmp eax,8
004B75A7    .  7D 3B                  jge short unpacked.004B75E4          ; >=8 跳
004B75A9    .  8D45 D4               lea eax,dword ptr ss:[ebp-2C]
004B75AC    .  50                     push eax
004B75AD    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]
004B75B0    .  E8 87CBF4FF            call unpacked.0040413C
004B75B5    .  B9 08000000            mov ecx,8
004B75BA    .  2BC8                  sub ecx,eax
004B75BC    .  BA 01000000            mov edx,1
004B75C1    .  8B45 F0               mov eax,dword ptr ss:[ebp-10]
004B75C4    .  E8 7BCDF4FF            call unpacked.00404344
004B75C9    .  8B45 D4               mov eax,dword ptr ss:[ebp-2C]
004B75CC    .  50                     push eax
004B75CD    .  8D55 D0               lea edx,dword ptr ss:[ebp-30]
004B75D0    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]
004B75D3    .  E8 E026F5FF            call unpacked.00409CB8
004B75D8    .  8B55 D0               mov edx,dword ptr ss:[ebp-30]
004B75DB         8D45 FC             lea eax,dword ptr ss:[ebp-4]
004B75DE    .  59                     pop ecx
004B75DF    .  E8 A4CBF4FF            call unpacked.00404188               ; 用户名长度小于 8，则在用户名后面补空格到 8 位
004B75E4    >  8D45 CC               lea eax,dword ptr ss:[ebp-34]
004B75E7    .  50                     push eax
004B75E8    .  B9 01000000            mov ecx,1
004B75ED    .  BA 01000000            mov edx,1
004B75F2    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]         ; 用户名
004B75F5    .  E8 4ACDF4FF            call unpacked.00404344               ; 取用户名第 1 位
004B75FA    .  8B45 CC               mov eax,dword ptr ss:[ebp-34]
004B75FD    .  E8 76F9FFFF            call unpacked.004B6F78
004B7602    .  8BF0                  mov esi,eax
004B7604    .  81E6 FF000000          and esi,0FF                          ; ESI 保存用户名第 1 位
004B760A    .  8D45 C8               lea eax,dword ptr ss:[ebp-38]
004B760D    .  50                     push eax
004B760E    .  B9 01000000            mov ecx,1
004B7613    .  BA 02000000            mov edx,2
```



```asm
004B7618    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B761B    .  E8 24CDF4FF          call unpacked.00404344           ;  取用户名第 2 位  

004B7620    .  8B45 C8               mov eax,dword ptr ss:[ebp-38]  

004B7623    .  E8 50F9FFFF          call unpacked.004B6F78  

004B7628    .  8BF8                  mov edi,eax  

004B762A    .  81E7 FF000000        and edi,0FF                      ;  EDI 保存用户名第 2 位  

004B7630    .  8D45 C4               lea eax,dword ptr ss:[ebp-3C]  

004B7633    .  50                   push eax  

004B7634    .  B9 01000000          mov ecx,1  

004B7639    .  BA 03000000          mov edx,3  

004B763E    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B7641    .  E8 FECCF4FF          call unpacked.00404344           ;  取用户名第 3 位  

004B7646    .  8B45 C4               mov eax,dword ptr ss:[ebp-3C]  

004B7649    .  E8 2AF9FFFF          call unpacked.004B6F78  

004B764E    .  25 FF000000          and eax,0FF  

004B7653    .  8945 EC               mov dword ptr ss:[ebp-14],eax     ;  保存用户名第 3 位  

004B7656    .  8D45 C0               lea eax,dword ptr ss:[ebp-40]  

004B7659    .  50                   push eax  

004B765A    .  B9 01000000          mov ecx,1  

004B765F    .  BA 04000000          mov edx,4  

004B7664    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B7667    .  E8 D8CCF4FF          call unpacked.00404344           ;  取用户名第 4 位  

004B766C    .  8B45 C0               mov eax,dword ptr ss:[ebp-40]  

004B766F    .  E8 04F9FFFF          call unpacked.004B6F78  

004B7674    .  25 FF000000          and eax,0FF  

004B7679    .  8945 E8               mov dword ptr ss:[ebp-18],eax     ;  保存用户名第 4 位  

004B767C    .  8D45 BC               lea eax,dword ptr ss:[ebp-44]  

004B767F    .  50                   push eax  

004B7680    .  B9 01000000          mov ecx,1  

004B7685    .  BA 05000000          mov edx,5  

004B768A    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B768D    .  E8 B2CCF4FF          call unpacked.00404344  

004B7692    .  8B45 BC               mov eax,dword ptr ss:[ebp-44]  

004B7695    .  E8 DEF8FFFF          call unpacked.004B6F78  

004B769A    .  25 FF000000          and eax,0FF  

004B769F    .  8945 E4               mov dword ptr ss:[ebp-1C],eax     ;  保存用户名第 5 位  

004B76A2    .  8D45 B8               lea eax,dword ptr ss:[ebp-48]  

004B76A5    .  50                   push eax  

004B76A6    .  B9 01000000          mov ecx,1  

004B76AB    .  BA 06000000          mov edx,6  

004B76B0    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B76B3    .  E8 8CCCF4FF          call unpacked.00404344  

004B76B8    .  8B45 B8               mov eax,dword ptr ss:[ebp-48]  

004B76BB    .  E8 B8F8FFFF          call unpacked.004B6F78  

004B76C0    .  25 FF000000          and eax,0FF  

004B76C5    .  8945 E0               mov dword ptr ss:[ebp-20],eax     ;  保存用户名第 6 位  

004B76C8    .  8D45 B4               lea eax,dword ptr ss:[ebp-4C]  

004B76CB    .  50                   push eax  

004B76CC    .  B9 01000000          mov ecx,1  

004B76D1    .  BA 07000000          mov edx,7  

004B76D6    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B76D9    .  E8 66CCF4FF          call unpacked.00404344  

004B76DE    .  8B45 B4               mov eax,dword ptr ss:[ebp-4C]  

004B76E1    .  E8 92F8FFFF          call unpacked.004B6F78  

004B76E6    .  25 FF000000          and eax,0FF  

004B76EB    .  8945 DC               mov dword ptr ss:[ebp-24],eax     ;  保存用户名第 7 位  

004B76EE    .  8D45 B0               lea eax,dword ptr ss:[ebp-50]  

004B76F1    .  50                   push eax  

004B76F2    .  B9 01000000          mov ecx,1  

004B76F7    .  BA 08000000          mov edx,8  

004B76FC    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]  

004B76FF    .  E8 40CCF4FF          call unpacked.00404344  

004B7704    .  8B45 B0               mov eax,dword ptr ss:[ebp-50]  

004B7707    .  E8 6CF8FFFF          call unpacked.004B6F78  

004B770C    .  25 FF000000          and eax,0FF  

004B7711    .  8945 D8               mov dword ptr ss:[ebp-28],eax     ;  保存用户名第 8 位  

004B7714    .  33C0                 xor eax,eax  

004B7716    .  55                   push ebp  

004B7717    .  68 5C774B00          push unpacked.004B775C  

004B771C    .  64:FF30              push dword ptr fs:[eax]  

004B771F    .  64:8920              mov dword ptr fs:[eax],esp  

004B7722    .  8D45 A8               lea eax,dword ptr ss:[ebp-58]  

004B7725    .  50                   push eax  

004B7726    .  B9 08000000          mov ecx,8  

004B772B    .  BA 07000000          mov edx,7  

004B7730    .  8B45 F8               mov eax,dword ptr ss:[ebp-8]       ;  试炼码  

004B7733    .  E8 0CCCF4FF          call unpacked.00404344           ;  字符串截取函数  

004B7738    .  8B4D A8               mov ecx,dword ptr ss:[ebp-58]      ;  取得试炼码第 7-14 位,共 8 位  

004B773B    .  8D45 AC               lea eax,dword ptr ss:[ebp-54]  

004B773E    .  BA 20794B00          mov edx,unpacked.004B7920  

004B7743    .  E8 40CAF4FF          call unpacked.00404188  

004B7748    .  8B45 AC               mov eax,dword ptr ss:[ebp-54]  

004B774B    .  E8 D828F5FF          call unpacked.0040A028  

004B7750    .  8BD8                  mov ebx,eax                       ;  转成 16 进制  

004B7752    .  33C0                 xor eax,eax  

004B7754    .  5A                   pop edx  

004B7755    .  59                   pop ecx  

004B7756    .  59                   pop ecx  

004B7757    .  64:8910              mov dword ptr fs:[eax],edx  

004B775A    .  EB 14                 jmp short unpacked.004B7770  

004B775C    .^ E9 13BFF4FF          jmp unpacked.00403674  

004B7761    .  E8 6AC2F4FF          call unpacked.004039D0  

004B7766    .  E9 57010000          jmp unpacked.004B78C2  

004B776B    .  E8 60C2F4FF          call unpacked.004039D0  

004B7770    >  2B5D D8               sub ebx,dword ptr ss:[ebp-28]      ;  减去用户名第 8 位  

004B7773    .  8BC3                  mov eax,ebx  

004B7775    .  B9 07000000          mov ecx,7                        ;  除 7  

004B777A    .  33D2                 xor edx,edx  

004B777C    .  F7F1                  div ecx  

004B777E    .  85D2                 test edx,edx                     ;  不能被 7 整除,over  

004B7780    .  0F85 3C010000         jnz unpacked.004B78C2  

004B7786    .  895D A0               mov dword ptr ss:[ebp-60],ebx  

004B7789    .  33C0                 xor eax,eax  

004B778B    .  8945 A4               mov dword ptr ss:[ebp-5C],eax  

004B778E    .  DF6D A0               fild qword ptr ss:[ebp-60]  

004B7791    .  D835 24794B00        fdiv dword ptr ds:[4B7924]  

004B7797    .  E8 84B4F4FF          call unpacked.00402C20  

004B779C    .  8BD8                  mov ebx,eax                       ;  余数传给 EBX  

004B779E    .  2B5D DC               sub ebx,dword ptr ss:[ebp-24]      ;  减去用户名第 7 位  

004B77A1    .  8BC3                  mov eax,ebx  

004B77A3    .  B9 07000000          mov ecx,7                        ;  除 7  

004B77A8    .  33D2                 xor edx,edx  

004B77AA    .  F7F1                  div ecx  

004B77AC    .  85D2                 test edx,edx  

004B77AE    .  0F85 0E010000         jnz unpacked.004B78C2            ;  不能被 7 整除，OVER  

004B77B4    .  895D A0               mov dword ptr ss:[ebp-60],ebx  

004B77B7    .  33C0                 xor eax,eax  

004B77B9    .  8945 A4               mov dword ptr ss:[ebp-5C],eax  

004B77BC    .  DF6D A0               fild qword ptr ss:[ebp-60]  

004B77BF    .  D835 24794B00        fdiv dword ptr ds:[4B7924]  

004B77C5    .  E8 56B4F4FF          call unpacked.00402C20  
```



```
004B77CA    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B77CC    .  2B5D E0                  sub ebx,dword ptr ss:[ebp-20]        ;  减去用户名第 6 位  

004B77CF    .  8BC3                     mov eax,ebx  

004B77D1    .  B9 07000000             mov ecx,7  

004B77D6    .  33D2                     xor edx,edx  

004B77D8    .  F7F1                     div ecx  

004B77DA    .  85D2                     test edx,edx  

004B77DC    .  0F85 E0000000           jnz unpacked.004B78C2                 ;  不能被 7 整除，OVER  

004B77E2    .  895D A0                  mov dword ptr ss:[ebp-60],ebx  

004B77E5    .  33C0                     xor eax,eax  

004B77E7    .  8945 A4                  mov dword ptr ss:[ebp-5C],eax  

004B77EA    .  DF6D A0                  fild qword ptr ss:[ebp-60]  

004B77ED    .  D835 24794B00           fdiv dword ptr ds:[4B7924]  

004B77F3    .  E8 28B4F4FF             call unpacked.00402C20  

004B77F8    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B77FA    .  2B5D E4                  sub ebx,dword ptr ss:[ebp-1C]        ;  减去用户名第 5 位  

004B77FD    .  8BC3                     mov eax,ebx  

004B77FF    .  B9 07000000             mov ecx,7  

004B7804    .  33D2                     xor edx,edx  

004B7806    .  F7F1                     div ecx  

004B7808    .  85D2                     test edx,edx  

004B780A    .  0F85 B2000000           jnz unpacked.004B78C2                 ;  不能被 7 整除，OVER  

004B7810    .  895D A0                  mov dword ptr ss:[ebp-60],ebx  

004B7813    .  33C0                     xor eax,eax  

004B7815    .  8945 A4                  mov dword ptr ss:[ebp-5C],eax  

004B7818    .  DF6D A0                  fild qword ptr ss:[ebp-60]  

004B781B    .  D835 24794B00           fdiv dword ptr ds:[4B7924]  

004B7821    .  E8 FAB3F4FF             call unpacked.00402C20  

004B7826    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B7828    .  2B5D E8                  sub ebx,dword ptr ss:[ebp-18]        ;  减去用户名第 4 位  

004B782B    .  8BC3                     mov eax,ebx  

004B782D    .  B9 07000000             mov ecx,7  

004B7832    .  33D2                     xor edx,edx  

004B7834    .  F7F1                     div ecx  

004B7836    .  85D2                     test edx,edx  

004B7838    .  0F85 84000000           jnz unpacked.004B78C2                 ;  不能被 7 整除，OVER  

004B783E    .  895D A0                  mov dword ptr ss:[ebp-60],ebx  

004B7841    .  33C0                     xor eax,eax  

004B7843    .  8945 A4                  mov dword ptr ss:[ebp-5C],eax  

004B7846    .  DF6D A0                  fild qword ptr ss:[ebp-60]  

004B7849    .  D835 24794B00           fdiv dword ptr ds:[4B7924]  

004B784F    .  E8 CCB3F4FF             call unpacked.00402C20  

004B7854    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B7856    .  2B5D EC                  sub ebx,dword ptr ss:[ebp-14]        ;  减去用户名第 3 位  

004B7859    .  8BC3                     mov eax,ebx  

004B785B    .  B9 07000000             mov ecx,7  

004B7860    .  33D2                     xor edx,edx  

004B7862    .  F7F1                     div ecx  

004B7864    .  85D2                     test edx,edx  

004B7866    .  75 5A                    jnz short unpacked.004B78C2           ;  不能被 7 整除，OVER  

004B7868    .  895D A0                  mov dword ptr ss:[ebp-60],ebx  

004B786B    .  33C0                     xor eax,eax  

004B786D    .  8945 A4                  mov dword ptr ss:[ebp-5C],eax  

004B7870    .  DF6D A0                  fild qword ptr ss:[ebp-60]  

004B7873    .  D835 24794B00           fdiv dword ptr ds:[4B7924]  

004B7879    .  E8 A2B3F4FF             call unpacked.00402C20  

004B787E    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B7880    .  2BDF                     sub ebx,edi                       ;  减去用户名第 2 位  

004B7882    .  8BC3                     mov eax,ebx  

004B7884    .  B9 07000000             mov ecx,7  

004B7889    .  33D2                     xor edx,edx  

004B788B    .  F7F1                     div ecx  

004B788D    .  85D2                     test edx,edx  

004B788F    .  75 31                    jnz short unpacked.004B78C2           ;  不能被 7 整除，OVER  

004B7891    .  895D A0                  mov dword ptr ss:[ebp-60],ebx  

004B7894    .  33C0                     xor eax,eax  

004B7896    .  8945 A4                  mov dword ptr ss:[ebp-5C],eax  

004B7899    .  DF6D A0                  fild qword ptr ss:[ebp-60]  

004B789C    .  D835 24794B00           fdiv dword ptr ds:[4B7924]  

004B78A2    .  E8 79B3F4FF             call unpacked.00402C20  

004B78A7    .  8BD8                     mov ebx,eax                       ;  余数传给 EBX  

004B78A9    .  2BDE                     sub ebx,esi                       ;  减去用户名第 1 位  

004B78AB    .  8BC3                     mov eax,ebx  

004B78AD    .  B9 07000000             mov ecx,7  

004B78B2    .  33D2                     xor edx,edx  

004B78B4    .  F7F1                     div ecx  

004B78B6    .  85D2                     test edx,edx  

004B78B8    .  75 08                    jnz short unpacked.004B78C2           ;  不能被 7 整除，OVER  

004B78BA    .  A1 60235900             mov eax,dword ptr ds:[592360]  

004B78BF    .  C600 01                 mov byte ptr ds:[eax],1             ;  置注册标志位为 1  

004B78C2    >  33C0                     xor eax,eax  

004B78C4    .  5A                       pop edx  

004B78C5    .  59                       pop ecx  

004B78C6    .  59                       pop ecx  

004B78C7    .  64:8910                 mov dword ptr fs:[eax],edx  

004B78CA    .  68 F9784B00             push unpacked.004B78F9  

004B78CF    >  8D45 A8                  lea eax,dword ptr ss:[ebp-58]  

004B78D2    .  BA 0C000000             mov edx,0C  

004B78D7    .  E8 F4C5F4FF             call unpacked.00403ED0  

004B78DC    .  8D45 F0                  lea eax,dword ptr ss:[ebp-10]  

004B78DF    .  E8 C8C5F4FF             call unpacked.00403EAC  

004B78E4    .  8D45 F8                  lea eax,dword ptr ss:[ebp-8]  

004B78E7    .  BA 02000000             mov edx,2  

004B78EC    .  E8 DFC5F4FF             call unpacked.00403ED0  

004B78F1    .  C3                       retn  

004B78F2    .^ E9 31C0F4FF             jmp unpacked.00403928  

004B78F7    .^ EB D6                   jmp short unpacked.004B78CF  

004B78F9    .  8B45 F4                  mov eax,dword ptr ss:[ebp-C]  

004B78FC    .  5F                       pop edi  

004B78FD    .  5E                       pop esi  

004B78FE    .  5B                       pop ebx  

004B78FF    .  8BE5                    mov esp,ebp  

004B7901    .  5D                       pop ebp  

004B7902    .  C3                       retn  
```

由此可得出注册码的第 `7-14` 位要求满足的条件，将用户名字符串转换成 `7` 进制表示即可满足要求。

### 注册算法第三部分分析

程序在主界面(`TForm1`)显示后，弹出 Nag 窗口，会不会在 `TForm1@FormActivate` 中验证注册码？在 `TForm1` 的 `TForm1@FormActivate` 事件下断 `bp 00581A00`，向下查找来到：

```
00581C97   |>\8B15 4C1E5900          mov edx,dword ptr ds:[591E4C]        ;  unpacked.00593920  

00581C9D   |.  8B12                   mov edx,dword ptr ds:[edx]           ;  注册码  

00581C9F   |.  A1 6C225900           mov eax,dword ptr ds:[59226C]  

00581CA4   |.  8B00                   mov eax,dword ptr ds:[eax]           ;  用户名  

00581CA6   |.  E8 7D5CF3FF           call unpacked.004B7928               ;  重要(检查注册码后 8 位)  

00581CAB   |.  48                     dec eax  

00581CAC      75 08                   jnz short unpacked.00581CB6          ;  爆破点  

00581CAE   |.  A1 78225900           mov eax,dword ptr ds:[592278]  

00581CB3   |.  C600 01               mov byte ptr ds:[eax],1             ；置注册标志位为 1  

00581CB6   |>  8D55 B4                lea edx,[local.19]  

00581CB9   |.  B8 001E5800           mov eax,unpacked.00581E00           ;  ASCII  

"2852314454075600CE2E435312522D36232944210D31445407CF"  

00581CBE   |.  E8 1956F3FF           call unpacked.004B72DC  

00581CC3   |.  8B55 B4               mov edx,[local.19]                  ;  未注册版...  
```



## 汇编代码片段 1

```
00581CC6   |.  A1 1C225900            mov eax,dword ptr ds:[59221C]  
00581CCB   |.  8B00                   mov eax,dword ptr ds:[eax]  
00581CCD <>|.  8B80 DC020000          mov eax,dword ptr ds:[eax+2DC]     ;  *epEffectImage1:N.A.  
00581CD3 <>|.  E8 8453EBFF            call unpacked.0043705C           ;  ->controls.TControl.SetText(TControl;TCaption);  
00581CD8   |.  A1 78225900            mov eax,dword ptr ds:[592278]  
00581CDD   |.  8038 00                cmp byte ptr ds:[eax],0  
00581CE0   |.  74 32                  je short unpacked.00581D14  
00581CE2   |.  8D55 B0                lea edx,[local.20]  
00581CE5   |.  B8 401E5800            mov eax,unpacked.00581E40        ;  ASCII "314454075E1EDC"  
00581CEA   |.  E8 ED55F3FF            call unpacked.004B72DC  
00581CEF   |.  8D45 B0                lea eax,[local.20]              ;  注册给:...  
00581CF2   |.  8B15 6C225900          mov edx,dword ptr ds:[59226C]    ;  unpacked.0059391C  
00581CF8   |.  8B12                   mov edx,dword ptr ds:[edx]       ;  用户名  
00581CFA <>|.  E8 4524E8FF            call unpacked.00404144           ;  ->system.@LStrCat;  
00581CFF   |.  8B55 B0                mov edx,[local.20]  
00581D02   |.  A1 1C225900            mov eax,dword ptr ds:[59221C]  
00581D07   |.  8B00                   mov eax,dword ptr ds:[eax]  
00581D09 <>|.  8B80 DC020000          mov eax,dword ptr ds:[eax+2DC]     ;  *epEffectImage1:N.A.  
00581D0F <>|.  E8 4853EBFF            call unpacked.0043705C           ;  ->controls.TControl.SetText(TControl;TCaption);  
00581D14   |>  33C0                   xor eax,eax  
00581D16   |.  5A                      pop edx  
00581D17   |.  59                      pop ecx  
00581D18   |.  59                      pop ecx  
```

哈哈,果然是，`call unpacked.004B7928` 关键跟进：

`unpacked.004B7928`

## 汇编代码片段 2

```
004B7928    $  55                      push ebp  
004B7929    .  8BEC                    mov ebp,esp  
004B792B    .  83C4 F0                add esp,-10  
004B792E    .  53                      push ebx  
004B792F    .  56                      push esi  
004B7930    .  57                      push edi  
004B7931    .  33C9                    xor ecx,ecx  
004B7933    .  894D F0                mov dword ptr ss:[ebp-10],ecx  
004B7936    .  894D F4                mov dword ptr ss:[ebp-C],ecx  
004B7939    .  8955 FC                mov dword ptr ss:[ebp-4],edx     ;  注册码  
004B793C    .  8B45 FC                mov eax,dword ptr ss:[ebp-4]  
004B793F    .  E8 ACC9F4FF            call unpacked.004042F0           ;  system.@LStrAddRef  
004B7944    .  33C0                    xor eax,eax  
004B7946    .  55                      push ebp  
004B7947    .  68 007A4B00            push unpacked.004B7A00  
004B794C    .  64:FF30                push dword ptr fs:[eax]  
004B794F    .  64:8920                mov dword ptr fs:[eax],esp  
004B7952    .  837D FC 00             cmp dword ptr ss:[ebp-4],0      ;  注册码  
004B7956    .  0F84 81000000          je unpacked.004B79DD  
004B795C    .  33C0                    xor eax,eax  
004B795E    .  55                      push ebp  
004B795F    .  68 A4794B00            push unpacked.004B79A4  
004B7964    .  64:FF30                push dword ptr fs:[eax]  
004B7967    .  64:8920                mov dword ptr fs:[eax],esp  
004B796A    .  8D45 F4                lea eax,dword ptr ss:[ebp-C]  
004B796D    .  50                      push eax  
004B796E    .  B9 08000000            mov ecx,8                       ;  要复制的长度  
004B7973    .  BA 0F000000            mov edx,0F                     ;  起始位置  
004B7978    .  8B45 FC                mov eax,dword ptr ss:[ebp-4]    ;  目标字符串,返回地址[ESP+4]  
004B797B    .  E8 C4C9F4FF            call unpacked.00404344           ;  system.@LStrCopy  
004B7980    .  8D45 F0                lea eax,dword ptr ss:[ebp-10]  
004B7983    .  8B4D F4                mov ecx,dword ptr ss:[ebp-C]    ;  取的注册码后 8 位,即第 15-22 位  
004B7986    .  BA 1C7A4B00            mov edx,unpacked.004B7A1C  
004B798B    .  E8 F8C7F4FF            call unpacked.00404188           ;  system.@LStrCat3  
004B7990    .  8B45 F0                mov eax,dword ptr ss:[ebp-10]  
004B7993    .  E8 9026F5FF            call unpacked.0040A028           ;  sysutils.StrToInt(AnsiString):Integer  
004B7998    .  8BD8                    mov ebx,eax                   ;  转为数值  
004B799A    .  33C0                    xor eax,eax  
004B799C    .  5A                      pop edx  
004B799D    .  59                      pop ecx  
004B799E    .  59                      pop ecx  
004B799F    .  64:8910                mov dword ptr fs:[eax],edx  
004B79A2    .  EB 11                   jmp short unpacked.004B79B5  
004B79A4    .^ E9 CBBCEF4FF           jmp unpacked.00403674  
004B79A9    .  E8 22C0F4FF            call unpacked.004039D0  
004B79AE    .  EB 2D                   jmp short unpacked.004B79DD  
004B79B0    .  E8 1BC0F4FF            call unpacked.004039D0  
004B79B5    >  81F3 AAAAAAAA          xor ebx,AAAAAAAA              ;  注册第 15-22 位数值 XOR $AAAAAAAA  
004B79BB    .  BA 00010000            mov edx,100                   ；十进制 256  
004B79C0    .  A1 701F5900            mov eax,dword ptr ds:[591F70]  
004B79C5    >  3B18                    cmp ebx,dword ptr ds:[eax]    ;  查表 A ，表长 256  
004B79C7       75 09                  jnz short unpacked.004B79D2  
004B79C9    .  C745 F8 01000000      mov dword ptr ss:[ebp-8],1    ；运算结果在表中置标志 1  
004B79D0    .  EB 0B                   jmp short unpacked.004B79DD  
004B79D2    >  83C0 04                add eax,4  
004B79D5    .  4A                      dec edx  
004B79D6    .^ 75 ED                   jnz short unpacked.004B79C5  
004B79D8    .  33C0                    xor eax,eax  
004B79DA    .  8945 F8                mov dword ptr ss:[ebp-8],eax  
004B79DD    >  33C0                    xor eax,eax  
004B79DF    .  5A                      pop edx  
004B79E0    .  59                      pop ecx  
004B79E1    .  59                      pop ecx  
004B79E2    .  64:8910                mov dword ptr fs:[eax],edx  
004B79E5    .  68 077A4B00            push unpacked.004B7A07  
004B79EA    >  8D45 F0                lea eax,dword ptr ss:[ebp-10]  
004B79ED    .  BA 02000000            mov edx,2  
004B79F2    .  E8 D9C4F4FF            call unpacked.00403ED0  
004B79F7    .  8D45 FC                lea eax,dword ptr ss:[ebp-4]  
004B79FA    .  E8 ADC4F4FF            call unpacked.00403EAC  
004B79FF    .  C3                      retn  
```

## 表 A 数据

表 A 的位置根据调试可知:`00586C1C~0058700C=00586C1C+(100*4)-1`

```
00586C1C      A56363C6    847C7CF8    997777EE    8D7B7BF6  
00586C2C      0DF2F2FF    BD6B6BD6    B16F6FDE    54C5C591  
00586C3C      50303060    03010102    A96767CE    7D2B2B56  
00586C4C      19FEFEE7    62D7D7B5    E6ABAB4D    9A7676EC  
00586C5C      45CACA8F    9D82821F    40C9C989    877D7DFA  
00586C6C      15FAFAEF    EB5959B2    C947478E    0BF0F0FB  
00586C7C      ECADAD41    67D4D4B3    FDA2A25F    EAAFAF45  
00586C8C      BF9C9C23    F7A4A453    967272E4    5BC0C09B  
00586C9C      C2B7B775    1CFDFDE1    AE93933D    6A26264C  
00586CAC      5A36366C    413F3F7E    02F7F7F5    4FCCCC83  
00586CBC      5C343468    F4A5A551    34E5E5D1    08F1F1F9  
00586CCC      937171E2    73D8D8AB    53313162    3F15152A  
00586CDC      0C040408    52C7C795    65232346    5EC3C39D  
00586CEC      28181830    A1969637    0F05050A    B59A9A2F  
00586CFC      0907070E    36121224    9B80801B    3DE2E2DF  
00586D0C      26EBEBCD    6927274E    CDB2B27F    9F7575EA  
00586D1C      1B090912    9E83831D    742C2C58    2E1A1A34  
00586D2C      2D1B1B36    B26E6EDC    EE5A5AB4    FBA0A05B  
00586D3C      F65252A4    4D3B3B76    61D6D6B7    CEB3B37D  
```



### 数据表

```
00586D4C      7B292952    3EE3E3DD    712F2F5E    97848413  
00586D5C      F55353A6    68D1D1B9    00000000    2CEDEDC1  
00586D6C      60202040    1FFCFCE3    C8B1B179    ED5B5BB6  
00586D7C      BE6A6AD4    46CBCB8D    D9BEBE67    4B393972  
00586D8C      DE4A4A94    D44C4C98    E85858B0    4ACFCF85  
00586D9C      6BD0D0BB    2AEFEFC5    E5AAAA4F    16FBFBED  
00586DAC      C5434386    D74D4D9A    55333366    94858511  
00586DBC      CF45458A    10F9F9E9    06020204    817F7FFE  
00586DCC      F05050A0    443C3C78    BA9F9F25    E3A8A84B  
00586DDC      F35151A2    FEA3A35D    C0404080    8A8F8F05  
00586DEC      AD92923F    BC9D9D21    48383870    04F5F5F1  
00586DFC      DFBCBC63    C1B6B677    75DADAAF    63212142  
00586E0C      30101020    1AFFFFE5    0EF3F3FD    6DD2D2BF  
00586E1C      4CCDCD81    140C0C18    35131326    2FECECC3  
00586E2C      E15F5FBE    A2979735    CC444488    3917172E  
00586E3C      57C4C493    F2A7A755    827E7EFC    473D3D7A  
00586E4C      AC6464C8    E75D5DBA    2B191932    957373E6  
00586E5C      A06060C0    98818119    D14F4F9E    7FDCDCA3  
00586E6C      66222244    7E2A2A54    AB90903B    8388880B  
00586E7C      CA46468C    29EEEEC7    D3B8B86B    3C141428  
00586E8C      79DEDEA7    E25E5EBC    1D0B0B16    76DBDBAD  
00586E9C      3BE0E0DB    56323264    4E3A3A74    1E0A0A14  
00586EAC      DB494992    0A06060C    6C242448    E45C5CB8  
00586EBC      5DC2C29F    6ED3D3BD    EFACAC43    A66262C4  
00586ECC      A8919139    A4959531    37E4E4D3    8B7979F2  
00586EDC      32E7E7D5    43C8C88B    5937376E    B76D6DDA  
00586EEC      8C8D8D01    64D5D5B1    D24E4E9C    E0A9A949  
00586EFC      B46C6CD8    FA5656AC    07F4F4F3    25EAEACF  
00586F0C      AF6565CA    8E7A7AF4    E9AEAE47    18080810  
00586F1C      D5BABA6F    887878F0    6F25254A    722E2E5C  
00586F2C      241C1C38    F1A6A657    C7B4B473    51C6C697  
00586F3C      23E8E8CB    7CDDDDA1    9C7474E8    211F1F3E  
00586F4C      DD4B4B96    DCBDBD61    868B8B0D    858A8A0F  
00586F5C      907070E0    423E3E7C    C4B5B571    AA6666CC  
00586F6C      D8484890    05030306    01F6F6F7    120E0E1C  
00586F7C      A36161C2    5F35356A    F95757AE    D0B9B969  
00586F8C      91868617    58C1C199    271D1D3A    B99E9E27  
00586F9C      38E1E1D9    13F8F8EB    B398982B    33111122  
00586FAC      BB6969D2    70D9D9A9    898E8E07    A7949433  
00586FBC      B69B9B2D    221E1E3C    92878715    20E9E9C9  
00586FCC      49CECE87    FF5555AA    78282850    7ADFDFA5  
00586FDC      8F8C8C03    F8A1A159    80898909    170D0D1A  
00586FEC      DABFBF65    31E6E6D7    C6424284    B86868D0  
00586FFC      C3414182    B0999929    772D2D5A    110F0F1E  
0058700C      CBB0B07B    FC5454A8    D6BBBB6D    3A16162C  
```

由此可得：注册第 15-22 位转为数值 `XOR` `$AAAAAAAA` 要求在表 A 中即可。

### XOR 运算性质

`xor` 运算有以下性质:

*   结合率: `(a ^ b) ^ c = a ^ (b ^ c)`
*   交换率: `a ^ b = b ^ a`
*   另外: `a⁰ = a` , `a^a = 0`

由此可知：

```
a^x = b
=> a^x^x = b^x
=> a^(x^x) = b^x
=> a⁰ = b^x
=> a = b^x
```

现在要的数据是 `a`，`x = 0xAAAAAAAA`，
`b = A(i)` (即表 A 第 `i` 项，`i=0~255`, 以 `0` 为起始坐标)

显然你要的 `a` 有 256 种可能，即 `a(i) = A(i) ^ 0xAAAAAAAA`

### 算法总结

算法共分 3 部分验证：

1.  在输入注册码时必须以 `'BINBIN'` 开头，才能激活注册按钮。
2.  注册码第 7-14 位要求：将用户名字符串转换成 7 进制表示即可满足要求;
3.  注册第 15-22 位转为数值 `XOR` `$AAAAAAAA` 要求在表 A 中
4.  注册码至少 22 位，可以同一用户名有多个注册码。

### 注册机 Delphi 源码

```delphi
Const
  Key: Array[0..255] Of DWORD = (
    $A56363C6, $847C7CF8, $997777EE, $8D7B7BF6
    , $0DF2F2FF, $BD6B6BD6, $B16F6FDE, $54C5C591
    , $50303060, $03010102, $A96767CE, $7D2B2B56
    , $19FEFEE7, $62D7D7B5, $E6ABAB4D, $9A7676EC
    , $45CACA8F, $9D82821F, $40C9C989, $877D7DFA
    , $15FAFAEF, $EB5959B2, $C947478E, $0BF0F0FB
    , $ECADAD41, $67D4D4B3, $FDA2A25F, $EAAFAF45
    , $BF9C9C23, $F7A4A453, $967272E4, $5BC0C09B
    , $C2B7B775, $1CFDFDE1, $AE93933D, $6A26264C
    , $5A36366C, $413F3F7E, $02F7F7F5, $4FCCCC83
    , $5C343468, $F4A5A551, $34E5E5D1, $08F1F1F9
    , $937171E2, $73D8D8AB, $53313162, $3F15152A
    , $0C040408, $52C7C795, $65232346, $5EC3C39D
    , $28181830, $A1969637, $0F05050A, $B59A9A2F
    , $0907070E, $36121224, $9B80801B, $3DE2E2DF
    , $26EBEBCD, $6927274E, $CDB2B27F, $9F7575EA
    , $1B090912, $9E83831D, $742C2C58, $2E1A1A34
    , $2D1B1B36, $B26E6EDC, $EE5A5AB4, $FBA0A05B
    , $F65252A4, $4D3B3B76, $61D6D6B7, $CEB3B37D
    , $7B292952, $3EE3E3DD, $712F2F5E, $97848413
    , $F55353A6, $68D1D1B9, $00000000, $2CEDEDC1
    , $60202040, $1FFCFCE3, $C8B1B179, $ED5B5BB6
    , $BE6A6AD4, $46CBCB8D, $D9BEBE67, $4B393972
    , $DE4A4A94, $D44C4C98, $E85858B0, $4ACFCF85
    , $6BD0D0BB, $2AEFEFC5, $E5AAAA4F, $16FBFBED
    , $C5434386, $D74D4D9A, $55333366, $94858511
    , $CF45458A, $10F9F9E9, $06020204, $817F7FFE
    , $F05050A0, $443C3C78, $BA9F9F25, $E3A8A84B
    , $F35151A2, $FEA3A35D, $C0404080, $8A8F8F05
    , $AD92923F, $BC9D9D21, $48383870, $04F5F5F1
    , $DFBCBC63, $C1B6B677, $75DADAAF, $63212142
    , $30101020, $1AFFFFE5, $0EF3F3FD, $6DD2D2BF
    , $4CCDCD81, $140C0C18, $35131326, $2FECECC3
    , $E15F5FBE, $A2979735, $CC444488, $3917172E
    , $57C4C493, $F2A7A755, $827E7EFC, $473D3D7A
    , $AC6464C8, $E75D5DBA, $2B191932, $957373E6
    , $A06060C0, $98818119, $D14F4F9E, $7FDCDCA3
    , $66222244, $7E2A2A54, $AB90903B, $8388880B
    , $CA46468C, $29EEEEC7, $D3B8B86B, $3C141428
    , $79DEDEA7, $E25E5EBC, $1D0B0B16, $76DBDBAD
    , $3BE0E0DB, $56323264, $4E3A3A74, $1E0A0A14
    , $DB494992, $0A06060C, $6C242448, $E45C5CB8
    , $5DC2C29F, $6ED3D3BD, $EFACAC43, $A66262C4
    , $A8919139, $A4959531, $37E4E4D3, $8B7979F2
    , $32E7E7D5, $43C8C88B, $5937376E, $B76D6DDA
    , $8C8D8D01, $64D5D5B1, $D24E4E9C, $E0A9A949
    , $B46C6CD8, $FA5656AC, $07F4F4F3, $25EAEACF
    , $AF6565CA, $8E7A7AF4, $E9AEAE47, $18080810
    , $D5BABA6F, $887878F0, $6F25254A, $722E2E5C
    , $241C1C38, $F1A6A657, $C7B4B473, $51C6C697
    , $23E8E8CB, $7CDDDDA1, $9C7474E8, $211F1F3E
    , $DD4B4B96, $DCBDBD61, $868B8B0D, $858A8A0F
    , $907070E0, $423E3E7C, $C4B5B571, $AA6666CC
    , $D8484890, $05030306, $01F6F6F7, $120E0E1C
    , $A36161C2, $5F35356A, $F95757AE, $D0B9B969
    , $91868617, $58C1C199, $271D1D3A, $B99E9E27
    , $38E1E1D9, $13F8F8EB, $B398982B, $33111122
    , $BB6969D2, $70D9D9A9, $898E8E07, $A7949433
    , $B69B9B2D, $221E1E3C, $92878715, $20E9E9C9
    , $49CECE87, $FF5555AA, $78282850, $7ADFDFA5
    , $8F8C8C03, $F8A1A159, $80898909, $170D0D1A
    , $DABFBF65, $31E6E6D7, $C6424284, $B86868D0
    , $C3414182, $B0999929, $772D2D5A, $110F0F1E
    , $CBB0B07B, $FC5454A8, $D6BBBB6D, $3A16162C);

Function StrToNum(S: String; Base: Integer): Integer;

Var
  i: Integer;

Begin
  Result := 0;
  For i := 1 To Length(S) - 1 Do Result := (Result + Ord(S[i])) * Base;
  Result := Result + Ord(S[Length(S)]);
End;

Procedure TForm1.btn1Click(Sender: TObject);

Var
  Name: String;

Begin
  Name := LeftStr(edt1.Text + '      ', 8);
  edt2.Text := 'BINBIN' + IntToHex(StrToNum(Name, 7), 8) + IntToHex(Key[Random(255)] Xor $AAAAAAAA, 8);
End;
```

大家可以验证一下：

*   用户名：`Baby2008`
*   注册码：`BINBIN0409601C00CCCC66`

哇，快 12 点钟了，眼睛都睁不开了，写的破文可能阐述的不是很清楚、有条理，望各位看官见谅，有兴趣请跟贴顶我一下^\_^。

### 特别感谢

`goldenegg` 对注册算法中的数学计算模式予以指正，为了保证文章的正确性，窃取 `goldenegg` 的思想为己有了。

#### 指正内容

> 你数学没学好啊
>
> 记 `xor` 运算符为 `^`
>
> `2ED6D652^AAAAAAAA` = `847C7CF8`, 为表 A 的第二项
>
> `2ED6D652` 居然没出现在你的列表中.
>
> 实际上, 表 A 有 256 项, 数据就有 256 项,
>
> 你只"穷举"出 16 项, 不知你怎么算的.

`xor` 运算有以下性质:

*   结合率: `(a ^ b) ^ c = a ^ (b ^ c)`
*   交换率: `a ^ b = b ^ a`
*   另外: `a⁰ = a` , `a^a = 0`

由此可知：

```
a^x = b
=> a^x^x = b^x
=> a^(x^x) = b^x
=> a⁰ = b^x
=> a = b^x
```

现在你要的数据是 `a`，`x = 0xAAAAAAAA`，
`b = A(i)` (即表Ａ第 `i` 项，`i=0~255`, 以 `0` 为起始坐标)

显然你要的 `a` 有 256 种可能 `a(i) = A(i) ^ 0xAAAAAAAA`

所以穷举是根本没必要的．

还有你不知怎么表达的那一步：

```
SN07_14 := (((((((Ord(Name[1])) * 7 + Ord(Name[2])) * 7 + Ord(Name[3])) * 7 +　Ord(Name[4])) * 7
```



`+ Ord(Name[5])) * 7 + Ord(Name[6])) * 7 + Ord(Name[7])) * 7 + Ord(Name[8]));`

这样你不明白，换一个表达就知道了吧：

`((((1*10)+2)*10)+3)*10+4 = 1234`

看出来了吧，后者就是我们最常见的十进制数的表示方法，

因此，原算法是一个 7 进制表示方法！

也就是把名字当成 7 进制，把 7 进制的数算出来即可．

所以数学重要啊，其实我数学也没学好，现在有点后悔呢．

`4dd466d0e1e0cc123afbe504da2462ca`
