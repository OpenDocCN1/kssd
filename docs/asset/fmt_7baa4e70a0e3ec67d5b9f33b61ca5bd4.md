
`System.Windows.Forms.Application::SetCompatibleTextRenderingDefault(bool)`

```
IL_0013:  /* 73   | (06)001848       */ newobj     instance void Xenocode.Fox.Gui.x3e4e23fadc83a77e::.ctor()
IL_0018:  /* 28   | (0A)000480       */ call       void [System.Windows.Forms]
System.Windows.Forms.Application::Run(class [System.Windows.Forms]System.Windows.Forms.Form)
IL_001d:  /* 2A   |                  */ ret
} // end of method x867eb3246b182488::xc447809891322395
```

这里写明了函数开始于 `RVA 0x112d68`，我们要做的是把正常运行前的 `if()` 判断去掉，也就是

```
IL_0000:  /* 28   | (06)001A21       */ call       bool Xenocode.Fox.Gui.x867eb3246b182488::_xc7c43f12e732db09()
IL_0005:  /* 2C   | 01               */ brfalse.s  IL_0008
IL_0007:  /* 2A   |                  */ ret
```

这三行，直接 `nop` 掉，在 `.net` 中 `nop` 指令最好记了，它就是 `00`。

找出一个二进制编辑器，跳到 `0x112d68-0x1000=0x111d68` 处，用眼睛扫描一下，`28 21 1A 00 06`（`ILDasm` 里显示的操作数居然不是内存顺序，晕）。

后面接着是 `2C 01 2A`，将这些全部改为 `00`，另存为 `UnFox.exe`，运行一下，成功！

再把注册表里的数改成过期数据，比如改为 `0`，再运行一下 `UnFox.exe`，呀，又成功了！

呵呵，祝贺一下（坏了！！！我本来想用 `Xenocode` 做什么来着？？？）

--------------------------------------------------------------------------------

【经验总结】

1. 用 `ILDasm` 反汇编完整文件，然后搜索 `System.Windows.Forms.Application::Run` 来发现程序入口点（其实搜索 `.entrypoint` 属性应该更准确了，还有 `STAThread` 对单线程程序也有效）；
2. `reflector` 就是好啊就是好，别忘了它还有个 `Analyzer` 查找引用（`Used By`）很有效，另外，在 `C#` 与 `IL` 间切换，可以学习 `IL` 语言哦（鼠标停在 `IL` 指令上能看到指令解释及指令机器码，上次有兄弟已经说过了，抄袭一下：）
3. `.net` 程序没有签名的话，好象很不安全呢。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪论坛，转载请注明出处。

2006 年 06 月 06 日 14:30:41

`4dd466d0e1e0cc123afbe504da2462ca`
