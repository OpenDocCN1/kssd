
# [原创]异常处理流程（二） 三种常见的异常

很多时候我们需要主动制造异常，那么异常代码长什么样呢？我们来了解三种常见的异常。

第一种，内存访问异常。

内存页面的属性有三种，可读（R），可写（W），可执行（E）。如果有一段内存段只有读属性，但调试员却想修改这段内存的数据。这时会显示：访问违规，并提示使用`shift+f7/8/9`来忽略异常。

怎么知道那个段有什么权限呢？来到M窗口观察“访问”列即可，里面是R，W，E三个字母的组合，如图。

![](https://bbs.kanxue.com/upload/attach/201805/803510_6TE5U3M8JU89X3K.jpg)

怎么修改权限呢？M窗口找到PE头，双击就可以查看PE文件了，如下图所示。

![](https://bbs.kanxue.com/upload/attach/201805/803510_VUEUZG2QYD76BR4.jpg)

最后几行，有一行叫：`Offset to PE signature`，该值加基址就是PE头开始的地方，翻译成：PE头的偏移量。介绍几个参数：

`Addressofentrypoint` 程序的入口，这是偏移，就是程序加载的起始位置，是EP，不是OEP，就是载入程序后从哪开始运行。

![](https://bbs.kanxue.com/upload/attach/201805/803510_2CN6V9GMKGW5CZE.jpg)

往下拉，看到ASCII`'text'`，这里是对`.text`的属性设置处。

![](https://bbs.kanxue.com/upload/attach/201805/803510_G8HCW8G7QM8H7S2.jpg)

`text`段最后：`characterastics=CODE|EXECUTE|READ|WRITE`，说明这段的权限是可执行，可读，可写。那么怎么修改属性呢？在这行右键-修改整数，这里显示为`E0000200`，将其改为`60000200`就将权限改为了可读可执行。就成功修改了权限。保存即可。

第二是除零异常。因为除数不能为0，即分母不能为0。这是四则运算的原则，我们来构造一段除零异常的代码，如下：

```
Xor ecx,ecx                将ECX是被除数置0
Mov eax,1                  给被除数赋初始值为1
Div ecx                    eax除以ecx
```

运行这段代码时，OD左下角会显示除零异常。

第三是无效指令和特权指令时异常。

我们无法创造一个系统不认识的指令。这类指令会被认为是无效的。

特权指令的典型是`int3`指令，在CC断点那一课我们了解过一些。中断的本质是异常。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
