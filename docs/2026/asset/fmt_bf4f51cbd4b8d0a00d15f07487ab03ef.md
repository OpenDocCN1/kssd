
# [原创]静态分析 破解Spices.net v5.1 --- 试谈.net程序的破解

- 【文章标题】: 静态分析+代码片断+十六进制编辑 破解Spices.net v5.1 --- 试谈.net程序的破解
- 【文章作者】: dreaman
- 【作者邮箱】: dreaman\_163@163.com
- 【作者主页】: 101K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5k6h3q4E0j5h3&6Q4x3X3g2Z5j5h3!0D9K9h3&6B7N6g2)9J5k6h3&6W2N6l9`.`.
- 【软件名称】: Spices.net v5.1
- 【软件大小】: 6788KB
- 【下载地址】: 8d8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0W2J5j5i4W2K6i4K6u0W2L8X3g2@1i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3g2S2M7%4m8^5
- 【加壳方式】: 无
- 【保护方式】: 无
- 【编写语言】: 未知
- 【使用工具】: `reflector`, `research.net`, `dis#`, `SnippetCompiler`, `ildasm`, `ultraedit`, `hexworkshop`等
- 【操作平台】: winxp , .net 2.0
- 【软件介绍】: .net 程序反编译、.net PE结构观察、assembly校验及混淆的工具。
- 【作者声明】: 基于不太多的破解经验总结，错误之处在所难免，欢迎讨论。

本文主要谈以静态分析为主的破解，并试图探讨以静态分析为主的破解模式，我们主要针对纯CLR程序，也就是程序中不含native指令的.net程序，这样的程序通常用C#或C++/CLI的pure clr编译选项生成。

特别的，我们主要从没有加壳的程序说起，如果是加壳的.net程序，则首先需要脱壳，近期DRT小组的rick在.net MaxtoCode壳方面已经取得突破。从原理上说，.net程序因为要支持reflection与JIT编译，其关键部分“元数据与IL代码”都是可以取得或重建的，也就是说脱壳几乎总是可能的，应只在难度高低之分。当然，如果将来有一种壳自己实现完整的JIT功能甚至CLR，上面的说法就无效了，不过这样一来，是否仍是.net平台则需要怀疑了。

## 一、基本思路

1、现在大多数.net程序都是加了强名的，这样一来，我们没法直接修改程序了，所以我们的第一步便是解除强名。本来也有工具可以直接去除文件的强名，不过现在许多.net程序将强名与它的加解密结合起来，直接去除有时会倒致许多麻烦，spices.net 5019的keygen里提供了一个比较好的办法，就是自己重新生成一个新的强名，用新的强名的KEY与TOKEN替换原来文件中的强名的KEY与TOKEN，然后再用`sn.exe`强制为原文件签名。有了强名替换与重新签名后，我们就可以根据需要修改原文件了，只需要在修改完重新签上新的强名就OK了。

2、一般的.net程序通常会利用混淆软件将名称搞成不可见字符或是一长串不好记的字符，分析起来十分不便，我们的第二步就是反混淆，这里使用我自己写的一个小工具，因为现在只有`dis#`的反混淆比较好用，但它不能同时对一组assembly反混淆，而且它的反混淆是在反编译时进行的，反过后的代码不容易对应到原程序的方法（比如我们要用`ildasm`打开原程序，想找到`dis#`反混淆后的某个方法，会非常困难），我们自己的小工具是直接对原二进制文件进行修改，这样修改后的文件可以由多个工具使用，确保了多个工具间名称的对应。（这个工具会生成新的section，将.net元数据搬到新的section中再补上反出来的名称，这样我们通常将反混淆后的文件放在一个新的地方进行分析，实际打补丁则直接在未反混淆的文件上进行，分析的方法与原文件方法的对应可以通过方法表的token来对应或者通过方法体中的特征字符串进行）。

3、文件的强名改变后，文件中与强名相关的加密解密的数据也需要作相应处理，比如原来有一段数据用旧的强名加密的，则我们需要将原数据解密再用新的强名加密替换原来的数据，这是一项比较烦的工作。首先我们通过分析找到这样的地方，查找的主要方法是用`reflector.exe`的analyzer,通过查找对`AssemblyName.GetPublicKey`与`AssemblyName.GetPublicKeyToken`的引用来定位使用强名称的代码。

