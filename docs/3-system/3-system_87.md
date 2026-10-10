

# 获得 PE 资源树代码

> 原文：[`www.pediy.com/kssd/pediy10/80121.html`](https://www.pediy.com/kssd/pediy10/80121.html)

当时做一个软件的副产品，贴出来，供有需要的参考（VB 代码）。  
  

代码:

```
' PE 文件资源部分结构
Private Type IMAGE\_RESOURCE\_DIRECTORY
    Characteristics As Long
    TimeDateStamp As Long
    MajorVersion As Integer
    MinorVersion As Integer
    NumberOfNamedEntries As Integer
    NumberOfIdEntries As Integer
End Type
Private Type IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    Name As Long
    OffsetToData As Long
End Type
Private Type IMAGE\_RESOURCE\_DATA\_ENTRY
    OffsetToData As Long
    Size As Long
    CodePage As Long
    Reserved As Long
End Type

'资源数据结构
Private Type ResourceLeafInfo
    nTypeID As Long      '资源类型
    pStructRA As Long    '此资源的 IMAGE\_RESOURCE\_DATA\_ENTRY 结构相对地址（从资源段算起）
    pDataAA As Long      '此资源的绝对地址（从文件首部算起）
    cbSize As Long       '资源大小
End Type

'递归遍历资源树，获得叶节点相关数据
Private Sub PickLeaves(hFile As Long, \_
                       pResOffset As Long, \_
                       pNodeEntry As Long, \_
                       nTypeID As Long, \_
                       tResLeafInfo() As ResourceLeafInfo)
    
    Dim tResDir As IMAGE\_RESOURCE\_DIRECTORY
    Dim tResDirEntry As IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    Dim tResDataEntry As IMAGE\_RESOURCE\_DATA\_ENTRY
    Dim i As Long
    
    Call llseek(hFile, pResOffset + pNodeEntry, FILE\_BEGIN)
    Call lread(hFile, tResDir, Len(tResDir))
    For i = 0 To tResDir.NumberOfIdEntries + tResDir.NumberOfNamedEntries - 1
        Call llseek(hFile, pResOffset + pNodeEntry + Len(tResDir) + Len(tResDirEntry) \* i, FILE\_BEGIN)
        Call lread(hFile, tResDirEntry, Len(tResDirEntry))
        If CBool(tResDirEntry.OffsetToData And &H80000000) Then '指向下一个目录节点
            Call PickLeaves(hFile, pResOffset, tResDirEntry.OffsetToData And &H7FFFFFFF, nTypeID, tResLeafInfo())
        Else '指向数据入口
            Call llseek(hFile, pResOffset + tResDirEntry.OffsetToData, FILE\_BEGIN)
            Call lread(hFile, tResDataEntry, Len(tResDataEntry))
            ReDim Preserve tResLeafInfo(UBound(tResLeafInfo) + 1) As ResourceLeafInfo
            With tResLeafInfo(UBound(tResLeafInfo))
                .nTypeID = nTypeID
                .pStructRA = tResDirEntry.OffsetToData
                .pDataAA = tResDataEntry.OffsetToData
                .cbSize = tResDataEntry.Size
            End With
        End If
    Next i
End Sub

```