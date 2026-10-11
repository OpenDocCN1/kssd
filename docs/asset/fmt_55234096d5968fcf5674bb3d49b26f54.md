
# Xenocode Fox 2007 Community Edition 主程序脱壳及网络验证爆破

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-629.htm`](https://www.pediy.com/kssd/pediy08/pediy8-629.htm)

`Xenocode` 是 `.net` 下非常著名的软件加密公司，`Fox` 是其产品之一，是一个反汇编器（另外还有个 `postbuild` 是加密的）。`Fox 2007` 的功能包括查看，分析，及 `Profile .Net` 程序。可惜免费的 `Community` 版不提供 `Profile` 的功能，`Professional` 版的网上又没的下。不过无所谓，`PEBrowse` 里连 `Profiler` 的源代码都提供了，为了这个花钱也不值得。`Fox 2007 CE` 的下载地址是 `http://www.xenocode.com/Products/Fox/Setup.msi`，让我们看看这款著名的软件其最新版的保护方式吧。

安装后目录下就 `Fox.exe` 这一个程序，用 `Reflector` 打开，发现错误，不含 `CLR` 头。不是吧，现在这么流行用 `win32` 加密 `.net` 程序吗？

![](img/2c19de93347f4e6127b5a9a9371d95f0.jpg)

用 `PEiD` 查一下壳，显示 `UPolyX v0.5 *`。通常这是不可信的，好像我试过的被本地代码加密过的 `.net` 程序查出都是这个壳。就算是，偶也不会脱，哈哈。用 `OD` 载入，果然是本地代码程序。试着跟了一会，没有头绪。还是从 `.net` 下 `PE` 执行的原理着手吧。

过去讲过，`.net` 下加壳分两种，一种是像 `MaxtoCode` 那样，分 `Method` 加密解密，在内存中不存在完整的 `assembly`。另一种是用传统的加壳技术，解密后原来完整的 `assembly` 会出现在内存中，这就给我们 `dump` 之提供了先决条件。`Fox` 用的是第二种。可是断点下在哪儿呢？就下在“万能断点”，`mscorjit.dll` 的 `compileMethod` 处，在我的机上是 `7906E7F4`。在 `OD` 中下断，`bp 7906E7F4`。这里要注意，就是在入口点下断会显示地址无效，因为 `mscorjit.dll` 还没有被载入，运行一段后就可以下断了。

当第一次中断时，我们看堆栈值

```
0012EDE8        79E9776F  返回到 mscorwks.79E9776F
0012EDEC        790AF170  mscorjit.790AF170
0012EDF0        0012EF2C
0012EDF4        0012EFB8
0012EDF8        00107210
0012EDFC        0012F06C
```

原来讲过，第四行的值指向了 `struct CORINFO_METHOD_INFO_` 结构，该结构如下：

```
struct CORINFO_METHOD_INFO
{
    CORINFO_METHOD_HANDLE       ftn;
    CORINFO_MODULE_HANDLE       scope;
    BYTE *                     ILCode;
    unsigned                   ILCodeSize;
    unsigned short             maxStack;
    unsigned short             EHcount;
    CorInfoOptions             options;
    CORINFO_SIG_INFO           args;
    CORINFO_SIG_INFO           locals;
};
```

我们在 `OD` 中这样选择：

![](img/abf7d597583a37ddfad773c10d92dcce.jpg)

然后在数据窗口中查看 `BYTE * ILCode` 的地址。

![](img/a20150e5415b581e670b4179610c6f1a.jpg)

至于这段代码的内容是什么，我们不关心，但这是 `JIT` 引擎编译的第一个 `method`，也就是说如果 `FOX` 是整个 `assembly` 解密，这时已经解密完成了。根据 `.net` 下 `PE` 结构的定义，`IL` 代码是定义在 `.text` 节中的，和 `win32` 下的 `PE` 差不多，在内存中也遵循 `rva` 偏移定位的原则。我们来到 `0138DA40` 所在的段：

![](img/59a4f32410e0f2dc51e675b020ad96cf.jpg)

双击，查看这段内存的数据，看到了什么？`PE` 文件头！难道这就是解密后的原文件？不妨 `dump` 下来看看。点右键，选“保存数据到文件”，然后把扩展名从 `mem` 改为 `exe`。呵呵，图标一下又变成 `fox` 了。

![](img/e9e2240856f9e8f2eac08d6a943b92c8.jpg)

用 `Reflector` 再次打开 `dump.exe`，这次 `Reflector` 能完全显示出来了。这就是 `Fox` 的本体。

![](img/78dfcf82049e1dfacb0d1e89a7712e92.jpg)