4、经历上面这些过程后，我们终于可以开始我们真正的破解了，相对来说，这部分倒似乎更简单一些，呵呵，对我们这次的目标Spices.net来说，主要就是去掉它的功能限制，因为是evaluation版本，没有写注册机的必要了。我们对反混淆的文件进行分析找出这些限制的地方，然后用`ildasm`反编译原始程序，找到对应的IL代码，然后用二进制查找与替换功能将原始程序的字节码改成我们修改后的代码。

5、完成所有的修改后，用我们1中准备好的强名称文件重新对所有修改过的文件签名，之后便可以运行看看成果了：）

## 二、工具集

破解.net程序对工具的依赖明显比win32程序要严重，这里列出一些（后面有用到，汗！）：

1、`reflector.exe`

这个反编译工具是不可不备了，免费+强大，特别是analyzer功能，简直是神了！！！

2、`research.net`

查看.net元数据信息的工具，许多时候得用它辅助对应反混淆后与反混淆前的程序。

3、`dis#`

反编译并且能保存为文件，这一点比较好，破解.net程序差不多总是要从原程序里抽一些代码片断作计算了。

4、`SnippetCompiler`

C#代码片断编译运行工具，从原程序里抽出代码来临时运行一下，总是用vs.net还要建工程就不爽了。

5、`ildasm`

低级但是强大的工具，到修改原程序之前，参照它的结果应该是必须的，至少总是要看一下IL字节码吧。

6、`ultraedit`

文本编辑器，在一般文本与十六进制间来回倒腾，或者是要搜索文本内容时，这个还是比较方便一些的。

7、`hexworkshop`

别的不说了，就感觉最爽的功能是对原文件进行二进制替换时，Select Block=>Paste Special，就这两步，恰到好处。

（`winhex`也有类似功能，不过感觉还是`hexworkshop`更顺手一些，呵呵）

## 三、破解过程

1、去强名，这个直接用原来5019的keygen的方法，包括它生成的强名文件，呵呵。

```
/SC 0024000004800000940000000602000000240000525341310004000001000100f9525ef4fb72db4053790103572ec74a576bfe4aa024ed19b78356c93193b1e0dbbe9cb01e1f4fa5c5b181a83e83e25b8261717bf4dbcecc3ad453a4a71ac72f3e102403ed2d2ed01fcc71802f2ee4527825aedc31eb91c0abe100d5eacac664486d4f422f5e2d73fb30ed326c49ccb54dc2568900f1c32b27739822876658b8 002400000480000094000000060200000024000052534131000400000100010067790bbf0dee12b24b2bc6feb5e9e5dfbfaeea35efa57bb4f3b6c017329dba78a14e7d03b95a8b790f190f258f2423b577bb9a1d053529e7c93b252d75cb3f9be7710f44c6c48f441aa7348df218fb3390f55e5d50eb7a7cab35a2c9ad5bf8694d887f560e3289c560f3b290c239046332a3178b7886a19b117814cc24bbd6b9

/UC 0024000004800000940000000602000000240000525341310004000001000100f9525ef4fb72db4053790103572ec74a576bfe4aa024ed19b78356c93193b1e0dbbe9cb01e1f4fa5c5b181a83e83e25b8261717bf4dbcecc3ad453a4a71ac72f3e102403ed2d2ed01fcc71802f2ee4527825aedc31eb91c0abe100d5eacac664486d4f422f5e2d73fb30ed326c49ccb54dc2568900f1c32b27739822876658b8 002400000480000094000000060200000024000052534131000400000100010067790bbf0dee12b24b2bc6feb5e9e5dfbfaeea35efa57bb4f3b6c017329dba78a14e7d03b95a8b790f190f258f2423b577bb9a1d053529e7c93b252d75cb3f9be7710f44c6c48f441aa7348df218fb3390f55e5d50eb7a7cab35a2c9ad5bf8694d887f560e3289c560f3b290c239046332a3178b7886a19b117814cc24bbd6b9

/B 0024000004800000940000000602000000240000525341310004000001000100f9525ef4fb72db4053790103572ec74a576bfe4aa024ed19b78356c93193b1e0dbbe9cb01e1f4fa5c5b181a83e83e25b8261717bf4dbcecc3ad453a4a71ac72f3e102403ed2d2ed01fcc71802f2ee4527825aedc31eb91c0abe100d5eacac664486d4f422f5e2d73fb30ed326c49ccb54dc2568900f1c32b27739822876658b8 002400000480000094000000060200000024000052534131000400000100010067790bbf0dee12b24b2bc6feb5e9e5dfbfaeea35efa57bb4f3b6c017329dba78a14e7d03b95a8b790f190f258f2423b577bb9a1d053529e7c93b252d75cb3f9be7710f44c6c48f441aa7348df218fb3390f55e5d50eb7a7cab35a2c9ad5bf8694d887f560e3289c560f3b290c239046332a3178b7886a19b117814cc24bbd6b9

/SC 59d4bed864488801 19d207f78a3e6d0f

/UC 59d4bed864488801 19d207f78a3e6d0f

/B 59d4bed864488801 19d207f78a3e6d0f
```

