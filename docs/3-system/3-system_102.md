
# 运行程序前执行自己的代码

> 原文：`www.pediy.com/kssd/pediy10/79808.html`

来看雪两年多了，到现在还是个相当菜的菜鸟，只索取，从来没有过回报，因为水平太次，实在是写不出来什么～

这个小程序实际是做毕业设计的时候写的，当时写完之后，没有太大的问题了，就没有再改(能蒙混过关就是目的)，里面现在仍然是 bug 多多。它主要是利用 `Tls CallBack` 来实现在运行程序之前先运行我们的代码，我们要把代码写入到 `exe` 中才行。

TLS 是 Thread Local Storage，即线程局部存储。线程局部存储有 2 种，动态的和静态的。动态线程局部存储使用 `TlsAlloc`, `TlsFree`, `TlsSetValue` 和 `TlsGetValue` 这些 API，利用 TLS 机制可以为进程中的每个线程关联若干个数据，各个线程通过由 TLS 分配的全局索引来访问与自己关联的数据。

而静态线程局部存储是在 Windows 的 PE/COFF 可执行文件格式中所支持的静态线程局部存储。即在 PE 头中数据目录表中的第 10 项，这一项指向的是 Tls 目录，Tls 目录的结构如下：

```
IMAGE_TLS_DIRECTORY32   STRUCT
StartAddressOfRawData   dd   ?
EndAddressOfRawData     dd   ?
AddressOfIndex          dd   ?
AddressOfCallBacks      dd   ?
SizeOfZeroFill          dd   ?
Characteristics         dd   ?
IMAGE_TLS_DIRECTORY32   ENDS
```

它的第三项是在线程初始化的时候要用的，使其指向一个可写的无用的数据区即可。第四项即是指向的 Tls CallBacks 的数组，这个数组以 0 结尾，在线程刚被创建的时候，系统会遍历这个 Tls Callbacks 的数组，它会依次调用各个线程回调函数。所以，只须在代码段中找到一些空闲的空间，然后将代码写到里面去，然后将再设置 Tls 目录的值，这样就实现了将 Tls 回调函数加入到其他可执行文件中。

具体的解释可以见 team509 的文章：

http://www.team509.com/modules.php?name=News&file=article&sid=73

我现在大概讲一下思路吧

在 Edit 中按一定格式写好代码之后，然后点编译，程序会先将 Edit 中的代码写到一个临时文件 `temp.asm` 中，然后加上头之类的，形成一个能编译的基本文件。

然后会调用 `nmake` 来编译，`makefile3` 会被调用，然后生成 `temp.exe`

然后从有效位置把有效的数据提取出来，也就是有效代码，在 `temp.exe` 中，一般有效代码开始位置为 `0x1000`，然后将其读出来，直到出现 `MarkLab` 标记，这个标记是在程序中自己设定的，为了能很好的找到结尾。(主要是在 `_GetBinaryCode` 中完成的)

提取出来之后就将这段有效代码，保存到 `Binary.dat` 中

然后在判断 PE 是否有效等，再查找代码段，然后判断代码段是否有足够的空间，如果有足够的空间，则会将 `Binary.dat` 中的数据写到 `exe` 中，作为 Tls Callback

实现起来并不难，主要是算 Tls Callback 的位置比较繁琐

因为当时写的时候，直接写死的，运行的时候最好放到 `C:\Temp\Tls`，当时好像用当前目录设置有问题，而且 `Binary.dat` 每次都会往这个目录下写，所以没办法，就直接把 `tls.exe` 放这里面执行就行了。

而且如果要自己编译的话，要改一下 `makefile` 和 `makefile3`

将里面的头文件以及库文件的路径设置为正确的

因为有 `Binary.dat` 了，所以也可以不编译，直接选择文件往里写，这时候会从当前文件夹下的 `Binary.dat` 中取数据，然后到目标 `exe` 中

测试了几个简单的 `exe`，正常，但有的 `exe` 写的时候会有错……

这马上就要到 2009 年了，也祝大家新年快乐。

上传的附件

| 文件 | 说明 |
| :--- | :--- |
| Tls.rar | 解压密码：`pediy` |
