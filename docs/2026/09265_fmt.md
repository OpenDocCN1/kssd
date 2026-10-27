
# [原创]VMP编译的完整笔记

很多年没有来论坛了，最近刚好有点时间，编译了VMP，发现对于初学者还是可能会有一点帮助的，记录下编译过程。

VMP源码泄露了一段时间了，吾爱破解论坛上发表了《VMProtect编译成功过程梳理》文章，过程较为繁琐，我利用最近时间仔细学习了一遍，很快编译成功了，然后自己重新完整尝试没有博文指导下，记录编译过程，发现原来文章其实有些步骤是多余的。

## 1、查看工程文件

利用记事本打开`vmprotect.sln`可以看到：

```text
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio 15
VisualStudioVersion = 15.0.28307.2092
MinimumVisualStudioVersion = 10.0.40219.1
```

原来的工程是VS2015编译的。

利用记事本打开`VMProtectSDK.vcxproj`，可以看到以下信息：

```xml
  <PropertyGroup Label="Globals">
<ProjectGuid>{F3B5035B-34E5-4A8A-9B2B-485C3C79B67A}</ProjectGuid>
    <RootNamespace>VMProtectSDK</RootNamespace>
    <Keyword>Win32Proj</Keyword>
    <ProjectName>VMProtectSDK</ProjectName>
<WindowsTargetPlatformVersion>7.0</WindowsTargetPlatformVersion>
  </PropertyGroup>
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'" Label="Configuration">
    <ConfigurationType>DynamicLibrary</ConfigurationType>
    <PlatformToolset>v90</PlatformToolset>
  </PropertyGroup>
```

`7.0`指用于构建项目的 Windows SDK 的版本号，实际上是支持winxp的sdk，`V90`对应着VS2008的编译器和相关C++库。

## 2、开发环境

根据上面，我电脑配置了开发环境为：VS2022 +QT5.7+Windows SDK10.0+.net4.8

## 3、编译过程

利用VS2022打开项目文件，自动检查解决方案不匹配的地方，显示如下对话框，允许项目升级（不考虑支持XP系统，之前在安装VS2022是选择安装vs2017 xp支持组件`v141_XP`），点击确定。

