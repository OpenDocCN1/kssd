

# HIVE 格式解析

> 原文：[`www.pediy.com/kssd/pediy10/59174.html`](https://www.pediy.com/kssd/pediy10/59174.html)

突然翻到这个东西，似乎之前没发过，就发来了。  
  

代码:

```
Attribute VB\_Name = "mHiveControl"

'By 炉子\[0GiNr\]
'http://hi.baidu.com/breakinglove\_
'http://0ginr.com

Option Explicit

Public Declare Function RegRestoreKey Lib "advapi32.dll" Alias "RegRestoreKeyA" ( \_
         ByVal hKey As Long, \_
         ByVal lpFile As String, \_
         ByVal dwFlags As RegRestoreFlags) As Long
Public Declare Function RegSaveKeyEx Lib "advapi32.dll" Alias "RegSaveKeyExA" ( \_
         ByVal hKey As Long, \_
         ByVal lpFile As String, \_
         ByVal lpSecurityAttributes As Long, \_
         ByVal dwFlags As RegSaveExFlags) As Long
Public Enum RegKeys
        HKEY\_CLASSES\_ROOT = &H80000000
        HKEY\_CURRENT\_USER = &H80000001
        HKEY\_LOCAL\_MACHINE = &H80000002
        HKEY\_USERS = &H80000003
        HKEY\_CURRENT\_CONFIG = &H80000005
End Enum
Public Enum RegRestoreFlags
        REG\_FORCE\_RESTORE = &H8
        REG\_WHOLE\_HIVE\_VOLATILE = &H1
End Enum
Public Enum RegSaveExFlags
        REG\_STANDARD\_FORMAT = &H1
        REG\_LATEST\_FORMAT = &H2
        REG\_NO\_COMPRESSION = &H4
End Enum
Public Type LARGE\_INTEGER
        LowPart As Long
        HighPart As Long
End Type
Public Type RegfBlock
        dwSignature As Long '字符串 - "regf" = 0x66676572
        dwUnknown1 As Long '未知
        dwUnknown2 As Long '总是为 0x00000004
        liLastEdit As LARGE\_INTEGER 'NT 时间格式
        dwNumber1 As Long '恒为 1
        dwNumber2 As Long '恒为 3
        dwNumber3 As Long '恒为 0
        dwNumber4 As Long '恒为 1 - 或许这个 1301 是版本 1.3.0.1?
        dwOffsetOfFirstKeyRecord As Long '第一个键纪录的偏移
        dwBlockSize As Long '数据块大小(文件大小-4kb)
        dwNumber5 As Long '恒为 1
        bytUnknownData(1 To &H1CC) As Byte '无需分析
        dwSum As Long '从 0x00000000 至 0x000001FB 的所有 DWORD 的数据总和
End Type

Public Type UnkownDataAfterRegfBlock '紧随 RegfBlock 之后
        bytReserved(1 To &HE00) As Byte '未知
End Type

Public Type HBinHeader
        dwSignature As Long '字符串 - "hbin" = 0x6E696268
        dwOffsetFromFirstHBinRecord As Long '第一个 Hbin 记录的偏移
        dwOffsetFromNextHBinRecord As Long '下一个 Hbin 记录的偏移
        dwUnknownData(1 To &H10) As Byte
        dwBlockSize As Long 'Hbin 记录长度
End Type

Public Type HBinData '如果这个段是一个负值(第 31 位被置 1)，则这个块是空的，并且长度被置为负的块大小
        dwDataBlockSize As Long
        szData() As Byte '长度取决于 dwDataBlockSize
End Type

Public Type NkRecord 'NameKey
        wSignature As Integer '字符串 - "nk" = 0x6B6E
        wKeyType As Integer '根键为 0x2C，否则为 0x20
        liLastEdit As LARGE\_INTEGER 'NT 时间格式
        bytUnknowData(1 To &H4) As Byte
        dwOffsetOfParentKey As Long '父键的偏移
        dwSubKeyNumber As Long '子键数目
        bytUnknowData2(1 To &H4) As Byte
        dwOffsetOfSubKeyLfRecords As Long '子键的 Lf 记录的偏移
        bytUnknowData3(1 To &H4) As Byte
        dwValuesNumber As Long '项的数目
        dwOffsetOfValueList As Long 'NkRecordValueList 的偏移
        dwOffsetOfSkRecord As Long 'Sk 记录的偏移
        dwOffsetOfClassName As Long '类名的偏移(???)
        bytUnused(1 To &H10) As Byte
        dwUnused As Long '无用数据
        wNameLength As Integer '项名长度
        wClassNameLength As Integer '类名的长度(???)
        szKeyName(1 To 1) As Byte '长度取决于 dwNameLength
End Type

Public Type NkRecordValueList
        dwValueOffset(1) As Long '数组数量取决于 dwValuesNumber
End Type

Public Type VkRecord 'ValueKey
        wSignature As Integer '字符串 - "vk" = 0x6B76
        wNameLength As Integer '项名长度
        dwDataLength As Long '数据长度 - 如果 dwDataLength <=4 那么这个值的数据就是该项的数据 (DWORD)；如果为 0 那么这个项无数据
        dwDataOffset As Long '数据偏移
        dwValueType As Long '数据类别 - 数据类别见 DataTypes
        wFlags As Integer '如果第 0 位被置 1，那么这条数据是有名称的，否则意味着这条数据是无名称的（“默认”）
        wUnused As Integer '无用数据
        szName(1 To 1) As Byte
End Type

Public Enum DataTypes
        REG\_SZ = &H1 '字符串 UNICODE
        REG\_EXPEND\_SZ = &H2 '可展开的字符串(使用环境变量，例如 "%SystemRoot%\\system32") UNICODE
        REG\_BINARY = &H3 '二进制数据
        REG\_DWORD = &H4 'DWORD
        REG\_MULTI\_SZ = &H7 '多个字符串，使用 vbNullChar 分隔 UNICODE
        REG\_UNKNOWN = &HFFFFFFFF
End Enum

Public Type HashRecord 'Lf 记录的哈希记录
        dwRecordOffset As Long '所属的 Lf 记录的偏移
        szKeyName(1 To 4) As Byte '键名的前 4 字节 如果修改了键名，这个也需要修改
End Type

Public Type LfRecord
        wSignature As Integer '字符串 - "lf" = 0x666C
        wKeyNumber As Integer '键的数目
        dwHashRecord(1 To 1) As HashRecord
End Type

Public Type SkRecord 'SecurityKey
        wSignature As Integer '字符串 - "sk" = 0x6B73
        wUnused As Integer
        dwPreviousSkRecordOffset As Long '前一个 Sk 记录的偏移
        dwNextSkRecordOffset As Long '后一个 Sk 记录的偏移
        dwUsageCounter As Long '使用计数 (???)
        dwRecordSize As Long 'Sk 记录的字节数
        '剩余部分为权限设置数据
End Type

Public Declare Function SafeCopyMemory \_
               Lib "NTDLL.DLL" Alias "ZwWriteVirtualMemory" \_
                               (ByVal ProcessHandle As Long, \_
                                ByVal pDest As Long, \_
                                ByVal pSrc As Long, \_
                                ByVal NumberOfBytesToCopy As Long, \_
                                ByRef NumberOfBytesCopied As Long) As Long
Public Const ZwGetCurrentProcess As Long \= -1 '//0xFFFFFFFF
Dim m\_pHive As Long
Dim m\_RegfBlock As RegfBlock, m\_HBinHeader As HBinHeader, m\_RootNkRecord As NkRecord
Dim m\_RaisedErr As Boolean
Dim m\_pRootNk As Long
Private Const GlobalOffset As Long \= &H1004
Private Const RegDefault As String \= "(Default)"

Public Sub dbg()
        Dim a As NkRecord
        Debug.Print Hex(LenB(a))
End Sub

'hHive should be the base of the hive file in memory
Public Function PreProcessHive(ByVal hHive As Long) As Boolean
        Dim st As Long
        Dim hBase As Long
        Dim unKnownData As UnkownDataAfterRegfBlock
        
        'save hive pointer
        m\_pHive = hHive
        
        hBase = hHive
        
        'read regf block
        st = CopyMemory(VarPtr(m\_RegfBlock), hBase, LenB(m\_RegfBlock))
        If (Not st) Then GoTo InitFaild\_
        
        'read hbin header
        hBase = hBase + LenB(m\_RegfBlock) + LenB(unKnownData)
        st = CopyMemory(VarPtr(m\_HBinHeader), hBase, LenB(m\_HBinHeader))
        If (Not st) Then GoTo InitFaild\_
        
        'read root nk header
        Dim HbData As HBinData
        st = CopyMemory(VarPtr(HbData), hBase + LenB(m\_HBinHeader), LenB(HbData))
        If (Not st) Then GoTo InitFaild\_
        hBase = hBase + GetHBinSize(HbData)
        st = CopyMemory(VarPtr(m\_RootNkRecord), hBase, LenB(m\_RootNkRecord))
        m\_pRootNk = hBase
        If (Not st) Then GoTo InitFaild\_
        
        'return
        PreProcessHive = True
        Exit Function
InitFaild\_:
        PreProcessHive = False
End Function

'这两段是测试用的。
Public Sub NOP()
        NOP1 (m\_pRootNk)
        
        DoEvents
End Sub

Public Sub NOP1(ByVal lpNk As Long)
        'MsgBox GetKeyNameByPointer(m\_pRootNk)
        Dim szReturn As String
        Dim I As Long, J As Long
        Dim nks() As Long
        Dim lfs() As Long
        lfs = GetSubKeyListNkPointers(lpNk)
        For I = LBound(lfs) To UBound(lfs)
                Dim vks() As Long
                vks = GetValueListVkPointers(lfs(I))
                szReturn = szReturn & "KeyName:" & GetKeyNameByPointer(lfs(I)) & vbCrLf
                If (Not m\_RaisedErr) Then
                        For J = LBound(vks) To UBound(vks)
                                If (vks(J)) = 0 Then Exit For
                                Dim dt As DataTypes
                                dt = GetValueTypeByPointer(vks(J))
                                Dim ret() As Byte
                                ret = GetValueDataByPointer(vks(J), dt)
                                If dt = REG\_DWORD Then
                                        Dim K As Long
                                        Call CopyMemory(VarPtr(K), VarPtr(ret(1)), 4)
                                        szReturn = szReturn & vbTab & GetValueNameByPointer(vks(J)) & vbTab & K & vbCrLf
                                ElseIf dt = REG\_SZ Then
                                        Dim szValue As String: szValue = ret
                                        szReturn = szReturn & vbTab & GetValueNameByPointer(vks(J)) & vbTab & szValue & vbCrLf
                                Else
                                        szReturn = szReturn & vbTab & GetValueNameByPointer(vks(J)) & vbTab & "(Unsupportted value type = " & dt & ")" & vbCrLf
                                End If
                                DoEvents
                        Next

                Else
                        ClearError
                End If
        Next
        DoEvents
        WriteFile App.Path & "\\Output.txt", StrConv(szReturn, vbFromUnicode)
End Sub

'get the size of the HBIN block
Private Function GetHBinSize(ByRef pHBinInfo As HBinData) As Long
        Dim HBHdrInfo As HBinHeader
        If (pHBinInfo.dwDataBlockSize And &H80000000) Then
                'the 31bit of 0x80000000 is 1, others are 0
                GetHBinSize = LenB(HBHdrInfo) + LenB(pHBinInfo.dwDataBlockSize)
                Exit Function
        Else
                GetHBinSize = LenB(HBHdrInfo) + LenB(pHBinInfo.dwDataBlockSize) + pHBinInfo.dwDataBlockSize
        End If
End Function

'get key name from NK record
Public Function GetKeyNameByPointer(ByVal pNkRecord As Long) As String
        Dim retByt() As Byte
        Dim szRetKeyName As String
        Dim Offset As Long
        Dim NkRec As NkRecord
        Dim st As Boolean
        ReDim pVkRec(1 To 1)
        st = CopyMemory(VarPtr(NkRec), pNkRecord, LenB(NkRec))
        If (Not st) Then GoTo ExitFunc\_
        With NkRec
                If (.wNameLength = 0) Then GoTo ExitFunc\_
                ReDim retByt(.wNameLength)
                Offset = VarPtr(.szKeyName(1)) - VarPtr(NkRec) + pNkRecord  ' - m\_pHive
                st = CopyMemory(VarPtr(retByt(LBound(retByt))), Offset, .wNameLength)
                If (Not st) Then GoTo ExitFunc\_
                szRetKeyName = StrConv(retByt, vbUnicode)
                If (InStr(szRetKeyName, vbNullChar)) Then szRetKeyName = Left(szRetKeyName, InStr(szRetKeyName, vbNullChar) - 1)
        End With
FinishFunc\_:
        Erase retByt
        GetKeyNameByPointer = szRetKeyName
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get key name from NK record
Public Function GetValueNameByPointer(ByVal pVkRecord As Long) As String
        Dim retByt() As Byte
        Dim szRetName As String
        Dim Offset As Long
        Dim VkRec As VkRecord
        Dim st As Boolean
        ReDim pVkRec(1 To 1)
        st = CopyMemory(VarPtr(VkRec), pVkRecord, LenB(VkRec))
        If (Not st) Then GoTo ExitFunc\_
        With VkRec
                If (.wNameLength = 0) Then szRetName = RegDefault: GoTo FinishFunc\_
                ReDim retByt(.wNameLength)
                Offset = VarPtr(.szName(1)) - VarPtr(VkRec) + pVkRecord  ' - m\_pHive
                st = CopyMemory(VarPtr(retByt(LBound(retByt))), Offset, .wNameLength)
                If (Not st) Then GoTo ExitFunc\_
                szRetName = StrConv(retByt, vbUnicode)
                If (InStr(szRetName, vbNullChar)) Then szRetName = Left(szRetName, InStr(szRetName, vbNullChar) - 1)
        End With
FinishFunc\_:
        Erase retByt
        GetValueNameByPointer = szRetName
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get value list, pNkRecord should be the NK record to list, return val is a array pointer to VK record
Public Function GetValueListVkPointers(ByVal pNkRecord As Long) As Long()
        Dim pVkRec() As Long
        Dim NkRec As NkRecord
        Dim dwNumber As Long
        Dim lOffset As Long
        Dim st As Boolean
        ReDim pVkRec(1 To 1)
        st = CopyMemory(VarPtr(NkRec), pNkRecord, LenB(NkRec))
        If (Not st) Then GoTo ExitFunc\_
        With NkRec
                dwNumber = .dwValuesNumber
                If (dwNumber = 0) Then GoTo ExitFunc\_
                ReDim pVkRec(1 To dwNumber)
                st = CopyMemory(VarPtr(lOffset), VarPtr(.dwOffsetOfValueList) - VarPtr(NkRec) + pNkRecord, LenB(lOffset))
                If (Not st) Then GoTo ExitFunc\_
                lOffset = lOffset + m\_pHive + GlobalOffset
                st = CopyMemory(VarPtr(pVkRec(LBound(pVkRec))), lOffset, dwNumber \* LenB(pVkRec(LBound(pVkRec))))
                If (Not st) Then GoTo ExitFunc\_
        End With
        
        'add offset to them :)
        Dim I As Long
        For I = LBound(pVkRec) To UBound(pVkRec)
                pVkRec(I) = pVkRec(I) + GlobalOffset + m\_pHive
        Next
FinishFunc\_:
        GetValueListVkPointers = pVkRec
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get sub-key list, pNkRecord should be the NK record to list, return val is a array pointer to HASH record
Public Function GetSubKeyListNkPointers(ByVal pNkRecord As Long) As Long()
        Dim pNkRec() As Long
        Dim NkRec As NkRecord
        Dim LfRec As LfRecord
        Dim HashRec() As HashRecord
        Dim st As Boolean
        ReDim pNkRec(1 To 1)
        st = CopyMemory(VarPtr(NkRec), pNkRecord, LenB(NkRec))
        If (Not st) Then GoTo ExitFunc\_
        With NkRec
                Dim dwNumber As Long
                Dim dwPosi As Long
                dwNumber = .dwSubKeyNumber
                If (dwNumber = 0) Then GoTo ExitFunc\_
                ReDim HashRec(1 To dwNumber)
                st = CopyMemory(VarPtr(dwPosi), VarPtr(.dwOffsetOfSubKeyLfRecords) - VarPtr(NkRec) + pNkRecord, LenB(dwPosi))
                If (Not st) Then GoTo ExitFunc\_
                dwPosi = dwPosi + GlobalOffset + m\_pHive
                st = CopyMemory(VarPtr(LfRec), dwPosi, LenB(LfRec))
                If (Not st) Then GoTo ExitFunc\_
                dwPosi = dwPosi + VarPtr(LfRec.dwHashRecord(1)) - VarPtr(LfRec)
                st = CopyMemory(VarPtr(HashRec(LBound(HashRec))), dwPosi, dwNumber \* LenB(HashRec(LBound(HashRec))))
                If (Not st) Then GoTo ExitFunc\_
        End With
        
        'calc NK record address
        Dim I As Long
        ReDim pNkRec(1 To dwNumber)
        For I = LBound(pNkRec) To UBound(pNkRec)
                pNkRec(I) = HashRec(I).dwRecordOffset + GlobalOffset + m\_pHive
        Next
FinishFunc\_:
        GetSubKeyListNkPointers = pNkRec
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get NK record pointer by HASH record
Public Function GetKeyNkRecordPointer(ByVal pHashRecord As Long) As Long
        Dim ret As Long
        Dim HashRec As HashRecord
        Dim st As Boolean
        st = CopyMemory(VarPtr(HashRec), pHashRecord, LenB(HashRec))
        If (Not st) Then GoTo ExitFunc\_
        With HashRec
                ret = .dwRecordOffset + GlobalOffset + m\_pHive
        End With
FinishFunc\_:
        GetKeyNkRecordPointer = ret
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get key value by VK record
Public Function GetValueDataByPointer(ByVal pVkRecord As Long, ByVal dwValueType As DataTypes) As Byte()
        Dim ret() As Byte
        Dim VkRec As VkRecord
        Dim lOffset As Long
        Dim st As Boolean
        ReDim ret(1 To 1)
        st = CopyMemory(VarPtr(VkRec), pVkRecord, LenB(VkRec))
        If (Not st) Then ReDim ret(1 To 1): GoTo ExitFunc\_
        With VkRec
                lOffset = .dwDataOffset + GlobalOffset + m\_pHive
                Select Case dwValueType
                        Case REG\_DWORD
                                ReDim ret(1 To LenB(.dwDataOffset))
                                        st = CopyMemory(VarPtr(ret(LBound(ret))), VarPtr(.dwDataOffset), UBound(ret) - LBound(ret) + 1)
                        Case REG\_SZ
                                If (.dwDataOffset = 0) Then GoTo ExitFunc\_
                                ReDim ret(1 To .dwDataLength)
                                        st = CopyMemory(VarPtr(ret(LBound(ret))), lOffset, UBound(ret) - LBound(ret) + 1)
                        Case Else
                                'unsupportted.
                                If (.dwDataOffset = 0) Then GoTo ExitFunc\_
                                ReDim ret(1 To .dwDataLength)
                                        st = CopyMemory(VarPtr(ret(LBound(ret))), lOffset, UBound(ret) - LBound(ret) + 1)

                End Select
                                
                'If (.dwDataOffset < 5) Then GoTo ExitFunc\_
                'lOffset = .dwDataOffset + GlobalOffset + m\_pHive
                'ReDim ret(1 To .dwDataLength)
                'st = CopyMemory(VarPtr(ret(LBound(ret))), lOffset, .dwDataLength)
                'If (Not st) Then ReDim ret(1 To 1): GoTo ExitFunc\_
        End With
FinishFunc\_:
        GetValueDataByPointer = ret
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

'get key value by VK record
Public Function GetValueTypeByPointer(ByVal pVkRecord As Long) As DataTypes
        Dim VkRec As VkRecord
        Dim st As Boolean
        Dim ret As DataTypes
        ret = REG\_UNKNOWN
        st = CopyMemory(VarPtr(VkRec), pVkRecord, LenB(VkRec))
        If (Not st) Then GoTo ExitFunc\_
        With VkRec
                ret = .dwValueType
        End With
FinishFunc\_:
        GetValueTypeByPointer = ret
        Exit Function
ExitFunc\_:
        m\_RaisedErr = True
        'Resume FinishFunc\_
End Function

Public Sub ClearError()
        m\_RaisedErr = False
End Sub

'return TRUE if the operation is successful
Public Function NT\_SUCCESS(ByVal Status As Long) As Boolean
        NT\_SUCCESS = (Status >= 0)
End Function

'copy data
Public Function CopyMemory(ByVal pDst As Long, ByVal pSrc As Long, ByVal nLength As Long) As Boolean
        Dim st As Long
        st = SafeCopyMemory(ZwGetCurrentProcess, pDst, pSrc, nLength, ByVal 0)
        CopyMemory = NT\_SUCCESS(st)
End Function


```