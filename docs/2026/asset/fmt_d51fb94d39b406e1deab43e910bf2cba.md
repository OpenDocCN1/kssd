
- `WldpIsDynamicCodePolicyEnabled` - 验证是否执行了用户模式代码完整性（UMCI）和动态代码安全（即“Enabled:Dynamic Code Security”选项）。

- `WldpSetDynamicCodeTrust` - 在文件上设置“$ Kernel.Purge.TrustClaim”NTFS扩展属性。

- `WldpQueryDynamicCodeTrust` - 验证是否在文件上设置了“$ Kernel.Purge.TrustClaim”NTFS扩展属性。

那么为了避免.cs [竞争条件劫持攻击](https://bbs.kanxue.com/elink@29cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8E0L8$3&6V1j5i4W2Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6N6Q4x3V1k6T1P5i4m8S2M7%4y4A6L8X3N6Q4x3X3c8V1k6i4k6A6j5$3g2Q4x3X3c8Y4N6h3q4J5k6q4)9J5k6s2N6A6N6r3S2Q4x3X3c8V1L8%4c8F1k6i4c8Q4x3X3c8E0k6i4c8Z5L8$3c8K6i4K6u0W2K9s2c8E0L8l9`.`.) ，最终设置和读取文件扩展属性的目的是什么呢？ 好吧，它的用处是以便可信用户模式代码可以将生成的.cs文件标记为可信，然后之后可以将其验证为来自可信进程。 为扩展属性使用“$ Kernel.Purge”前缀的好处是，如果文件被覆盖，内核将自动去除扩展属性。 这意味着博文中描述的劫持.cs文件的行为将强制删除扩展属性，使文件“不可信”。从表面上看，这似乎应该是一个很好的缓解措施...前提是缓解应用正确操作 - 即确保将正确的文件标记为受信任，并且不该被标记为可信的文件不应当受信任。

既然我已经很好地理解了事情的原委，现在我已经可以研究潜在的攻击面。

## 攻击面分析

想要绕过我现在非常熟悉的“Dynamic Code Security”缓解措施，我问自己以下问题：

1. 是否可以对任意文件调用`WldpSetDynamicCodeTrust`？例如，如果我劫持.cs文件，我是否可能以某种方式修改代码（如`MarkAsTrusted`方法）将攻击者的文件标记为受信任？

2. 是否有文件信任未经过验证或未经过正确地验证？如果是这样，我是否可以更改代码流使得攻击者的文件不会发生被验证。

3. 我是否可以以某种方式让宿主进程要么不将`wldp.dll`加载到当前进程中，要么提供不导出动态代码信任函数的`wldp.dll`版本？

4. 我是否可以更改`WldpIsDynamicCodePolicyEnabled`返回动态代码安全未执行的状态？这可能是阻力最小的路径，因为如果未启用该功能，则不会执行文件验证。这个问题将是我攻击研究的第一个焦点。

鉴于我目前对缓解实施的理解以及我对扩展属性实现的理解，我认为这些问题已经很全面了。但作为研究人员，不应该声称已经考虑过所有可能的攻击面。你作为一名攻击性研究者，本身就受到了知识和创造力的限制，这两者以后都有可能扩展并使你认识到以前从未了解到的攻击面。

## 试图绕过`WldpIsDynamicCodePolicyEnabled`

考虑到有许多与动态编译C＃代码和最终信任编译文件相关的不确定因素，因此明智地投入时间非常重要。所以，为了确定如何投入时间来试图绕过`WldpIsDynamicCodePolicyEnabled`，我需要考虑绕过缓解措施的最终操作目标 - 强制C＃编译过程加载我的不受信任的DLL。考虑到这个目标，阻力最小的路径是查看编译过程的最后阶段 - 在编译的DLL上调用`System.Reflection.Assembly.Load`（byte []）。这让我找到了`System.CodeDom.Compiler.FromFileBatch`方法：

```
if (!FileIntegrity.IsEnabled)



{



    compilerResults.CompiledAssembly = Assembly.Load(array, null, options.Evidence);



    return compilerResults;



}



if (!FileIntegrity.IsTrusted(fileStream2.SafeFileHandle))



{



    throw new IOException(SR.GetString("FileIntegrityCheckFailed", new object[]



    {



        options.OutputAssembly



    }));



}



compilerResults.CompiledAssembly = CodeCompiler.LoadImageSkipIntegrityCheck(array, null, options.Evidence);



return compilerResults;
```

因此，如果未启用“FileIntegrity”，则DLL将通过常规`Assembly.Load`方法加载。如果可能的话，我很乐意改变代码路径。怎么做？我必须看看“IsEnabled”属性是如何填充的，代码如下（之前也展示过）：

```
private static readonly Lazy<bool> s_lazyIsEnabled = new Lazy<bool>(delegate()



{



    Version version = Environment.OSVersion.Version;



    if (version.Major < 6 || (version.Major == 6 && version.Minor < 2))



    {



        return false;



    }



    bool result;



    using (SafeLibraryHandle safeLibraryHandle = SafeLibraryHandle.LoadLibraryEx("wldp.dll", IntPtr.Zero, 2048))



    {



        if (safeLibraryHandle.IsInvalid)



        {



            result = false;



        }



        else



        {



            IntPtr moduleHandle = UnsafeNativeMethods.GetModuleHandle("wldp.dll");



            if (!(moduleHandle != IntPtr.Zero) || !(IntPtr.Zero != UnsafeNativeMethods.GetProcAddress(moduleHandle, "WldpIsDynamicCodePolicyEnabled")) || !(IntPtr.Zero != UnsafeNativeMethods.GetProcAddress(moduleHandle, "WldpSetDynamicCodeTrust")) || !(IntPtr.Zero != UnsafeNativeMethods.GetProcAddress(moduleHandle, "WldpQueryDynamicCodeTrust")))



            {



                result = false;



            }



            else



            {



                int num = 0;



                int errorCode = UnsafeNativeMethods.WldpIsDynamicCodePolicyEnabled(out num);



                Marshal.ThrowExceptionForHR(errorCode, new IntPtr(-1));



                result = (num != 0);



            }



        }



    }



    return result;



});
```

此代码段的工作流程如下：

1. 通过调用[`LoadLibraryEx`](https://bbs.kanxue.com/elink@c5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3#2K6y4U0R3@1x3e0M7&6i4K6t1#2x3U0S2$3i4K6y4p5N6Y4y4Q4x3X3f1^5y4g2)9J5y4e0t1&6i4K6u0W2j5i4y4H3P5l9`.`.)确保将`wldp.dll`加载到当前进程中。为了清楚起见，必须将`wldp.dll`加载到进程中，因为它是实现我们之前逆向的“DynamicCode”函数的DLL。如果未将`wldp.dll`加载到进程中，则无法进行动态代码验证。攻击者能够以某种方式迫使`wldp.dll`无法加载到当前进程中吗？对！在正常情况下很容易，只需通过将`wldp.dll`复制到与执行程序相同的目录来强制执行Windows Defender Application Control，在PE文件中翻转一个无关紧要的位（例如，Rich头中的位），修改`wldp.dll`的签名为无效，就能导致无法加载。为了缓解这种攻击情形，这就是使用2048 flag调用`LoadLibraryEx`的原因，该flag引用了`LOAD_LIBRARY_SEARCH_SYSTEM32`选项。这个明显的选项覆盖默认的DLL加载顺序并首先从`%windir%\System32`加载`wldp.dll`，从而缓解我刚才描述的攻击。嗯，更具体地说，攻击被非管理员缓解了。管理员可以通过修改`System32`目录中的`wldp.dll`来执行此攻击。

2. 确保`wldp.dll`导出执行动态代码验证所需的功能：`WldpIsDynamicCodePolicyEnabled`，`WldpSetDynamicCodeTrust`和`WldpQueryDynamicCodeTrust`。考虑到这些是`wldp.dll`中的新函数，并非所有版本的Windows都具有这些功能。顺便说一下，这里还有另一种攻击情形。攻击者可以在当前目录中提供较旧版本的没有这些函数的`wldp.dll`。这会绕过验证，对吗？不！ `LoadLibraryEx`再次起到了救火作用。

3. 调用`WldpIsDynamicCodePolicyEnabled`，如果它指示启用了动态代码策略则会返回True。现在，我从未逆向过内核如何确定代码完整性策略中是否启用了动态代码安全，可能还存在别的值得探索的攻击面，我会把它作为练习留给那些好奇的人。

此刻，我不相信我可以绕过“FileIntegrity.IsEnabled”验证。接下来是什么？请看下一节。

## 试图绕过`WldpQueryDynamicCodeTrust`

所以，假设我无法绕过“FileIntegrity.IsEnabled”检查，我只需进入`FromFileBatch`方法的下一行：

```
if (!FileIntegrity.IsTrusted(fileStream2.SafeFileHandle))



{



    throw new IOException(SR.GetString("FileIntegrityCheckFailed", new object[]



    {



        options.OutputAssembly



    }));



}
```

这段代码会抛出异常，如果未标记为受信任，则不加载已编译的DLL。我意识到这可能听起来多余，但我更喜欢做出以下逆逻辑陈述：如果文件被标记为受信任，则不会抛出异常。用这种方式陈述的逻辑使得作为攻击者的我更明了了，我应该尝试找到一种方法来更改代码以将编译的DLL标记为可信。验证可行性的第一步是找到标记编译的DLL为可信的代码 - 即在文件上调用`WldpSetDynamicCodeTrust`。 .NET中没有该代码，因此我查看了`csc.exe`。它也没有，所以我认为该代码可能存在于由`csc.exe`加载的DLL中。为了验证，我在`powerhell.exe`启动时将`WinDbg`附加到`csc.exe`，为`wldp.dll`加载时设置断点（`sxe ld wldp`），然后在`WldpSetDynamicCodeTrust`上设置断点。我只找到了一次断点。我找到了`PEWriter::writemscorpehost.dll`中的函数。

幸运的是，该`PEWriter::write`函数是[开源的](https://bbs.kanxue.com/elink@cceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3V1k6U0L8%4u0W2j5$3I4J5i4K6u0r3j5X3I4G2j5W2)9J5c8U0t1H3x3U0M7#2j5h3p5$3y4o6N6U0y4e0M7K6x3$3u0U0y4h3t1I4z5e0t1&6j5$3u0S2x3$3k6V1x3e0l9&6y4r3x3$3y4$3k6T1x3h3c8Q4x3V1k6K6M7X3y4Q4x3V1k6V1L8r3I4K6i4K6u0r3L8i4y4U0L8%4u0H3k6g2)9J5c8Y4m8W2N6%4u0A6N6r3g2J5i4K6u0W2j5%4m8H3i4K6t1K6e0o6t1I4y4K6W2Q4x3X3c8x3x3U0t1%4y4R3`.`.)。不幸的是，调用`WldpSetDynamicCodeTrust`的最新版本还没有发布在GitHub上。没关系。旧版本源代码使IDA反汇编更加容易。这是调用`WldpSetDynamicCodeTrust`的代码部分：

![](https://bbs.kanxue.com/upload/attach/201902/93_NHPAY3KPD8DM8CJ.jpg)

`PEWriter::write`函数调用`WldpSetDynamicCodeTrust`

所以这里的攻击场景需要在调用`WldpSetDynamicCodeTrust`之前覆盖已编译的DLL。这是不可能的，但是因为`PEWriter::write`包含DLL的句柄，并且在句柄处于hold状态时，任何其他进程将无法将之覆盖。从获取句柄开始将DLL写入磁盘到调用`WldpSetDynamicCodeTrust`的过程中，不会释放句柄。此外，`mscorpehost.dll`使用与`System.dll`相同的`LoadLibraryEx`缓解措施来调用`wldp.dll`函数，从而防止上一节中描述的攻击。

我之前提到过，也可以劫持生成的`cmdline`文件并删除`/EnforceCodeIntegrity`开关。我能够成功劫持文件并删除开关，但最终，这并没有输出任何有价值的东西。删除开关的副作用是编译的DLL不会被标记为受信任，但`FromFileBatch`方法需要文件被标记为受信任。因此，当`FromFileBatch`验证DLL的可信度时，它将不会被标记为受信任，并且将抛出异常。

## 尾声

在评估“动态代码安全”缓解措施上，仍有可能存在尚未探索出来的攻击面。我特别关注的是使用不安全的`Assembly.Load`（byte []）方法可以加载未签名的DLL。在某处可能存在攻击性的绕过方法，目前并不明了。除了前面提到的程序集引用bug之外，我还要赞扬微软在缓解竞争条件绕过上投入了大量精力。

## 结论

所以，经过所有这些努力，我找到绕过方法了吗？ 并没有。我作为逆向人员和研究员失败了吗？也没有失败一说！在整个过程中，我学会了如何实现我所认为有效的缓解措施（基于我目前的知识/创造力）。我也在这个过程中使我的攻击性研究方法变得更成熟。另外，如果你不先看bug，怎么会发现bug呢？对我来说，寻找bug不仅仅是寻找bug，也满足了我的好奇心。

我花了很多时间记录整个过程和我的方法，所以我希望你觉得它有趣，有教育意义，并能有所激励。我也希望这篇文章能激励你讨论并记录你针对特定问题的研究方法。有什么更好的方法能够帮助那些会被过程吓倒的人呢？！

此外，希望微软修复这个愚蠢的程序集引用bug。如果不修复，那么那些看起来需要耗费很大精力才能实现的缓解措施将没有任何意义。

谢谢阅读！

原文链接：a51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4y4@1M7#2)9J5k6i4y4H3k6h3y4@1k6i4u0G2M7s2y4Q4x3X3g2A6L8#2)9J5c8X3c8G2j5%4g2E0k6h3&6@1K9h3&6Y4i4K6u0V1j5h3&6V1i4K6u0V1j5i4c8@1j5h3y4C8K9h3&6Y4i4K6u0V1j5g2)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8V1k6h3k6W2L8X3c8W2M7W2)9J5k6r3q4H3M7r3I4A6j5$3q4@1K9h3!0F1i4K6u0V1j5$3!0F1N6s2u0G2L8q4)9J5k6r3k6W2j5i4c8#2M7X3g2Q4x3X3c8@1K9r3g2Q4x3X3c8Z5j5i4u0V1i4K6u0V1N6$3q4&6i4K6u0V1j5g2)9J5k6r3y4S2M7$3g2Q4x3X3b7%4x3$3c8V1x3h3f1I4x3h3u0W2x3$3p5`.

编译：看雪翻译小组 SpearMint

校对：看雪翻译小组 skeep
