
# [原创][分享] VMP源码编译之Debug_x64模式(新)

> 最终依赖环境：Windows 7 x64 Ultimate、Visual Studio 2019、Windows SDK 8.1、`qt-opensource-windows-x86-msvc2015_64-5.6.0`、.NET Framework 4.8、QT Visual Studio Tools（v3.0.2 最新版）

## 0x00 源码

`https://bbs.kanxue.com/thread-279796.htm`

![](upload/attach/202312/780980_H9PSSAHRQ3ANCD3.webp)

链接：`179K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4c8%4l9@1y4i4u0q4P5h3#2b7N6U0V1@1c8W2W2H3N6h3q4%4K9$3#2u0k6H3`. 提取码：947c

## 0x01 操作系统

> windows 7 x64 sp1 环境搭建（虚拟机）

![](upload/attach/202312/780980_AK2NBEW4ERAESHW.webp)

## 0x02 软件安装

### 2.1 Visual Studio 2019

> `7c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8r3W2H3j5h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6K6i4K6u0r3k6g2k6t1P5r3D9&6M7q4k6h3e0q4p5`
>
> 提取码：tt80
>
> 双击 vs2019 安装包后，提示如下错误

![](upload/attach/202312/780980_87ZGTX995E8TBYE.webp)

### 2.2 .NET Framework 4.6.2（这个可无需安装，直接安装 .NET Framework 4.8）

> 解决：按照上面操作的提示先安装一个 .NET Framework 4.6.2

`88cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3X3c8X3M7X3q4E0k6i4N6G2M7X3E0Q4x3V1k6F1k6i4b7@1y4W2)9$3x3l9`

> 下载离线版
>
> 当运行安装时，出现如下错误：

![](upload/attach/202312/780980_MTBTUNSSR3NBJTE.webp)

> 修复：下载适应操作系统补丁

`1bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0j5i4c8S2L8r3!0Y4i4K6u0W2N6i4m8V1j5i4c8W2i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0W2j5i4y4H3P5q4)9K6c8Y4q4Q4x3@1c8C8j5U0b7@1y4K6b7@1x3e0W2Q4y4U0l9`

![](upload/attach/202312/780980_TSNCF2UEX3SJSPX.webp)

> 进行安装，然后根据提示重启计算机

![](upload/attach/202312/780980_R9Q3DT9ZTZV48VR.webp)

> 接着安装 .NET Framework 4.6.2

![](upload/attach/202312/780980_86TMMWHVV3SGZMY.webp)

### 2.3 .NET Framework 4.8

`e8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3X3c8X3M7X3q4E0k6i4N6G2M7X3E0Q4x3V1k6F1k6i4b7@1z5q4)9$3x3l9`

![](upload/attach/202312/780980_JKUK8YZQESK36M5.webp)

> 下载后直接安装即可

![](upload/attach/202312/780980_Y5U8B7AJJ4U2F5U.webp)

> 安装成功

![](upload/attach/202312/780980_U63M94VC7639TZ4.webp)

> 重新再安装 VS2019，选项勾选如下

![](upload/attach/202312/780980_MWFE3S4DJQK3NGY.webp)

![](upload/attach/202312/780980_THY39VFZ7K84NC4.webp)

![](upload/attach/202312/780980_XYM5WMBZAKSMWBH.webp)

![](upload/attach/202312/780980_VT2EVA4DFZYE8CP.webp)

![](upload/attach/202312/780980_P8T9KT7YK8ZA2SJ.webp)

> 勾选以上，选择安装位置后，点击安装即可

![](upload/attach/202312/780980_KE7TECFQTRQDP23.webp)

> VS2019 安装成功

![](upload/attach/202312/780980_AA98FG47DMHCFVD.webp)

> VS2019 Pro 激活码：NYWVH-HT4XC-R2WYW-9Y3CM-X4V3Y

### 2.4 QT 5.6.0

> 安装 QT，下载地址：`791K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4q4@1i4K6u0W2K9h3!0Q4x3V1k6F1k6i4N6Q4y4h3k6S2M7X3y4Z5K9i4k6W2i4K6u0r3M7i4c8Q4x3V1j5#2i4K6u0W2y4W2)9J5c8U0g2Q4x3X3f1$3i4K6u0W2x3q4)9J5c8W2)9$3x3l9`

![](upload/attach/202312/780980_R686W65BKEZF8VK.webp)

> 安装时，这里直接默认，没做任何修改，直接下一步安装即可

![](upload/attach/202312/780980_F7GVN3FQB69BKZD.webp)

### 2.5 SDK 8.1

> 下载地址：`d67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9q4)9J5k6r3y4F1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8Y4y4V1K9#2)9J5k6r3q4J5j5$3S2A6N6X3g2Q4x3V1k6Q4y4U0l9`

![](upload/attach/202312/780980_ADKRT33WEX99ETS.webp)

> 下载完成后，无需做任何修改，直接安装即可

![](upload/attach/202312/780980_CPJHF6MTATUQS6C.webp)

### 2.6 Notepad++

> 下载地址：`ddeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8%4c8W2M7r3q4V1i4K6u0V1M7r3I4#2M7#2)9J5k6s2m8D9N6i4y4Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1k6Q4y4U0l9`
>
> 后续会用作配置文件等修改

![](upload/attach/202312/780980_K46MUXUHHV546ZM.webp)

### 2.7 QT Visual Studio Tools

> 这里直接在 VS 中安装（选择“扩展”>“管理扩展”>“在线”，搜索 Qt Visual Studio 工具）

