
# 我用VB做了一份内存补丁。请大家鉴定。

- 标题：我用VB做了一份内存补丁。请大家鉴定。
- 发信人：蓝星
- 时间：2003年10月02日 08:32

详细信息：

这是一份我做的变脸王补丁。因为是演示功能，所以功能不多。

我感觉自己做补丁可以很好的解决一些软件自启动时自我检测的问题。比如这款变脸王我用keymaker等就做不出来好用的内存补丁。由于可以自己指定程序所在路径，所以可以不把补丁和被补程序放到一个路径，这对于光盘这种的程序很有好处。有什么问题和建议欢迎和我探讨。email: advice107@sohu.com

```
'本程序为变脸王4.1.0.0补丁,本补丁对鼠标设置和桌面颜色设置进行了内存修改,使为注册用户可以进行多次修改.
'如有问题请和我联系.
'Email:  advice107@sohu.com
' 声明一些需要的变量
Dim pHandle As Long ' 储存进程句柄
Dim res As String   'res 为修改后的数据
Dim pid As Long ' 储存进程标识符( Process Id )

Private Sub gethwnd(proname As String)
Dim hwnd As Long ' 储存 FindWindow 函数返回的句柄
' 首先取得目标窗口的句柄
hwnd = FindWindow(vbNullString, proname)
If (hwnd = 0) Then
    MsgBox "没有发现" + proname + "窗口！"
    Exit Sub
End If
' 取得进程标识符
GetWindowThreadProcessId hwnd, pid
' 使用进程标识符取得进程句柄
pHandle = OpenProcess(PROCESS_ALL_ACCESS, False, pid)
If (pHandle = 0) Then
    MsgBox "Couldn't get a process handle!"
    Exit Sub
End If
End Sub

Private Sub Command1_Click()
Dim path As String
On Error GoTo exi:
path = Text1.Text + "\WinBeautician.exe"
'Shell "C:\My Documents\Windows变脸王\"  '运行变脸王
Shell path
Call gethwnd("Windows变脸王")
'下面一次修改鼠标
If Check1.Value = 1 Then   '鼠标设置

'把4D7F6B处修改成

   res = "G"     '修改数据修改成47H,因为"G"的AscII码是47H,所以相当于把"G"放入此处.
                 'res可以设置成其他类型,如果哪位有其他修改方法,还请告诉我,在此谢过.
   WriteProcessMemory pHandle, &H4D7F6B, res, 1, 0&    '变脸王鼠标设置
End If

If Check2.Value = 1 Then   '桌面颜色设置
   res = "u"     '修改数据.把&H4E72C4处修改成75H
   WriteProcessMemory pHandle, &H4E72C4, res, 1, 0&
   res = "t"     '修改数据.把&H4E72CA处修改成74H
   WriteProcessMemory pHandle, &H4E72CA, res, 1, 0&
End If

' 关闭进程句柄
CloseHandle pHandle
Exit Sub

exi:
MsgBox "出现错误,请按提示进行操作!", vbCritical, "警告"
End Sub

Private Sub Command2_Click()
MsgBox "本程序适用于变脸王4.1.0.0版本" + vbCrLf + "作者: NBW" + vbCrLf + "Email:  advice107@sohu.com" + vbCrLf + "qq:  37122085", vbInformation, "关于"
End Sub
```
