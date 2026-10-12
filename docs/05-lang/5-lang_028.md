
# 修改 `System.dll` 绕过 .NET 的 License

> 原文：[`www.pediy.com/kssd/pediy12/130183.html`](https://www.pediy.com/kssd/pediy12/130183.html)

前两天同学给我发了个 .NET 编写的 CPU 优化的程序，由于注册机制无法正常使用，他寻思看我能不能有什么解决办法。以前搞过一些简单的 crackme，但对 .NET 程序的机制还很陌生。破解过程当中遇到了很多问题，发个帖子叙述一下具体情况，当中一些遗留问题也希望能得到达人们的指点~![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

![](img/4d28e802273a22d94f6fd03e1f6dd144.jpg)

解压后就这些文件，`DLLInject` 是主程序，`HID` 是机器码文件生成用的，那个 dll 文件至今不知道做什么用的（猜测可能是优化 CPU 时用到）。直接运行 `DLLInject` 会弹出需要注册的对话框。

![](img/f222153112ead5cd59ed924bc41087ae.jpg)

这种对话框乍看来只有许可证文件存在程序才能正常运行，点击 OK 程序就退出了。后来才查到这样的验证机制是 .NET 本身提供的 `System.ComponentModel.LicenseXXX`。.NET 许可证机制提供了框架，具体的许可证校验算法肯定还是作者自己实现的，以配合硬件机器码完成验证。

![](img/a0e301f0565ae0fefa9e7e9b90d04a20.jpg)

用 `Reflector` 无法正常载入，看来文件还被特意修改过啊，解决方法：`http://hi.baidu.com/dreamzgj/blog/item/3c0bc18e243592f0513d928a.html`。不过马上又出现了新的错误：`Reflector Error:Module '…' contains zero or multiple module definitions`。解决方法：`http://hi.baidu.com/dreamzgj/blog/item/5cd5f1456fedc388b2b7dc8a.html`。都是同一个人写的文章。这回 `Reflector` 载入虽然还是提示数组越界的错误（看来还有其他被修改的小地方），名称也都被混淆过（另外用 `StrongNameRemove` 根本不能正常载入该程序），但起码可以看个大致结构了，`entrypoint` 函数（`S9oGE4FMD`）还是无法显示。

![](img/6763f31da2a4ba6095286e9e3e0c3b60.jpg)

其他很多函数内容都是形如下式，都是一些看起来毫无作用的垃圾信息。估计是干扰函数或者是运行时动态修复吧。

![](img/0cea6ffe77aa3f5326a335e385e612cc.jpg)

用 `ILDASM` 把代码导成 il 形式看入口函数终于能正常查看了，也是为数不多的能看到一些语句的函数（其他函数都是返回个 null 值就没了）。

代码:

```
.method assembly static void S9oGE4FMD(string[] A_0) cil managed noinlining
{
  .entrypoint
  .custom instance void [mscorlib]System.STAThreadAttribute::.ctor() = ( 01 00 00 00 ) 
  .custom instance void [System]System.ComponentModel.EditorBrowsableAttribute::.ctor(valuetype [System]System.ComponentModel.EditorBrowsableState) = ( 01 00 02 00 00 00 00 00 ) 
  .custom instance void [mscorlib]System.Diagnostics.DebuggerHiddenAttribute::.ctor() = ( 01 00 00 00 ) 
  // Code size      50 (0x32)
  .maxstack  3
  IL_0000:  br         IL_0007
  IL_0005:  pop
  IL_0006:  ldc.i4.0
  .try
  {
    IL_0007:  call       bool [Microsoft.VisualBasic]Microsoft.VisualBasic.ApplicationServices.WindowsFormsApplicationBase::get_UseCompatibleTextRendering()
    IL_000c:  call       void [System.Windows.Forms]System.Windows.Forms.Application::SetCompatibleTextRenderingDefault(bool)
    IL_0011:  ldtoken    EIOtYCk427B2j46XCZ.aC4npiTU1ygUa7KS3q
    IL_0016:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
    IL_001b:  call       void [System]System.ComponentModel.LicenseManager::Validate(class [mscorlib]System.Type)
    IL_0020:  leave      IL_0026
  }  // end .try
  finally
  {
    IL_0025:  endfinally
  }  // end handler
  IL_0026:  call       class EIOtYCk427B2j46XCZ.aC4npiTU1ygUa7KS3q usJHEvC4iTtNdZDPye.bhVRye8yqVI1niw4o6::vaCk4npiU()
  IL_002b:  ldarg.0
  IL_002c:  callvirt   instance void [Microsoft.VisualBasic]Microsoft.VisualBasic.ApplicationServices.WindowsFormsApplicationBase::Run(string[])
  IL_0031:  ret
} // end of method aC4npiTU1ygUa7KS3q::S9oGE4FMD
```

`Validate` 应该就是许可证验证函数了吧，不过网上说这种 .NET 许可证机制好像不能直接跳过去，因为后面会用其中的一些信息数据，这里一直没有弄明白，哪位了解这种许可证机制的给解释一下能不能直接跳过吧![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")。用 `PEBrowsePro` 跟入 `Validate` 并没有直接跳到 `System.dll` 空间，这点倒是和纯粹 PE 文件的函数调用大不相同啊，并且执行中总是提示各种执行异常。

无奈还是直接把 validate 函数去掉试试吧，不过得直接在二进制文件上改（`ILDASM` 导出的 `xx.il` 文件用 `ILASM` 虽然编译成功但执行时会弹出错误）。用 `IDA` 打开 exe 文件，找到两个函数的调用字节码的文件偏移，再用 `ultraedit` 打开 exe 填写 `0x00`（nop）覆盖对应位置。不过执行后还是会弹出许可证那个窗口，看来程序在运行中还多次调用了 `Validate` 函数，静态找不到其他的调用迹象，动态又不太顺利。

用 `DotNet_Id` 的查了一下壳，天啊![](img/e06ce445d64470eaf3496ba167dfb4ac.jpg "碌眼")，原来是加过壳啊，难怪分析起来这么费劲呢。

![](img/3ec931f8073554a6f141445142ae22f2.jpg)

`http://jithook.blogspot.com` 上面提供了各个版本的 `MaxToCode` 的脱壳工具，最新版本是 `RE-MaxV3.40`。`MaxToCode` 加壳后，会生成一个 dll 文件随着 exe 文件一起在运行时提供支持，脱壳工具要载入 dll 后才能脱壳。不过这个 `DLLInject.exe` 配的 `pe.dll` 看来并不是这个 `MaxToCode` 壳用的 dll（删了该 dll，`DLLInject.exe` 也能正常运行）。`MaxToCode` 支持把 dll 融入到 exe 中，如果它真的是加了 `MaxToCode` 的话那一定是融合了，但没有找到提取方法。看来之前载入错误，编译后不能运行都是这个壳导致的。

突发奇想![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")，我寻思想改掉 `System.ComponentModel.LicenseManager` 下验证函数本身应该就能绕过了吧。

![](img/1200b0be5c84993253401c9b348e162b.jpg)

先从 `Validate` 函数开始改起吧，两个版本的都要处理一下。ret 字节码 `0x2a`。在 `C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\system.dll` 文件的相应偏移位置找到对应字节码修理一下。

改前：

![](img/7ccd99db45743950ee5abbe128d1e565.jpg) 

![](img/8ec4713a50ac8a9a61d4ee845e8312bc.jpg)

改后：

![](img/7266c64e7fc6908d44e4e7926ea37694.jpg)

![](img/826fe61d2c5f6129895131b779a00345.jpg) 

由于第二个版本的 `Validate` 有返回值（类型为 `License`）所以在返回前要在把一个 license 放到堆栈上，再 `ret`。

执行程序仍旧弹框框，可问题是删掉 `system.dll` 还是弹框框！难道程序没有依赖 `system.dll` 吗！用 `Filemon` 监视了一下程序执行时访问文件的情况才发现，原来程序加载用的 `system.dll` 并不是 `c:\windows\Microsoft.net\...` 目录下的 `system.dll` 而是 `C:\WINDOWS\assembly\GAC_MSIL\System\2.0.0.0__b77a5c561934e089\system.dll`。

这样就把改好的 dll 替换一下，再运行程序，发现其他函数 `isvalid` 等已经不用改了，程序已经不会被那个许可证困扰了。

![](img/89d4b70f32d33f13d7cda8c2127ec844.jpg)

这回算是正常运行了吧（这点还真不敢确认，程序没发现有什么功能，该不会是和许可证有关吧），还有那个 `pe.dll` 的谜团都有待解开~希望哪位知道的话提示一下~总觉得改了系统的 `system.dll` 好像很不妥的方法，似乎绕过了验证。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | CPU 优化终身版.rar  
|
