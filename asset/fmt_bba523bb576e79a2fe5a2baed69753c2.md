
```
:00402B9B  27F0FA                LitVar                 ;PushVar LOCAL\_0510          \\  
:00402B9E  2700FB                LitVar                 ;PushVar LOCAL\_0500          | Msgbox 函数参数  
:00402BA1  04CCFD                FLdRfVar               ;Push LOCAL\_0234 // 加载字符 |  
:00402BA4  04ACFD                FLdRfVar               ;Push LOCAL\_0254 // 加载字符 |  
:00402BA7  FB9410FB             AddVar                 ;// 字符叠加  
:00402BAB  F530000000           LitI4                  ;Push 00000030  
:00402BB0  044CFF                FLdRfVar               ;Push LOCAL\_00B4 // 加载字符 |  
:00402BB3  041CFD                FLdRfVar               ;Push LOCAL\_02E4 // 加载字符 |  
:00402BB6  FB94D4FC             AddVar                 ;// 字符叠加  
:00402BBA  04FCFC                FLdRfVar               ;Push LOCAL\_0304 // 加载字符 |  
:00402BBD  FB9480FC             AddVar                 ;// 字符叠加  
:00402BC1  04FCFE                FLdRfVar               ;Push LOCAL\_0104 // 加载字符 |  
:00402BC4  FB9460FC             AddVar                 ;// 字符叠加  
:00402BC8  040CFD                FLdRfVar               ;Push LOCAL\_02F4 // 加载字符 |  
:00402BCB  FB9440FC             AddVar                 ;// 字符叠加  
:00402BCF  04ECFE                FLdRfVar               ;Push LOCAL\_0114 // 加载字符 |  
:00402BD2  FB9430FC             AddVar                 ;// 字符叠加  
:00402BD6  048CFD                FLdRfVar               ;Push LOCAL\_0274 // 加载字符 |  
:00402BD9  FB9420FC             AddVar                 ;// 字符叠加  
:00402BDD  047CFD                FLdRfVar               ;Push LOCAL\_0284 // 加载字符 |  
:00402BE0  FB9410FC             AddVar                 ;// 字符叠加  
:00402BE4  04DCFD                FLdRfVar               ;Push LOCAL\_0224 // 加载字符 |  
:00402BE7  FB9400FC             AddVar                 ;// 字符叠加  
:00402BEB  04DCFE                FLdRfVar               ;Push LOCAL\_0124 // 加载字符 |  
:00402BEE  FB94F0FB             AddVar                 ;// 字符叠加  
:00402BF2  04ECFD                FLdRfVar               ;Push LOCAL\_0214 // 加载字符 |  
:00402BF5  FB94E0FB             AddVar                 ;// 字符叠加  
:00402BF9  049CFE                FLdRfVar               ;Push LOCAL\_0164 // 加载字符 |  
:00402BFC  FB94D0FB             AddVar                 ;// 字符叠加  
:00402C00  04BCFD                FLdRfVar               ;Push LOCAL\_0244 // 加载字符 |  
:00402C03  FB94C0FB             AddVar                 ;// 字符叠加  
:00402C07  049CFD                FLdRfVar               ;Push LOCAL\_0264 // 加载字符 |  
:00402C0A  FB94B0FB             AddVar                 ;// 字符叠加  
:00402C0E  046CFD                FLdRfVar               ;Push LOCAL\_0294 // 加载字符 |  
:00402C11  FB94A0FB             AddVar                 ;// 字符叠加  
:00402C15  044CFD                FLdRfVar               ;Push LOCAL\_02B4 // 加载字符 |  
:00402C18  FB9490FB             AddVar                 ;// 字符叠加  
:00402C1C  045CFD                FLdRfVar               ;Push LOCAL\_02A4 // 加载字符 |  
:00402C1F  FB9480FB             AddVar                 ;// 字符叠加  
:00402C23  043CFD                FLdRfVar               ;Push LOCAL\_02C4 // 加载字符 |  
:00402C26  FB9470FB             AddVar                 ;// 字符叠加  
:00402C2A  04ECFE                FLdRfVar               ;Push LOCAL\_0114 // 加载字符 |  
:00402C2D  FB9460FB             AddVar                 ;// 字符叠加  
:00402C31  042CFD                FLdRfVar               ;Push LOCAL\_02D4 // 加载字符 |  
:00402C34  FB9450FB             AddVar                 ;// 字符叠加  
:00402C38  042CFD                FLdRfVar               ;Push LOCAL\_02D4 // 加载字符 |  
:00402C3B  FB9440FB             AddVar                 ;// 字符叠加  
:00402C3F  042CFD                FLdRfVar               ;Push LOCAL\_02D4 // 加载字符 |  
:00402C42  FB9430FB             AddVar                 ;// 字符叠加  
:00402C46  04ECFE                FLdRfVar               ;Push LOCAL\_0114 // 加载字符 |  
:00402C49  FB9420FB             AddVar                 ;// 字符叠加                    /  
********************************************************Reference To->msvbvm60.rtcMsgBox  
                           |  
:00402C4D  0A28001400         ImpAdCallFPR4          ;Call ptr\_0040103C; check stack 0014; Push EAX  
:00402C52  363200D4FC80FC60   FFreeVar               ;Free 0032/2 variants  
:00402C87  FCC804             End                    ;// 结束程序  
:00402C8A  4C                 FnLBound               ;vbaLBound  
:00402C8B  FE04E4FC           ThisVCallR8            ;  
:00402C8F  21                 FLdPrThis              ;\[SR\]=\[stack2\]          \\  
:00402C90  0F0403             VCallAd                ;Return the control index 03 / 获得窗体句柄  
:00402C93  19E8FC             FStAdFunc              ;// 取 propget 过程地址  
:00402C96  08E8FC             FLdPr                  ;\[SR\]=\[LOCAL\_0318\] // 加载过程  
********************************************************Reference To:\[propget\]TextBox.Text  
                          |  
:00402C99  0DA0000000         VCallHresult           ;Call ptr\_004020FC // 获得文本框中的内容\[假码\]  
:00402C9E  3EE4FC             FLdZeroAd              ;Push DWORD \[LOCAL\_031C\]; \[LOCAL\_031C\]=0  
:00402CA1  46D4FC             CVarStr                ;// 将字符串类型改为变量  
:00402CA4  5D                 HardType               ;  
:00402CA5  FB33               EqVarBool              ;// 比较真码假码  
:00402CA7  1AE8FC             FFree1Ad               ;Push \[LOCAL\_0318\]; Call \[\[LOCAL\_0318\]\]+8\]; \[\[LOCAL\_0318\]\]=0   
:00402CAA  35D4FC             FFree1Var              ;Free LOCAL\_032C  
:00402CAD  1CDD03             BranchF                ;If Pop=0 then ESI=00402D5D //不等则跳 00402D5D -〉1CDD03 改为 10DD03  
:00402CB0  27E0FB             LitVar                 ;PushVar LOCAL\_0420          \\        即可以爆破！  
:00402CB3  27F0FB             LitVar                 ;PushVar LOCAL\_0410          | Msgbox 函数参数  
:00402CB6  043CFF             FLdRfVar               ;Push LOCAL\_00C4 // 加载字符 |  
:00402CB9  041CFF             FLdRfVar               ;Push LOCAL\_00E4 // 加载字符 |  
:00402CBC  FB9400FC           AddVar                 ;// 字符叠加  
:00402CC0  F540000000         LitI4                  ;Push 00000040  
:00402CC5  042CFE             FLdRfVar               ;Push LOCAL\_01D4 // 加载字符 |  
:00402CC8  04BCFE             FLdRfVar               ;Push LOCAL\_0144 // 加载字符 |  
:00402CCB  FB94D4FC           AddVar                 ;// 字符叠加  
:00402CCF  04ECFE             FLdRfVar               ;Push LOCAL\_0114 // 加载字符 |  
:00402CD2  FB9480FC           AddVar                 ;// 字符叠加  
:00402CD6  044CFF             FLdRfVar               ;Push LOCAL\_00B4 // 加载字符 |  
:00402CD9  FB9460FC           AddVar                 ;// 字符叠加  
:00402CDD  043CFF             FLdRfVar               ;Push LOCAL\_00C4 // 加载字符 |  
:00402CE0  FB9440FC           AddVar                 ;// 字符叠加  
:00402CE4  041CFF             FLdRfVar               ;Push LOCAL\_00E4 // 加载字符 |  
:00402CE7  FB9430FC           AddVar                 ;// 字符叠加  
:00402CEB  04FCFE             FLdRfVar               ;Push LOCAL\_0104 // 加载字符 |  
:00402CEE  FB9420FC           AddVar                 ;// 字符叠加  
:00402CF2  04ECFE             FLdRfVar               ;Push LOCAL\_0114 // 加载字符 |  
:00402CF5  FB9410FC           AddVar                 ;// 字符叠加                    /  
********************************************************Reference To->msvbvm60.rtcMsgBox  
                           |  
:00402CF9  0A28001400         ImpAdCallFPR4          ;Call ptr\_0040103C; check stack 0014; Push EAX  
:00402CFE  361400D4FC80FC60   FFreeVar               ;Free 0014/2 variants  
:00402D15  041CFE             FLdRfVar               ;Push LOCAL\_01E4 // 加载字符  
:00402D18  04FCFD             FLdRfVar               ;Push LOCAL\_0204 // 加载字符  
:00402D1B  FB94D4FC           AddVar                 ;// 字符叠加  
:00402D1F  040CFE             FLdRfVar               ;Push LOCAL\_01F4 // 加载字符  
:00402D22  FB9480FC           AddVar                 ;// 字符叠加  
:00402D26  FDFEE4FC           CStrVarVal             ;  
:00402D2A  21                 FLdPrThis              ;\[SR\]=\[stack2\]          \\  
:00402D2B  0F0003             VCallAd                ;Return the control index 02 / 获得窗体句柄
```
