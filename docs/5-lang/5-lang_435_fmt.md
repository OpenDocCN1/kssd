
# QQ 挂太阳专家 1.0

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-379.htm`](https://www.pediy.com/kssd/pediy08/pediy8-379.htm)

- **【目 标】**: `QQ 挂太阳专家 1.0`
- **【工 具】**: `PEiD0.94`、`VBExplorer1.1`
- **【任 务】**: `VB P-CODE` 逆向分析
- **【操作平台】**: `Win2003.SE.sp1`
- **【作 者】**: `KuNgBiM{BCG}{DFCG}{DCM}{DCT}{SLT}`
- **【相关链接】**: `N/A`

**【简要说明】**: QQ 升级已经是我们使用 QQ 很重要的部分，只要你事先输入 QQ 号码和密码，然后点升级，就可以模拟 QQ 登录了，它最主要的特点是可以同时在一台机器上使用成千上万个 QQ 号升级! 每天点一次，万个 QQ 升级好轻松 (为了保护你的 QQ 密码,请不要在网吧使用该软件)

**【作者声明】**: 很久没有写什么文章了，手也生疏了，今天从网上抓了个小软件拿来练练手，哪知....

**【详细过程】**: 首先我们一定要查壳，作为一名初学破解的朋友来说，这点非常重要。如果软件程序是加的一个猛壳的话，就要提前作好“知难而退”的打算了...废话不多说...开始我们今天所要做的 `VB` 程序的 `P-CODE` 轻松之旅~~

还好，作者可能也是个初学者~~呵呵，没有对软件程序做任何保护措施，`Microsoft Visual Basic 5.0 / 6.0` 的程序，起初用 `Ollydbg` 加载下，随便设置了几个断点（？？？？？）纳闷了一下...怎么什么 API 都没有啊？难道是 `P-CODE` 的编译方式？

换一个调试器试试（也不完全叫做调试器，只能算是 `VB` 资源修改器吧），这下`"VBExplorer"`登台演出...

随便捣鼓了一番~~看来该程序确实为 `P-CODE` 编译方式所编译~

装载完毕后，双击左侧的`"frmLogin"`窗体，查看该窗体中全部的调用指令：

```text
:0040592B  050000             ImpAdLdRf            ;Push ptr
:0040592E  240100             NewIfNullPr          ;[Pop] [SR]
:00405931  0D10000200         VCallHresult         ;Call ptr_004043BC
:00405936  1A78FF             FFree1Ad             ;Push [LOCAL_0088]; Call [[[LOCAL_0088]]+8]; [[LOCAL_0088]]=0 
:00405939  13                 ExitProcHresult      
:0040593A  FF                 Unknown              
:0040593B  FF                 Unknown              

[Form.Load]                                      //双击来到这里，装载程序的窗体

:004059CC  0478FF             FLdRfVar             ;Push LOCAL_0088
****************Reference To:sub_00405E30

                       |
:004059CF  1004070800       ThisVCallHresult       ;Call ptr_00402AAF
:004059D4  6C78FF             ILdRf                ;Push DWORD [LOCAL_0088]
:004059D7  FBFE              CStrI4               ;vbaStrI4
:004059D9  2374FF             FStStrNoPop          ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=[stack]
:004059DC  21                 FLdPrThis            ;[SR]=[stack2]
:004059DD  0FFC02             VCallAd              ;Return the control index 01
:004059E0  1970FF             FStAdFunc            
:004059E3  0870FF             FLdPr                ;[SR]=[LOCAL_0090]
****************Reference To:[propput]TextBox.Text             //赋值给一个文本框

                       |
:004059E6  0DA4000300         VCallHresult         ;Call ptr_00404550
:004059EB  2F74FF             FFree1Str            ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=0
:004059EE  1A70FF             FFree1Ad             ;Push [LOCAL_0090]; Call [[[LOCAL_0090]]+8]; [[LOCAL_0090]]=0 
:004059F1  13                 ExitProcHresult      

[sub_00405E30]                                   //声明变量

