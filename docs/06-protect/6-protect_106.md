
# 利用脚本语言 VBS 识别加密锁具体型号

> 原文：`[www.pediy.com/kssd/pediy12/130031.html](https://www.pediy.com/kssd/pediy12/130031.html)`

利用脚本语言 VBS 识别加密锁具体型号。

VBS(VBScript) 是基于 Visual Basic 的脚本语言，直接由宿主(host) 解释源代码并执行，不需要编译成 `.exe`。所以不用担心被人植入木马，还可以直接用记事本修改源码，保存执行就行。

程序功能：识别加密锁的具体型号，使用者自己可以添加、修改加密锁型号识别标志（PID，VID），程序用简单易学的 `vbs` 编写，方便大家完美程序代码，完善程序功能。

查看设备 PID，VID 的方法：

桌面-我的电脑-右键-属性-硬件-设备管理器-人体学输入设备-选择一个 `HID-compliant device`-右键属性-详细信息，如下图：

![](img/bf8a6462e33dafdaf556db7c76bd4ad0.jpg)

图中 `VID_3689&PID_8762` 就是域天加密锁简单型的厂商标识和产品标识

## 编辑识别加密锁的 XML 文件

用记事本打开 `data.xml`，修改或者添加加密锁的标识，如域天加密简单型的（yt88,密码型,等）。

Data.xml 的例子文件如下：

```xml
<?xml version="1.0" encoding="utf-8"?>

<XMLobject>

  <page>

    <!-- 加密锁的 vid -->

    <hid><![CDATA[VID_3689&PID_8762]]></hid>

    <!-- 加密锁的型号 -->

    <type>域之天密码型加密锁</type>

    <!-- 相关链接 -->

    <url>www.money988.com</url>

    <fuck>硬件复制，软件模拟</fuck>

  </page>

  <page>

    <!-- 狗的 vid -->

    <hid><![CDATA[VID_0925&PID_6B05]]></hid>

    <!-- 狗的型号 -->

    <type>世纪龙脉 x2 加密锁</type>

    <!-- 相关链接 -->

    <url>site_configuration/intro.xml</url>

    <fuck>硬件复制，软件模拟</fuck>

  </page>

</XMLobject>
```

## VBS 加密锁类型查询程序源码

```vbs
On Error Resume Next

Dim n,h,s

Dim xmlDoc,xmlRoot,xmlNode,lastNode,newNode

Dim doc,docRoot,i,flag

Dim strNodeName,strPath,docPath

Set fso = CreateObject("Scripting.FileSystemObject")

Set f1 = fso.CreateTextFile("123.txt")

strComputer = "."

'f1.write (strComputer)

If Err.Description = "" Then
    Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")

    Set colDevices = objWMIService.ExecQuery _
        ("Select * From Win32_USBControllerDevice")

    For Each objDevice in colDevices
        n = InStr(1,objDevice.Dependent,"HID\\VID_")

        If n>0 Then
            h = mid(objDevice.Dependent,n+5,17)

            GetkeyInfo(h)

        End if
    Next

    Public Function GetKeyInfo(s)
        Set xmlDoc = CreateObject("Microsoft.XMLDOM")

        xmlDoc.async = False

        xmlDoc.load "data.xml"

        If xmlDoc.parseError.errorCode <> 0 Then
            Wscript.Echo "错误:" & Chr(13) & xmlDoc.parseError.reason
        End If

        Set objNodeList = xmlDoc.getElementsByTagName("page")

        i = 0

        Do While i < objNodeList.length
            h = objNodeList(i).selectSingleNode("hid").Text
            If h = s Then
                s="硬件 ID：" & s & Chr(13) & "加密锁类型：" & objNodeList(i).selectSingleNode("type").Text & Chr(13) & "厂商网站：" & objNodeList(i).selectSingleNode("url").Text & Chr(13) & "备份方案：" & objNodeList(i).selectSingleNode("fuck").Text
                f1.write (s)
                f1.write chr(13) & "#############################################################" & Chr(13)
                Exit Do
            Else
                's = "未知设备：" & s & Chr(13)
                'f1.write (s)
                i = i + 1
            End If
        loop

        End Function
    End if
```

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

说明：把 `data.xml` 同 `vbs` 文件放在同一文件夹下，运行 `vbs` 文件，查询结果会保存到 `123.txt` 中。代码写的很简陋，希望有兴趣的，或者闲得无聊的帮忙跟贴修改完善代码及功能，欢迎把你知道的加密锁的 `pid`、`vid` 补充到 `xml` 文件中，以方便更多的人更准确地了解手上的加密锁、加密狗的类型。

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `查狗工具.rar` |
|---|---|
