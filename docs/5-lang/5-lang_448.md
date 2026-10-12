
# VB PCODE 《万年历任你查 V3.09》注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-434.htm`](https://www.pediy.com/kssd/pediy07/pediy7-434.htm)

【软件名称】《万年历任你查》3.0.9

【原创作者】forever\[RCT\]

【目标语言】VB PCODE

【保护方式】注册码

【保护级别】简单

【使用工具】`PEiD`, `VBExplorer`

【目标简介】最全的电子万年历，有公历，农历，星期，节日，节气，。

【下载地址】http://www2.skycn.com/soft/10861.html

【正文】

这个软件功能很单一，用着也很方便。可惜要银子的。看看程序的编程语言，还是 vb pcode 的，晕。还好不是很复杂。用 `VBExplorer` 反编译出来，很容易找到下面这里。这是在您点击注册窗口的确定按钮时触发的事件：

```
[Command1.Click]

:0040B078  0474FF              FLdRfVar            ;Push LOCAL_008C //缓冲区
:0040B07B  21                  FLdPrThis           ;[SR]=[stack2]
:0040B07C  0F0C03              VCallAd             ;Return the control index 05
:0040B07F  1978FF              FStAdFunc
:0040B082  0878FF              FLdPr               ;[SR]=[LOCAL_0088]

***********Reference To:[propget]TextBox.Text
                      |
:0040B085  0DA0000800          VCallHresult        ;Call ptr_004046E0
:0040B08A  6C74FF              ILdRf               ;Push DWORD [LOCAL_008C] //取得注册码
:0040B08D  4A                  FnLenStr            ;vbaLenBstr               //取长度
:0040B08E  F508000000          LitI4               ;Push 00000008
:0040B093  CC                  NeI4                ;                         //比较注册码是否是 8 个字符
:0040B094  2F74FF              FFree1Str           ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=0
:0040B097  1A78FF              FFree1Ad            ;Push [LOCAL_0088]; Call [[[LOCAL_0088]]+8]; [[LOCAL_0088]]=0
:0040B09A  1C7800              BranchF             ;If Pop=0 then ESI=0040B0F0 //相等则跳走
:0040B09D  27F4FE              LitVar              ;PushVar LOCAL_010C       //这里要跳走，否则就提示位数不对了
:0040B0A0  2714FF              LitVar              ;PushVar LOCAL_00EC
:0040B0A3  2734FF              LitVar              ;PushVar LOCAL_00CC
:0040B0A6  F500000000          LitI4               ;Push 00000000

******Possible String Ref To->"胑覵蕆萵?瞖?籵?拭灿妒Ｈ亲植新驶任"
                      |
:0040B0AB  3A64FF0900          LitVarStr           ;PushVarString ptr_004046F4
:0040B0B0  4E54FF              FStVarCopyObj       ;[LOCAL_00AC]=vbaVarDup(Pop)
:0040B0B3  0454FF              FLdRfVar            ;Push LOCAL_00AC

*********Reference To->msvbvm60.rtcMsgBox
                      |
:0040B0B6  0A0A001400          ImpAdCallFPR4       ;Call ptr_00401084; check stack 0014; Push EAX
:0040B0BB  36080054FF34FF14     FFreeVar           ;Free 0008/2 variants

******Possible String Ref To->""
                      |
:0040B0C6  1B0B00              LitStr              ;Push ptr_00404510
:0040B0C9  21                  FLdPrThis           ;[SR]=[stack2]
:0040B0CA  0F0C03              VCallAd             ;Return the control index 05
:0040B0CD  1978FF              FStAdFunc
:0040B0D0  0878FF              FLdPr               ;[SR]=[LOCAL_0088]

**********Reference To:[propput]TextBox.Text
                      |
:0040B0D3  0DA4000800          VCallHresult        ;Call ptr_004046E0
:0040B0D8  1A78FF              FFree1Ad            ;Push [LOCAL_0088]; Call [[[LOCAL_0088]]+8]; [[LOCAL_0088]]=0
:0040B0DB  21                  FLdPrThis           ;[SR]=[stack2]
:0040B0DC  0F0C03              VCallAd             ;Return the control index 05
:0040B0DF  1978FF              FStAdFunc
:0040B0E2  0878FF              FLdPr               ;[SR]=[LOCAL_0088]

**********Reference To:TextBox.SetFocus
                      |
:0040B0E5  0D04020800          VCallHresult        ;Call ptr_004046E0
:0040B0EA  1A78FF              FFree1Ad            ;Push [LOCAL_0088]; Call [[[LOCAL_0088]]+8]; [[LOCAL_0088]]=0
:0040B0ED  1EF701              Branch              ;ESI=0040B26F
:0040B0F0  0474FF              FLdRfVar            ;Push LOCAL_008C   //注册码
:0040B0F3  21                  FLdPrThis           ;[SR]=[stack2]
:0040B0F4  0F0C03              VCallAd             ;Return the control index 05
:0040B0F7  1978FF              FStAdFunc
:0040B0FA  0878FF              FLdPr               ;[SR]=[LOCAL_0088]

**********Reference To:[propget]TextBox.Text
                      |
:0040B0FD  0DA0000800          VCallHresult        ;Call ptr_004046E0
:0040B102  04E8FE              FLdRfVar            ;Push LOCAL_0118   //缓冲区
:0040B105  21                  FLdPrThis           ;[SR]=[stack2]
:0040B106  0F0C03              VCallAd             ;Return the control index 05
:0040B109  19ECFE              FStAdFunc
:0040B10C  08ECFE              FLdPr               ;[SR]=[LOCAL_0114]

**********Reference To:[propget]TextBox.Text
                      |
:0040B10F  0DA0000800          VCallHresult        ;Call ptr_004046E0 //再次取得注册码
:0040B114  28D8FE0100          LitVarI2            ;PushVarInteger 0001 //长度
:0040B119  F504000000          LitI4               ;Push 00000004    //起始位置
:0040B11E  3EE8FE              FLdZeroAd           ;Push DWORD [LOCAL_0118]; [LOCAL_0118]=0 //注册码
:0040B121  46F4FE              CVarStr
:0040B124  04C8FE              FLdRfVar            ;Push LOCAL_0138  //结果

*********Reference To->msvbvm60.rtcMidCharVar                            //取第四个字符
                      |                                                 //local_138 = Mid(注册码,4,1)
:0040B127  0A0C001000          ImpAdCallFPR4       ;Call ptr_00401090; check stack 0010; Push EAX
:0040B12C  04C8FE              FLdRfVar            ;Push LOCAL_0138
:0040B12F  FDFEC4FE            CStrVarVal

*********Reference To->msvbvm60.rtcR8ValFromBstr                          //转换成数字
                      |
:0040B133  0A0D000400          ImpAdCallFPR4       ;Call ptr_004010AE; check stack 0004; Push EAX
:0040B138  7480FE              FStFPR8             ;Fstp#8 [LOCAL_0180]  //保存在 local_180
:0040B13B  04BCFE              FLdRfVar            ;Push LOCAL_0144  //再次取得注册码
:0040B13E  21                  FLdPrThis           ;[SR]=[stack2]
:0040B13F  0F0C03              VCallAd             ;Return the control index 05
:0040B142  19C0FE              FStAdFunc
:0040B145  08C0FE              FLdPr               ;[SR]=[LOCAL_0140]

**********Reference To:[propget]TextBox.Text
                      |
:0040B148  0DA0000800          VCallHresult        ;Call ptr_004046E0
:0040B14D  289CFE0100          LitVarI2            ;PushVarInteger 0001
:0040B152  F507000000          LitI4               ;Push 00000007
:0040B157  3EBCFE              FLdZeroAd           ;Push DWORD [LOCAL_0144]; [LOCAL_0144]=0
:0040B15A  46ACFE              CVarStr
:0040B15D  048CFE              FLdRfVar            ;Push LOCAL_0174

*********Reference To->msvbvm60.rtcMidCharVar                           //取第七个字符
                      |                                                //local_174 = Mid(注册码,7,1)
:0040B160  0A0C001000          ImpAdCallFPR4       ;Call ptr_00401090; check stack 0010; Push EAX
:0040B165  048CFE              FLdRfVar            ;Push LOCAL_0174
:0040B168  FDFE88FE            CStrVarVal

*********Reference To->msvbvm60.rtcR8ValFromBstr
                      |
:0040B16C  0A0D000400          ImpAdCallFPR4       ;Call ptr_004010AE; check stack 0004; Push EAX
```



