
# [原创]国产电纸书Bambook破解笔记

感觉这个属于一个硬件分析的笔记, 比起android版发这里更合适.

word版不知道怎么附件传不上, 只得一点点粘过来

从硬件到软件

# 引子

事情的开头要从一个国产单片机群说起.

群里潜水多年, 经常会遇见一群神人发表这样的言论:

1. 我的代码检测到自己被修改后, 就会擦写自毁
2. 我的代码会把电源连到地上, 让芯片自毁
3. 我的代码会控制一个接地的gpio引脚输出VCC, 烧了芯片
4. 我的代码会控制2个连在一起的gpio一个输出VCC一个输出GND, 烧了芯片
5. 我的电路会在打开产品外壳时候, 产生高压击毁芯片
6. 我的电路会在抄板后, 产生短路, 工作不起来
7. 我的电路板上一些元件是错的, 标的是电阻其实是电容

看起来大多数程序员都对"烧毁芯片"有特殊的爱好

## 锦书硬破

可能以后我会写一些笑话帖子时候, 再继续这个话题, 今天我们说的是, 改造盛大的一个封闭系统电纸书. 当年, 大部分的eink reader都经过深度定制和限制, 不支持用户安装app, 开机自动进入阅读器主程序. 看书, 设置, 书城, 全部一个阅读器apk搞定. 这机器型号是SD928, 面向公众的第一款. 从出错画面看, 系统应该是android.

我曾经想要把它的系统dump出来, 然后研究自制固件, 刷机, root. 为此还买了一个碎屏的机子, 准备拆下来用编程器读取. 但当年我烙铁焊不下来屏蔽罩, 当时也有办法就是从背面挖穿, 但完美拆机主义的我, 舍不得破坏完整的电路板.

很多年过去了, 偶然淘宝一看还有人在卖坏机, 电池鼓包. 卖家表示应该还是好的, 遂77元购入, 拆开一看已经鼓得后盖变形了, 还好电路板没事. 又起了念头继续研究下刷机, 说时迟那时快, 请了热心网友帮我拆下了eMMC芯片, 又吹到eMMC转接板上, 插读卡器, 读取. 然后从dump下来的分区发现, 这机器的eMMC芯片是当作用户数据用的, 系统并不从eMMC启动. 不过eMMC的分区里面, 存储的有解密后的固件镜像. 应该是上次刷机过程残留的. 既然系统不从eMMC引导, 那么是从哪里引导的呢? 在电路板上找到另一块之前忽略的芯片MT29C2G24MAKJAJC-75, eMCP的, 封装了DRAM和NAND flash.

通过解压固件镜像, 我们可以提取系统的rootfs, 通过分析发现固件由PC端发到设备, 设备从服务器获取key, 解密后在`/mnt/data/updates`目录得出`snda_s.firmware`(System), `snda_l.firmware`(Logo), `snda_k.firmware`(Kernel), `boot_nontrust.bin`(Bootloader) 几个文件, 再通过`fwupdate`和`bootloaderupdate`工具刷到eMCP中.

官方的固件里面, `adbd`是禁用的, usb连接上以后就是一个RNDIS设备, 没有其他功能. 如果我们能在系统开机后, 执行`fwupdate`, 我们就可以实现本地刷机了.

之前主板的eMMC拆下后, 量出来了DAT/CLK/CMD等和旁边的电阻对应的接线关系, 我们可以实现免拆芯片在板烧写eMMC, 但还缺乏一个入口.

