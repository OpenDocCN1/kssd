
# [原创+开源]VM高强度虚拟机保护完全开源,最后更新(2015-10-31)

![](https://bbs.kanxue.com/view/img/face/16.gif)

## 更新日志

2015-10-11 11:52:45 修正Delphi支持

修正PEStruct中部分错误定义 修改是否保护某区段为取属性是否有E的方式 支持Delphi 修正易语言SDK格式问题

2015/10/12:添加Lua模块支持

添加Lua BuildVMCode模块支持,现在可以用Lua写指令处理

2015-10-12：将所有已定义指令全部转换到lua脚本

2015年10月19日

Fix VMStack Check 修正以前生成VM虚拟机时，偶尔出错的问题。 检查VM堆栈。

2015-10-26 修改部分GUI配置

添加QT界面

重写CMakeFiles。目录改变起来更容易了。 添加编译参数 使WProtectGui.exe 打开时隐藏控制台。

2015-10-29 添加完整VS2010支持

CUI + GUI 直接Git下来,CMake生成,然后编译就可以了

2015年10月30日 JunkCode支持

随机给PCode加垃圾指令。 修正LUA的随机数问题 加密脚本更新

2015-10-31 修复BUG

修复在mem操作数scale为0时虚拟机堆栈错误

2015-10-31 Fix vm_add

修复lua vm_and一个错误

现在已经可以编译Windows Linux Mac的WProtect

## 项目说明

引爆一下vm保护的现状

自我感觉就VMCODE解码来说vmp也没有我的强 因为这个vmcode解码是上个vmopcode解码下个vmopcode

其实发不发这个项目我想了很久,然后一脑热就决定无论如何都要发出来  取之与民用之与民  希望极齐7颗龙珠  创造最强软件保护(大伙的力量就是七龙珠)

如果能看懂加密vm的这些代码 大概就可以了解市面上的vm加密过程了(大同小异)  其实vm保护没你们想象的那么恐怖  如果理解了 完全可以还原

代码太大我就不发论坛了

希望大家有能力的捧个提交场,没能力的捧个人场

项目是用gcc和cmake编译的 windows的朋友请下载mingw32 cmake

WProtect项目地址:

`664K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9h3q4G2N6$3g2A6L8h3g2Q4x3V1k6i4f1s2u0G2N6r3g2U0N6l9`.`.

WNetLicensor项目地址:

`755K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9h3q4G2N6$3g2A6L8h3g2Q4x3V1k6i4e0X3g2@1e0r3W2U0k6h3&6K6L8%4t1`.

### 题外话

有些东西发出去了就人也就轻松了

## 编译步骤

重新下载项目源代码

下载`git-for-windows`

下载`cmake`

下载`codeblocks for mingw32` or `vs2010`

### windows:

1. 进入git命令行 输入 `git clone 6f6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9h3q4G2N6$3g2A6L8h3g2Q4x3V1k6i4f1s2u0G2N6r3g2U0N6q4)9J5k6h3N6A6N6l9`.`. 获取项目源代码

2. 然后进入`WProtect`文件夹 新建一个`build`文件夹

3. 运行`CMake` 在where is the source code填入项目地址 在 where to build the binaries输入刚刚新建的`build`目录地址

4. 点击configure 然后在选择编译器的时候选择 `codeblocks-mingw makefiles` 点击 finish ![](https://bbs.kanxue.com/upload/attach/201509/370718_au6rea4qz3yb13w.png)

5. 把选项里面WINDOWS的钩点上,把library_type_shared的钩 去掉  点击 generate 生成

![](https://bbs.kanxue.com/upload/attach/201509/370718_9k3fd4zqicb63zw.png)

6. 进去刚刚新建的`build`目录就能看到`WProtect.cbp`用`codeblocks`打开这个项目文件

![](https://bbs.kanxue.com/upload/attach/201509/370718_z7wisd64g300v9a.png)

7. 点击菜单上的构建菜单 然后点击构建选项 进行编译 稍等就能看到、

![](https://bbs.kanxue.com/upload/attach/201509/370718_sgoib7gdg6p2kez.png)

然后在项目目录的`Bin`目录下的`windows`目录 那个`wprotect.exe`就是 编译出来的文件了

项目目录根目录/`WProtect/bin/Windows/WProtect.exe`

## 使用方法

在代码里面使用WProtect SDK文件 然后编译生成可执行文件 复制到`WProtect/bin/windows`目录

在控制台下切换到`WProtect/bin/windows`目录运行 `WProtect.exe -f` 要加密的文件名

查看加密完成的demo请到这里下载:http://bbs.pediy.com/showthread.php?t=203659

WProtect交流群:210887170

[WProtect.part01.rar](https://bbs.kanxue.com/attach-download-99797.htm)

[WProtect.part02.rar](https://bbs.kanxue.com/attach-download-99798.htm)

[WProtect.part03.rar](https://bbs.kanxue.com/attach-download-99799.htm)

[WProtect.part04.rar](https://bbs.kanxue.com/attach-download-99800.htm)

[WProtect.part05.rar](https://bbs.kanxue.com/attach-download-99801.htm)

[WProtect.part06.rar](https://bbs.kanxue.com/attach-download-99802.htm)

[WProtect.part07.rar](https://bbs.kanxue.com/attach-download-99803.htm)

[WProtect.part08.rar](https://bbs.kanxue.com/attach-download-99804.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [捕获.PNG](javascript:void(0);)

  （116.64kb，377次下载）

- [捕获1.PNG](javascript:void(0);)

  （177.16kb，255次下载）

- [捕获3.PNG](javascript:void(0);)

  （140.64kb，115次下载）

- [捕获6.PNG](javascript:void(0);)

  （166.96kb，446次下载）

- [WProtect.part01.rar](javascript:void(0);)

  （3.00MB，1150次下载）

- [WProtect.part02.rar](javascript:void(0);)

  （3.00MB，999次下载）

- [WProtect.part03.rar](javascript:void(0);)

  （3.00MB，1048次下载）

- [WProtect.part04.rar](javascript:void(0);)

  （3.00MB，1024次下载）

- [WProtect.part05.rar](javascript:void(0);)

  （3.00MB，988次下载）

- [WProtect.part06.rar](javascript:void(0);)

  （3.00MB，1004次下载）

- [WProtect.part07.rar](javascript:void(0);)

  （3.00MB，989次下载）

- [WProtect.part08.rar](javascript:void(0);)

  （1.00MB，876次下载）
