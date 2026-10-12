
```
01041C81 50              PUSH EAX  
01041C82 E8 3135FFFF    CALL 010351B8 ; JMP to kernel32.GetModuleHandleA  
01041C87 5D              POP EBP  
01041C88 C2 0400         RETN 4  
```

调用`GetModuleHandleA`得到目标程序的基址，因此，入口点是`004073A0`但是我们必须把`CALL target.004072DC`改为`CALL GetModuleHandleA`。这是一个非常好的方法……

通过继续调试你可以确定不会再跳回壳中，因此可以确定现在一直在程序代码里了。另外，像我前面说的，你应该能看到对常用API的调用，很可能所有的伪装API调用已经被壳加载了（使用`GetProcAddress`或其他方法）。

（未完待续，持续更新，敬请关注）

4dd466d0e1e0cc123afbe504da2462ca
