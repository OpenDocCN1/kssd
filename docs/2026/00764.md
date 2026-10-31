
# [讨论]OD能不能对API下Unicode String断点？

多字节字符串：`bp CreateFileA,[STRING [esp+4]]=="c:\\add.txt"`

有的软件不调用 `CreateFileA` 的函数，那有没有办法在 `CreateFileW` 函数上下字符串条件断点？

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
