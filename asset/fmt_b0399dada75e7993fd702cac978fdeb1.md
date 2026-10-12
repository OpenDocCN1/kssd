
# .Net 强名称验证带本机代码混编译程序调试一例--强名去见鬼

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-279.htm`](https://www.pediy.com/kssd/pediy08/pediy8-279.htm)

`[文章标题]` `.Net` 强名称带本机代码程序调试一例

`[Subject]` Crack a `.Net` Assembly with Strong Name Signature and Native Code

`[Author]` FucKiNi (It's lccracker)

`[withTool]` `OD` + `Ms.Net SDK` + `Reflector` and so on

`[System]` `Win2003` + `.Net Framework 1.1` and `SDK`

`[SoftName]` a Commercial Program in Specified Field

`[Download]` N/A

`[Protect]` Native Code Mixed + Soft Dog Hardware

`[Descrip.]` This software is not protected with S/N or Lience but Soft Dog Hardware. It's a really bad thing that we can't enter any Lience by any way to make it run corectly. This article will show you how to make it working.

`[Disclaim]` Only tell what I had done, no any warranties of any kind for responsibility.

------------------------------------------------------------------------

手头有一个行业软件，是需要插上加密狗才能正常运行的，`C#` / `.Net Framework 1.1` 环境开发的。

这是我们公司购买的正版软件，所以是能正常使用的，但是由于电脑多，加密狗难免要拔来拔去的。

闲来无事，就研究研究吧，也好多学些调试知识。

`PEiD` 可以“鉴定”出该软件是 `Microsoft Visual C# / Basic .NET`，新手不要以为脱壳查壳才用到 `PEiD` 啊。

那就用 `Reflector` 分析吧，注意到不插加密狗会弹出个对话框提示“`*****未注册*****`”的字样，部分功能受限。

按 `F3` 显示搜索窗口，输入“未注册”，点其右侧“`String Seach`”图标进行字符串搜索。

找到一项，是程序启动的一个模块，双击来到左侧树状列表，再双击打开代码，我选的 `C#` 格式。一般我是 `IL` 和 `C#` 两种结合对照使用。鉴于诸多原因，代码已经精简并修改，特此说明。下同。

```csharp
GlobalVariant.zhuce = Check.Textxyz;
if (!GlobalVariant.zhuce)
{
    MessageBox.Show(this, "*****未注册*****");
}
```

一看，里面没有一个汉字，用许多“`\u7237\u7016`”之类的字串，这就是汉字的 Unicode 了。

把模块代码复制出来，找个 Unicode 转换工具转一下，现在一目了然了，直接找到弹出“`*****未注册*****`”的条件判断，该判断调用了 `some.dll` 中的一个过程 `Check.Textxyz`，点击过程名字跟踪过去，点开如下：

```csharp
public static bool Textxyz
{
    get
    {
        return ((Check() != 0) ? 1 : 0);
    }
}
```

再点击追查 `Check()` 函数，除了定义什么都没有，已经是传说中的 Native Code 了(本机代码)。`[后来反编译才发现的]`

```csharp
[PreserveSig, MethodImpl(MethodImplOptions.Unmanaged, MethodCodeType=MethodCodeType.Native), SuppressUnmanagedCodeSecurity]
public static int modopt(CallConvCdecl) Check();
```

但是无所谓了，分析一下，只需修改上面那个 `Textxyz` 中的

`return ((Check() != 0) ? 1 : 0);` 根据判断返回 `1(true)` 或 `0(false)`，应该是检测软件狗的 Native Code

为：

`return ((Check() != 0) ? 1 : 1);` 不管怎么样都返回 `1(true)`

嗯，方案确定，那就干吧。。。。

程序有 5 个 DLL 和 2 个.EXE，其中一个.EXE 不是 C# .Net 程序(Assembly)，只需解决这 5 个 DLL 和 1 个 EXE 就完了。简单！

全部 `ildasm` 出来，将 `some.il` 的代码根据上面改法做了修改，很简单，就是把那段里的 `ldc.i4.0` 改为 `ldc.i4.1`

```
L_0009: ldc.i4.0      /这里改成 ldc.i4.1
L_000a: br.s L_000d
L_000c: ldc.i4.1
L_000d: ret
```

全部去掉强名称，用 `ilasm` 编译回去。。。。。。。

到 `some.il` 卡壳了，我#@#@$!%$#&^$%\*    还以为胜利在望了呢。

这才注意到 `some.dll` 使用了本机代码(Native Code)。`some.dll` 中还有许多.Net 代码的软件功能。

大家一致认为，混合了本机代码的 Assembly 是无法重新编译的，因为无法反编译（不信你试试;)~）

唉，怎么办，查资料，眼都看晕了，结果还是 NO WAY。

