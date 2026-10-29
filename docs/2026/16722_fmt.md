
# [原创] Textobot-VSCode 环境部署并运行第一个 TB 插件

## 导读

- `0x00` 交个朋友
- `0x01` 安装 `textobot-editor.vsix`
- `0x02` 配置详解
- `0x03` 命令详解
- `0x04` 模板项目格式
- `0x05` 运行第一个 TB 插件
- `0x06` 下篇再见

## `0x00` 交个朋友

手游从业者模拟器玩家请关注云游模拟器 PantaWin。

Android 开发者请关注云游模拟器 PantaWin/PantaMac/PantaLinux。

iOS 越狱开发者请关注晓文框架 Textobot。

Android 插件开发者请关注飞度框架 Fridobot。

移动调试器重度用户请关注利达调试器 LidaDbg。

我们的产品推荐使用 `git` 下载和更新，[在码云搜索 geekneo 即可](https://bbs.kanxue.com/elink@f50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3N6W2k6h3E0F1k6h3!0Q4x3V1j5`.)。

## `0x01` 安装 `textobot-editor.vsix`

前置声明，下文 `TBROOT` 是指本产品的根目录，VSCode 是指微软旗下的跨平台代码编辑器 Visual Studio Code。Textobot 脚本插件的主力开发语言是 Python 和 JavaScript，辅助开发语言是 C++/Objective-C。在众多的代码编辑器中，我们最终选择了 VSCode，因为它开源、易用、强大、可扩展。针对 Textobot 插件开发、部署等使用场景，我们定制了一个 VSCode 插件 `textobot-editor.vsix` 扩展。目前测试过兼容性的平台包括 macOS、Windows，安装方式是使用 VSCode 的 Install From VSIX 功能，如下图：

![](https://bbs.kanxue.com/upload/attach/202002/461021_M32T3RVERPBZT58.jpg)

安装完成之后，打开 OUTPUT 输出窗口，右上角切换至 Textobot 输出模式就可以看到 `textobot-editor` 的输出日志了，如下图：

![](https://bbs.kanxue.com/upload/attach/202002/461021_B4HY933B4H9N22N.jpg)

## `0x02` 配置详解

完成 TB 编辑器的安装之后，为了和运行在 iPhone 上的 Textobot 后台服务通信，我们需要配置一些参数。当前一共有四个配置参数，如下图：

![](https://bbs.kanxue.com/upload/attach/202002/461021_WRGPBBQR89BS9MZ.jpg)

- `Ip`：这是必须设置的参数，填入目标 iPhone 与本机所在的局域网 IPv4 地址，填好这个参数 TB 编辑器才可以与 iPhone 上的 Textobot 后台服务通过 HTTP 协议通信。
- `Iplist`：这是可选参数，填入一个 IPv4 地址列表文本文件全路径，格式为一行一个 IPv4 地址，这个在需要同时控制多台 iPhone 时使用。
- `Log`：这是可选参数，在想查看通过 JavaScript 调用 Textobot 导出的日志函数输出内容时使用，这个路径必须是 `TB_logpath` 函数返回的值，后续在开发 TB 插件的文章中我们再详细介绍。
- `Target`：这是可选参数，填入 iPhone 目标进程的名字或者 BundleID，在开发 JavaScript 代码时使用，这个参数是提供给 Frida 注入 JavaScript 代码时使用的。

## `0x03` 命令详解

当完成目标 iPhone 的 IPv4 地址设置之后，我们就可以通过 Textobot Editor 提供的命令与 Textobot 手机服务通信了。通过 Command Palette 打开插件命令搜索界面，然后输入 Textobot：

![](https://bbs.kanxue.com/upload/attach/202002/461021_ZGHUP6KTVZTTYEX.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/461021_49AF95M8V6J9GJS.jpg)

这里就把所有 TB 编辑器导出的命令罗列出来了，我们逐一解释它的功能：

- `Install`：选择 `.tb` 文件安装至手机，`.tb` 就是 TB 插件，由 `Pack` 命令打包而来，本质上就是 `zip` 文件改了后缀名。
- `Pack`：将当前 TB 项目打包成 `.tb` 插件，执行该命令时当前打开的文档必须是 TB 项目根目录的文档，否则打包出来的 `.tb` 文件格式无效，无法安装。
- `List`：列出当前安装在手机的 TB 插件，输出的内容是格式化后的 JSON，对应的 key 就是每一个插件的 package 名字，有些命令比如 `Start` 就需要输入插件包名。
- `Log`：获取当前 Python `print`/JavaScript `console.log` 输出的日志内容。
- `LogReset`：清空所有日志，一般在开发插件时使用。
- `Start`：启动 TB 插件，需要填入插件的 package 名字，由 `List` 命令返回。
- `Stop`：停止 TB 插件，需要填入插件的 package 名字，由 `List` 命令返回。
- `Uninstall`：卸载 TB 插件，需要填入插件的 package 名字，由 `List` 命令返回，卸载之前必须先执行 `Stop` 命令停止，否则将会卸载失败。
- `AppLog`：获取当前配置项 `Log` 指向的文件内容。
- `TextFile`：获取 iPhone 手机上的文本文件内容。
- `Enable`：启用 TB 插件，需要填入插件的 package 名字，由 `List` 命令返回。
- `Disable`：禁用 TB 插件，需要填入插件的 package 名字，由 `List` 命令返回，禁用之后将无法 `Start`。
- `Run`：将 VSCode 当前打开的 `.py`/`.js` 文件代码发送至手机端执行，一般是在开发时使用，下一篇文章详细解释。

## `0x04` 模板项目格式

![](https://bbs.kanxue.com/upload/attach/202002/461021_MK59WCW33NHSCZG.jpg)

`TBROOT/Template` 是一个完整的模版项目，一般情况下新的插件只需要拷贝它，修改配置和脚本代码就可以了。一个有效的 TB 插件项目必须包含两个文件：`__init__.py` 以及 `textobot.json`，其他文件比如 `dylib`、`js` 等酌情处理。

`__init__.py` 就是标准的 Python 模块文件。当安装至手机端，Textobot 后台服务就是按照 Python 模块的方式引入插件代码的。如果这个文件不存在，也就不符合 Python 模块的定义，所以也无法加载插件。

`textobot.json` 是本插件的描述信息，用于指明要收到哪些进程的事件、插件包名、描述、作者等元数据，后续文章我们再详细解释它的字段。

`textobot.js` 是可选的，文件名字也是可以任意指定的，它是在需要注入特定进程执行 JavaScript 代码时才需要。了解 Frida 的朋友应该比较清楚，它就是给 Frida 的 `script` 对象使用的。后续文章我们再详细说明如何使用。

## `0x05` 运行第一个 TB 插件

了解完模板项目格式，我们演示一个 `TBROOT/Sample` 目录下面的例子插件。为了有直观的可视化效果，我们选择了 AppleIDLogin 这个插件。这个插件大致意思就是获取 Preferences 文本事件、解析文本事件、点击设定的文本、登录硬编码的无效 AppleID 账号，然后一直重复下去。通过 `Install` 命令选择 `TBROOT/Sample/AppleIDLogin.tb` 插件，安装好之后，在手机端用管理器启动它。（看雪不支持视频，观看录制的插件视频请关注我的公众号或[这里](https://bbs.kanxue.com/elink@3d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3i4K6u0W2P5h3!0#2K9%4g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3i4K6g2X3M7$3S2G2N6#2)9J5c8X3W2V1i4K6g2X3h3p5&6p5f1e0g2y4g2p5p5J5e0h3A6m8N6@1&6m8i4K6y4p5i4K6y4p5i4K6u0W2K9s2c8E0L8q4)9K6c8Y4y4H3L8g2)9K6c8r3p5J5K9o6y4B7i4K6u0W2z5o6b7J5z5o6M7%4x3q4)9J5k6e0x3@1x3e0j5H3y4e0W2Q4x3X3f1I4)）

![](https://bbs.kanxue.com/upload/attach/202002/461021_BNDJVQKC9Y92YZJ.png)

## `0x06` 下篇再见

至此，我们就完成了 Textobot 插件体系手机端、桌面端的安装部署，也演示了一个可视化的 AppleID 账号登录样例插件。下一篇文章开始，我们进入 TB 插件开发模式，从最简单的 Hello World 代码开始，编写我们的 TB 插件。

[[招生] 科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)
