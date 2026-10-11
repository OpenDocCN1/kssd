
# 使用 C# 或者 VB6 开发 IDA 插件

> 原文：[www.pediy.com/kssd/pediy08/pediy8-787.htm](https://www.pediy.com/kssd/pediy08/pediy8-787.htm)

这篇文章提供了如何使用高级语言如 C# 或者 VB6 为 IDA 插件创建图形用户界面的技术。

这两种语言实现复杂用户界面非常简单，也容易调试，更为强大的是快速原型化开发。

我将要完成的是使用 VC 创建一个传统的 IDA 桩插件，这个插件可以向 IDA 添加菜单项，并且导出可以供高级语言插件利用的一系列预先包装的 IDA API 函数。我们将要完成的程序的基本图表示意如下：

![](img/705eaf20f5d0ed2f1305976758082a3f.jpg)

**概念图**

为了使 VB6，.NET 和 VC 能够一起使用，我们已经建立了自己的插件以便它们将 DLL 作为 COM 类进行存取。为了作为 COM 类来访问 .NET 的 dll，你将不得不使用 `regasm` 效用来建立界面的适当的注册入口，同样需要生成供在我们的 VC 桩模块早期绑定使用的类型库文件（`.tlb`）。VB6 只能产生 ActiveX DLL，并且在编译的时候已经建立所有注册入口和在 DLL 中增加了类型库信息。如果你仅仅想看看演示，你必须使用 `regsvr32` 注册 VB6 示例 dll。

因此，为了能快速进入代码，在我们的两个例子插件中，我们都早期绑定了我们的 DLLs。这也就意味着你必须先编译 VB6 和 .NET 的 DLLs，并且你要确认所设置的二进制的兼容性，以便下次你编译后不会改变用户界面（或者你重新编译了你的桩模块）。这不是实际的问题，因为启动插件的界面非常简单，也不需要改变。仅仅用于初始化库文件和运行。在我们的 VC 桩模块中你将看到下面的声明：

```cpp
#import "./bin_4.7/CSharpIDASample.tlb" raw_interfaces_only

using namespace CSharpIDASample;

ICpluginPtr pIPlug(__uuidof(Cplugin));
```

这里使用了一些 VC 声明来加载类型库界面，创建我们主类的一个已经初始化的实例指针。命名空间、类、和界面指针的名字都已经在类型库中设定，可以使用类型库查看器（来自 Matt CurlandsPower VB 站点的免费演示版程序或者其他的工具象 Microsoft Ole View utility）查看。

![](http://powervb.com)

下面来显示我们的界面，我们要做的就是调用我们的类方法启动它。在下面的例子代码中，我们传递了 IDA 主窗口的句柄，以便使我们的窗口能够以模态形式显示。（在这一点上我没有试着将主窗体设为 MDI 子窗体，使用 `SetParent` 也许能做）。

```cpp
int h = (int)callui(ui_get_hwnd).vptr; //ida main window hwnd
pIPlug->DopluginAction(h, &lRet);
```

这里 `lRet` 变量将接收 `HRESULT` COM 的返回值，这我们不用担心。一旦我们的插件窗口显示，IDA API 功能将经过在我们窗口中声明的函数执行。这些声明提供给 VB 开发着执行标准 Win32 API 的方法，同样也在 .NET 提供了，以便你能访问定制的 C dlls。下面是 C 原型声明和相应的 C#和 VB6 声明：

```cpp
int __stdcall NumFuncs(void)
int __stdcall FunctionStart(int n)
int __stdcall FunctionEnd(int n)
```

**VB6:**

```vb
Private Declare Function NumFuncs Lib "vcSample.plw" () As Long
Private Declare Function FunctionStart Lib "vcSample.plw" (ByVal functionIndex As Long) As Long
Private Declare Function FunctionEnd Lib "vcSample.plw" (ByVal functionIndex As Long) As Long
```

**C#:**

```csharp
[DllImport("c_stub.plw")]
public static extern int NumFuncs();

[DllImport("c_stub.plw")]
public static extern int FunctionStart(int index);

[DllImport("c_stub.plw")]
public static extern int FunctionEnd(int index);
```

使用这种机制来访问从我们 CPP 桩模块中导出的预先封装的 API 带给我们一定的机动性，并且使我们不用担心 COM 的兼容性带来的界面改变问题。

另外提供了基本例子框架，另外还包含了另外的 36 个 IDA API 包，你可以在你的程序中使用。如果你喜欢晚绑定 COM 类，你可以在我们的站点找其他的老例子。我在这里使用早绑定的原因是晚绑定需要花一页半的通信代码。

确信已经阅读了应用程序目录中的 `README` 文件，去学习如何建立工程，IDA4.7 提供了预编译功能。

Anyway, enjoy..
4dd466d0e1e0cc123afbe504da2462ca
