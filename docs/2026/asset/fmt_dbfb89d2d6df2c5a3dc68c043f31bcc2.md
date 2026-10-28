
```
0040100E  |.  FF15 08204000 CALL    NEAR DWORD PTR DS:[<&user32.Mess>; \MessageBoxA

00401014  |.  6A 00         PUSH    0                                ; /ExitCode = 0

00401016  \.  FF15 00204000 CALL    NEAR DWORD PTR DS:[<&kernel32.Ex>; \ExitProcess
```

2个`call`完全优化了。

不过要注意，用他们这个`inc`，不能使用`vs2008`自带的`ml`否则编译就报错，可以用`RadASM`里自带的`masm`编译。之需要把参数`/I`指向下载的`win32inc`的`include`就可以了。具体功能怎么实现的，我是不理解，还得请高手们帮忙看看。

星期日, 五月 03, 2009

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
