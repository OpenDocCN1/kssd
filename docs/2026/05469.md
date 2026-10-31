
# 利用磁盘的序列号进行软件加密 (3千字)

> 发信人: beny (刺客 ), 信区: Software  
> 标  题: 利用磁盘的序列号进行软件加密  
> 发信站: 紫金飞鸿 (2001年03月16日19:03:42 星期五), 站内信件

用过共享软件的人都知道，一般的共享软件(特别是国外的)在使用一段时间后都会提出一些“苛刻”的要求，如让您输入注册号等等。如果您想在软件中实现该“功能”的话，方法有很多。在这里我介绍一种我认为安全性比较高的一种，仅供参考。

大家都知道，当您在命令行中键入`dir`指令后，系统都会读出一个称作Serial Number的十六进制数字。这个数字理论上有上亿种可能，而且很难同时找到两个序列号一样的硬盘。这就是我这种注册方法的理论依据，通过判断指定磁盘的序列号决定该机器的注册号。

要实现该功能，如何获得指定磁盘的序列号是最关键的。在Windows中，有一个`GetVolumeInformation`的API函数，我们利用这个函数就可以实现。

下面是实现该功能所需要的代码：

```vb
Private Declare Function GetVolumeInformation& Lib "kernel32" _
Alias "GetVolumeInformationA" (ByVal lpRootPathName As String, _
ByVal pVolumeNameBuffer As String, ByVal nVolumeNameSize As Long, _
lpVolumeSerialNumber As Long, lpMaximumComponentLength As Long, _
lpFileSystemFlags As Long, ByVal lpFileSystemNameBuffer As String, _
ByVal nFileSystemNameSize As Long)

Private Const MAX_FILENAME_LEN = 256

Public Function DriveSerial(ByVal sDrv As String) As Long
'Usage:
'Dim ds As Long
'ds = DriveSerial("C")
Dim RetVal As Long
Dim str As String * MAX_FILENAME_LEN
Dim str2 As String * MAX_FILENAME_LEN
Dim a As Long
Dim b As Long
GetVolumeInformation sDrv & ":\", str, MAX_FILENAME_LEN, RetVal, _
a, b, str2, MAX_FILENAME_LEN
DriveSerial = RetVal
End Function
```

如果我们需要某个磁盘的序列号的话，只要`DriverSerial`(该磁盘的盘符)即可。如`DriverASerialNumber=DriverSerial("A")`。

下面，我们就可以利用返回的磁盘序列号进行加密，需要用到一些数学知识。在这里我用了俄罗斯密码表的加密算法对进行了数学变换的序列号进行加密。下面是注册码验证部分的代码：

```vb
Public Function IsValidate(ByVal SRC As Long, ByVal Value As String) As Boolean
Dim SourceString As String
Dim NewSRC As Long
For i = 0 To 30
If (SRC And 2 ^ i) = 2 ^ i Then
SourceString = SourceString + "1"
Else
SourceString = SourceString + "0"
End If
Next i
If SRC < 0 Then
SourceString = SourceString + "1"
Else
SourceString = SourceString + "0"
End If
Dim Table As String
Dim TableIndex As Integer
'=======================================================================
'这是密码表，根据你的要求换成别的，不过长度要一致
'=======================================================================
'注意：这里的密码表变动后，对应的注册号生成器的密码表也要完全一致才能生成
正确的注册号
Table = "JSDJFKLUWRUOISDH;KSADJKLWQ;ABCDEFHIHL;KLADSHKJAGFWIHERQOWRLQH"
'=======================================================================
Dim Result As String
Dim MidWord As String
Dim MidWordValue As Byte
Dim ResultValue As Byte
For t = 1 To 1
For i = 1 To Len(SourceString)
MidWord = Mid(SourceString, i, 1)
MidWordValue = Asc(MidWord)
TableIndex = TableIndex + 1
If TableIndex > Len(Table) Then TableIndex = 1
ResultValue = Asc(Mid(Table, TableIndex, 1)) Mod MidWordValue
Result = Result + Hex(ResultValue)
Next i
SourceString = Result
Next t
Dim BitTORool As Integer
For t = 1 To Len(CStr(SRC))
BitTORool = SRC And 2 ^ t
For i = 1 To BitTORool
SourceString = Right(SourceString, 1) _
+ Left(SourceString, Len(SourceString) - 1)
Next i
Next t
If SourceString = Value Then IsValidate = True
End Function
```

由于代码较长，还有一些部分的代码在此省略，您可以去我的网站(http://vbtechnology.yeah.net)下载源程序研究一下。

最后，我们就可以利用这些子程序进行加密了。
