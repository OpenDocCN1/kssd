
`objection` 是基于 `Frida` 框架开发的自动化 `Hook` 工具包，该工具支持 `Android` 和 `iOS` 平台。如果你不擅长进行代码开发，但又想使用 `Frida` 进行一些复杂的 `Hook` 操作，那么 `objection` 将是一个非常不错的工具。

`Frida` 依赖 `Python 3.x` 版本，安装 `objection` 可使用以下命令：

```bash
pip3 install objection
```

完成安装后，在终端中输入 `objection` 命令。执行结果显示如下图，说明已经安装成功。正式使用 `objection` 前，需在手机端启动 `frida-server`，并进行端口转发。

![图片描述](upload/attach/202306/452101\_8NZ7SABSQABXJJK.png)

通过应用的包名或者 `bundleID` 附加需要调试的目标应用，进入到 `objection` 提供的交互界面，类似于 `IPython` 的交互环境。

```bash
objection -g [packageName/bundleID] explore
```

`objection` 采用 `attach` 附加模式进行 `Hook`。如果我们需要在应用启动时就进行 `Hook`，可以通过如下方式启动 `objection`：

```bash
objection -g packageName explore --startup-command 'android hooking watch class_method xxxx'
```

查看目标应用程序在设备内的存储路径，直接在交互界面输入 `env` 命令即可，结果如图所示：

![图片描述](upload/attach/202306/452101\_3GW53G367TEFSMB.png)

如目标应用为 `Android` 应用，可通过以下命令查看该应用的组件信息：

```bash
# 查看应用的 activity 组件信息
android hooking list activities
# 查看应用的 service 组件信息
android hooking list services
# 查看应用的 broadcast receiver 组件信息
android hooking list receivers
```

既然能查看目标应用的组件信息，亦可启动目标应用的组件：

```bash
# 启动指定的 activity 组件
android intent launch_activity [class_activity]
# 启动指定的 service 组件
android intent launch_service [class_service]
```

## 目标应用已加载到内存中的类和方法的相关操作

```bash
# 查看内存中所有的类
android/ios hooking list classes
# 查看指定类中所有的方法
android/ios hooking list class_methods <class_name>
# 查找指定特征的类
android/ios hooking search classes <class_name>
# 查找指定特征的方法
android/ios hooking search methods <method_name>
```

## 使用 `objection` 进行 `Hook` 操作

```bash
# hook 指定类下所有的方法
android/ios hooking watch class [class_name]
# hook 指定类中的指定方法
# --dump-args：打印参数
# --dump-backtrace：打印调用栈
# --dump-return：打印返回值
android/ios hooking watch class_method [class_name] --dump-args --dump-backtrace --dump-return
# 设置返回值，目前仅支持 bool 类型
android/ios hooking set return_value [class_name] false
# 生成 Frida 的 hook 代码
android/ios hooking generate simple [class_name]
```

## 调用执行目标应用中的方法

```bash
# 搜索指定类的实例，获取该类的实例 ID
search instances search instances [class_name]
# 通过实例 ID 直接调用该类中的方法
android heap execute [instance_id] [method_name]
```

如果目标应用启用 `SSL` 校验，可通过 `objection` 进行关闭：

```bash
android/ios sslpinning disable
```

## 目标应用内存操作

```bash
# 枚举当前进程模块
memory list modules
# 查看指定模块的导出函数
memory list exports [lib_name]
# 将导出函数的结果保存到指定的文件
memory list exports [lib_name] --json result.json
# 搜索内存
memory search --string --offsets-only
```

## 其它常用命令

```bash
# 尝试对抗 root 检测
android root disable
# 尝试模拟 root 环境
android root simulate
# 截图
android/ios ui screenshot [image.png]
# 对抗越狱检测
ios jailbreak disable
# dump keychain 中的内容
ios keychain dump
# 尝试关闭 iOS 生物特征认证
ios ui biometrics_bypass
# 执行 shell 命令
android shell_exec [command]
```

### 2.1.6 `Hook` 工具 `Tweak`

