
# ModelMaker CodeExplorer v1.05

> 原文：[`www.pediy.com/kssd/pediy04/kanxue496.htm`](https://www.pediy.com/kssd/pediy04/kanxue496.htm)

现在我知道如何去掉时间限制了。

在注册表中查找`Trapezium.InternalLexer`然后删掉，再重新装。找到答案不是关键（何况这个答案肯定会令大多数人不满意），关键在于弄清楚`Code Explorer`的保护原理，这才是真正的`Cracker`的目的，所以请熟悉`Delphi`破解的高人帮俺分析分析，究竟过期以后，内部都发生了些什么。

`key:`

```
mov        eax,[ebp][-08]   

call        @Sysutils@FileDateToDateTime   

mov        eax,[ebp][-04]   

fstp        q,[eax][10]   

fwait   

call        @Sysutils@Now$qqrv ;rtl60.bpl   

mov        eax,[ebp][-04]   

fsubr        q,[eax][10]   

fcomp        d,[0042E4E4]   

fstsw        ax   

sahf   

setbe        bl   

mov        eax,[ebp][-04]   

mov        [eax][08],bl   

test        bl,bl   

je        .00042E487
```

这东西实在是奸诈。

(1) 判断时间不是用的`GetLocalTime`之类，而是通过`FileTime`来取得当前的时间。原理类似于先更新一个文件，然后取得文件日期，这样来得到系统当前的时间。

(2) 内部还有一些判断，比如判断时间是否超了`2002`年等等。

(3) 这种判断的地方很多，只要有一个地方没通过，就会把内部的一些变量用`free`方法删除，这样就会造成运行时出错。

(4) 安装程序会创建楼上所说的键值，里面有一个类似`GUID`的东西，里面是安装时间和校验和。

这东西`bug`多多，最好不用`:)` `4dd466d0e1e0cc123afbe504da2462ca`
