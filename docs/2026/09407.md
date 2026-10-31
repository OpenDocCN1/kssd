
# 如何破解.NET Reactor

NET Reactor是EZIRIZ公司出品的一款.net保护软件，功能也相当强大，从3.3.1.1版本开始，他采用了新的加密方法，之前的版本如何破解，tankaiha已经发表过一篇文章，利用reflector+`ildasm`+`ilasm`，做起来非常简单，但这个方法在3.3.1.1版以后就不能用了。现在我就来介绍如何破解3.3.1.1版。

## 准备的工具

- .NET Reactor 破解他，当然要先安装先下载地址：http://www.eziriz.com/downloads/dotnet\_reactor\_setup\_3\_3\_1\_1.exe
- .NET Generic Unpacker 和 SNSRemover，用来脱掉 Reactor 本身的壳和去去掉强名称
- Reflector，这个不用说了吧…
- 十六进制编辑器，主要有查找、替换和保存功能就OK了。

## 脱壳并去除强名称

我们安装完Reactor，发现他不是.NET程序，直接用Reflector不能反编译他，这时候我们就要用.NET Generic Unpacker，将Reactor的壳脱掉如图1。

![](https://www.kanxue.com/pediy09-267.files/image001.png)

（图1 对Reactor进行脱壳）

每次脱出来的数量都不一定相同的。好了，我们用Reflector打开他看看，如图2

![](https://www.kanxue.com/pediy09-267.files/image003.png)

（图2 用Reflector对Reactor进行反编译）

这时候，我们发现混淆后的类名竟然是乱码，这个没所谓，`ilasm`是支持乱码的，是不是我们也可以用`ildasm`进行反编译，然后修改他的代码，然后用`ilasm`将他重新编译呢？理论上是可以，但实际上由于反编译出来的资源文件的文件名是乱码，`ilasm`没办法找到那些文件，编译会失败。那怎么办，那就直接修改原程序的了。

首先用SNSRemover去掉他的强名称先。

![](https://www.kanxue.com/pediy09-267.files/image005.png)

（图3 用SNSRemover去掉Reactor的强名称）

现在你是不是有这个疑问，本来脱出来的程序就已经不能运行的了，现在去掉了强名称后，反而运行起来会提示出错。不用怕，试一下用原版的Reactor对现在去掉强名称后的Reactor进行一次加密。将加密出来的程序放在Reactor目录下运行。

![](https://www.kanxue.com/pediy09-267.files/image007.png)

（图4 重新加密后运行时发生的错误）

哈哈，看到是什么错误没有？现在这个已经不是验证强名称错误了，是一个运算错误，是某数除于0时时发生的错误。再看看他发生错误是在那里，我们通过Reflector来找出`Cv. Cc..cctor()`这个方法,由于Reactor他本身是经过混淆的，已经不能用C#来反编译，新版也做出了新的混淆，就算去掉了

```
L_0000: br L_0007
L_0005: pop
L_0006: ldc.i4.0
```

这三段代码，也不能用C#来反编译，我们只能用IL了。

![](https://www.kanxue.com/pediy09-267.files/image009.png)

（图5 用Reflector找出发生异常的方法）

熟悉IL的就会知道，除的命令是`div`，那我们就在这里里面搜索`div`，发现这段代码：

```
L_004f: ldc.i4 0x10
L_0054: stloc.s num
L_0056: ldloc num
L_005a: ldloc num
L_005e: sub
L_005f: conv.u1
L_0060: stloc.s num
L_0062: ldloc num
L_0066: ldloc num
L_006a: div
L_006b: conv.u1
L_006c: stloc.s num
```

看到这里应该知道了吧,将他换成C#的代码应该是

```
num = 0x10;
num = num - num;
num = num / num;
```

知道错误的原因了，那我们将这个`div`改掉就行了，但有一个问题，为什么原版的程序就不会发生这个问题？原因很简单，你搜索一下`GetPublicKeyToken()`，你就会发现上面那段代码是他验证强名称失败时才会执行的，因为我们去掉了强名称，所以肯定会执行那段代码的。

我们知道了出错的原因了，但我们怎样改呢？方法很简单，因为Reflector他有提示该代码对应的十六进制，

![](https://www.kanxue.com/pediy09-267.files/image011.png)

（图6 找出该代码对应的十六进制）

这时候我们是不是该想一下，其他的方法里是不是也同样也有这样的验证。好，我们随便找几个方法，发现有些方法是有，有些是没有。但有些的IL代码不一样，有点区别如：

```
L_003b: ldc.i4 0x24
L_0040: stloc.s num
L_0042: ldloc.s num
L_0044: ldloc.s num
L_0046: sub
L_0047: conv.u1
L_0048: stloc.s num
L_004a: ldloc.s num
L_004c: ldloc.s num
L_004e: div
L_004f: conv.u1
L_0050: stloc.s num
```

但运行出来的效果是一样的，只不过是他对应的十六进制不一样而已

![](https://www.kanxue.com/pediy09-267.files/image013.png)

（图7 ldloc.s对应的十六进制）

好了，如果我们一个个方法都要去看，那花的时间太多了，不如我们先处理掉一部分先，如果再发现那里的错误，我们就去那里找出来。

我们很容易通过Reflector可以知道这两段代码对应的十六进制应该是

`FE0C0000FE0C000059D21300FE0C0000FE0C00005BD213`和`1100110059D21300110011005BD213`

从Reflector那里我们可以知道，`sub`对应是`59`，`div`对应是`5B`，那我们将`5B`换成`59`那程序就不会发生异常了，用十六进制编辑器，替换`FE0C0000FE0C000059D21300FE0C0000FE0C00005BD213`为`FE0C0000FE0C000059D21300FE0C0000FE0C000059D213`，替换`1100110059D21300110011005BD213`为`1100110059D213001100110059D213`

![](https://www.kanxue.com/pediy09-267.files/image015.png)

（图8 替换代码）

现在我们又用原版的Reactor重新加密一次我们刚处理完的文件。发现现在可以正常运行了。

## 将脱壳程序转为正式版

我们已经可以正常运行我们脱壳后的程序了，现在开始我们就要将他变成正式版。

用Reflector打开我们刚处理完的文件，使用Reflector自带的功能，跳到程序的入口点

![](https://www.kanxue.com/pediy09-267.files/image017.png)

（图9 找到入口点）

分析一下程序的，不难的可以找到

```
L_0662: call bool Cv.Cc::'2()
```

这段代码就是验证的代码了，但如果在这里直接修改，难度会相当大，不如我们修改`'2()`的返回值，只要他永远返回`true`，那就达到我们的目的了。

![](https://www.kanxue.com/pediy09-267.files/image019.png)

（图10 来到‘2()）

我们不难的找到了`L_0000: br L_0007`对应的地址是`0x17cd28`,将原来的`3802`改成`172A`

![](https://www.kanxue.com/pediy09-267.files/image021.png)

（图10 修改3802为172A）

保存后，我们在次用原版的Reactor对刚处理完的程序进行加密，将加密后的程序放在Reactor目录下运行。看看，现在是FULL VERSION了，但我们现在测试一下他，会发现出现这样的异常

![](https://www.kanxue.com/pediy09-267.files/image023.png)

（图11 修改完，运行出现的异常）

看一下他的异常，还是`System.DivideByZeroException`，也就是说还有一部分的强名称验证的代码还没有修改，只要重复上面的操作，找出他的十六进制，将`5B`换成`59`就行了。

该版本我已经发布了他的破解版，3.3.8.0也已经发布了(http://bbs.pediy.com/showthread.php?t=51325)，3.3.8.0的破解会比这个更难，有兴趣的朋友可以来研究一下，可以拿3.3.8.9版来试一下，这个跟3.3.8.0是一样的破解。..
