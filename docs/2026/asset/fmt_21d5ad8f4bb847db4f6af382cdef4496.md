
```vb
GetWindowThreadProcessId WndHandle, ProcessID                '获取PID

If ProcessID = 0 Then

    MsgBox "获取程序PID错误"

Else

    proHandle = OpenProcess(PROCESS_ALL_ACCESS, False, ProcessID)

    '获取进程句柄

    ReadProcessMemory proHandle, ByVal &H101200C, ReadData(0), 4, 0&

    ESI4 = ReadData(0)

    ReadProcessMemory proHandle, ByVal &H1012010, ReadData(0), 4, 0&

    ESI8 = ReadData(0)

    ReadProcessMemory proHandle, ByVal &H1012014, ReadData(0), 4, 0&

    ESIC = ReadData(0)

    '保存了关键的地址

    For n = 0 to 9                                                        '行数

        List1.AddItem "第" & n + 1 & "列"

        For i = 1 To 11                                                        '列数

            EAX = EBX = ECX = EDI = 0                                '初始化

            If (J = 0) Then

                lAddress = ESI8 + n * 4         'EAX*4

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                lAddress = ReadData(0)

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                lAddress = ReadData(0)

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                EDI = ReadData(0)                        '保存EDI

            Else

                lAddress = ESI8 + n * 4         'EAX*4

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                lAddress = ReadData(0)

                For I2 = 1 To J        '根据列数循环查找对应的地址

                    ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                    lAddress = ReadData(0) + 8

                Next I2

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                lAddress = ReadData(0)

                ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

                EDI = ReadData(0)                        '保存EDI

            End If

            J = J + 1

            EAX = EDI

            lAddress = ESI4 + 12

            ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

            lAddress = ReadData(0) + (EAX + EAX * 2) * 4

            ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

            EBX = ReadData(0) * 13

            lAddress = ESI4 + 12

            ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

            lAddress = ReadData(0) + (EAX + EAX * 2) * 4 + 4

            'B(m) = lAddress        '定义的一个地址数组B(109)存放的所有地址

            ReadProcessMemory proHandle, ByVal lAddress, ReadData(0), 4, 0&

            EAX = EBX + ReadData(0) + 1

            List1.AddItem 'CARD' & EAX                '输出为CARD？的字符

            m = m + 1

        Next i

        J = 0

    Next n

    CloseHandle (proHandle)

End If

End If
```

以上是读取每张扑克的点数，程序很简陋，只是用来测试一下的。程序中可以注意到我定义了`B(109)`的数组，用来存储地址，所以就可以来修改内存，这样就可以把每张牌修改成自己想要的点数

程序格式如下

```vb
WriteProcessMemory proHandle, ByVal B(0), 0, 4, 0&        '这里0表示A的意思
WriteProcessMemory proHandle, ByVal B(109), 1, 4, 0&        '1表示2的意思
```

其他的同理修改即可。

```vb
InvalidateRect AppHandle, rect, True        '用来刷新屏幕(rect取窗体大小即可)
```

发个图给大家看一下吧，如下图

经过测试可以在初级中实现秒级破解，在中级和高级中效果一样，不过会涉及到扑克牌花色的影响，可以再改进的，这里我没有在继续了，思路都是差不多的，有兴趣的可以试试。

![](https://bbs.kanxue.com/upload/attach/200911/153401_5o9os45qnr65d2y.jpg)

![](https://bbs.kanxue.com/upload/attach/200911/153401_34on3g6l0n1se5t.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [蜘蛛纸牌的游戏辅助.doc](javascript:void(0);)

  （389.50kb，179次下载）

- [2.JPG](javascript:void(0);)

  （53.80kb，658次下载）

- [3.JPG](javascript:void(0);)

  （28.40kb，657次下载）
