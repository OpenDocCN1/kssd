
# [求助]`WriteProcessMemory`之后一定要`FlushInstructionCache`么？

如题，很多地方都这么说，包括MSDN。

但是我用`WriteProcessMemory`修改代码之后没有`Flush`，然后立即跳转回去执行。按说刚刚执行过的指令应该在cache里才对，那样修改内存应该是无效的。但是运行结果显示执行的是修改过的代码。

莫非`WriteProcessMemory`自动flush了？

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
