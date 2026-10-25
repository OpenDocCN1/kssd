
# \[原创\]VB P-code 粗略分析(2)

> 原文：`www.pediy.com/kssd/pediy07/pediy7-55.htm`

前言：VB-Pcode 反编译文件的粗略分析,高手莫要笑话!

有了 Pcode 粗略分析(1)的分析，我们大概了解了一下 VB-Pcode 代码是怎样工作和参数调用的，下面在(1)的基础上再稍微加深分析一下。

附件:`Pcode(2-2).rar.rar)`

## 分析 1：(源文件)

```
=============================================

Private Sub Command1_Click()
  
  Dim name As String, code As String
  
  Dim i As Integer
  
  name = Text1.Text
  
  For i = 1 To Len(name)
    
    code = code & CStr(Asc(Mid(name, i, 1)))
  
  Next i
  
  Text2.Text = code

End Sub

=============================================

(P-Code)

=============================================

[Command1.Click]

:00401BD8  0468FF               FLdRfVar             ;Push LOCAL_0098 //开辟内存空间

:00401BDB  21                  FLdPrThis            ;[SR]=[stack2]               //和下句配套使用

:00401BDC  0F0403              VCallAd              ;Return the control index 03 //获得窗体句柄

:00401BDF  196CFF              FStAdFunc            ;//取 propget 过程地址

:00401BE2  086CFF              FLdPr                ;[SR]=[LOCAL_0094] //加载过程

**************Reference To:[propget]TextBox.Text //propget,TextBox.Text 的取过程

                              |

:00401BE5  0DA0000000          VCallHresult         ;Call ptr_004014A8 //获得文本框中的内容

:00401BEA  3E68FF              FLdZeroAd            ;Push DWORD [LOCAL_0098]; [LOCAL_0098]=0 //将内容入栈

:00401BED  3178FF              FStStr               ;SysFreeString [LOCAL_0088]; [LOCAL_0088]=Pop //将字符释放到 0088

:00401BF0  1A6CFF              FFree1Ad             ;Push [LOCAL_0094]; Call [[[LOCAL_0094]]+8]; [[LOCAL_0094]]=0 

:00401BF3  F401                LitI2_Byte           ;Push 01

:00401BF5  0472FF              FLdRfVar             ;Push LOCAL_008E //将文本框中的内容入栈

:00401BF8  6C78FF              ILdRf                ;Push DWORD [LOCAL_0088] //字符串入栈作为参数

:00401BFB  4A                  FnLenStr             ;vbaLenBstr //计算字符串长度

:00401BFC  E4                  CI2I4                ;Verify [stack] high word is 0000, ECX=[ECX] 

**************循环计算开始

:00401BFD  FE6364FF7200        ForI2                ;//For 运算

:00401C03  6C74FF              ILdRf                ;Push DWORD [LOCAL_008C] \\

:00401C06  2834FF0100          LitVarI2             ;PushVarInteger 0001      |

:00401C0B  6B72FF              FLdI2                ;Push WORD [LOCAL_008E]  | MID 函数参数入栈

:00401C0E  E7                  CI4UI1               ;                          |

:00401C0F  0478FF              FLdRfVar             ;Push LOCAL_0088          |

:00401C12  4D54FF0840          CVarRef              ;                          |

:00401C17  0424FF              FLdRfVar             ;Push LOCAL_00DC          /

**************Reference To->msvbvm60.rtcMidCharVar //MID

                              |

:00401C1A  0A01001000          ImpAdCallFPR4        ;Call ptr_00401030; check stack 0010; Push EAX //MID 取字符

:00401C1F  0424FF              FLdRfVar             ;Push LOCAL_00DC //取得字符入栈

:00401C22  FDFE68FF            CStrVarVal           ;

**************Reference To->msvbvm60.rtcAnsiValueBstr //ASC

                              |

:00401C26  0B02000400          ImpAdCallI2          ;Call ptr_00401036; check stack 0004; Push EAX //ASC 运算

:00401C2B  FBFD                CStrUI1              ;vbaStrI2 //将整数转换为字符

:00401C2D  2320FF              FStStrNoPop          ;SysFreeString [LOCAL_00E0]; [LOCAL_00E0]=[stack] //将字符释放

:00401C30  2A                  ConcatStr            ;vbaStrCat //连接字符串

:00401C31  3174FF              FStStr               ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=Pop //将字符释放

:00401C34  32040068FF20FF      FFreeStr             ;Do SysFreeString [arg_n]; [arg_n]=0 0004/2 times ~ arg 

:00401C3B  36040034FF24FF      FFreeVar             ;Free 0004/2 variants //释放变量

:00401C42  0472FF              FLdRfVar             ;Push LOCAL_008E //将文本框中的内容入栈

:00401C45  6464FF2B00          NextI2               ;

**************循环计算结束

:00401C4A  6C74FF              ILdRf                ;Push DWORD [LOCAL_008C]

:00401C4D  21                  FLdPrThis            ;[SR]=[stack2]               //和下句配套使用

:00401C4E  0F0003              VCallAd              ;Return the control index 02 //获得窗体句柄

:00401C51  196CFF              FStAdFunc            ;//取 propput 过程地址

:00401C54  086CFF              FLdPr                ;[SR]=[LOCAL_0094] //加载过程

**************Reference To:[propput]TextBox.Text //propput,TextBox.Text 的赋值过程

                              |

:00401C57  0DA4000000          VCallHresult         ;Call ptr_004014A8 //给 TextBox.Text 赋值

:00401C5C  1A6CFF              FFree1Ad             ;Push [LOCAL_0094]; Call [[[LOCAL_0094]]+8]; [[LOCAL_0094]]=0 

:00401C5F  13                  ExitProcHresult      ;//退出过程
```

