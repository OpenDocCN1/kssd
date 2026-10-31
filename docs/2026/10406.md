
# [原创] VS2008、DDK XP 和 DDKWizard 搭建驱动开发环境

在 Windows XP 下，VisualStudio6.0 的编译器无法直接对 DDK XP 的文件进行编译，命令行方式使用起来不方便，无法享受 IDE 带来的便利，因此，寻找其他的解决方案。

下面简要介绍一下配合 DDKWizard 对 Windows XP 下驱动程序的开发环境进行配置：

## 1. 安装 VisualStudio 2008

既然是开发 Windows 下的驱动程序，MS 的开发工具是一定的了，出了 VS2008，当然使用。

## 2. 安装 DDK XP

再此多说几句，我先是安装了 DDK 2003，并选择使用 DDK2003，但是在 Windows XP 系统下编译的时候总提示：

```
error C1083: Cannot open include file: 'D:\WINDDK\3790~1.183\inc\wnet\warning.h': No such file or directory
```

但实际的驱动程序源文件中并没有用到 `warning.h` 文件，由此看来在 XP 系统下使用 DDK 2003 还是有配置不当的问题，但也很有可能是我对 DDKWizard 的使用还有不清楚的地方，总之，使用了 DDK XP 后，一切顺利。

## 3. 安装 `ddkwizard_setup`

默认安装，一路 Next 点过即可。

## 4. 拷贝 `ddkbuild.cmd`

将 `ddkbuild.cmd` 拷贝到特定 DDK 版本安装的父目录下，例如，我的 DDK XP 的安装目录是 `D:\WINDDK\2600`，就将 `ddkbuild.cmd` 拷贝到 `D:\WINDDK` 目录下。在此说明一下，安装 DDK 时，无论是 DDK XP、DDK 2003 或 DDK Vista，最好不要改变安装时的默认目录名称，例如 XP 的目录是 `2600`，2003 的目录是 `3790.1830`，因为如果改了目录名称，还要对 DDKWizard 进行更多的修改，麻烦，也没必要。

## 5. 打开 VS2008 进行配置

打开 VS 2008，菜单“工具”->“选项”->左侧“项目和解决方案”->“VC++目录”->右侧“可执行文件”下拉列表->添加一项 `D:\WINDDK`（根据你的实际情况）。如下图：

![](https://bbs.kanxue.com/upload/attach/200807/149995_yc3rwpr4su5tr0o.jpg)

至此，设置工作就都完成了。

## 编译一个驱动程序体验一下

### 1. 新建一个空的驱动程序项目

新建一个空的驱动程序项目，如图：

![](https://bbs.kanxue.com/upload/attach/200807/149995_ozvu1l878tw8a3o.jpg)

点击确定后，出现 DDKWizard 选项对话框：

![](https://bbs.kanxue.com/upload/attach/200807/149995_30p4apbsss8n467.jpg)

如果是 Windows XP 系统，并且安装了 DDK XP，选择 Windows XP DDK 项。对于 Create PREfast configuration 项，建议不选择，因为我选择后编译会有问题（也可能是我对 DDKWizard 理解不深），点击 Finish 结束设置。

### 2. 将驱动程序的所有源文件加入项目中

将驱动程序的所有源文件加入项目中。

![](https://bbs.kanxue.com/upload/attach/200807/149995_bp1kqe17hzufzvc.jpg)

![](https://bbs.kanxue.com/upload/attach/200807/149995_7kjejd3222kpyy8.jpg)

### 3. 点击 F7 进行编译

点击 F7 进行编译，顺利编译。

![](https://bbs.kanxue.com/upload/attach/200807/149995_sk07q4j7sb6lrjf.jpg)

OK，享受 IDE 带来的便利吧！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`1.JPG`](javascript:void(0);)（54.18kb，4944次下载）
- [`2.JPG`](javascript:void(0);)（51.69kb，4916次下载）
- [`3.JPG`](javascript:void(0);)（55.06kb，4894次下载）
- [`4.JPG`](javascript:void(0);)（74.51kb，4885次下载）
- [`5.JPG`](javascript:void(0);)（95.95kb，4901次下载）
- [`6.JPG`](javascript:void(0);)（89.78kb，4885次下载）
