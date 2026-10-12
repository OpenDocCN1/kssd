
# 【成果 3.2】`inf` 文件的编写

> 原文：[`www.pediy.com/kssd/pediy10/57571.html`](https://www.pediy.com/kssd/pediy10/57571.html)

这部分主要讲解 `inf` 文件的编写，其实 `inf` 文件不仅仅可以用来做为安装驱动使用，还有很多其他用法，这里就不涉及了，先讲下 `inf` 的语法，这些语法都是 `google` 上有的。

`inf` 文件其实是一种纯文本文件，可以用任意一款文本编辑软件来打开进行编辑。`inf` 是不区分大小写的。

`inf` 中包含各节的含义：

## 1. `Version` 节

每一个 `INF` 文件都包含一个这样的节，该节中的条目主要是描述此 `INF` 文件支持的设备类型和适用的操作系统。在该节中如果出现 `"signature=$CHICAGO$"` 这样的条目则表示该 `INF` 文件适用于 `Windows 98` 之后的所有操作系统，如果包含 `"signature=$Windows NT$"` 这样的条目则表示该 `INF` 文件适用于 `Windows 2000/XP/2003` 操作系统，而且两者必具其一。

包含在 `version` 中 `class` 说明了设备的类型，常见的类型有：`Display`（显示设备，如显卡）、`Media`（多媒体设备，如声卡）、`Net`（网络设备，通常是网卡）、`Modem`（调制解调器）、`Printer`（打印设备）、`Image`（图像捕获设备，如摄像头）。

## 2. `Manufacturer` 节

该节中的条目主要是描述 `INF` 文件可以识别的所有硬件设备，其中包含有设备的生产厂家，以便设备的正确安装。如 `"%ATi%=ATi"` 指明设备的生产商为 `"ATi"`，这个大家都不会陌生吧，是显卡的生产商。

## 3. `SourceDisksNames` 节

该节主要指明安装文件所在的介质。如 `:"1="ATi Drivers Release CD""` 表明所有驱动程序都在零售版的 `CD` 盘上。

## 4. `SourceDiskFiles` 节

驱动程序文件列表及被安装的位置，该节必须结合 `\[SourceDisksNames\]` 节才能知道具体的位置。如 `"atinbtxx.sys=1"`，则你必须到 `\[DestinationDirs\]` 节去查看 `"1"` 具体代表那个位置。

## 5. `DestinationDirs` 节

`INF` 文件会指示安装程序在安装的过程中，将一些文件复制到硬盘上，或者将硬盘上的一些文件删除、重命名等。该节即指定了为实现上述目的的文件所在的目的路径。

## 6. `DefaultInstall` 节和 `Install` 节

在这一节中描述了设备驱动程序与硬件设备的实际属性。默认情况下，首先执行 `\[DefaultInstall\]` 节，该节指定了要复制或删除的文件，注册表的更新，`INF` 文件的更新等等信息，同时又包含指向其他节的指针。

## 7. `String` 节

这一节中定义了字符串变量，当某些字符串频繁地出现在 `INF` 文件中，为简化输入，可以在该节中定义一个字符串变量，代表该字符串出现在 `INF` 文件中。

（以上解释内容来自 `http://www.roarsoft.net/dispArticle.asp?boardid=34&ArticleID=9200`）

这是 `inf` 文件的基本语法了。现在来看具体的 `inf` 文件的写法，我在这里写了一个最简单的 `helloworld` 驱动，其功能就是打印了一句“hello world”。可以通过 `dbgview` 观察。这个 `inf` 就是为了安装 `helloworld` 这个驱动。

下面是 `inf` 文件：

```
;;;
;;; HelloWorld
;;;
;;;
;;; Copyright (c) 2008, bbs.pediy.com
;;;

\[Version\]
signature       =  "$Chicago$"                    
;这个是必须的,也可以换成 WIndows95 和 nT,但是这个最好,它表示 windows
Class    =  NULL       ;属于什么类型，这里是没有
ClassGuid   =  {00000000-0000-0000-0000-000000000000}   
;该类型的类型 id，这里全填 0
Provider   =  %PEDIY%                             ;提供商的名称
DriverVer       =  01/02/2008,1.0.0.1           ;驱动的版本号

\[DestinationDirs\]
HelloWorld.DriverFiles      = 12      ;%windir%\\system32\\drivers
;12 代表%windir%\\system32\\drivers 这个目录，意思就是将 HelloWorld.DriverFiles 中的内容拷贝到%windir%\\system32\\drivers 中

;;
;; Default install sections
;;

\[DefaultInstall\]
CopyFiles           = HelloWorld.DriverFiles
;CopyFiles 是内建命令,表示拷贝将等号右边的节里面的文件拷贝到目的地


\[DefaultInstall.Services\]
AddService          = %HelloWorldServiceName%,,HelloWorld.Service
;AddService 是内建命令,表示找右边来新建服务.
;%HelloWorldServiceName%这个是服务的名称，加%表示在 string section 中，HelloWorld.Service 代表下面的节点名称

;;
;; Default uninstall sections
;;

\[DefaultUninstall\]
DelFiles   = HelloWorld.DriverFiles
;DelFiles 表示卸载时删除文件

\[DefaultUninstall.Services\]
DelService = HelloWorld,0x200    ; Flags note to stop service first
;删除服务，在删除服务前，先停止服务

;
; Services Section
;

\[HelloWorld.Service\]
DisplayName      = %HelloWorldServiceName%      ;系统服务中显示的名称
Description      = %HelloWorldServiceDesc%      ;系统服务中描述的名称
ServiceBinary    = %12%\\helloworld.sys  ;%windir%\\system32\\drivers\\filespy.sys
ServiceType      = 1       ;SERVICE_KERNEL_DRIVER
StartType        = 3       ;SERVICE_DEMAND_START，手动启动
ErrorControl     = 1       ;SERVICE_ERROR_NORMAL

;
; Copy Files
;

\[HelloWorld.DriverFiles\]
helloworld.sys


;;
;; String Section
;;

\[Strings\]
PEDIY          = "看雪论坛"
HelloWorldServiceDesc = "看雪论坛--HelloWorld.Inf 演示"
HelloWorldServiceName = "HelloWorld"
HelloWorldRegistry    = "system\\currentcontrolset\\services\\HelloWorld"
```

这是一个简单的 `inf`，没有包括所有的 `inf` 节名，但是已经能够正常安装了。只需要右键该文件，然后点击“安装”，就可以安装成功了，由于需要手动启动，所以需要在 `cmd` 下运行 `net start helloworld`,`helloworld` 是我们服务的名称，就可以看到启动成功了。

![](img/e75d20acd5401acca9af8cd4a55331b8.jpg)

在启动服务前，先打开 `dbgview`，就可以看到如下信息：

![](img/12198ca60b8a70f26608c19f27e2b7eb.jpg)

可以看到服务已经启动成功了，也就是说，驱动已经成功加载了！

最后附带 `helloworld.sys`,`helloworld.inf`,`simple helloworld.inf` 代码。`helloworld.inf` 是比较详细的 `inf` 文件。大家可以参考下。

`HelloWorld.rar`
