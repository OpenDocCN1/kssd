
```
0006ff3c : 7ffdf000 = (UNICODE) ""
0006ff3c : 7ffdf000 = (ANSI)     ""
0006ff40 : 80543940 =  Address Not Valid
0006ff44 : f4910c5c =  Address Not Valid
0006ff48 : 00000044 =  Address Not Valid
0006ff4c : 00092b30 = (UNICODE) ""
0006ff4c : 00092b30 = (ANSI)     ""
0006ff50 : 00092b50 = (UNICODE) ""
0006ff50 : 00092b50 = (ANSI)     "WinSta0\\Default"
0006ff54 : 00092b78 = (UNICODE) ""
0006ff54 : 00092b78 = (ANSI)     "C:\\WINDOWS.0\\System32\\notepad.exe"
0006ff58 : 00000000 =  Address Not Valid
```

`0:000> !tdbg.dumpstrings`

```
STACK   ADDR   STRING
0006ff5c : 00000000 =  Address Not Valid
0006ff60 : 00000000 =  Address Not Valid
0006ff64 : 00000000 =  Address Not Valid
0006ff68 : 00000000 =  Address Not Valid
0006ff6c : 00000000 =  Address Not Valid
0006ff70 : 00000000 =  Address Not Valid
0006ff74 : 00000000 =  Address Not Valid
0006ff78 : 00000000 =  Address Not Valid
0006ff7c : 00000000 =  Address Not Valid
0006ff80 : ffffffff =  Address Not Valid
0006ff84 : ffffffff =  Address Not Valid
0006ff88 : ffffffff =  Address Not Valid
0006ff8c : 00091ee8 = (UNICODE) ""
0006ff8c : 00091ee8 = (ANSI)     ""
0006ff90 : 00000000 =  Address Not Valid
0006ff94 : 00000001 =  Address Not Valid
0006ff98 : 00272620 = (UNICODE) ""
0006ff98 : 00272620 = (ANSI)     "(&'"
0006ff9c : 00272d00 = (UNICODE) ""
0006ff9c : 00272d00 = (ANSI)     "░-'"
0006ffa0 : 00000000 =  Address Not Valid
0006ffa4 : 00000000 =  Address Not Valid
0006ffa8 : 0006ff34 = (UNICODE) ""
0006ffa8 : 0006ff34 = (ANSI)     ""
```

我们在栈中找到了两个字符串，看起来不错。你会得到很多垃圾和没用的信息，还有你想得到的字符串信息。

## 总结

我希望你很高兴自己学习编写调试扩展，并且这个例子会给你带来帮助。另外还有其他一些 `API` 你可以使用，后面的教程可能也会解释一些，基本的操作这里都已经讲述了。将来，我们会探索更多的高级调试命令。
