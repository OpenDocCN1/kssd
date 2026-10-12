
# 变脸王

> 原文：[`www.pediy.com/kssd/pediy06/pediy6255.htm`](https://www.pediy.com/kssd/pediy06/pediy6255.htm)

这是一份我做的变脸王补丁。因为是演示功能，所以功能不多。 我感觉自己做补丁可以很好的解决一些软件自启动时自我检测的问题。比如 这款变脸王我用`keymaker`等就做不出来好用的内存补丁。由于可以自己指定程序所在路径，所以可以不把补丁和被补程序放到一个路径，这对于光盘这种的程序很有好处。 有什么问题和建议欢迎和我探讨。 email: `advice107@sohu.com`

```
'本程序为变脸王 4.1.0.0 补丁,本补丁对鼠标设置和桌面颜色设置进行了内存修改,使为注册用户可以进行多次修改.   

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

pHandle = OpenProcess(PROCESS\_ALL\_ACCESS, False, pid)   

If (pHandle = 0) Then   

MsgBox "Couldn't get a process handle!"   

Exit Sub   

End If   

End Sub   

Private Sub Command1\_Click()   

Dim path As String   

On Error GoTo exi:   

path = Text1.Text + "\\WinBeautician.exe"   

'Shell "C:\\My Documents\\Windows 变脸王\\"  '运行变脸王   

Shell path   

Call gethwnd("Windows 变脸王")   

'下面一次修改鼠标   

If Check1.Value = 1 Then   '鼠标设置   

'把 4D7F6B 处修改成   

   res = "G"     '修改数据修改成 47H,因为"G"的 AscII 码是 47H,所以相当于把"G"放入此处.   

                 'res 可以设置成其他类型,如果哪位有其他修改方法,还请告诉我,在此谢过.   

   WriteProcessMemory pHandle, &H4D7F6B, res, 1, 0&    '变脸王鼠标设置   

End If   

If Check2.Value = 1 Then   '桌面颜色设置   

   res = "u"     '修改数据.把&H4E72C4 处修改成 75H   

   WriteProcessMemory pHandle, &H4E72C4, res, 1, 0&   

   res = "t"     '修改数据.把&H4E72CA 处修改成 74H   

   WriteProcessMemory pHandle, &H4E72CA, res, 1, 0&   

End If   

' 关闭进程句柄   

CloseHandle pHandle   

Exit Sub   

exi:   

MsgBox "出现错误,请按提示进行操作!", vbCritical, "警告"   

End Sub   

Private Sub Command2\_Click()   

MsgBox "本程序适用于变脸王 4.1.0.0 版本" + vbCrLf + "作者: NBW" + vbCrLf + "Email:  advice107@sohu.com" + vbCrLf + "qq:  37122085", vbInformation, "关于"   

End Sub
```

`4dd466d0e1e0cc123afbe504da2462ca`