`Tweak` 是一款依赖 `Cydia Substrate` 框架的越狱插件开发工具，该工具通过创建 `dylib` 动态库注入到宿主进程，完成各种 `Hook` 操作，让开发者在无需要破解 `iOS` 系统的情况下，快速开发出功能强大的 `Tweak` 插件。

进行 `Tweak` 开发前需要安装并配置 `Theos` 环境。`Theos` 提供了一组工具和库，可以帮助开发者快速创建、编译和部署 `Tweak`。具体环境配置命令如下所示：

```bash
# 将 Theos 安装到指定目录中
git clone --recursive https://github.com/theos/theos.git /opt/theos
# 将 Theos 添加到环境变量中
export THEOS=/opt/theos
export PATH="$THEOS/bin:$PATH"
# 安装签名工具，用于对编译完成后的文件签名
brew install ldid
```

`Theos` 环境搭建完成后，就可以开始创建 `Tweak` 项目。在终端中执行 `nic.pl` 命令，根据输出的提示信息选择 `Tweak` 模板进行创建。然后按照提示依次输入工程名称、包名、作者名称等信息即可完成创建，具体命令如下图所示：

![图片描述](upload/attach/202306/452101\_SRFKF95BT738TSC.png)

命令执行成功后，将在当前目录下创建一个名为 `testtweak` 的工程目录。工程目录中默认共生成四个文件，具体文件信息如下图所示：

![图片描述](upload/attach/202306/452101\_T4S7YNBAJBSUMGJ.png)

- `control`：该文件中是当前工程的配置信息介绍，包括工程名称、包名、作者名称、版本号、依赖等。
- `Makefile`：主要作用是自动化构建项目，文件中指定待编译的源代码、依赖信息和编译配置信息等。
- `TestTweak.plist`：此文件用于存储配置信息。配置信息中的 `Bundle ID` 决定了当前项目对哪些应用生效。具体配置信息如下所示：

```text
{ Filter = { Bundles = ( "com.test.demo","com.test.demo1" ); }; }
```

- `Tweak.x`：是 `Theos` 生成的源代码文件，该文件中的代码使用 `Logos` 语法进行开发。具体代码示例如下图所示：

```logos
%hook ClassName
// Hooking a class method
+ (id)sharedInstance {
return %orig;
}
// Hooking an instance method with an argument.
- (void)messageName:(int)argument {
%orig;
}
// Hooking an instance method with no arguments.
- (id)noArguments {
%orig;
}
%end
```

正式进行 `Tweak` 插件开发前，还需要学习一下 `Logos` 语法。`Logos` 是 `Cydia Substrate` 框架所提供的一组用于实现 `Hook` 操作的宏定义，语法简单，便于开发者快速针对目标应用进行 `Hook` 开发。下面将简单介绍一些经常用到的语法，如果想要更加深入地学习 `Logos` 语法，可以查看官方文档。

### `%hook` 和 `%end`

`%hook` 和 `%end` 组合用于 `Hook` 目标类，在代码块中可以直接写需要 `Hook` 的方法，代码示例如下：

```logos
%hook ClassName
// Hooking a class method
+ (id)sharedInstance {
return %orig;
}
%end
```

### `%group` 和 `%end`

`%group` 和 `%end` 组合用于对 `Logos` 代码进行分组管理，每个声明的组都需要使用 `%ctor` 和 `%init` 进行初始化操作，代码示例如下：

```logos
%group testGroup
%hook ClassName
// Hooking a class method
+ (id)sharedInstance {
return %orig;
}
%end
%end
%ctor {
%init(testGroup);
}
```

### `%new`

`%new` 用于在 `Hook` 的目标类中添加新的方法，代码示例如下：

```logos
%hook ClassName
%new
- (void)addNewMethod {
}
%end
```

### `%orig`

`%orig` 用于调用原始方法，可以传入自定义参数和接收原始方法的返回值，示例代码如下：

```logos
%hook ClassName
- (int)add:(int)a to:(int)b {
if (a != 0) {
// Return original result if `a` is not 0
return %orig;
}
// Otherwise, use 1 as `a`
return %orig(1, b);
}
%end
```

### `%log`