:00405DBC  F500010000         LitI4                ;Push 00000100
:00405DC1  6C6CFF             ILdRf                ;Push DWORD [LOCAL_0094]
:00405DC4  0454FF             FLdRfVar             ;Push LOCAL_00AC
:00405DC7  34                 CStr2Ansi            ;vbaStrToAnsi
:00405DC8  6C54FF             ILdRf                ;Push DWORD [LOCAL_00AC]
:00405DCB  0464FF             FLdRfVar             ;Push LOCAL_009C
:00405DCE  0468FF             FLdRfVar             ;Push LOCAL_0098
:00405DD1  0474FF             FLdRfVar             ;Push LOCAL_008C
:00405DD4  F500010000         LitI4                ;Push 00000100
:00405DD9  6C70FF             ILdRf                ;Push DWORD [LOCAL_0090]
:00405DDC  045CFF             FLdRfVar             ;Push LOCAL_00A4
:00405DDF  34                 CStr2Ansi            ;vbaStrToAnsi
:00405DE0  6C5CFF             ILdRf                ;Push DWORD [LOCAL_00A4]
******Possible String Ref To->"c:\\"                     //定位系统 C 盘的识别符

                       |
:00405DE3  1B1000             LitStr               ;Push ptr_004045E0
:00405DE6  0460FF             FLdRfVar             ;Push LOCAL_00A0
:00405DE9  34                 CStr2Ansi            ;vbaStrToAnsi
:00405DEA  6C60FF             ILdRf                ;Push DWORD [LOCAL_00A0]
****************Reference To:kernel32.GetVolumeInformationA      //获取与一个磁盘卷有关的信息

                       |
:00405DED  0A11002000         ImpAdCallFPR4        ;Call ptr_0040415C; check stack 0020; Push EAX
:00405DF2  3C                 SetLastSystemError   ;Kernel GetLastError
:00405DF3  6C5CFF             ILdRf                ;Push DWORD [LOCAL_00A4]
:00405DF6  0458FF             FLdRfVar             ;Push LOCAL_00A8
:00405DF9  FC58              CStr2Uni             ;vbaStrToUnicode
:00405DFB  6C58FF             ILdRf                ;Push DWORD [LOCAL_00A8]
:00405DFE  6C70FF             ILdRf                ;Push DWORD [LOCAL_0090]
:00405E01  470000             StFixedStr           ;vbaLsetFixstr
:00405E04  6C54FF             ILdRf                ;Push DWORD [LOCAL_00AC]
:00405E07  0450FF             FLdRfVar             ;Push LOCAL_00B0
:00405E0A  FC58              CStr2Uni             ;vbaStrToUnicode
:00405E0C  6C50FF             ILdRf                ;Push DWORD [LOCAL_00B0]
:00405E0F  6C6CFF             ILdRf                ;Push DWORD [LOCAL_0094]
:00405E12  470000             StFixedStr           ;vbaLsetFixstr
:00405E15  320A0060FF5CFF58    FFreeStr             ;Do SysFreeString [arg_n]; [arg_n]=0 000A/2 times ~ arg 
:00405E22  6C74FF             ILdRf                ;Push DWORD [LOCAL_008C]
:00405E25  7178FF             FStR4                ;Pop DWORD [LOCAL_0088]
:00405E28  FF2F0C000400       ExitProcCbHresult    
:00405E2E  FF                 Unknown              
:00405E2F  FF                 Unknown              

////////////////////////////////////////////////////////////////////////////////////////
'获取与一个磁盘卷有关的信息的标准模块：

Public Declare Function GetVolumeInformation Lib "kernel32" _
Alias "GetVolumeInformationA" (ByVal lpRootPathName As String, _
ByVal lpVolumeNameBuffer As String, ByVal nVolumeNameSize As Long, _
lpVolumeSerialNumber As Long, lpMaximumComponentLength As Long, _
lpFileSystemFlags As Long, ByVal lpFileSystemNameBuffer As String, _
ByVal nFileSystemNameSize As Long) As Long

'窗体启动时取机器码：

Private Sub Form_Load()

Dim Driver, VolName, Fsys As String

Dim volNumber, MCM, FSF As Long

Dim res As Long

Driver = "c:\\"

