
```
|:00461BFC(C), :00461C5A(U)
|
:00461C6E 33C0
  xor eax, eax
:00461C70 5A
  pop edx
:00461C71 59
  pop ecx
```

该段代码从文件`SC97.cfg`中读取`RegName`、`RegKey`，并进行注册码的验证，这样我们可以在`004611A0`处改代码为`(1)mov eax, 00000001 (2)ret`,然后删掉文件`SC97.cfg`，重新启动程序，nag screen不见了，但about中"registered to"还是"unregistered",没关系，用NotePad打开文件`SC97.cfg`，按照我软件的保护机制中讲的`SC97.cfg`文件格式添上`RegName`、`RegKey`这两行，内容随意输入你想要的,保存，再重新启动程序，ABOUT会告诉你"registered to xxxx"，Kool!

## The Patch

在文件`sc97.exe`的`offset:000605A0h`处，修改`558BEC33C951`为`B801000000C3`。