这是原keygen里的一段脚本，用一个`snr.exe`的小程序执行，其实就是二进制替换，前三个是替换强名称公钥，后三个是替换强名称公钥的token，三个参数看起来像是字符串的三种格式ASCII、UTF8、UNICODE（不确定）。

2、反混淆

用我们的`DeObfuscator.exe`，运行后添加进这几个assembly:

```
NineRays.AsmBrowser.DLL
NineRays.Build.Tasks.dll
NineRays.Controls.DLL
NineRays.Decompiler.dll
NineRays.Documenter.dll
NineRays.ILOMD.DLL
NineRays.ILOMD.Options.DLL
NineRays.Informer.dll
NineRays.Investigator.dll
NineRays.Localizer.dll
NineRays.Modeler.dll
NineRays.Obfuscator.DLL
NineRays.Services.DLL
NineRays.UML.DLL
NineRays.Utils.DLL
NRObfuscator.exe
Spices.exe
```

添加完后，选择一下输出目录，然后点“反混淆”，这样就会在输出目录下生成反混淆后的文件了。

![](http://bbs.pediy.com/upload/2006/4/image/sp1.jpg)

3、破解与强名称相关的字符串加密

用`reflector.exe`装入我们前面反混淆过的文件，然后在`mscorlib`（如果没有就装入一下`mscorlib.dll`）下面找到`System.Reflection.AssemblyName.GetPublicKeyToken`，右键Analyzer，此时会看到右边列出所有引用这个方法的地方，依次查看与我们目标程序相关的那些地方，可以发现有三处是字符串加密的。

```
System.Reflection.AssemblyName.GetPublicKeyToken() : Byte[]

        Depends On

        Used By

                System.AppDomain.ApplyPolicy(String) : String

                System.Reflection.Assembly.IsSimplyNamed(AssemblyName) : Boolean

                System.Resources.ResourceManager.CompareNames(String, String, AssemblyName) : Boolean

                System.Resources.ResourceManager.InternalGetResourceSet(CultureInfo, Boolean, Boolean) : ResourceSet

                System.Xml.Serialization.TempAssembly.LoadGeneratedAssembly(Type, String, XmlSerializerImplementation&) : Assembly

                x15.x4.x33ED8() : Object

                x2.x4.x332B4() : Object

                x2.x4.x3381() : Object

                x9.x26.x9(AssemblyName, AssemblyName) : Boolean

                x9.x73.x9(Assembly) : x921

                x9.x8.x35(Object) : Void
```

现在用`dis#.exe`装入刚刚有字符串加密的那个assembly，找到字符串加密的类，将它的代码保存成c#文件，启动`SnippetCompiler`，打开刚刚保存的C#文件，加上一些读取字符串并写文件的代码，运行一下，就得到了这个类里所有加密的字符串，注意那些包含evaluation的字符串对应的解密的方法，稍后会用到。

我们来看一下典型的几个方法

```
internal sealed class x4
{
  internal static byte[] x33;
  internal static Encoding x33AB = Encoding.Unicode;

        static x4()
        {
               x4.x33 = x4.x33(new byte[] { ... });
        }

        internal static byte[] x33(byte[] obj1)
        {
      return ((TripleDESCryptoServiceProvider) x4.x332B4()).CreateDecryptor().TransformFinalBlock((byte[]) obj1, 0, obj1.Length);
        }

        internal static object x332B4()
        {
              TripleDESCryptoServiceProvider provider1 = new TripleDESCryptoServiceProvider();
              provider1.Key = new byte[] {
                    0x18, 0xac, 3, 0x7c, 0x7d, 0xb8, 0xaf, 13, 0xcb, 0x8d, 0xb2, 200, 0xc2, 0xe1, 0x5e, 170,
                    40, 0xa4, 0xea, 0x58, 0x98, 0x76, 0x3a, 0x77
               };
              provider1.IV = Assembly.GetExecutingAssembly().GetName().GetPublicKeyToken();
              return provider1;
        }

        internal static object x33()
        {
              return x4.x33AB.GetString((byte[]) x4.x33, 0x1e1c, 0x48);
        }
}
```
