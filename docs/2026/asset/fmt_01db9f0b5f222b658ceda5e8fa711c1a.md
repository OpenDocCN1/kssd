
# [原创]iOS逆向初探-变种AES加密的分析与还原

[TOC]

## 一. 前言

从事移动安全行业以来，一直在做Android方面的安全及逆向，也曾想过了解下iOS的机制，奈何总是对自己下不了决心，一方面觉得精力有限，Android上好多东西自己也并没有完全熟练掌握。另一方面可能自己太懒，不太想花费太多时间成本，何况如果没有实操，所学的一切很快就会忘记，所以之前也仅仅是在心里埋下了这个种子而已。不过大概几周前，笔者实属有幸，因机缘巧合，向iOS逆向大佬猫大人好好请教了一番，至此也算是入了个iOS的小门。

因此本文也算是一个从Android视角来看待iOS逆向，iOS相关的深入点不会进行阐述。也仅以文本纪念下和大佬们在一起做安全逆向的时光～

## 二. 入门基础

从这里开始，是比较基础的东西，但是由于我也是刚刚学习到，所以就列出来记录一下，有基础可以直接跳过第二节，从第三节/或第四节阅读。

### 2.1 工具

#### 2.1.1 手机选型

在Android上通常用于逆向的手机是pixel/nexus系列，Android系统6~13都可，然后自己刷入`magisk`进行root。而iOS呢肯定是iPhone了，但是如何选系统如何自己越狱呢？因此在了解相关版本后，为了方便，在某宝买了一台二手的iphone8（800左右），ios14左右的系统，商家已经帮你越狱好了(`unc0ver`，14系统上每次手机重启还需点击软件进行越狱，不麻烦)。

当然其他手机也可以，按照大佬的建议，iOS系统最好不要最新，像13,14左右就可以，手机的话像年头久一点的iPhone 6（大概3,4百左右），iphone8/X/SE，也都可以做逆向。切记！买手机时要问有无id锁，是否可以刷机。如果手机来源不正规，自然就会被锁住，也不方便逆向用了。

#### 2.1.2 手机软件

手机选好后，自然就要安装一些相关软件，就像android逆向root之后要按各种插件，比如抓包用`AlwaysTrustUserCerts`信任证书之类的。iOS也是一样，iOS越狱后有个Cydia的商店，里边可以下载安装各种越狱插件，包括自己写的越狱开发插件也会在这里进行管理。

常用的iOS插件：

- Apple File Conduit"2"：用于激活助手类工具对iOS设备所有路径的访问权限。
- AppSync Unified：用来绕过系统对应用的签名验证，可以随意安装和运行脱壳后的`ipa`
- Filza File Manager ：手机上的文件管理器（简称“Filza”），用来浏览手机文件、修改文件的权限等
- SSL Kill Switch 2: ios版的 `justtrustme`
- `OpenSSH`: 链接电脑

当然以上的插件，是我在买好手机后，就已经安装好了，自己并没有额外做什么操作，除了`frida`。

