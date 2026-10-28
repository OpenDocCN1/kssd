
通过这种方法，可以混淆安全分析人员，使其以为`0x570000`上的内存有问题，然后dump并分析它。但实际存在shellcode的内存`0x60000`并没有找到。

![](https://bbs.kanxue.com/upload/attach/201907/749942_5MSZ4AYGW39MEGP.jpg)

`hollowfind`插件就是用来检测各种类型的进程注入。`hollowfind`插件会检测所有可疑的内存保护属性（带执行属性的）。下面截图中显示了`hollowfind`找到了`explorer.exe`进程（PID 2588），同时也找到了入口点的差异，跳转到`0x60000` shellcode的指令。

![](https://bbs.kanxue.com/upload/attach/201907/749942_PBBFXN7EDQ3RKBN.jpg)

下面截图中，`hollowfind`插件显示了相似的进程以及可疑的内存区域，同时也认为`0x60000`地址是可疑的，因为这块内存上没有包含一个PE文件却存在`PAGE_EXECUTE_WRITECOPY`属性。

![](https://bbs.kanxue.com/upload/attach/201907/749942_7WU9TTHA8NESB5R.jpg)

## e）案例5：Kronos木马中实现的进程注入（修改内存保护属性为`PAGE_EXECUTE_WRITECOPY`）

几天前，我发现一个名为Kronos的木马使用了以上一个案例中类似的手段，这个样本注入到了`explorer.exe`进程，然后修改了explorer入口点的地方为跳转指令，跳转到一处内存属性为`PAGE_EXECUTE_WRITECOPY`的地址上执行。

虽然在注入到`exolorer.exe`后，然后恢复进程运行后，进程崩溃了，此时仍旧可以保存dump来分析一下。

![](https://bbs.kanxue.com/upload/attach/201907/749942_JW3GDFD6AWMQNHU.jpg)

运行`HollowFind`插件来检测被kronos感染过的内存dump文件，检测到了可疑的进程和入口一个跳转指令，尝试跳到`0x6f60b`地址上执行。

![](https://bbs.kanxue.com/upload/attach/201907/749942_3BSD6ZJHBFTWFW3.jpg)

这个插件同时也检测到了可疑内存区域`0x60000`，这里保存的是一个pe文件，但是并不是在内存中展开的。并且有`PAGE_EXECUTE_WRITECOPY`属性。

![](https://bbs.kanxue.com/upload/attach/201907/749942_7HH3XMQX93H797E.jpg)

实验证明，通过注入代码到内存属性为`PAGE_EXECUTE_WRITECOPY`的内存中，使用`hollowfind`插件仍然可以有效检测这类代码注入攻击。

# 总结

Process Hollowing是一种将恶意代码注入到系统进程中执行的技术，看起来黑客通过使用不同类型的进程注入方式，不仅要注入系统进程，还要保持自身隐蔽性，来对抗取证工具、欺骗安全分析人员。

但是从应急响应的角度来看，了解这种隐藏方式的原理非常有必要。了解这些技术有助于我们更好的应对此类木马攻击。

`HollowFind`插件就是用来专门检测这些注入技术，插件通过查找VAD和PEB中的差异来检测此类攻击，还分反汇编入口点的地址来检测是否存在可疑跳转指令，并展示任何可能存在注入代码的可疑内存信息。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
