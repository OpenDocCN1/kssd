
:00402D2E  19E8FC                FStAdFunc            ;// 取 `propget\[CommandButton.Caption\]` 过程地址  
:00402D31  08E8FC                FLdPr               ;`[SR]=[LOCAL_0318]` // 加载过程  

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propput\]CommandButton.Caption  

                         |  

:00402D34  0D54002900           VCallHresult         ;Call `ptr_00402238` // 改变 `Button.Caption`  
:00402D39  2FE4FC                FFree1Str            ;`SysFreeString [LOCAL_031C]; [LOCAL_031C]=0`  
:00402D3C  1AE8FC                FFree1Ad             ;`Push [LOCAL_0318]; Call [[LOCAL_0318]+8]; [LOCAL_0318]=0`  
:00402D3F  360400D4FC80FC        FFreeVar             ;Free `0004/2` variants  
:00402D46  F400                LitI2_Byte            ;Push `00`                     \\  
:00402D48  21                  FLdPrThis             ;`[SR]=[stack2]`              | Enabled 参数入栈  
:00402D49  0F0003              VCallAd               ;Return the control index `02` |  
:00402D4C  19E8FC                FStAdFunc            ;                             |  
:00402D4F  08E8FC                FLdPr               ;`[SR]=[LOCAL_0318]`           /  

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propput\]CommandButton.Enabled  

                         |  

:00402D52  0D8C002900           VCallHresult         ;Call `ptr_00402238` // 改变 `Button.Enabled`  
:00402D57  1AE8FC                FFree1Ad             ;`Push [LOCAL_0318]; Call [[LOCAL_0318]+8]; [LOCAL_0318]=0`  
:00402D5A  1E9904                Branch              ;`ESI=00402E19` //程序跳转  
:00402D5D  2790FB                LitVar              ;`PushVar LOCAL_0470`           \\  
:00402D60  27A0FB                LitVar              ;`PushVar LOCAL_0460`           | Msgbox 函数参数  
:00402D63  042CFF                FLdRfVar            ;`Push LOCAL_00D4` // 加载字符 |  
:00402D66  040CFF                FLdRfVar            ;`Push LOCAL_00F4` // 加载字符 |  
:00402D69  FB94B0FB              AddVar              ;// 字符叠加  
:00402D6D  F530000000            LitI4               ;Push `00000030`  
:00402D72  044CFF                FLdRfVar            ;`Push LOCAL_00B4` // 加载字符 |  
:00402D75  047CFE                FLdRfVar            ;`Push LOCAL_0184` // 加载字符 |  
:00402D78  FB94D4FC              AddVar              ;// 字符叠加  
:00402D7C  04CCFE                FLdRfVar            ;`Push LOCAL_0134` // 加载字符 |  
:00402D7F  FB9480FC              AddVar              ;// 字符叠加  
:00402D83  04ACFE                FLdRfVar            ;`Push LOCAL_0154` // 加载字符 |  
:00402D86  FB9460FC              AddVar              ;// 字符叠加  
:00402D8A  048CFE                FLdRfVar            ;`Push LOCAL_0174` // 加载字符 |  
:00402D8D  FB9440FC              AddVar              ;// 字符叠加  
:00402D91  049CFE                FLdRfVar            ;`Push LOCAL_0164` // 加载字符 |  
:00402D94  FB9430FC              AddVar              ;// 字符叠加  
:00402D98  045CFE                FLdRfVar            ;`Push LOCAL_01A4` // 加载字符 |  
:00402D9B  FB9420FC              AddVar              ;// 字符叠加  
:00402D9F  046CFE                FLdRfVar            ;`Push LOCAL_0194` // 加载字符 |  
:00402DA2  FB9410FC              AddVar              ;// 字符叠加  
:00402DA6  044CFF                FLdRfVar            ;`Push LOCAL_00B4` // 加载字符 |  
:00402DA9  FB9400FC              AddVar              ;// 字符叠加  
:00402DAD  042CFF                FLdRfVar            ;`Push LOCAL_00D4` // 加载字符 |  
:00402DB0  FB94F0FB              AddVar              ;// 字符叠加  
:00402DB4  040CFF                FLdRfVar            ;`Push LOCAL_00F4` // 加载字符 |  
:00402DB7  FB94E0FB              AddVar              ;// 字符叠加  
:00402DBB  04FCFE                FLdRfVar            ;`Push LOCAL_0104` // 加载字符 |  
:00402DBE  FB94D0FB              AddVar              ;// 字符叠加  
:00402DC2  04ECFE                FLdRfVar            ;`Push LOCAL_0114` // 加载字符 |  
:00402DC5  FB94C0FB              AddVar              ;// 字符叠加                     /  

\*\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcMsgBox  

                          |  

:00402DC9  0A28001400           ImpAdCallFPR4       ;Call `ptr_0040103C; check stack 0014; Push EAX`  
:00402DCE  361E00D4FC80FC60      FFreeVar             ;Free `001E/2` variants  

\*\*\*\*\*\*Possible String Ref To->""  

                          |  

:00402DEF  1B0100                LitStr              ;`Push ptr_00402110`  // 装入字符串变量  
:00402DF2  21                  FLdPrThis             ;`[SR]=[stack2]`              \\  
:00402DF3  0F1403              VCallAd               ;Return the control index `07` / 获得窗体句柄  
:00402DF6  19E8FC                FStAdFunc            ;// 取 `propput` 过程地址  
:00402DF9  08E8FC                FLdPr               ;`[SR]=[LOCAL_0318]` //加载过程  

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propput\]TextBox.Text  

                         |  

:00402DFC  0DA4000000           VCallHresult         ;Call `ptr_004020FC` // 写入  
:00402E01  1AE8FC                FFree1Ad             ;`Push [LOCAL_0318]; Call [[LOCAL_0318]+8]; [LOCAL_0318]=0`  

\*\*\*\*\*\*Possible String Ref To->""  

                          |  

:00402E04  1B0100                LitStr              ;`Push ptr_00402110`  // 装入字符串变量  
:00402E07  21                  FLdPrThis             ;`[SR]=[stack2]`              \\  
:00402E08  0F0403              VCallAd               ;Return the control index `03` / 获得窗体句柄  
:00402E0B  19E8FC                FStAdFunc            ;// 取 `propput` 过程地址  
:00402E0E  08E8FC                FLdPr               ;`[SR]=[LOCAL_0318]` //加载过程  

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propput\]TextBox.Text  

                         |  

:00402E11  0DA4000000           VCallHresult         ;Call `ptr_004020FC` // 写入  
:00402E16  1AE8FC                FFree1Ad             ;`Push [LOCAL_0318]; Call [[LOCAL_0318]+8]; [LOCAL_0318]=0`  
:00402E19  13                  ExitProcHresult      ;// 退出过程  
:00402E1A  0000                LargeBos             ;IDE beginning of line with `00` byte codes  

\[cmdExit.Click\]  

:004028A8  FCC813                End                 ;// 结束程序  

算法：`asc(mid(name,i,1)) * i mod &H1A * &H4EC6637`  

再次感谢 `MengLong` 兄弟的帮助！

Moodsky\[DFCG\]  

2005.02.18 

`4dd466d0e1e0cc123afbe504da2462ca`