![](upload/attach/202312/780980_PB2MND3WWRZZBHY.webp)

> 或者直接下载 v3.0.2 版本

`559K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4q4@1i4K6u0W2K9h3!0Q4x3V1k6G2k6X3k6A6j5$3W2S2L8q4)9#2k6Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3N6Y4y4S2k6r3c8A6L8W2)9J5c8W2)9$3x3l9`

![](upload/attach/202312/780980_NU9G6CD6QNYZBM9.webp)

> 安装成功后，退出 VS2019，提示如下，点击 Modify 等待完成即可

![](upload/attach/202312/780980_FKNMMMRMFN6AGQR.webp)

## 0x03 环境变量配置

![](upload/attach/202312/780980_EMUFK6Z6FH89U4U.webp)

## 0x04 项目属性配置

> 解压源码

![](upload/attach/202312/780980_MPDDSNTEJYDG7EZ.webp)

> 找到 `vmprotect.sln`，用 VS2019 加载后弹出如下界面，全部选择【无升级】后点击确定即可

![](upload/attach/202312/780980_G5UWT7TYCQWT2JK.webp)

> 设置 VS 属性管理器，方便设置项目属性

![](upload/attach/202312/780980_SQ8SRFNVTT3RYE9.webp)

> 切换到属性管理器，将每个项目属性设置为：

![](upload/attach/202312/780980_66BCHVPC6SPRN3T.webp)

1. Windows SDK 版本：8.1
2. 平台工具集：Visual Studio 2015 – Windows XP（`v140_xp`）

![](upload/attach/202312/780980_4KY6FRKK85H43T9.webp)

![](upload/attach/202312/780980_UWNSHS3BUP4YN3F.webp)

> 修改编译模式：`Debug_x64`

![](upload/attach/202312/780980_XZB6UBJZD8J8JPX.webp)

> 确定 QT 插件是否识别

![](upload/attach/202312/780980_78YQ7F3S86HJGQT.webp)

> 如果没有识别到，手动指定即可

![](upload/attach/202312/780980_3EANJCY8CN2TD6P.webp)

![](upload/attach/202312/780980_YPKWRPK4M494U9T.webp)

> 转化 `VMProtect` 项目

![](upload/attach/202312/780980_BGCPUVSZZD2F3ND.webp)

> 转化完成后，继续右键该项目，选择属性设置

![](upload/attach/202312/780980_T2YTQZMP435DC5G.webp)

![](upload/attach/202312/780980_F52SU9G2G4GZ7ZY.webp)

![](upload/attach/202312/780980_M68CXKAJU2D48PD.webp)

## 0x05 源码bug修复

> 遵循下面顺序编译原则：每个模块单独编译，修复所有报错

### 5.1 `Common`模块

> `Intel.cc`：修复 `Mutate` 函数

```c++
void IntelFunction::Mutate(const CompileContext &ctx, bool for_virtualization, int index)
```

![](upload/attach/202312/780980_SADU37NU77DDW4X.webp)

> `Intel.cc`：修复 `Compile` 函数

```c++
void IntelObfuscation::Compile(IntelFunction *func, size_t index, size_t end_index, bool for_virtualization)
```

![](upload/attach/202312/780980_JXQ52ASMA7JTTYY.webp)

### 5.2 `Misc`模块

> 通过 Notepad++ 将 `/vmp/unit-tests/core_tests.cc` 编码格式转换为 UTF-8 后保存
>
> 这个模块重新生成后，还遗留 4 个错误，暂时不管，先编译其它模块

![](upload/attach/202312/780980_RXYRMM8B5XWY263.webp)

### 5.3 `MiniDumpFixer`

> 无需修复

### 5.4 `VMProtect.Runtime`

![](upload/attach/202312/780980_NYYYFYAV7R9GZG2.webp)

单击 `VMProtect.Netcore` 模块进入 `csproj` 文件，修改如下内容：

```xml
<TargetFrameworks>netstandard2.1;netcoreapp3.0;net20;net40;net48</TargetFrameworks>
<RuntimeIdentifier>win-x64</RuntimeIdentifier>
```

![](upload/attach/202312/780980_9M4K9XVGRTEQNVN.webp)

### 5.5 `VMProtect.SDK`

> 无需修复

### 5.6 `VMProtectSDK`

> 无需修复

### 5.7 `VMProtectCon`

> 无需修复

### 5.8 `Win_runtime`

> 无需修复

### 5.9 `VMProtect`

> 查看报错信息，显示 `res.bat` 存在问题

![](upload/attach/202312/780980_E3ZEE2SWBAKB2XD.webp)

![](upload/attach/202312/780980_G4HF43DHZWMNCFK.webp)

```text
修复：
在源码 /vmp/VMProtect/res.bat 文件中
将相对路径：%QTDIR%/msvc2015_64/bin/rcc.exe %RC_DIR%application.qrc -o %rc_out%
修改为绝对路径：
E:/Qt/Qt5.6.0/5.6/msvc2015_64/bin/rcc.exe %RC_DIR%application.qrc -o %rc_out%
```

## 0x06 初运行

> 当点击 `本地Windows调试器` 执行时，弹出如下错误

![](upload/attach/202312/780980_GQC67D8ZY4RHKF8.webp)

> 将 `VMProtect` 模块右键设置为启动项目，重新执行则正常运行

![](upload/attach/202312/780980_ADGZBHNNGTG5KKG.webp)

## 0x07 VMProtect->ScintillaQT

> 经测试，这个 QT 模块中的错误全部来源于没有相应的头文件导致
>
> 相关开源代码可参考 notepad-plus-plus