res = GetVolumeInformation(Driver, VolName, 127, volNumber, MCM, FSF, Fsys, 127)

txtUserName = volNumber

End Sub

////////////////////////////////////////////////////////////////////////////////////////

[cmdOK.Click]                                    //"确定"按钮

:00405E88  0474FF             FLdRfVar             ;Push LOCAL_008C
:00405E8B  21                 FLdPrThis            ;[SR]=[stack2]
:00405E8C  0F0803             VCallAd              ;Return the control index 04
:00405E8F  1978FF             FStAdFunc            
:00405E92  0878FF             FLdPr                ;[SR]=[LOCAL_0088]
****************Reference To:[propget]TextBox.Text             //赋值给另一个文本框

                       |
:00405E95  0DA0000300         VCallHresult         ;Call ptr_00404550
:00405E9A  6C74FF             ILdRf                ;Push DWORD [LOCAL_008C]
******Possible String Ref To->"hge5768ghdg"                 //取一个固定的字符串？？难道这个是注册码？

                       |
:00405E9D  1B0400             LitStr               ;Push ptr_00404564
:00405EA0  FB30              EqStr                
:00405EA2  2F74FF             FFree1Str            ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=0
```



:00405EA5 `1A78FF` `FFree1Ad` ;Push `[LOCAL_0088]`; Call `[[[LOCAL_0088]]+8]`; `[[LOCAL_0088]]=0`

:00405EA8 `1C8600` `BranchF` ;If Pop=0 then ESI=00405F0E

\*\*\*\*\*\*Possible String Ref To->`llw_start` `//取另一个固定的字符串`

:00405EAB `1B0500` `LitStr` ;Push `ptr_00404598`

:00405EAE `436CFF` `FStStrCopy` ;`[LOCAL_0094]=SysAllocStringByteLen(Pop, [Pop-4]);`

SysFreeString Pop

:00405EB1 `046CFF` `FLdRfVar` ;Push `LOCAL_0094`

\*\*\*\*\*\*Possible String Ref To->`Run` `//取另一个固定的字符串`

:00405EB4 `1B0600` `LitStr` ;Push `ptr_0040458C`

:00405EB7 `4370FF` `FStStrCopy` ;`[LOCAL_0090]=SysAllocStringByteLen(Pop, [Pop-4]);`

SysFreeString Pop

:00405EBA `0470FF` `FLdRfVar` ;Push `LOCAL_0090`

\*\*\*\*\*\*Possible String Ref To->`reg` `//取另一个固定的字符串`

:00405EBD `1B0700` `LitStr` ;Push `ptr_00404580`

:00405EC0 `4374FF` `FStStrCopy` ;`[LOCAL_008C]=SysAllocStringByteLen(Pop, [Pop-4]);`

SysFreeString Pop

:00405EC3 `0474FF` `FLdRfVar` ;Push `LOCAL_008C`

\*\*\*\*\*\*\*\*\*\*\*Reference To:`sub_00405C04` `//返回一个变量值提供给对话框`

:00405EC6 `1008070800` `ThisVCallHresult` ;Call `ptr_00402AC3`

:00405ECB `32060074FF70FF6C` `FFreeStr` ;Do SysFreeString `[arg_n]`; `[arg_n]=0` 0006/2 times ~ arg

:00405ED4 `27ECFE` `LitVar` ;PushVar `LOCAL_0114`

:00405ED7 `270CFF` `LitVar` ;PushVar `LOCAL_00F4`

:00405EDA `272CFF` `LitVar` ;PushVar `LOCAL_00D4`

:00405EDD `F500000000` `LitI4` ;Push `00000000`

\*\*\*\*\*\*Possible String Ref To->`"????\n"` `//这里应该是“注册码错误”`

:00405EE2 `3A5CFF0900` `LitVarStr` ;PushVarString `ptr_004045B0`

:00405EE7 `4E4CFF` `FStVarCopyObj` ;`[LOCAL_00B4]=vbaVarDup(Pop)`

:00405EEA `044CFF` `FLdRfVar` ;Push `LOCAL_00B4`

\*\*\*\*\*\*\*\*\*\*Reference To->`msvbvm60.rtcMsgBox` `//调用 VB 中的对话框函数`