![](https://bbs.kanxue.com/upload/attach/201704/670_r5g1wgq6w4lv4b7.png)

图1-1 在线烧写eMMC焊线图

经过多次尝试, 发现当在菜单里面选择"恢复工厂设定", 系统就会执行`/mnt/data/scripts/clear-user-data.sh`脚本文件. 同目录下还有个`start.sh`, 但它只在刷机下一次进入系统前执行. 以后每次重启和开机都不会执行. (可能老版本固件也会每次执行吧) 因为我们测试比较频繁, 所以选择修改`clear-user-data.sh`, 去掉该脚本原有的清除用户数据命令, 并且加上如下几行

```
setprop service.adb.tcp.port 5555
/mnt/data/scripts/tinyftp -s 0.0.0.0 -p 3389 -c / > ./ftp.txt &
/sbin/adbd > ./adbd.txt
```

这样在系统设置里面执行"恢复工厂设定"以后, 就会在5555端口提供`adbd`服务并且在3389开启ftp服务. 这个`tinyftp`是从网上的代码修改的, ndk编译, 需要设置`APP_PLATFORM := android-3`. 然后我通过固件解包再打包的方式, 修改`init.rc`, 生成新的`snda_k.firmware`重新刷机, 成功后就开启了`adbd`服务并且可以通过wifi连接上了.

## 硬改到软改

当然, 我不可能让大家都跑去拆机硬改, 这只是挖出厂家做为"秘密"封闭起来的镜像和系统架构的攻城车. 有了`adb shell`以后, 我们可以编译一些工具进系统测试了. 考虑先试试老的CVE有没有办法利用, 查看系统是android 1.5, 内核2.6.28, 找了一个浏览器入口的CVE-2010-1807, 影响Android 2.2以前浏览器. 锦书默认是个封闭系统但后面新的固件支持阅读器(`SndaBrowser`)里面运行html的widget, 可以用来测试该bug.

我做了一个widget打开这个cve网页. 再在阅读器里面载入. 经过反复测试和dump `SndaBrowser`进程, 我修改了Itzhak Zuk的利用代码, 企图让该shellcode在锦书上执行. 最后试验结果是, 会引发Segmentation fault, 内存里面滑板代码也得到了填充, 但并没有成功跳转. 具体分析原因当时钉钉是web版没记录记不清了, 在此感谢村长的帮助.

![](https://bbs.kanxue.com/upload/attach/201704/670_4el7omio44kos9j.png)

图1-2 `SndaBrowser`崩溃前的内存dump

不过不用担心, 别人的洞不好用自己挖也可以. 之前分析它官方PC客户端跟设备之间通讯的时候, 我们在`libSndaEBook.so`的`onSyncServerReceiveFile()`函数看到一些可以利用的地方:

![](https://bbs.kanxue.com/upload/attach/201704/670_g4crrx2x74hg1xv.png)

此bug有2个子bug, 其一是拼接字符串时候, 用到了`filename`, 这个字符串我们可以通过自制PC客户端来控制, 在文件名中传入分隔符后, `system`调用可以执行额外指令. 其二是`install-bookimage.sh`写的不好. 我们来看一下`install-bookimage.sh`:

```
#!/system/bin/sh
echo 'Decompress book image to /mnt/data/'
tar zxf $1 -C /mnt/data/
echo 'Install keybase'
chmod 777 /mnt/data/key/skb-newrsa-file
/mnt/data/key/skb-newrsa-file /mnt/data/key/rsa-pair.txt
echo 'Remove key directory'
rm -r /mnt/data/key/
echo 'Install book image success!!'
```

这个脚本做的事情非常简单, 将PC端发来的文件解压到iNAND的用户分区, 然后给`/mnt/data/key/skb-newrsa-file`加上可执行权限, 再执行它. 我查看了下, 用户分区本身没有这个可执行文件. 也就是说设计上他是上一步释放出来的. 很显然, 这是一个比刚才更好的bug, 它不但能够直接植入文件, 还顺便帮你加可执行权限, 再执行它. 从提示信息来看它应该是厂家为了弄一些"正版图书大礼包"预留的接口, 不是开放给用户使用的. 经过分析, 官方的云梯客户端, 也没有这个接口.

![](https://bbs.kanxue.com/upload/attach/201704/670_hz28bgvh3uee6ib.png)

这里跟刚才那个`case`属于同一个`switch`, 可以看到传送固件也是走的这个接口, 只是封包中包含的`fileType`字段不同. 我们可以模仿官方的协议来通信, 或者偷懒在官方的dll上补一下实现我们的需求. 经过分析`BambookCore.dll`, 我们找到一处给设备传输书架信息的`ReplaceCatelogFile()`函数, 它会给设备发一个`fileType`是12的`catelog.xml`文件. 注意这里的`0D`设备上`switch`的`fileType`将是`0C`.

```
.text:1000B5C7 0A4
C7 44 24 7C 0D 00 00 00
mov     [esp+0A0h+type], 0Dh
.text:1000B5B0 0A4 68 C4 0F 0E 10                          push    offset aCatalog_xml ;
"catalog.xml"
```

PC端通讯参数

![](https://bbs.kanxue.com/upload/attach/201704/670_6nd6sjcxiu1fg19.png)

设备端逻辑分支

这个可以结合上面shell脚本的bug, 实现执行我们的代码. 我做了一个小demo, 把`mov`补为`0E`, 然后发送自己构造的`tar.gz`. 我们把tar包里面`key/skb-newrsa-file`换成了可执行文件(后来发现脚本也能用), 感觉它这个`skb-newrsa-file`原本是可执行文件, 作用是给`snb`文件安装授权的(因为不是用户通过云中书城下载的私有书). 经过测试成功得到执行, 而且这个bug中开启的shell是root权限. 我们设置了`adbd`的属性后直接执行`adbd`, 就可以通过内置的`fwupdate`进行刷机了. 该工具使用未加密非打包的固件镜像, 也不用考虑再封包的问题.

# 由软入硬

## 刷机工具和`Bootloader`

通过分析刷机工具和系统mtd设备, 我们整理出了eMCP的NAND分区划分:

```
0x00000000-0x00100000 : "Bootloader"            active
0x00100000-0x00400000 : "Kernel"
0x00400000-0x04a00000 : "system"
0x04a00000-0x06300000 : "userdata"
0x06300000-0x06400000 : "logo"
0x06400000-0x06500000 : "Bootloader_backup"     factory?      mdtblock5
0x06500000-0x06800000 : "Kernel_backup"         factory       mdtblock6
0x06800000-0x0ae00000 : "system_backup"         factory       mtdblock7
0x0ae00000-0x0af00000 : "logo_backup"           factory       mtdblock8
0x0af00000-0x0b200000 : "Kernel_reserve"        shadow        mtdblock9
0x0b200000-0x0f800000 : "system_reserve"        shadow        mtdblock10
0x0f800000-0x0f900000 : "logo_reserve"          shadow        mtdblock11
0x0f900000-0x0f920000 : "flags"          mtdblock12
0x0f920000-0x0f940000 : "systeminfo"     mtdblock13
0x0f940000-0x0f9c0000 : "EinkFW"         mtdblock14
```

可见`Kernel`/`system`/`logo`都存在了3份拷贝. 最前面是活跃的, 刷机时候`fwupdate`参数可以控制刷入到`backup`(出厂预设)或者`reserve`(正常刷机)的分组, 不会刷新当前活跃的分区. 我们推测`Bootloader`会在刷完机下次引导时候选择`reserve`复制到活跃的分区. 额外提一句, 这里的分区表是在设计系统时候预先划分好的, bootloader和系统都使用同样的硬编码偏移. 所对应的地址就是NAND的原始地址.

我们先看`Bootloader`. 根据Marvell的文档(`PXA3xx_TavorP_BootROM_Ref_Manual.pdf`), 我们知道它是有结构的, bootrom按照头部进行装载. 不是整个装入内存或者挂载到指令总线上从0偏移执行.

![](https://bbs.kanxue.com/upload/attach/201704/670_uvkkz4jwsyc24nf.png)

可以看出这个`Bootloader`没有`Trusted`标记, 有3个子image, 一个是`TIMH`, 就是这个头部本身. 一个是`OBMI`, 位于flash的`0x20000`偏移, 会被bootrom载入到内存的`0x5C013000`, 另一个是`OSLO`, 位于`0x40000`会被载入到`0x81000000`. 那么`OBMI`跟`OSLO`是不是就运行在这2个内存地址呢? 这个我们要再次确认.

查看`OBMI`的`ResetHandler()`, 它会把自身image复制到`0x800007FC`位置, `0x80000000`位置存放的是`ResetHandler()`地址本身, 这中间的`7F8`空间可能是被用作变量空间了. 这类的image, 并不像大家想的那样都是位置无关的, 大部分都是位置相关再加一个检查/复制/跳转代码的.

![](https://bbs.kanxue.com/upload/attach/201704/670_w01avvnbailk66g.png)

举个例子, `iBoot`的`ResetHandler()`, 根据`VectorTable`地址我们可以判断代码需要运行在`9FF00000`地址, 通常`iBoot`的loader会初始化好内存map并将其加载到合适的地址, 但`iBoot`自身也额外带了检查和复制代码. 当发现自己不在指定位置时候, 就会依次复制代码/常量/变量到指定的地址. 这些信息可以用来指导我们进一步划分/增加区段.

![](https://bbs.kanxue.com/upload/attach/201704/670_g5wvb3r4tvu3578.png)

代码2-1 `iBoot`拷贝片段注意`ADR`是取PC相对地址.

![](https://bbs.kanxue.com/upload/attach/201704/670_x7x3oo7h12pwnhi.png)

图2-2 初步整理后的`iBoot`区段

有一些芯片, 是支持从flash直接执行的, 不需要占用内存. 这时候不管是rtOS还是baremetal的image, 都会有一个从链接后的image中, 复制一部分已有初值的变量到sram/dram的流程. 具体的内存布局不同, 要看对应的编译链接脚本和安排这一切的程序员. 介绍这些并不是废话, 在我们分析时候, 进行正确的分段和添加堆的内存区域能提高逆向效率. 例如`iBoot`我们确定了`.data`这部分信息, 逆向过程中可以新建RAM段并且载入, 以便观察一些全局变量的初值.

最后我们确定在`OBMI`中包含名为`Blob`的xscale专用bootloader, 运行在`80008000`. 在flash的偏移是`40000`. 正常刷机流程中, 应该是`fwupdate`设置`systeminfo`分区的里面的几个标志位, 下次启动时候, bootloader检测这个标志位并更新活跃的分区, 再去掉该标志. 而另一套备份固件, 怀疑需要特殊的按键组合触发.

我们怎么找这些按键的组合呢? 首先这些电子设备, 一般不会像电脑一样, 还做一套PS2/USB接口来外挂键盘. 最为常见的线路接法有GPIO单独检测, 行列扫描两种. 两种都支持多键组合检测. 该设备cpu型号是PXA310, 根据marvell提供的介绍, 该cpu具有directkey和matrix key两套键盘接口. 它的matrix扫描是片内设备实现的, 并且支持自动扫描, 我们可以直接从Keypad Matrix Key (KPMK) Register读取有哪几行被按下, 然后从`KPASMKP0`~`KPASMKP3`读出每行按下的列. 每个寄存器包含2行, 一共64个按键都在其中.

通过查找参考手册里面的寄存器地址, 我们在IDA中寻找敏感的地址, 找到如下代码 (参考手册`PXA3xx_DM_vol_III.pdf`, Table 254: Keypad Controller Register Summary)

```
ROM:8000F5FC 41 04 A0 E3+                MOV     R0, #0x41500000 ; keypadbase
ROM:8000F604 3D FD FF EB                 BL      func_config_keymatrix_MFPR
```

各位, 你们一定以为我是二进制搜索`00005041`来定位的吧, 其实还真不是, 我在整理函数名时候把初始化部分挨个看了一遍, 所有落在外设范围内的地址我都肉眼观察了一下. 再说了, XScale是活在没有thumb2(主要是`MOVW`, `MOVT`)的时代, ARM代码中很常见的优化会把一个常数用各种移位和算术运算拼出来, 拼不出来才会用2条`LDR`加一个函数末尾的常数的方式(效率较低). 二进制搜索很难奏效. 不过贴心的IDA里面默认会将拼接常量的指令合并为一条伪指令.
