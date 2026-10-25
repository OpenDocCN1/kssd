
# windbg 内核调试模式下条件断点无法断下

如`bp /w "@rcx==94" nt!NtWaitForSingleObject`

`g`只有VM虚拟机是繁忙状态(无法操作),只能通过中断`Ctrl-B`，经测试在用户模式可以断下的，但是内核模式下不行，哪位大佬知道是什么原因呢？

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
