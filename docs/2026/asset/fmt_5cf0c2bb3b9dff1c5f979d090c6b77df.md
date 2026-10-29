
`https://github.com/obfuscator-llvm/obfuscator/wiki/Instructions-Substitution`

前面三种都是混淆用的

最后一个选项是自己创建函数选择要不要混淆函数

看第一个选项再官网中的描述，也就是其他文章中的指令替换

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_JBSCHN97XZ7PBBP.png)

前两种混淆再ida中都可以自动的帮我们优化掉，然后加了随机数之后ida就不行了

## 指令替换

然后编写c文件作为测试用例：

```c
#include <stdio.h>
int main(int argc, char const *argv[])
{
    int n = argc + 8;
    if (n >= 10) {
        printf("hello ollvm:%d\r\n", n);
    } else {
        printf("hello kanxue\r\n");
    }
    return 0;
}
```

然后用刚刚生成的`clang`（里面已经集成及ollvm的pass）编译一下：

从编译的结果并看不出来什么不同

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_WE9FWVBXBHV9PS8.png)

然后拖入ida7.5看看

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_FEWG4D29QPYUBYB.png)

指令只有这一处发生了变化：

```
add eax, 8
```

变成了：

```
sub eax, 8BAA3A92h
add eax, 8
add eax, 8BAA3A92h
```

如果看伪c代码两个的结果是一样的

然后增加混淆力度看看：

```bash
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' -mllvm -sub -mllvm -sub_loop=6 hello_ollvm_bcf.c -o hello_ollvm_sub_6
```

拖入ida中看看：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_FNP8VM84TYW6K94.png)

## 虚假控制流

贴一下c源码：

```c
#include <stdlib.h>

int main(int argc, char** argv) {
    int a = atoi(argv[1]);
    if (a == 0)
        return 1;
    else
        return 10;
    return 0;
}
```

执行一下混淆命令：

```bash
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' -mllvm -bcf -mllvm -bcf_loop=6 hello_ollvm_bcf.c -o hello_ollvm_bcf_6
```

然后拖入ida看看

先看看`bcf_loop=1`的时候：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_Y8GMEJX6RAVF3P3.png)

他就比源程序混淆出来一个`while`语句（然后我刚刚动态调试的时候发现`x`、`y`的值都是0，所以这个`while`分支是不成立的）：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_8TBKPAPCSCDHKD2.png)

这个时候还能看看，然后加到3：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2AGM87KTWNCCWHK.png)

发现他的混淆力度还是可以的一条`if`语句出来这么多分支

然后要是动态跟着一路走过去发现，所有的`while`语句都只判断了前半部分，剩下的都没执行

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_74723ZDQERQPBT7.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_QW7EDAA895MEXFM.png)

走到了最后函数的返回值是10，然后返回linux虚拟机看看返回值，确实是10

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TBEZRTVAS3E54DH.png)

这一大堆不成立的`while`语句就是其他文章中的虚假控制流吧

我给大家贴一下程序的执行流程吧:

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_JJ54DSEYACZQDDT.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_WHFMEW8MDS6PM5G.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TQ7EJ6482E9CCA9.png)

然后一直到这里才是我们程序的原始的分支

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_YFSBYPH7YCQZDWS.png)

然后又是虚假控制流：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TS63VKJ3Y4ETF8Z.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_URSBQXT6P8MRRCB.png)

一直到这里程序才执行完：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_U3MP2RJ88ZZD7TJ.png)

混淆到9的时候直接出不来了xswl：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_5KHJEMCSABNVFX2.png)

然后改成999：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_HY6ZFZSJ3WFZGEB.png)

然后如果想看伪c代码的话也会耗费很长时间：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_XRXA3D54UJVYY4W.png)

bcf：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_699NJHMBM5DVF2R.png)

太6了

然后我们会发现一个规律，就是无论你`bcf_loop`到多少，总会有一段或者两端的`while1`的内容来恶心你：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_JAWZZXH537T89AX.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_SDUF8X4PTR7BR8H.png)

如果要是混淆用到这个手法的话，可以用动态调试的方法来跟着程序执行，不过这样比较慢，应为许多不执行的`while`语句（但是ida不这么认为啊）也会跟到程序的执行过程中，所以我想出来了一种跟踪变量的方法，可以正着跟，也可以从函数的返回值出发：

这里我简单的说一下我的想法，当然网上也有很多别的更好的方法，一会会说一下`unicorn`模拟执行，我这个跟踪变量的方法只能用于比较简单的程序：

先从`v19`-》`v22`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_27H5YBBQH6RW8ZQ.png)

然后往上跟，发现了两个值，正好是我们刚刚源程序中写的值1，10：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_JFKHFZGMV5CXJ26.png)

然后这样一直网上跟也能发现程序的逻辑，不过这种方法比较恶心，还是学习一下其他大佬的`unicorn`方法吧，还有符号执行方法

然后这个虚假控制流还可以指定混淆的百分比：

```bash
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' -mllvm -bcf -mllvm -bcf_loop=3 -mllvm -bcf_prob=80 hello_ollvm_bcf.c -o hello_ollvm_bcf_3_80
```

发现他的混淆力度也是很强的：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_RBZ679VN7AJWPDA.png)

## 控制流平坦化

`https://github.com/obfuscator-llvm/obfuscator/wiki/Features`

看官方文档介绍就是将`if-else`分支的基本块，转化成`switch`的基本快

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2ED3JHX5N7F3ESA.png)

我们这里还是用官网中给的c代码：

```c
#include <stdlib.h>

int main(int argc, char** argv) {
    int a = atoi(argv[1]);
    if (a == 0)
        return 1;
    else
        return 10;
    return 0;
}
```

执行一下混淆代码：

```bash
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' -mllvm -fla hello_ollvm_fla.c -o hello_ollvm_fla
```

然后拖入ida看看:

混淆的也比较明显了

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_RKG9MPWNR5QE6N2.png)

ida中识别的`switch-case`语句不是很准确

然后生成一下`.ll`文件看看程序流程就比较明显了：

```bash
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' -mllvm -fla -emit-llvm -S hello_ollvm_fla.c -o hello_ollvm_fla.ll
```

代码如下：