:00405EED `0A0A001400` `ImpAdCallFPR4` ;Call `ptr_00401066`; check stack `0014`; Push `EAX`

:00405EF2 `3608004CFF2CFF0C` `FFreeVar` ;Free `0008/2` variants

\*\*\*\*\*\*Possible String Ref To->`QQ??膙` `//这里应该是“QQ 挂太阳专家”`

:00405EFD `1B0B00` `LitStr` ;Push `ptr_004045C0`

:00405F00 `050C00` `ImpAdLdRf` ;Push `ptr`

:00405F03 `240D00` `NewIfNullPr` ;`[Pop] [SR]`

:00405F06 `0D54000E00` `VCallHresult` ;Call `ptr_00403BD8`

:00405F0B `1EAF00` `Branch` ;ESI=`00405F37`

:00405F0E `27ECFE` `LitVar` ;PushVar `LOCAL_0114`

:00405F11 `270CFF` `LitVar` ;PushVar `LOCAL_00F4`

:00405F14 `272CFF` `LitVar` ;PushVar `LOCAL_00D4`

:00405F17 `F500000000` `LitI4` ;Push `00000000`

\*\*\*\*\*\*Possible String Ref To->`?????` `//这里应该是“注册成功”`

:00405F1C `3A5CFF0F00` `LitVarStr` ;PushVarString `ptr_004045D0`

:00405F21 `4E4CFF` `FStVarCopyObj` ;`[LOCAL_00B4]=vbaVarDup(Pop)`

:00405F24 `044CFF` `FLdRfVar` ;Push `LOCAL_00B4`

\*\*\*\*\*\*\*\*\*\*Reference To->`msvbvm60.rtcMsgBox` `//调用 VB 中的对话框函数`

:00405F27 `0A0A001400` `ImpAdCallFPR4` ;Call `ptr_00401066`; check stack `0014`; Push `EAX`

:00405F2C `3608004CFF2CFF0C` `FFreeVar` ;Free `0008/2` variants

:00405F37 `13` `ExitProcHresult` ;

`[sub_00405C04]` `//声明变量`

:00405B90 `0478FF` `FLdRfVar` ;Push `LOCAL_0088`

\*\*\*\*\*\*Possible String Ref To->`"Software\\112334\\"` `//定位注册表中的位置`

:00405B93 `1B1200` `LitStr` ;Push `ptr_004045EC`

:00405B96 `801000` `ILdI4` ;Push `DWORD [STACK_0010]`

:00405B99 `2A` `ConcatStr` ;`vbaStrCat`

:00405B9A `2374FF` `FStStrNoPop` ;SysFreeString `[LOCAL_008C]`; `[LOCAL_008C]=[stack]`

:00405B9D `0470FF` `FLdRfVar` ;Push `LOCAL_0090`

:00405BA0 `34` `CStr2Ansi` ;`vbaStrToAnsi`

:00405BA1 `6C70FF` `ILdRf` ;Push `DWORD [LOCAL_0090]`

:00405BA4 `F502000080` `LitI4` ;Push `80000002`

\*\*\*\*\*\*\*\*\*\*\*Reference To:`advapi32.dll.RegCreateKeyA` `//在指定的项下创建一个新项。`

:00405BA9 `0A13000C00` `ImpAdCallFPR4` ;Call `ptr_004041B8`; check stack `000C`; Push `EAX`

:00405BAE `3C` `SetLastSystemError` ;`Kernel GetLastError`

:00405BAF `32040074FF70FF` `FFreeStr` ;Do SysFreeString `[arg_n]`; `[arg_n]=0` 0004/2 times ~ arg

:00405BB6 `800C00` `ILdI4` ;Push `DWORD [STACK_000C]`

:00405BB9 `4A` `FnLenStr` ;`vbaLenBstr`

:00405BBA `800C00` `ILdI4` ;Push `DWORD [STACK_000C]`

:00405BBD `0470FF` `FLdRfVar` ;Push `LOCAL_0090`

:00405BC0 `34` `CStr2Ansi` ;`vbaStrToAnsi`