![](https://bbs.kanxue.com/upload/attach/202312/34757_QAATSE7RDVTCAHE.webp)

升级完成后，设置工程为X64，直接在`VMProtectSDK`工程上右键“生成”，很快显示成功。

```text
1>  正在创建库 E:\vmp3\vmp\bin\64\Debug\VMProtectSDK64.lib 和对象 E:\vmp3\vmp\bin\64\Debug\VMProtectSDK64.exp
1>VMProtectSDK.vcxproj -> E:\vmp3\vmp\bin\64\Debug\VMProtectSDK64.dll
========== 生成: 1 成功，0 失败，0 最新，0 已跳过 ==========
========== 生成 于 下午 3:03 完成，耗时 04.115 秒 ==========
```

在`VMProtect`工程上右键“生成”，报错106个，找到第一错误：

| 严重性 | 代码 | 说明 | 项目 | 文件 | 行 | 禁止显示状态 | 详细信息 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 错误 | C2511 | `void IntelFunction::Mutate(const CompileContext &,bool)`：`IntelFunction`中没有找到重载的成员函数 | `core` | `e:\vmp3\vmp\core\intel.cc` | 16040 |  |  |

找到该函数声明：

```cpp
void Mutate(const CompileContext &ctx, bool for_virtualization, int index = 0);
```

原因是声明中使用默认构造参数，定义中直接缺少了构造参数。

`intel.cc`将其修改为：

```cpp
void IntelFunction::Mutate(const CompileContext& ctx, bool for_virtualization, int index)
```

重新编译，报错38个，找到第一个错误：

| 严重性 | 代码 | 说明 | 项目 | 文件 | 行 | 禁止显示状态 | 详细信息 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 错误 | C2511 | `void IntelObfuscation::Compile(IntelFunction *,size_t)`：`IntelObfuscation`中没有找到重载的成员函数 | `core` | `e:\vmp3\vmp\core\intel.cc` | 17335 |  |  |

同样的原因，`intel.cc`将其修改为：

```cpp
void IntelObfuscation::Compile(IntelFunction* func, size_t index, size_t end_index, bool for_virtualization)
```

重新编译，报错1个：

```text
res.bat: QT resources 'E:\vmp3\vmp\VMProtect\resources.cc' are up to date
```

直接在命令行中运行`res.bat`发现：

```text
E:\vmp3\vmp\VMProtect>D:\DATA\Qt1\5.7\msvc2015_64/msvc2015_64/bin/rcc.exe E:\vmp3\vmp\VMProtect\application.qrc -o E:\vmp3\vmp\VMProtect\resources.cc.tmp
系统找不到指定的路径。
```

检查系统变量：

![](https://bbs.kanxue.com/upload/attach/202312/34757_MW8FHQV2GQ7XUR6.webp)

实际上是设置的与作者系统变量不一致导致的，修改`res.bat`中的语句如下：

```bat
%QTDIR%/bin/rcc.exe %RC_DIR%application.qrc -o %rc_out%
```

重新编译，报错53个，找到第一个错误：

| 严重性 | 代码 | 说明 | 项目 | 文件 | 行 | 禁止显示状态 | 详细信息 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 错误(活动) | E1696 | 无法打开源文件 `QtCore/QMap` | `VMProtect` | `E:\vmp3\vmp\VMProtect\precompiled.h` | 5 |  |  |

说明程序编译QT时没有找到QT的正确路径，右键`VMProtect`属性，修改VC++目录的包含目录和库目录如下：

![](https://bbs.kanxue.com/upload/attach/202312/34757_4V9T3HEW87U99X7.webp)

编译成功。

```text
1>VMProtect.vcxproj -> E:\vmp3\vmp\bin\64\Debug\VMProtect.exe
```

然后`windeployqt VMProtect.exe`完成qt程序的打包，即可以运行程序。

![](https://bbs.kanxue.com/upload/attach/202312/34757_HCP32REQZTM46SB.webp)

---

## 补充 Ultimate版本的编译过程

### 5.1 QT静态版本的编译

**准备工作：**

1. 在Qt官网下载Qt库的源码。

`qt-opensource-windows-x86-msvc2015_64-5.6.0.exe`

安装过程中选择需要源代码。

2. 下载并安装Python。

3. 下载并安装Perl。

4. 下载并安装Ruby。版本对应如下：

![](https://bbs.kanxue.com/upload/attach/202312/34757_X4RPHC6TJ33NMQ4.webp)

VS2015编译按照网上方法没有任何障碍：

```bat
configure -prefix "D:\Qt5.6.0\5.6\msvc2015_64_static" -shared -release -nomake examples -nomake tests -skip qtwebengine -opensource -confirm-license
nmake
nmake install
```

但是这样实际上会有问题，没有生成`qwindows.lib`，但是会生成`libEGL.lib`和`libGLESv2.lib`后面有用。

正确的必须编译QT时支持OPENGL，巨大的坑，基本上每次必须都删除`SRC`文件夹重新`configure`、`nmake`否则会出现各种错误。

```bat
configure -prefix "D:\Qt5.6.0\5.6\msvc2015_64_static" -static -static-runtime -release -nomake examples -nomake tests -skip qtwebengine -opensource -confirm-license -opengl desktop
nmake
nmake install
```

将两次结果合并在一起，就可以使用了。

得到Qt静态库之后，剩下的就很简单了，在VS2015下编译源程序，步骤与前面基本一样，个别地方需要修改代码，

`VMProtect.Runtime`工程中，类似下面代码会报错`_`变量无法识别：

```csharp
if (NtQueryInformationProcess(CurrentProcess, PROCESSINFOCLASS.ProcessBasicInformation, out obj, Marshal.SizeOf(typeof(PROCESS_BASIC_INFORMATION)), out _) == 0)
{
    if (Marshal.ReadByte(new IntPtr(((PROCESS_BASIC_INFORMATION)obj).PebBaseAddress.ToInt64() + 2)) != 0) // PEB.BeingDebugged
        return true;
}
```

修改代码为：

```csharp
uint p;
if (NtQueryInformationProcess(CurrentProcess, PROCESSINFOCLASS.ProcessBasicInformation, out obj, Marshal.SizeOf(typeof(PROCESS_BASIC_INFORMATION)), out p) == 0)
{
    if (Marshal.ReadByte(new IntPtr(((PROCESS_BASIC_INFORMATION)obj).PebBaseAddress.ToInt64() + 2)) != 0) // PEB.BeingDebugged
        return true;
}
```

然后编译`core`、`misc`、`VMProtectSDK`、`VMProtect`即可，最后完成VMP授权。

![](https://bbs.kanxue.com/upload/attach/202312/34757_MQRNFH9AS5YETDR.webp)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2023-12-20 15:14 被laowanghai编辑 ，原因： 补充VMP Ultimate版本的编译过程

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)