:0040B171 7478FE FStFPR8 ;Fstp#8 \[LOCAL\_0188\] //转换成数字保存在 local\_188  

:0040B174 2834FF0100 LitVarI2 ;PushVarInteger 0001  

:0040B179 F502000000 LitI4 ;Push 00000002  

:0040B17E 3E74FF FLdZeroAd ;Push DWORD \[LOCAL\_008C\]; \[LOCAL\_008C\]=0  

:0040B181 4654FF CVarStr ;  

:0040B184 0414FF FLdRfVar ;Push LOCAL\_00EC  

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcMidCharVar  

 |  

:0040B187 0A0C001000 ImpAdCallFPR4 ;Call ptr\_00401090; check stack 0010; Push EAX  

:0040B18C 0414FF FLdRfVar ;Push LOCAL\_00EC //取得注册码第二个字符  

:0040B18F FDFEF0FE CStrVarVal ;  

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcR8ValFromBstr //转换成实数  

 |  

:0040B193 0A0D000400 ImpAdCallFPR4 ;Call ptr\_004010AE; check stack 0004; Push EAX  

:0040B198 6F80FE FLdFPR8 ;Fld#8 \[LOCAL\_0180\] //加上 local\_180  

:0040B19B AB AddR8 ;  

:0040B19C 6F78FE FLdFPR8 ;Fld#8 \[LOCAL\_0188\] //加上 local\_188  

