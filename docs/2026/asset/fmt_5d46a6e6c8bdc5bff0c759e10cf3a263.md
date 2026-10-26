
看到序列号后，我们就需要找到序列号的生成算法了，继续向上看，找到：

```
005ED976  |.  D80D A0DF5E00 fmul    dword ptr [5EDFA0]               ;  浮点型的注册码*5
005ED97C  |.  D825 A4DF5E00 fsub    dword ptr [5EDFA4]               ;  浮点型的注册码*5-1982
005ED982  |.  83C4 F4       add     esp, -0C
005ED985  |.  DB3C24        fstp    tbyte ptr [esp]
005ED988  |.  9B            wait
005ED989  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
005ED98C  |.  E8 9FD8E1FF   call    0040B230                         ;  将浮点型序列号转换成字符串
```

这个就是注册算法了，是不是很简单，只需将 注册码*5-1982就得到了序列号，连注册机都省了。

总结，序列号=注册码*5-1982。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
