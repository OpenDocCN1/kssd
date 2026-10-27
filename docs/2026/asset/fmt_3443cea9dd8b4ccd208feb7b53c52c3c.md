
# [原创]以虚御虚-用虚拟机调试vt程式

Intel vt技术十分诱人，但是如何调试vt程式却是让人头疼，让钱包心痛的问题，在走了各种弯路后，终于摸索出一套简单易懂又省钱的vt调试方法。如果你有比我更好的方法请第一个告诉我哟！！！![](https://bbs.kanxue.com/view/img/face/41.gif)

## 准备工作

首先你需要安装以下程式：

1. vmware（版本6.0-7.1）
2. ida（不一定用得到）
3. windbg（也不一定用得到）
4. 自行编译一份带vt的bochs（版本2.4.1-2.4.6，2.4.1中vmx有些部分没实现，推荐用2.4.5以上版本）

然后你需要安装一下操作系统，因为bochs装系统太慢，所以我们这里用vmware生成一个磁盘文件并安装好系统给bochs用。以后也能利用vmware把文件放入虚拟机中的硬盘。

## 创建虚拟机的关键步骤

![](https://bbs.kanxue.com/upload/attach/201105/135566_24iieqbjeontpi5.jpg)

在选择创建硬盘的时候要选择“直接分配所有磁盘空间”。

在创建后会有以下两个文件：

![](https://bbs.kanxue.com/upload/attach/201105/135566_yjsws7kvp8bd40u.jpg)

其中`xxx.vmdk`比较小，存储了硬盘描述信息，其中的信息之后会在bochs中用到；`xxx-flat.vmdk`为硬盘数据，bochs也能使用。

在生成完后你需要安装一个操作系统，这里我使用的是xp sp3。

（注：在vmware安装操作系统的时候，尽量使用原版和纯净版，减少神奇的意外发生。）

打开`xxx.vmdk`查看`# The Disk Data Base`这节，记录下以下信息：

```
ddb.geometry.cylinders = "8322"
ddb.geometry.heads = "16"
ddb.geometry.sectors = "63"
```

打开bochs将这些信息填入：

![](https://bbs.kanxue.com/upload/attach/201105/135566_nxm7ucqjayrm4z9.jpg)

好了，这样bochs和vmware就能共用一个磁盘文件了。

在设置完后，启动bochs时也有一个地方需要注意：

![](https://bbs.kanxue.com/upload/attach/201105/135566_vhria74cnlhiziz.jpg)

因为vmware生成的磁盘文件的`cylinders`不一定是512的倍数，bochs会因为这个问题而发出磁盘大小不匹配的警告，但其实是可以用的，这里直接`continue`就行了。

## 两种调试选择

完成准备工作后，接下来你可以有两种选择（有钱人才有的选啊）：

1. 你拥有ida 5.7及以上版本且使用bochs 2.4.5（或者你拥有ida 5.5且使用bochs 2.4.1）。

    ![](https://bbs.kanxue.com/upload/attach/201105/135566_38ndux8jshisudl.jpg)

    直接使用ida中自带的bochs调试功能连接你编译好的bochs进行调试，轻松方便。（有钱真好，羡慕嫉妒恨啊！）

2. 想用bochs 2.4.5但没有ida 5.7或以上版本。

    使用windbg做为辅助进行调试，配合程式的com输出的信息，在bochs中下断点调试。

    ![](https://bbs.kanxue.com/upload/attach/201105/135566_nzoqn9y8k6q86tf.jpg)

    Windbg的设置和用vmware调试的时候完全一样，如果你以前设置过直接就可以用了。

    一些符号和地址可以由windbg和com输出获得，然后再在bochs中下断点进行调试。虽然步骤比有ida的同学麻烦点，但是买ida的钱是省下来了。

    ![](https://bbs.kanxue.com/upload/attach/201105/135566_cxr7c83xhoc9jg9.jpg)

    上图为bochs中自带的调试界面，可以在这里下断点。

    ![](https://bbs.kanxue.com/upload/attach/201105/135566_donh1qwmghdrxfo.jpg)

    在命令行窗口可以看见系统退出vt的原因。

至此vt的调试环境就搭建好了，勇敢的少年啊快去调试vt!!!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

## 上传的附件

- `32de60ea10c61c63b90e2d91.jpg`（38.66kb，1856次下载）
- `d34fe109401f8162b0351d91.jpg`（6.51kb，1839次下载）
- `0d35221588ea721621a4e993.jpg`（41.23kb，1856次下载）
- `b51f03da21196a7010df9b9d.jpg`（33.64kb，1832次下载）
- `2b20911c79e310a9e1fe0b9c.jpg`（30.76kb，1846次下载）
- `3d45c71df71dc04340341798.jpg`（83.14kb，1834次下载）
- `90e0d83baa9f729ed4622599.jpg`（51.13kb，1834次下载）
- `d34fe10940148162b0351d9e.jpg`（45.29kb，1835次下载）