在 1 的基础上我们在加个判断看看！

## 分析 2：(源文件)

```
=============================================

Private Sub Command1_Click()
  
  Dim name As String, code As String, T As String, F As String
  
  Dim i As Integer
  
  T = "True code!"
  
  F = "False code!"
  
  name = Text1.Text
  
  For i = 1 To Len(name)
    
    code = code & CStr(Asc(Mid(name, i, 1)))
  
  Next i
  
  If Text2.Text = code Then
    
    MsgBox T, vbOKOnly, "P-Code(2-2)"
  
  Else
    
    MsgBox F, vbOKOnly, "P-Code(2-2)"
  
  End If

End Sub

=============================================

(P-Code)

=============================================

[Command1.Click]

********Possible String Ref To->"True code!"

                              |

:00401C48  1B0000              LitStr               ;Push ptr_004016F4 //装入"True code!"字符

:00401C4B  4370FF              FStStrCopy           ;[LOCAL_0090]=SysAllocStringByteLen(Pop, [Pop-4]); SysFreeString Pop //复制到内存 0090

********Possible String Ref To->"False code!"

                              |

:00401C4E  1B0100              LitStr               ;Push ptr_00401710 //装入"False code!"字符

:00401C51  436CFF              FStStrCopy           ;[LOCAL_0094]=SysAllocStringByteLen(Pop, [Pop-4]); SysFreeString Pop //复制到内存 0094

:00401C54  0460FF              FLdRfVar             ;Push LOCAL_00A0 //开辟内存空间

:00401C57  21                  FLdPrThis            ;[SR]=[stack2]               //和下句配套使用

:00401C58  0F0403              VCallAd              ;Return the control index 03 //获得窗体句柄

:00401C5B  1964FF              FStAdFunc            ;//取 propget 过程地址

:00401C5E  0864FF              FLdPr                ;[SR]=[LOCAL_009C] //加载过程

**************Reference To:[propget]TextBox.Text //propget,TextBox.Text 的取过程

                              |

:00401C61  0DA0000200          VCallHresult         ;Call ptr_00401728 //获得文本框中的内容

:00401C66  3E60FF              FLdZeroAd            ;Push DWORD [LOCAL_00A0]; [LOCAL_00A0]=0 //将内容入栈

:00401C69  3178FF              FStStr               ;SysFreeString [LOCAL_0088]; [LOCAL_0088]=Pop //将字符释放到 0088

:00401C6C  1A64FF              FFree1Ad             ;Push [LOCAL_009C]; Call [[[LOCAL_009C]]+8]; [[LOCAL_009C]]=0 

:00401C6F  F401                LitI2_Byte           ;Push 01

:00401C71  046AFF              FLdRfVar             ;Push LOCAL_0096 //将文本框中的内容入栈

:00401C74  6C78FF              ILdRf                ;Push DWORD [LOCAL_0088] //字符串入栈作为参数

:00401C77  4A                  FnLenStr             ;vbaLenBstr //计算字符串长度

:00401C78  E4                  CI2I4                ;Verify [stack] high word is 0000, ECX=[ECX] 

**************循环计算开始

:00401C79  FE635CFF7E00        ForI2                ;//For 运算

:00401C7F  6C74FF              ILdRf                ;Push DWORD [LOCAL_008C] \\
```