:00405BC1 `6C70FF` `ILdRf` ;Push `DWORD [LOCAL_0090]`

:00405BC4 `F501000000` `LitI4` ;Push `00000001`

:00405BC9 `F500000000` `LitI4` ;Push `00000000`

:00405BCE `801400` `ILdI4` ;Push `DWORD [STACK_0014]`

:00405BD1 `0474FF` `FLdRfVar` ;Push `LOCAL_008C`

:00405BD4 `34` `CStr2Ansi` ;`vbaStrToAnsi`

:00405BD5 `6C74FF` `ILdRf` ;Push `DWORD [LOCAL_008C]`

:00405BD8 `6C78FF` `ILdRf` ;Push `DWORD [LOCAL_0088]`

\*\*\*\*\*\*\*\*\*\*\*Reference To:`advapi32.dll.RegSetValueExA` `//设置指定项的值`

:00405BDB `0A14001800` `ImpAdCallFPR4` ;Call `ptr_00404234`; check stack `0018`; Push `EAX`

:00405BE0 `3C` `SetLastSystemError` ;`Kernel GetLastError`

:00405BE1 `6C74FF` `ILdRf` ;Push `DWORD [LOCAL_008C]`

:00405BE4 `6C1400` `ILdRf` ;Push `DWORD [STACK_0014]`

:00405BE7 `FC58` `CStr2Uni` ;`vbaStrToUnicode`

:00405BE9 `6C70FF` `ILdRf` ;Push `DWORD [LOCAL_0090]`

:00405BEC `6C0C00` `ILdRf` ;Push `DWORD [STACK_000C]`

:00405BEF `FC58` `CStr2Uni` ;`vbaStrToUnicode`

:00405BF1 `32040074FF70FF` `FFreeStr` ;Do SysFreeString `[arg_n]`; `[arg_n]=0` 0004/2 times ~ arg

:00405BF8 `6C78FF` `ILdRf` ;Push `DWORD [LOCAL_0088]`

\*\*\*\*\*\*\*\*\*\*\*Reference To:`advapi32.dll.RegCloseKey` `//关闭系统注册表中的一个项（或键）`

:00405BFB `0A15000400` `ImpAdCallFPR4` ;Call `ptr_004041EC`; check stack `0004`; Push `EAX`

:00405C00 `3C` `SetLastSystemError` ;`Kernel GetLastError`

:00405C01 `13` `ExitProcHresult` ;

:00405C02 `FF` `Unknown` ;



#### :00405C03 FF

`Unknown`

'注册表写入部分:

```vb
Public Const HKEY_LOCAL_MACHINE = &H80000002

Public Const REG_SZ = 1

'在指定的项下创建一个新项的标准模块:

Declare Function RegCreateKey Lib "advapi32.dll" Alias "RegCreateKeyA" (ByVal hKey As Long, ByVal lpSubKey As String, phkResult As Long) As Long

'关闭系统注册表中的一个项(或键)的标准模块:

Declare Function RegCloseKey Lib "advapi32.dll" (ByVal hKey As Long) As Long

'设置指定项的值的标准模块:

Declare Function RegSetValueEx Lib "advapi32.dll" Alias "RegSetValueExA" (ByVal hKey As Long, ByVal lpValueName As String, ByVal Reserved As Long, ByVal dwType As Long, lpData As Any, ByVal cbData As Long) As Long
```

'注册码判断：

```vb
Private Sub cmdOK_Click()
    Dim hKey, ret As Long

    If txtPassword <> "hge5768ghdg" Then
        MsgBox "注册码错误"
    Else
        RegCreateKey HKEY_LOCAL_MACHINE, "Software\\112334\\Run", hKey
        RegSetValueEx hKey, "llw_start", 0, REG_SZ, ByVal "reg", 13
        MsgBox "注册成功"
        RegCloseKey hKey
    End If
End Sub
```

根据以上的分析，我们就能完完全全的模拟出来该软件的注册程序了。

#### Cracked By

`KuNgBiM{BCG}{DFCG}{DCM}{DCT}{SLT}`

`2006-05-26 22:00:00 PM`

`4dd466d0e1e0cc123afbe504da2462ca`
