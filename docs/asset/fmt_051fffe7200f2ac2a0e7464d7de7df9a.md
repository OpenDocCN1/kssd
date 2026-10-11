
# VFP 结构学习

> 原文：[`www.pediy.com/kssd/pediy05/pediy50273.htm`](https://www.pediy.com/kssd/pediy05/pediy50273.htm)

VFP 结构学习小工具

====================================

原代码及编译后的文件：`http://www.foxlock.net/fox.zip`

```vb
'-----------------------------------------------------------------------------------------
' VFP-STRUCTURE Written By Aming Plindge Studio,1997-2002
'
' VFP 反编译的第一步，标准 APP 文件的识别，但愿此对 VFP 研究的朋友有一丝帮助
' 接触 Fox 是 1997 在 JNU 的时候，研究也是在 1997 的年底，不是 Fox，起码不会“认识”琳丫头，更不会
' 有今天的“品琳居”，呵呵。。
'
' Http://plindge.yeah.net Email:plindge@163.com
'-----------------------------------------------------------------------------------------

Option Explicit

Public Type VfpStructure
    aAppFileName As String                 '*// APP 文件的名称
    aIsApp As Boolean                       '*// 是否为 VFP 的 APP 文件
    aAppSize As Long                        '*// APP 文件的实际长度
    aFoxVersion As String                   '*// VFP 文件的版本
    aEncrypt As String                      '*// 文件是否加密编译
    aFileNumbers As Long                    '*// APP 中所包含文件的总数
    aFileListStartPos As Long               '*// 文件列表的开始位置
    aFileListEndPos As Long                 '*// 文件列表的结束位置
    aFileListSize As Long                   '*// 文件列表的长度
    aMainFileInFileList As Long             '*// 主文件在文件列表的序号
    aMainFileName As String                 '*// 主文件的名称
    aFilesName() As String                 '*// 所有文件的名称
    aFileNamePos() As Long                  '*// 所有文件各自名字在文件列表的偏移
    aFilesType() As Byte                    '*// 所有文件各自的类型
    aFilesStartPos() As Long                '*// 所有文件各自在 APP 的开始位置
    aFilesEndPos() As Long                  '*// 所有文件各自在 APP 的结束位置
    aFilesSize() As Long                    '*// 所有文件各自的长度
    aFoxChkSum As String                   '*// FOX 程序的检校 FoxChkSum
End Type

'--------------------------------------------------------------------------------------------
' - 对于 VFP6.0/7.0 的 APP 文件而言，其标志为 FE F2 FF[EE] 20 02
' - 00h-04h 为 VFP 的版本标识 其中 02h FF 表示为不加密编译，EE 表示加密编译，20 表明该文件为 VFP6/7
' - * 对 VFP 的反编译而言，第一步就是将加密的 APP 转成普通的 APP。
' - 05h-06h 文件的数目 07h-08h 主文件在文件列表中的序号
' - 09h-0Ch 文件列表的结束位置[EPOS] 0D-10h 文件列表的开始位置[SPOS] 11h-14h 文件列表长度
' - 15h-27h 保留 28h-29h 前 37 个字节的检校，进一步判断是否为 FOX 文件
'--------------------------------------------------------------------------------------------

Public Function GetFoxFile(aFile As String) As VfpStructure

    Dim aHeard(0 To 40) As Byte

    On Error GoTo MetErrors:

    GetFoxFile.aAppFileName = aFile

    Open aFile For Binary As #1

        Get #1, , aHeard()

        If aHeard(0) <> &HFE Or aHeard(1) <> &HF2 Then
            Close #1
            GetFoxFile.aIsApp = False
            Exit Function
        ElseIf aHeard(2) = &HFF Then
            GetFoxFile.aIsApp = True
            GetFoxFile.aEncrypt = "[不加密]"
            If aHeard(3) = &H1B Then
                GetFoxFile.aFoxVersion = "VFP 3.0"
            ElseIf aHeard(3) = &H1F Then
                GetFoxFile.aIsApp = True
                GetFoxFile.aFoxVersion = "VFP 5.0"
            ElseIf aHeard(3) = &H20 Then
                GetFoxFile.aIsApp = True
                GetFoxFile.aFoxVersion = "VFP 6.0/7.0"
            End If
        ElseIf aHeard(2) = &HEE Or aHeard(2) = &HFE Then
            GetFoxFile.aIsApp = True
            GetFoxFile.aEncrypt = "[加密]"
            Close #1
            Exit Function
        Else
            GetFoxFile.aIsApp = False
            Close #1
            Exit Function
        End If

    GetFoxFile.aFoxChkSum = Hex(aHeard(40)) & Hex(aHeard(39))      '// ChkSum 检校
    GetFoxFile.aFileNumbers = aHeard(5) + aHeard(6) * 16 ^ 2        '// 文件数目
    GetFoxFile.aMainFileInFileList = aHeard(7) + aHeard(8) * 16 ^ 2 '// 主文件在文件列表的序号

    Seek #1, 10
        Get #1, 10, GetFoxFile.aFileListEndPos '// 文件列表的结束地址

    Seek #1, 14
        Get #1, 14, GetFoxFile.aFileListStartPos '// 文件列表的开始地址

    Seek #1, 18
        Get #1, 18, GetFoxFile.aFileListSize '// 文件列表的长度，实际也 = 文件列表的结束地址 - 文件列表的开始地址

    ' 以下代码重新定义各数组大小
        ReDim GetFoxFile.aFilesName(0 To GetFoxFile.aFileNumbers - 1) As String    '// 文件名
        ReDim GetFoxFile.aFilesType(0 To UBound(GetFoxFile.aFilesName)) As Byte    '// 文件类型
        ReDim GetFoxFile.aFileNamePos(0 To UBound(GetFoxFile.aFilesName)) As Long  '// 文件名在列表中的位置
        ReDim GetFoxFile.aFilesStartPos(0 To UBound(GetFoxFile.aFilesName)) As Long '// 文件的开始地址
        ReDim GetFoxFile.aFilesEndPos(0 To UBound(GetFoxFile.aFilesName)) As Long  '// 文件的结束地址
        ReDim GetFoxFile.aFilesSize(0 To UBound(GetFoxFile.aFilesName)) As Long    '// 文件的实际长度也= 结束地址 - 开始地址

    ' 每个文件结构的存放空间是 25 字节，因此 APP 中，文件列表的结束位置开始算的话，
    ' 后面的部分刚好 = 文件数 * 25，在 EXE 则比标准 APP 多 14 字节，= 文件数 * 25 + 14
    ' 14 个字节来保存 APP 的长度及编译参数。

        Dim m As Long, n As Long

        For m = 0 To GetFoxFile.aFileNumbers - 1
            Seek #1, (GetFoxFile.aFileListEndPos + 1) + m * 25
                Get #1, (GetFoxFile.aFileListEndPos + 1) + m * 25, GetFoxFile.aFilesType(m)    '// 文件类型，00 表示代码，0X 表示数据
            Seek #1, (GetFoxFile.aFileListEndPos + 2) + m * 25
                Get #1, (GetFoxFile.aFileListEndPos + 2) + m * 25, GetFoxFile.aFilesStartPos(m) '// 每个文件的开始地址
            Seek #1, (GetFoxFile.aFileListEndPos + 2) + m * 25 + 12
                Get #1, (GetFoxFile.aFileListEndPos + 2) + m * 25 + 12, GetFoxFile.aFileNamePos(m) '// 每个文件名在文件列表的开始地址
        Next m

    ' 以下代码用于取每个文件的结束地址
        For m = 0 To UBound(GetFoxFile.aFilesEndPos) - 1
            GetFoxFile.aFilesEndPos(m) = GetFoxFile.aFilesStartPos(m + 1) - 1
        Next m

    ' 最后一个文件的结束地址当然是文件列表开始的前一字节啦
        GetFoxFile.aFilesEndPos(UBound(GetFoxFile.aFilesEndPos)) = GetFoxFile.aFileListStartPos - 1

    ' 以下代码用于取每个文件的实际长度
        For m = 0 To UBound(GetFoxFile.aFilesEndPos)
            GetFoxFile.aFilesSize(m) = GetFoxFile.aFilesEndPos(m) - GetFoxFile.aFilesStartPos(m)
        Next m

    ' 以下代码用于取文件列表的文件名，一个个地赋予文件名数组
        For m = 0 To UBound(GetFoxFile.aFilesName)
            GetFoxFile.aFilesName(m) = Space(255)
            Seek #1, GetFoxFile.aFileListStartPos + 1 + GetFoxFile.aFileNamePos(m)
            Get #1, GetFoxFile.aFileListStartPos + 1 + GetFoxFile.aFileNamePos(m), GetFoxFile.aFilesName(m)
        Next m

        Close #1 '// 关闭我们打开的文件

        GetFoxFile.aMainFileName = GetFoxFile.aFilesName(GetFoxFile.aMainFileInFileList)

        Exit Function

MetErrors:
        MsgBox "发现错误！", vbInformation + vbSystemModal, "信息提示"
        Close #1
        Exit Function

End Function

' -----------------------------------------------------------------------------
' - 将 VFP 的 EXE 文件转化为 APP 文件，APP 文件的实际长度=EXE 文件最后 4 个字节表示的长度
' - VFP 的 EXE = PE 头 + APP 文件
' -----------------------------------------------------------------------------

Public Sub Exe2App(sFile As String, dFile As String)

    Dim ExeSize  As Long             '// EXE 文件的长度
    Dim AppSize  As Long             '// APP 文件的长度
    Dim dData()  As Byte            '// 用于保存 APP 文件数据的数组

    ExeSize = FileLen(sFile)       '// 取 EXE 文件的长度

    Open sFile For Binary As #2

        Seek #2, ExeSize - 3
        Get #2, ExeSize - 3, AppSize '// 取得 APP 文件的实际长度
        ReDim dData(0 To AppSize - 1) As Byte '// 重新定义数组大小
        Seek #2, ExeSize - AppSize + 1        '// APP 文件的开始位置
        Get #2, , dData()

    Close #2

    Open dFile For Binary As #3

        Put #3, , dData()              '// 将数据写入 APP 文件

    Close #3

End Sub

'------------------------------------------------------------------------
'- aBuffer 是所取得的 FOX 结构类型，ID 是编号，dPath 是要分离到的目录
'- 算法设计：定义一个数组，大小=从当前编号的文件的大小，从当前编号的文件
'- 开始的地方读取大小相当的数据，赋予数组
'------------------------------------------------------------------------

Public Sub SplitFile(aBuffer As VfpStructure, ID As Long, dPath As String)

    Dim tData() As Byte
```