`%log` 用于输出目标方法的函数类名、参数等信息，示例代码如下：

```logos
%hook ClassName
- (void)targetMethod:(id)arg1 {
%log
}
%end
```

## 第一个 `Tweak` 插件

现在开始我们的第一个 `Tweak` 插件开发，目标进程以 `SpringBoard` 为例进行讲解。对 `SpringBoard` 进程中的 `[SpringBoard applicationDidFinishLaunching]` 方法进行 `Hook`，添加弹窗代码。`SpringBoard` 启动时弹出窗口，具体代码如下：

```objc
#import <SpringBoard/SpringBoard.h>
%hook SpringBoard
-(void)applicationDidFinishLaunching:(id)application {
%orig;
UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Test"
message:@"Tweak test！！！"
delegate:nil
cancelButtonTitle:@"确定"
otherButtonTitles:nil];
[alert show];
}
%end
```

在编译安装之前，还需要设置目标手机的 `IP`，否则将安装失败。配置完成后，便可在终端中切换到项目所在目录进行编译安装。相关命令如下图所示：

```bash
# 设置 Tweak 插件安装到目标手机的 IP，否则安装将失败
export THEOS_DEVICE_IP=[目标手机IP]
# 编译并安装 Tweak 插件
make do
```

安装成功后，`SpringBoard` 将重新启动并加载 `Tweak` 插件，效果如下图所示。至此你已经初步掌握了 `Tweak` 工具的使用。

![图片描述](upload/attach/202306/452101\_VXMR6XGJ9PA4FKK.png)

### 2.1.7 安全测试工具 `Drozer`

`Drozer` 是由 MWR InfoSecurity 开发的针对 `Android` 应用的安全测试框架。该框架是一款交互式的安全测试工具，支持真实的 `Android` 设备和模拟器，可以通过测试应用与其它应用程序进行交互，开展复杂的测试活动，只需很少时间即可评估与 `Android` 应用安全相关的问题。通过该工具可以帮助安全人员或者开发者快速发现 `Android` 应用中的安全漏洞。

`Drozer` 运行需要依赖 `Python 2.x` 和 `Java` 环境。在正式安装使用前，需要先将其依赖的运行环境配置好。`Drozer` 分为两部分，一部分是客户端，另一部分是手机端代理。手机端代理需要安装在目标设备，用于客户端和目标设备通信时进行数据代理转发。官网下载地址：

<mark class="encrypted">0d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3k6s2u0G2P5X3g2J5i4@1f1K6i4K6R3H3i4K6R3J5</mark>

![图片描述](upload/attach/202306/452101\_Z628945JK4Z7CJX.png)

以 Mac 电脑为例进行安装介绍。在终端中切换到存放 `drozer-2.4.4-py2-none-any.whl` 文件的目录下，然后运行以下命令：

```bash
# Drozer 依赖 Python 2.x，此处的 pip 必须为 Python 2.x 的版本
pip install drozer-2.4.4-py2-none-any.whl --ignore-installed pyOpenSSL
```

安装 `Drozer` 运行时所必需的依赖：

```bash
pip install protobuf==3.17.3 pyOpenSSL Twisted service_identity
# 安装 Mac Command Line Tools
xcode-select --install
```

备注：安装过程中可能会遇到网络原因导致下载文件超时报错的情况，重新执行命令直到安装成功即可。

使用数据线将手机和电脑进行连接，安装 `agent` 到手机并开启服务。电脑的终端中执行端口转发命令和连接手机命令后，如下图所示即可正式开始使用 `Drozer` 工具。

```bash
# 端口转发
adb forward tcp:31415 tcp:31415
# 连接 Drozer
drozer console connect
```

![图片描述](upload/attach/202306/452101\_BJU626HJCFRA3UM.png)

## 查看目标应用的基本信息

通过 `app.package.info` 模块查看目标应用的基本信息：

```text
dz> run app.package.info -a package_name
```

## 查看目标应用的攻击面

查看目标应用的攻击面，主要是指 `Android` 四大组件中 `exported` 属性设置为 `true` 的组件：

```text
dz> run app.package.attacksurface package_name
```
