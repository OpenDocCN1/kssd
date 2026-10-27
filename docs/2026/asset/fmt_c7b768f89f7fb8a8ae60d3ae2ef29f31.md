
```
:00402B53  FBB480FC                        MulVar                     ;// 相乘：asc(mid(name,i,1))*i

:00402B57  2870FC1A00                      LitVarI2                   ;PushVarInteger 001A //装立即数&H1A

:00402B5C  FBA460FC                        ModVar                     ;// 取模 asc(mid(name,i,1)) * i mod &H1A

:00402B60  FEC150FC3766EC04                LitVarI4                   ;// &H4EC6637

:00402B68  FBB440FC                        MulVar                     ;// 相乘：asc(mid(name,i,1)) * i mod &H1A * &H4EC6637

:00402B6C  FCF65CFF                        FStVar                     ;// 感谢MengLong兄的帮助才找到这个I4

:00402B70  320400E4FC90FC                  FFreeStr                   ;Do SysFreeString [arg_n]; [arg_n]=0 0004/2 times ~ arg

:00402B77  1AE8FC                          FFree1Ad                   ;Push [LOCAL_0318]; Call [[[LOCAL_0318]]+8]; [[LOCAL_0318]]=0

:00402B7A  35D4FC                          FFree1Var                  ;Free LOCAL_032C

:00402B7D  04ECFC                          FLdRfVar                   ;Push LOCAL_0314

:00402B80  FE7E94FCA101                    NextStepVar                ;// NEXT

:00402B86  045CFF                          FLdRfVar                   ;Push LOCAL_00A4 // ""字符入栈

:00402B89  FD004CFE                        FStVarCopy                 ;[LOCAL_01B4]=vbaVarCopy(Pop)

:00402B8D  043CFE                          FLdRfVar                   ;Push LOCAL_01C4 // 用户名入栈

:00402B90  28C4FC0000                      LitVarI2                   ;PushVarInteger 0000

:00402B95  5D                              HardType                   ;

:00402B96  FB33                            EqVarBool                  ;// 用户名是否等于""

:00402B98  1C0903                          BranchF                    ;If Pop=0 then ESI=00402C89 //不等于则跳

:00402B9B  27F0FA                          LitVar                     ;PushVar LOCAL_0510          \
:00402B9E  2700FB                          LitVar                     ;PushVar LOCAL_0500          | Msgbox函数参数
:00402BA1  04CCFD                          FLdRfVar                   ;Push LOCAL_0234 // 加载字符 |
:00402BA4  04ACFD                          FLdRfVar                   ;Push LOCAL_0254 // 加载字符 |
:00402BA7  FB9410FB                        AddVar                     ;// 字符叠加
:00402BAB  F530000000                      LitI4                      ;Push 00000030
:00402BB0  044CFF                          FLdRfVar                   ;Push LOCAL_00B4 // 加载字符 |
:00402BB3  041CFD                          FLdRfVar                   ;Push LOCAL_02E4 // 加载字符 |
:00402BB6  FB94D4FC                        AddVar                     ;// 字符叠加
:00402BBA  04FCFC                          FLdRfVar                   ;Push LOCAL_0304 // 加载字符 |
:00402BBD  FB9480FC                        AddVar                     ;// 字符叠加
:00402BC1  04FCFE                          FLdRfVar                   ;Push LOCAL_0104 // 加载字符 |
:00402BC4  FB9460FC                        AddVar                     ;// 字符叠加
:00402BC8  040CFD                          FLdRfVar                   ;Push LOCAL_02F4 // 加载字符 |
:00402BCB  FB9440FC                        AddVar                     ;// 字符叠加
:00402BCF  04ECFE                          FLdRfVar                   ;Push LOCAL_0114 // 加载字符 |
:00402BD2  FB9430FC                        AddVar                     ;// 字符叠加
:00402BD6  048CFD                          FLdRfVar                   ;Push LOCAL_0274 // 加载字符 |
:00402BD9  FB9420FC                        AddVar                     ;// 字符叠加
:00402BDD  047CFD                          FLdRfVar                   ;Push LOCAL_0284 // 加载字符 |
:00402BE0  FB9410FC                        AddVar                     ;// 字符叠加
:00402BE4  04DCFD                          FLdRfVar                   ;Push LOCAL_0224 // 加载字符 |
:00402BE7  FB9400FC                        AddVar                     ;// 字符叠加
:00402BEB  04DCFE                          FLdRfVar                   ;Push LOCAL_0124 // 加载字符 |
:00402BEE  FB94F0FB                        AddVar                     ;// 字符叠加
:00402BF2  04ECFD                          FLdRfVar                   ;Push LOCAL_0214 // 加载字符 |
:00402BF5  FB94E0FB                        AddVar                     ;// 字符叠加
:00402BF9  049CFE                          FLdRfVar                   ;Push LOCAL_0164 // 加载字符 |
:00402BFC  FB94D0FB                        AddVar                     ;// 字符叠加
:00402C00  04BCFD                          FLdRfVar                   ;Push LOCAL_0244 // 加载字符 |
:00402C03  FB94C0FB                        AddVar                     ;// 字符叠加
:00402C07  049CFD                          FLdRfVar                   ;Push LOCAL_0264 // 加载字符 |
:00402C0A  FB94B0FB                        AddVar                     ;// 字符叠加
:00402C0E  046CFD                          FLdRfVar                   ;Push LOCAL_0294 // 加载字符 |
:00402C11  FB94A0FB                        AddVar                     ;// 字符叠加
:00402C15  044CFD                          FLdRfVar                   ;Push LOCAL_02B4 // 加载字符 |
:00402C18  FB9490FB                        AddVar                     ;// 字符叠加
:00402C1C  045CFD                          FLdRfVar                   ;Push LOCAL_02A4 // 加载字符 |
:00402C1F  FB9480FB                        AddVar                     ;// 字符叠加
:00402C23  043CFD                          FLdRfVar                   ;Push LOCAL_02C4 // 加载字符 |
:00402C26  FB9470FB                        AddVar                     ;// 字符叠加
:00402C2A  04ECFE                          FLdRfVar                   ;Push LOCAL_0114 // 加载字符 |
:00402C2D  FB9460FB                        AddVar                     ;// 字符叠加
:00402C31  042CFD                          FLdRfVar                   ;Push LOCAL_02D4 // 加载字符 |
:00402C34  FB9450FB                        AddVar                     ;// 字符叠加
:00402C38  042CFD                          FLdRfVar                   ;Push LOCAL_02D4 // 加载字符 |
:00402C3B  FB9440FB                        AddVar                     ;// 字符叠加
:00402C3F  042CFD                          FLdRfVar                   ;Push LOCAL_02D4 // 加载字符 |
:00402C42  FB9430FB                        AddVar                     ;// 字符叠加
:00402C46  04ECFE                          FLdRfVar                   ;Push LOCAL_0114 // 加载字符 |
:00402C49  FB9420FB                        AddVar                     ;// 字符叠加                 /

****Reference To->msvbvm60.rtcMsgBox

                               |

:00402C4D  0A28001400                      ImpAdCallFPR4              ;Call ptr_0040103C; check stack 0014; Push EAX

:00402C52  363200D4FC80FC60                FFreeVar                   ;Free 0032/2 variants

:00402C87  FCC804                          End                        ;// 结束程序

:00402C8A  4C                              FnLBound                   ;vbaLBound

:00402C8B  FE04E4FC                        ThisVCallR8                ;

:00402C8F  21                              FLdPrThis                  ;[SR]=[stack2]               \
:00402C90  0F0403                          VCallAd                    ;Return the control index 03 / 获得窗体句柄

:00402C93  19E8FC                          FStAdFunc                  ;// 取propget过程地址

:00402C96  08E8FC                          FLdPr                      ;[SR]=[LOCAL_0318] // 加载过程

****Reference To:[propget]TextBox.Text

                              |

:00402C99  0DA0000000                      VCallHresult               ;Call ptr_004020FC // 获得文本框中的内容[假码]

:00402C9E  3EE4FC                          FLdZeroAd                  ;Push DWORD [LOCAL_031C]; [LOCAL_031C]=0

:00402CA1  46D4FC                          CVarStr                    ;// 将字符串类型改为变量

:00402CA4  5D                              HardType                   ;

:00402CA5  FB33                            EqVarBool                  ;// 比较真码假码
```
