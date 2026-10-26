
# [原创]发一个重定向hosts文件的方法

用`procexp.exe`找到启动参数是`C:\WINDOWS\system32\svchost.exe -k NetworkService`的`svchost`进程，

然后用`winhex`打开此进程，搜索`\drivers\etc\hosts`，修改一下这个文件名，比如`\drivers\etc\hao2`，

然后随便改改`etc`目录下的文件(`FindFirstChangeNotificationW`)，`svchost`就会读取`\drivers\etc\hao2`更新`hosts` cache，

真正起作用的就变成`hao2`，而不是`hosts`了。

跟`hosts`文件有关的`DLL`是`dnsrslvr.dll`和`dnsapi.dll`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)
