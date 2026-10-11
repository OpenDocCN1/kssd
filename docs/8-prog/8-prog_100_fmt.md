
# 使用 TASM 编译 COFF 格式和连接

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-670.htm`](https://www.pediy.com/kssd/pediy08/pediy8-670.htm)

看到网络上流传的一份 `Drocon` 的 `mercury` 的代码。

程序源码使用 `TASM32` 编译，使用 `MASM32` 来连接...关键的地方就在这里。

为什么要使用 `TASM` 编译？正常情况下，`TASM` 连接出来的程序代码体积远远大于 `MASM32` 连接出来的。

其实具体看一下就不难发现，`TASM` 编译出来的 `obj` 体积很小，连接出来以后体积增加了。既然编译器原理差不多，为什么不可以这样使用（我是以 `FASM` 来对比的）？

使用 `MASM32` 编译出来的 `OBJ` 体积和连接出来后的程序没有什么区别。即使你编译一个 `MessageBox` 的程序还是会生成一个 1.9k 的 `obj`，相对来说 `TASM` 体积就小巧多了。

回头来说说 `mercury` 吧。

先说说大概的代码情况，首先他是 `TASM32` 编译的源代码（编译成 `*.obj`），然后用 `MASM32` 的 `link`（连接器）来连接 `EXE` 的。（看他的 `make.bat` 就知道了）连接库路径是 `TASM` 的。所以有的人就用 `tasm5plus` 里面的 `implib.exe` 来生成 `lib`，但是发现还是不能编译。

这里翻翻 `MASM32` 的 `link` 说明就知道，`link.exe` 只能连接 `COFF` 格式的 `obj`，`TASM32` 编译出来的 `obj` 是 `OMF` 格式的，想要让 `link` 可以正常连接，只有让 `TASM32` 编译出 `COFF` 格式的 `obj`。

如何做到？一些汇编语言编译器自带的例程中都有教如何编译 `COFF` 格式的例子。暂时我只看到 `FASM` 有这个。好了下面说说如何编译吧。

首先看看他的代码（`drocon` 删除了几段代码--这里鄙视他一下！！！），首先参数传递方式和 `COFF` 的导入表调用方式。举个例子，`MessageBoxA` 的导入表正常就是 `MessageBoxA` 等，但是用记事本打开文件看看你就会发现他的格式是 `__imp__MessageBoxA@16` 这样的格式（大多数都是这样的）。于是我们就写个宏前面添加 `__imp__` 就可以了，`drocon` 的代码中也有不过他删除了。

其实如何发现是使用 `__imp__MessageBoxA@16`？

打开 `VC` 写个 `MessageBox` 的程序，调试 -> 反汇编：就而已看到了。

编译的时候把代码入口点定位在 `Start` 上。

下面写个简单的例子：

```
callw  macro   x
 extern  C _imp__&x:DWORD
 call _imp__&x
endm

.586
.model flat

public C Start

.code
 Start:
   push 0
   push 0
   push 0
   push 0
   callw MessageBoxA
  RET

end Start
```

```
bin\tasm32 /mx /m4 /z MsgBox.asm
bin\link /subsystem:windows lib\kernel32.lib lib\user32.lib /ignore:4033,4078 /nologo /entry:Start /FILEALIGN:0x200 /MERGE:.data=.text /MERGE:.rdata=.text /SECTION:.text,RWX MsgBox.obj
Del MsgBox.obj
pause
```

编译小程序不会发现什么特别的地方，但是编译一个稍微大一点的程序就可以发现了，`mercury` 代码 13k 但是编译出来以后只有 3.5k（不加壳）。

如果 `API` 函数全部采用内存搜索的话（和写病毒差不多），编译后体积 2k（不加壳）。

怎么说好呢？`TASM` 的编译器优秀？`MASM` 连接器优秀？

许多资料可以从 29A 的一些电子杂志中找到。

`Delphi` 编译出来的 `OBJ` 也可以使用此方法，采用 `MASM` 来连接。

小弟技术菜只能做到 640 字节的 `MessageBox` 程序。

引用: *最初由 dwing 发布*

哦，真的可以编译出不按照 512 字节对齐的文件吗？

`MASM` 需要加什么参数？

可以！

```
bin\tasm32 /mx /m4 /z a.asm

bin\link /ALIGN:32 /FORCE:UNRESOLVED /subsystem:windows lib\kernel32.lib lib\user32.lib /ignore:4033,4078 /nologo
/entry:Start /FILEALIGN:0x200 /MERGE:.data=.text /MERGE:.rdata=.text
/SECTION:.text,RWX a.obj

Del a.obj
pause
```

加参数：`/ALIGN:32 /FORCE:UNRESOLVED`

就可以了，原代码可以编译出 604 字节的 `EXE`。

（忘记在那个代码里看到的了，感谢那个被人们遗忘的 Hero~）

不过这样编译以后加壳就会出问题了，估计是加壳软件无法修改这种对齐方式了。

引用: *最初由 Anskya 发布*

加参数：`/ALIGN:32 /FORCE:UNRESOLVED`

`align` 设置为 32 可能会有兼容性问题，如 `Win9x` 中可能无法运行。`WinXP` 的兼容性就比较好了（因为要合并 `Win2000` 和 `Win9x/ME`）。
4dd466d0e1e0cc123afbe504da2462ca