- `frida`：`frida`的安装在`cydia`里默认是最新的，因此可以去`frida`官网的release(<mark class="encrypted">c60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4y4Q4x3U0W2Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0g2Q4b7f1k6Q4b7U0W2Q4c8e0g2Q4b7V1q4Q4z5e0c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5y4Q4z5o6g2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0W2Q4z5o6N6Q4z5p5y4Q4c8e0g2Q4z5e0u0Q4z5p5y4S2L8X3c8J5L8$3W2V1i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6V1^5i4@1q4r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1p5$3i4K6R3I4i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5k6r3g2T1i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6S2o6i4K6R3#2i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4K6R3@1i4@1t1$3i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1&6i4K6R3H3i4K6W2m8i4@1f1^5i4@1u0r3i4K6R3%4i4@1f1#2i4@1p5$3i4K6R3J5K9h3k6#2L8X3u0G2P5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2!0n7y4#2!0m8y4g2!0q4y4g2)9^5y4g2!0n7y4#2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9^5y4q4!0n7y4W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4W2)9^5z5g2)9^5b7W2!0q4y4W2)9&6b7#2!0n7b7g2!0q4y4q4!0n7z5q4)9^5b7g2!0q4z5g2)9^5x3q4)9&6b7g2!0q4z5q4!0n7c8W2)9^5y4@1k6A6L8s2A6S2i4@1f1%4i4K6R3J5i4@1t1&6i4@1f1#2i4K6R3%4i4@1u0n7k6r3g2T1i4@1f1#2i4K6S2o6i4K6R3#2i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1#2i4@1q4q4i4K6R3&6i4@1f1^5i4@1p5K6i4K6R3#2i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1#2i4@1q4q4i4K6R3&6i4@1f1^5i4@1p5K6i4K6R3#2i4@1f1#2i4@1p5#2i4@1u0p5i4@1f1#2i4K6V1H3i4K6S2q4i4@1g2r3i4@1u0o6i4K6S2o6k6Y4u0A6k6r3q4Q4c8e0N6Q4z5e0c8Q4b7U0q4U0P5h3c8A6j5g2!0q4z5q4!0n7c8W2)9&6b7W2!0q4z5q4!0m8x3g2)9^5b7#2!0q4y4#2!0m8c8g2!0m8x3g2!0q4y4#2)9&6x3q4)9^5y4W2!0q4c8W2!0n7b7#2)9^5b7$3k6J5K9h3c8S2i4K6u0V1M7$3g2J5N6X3g2J5i4@1f1&6i4@1u0n7i4K6V1^5i4@1f1^5i4@1q4q4i4@1p5@1i4@1f1#2i4@1u0o6i4K6R3H3i4@1f1#2i4K6V1H3i4@1q4r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4@1t1I4i4@1u0n7i4@1f1@1i4@1u0o6i4@1u0o6b7h3&6V1M7X3!0A6k6q4!0q4y4q4!0n7z5q4)9^5b7g2!0q4y4#2)9&6b7g2)9^5y4p5#2S2k6$3W2K6K9@1k6J5K9h3c8S2</mark>

![1](upload/attach/202306/804803_7UEVWCNF4X2JRVK.jpg)

#### 2.1.3 电脑工具

- `ifunbox`：上图所使用的iPhone管理软件就是`ifunbox`，进行文件管理挺好用的
- `frida`：和Android一样，电脑上也要有`frida`
- `frida-ios-dump`: 一键脱壳工具。使用：`./dump.py 包名` --> 在当前文件夹下生成 `包名.ipa` 脱壳后的文件
- `class-dump`: 提取所有的头文件，方便查看一个类中的方法。使用：`class-dump -H 脱壳后的主包 -o 导出头文件需要存放路径`
- `iproxy`：端口转发，通过用于`ssh`链接手机。安装方法：`brew install usbmuxd`。使用：`iproxy 2222 22` 转发后，

### 2.2 一些问题

这里记录一下遇到过的问题，及一些杂项。比如手机越狱后，发现开不开机无法进入主界面，有可能是注入的插件有问题。可以通过`ssh`进入手机目录：`$ ssh root@127.0.0.1 -p 2222` ，默认密码是`alpine`。然后可以进入插件列表 `cd /Library/MobileSubstrate/DynamicLibraries`，这里是所有安装过的插件列表，比如我这里是这样的，也可以看到我这个二手手机可能也是用了好几年淘汰下来的

![](upload/attach/202306/804803_9CZXBAUBTT45DN5.jpg)

所以如果你怀疑哪个插件有问题，可以重命名这个插件，然后在上述目录重启系统进程：`killall -9 SpringBoard; killall -9 backboard`。

或者可以重启所有进程：`ldrestart`。总之到这里，我开不开机的问题是解决了。

## 三. 开始逆向

通过上边的知识，即便没有ios逆向基础，也可以开始准备逆向了。

### 3.1 准备

#### 3.1.1 脱壳

由于软件要上架appstore，苹果市场是默认会对应用进行加壳的。因此我们的第一步在appstore下载好相关软件后，就可以进行脱壳。

手机上安装好软件后，电脑开启端口转发：`iproxy 2222 22`。然后进入`frida-ios-dump`脚本的目录直接执行`./dump 包名`

```sh
./dump.py com.xxx
Dumping xxx to /var/folders/rl/6nvyvpmj3z352q0m8xvm0db40000gn/T
[frida-ios-dump]: ZmFFmpeg.framework has been loaded.
[frida-ios-dump]: libswift_Concurrency.dylib has been dlopen.
...
libswift_Concurrency.dylib.fid: 100%|█████████████████████████████| 408k/408k [00:00<00:00, 5.97MB/s]
Validated.plist: 251MB [00:14, 18.2MB/s]
0.00B [00:00, ?B/s]Generating "xxx.ipa"
```

稍等片刻，在当前目录会生成脱壳后的`ipa`文件。然后我们需要找到该应用的主包，以便拖入`ida`分析：

1. 首先将脱壳后的`.ipa`文件改后缀为`.zip`(和Android APK一样，也是个压缩包)，解压后进入`Payload`，会有一个`.app`的文件包

![](upload/attach/202306/804803_R44XG2BYVR3B8Z3.jpg)

2. 点击显示包内容，通常我们要拖入`ida`里分析的包，名字和上层`xxx.app`是相同的，然后就可以拖进`ida`，由于包比较大，`ida`分析时间会很慢

![](upload/attach/202306/804803_U629S63YE47CUAW.jpg)

#### 3.1.2 class-dump

这里也使用`class-dump`将头文件导出`class-dump -H ./osee2unifiedRelease.app/osee2unifiedRelease -o ./osee2unifiedReleaseH`，导出头文件的作用是，方便我们查看OC中类的所有方法/属性

#### 3.1.3 抓包

这里和Android没什么区别，在手机上安装`charles`证书，信任证书，然后抓包。我们关注下相关的登录接口

![](upload/attach/202306/804803_3VEBT7XYEZ8JZGF.jpg)

发现`body`是加密的，于是看看`body`是如何加密的。

### 3.2 逆向分析

#### 3.2.1 通过关键字回溯（未定位到算法）

首先在`ida`里搜索登录相关字符串`/api/account/prod/sign_in`，发现可以直接找到，查看相关交叉引用

![](upload/attach/202306/804803_PQF5FE5HT2MUZM9.jpg)

发现很多，随便找几个先看看，都是调用了同一个函数`sub_1063DF0A8()`，但是奇怪的是这个函数的第二个参数`/api/account/prod/sign_in`，在F5里并没有看到

![](upload/attach/202306/804803_2HC4WWYZV2ZR7SG.jpg)

但是在汇编里是能看到的，我不知道这样做的目的是什么，看了很多iOS逆向的帖子也没有看到这样的情况，或者说这是`ida`反混淆的问题？总之，这不重要。（如果有大佬知道，烦请解答）

![](upload/attach/202306/804803_CXK99YW8EK38E9V.jpg)

然后通过回溯堆栈（`console.log(Thread.backtrace(this.context, Backtracer.ACCURATE) .map(DebugSymbol.fromAddress).join('\n'));`），看能否定位到关键信息。

```c
0x106913598 osee2unifiedRelease!0x63d7598 (0x1063d7598)
0x1069131fc osee2unifiedRelease!0x63d71fc (0x1063d71fc)
0x1068e27d0 osee2unifiedRelease!0x63a67d0 (0x1063a67d0)
0x10690767c osee2unifiedRelease!0x63cb67c (0x1063cb67c)
0x1020724bc osee2unifiedRelease!0x1b364bc (0x101b364bc)
0x10207256c osee2unifiedRelease!0x1b3656c (0x101b3656c)
0x102061e10 osee2unifiedRelease!0x1b25e10 (0x101b25e10)
0x100608f90 osee2unifiedRelease!0xccf90 (0x1000ccf90)
0x1a0ec1298 libdispatch.dylib!_dispatch_call_block_and_release
```

这里和Android逆向`so`也完全一样，稍有区别的是，iOS查找基址填入的是整个库名，如：

```
var base = Module.getBaseAddress("osee2unifiedRelease");
console.log("base: ",base);
```

回溯出来堆栈之后，可以对整个堆栈链路的函数进行分析及`hook`，不过遗憾的是，或许是对iOS网络框架不熟，我并没有办法仅凭查找`url`，就能定位到加密算法。不过逆向有意思的地方也在这里，当一条路走不通了，放松下自己换一条。

#### 3.2.2 通过hook base64

我们观察`body`其实可以发现他是个`base64`，那我们大概猜一下，他使用系统库的方式。

经查资料，可以hook OC中`NSData`的`base64EncodedStringWithOptions`方法，在OC的语法中函数调用的方式可以用`[类名 方法名:参数]`，hook的方式发现网上大多采用`frida-trace`。在我印象里好像没什么印象，即便有，也是听了个名词，因为在Android中我基本没用到过。于是使用这个命令进行hook，减号代表实例方法，相反加号代表类方法，只是个格式而已，也可以用`*`匹配

```c
frida-trace -UF -m "-[NSData base64EncodedStringWithOptions:]"
```

这个脚本会在当前目录生成`./__handlers__/`文件夹，并生成对应函数的`js`代码，发现其实这就是`Interceptor.attach`的那个回调函数，只不过`frida-trace`帮你自动生成好了，方便你改脚本。

```c
{
onEnter(log, args, state) {
this.self = args[0];
},
onLeave(log, retval, state) {
var before = ObjC.classes.NSString.alloc().initWithData_encoding_(this.self, 4);
var after = new ObjC.Object(retval);
log(`-[NSData base64EncodedStringWithOptions:]before=${before}=`);
log(`-[NSData base64EncodedStringWithOptions:]after=${after}=`);
if(after.toString().indexOf("sEn8t")>=0){
console.log(Thread.backtrace(this.context, Backtracer.ACCURATE) .map(DebugSymbol.fromAddress).join('\n'));
}
}
}
```

当然到这里，运气也比较好，发现返回值的确可以跟抓包的`body`对应上，于是打堆栈

```
0x102e0a7d8 osee2unifiedRelease!+[ZHWhiteBoxEncryptTool encryptDataBase64String:]
```
