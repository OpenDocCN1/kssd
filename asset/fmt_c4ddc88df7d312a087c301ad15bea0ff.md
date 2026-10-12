
### 代码段 1：字符串长度检查

```
:00401C79  0F0003             VCallAd            ;Return the control index 02   
:00401C7C  1974FF             FStAdFunc          ;   
If Len(Text1.Text) >= 3 Then   
:00401C7F  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text       
:00401C82  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401C87  6C70FF             ILdRf              ;Push DWORD [LOCAL_0090]   
:00401C8A  4A                 FnLenStr     ; 输入字符串长度   
:00401C8B  F503000000         LitI4        ；常数 3 用于比较   
:00401C90  E0                 GeI4           ;   
:00401C91  2F70FF             FFree1Str          ;SysFreeString [LOCAL_0090];   
:00401C94  1A74FF             FFree1Ad           ;Push [LOCAL_008C];   
:00401C97  1C1002             BranchF      ；比较判断   
```

### 变量 `get_1` 赋值：获取第一个字符

`get_1` = `CInt(Mid(Text1.Text, 1, 1))`

```
:00401C9A  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401C9D  21                 FLdPrThis          ;[SR]=[stack2]   
:00401C9E  0F0003             VCallAd            ;Return the control index 02   
:00401CA1  1974FF             FStAdFunc          ;   
:00401CA4  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401CA7  0DA0000000         VCallHresult       ;   
:00401CAC  283CFF0100         LitVarI2        ; Push 1 参数，用于 MID 函数 取几个字符   
:00401CB1  F501000000         LitI4          ；Push 1 参数，用于 MID 函数   
                                              ；表示 从哪个字符位置开始取   
:00401CB6  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401CB9  465CFF             CVarStr            ;   
:00401CBC  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar      ；mid 函数调用   
                      |   
:00401CBF  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020; check stack 0010; Push EAX   
:00401CC4  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401CC7  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401CC9  441CFF             CVarI2             ;   
:00401CCC  FCF684FE           FStVar             ;   
:00401CD0  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401CD3  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_2` 赋值：获取第二个字符

`get_2` = `CInt(Mid(Text1.Text, 2, 1))`

```
:00401CDE  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401CE1  21                 FLdPrThis          ;[SR]=[stack2]   
:00401CE2  0F0003             VCallAd            ;Return the control index 02   
:00401CE5  1974FF             FStAdFunc          ;   
:00401CE8  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401CEB  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401CF0  283CFF0100         LitVarI2           ;PushVarInteger 0001   
:00401CF5  F502000000         LitI4              ;Push 00000002   
:00401CFA  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401CFD  465CFF             CVarStr            ;   
:00401D00  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar   
:00401D03  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020;   
:00401D08  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401D0B  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401D0D  441CFF             CVarI2             ;   
:00401D10  FCF674FE           FStVar             ;   
:00401D14  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401D17  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_3` 赋值：获取第三个字符

`get_3` = `CInt(Mid(Text1.Text, 3, 1))`

```
:00401D22  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401D25  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D26  0F0003             VCallAd            ;Return the control index 02   
:00401D29  1974FF             FStAdFunc          ;   
:00401D2C  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401D2F  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401D34  283CFF0100         LitVarI2           ;PushVarInteger 0001   
:00401D39  F503000000         LitI4              ;Push 00000003   
:00401D3E  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401D41  465CFF             CVarStr            ;   
:00401D44  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar   
:00401D47  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020; check stack 0010; Push EAX   
:00401D4C  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401D4F  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401D51  441CFF             CVarI2             ;   
:00401D54  FCF664FE           FStVar             ;   
:00401D58  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401D5B  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_4` 赋值：计算和后两位之和

`get_4` = `get_2` + `get_3`

```
:00401D66  0474FE             FLdRfVar       ;Push 变量 1 用于相加   
:00401D69  0464FE             FLdRfVar       ;Push 变量 2 用于相加   
:00401D6C  FB945CFF           AddVar           ;   
:00401D70  FCF654FE           FStVar           ;   
```

### 条件判断：验证输入

若 `get_1` = `get_4`，则执行后续逻辑。

```
:00401D74  0484FE             FLdRfVar       ;Push 变量 1 用于比较   
:00401D77  0454FE             FLdRfVar       ;Push 变量 2 用于比较   
:00401D7A  FB33               EqVarBool        ;   
:00401D7C  1CFB01             BranchF       ;比较判断结果   
******Possible String Ref To->"good"   
```

### 分支 1：输入正确

`Text2.Text` = `"good"`

```
:00401D7F  1B0500             LitStr           ;Push ptr_00401718   
:00401D82  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D83  0F0803             VCallAd            ;Return the control index 04   
:00401D86  1974FF             FStAdFunc          ;   
:00401D89  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propput]TextBox.Text   
:00401D8C  0DA4000000         VCallHresult       ;Call ptr_004016B8   
:00401D91  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
Else   
```

### 分支 2：输入错误

`Text2.Text` = `"bad"`

```
:00401D94  1E1002             Branch           ;ESI=00401DAC   
******Possible String Ref To->"bad"   
:00401D97  1B0600             LitStr           ;Push ptr_00401728   
:00401D9A  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D9B  0F0803             VCallAd            ;Return the control index 04   
:00401D9E  1974FF             FStAdFunc          ;   
:00401DA1  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propput]TextBox.Text   
:00401DA4  0DA4000000         VCallHresult       ;Call ptr_004016B8   
:00401DA9  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
```

### 程序退出

```
:00401DAC  13                 ExitProcHresult    ;   
:00401DAD  0000               LargeBos           ;IDE beginning of line   
```

> 注意：以上伪码中提示的 `VB` 源代码行，可能并不是精确的定位，仅是凭经验加入的，仅供参考。由于对 `P-CODE` 以及 `VBExplorer` 都不甚了解，许多伪代码的含义，以及其在 `VBExplorer` 中给出的注释，都不能明了个中的含义，还望了解者指点，多谢。

> 不论如何，根据 `P-CODE` 代码，我们可以大致分析出粗略的程序流程，变量的使用和定位并不一定精确，许多还要凭您的破解经验和编程经验。但是，这个粗略的分析可以帮助我们模拟出大致的 `VB` 源代码。