PS 看来使用本机代码混编译.Net 是保护.Net Assembly 程序很强的方法。

只能 16 进制编辑了，弄了一份 `some.dll`，对照 `some.il` 找到特征字串，在编辑器中搜索到以后

`16(false)` 就改成了 `17(true)`。

这里 `some.il` 在用 `ildasm` 导出（即转储）时一定要把那几个方框打上勾，这样反编译出来的代码就有 IL 指令对应的 `some.dll` 中的 16 进制代码。很方便去编辑器中搜索定位。

保存后打开软件，看看能不能运行－－－弹出公共语言运行库错误提示！

运行 `.Net Framework SDK 1.1` 的 `DbgCLR` 进行调试，查到是强名称认证出了问题！

对有强名称的 Assembly.Net 程序，只要用 16 进制编辑器修改了任何字节，都会导致强名称验证失败而使程序无法运行。(不知道修改 header 会不会也这样，没试)

解决强名称认证问题的常规方法就是重新编译程序，可是刚才就试过了，不行！

反编译带本机代码的 Assembly？搜索和研究了好久没找到解决方案。

那么，能不能将全部 DLL/EXE 强行去掉强名称，试了一下不能运行，而且也不可能加载到 GAC 中运行。

那么，怎么根据算法给 `some.dll`“更新”成修改后的正确强名称？没有答案。这一点我很感兴趣，以后还会留意。

没辙了，最后一招：怎么欺骗 Microsoft.Net 让它不去检查强名称，或者让它怎么看都是对的？

网上找到写注册表的方法，但我试了几次都不行，怀疑是不是需要重新启动一下电脑？？没再试。

干脆，把 `.Net Framework 1.1` 的这个强名称机制破解了算了，我是没看到它到底有什么用。

破解系统的强名称验证机制：

结合网上找到的资料，StrongName 强名称相关的操作在 `.Net Framework` 运行库的文件 `mscorsn.dll` 里，具体位置，一般在系统目录 `C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\` 下，自己找。

涉及到的函数是 `StrongNameSignatureVerification` 和 `StrongNameSignatureVerificationFromImage`。

于是复制了一份 `mscorsn.dll`，用 `OD` 打开，先点运行让它运行一遍（管它运行不运行呢），然后点菜单中 调试 --> 调用 DLL 输出，找到相关的函数并参照着找到函数入口点，修改一下让它们直接返回一个 True。

修改保存，覆盖掉系统原来的那个文件，注意先备份一下以防修改失败或事后系统恢复。

覆盖时，要先覆盖 `Windows/System32/dllcache/` 目录下的那个，然后再覆盖 `Windows/Microsoft.NET/` 下的目录下的那个。

现在点击已经修改好的程序看看怎么样了？

运行成功！无异常。

同事说我把这个软件和.Net 一块给 Crack 了。

只可惜要替换 `.Net Framework` 的系统文件 `mscorsn.dll` 才行，安全性是否下降尚不知道。

这个系统文件的替换做法，在 `.Net Framework 2.0` 中应该是一样可以对付强名称的。

=======================

最后总结： （很重要）

=======================

1. 如果你都看到了这里，证明你很有耐心，也很谦虚，麻烦不要挖苦我:-），请多多指点
2. .Net 程序(Assembly)的逆向研究难点就在 Native Code 混编，这个应该是研究方向
3. 不可逆强名称程序的强名称验证能否从正面解决能否“更新”强命名是不是不可完成的任务？
4. 这一点很重要：本文提出了一个强名称验证+编译本机代码保护程序的一个 Crack 思路，只要找到需要修改的关键点，只要能用 16 进制编辑器让它突破限制，那么程序也是可以照常使用的。只不过再给系统打个免强名验证补丁而已。虽然是有所得有所失，但反过来想想，对软件到了要 Crack 的地步的“强烈需求”，会在乎给系统打个并不是很令人讨厌的补丁吗?这个补丁怎么样，还需要大家大面积的试验，实践出真知。可能有人会说，你想的也太天真了，如果软件的核心功能和验证都是 NativeCode 那你怎么办？其实不用担心，因为你说的程序已经算不上.Net 程序了。只是披上.Net 外衣欺骗用户而已。为什么要使用.Net 开发程序？答案很多，开发快捷，功能强大易用(都是针对开发人员)等等等等，所以呢。。。如果用.Net 进行开发，肯定软件的核心功能是靠.Net NonNative Code 实现的，至于验证非法用户的核心功能，就写道 Native Code 中，然后混编一下。然后，这样的程序应该还是很容易 Crack 的。没有强名称验证，Native Code 作用有限。
5. 饿死了，去吃饭，回来再修改吧。

---------------------------------------

FucKiNi 2006.08.08.00:30

`4dd466d0e1e0cc123afbe504da2462ca`