:0040B19F AB AddR8 ;  

:0040B1A0 ED CR8R8 ;  

:0040B1A1 F412 LitI2\_Byte ;Push 12 //整数&H12 转换成实数  

:0040B1A3 EB CR8I2 ;  

:0040B1A4 C8 EqR4 ; //和上面的和比较  

:0040B1A5 320600F0FEC4FE88 FFreeStr ;Do SysFreeString \[arg\_n\]; \[arg\_n\]=0 0006/2 times ~ arg  

:0040B1AE 29060078FFECFEC0 FFreeAd ;  

:0040B1B7 36120054FF34FF14 FFreeVar ;Free 0012/2 variants  

:0040B1CC 1CA701 BranchF ;If Pop=0 then ESI=0040B21F //不等则跳去提示失败  

:0040B1CF 0474FF FLdRfVar ;Push LOCAL\_008C //相等则下面是保证注册信息  

:0040B1D2 0478FF FLdRfVar ;Push LOCAL\_0088  

:0040B1D5 050000 ImpAdLdRf ;Push ptr  

:0040B1D8 240100 NewIfNullPr ;\[Pop\] \[SR\]  

\*\*\*\*\*\*\*\*\*\*Reference To:Global.App  

 |  

:0040B1DB 0D14000200 VCallHresult ;Call ptr\_004042A8  

:0040B1E0 0878FF FLdPr ;\[SR\]=\[LOCAL\_0088\]  

\*\*\*\*\*\*\*\*\*\*Reference To:\[propget\]App.EXEName  

 |  

:0040B1E3 0D58000300 VCallHresult ;Call ptr\_00404628  

:0040B1E8 F400 LitI2\_Byte ;Push 00  

:0040B1EA FBFD CStrUI1 ;vbaStrI2  

:0040B1EC 23F0FE FStStrNoPop ;SysFreeString \[LOCAL\_0110\]; \[LOCAL\_0110\]=\[stack\]  

\*\*\*\*\*\*Possible String Ref To->"許苧??  

??藼蔈"  

 |  

:0040B1EF 1B0500 LitStr ;Push ptr\_00404518  

\*\*\*\*\*\*Possible String Ref To->"Options"  

 |  

:0040B1F2 1B0600 LitStr ;Push ptr\_0040463C  

:0040B1F5 6C74FF ILdRf ;Push DWORD \[LOCAL\_008C\]  

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcSaveSetting  

 |  

:0040B1F8 0A07001000 ImpAdCallFPR4 ;Call ptr\_00401078; check stack 0010; Push EAX  

:0040B1FD 32040074FFF0FE FFreeStr ;Do SysFreeString \[arg\_n\]; \[arg\_n\]=0 0004/2 times ~ arg  

:0040B204 1A78FF FFree1Ad ;Push \[LOCAL\_0088\]; Call \[\[\[LOCAL\_0088\]\]+8\]; \[\[LOCAL\_0088\]\]=0  

:0040B207 6C0800 ILdRf ;Push DWORD \[STACK\_0008\]  

:0040B20A FD9C78FF FStAdNoPop ;  

:0040B20E 050000 ImpAdLdRf ;Push ptr  

:0040B211 240100 NewIfNullPr ;\[Pop\] \[SR\]  

\*\*\*\*\*\*\*\*\*\*Reference To:Global.UnLoad  

 |  

:0040B214 0D10000200 VCallHresult ;Call ptr\_004042A8  

:0040B219 1A78FF FFree1Ad ;Push \[LOCAL\_0088\]; Call \[\[\[LOCAL\_0088\]\]+8\]; \[\[LOCAL\_0088\]\]=0  

:0040B21C 1EF701 Branch ;ESI=0040B26F  

:0040B21F 27F4FE LitVar ;PushVar LOCAL\_010C  

:0040B222 2714FF LitVar ;PushVar LOCAL\_00EC  

:0040B225 2734FF LitVar ;PushVar LOCAL\_00CC  

:0040B228 F500000000 LitI4 ;Push 00000000  

\*\*\*\*\*\*Possible String Ref To->"?睟????荈??????萀"  

 |  

:0040B22D 3A64FF0E00 LitVarStr ;PushVarString ptr\_00404720  

:0040B232 4E54FF FStVarCopyObj ;\[LOCAL\_00AC\]=vbaVarDup(Pop)  

:0040B235 0454FF FLdRfVar ;Push LOCAL\_00AC  

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcMsgBox  

 |  

:0040B238 0A0A001400 ImpAdCallFPR4 ;Call ptr\_00401084; check stack 0014; Push EAX  

:0040B23D 36080054FF34FF14 FFreeVar ;Free 0008/2 variants  

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  

可以看出验证注册的算法是非常简单的：取注册码第 2，4，7 位的字符，转换成数字，加在一起的和是 18 就可以了。  

【全文完】 

 4dd466d0e1e0cc123afbe504da2462ca