这是一个汇编代码片段，展示了基于 `MID` 和 `ASC` 函数的字符串处理逻辑，以及后续的变量比较和消息框调用。以下是格式化后的代码内容：

```assembly
:00401C82  282CFF0100          LitVarI2             ;PushVarInteger 0001     |  

:00401C87  6B6AFF              FLdI2                ;Push WORD [LOCAL_0096]  |  

:00401C8A  E7                  CI4UI1               ;                        | MID 函数参数入栈  

:00401C8B  0478FF              FLdRfVar             ;Push LOCAL_0088         |  

:00401C8E  4D4CFF0840          CVarRef              ;                        |  

:00401C93  041CFF              FLdRfVar             ;Push LOCAL_00E4         /  

****************Reference To->msvbvm60.rtcMidCharVar //MID  

                   |  

:00401C96  0A03001000          ImpAdCallFPR4        ;Call ptr_00401030; check stack 0010; Push EAX //MID 取字符  

:00401C9B  041CFF              FLdRfVar             ;Push LOCAL_00E4 //取得字符入栈  

:00401C9E  FDFE60FF            CStrVarVal           ;  

****************Reference To->msvbvm60.rtcAnsiValueBstr  

                   |  

:00401CA2  0B04000400          ImpAdCallI2          ;Call ptr_00401036; check stack 0004; Push EAX //ASC 运算  

:00401CA7  FBFD                CStrUI1              ;vbaStrI2 //将整数转换为字符  

:00401CA9  2318FF              FStStrNoPop          ;SysFreeString [LOCAL_00E8]; [LOCAL_00E8]=[stack] //将字符释放  

:00401CAC  2A                  ConcatStr            ;vbaStrCat //连接字符串  

:00401CAD  3174FF              FStStr               ;SysFreeString [LOCAL_008C]; [LOCAL_008C]=Pop //将字符释放  

:00401CB0  32040060FF18FF      FFreeStr             ;Do SysFreeString [arg_n]; [arg_n]=0 0004/2 times ~ arg  

:00401CB7  3604002CFF1CFF      FFreeVar             ;Free 0004/2 variants //释放变量  

:00401CBE  046AFF              FLdRfVar             ;Push LOCAL_0096 //将文本框中的内容入栈  

:00401CC1  645CFF3700          NextI2               ;  

****************循环计算结束  

:00401CC6  0460FF              FLdRfVar             ;Push LOCAL_00A0 //将文本框 2 中的内容入栈  

:00401CC9  21                  FLdPrThis            ;[SR]=[stack2]            //和下句配套使用  

:00401CCA  0F0003              VCallAd              ;Return the control index 02 //获得窗体句柄  

:00401CCD  1964FF              FStAdFunc            ;//取 propput 过程地址  

:00401CD0  0864FF              FLdPr                ;[SR]=[LOCAL_009C] //加载过程  

****************Reference To:[propget]TextBox.Text //propget,TextBox.Text 的取过程  

                   |  

:00401CD3  0DA0000200          VCallHresult         ;Call ptr_00401728 //获得文本框中的内容  

:00401CD8  6C60FF              ILdRf                ;Push DWORD [LOCAL_00A0] //将文本框 2 中的内容入栈  

:00401CDB  6C74FF              ILdRf                ;Push DWORD [LOCAL_008C] //正确 code  

:00401CDE  FB30                EqStr                ;//字符串相等比较  

:00401CE0  2F60FF              FFree1Str            ;SysFreeString [LOCAL_00A0]; [LOCAL_00A0]=0  

:00401CE3  1A64FF              FFree1Ad             ;Push [LOCAL_009C]; Call [[[LOCAL_009C]]+8]; [[LOCAL_009C]]=0  

:00401CE6  1CD000              BranchF              ;If Pop=0 then ESI=00401D18 //不相等则跳  

:00401CE9  27E8FE              LitVar               ;PushVar LOCAL_0118        \\  

:00401CEC  271CFF              LitVar               ;PushVar LOCAL_00E4        |  

******Possible String Ref To->"P-Code(2-2)"                                    |  

                   |                                                     |  

:00401CEF  3A3CFF0500          LitVarStr            ;PushVarString ptr_0040173C  | MsgBox 函数参数入栈  

:00401CF4  4E2CFF              FStVarCopyObj        ;[LOCAL_00D4]=vbaVarDup(Pop) | 具体怎么看请找我的  

:00401CF7  042CFF              FLdRfVar             ;Push LOCAL_00D4           | Pcode 粗略分析(1)  

:00401CFA  F500000000          LitI4                ;Push 00000000             |  

:00401CFF  0470FF              FLdRfVar             ;Push LOCAL_0090           /  

:00401D02  4D4CFF0840          CVarRef              ;  

****************Reference To->msvbvm60.rtcMsgBox  

                   |  

:00401D07  0A06001400          ImpAdCallFPR4        ;Call ptr_0040103C; check stack 0014; Push EAX MsgBox  

:00401D0C  3606002CFF1CFFE8    FFreeVar             ;Free 0006/2 variants //释放变量  

:00401D15  1EFC00              Branch               ;ESI=00401D44 //跳转到 00401D44  

:00401D18  27E8FE              LitVar               ;PushVar LOCAL_0118        \\  

:00401D1B  271CFF              LitVar               ;PushVar LOCAL_00E4        |  

******Possible String Ref To->"P-Code(2-2)"                                    |  

                   |                                                     |  

:00401D1E  3A3CFF0500          LitVarStr            ;PushVarString ptr_0040173C  | MsgBox 函数参数入栈  

:00401D23  4E2CFF              FStVarCopyObj        ;[LOCAL_00D4]=vbaVarDup(Pop) |  

:00401D26  042CFF              FLdRfVar             ;Push LOCAL_00D4           |  

:00401D29  F500000000          LitI4                ;Push 00000000             |  

:00401D2E  046CFF              FLdRfVar             ;Push LOCAL_0094           /  

:00401D31  4D4CFF0840          CVarRef              ;  

****************Reference To->msvbvm60.rtcMsgBox  

                   |  

:00401D36  0A06001400          ImpAdCallFPR4        ;Call ptr_0040103C; check stack 0014; Push EAX //MsgBox  

:00401D3B  3606002CFF1CFFE8    FFreeVar             ;Free 0006/2 variants //释放内存变量  

:00401D44  13                  ExitProcHresult      ;//退出过程  

:00401D45  0000                LargeBos             ;IDE beginning of line with 00 byte codes  

                                                    Moodsky[DFCG]  

                                                      2005.02.03

4dd466d0e1e0cc123afbe504da2462ca
```

