
# [原创]Asm的魅力（三）

又是好长时间没有来看雪了，一直跟着师傅在闭关研究内核，希望将来可以有所成果，发出来和大家分享。

还是接着以前的主题，给大家做点科普。人都说我是“纯语言研究者”，哉也！！！呼呼！！![](https://bbs.kanxue.com/view/img/face/50.gif)

今天说下`asm`和`c`的杂交。。。。。。貌似很乱。。。。呼呼！！

进入主题-----------------------------------------------------------------

大家可能对`inline asm`比较熟悉了，说实话，`inline asm`确实很强大，实际上比纯`asm`都要难，为什么？因为他限制多，因为他更灵活。

那`inline`呢我就不说了，提醒下大家，不要以为能够写出个小程序就会了，重点要知道两方面的东西

->纯`asm`和`inline asm`的区别

->弄清楚什么时候用`inline asm`

两者的区别的大概有20多条，慢慢看！

下面我分情况来介绍杂交：

## <一>

### C调用`asm`模块：

什么意思呢？就是说我们的程序是由`c`写的主函数来启动的，但是里面的一些函数是在`asm`里面实现的。

为什么这样写的？因为有些子过程用`asm`写更方便或者是其他的`xx`目的，呼呼！

这里面有一些细节的东西是需要注意的，但是基本算是套路，掌握了的话那就掌握了！真绕嘴！

C主函数部分：

```
#include "stdio.h"

extern void _stdcall output(void);///////////////////////////////////////////////////////////////*

int main(void){



    //sub proc

	output();

    return 0;}
```

`asm`模块部分：

```
public output///////////////////////////////////////////////////////////////**

.data

szTtitle db 'charme',0

szText db '这是调用asm模块output子过程输出的信息',0

.code

output proc

    push 40h

    push offset szTtitle

    push offset szText

    push 0h

    call MessageBox

    ret

output endp

end output
```

重点注意的就是我注释了的部分！意思大家一看就明白！我不多说了，运行结果看打包的代码！

## <二>

### Asm调用`c`模块：

什么意思呢？就是说主函数是用`asm`写的，子过程我们用`c`来实现！

为什么这样写呢？因为有时候可能一些子过程我们用`c`已经写过了，或者说相对于写`asm`代码要简略，所以何必做重复的工作呢？

同样只需要注意一些细节！

Asm主模块：

```
.386

.model flat,stdcall

option casemap:none



include windows.inc

include user32.inc

includelib user32.lib



output proto  ////////////////////////////////////////////*



.code

main proc C

   call output

   ret

main endp

end main
```

`c`子模块：

```
#include "stdio.h"

#include "windows.h"



extern void _stdcall output(void){///////////////////////////////////////////////////////**

MessageBox(NULL,"charme","这是调用C子过程的时候输出的信息",0x40);}
```

注意这个`c`模块的写法，我们把他看作是个子模块，所以他没有入口点，也不需要，`ok`?

着重的注意下注释的部分。

那么杂交的话基本就介绍完了，实际上很简单！

我们再延伸下思路：为什么要介绍这个东西？不要把他看成是个简单的把戏，实际上你要掌握了这样的写法，你会觉得写程序很自在的，游刃有余！子模块或者子函数之间互相调用，好处多多！

大家都说我不写`XX`的东西出来。光是研究什么`asm` 啊，`c`啊怎么怎么写。。。。。。。我想了下，对啊，为什么不做点实际点的东西出来？后来再一想，不对，那样有意思？比如我说的杂交，你都不知道怎么杂交，还提生个什么样的杂种，这不是很可笑么？所以我还是当个接生的把！呼呼！

上面我说子模块或者子函数，那么这个叫法是我的习惯，我也不知道书本上的话这两个是不是一样的意思，我是区分开的！

就用`asm`的代码来解释这个东西

### Asm调用`asm`子模块：

那么假设我们有两个`asm`文件—`asm.asm`    `subtest.asm`那么我们要在`asm.asm`里面调用`subtest.asm`，这样的做法就是模块的调用，因为一旦调用，所有`subtest.asm`里的代码都会被解释执行。所以我叫做子模块。看看代码---

`Asm.asm`:

```
.386

.model flat,stdcall

option casemap:none



include windows.inc

include kernel32.inc

include user32.inc



includelib kernel32.lib

includelib user32.lib



.data

szText db 'asm自身模块调用',0

szTitle db 'charme',0

szInfo db 'asm模块调用示例',0

.code

start:

    push 40h

    push offset szInfo

    push offset szText

    xor ebx,ebx

    push ebx

    CALL MessageBox



include subtest.asm

end start
```

`subtest.asm`

```
subtest proc

    push 40h

    push offset szText

    push offset szTitle

    

    push ebx

    call MessageBox

    push ebx

    call ExitProcess

subtest endp
```

### `asm`调用`asm`子函数：

假设我们还是有两个`asm`文件：`main.asm` `output.asm`

那么我们要在`main.asm` 里面调用`output.asm`里面的部分函数来实现一些功能，那么这个时候编译连接的时候是有选择的来获取`output.asm`里面的代码的，所以我叫做子函数调用。

`Main.asm`:

```
.386

.model flat,stdcall

option casemap:none



include windows.inc

include user32.inc

includelib user32.lib



output proto



.code

main proc C

   call output

   ret

main endp

end main
```

`output.asm`:

```
.386

.model flat,stdcall

option casemap:none



include windows.inc

include kernel32.inc

include user32.inc

includelib kernel32.lib

includelib user32.lib



public output

.data

szTtitle db 'charme',0

szText db 'asm自身模块调用',0

.code

output proc

    push 40h

    push offset szTtitle

    push offset szText

    push 0h

    call MessageBox

    ret

output endp

end output
```

好了，现在子模块和子函数的调用概念也清楚了，那么掌握这个东西基本没有难点了。

`Intel`汇编程序设计上还有很古老的一种杂交方法，不过现在真的很少用了，我不介绍了，现在的这些足够了！

大家看到了，每个示例代码都很简单的，不是为了实现什么功能，就是为了让大家更清晰的掌握本质。

文章写的仓促，虽然很短！有错误或者是遗漏的地方请指出！也欢迎大家娶我空间转转b27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3q4J5L8h3f1H3x3o6m8Q4x3V1y4Q4c8e0N6Q4z5p5g2Q4b7U0m8Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0c8Q4b7V1q4Q4b7V1q4Q4c8e0S2Q4b7V1k6Q4b7U0W2Q4c8e0N6Q4b7V1c8Q4z5e0g2Q4c8e0S2Q4z5o6N6Q4b7U0y4Q4c8e0g2Q4z5e0g2Q4z5p5q4Q4c8f1k6Q4b7V1y4Q4z5o6q4Q4c8e0g2Q4z5e0q4Q4b7V1y4Q4c8e0g2Q4z5e0q4Q4b7V1y4Q4c8f1k6Q4b7V1y4Q4z5o6p5`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [asm+c杂交研究.rar](javascript:void(0);)

  （58.99kb，124次下载）
