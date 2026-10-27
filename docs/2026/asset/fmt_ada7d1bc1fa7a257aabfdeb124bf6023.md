
`:004030D2 FDFED4FE CStrVarVal ; / // ASC函数参数入栈`

`**Reference To->msvbvm60.rtcAnsiValueBstr**

|`

`:004030D6 0B04000400 ImpAdCallI2 ;Call ptr_00401030; check stack 0004; Push EAX //ASC()`

`:004030DB F42F LitI2_Byte ;Push 2F // 47入栈`

`:004030DD D5 LeI2 ;// 小于等于比较操作`

`:004030DE C4 AndI4 ;// AND`

`:004030DF 3204005CFFD4FE FFreeStr ;Do SysFreeString [arg_n]; [arg_n]=0 0004/2 times ~ arg`

`:004030E6 36080028FF18FFE8 FFreeVar ;Free 0008/2 variants// 释放临时变量`

`:004030F1 1C2609 BranchF ;If Pop=0 then ESI=0040310A //条件为假则跳0040310A`

`**Possible String Ref To->""**

|`

`:004030F4 1B0200 LitStr ;Push ptr_00402228 // NULL字符入栈`

`:004030F7 21 FLdPrThis ;[SR]=[stack2] \`

`:004030F8 0F0403 VCallAd ;Return the control index 03 / // 获得窗体句柄`

`:004030FB 1960FF FStAdFunc ;// 取propput过程地址`

`:004030FE 0860FF FLdPr ;[SR]=[LOCAL_00A0] //加载过程`

`**Reference To:[propput]TextBox.Text // propput,TextBox.Text的赋值过程**

|`

`:00403101 0DA4000300 VCallHresult ;Call ptr_0040222C // 给TextBox.Text赋值`

`:00403106 1A60FF FFree1Ad ;Push [LOCAL_00A0]; Call [[[LOCAL_00A0]]+8]; [[LOCAL_00A0]]=0 // 释放加载过程空间`

`:00403109 13 ExitProcHresult ;// 退出过程`

```
If Asc(Mid(name, i, 1)) >= 0 And Asc(Mid(name, i, 1)) <= 47 Then
  Text1.Text = ""
  Exit Sub
End If
```

`:0040310A 2828FF0100 LitVarI2 ;PushVarInteger 0001 \`

`:0040310F 6B66FF FLdI2 ;Push WORD [LOCAL_009A] | // MID函数参数入栈`

`:00403112 E7 CI4UI1 ; |`

`:00403113 0478FF FLdRfVar ;Push LOCAL_0088 /`

`:00403116 4D48FF0840 CVarRef ;// 创建临时变量`

`:0040311B 0418FF FLdRfVar ;Push LOCAL_00E8 // 加载临时变量`

`**Reference To->msvbvm60.rtcMidCharVar**

|`

`:0040311E 0A05001000 ImpAdCallFPR4 ;Call ptr_00401036; check stack 0010; Push EAX // MID()`

`:00403123 0418FF FLdRfVar ;Push LOCAL_00E8 \`

`:00403126 FDFE5CFF CStrVarVal ; / // ASC函数参数入栈`

`**Reference To->msvbvm60.rtcAnsiValueBstr**

|`

`:0040312A 0B04000400 ImpAdCallI2 ;Call ptr_00401030; check stack 0004; Push EAX // ASC()`

`:0040312F F43A LitI2_Byte ;Push 3A // 58入栈`

`:00403131 DF GeI2 ;// 大于等于比较操作`

`:00403132 28E8FE0100 LitVarI2 ;PushVarInteger 0001 \`

`:00403137 6B66FF FLdI2 ;Push WORD [LOCAL_009A] | // MID函数参数入栈`

`:0040313A E7 CI4UI1 ; |`

`:0040313B 0478FF FLdRfVar ;Push LOCAL_0088 /`

`:0040313E 4D08FF0840 CVarRef ;// 创建临时变量`

`:00403143 04D8FE FLdRfVar ;Push LOCAL_0128 // 加载临时变量`

`**Reference To->msvbvm60.rtcMidCharVar**

|`

`:00403146 0A05001000 ImpAdCallFPR4 ;Call ptr_00401036; check stack 0010; Push EAX //MID()`

`:0040314B 04D8FE FLdRfVar ;Push LOCAL_0128 \`

`:0040314E FDFED4FE CStrVarVal ; / // ASC函数参数入栈`

`**Reference To->msvbvm60.rtcAnsiValueBstr**

|`

`:00403152 0B04000400 ImpAdCallI2 ;Call ptr_00401030; check stack 0004; Push EAX //ASC()`

`:00403157 F440 LitI2_Byte ;Push 40 // 64入栈`

`:00403159 D5 LeI2 ;// 小于等于比较操作`

`:0040315A C4 AndI4 ;// AND`

`:0040315B 3204005CFFD4FE FFreeStr ;Do SysFreeString [arg_n]; [arg_n]=0 0004/2 times ~ arg`

`:00403162 36080028FF18FFE8 FFreeVar ;Free 0008/2 variants // 释放临时变量`

`:0040316D 1CA209 BranchF ;If Pop=0 then ESI=00403186 //条件为假则跳00403186`

`**Possible String Ref To->""**

|`

`:00403170 1B0200 LitStr ;Push ptr_00402228 // NULL字符入栈`

`:00403173 21 FLdPrThis ;[SR]=[stack2] \`

`:00403174 0F0403 VCallAd ;Return the control index 03 / // 获得窗体句柄`

`:00403177 1960FF FStAdFunc ;// 取propput过程地址`

`:0040317A 0860FF FLdPr ;[SR]=[LOCAL_00A0] //加载过程`

`**Reference To:[propput]TextBox.Text // propput,TextBox.Text的赋值过程**

|`

`:0040317D 0DA4000300 VCallHresult ;Call ptr_0040222C // 给TextBox.Text赋值`

`:00403182 1A60FF FFree1Ad ;Push [LOCAL_00A0]; Call [[[LOCAL_00A0]]+8]; [[LOCAL_00A0]]=0 // 释放加载过程空间`

`:00403185 13 ExitProcHresult ;// 退出过程`

```
If Asc(Mid(name, i, 1)) >= 58 And Asc(Mid(name, i, 1)) <= 64 Then
  Text1.Text = ""
  Exit Sub
End If
```

### 下面和上面的两个过程是一样的判断NAME中是否是字母或字符不是则给TEXT赋空值退出过程

`:00403186 2828FF0100 LitVarI2 ;PushVarInteger 0001`

`:0040318B 6B66FF FLdI2 ;Push WORD [LOCAL_009A]`

`:0040318E E7 CI4UI1 ;`

`:0040318F 0478FF FLdRfVar ;Push LOCAL_0088`

`:00403192 4D48FF0840 CVarRef ;`

`:00403197 0418FF FLdRfVar ;Push LOCAL_00E8`

`**Reference To->msvbvm60.rtcMidCharVar**

|`

`:0040319A 0A05001000 ImpAdCallFPR4 ;Call ptr_00401036; check stack 0010; Push EAX`

`:0040319F 0418FF FLdRfVar ;Push LOCAL_00E8`
