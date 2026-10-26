
```text
invoke printbuftodump, reg05

;invoke printbuf, reg05, 5

;invoke MsgYn, "Process?"

;or reg00, 0

;je lp

invoke readmemlong , reg05, 4

invoke writememlong, reg01, reg00, 4

add reg01, 4

add reg05, 4

invoke readmemlong, reg05, 1

invoke writememlong, reg01, reg00, 1

sub reg01, 4

invoke printbuftodump, reg01

;invoke MsgYn, "Continue?"

;or reg00, 0

jne lp

end1:

;//////////////////////////////////////////////////////
```

运行上述两个脚本，以及之前的修复IAT脚本。 Dump出来，用`ImportRes`修复。（啤酒拿出来，准备庆祝）

运行一下新的`DIGI_DUMP_.EXE`程序 ~~ 程序界面飘然而至 ：）

## 七、（狂欢）I am a super dancing queen!

为了确定已经脱光了，用`OllyDbg`打开之，清除所有断点，对`.perplex`段下访问断点（在Alt+M中），运行。

试验各项功能，并没有被断下来，说明`.perplex`中的壳代码再也没有运行。

脱壳成功！！！

---

## 【经验总结】

至此，神州数码的UltraProtect壳已经被彻底的脱下来了。为后续的工作做好了准备。

脱壳中注意的关键点有：

1. 壳会对进程列表进行比对，关键点是`Process32First`函数   .........  `LOAD.TXT`脚本
2. 壳中有奇怪形式的IAT表，注意修复     ..........................  `修复IAT.TXT`脚本
3. 壳使用了Code Splitting技术，注意还原     .....................  `修复代码1.TXT` `修复代码2.TXT`脚本
4. 脚本真是个好东西

### 突破封锁线：第一章--脱壳篇 [完]

谢谢您耐心的看到这里，在下一章--资源篇中，将详细讲述如何修复资源和加入自己的资源到程序中。

---

## 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006年04月28日 14:49:03

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