试着运行一下，报错。“`Unalbe to find a runtime version to run this application.`”。没事，这种小困难不算啥，我不会修复让微软帮忙。直接用 `ildasm.exe` 反编译，然后原封不动地再用 `ilasm.exe` 编译回来。`OK`，一切问题解决，程序可以正常运行了。

/////////////////////////////////////////////////////////////

下面解决网络验证问题。当你联网运行 `fox` 时，会出现个小窗口，提示你输入在 `fox` 网站的用户名和密码，进行验证。这可不爽，难道每次运行 `FOX` 我都得联网？而且还有天数限制。不过既然源代码已经出来了，就把网络验证块给爆了吧。

![](img/01a791a1d9fcc4e327e09dca74c3d801.jpg)

在 `Reflector` 中来到入口处，代码如下。（跟踪窗口的产生过程，用 `PEBrowse` 调试比较方便。如果哪个 `call` 你跳过去后，窗口就显示的话，就跟进去，直到找到网络验证的地方，就像 `OD` 中的 `F7,F8`。我已经调试过了，所以这里直接给出静态的流程。）

```
[STAThread]
private static void xc447809891322395()
{
    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);
    x3e4e23fadc83a77e xeefadcae1 = new x3e4e23fadc83a77e();
    while (xeefadcae1.x69b9cb33c60f0e3b)
    {
        Application.Run(xeefadcae1);
        if (2 != 0)
        {
            return;
        }
    }
}
```

注意红色的那句 `x3e4e23fadc83a77e()`，直接跟进去。来到一个很大的代码段。先通篇浏览一下，没有出现任何和网络验证相关的函数调用，看来还要跟进去。真正调用的是下面代码中加红色的一段（同样，这是动态调试跟踪出来的）：

```
Label_06C4:
    this.x85601834555fb7d5();
    base.Height = Screen.PrimaryScreen.WorkingArea.Height;
    using (xf17d6cdd9cc7b699 xfdcddccb1 = new xf17d6cdd9cc7b699(0xfa0))
    {
        xfdcddccb1.ShowDialog(this);
    }
    this._x8af6efb682bc47ce = new x8af6efb682bc47ce();
    using (x02e1e2a3949fc710 xeeafc1 = new x02e1e2a3949fc710())
    {
        xeeafc1.Text = x77fa6322561797a0.x6886d5a1867d55cb;
        xeeafc1.Icon = base.Icon;
        xeeafc1.x7598bfd06959c5c4 = x77fa6322561797a0.x6886d5a1867d55cb;
        xeeafc1.x77fa6322561797a0 = x77fa6322561797a0.x4cdaa3594901b8ae;
        x02e1e2a3949fc710.x658c509a55e4e71a xcaeea1 = xeeafc1.x59b56fb51064be4a(this);
}
```

再次点 `x59b56fb51064be4a(this)`，跟进去。终于，我们看到了“`UserSerial`”，和 `WebRequest`，这明显是网络验证的代码段了。

到这里一切很明显了，下面的分析一切从简。首先在 `Software\Oak Vale Networks` 这项中新建 `UserSerial` 项，然后随便输入注册码。然后爆破三个地方。爆破时在反编译出的 `il` 文件中修改。

第一处，`il` 文件的 `354889` 行

```
IL_00b8:  /* 33   | 5D                    */ bne.un.s   IL_0117
```

改为 `beq.s`

第二处，`il` 文件的 `354853` 行

```
IL_0075:  /* 2D   | 51                    */ brtrue.s   IL_00c8
```

改为 `brfalse.s`

第三处，

```
IL_00d0:  /* 2C   | 13                    */ brfalse.s  IL_00e5
```

改为 `brtrue.s`

改完后保存，用 `ilasm.exe` 编译，命令行如下：

`ilasm /resource=dump.res dump.il`

完成后运行，`OK`，一切搞定。即使联网也不会提示激活了。

最后说几句，对 `dump` 后的 `fox.exe` 进行动态调试是很重要的一环，因为看着网络验证函数的返回值调试起来非常方便。比如，在 `PEBrowse` 中可以清楚地看到验证服务器的返回值。如下图

![](img/d0428ff9a7ab6eb83f6fa6ba7444e180.jpg)

这句是建立一个读写流，读取网络验证的返回值，保存在 `eax` 中。我们双击 `eax`，就会看到服务器返回的验证值。这里返回的是“`FAIL`”，失败。还有的值包括“`BAD`”，“`EXP`”过期等。这样结合静态分析，看代码是非常方便的。 

`4dd466d0e1e0cc123afbe504da2462ca`
