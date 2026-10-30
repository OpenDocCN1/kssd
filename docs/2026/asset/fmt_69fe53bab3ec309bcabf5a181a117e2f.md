
# [原创]华为HG532路由器RCE漏洞复现

## 前言

作为刚开始学习IOT设备漏洞的新人，看到一篇写得很好的经典IoT漏洞的分析与复现文章，于是尝试复现这些漏洞。期间也是遇到很多问题，对于环境与这些固件设备的了解还很少，所以环境配置等方面与师傅文章里写的不太一样。我的虚拟机是公司设备使用的是NAT连接，如果是桥接的可以去看这位师傅的原文。

参考文章[一些经典IoT漏洞的分析与复现（新手向） - IOTsec-Zone](https://bbs.kanxue.com/elink@df1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8%4c8K6k6h3y4Q4x3X3c8*7L8$3&6W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8U0x3^5y4l9`.`.)

感谢这位师傅的分享。

## 漏洞简介

CVE-2017-17215是CheckPoint团队披露的远程命令执行（RCE）漏洞，存在于华为HG532路由器中。

该设备支持名为DeviceUpgrade的一种服务类型，可通过向`/ctrlt/DeviceUpgrade_1`的地址提交请求，来执行固件的升级操作。

可通过向37215端口发送数据包，启用UPnP协议服务，并利用该漏洞，在`NewStatusURL`或`NewDownloadURL`标签中注入任意命令以执行。

漏洞披露：ccbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3k6$3!0G2k6q4)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2Q4x3X3c8K6K9$3W2V1k6r3W2W2

## 环境准备

我这里使用的是ubuntu18.04

### 配置网络环境

安装网络配置工具：

```
sudo apt-get install bridge-utils uml-utilities
```

随后执行命令：

```
sysctl -w net.ipv4.ip_forward=1
```

![](https://bbs.kanxue.com/upload/attach/202511/1056642_XNUPSG93GWTPSA8.webp)

1. 修改ubuntu网络配置文件`/etc/network/interfaces`

   首先查看我的接口信息

   ```
   ip addr
   ```

   发现接口名为`ens33`，于是修改文件名为如下内容

   ```
   sudo nano /etc/network/interfaces
   ```

   ```
   auto lo

   iface lo inet loopback

   auto ens33

   iface ens33 inet dhcp
   ```

   保存后执行

   ```
   sudo /etc/init.d/networking restart
   ```

2. 编辑qemu的网络接口启动脚本`/etc/qemu-ifup`

   若还没安装qemu，请等qemu安装完成后再配置此项

   ```
   #!/bin/sh

   echo "Executing /etc/qemu-ifup"

   echo "Bringing up $1 for bridge mode..."

   sudo /sbin/ifconfig $1 0.0.0.0 promisc up

   echo "Adding $1 to br0..."

   sudo /sbin/brctl addif br0 $1

   sleep 2
   ```

   若是本身就没有这个文件的话，就先创建后写入，再用命令赋予权限。

   ```
   sudo chmod a+x /etc/qemu-ifup
   ```

   这里我是原本就有这个文件的。

3. 创建包含qemu使用的所有桥的名称的配置文件`/etc/qemu/bridge.conf`

   创建文件夹

   ```
   sudo mkdir -p /etc/qemu
   ```

   修改配置文件

   ```
   sudo nano /etc/qemu/bridge.conf
   ```

   在`/etc/qemu/bridge.conf`中写入下面的语句即可。

   ```
   allow br0
   ```

   ![](https://bbs.kanxue.com/upload/attach/202511/1056642_ZH5VR4R82SWYMFP.webp)

注：原文师傅说在网络环境按上述配置完成后，建议重启下Ubuntu虚拟机。我是菜鸟我听师傅的。

## 解压固件并查看信息

主要用到binwalk工具，及其配套工具sasquatch。

binwalk安装：

```
sudo apt install binwalk
```

sasquatch安装：

```
435K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6@1N6s2W2K6x3q4)9J5c8Y4y4S2M7%4q4#2j5i4c8U0K9l9`.`.
```

安装完成后，执行命令，

```
binwalk -Me HG532eV100R001C02B015_upgrade_main.bin
```

并在解压出来的`squashfs-root`文件夹内可以看到文件系统，包含`bin`，`etc`，`lib`等文件夹。

![](https://bbs.kanxue.com/upload/attach/202511/1056642_77UJ656G2GJCAPK.webp)

![](https://bbs.kanxue.com/upload/attach/202511/1056642_YJ87X93Z4NQQDGM.webp)

我们可以在`bin`文件夹内找到存在漏洞的`upnp`二进制文件，并使用`checksec`查看其信息

```
./checksec.bash --file=/home/ubuntu/桌面/_HG532eV100R001C02B015_upgrade_main.bin.extracted/squashfs-root/bin/upnp
```

![](https://bbs.kanxue.com/upload/attach/202511/1056642_SHJSPWVHS46XTQQ.webp)

RELRO：No RELRO → 没有启用只读重定位，容易被GOT/PLT攻击。

Stack Canary：No canary found → 没有栈保护，栈溢出攻击可行。

NX：NX disabled → 栈可执行，Shellcode 可以直接跑。

PIE：No PIE → 地址不随机化，ROP 攻击更容易。

RPATH / RUNPATH / Symbols：没有特殊保护或符号。

是一个32位大端序的mips架构下的二进制文件，且没有开任何保护。

## 配置qemu虚拟机并连接

qemu的安装命令如下：

```
sudo apt-get install qemu 

sudo apt-get install qemu-user-static

sudo apt-get install qemu-system
```

由于该固件是32位大端序的mips架构，因此，我们也要下载相对应的内核及镜像文件。

下载地址：

```
b45K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8X3#2A6M7s2y4Q4x3V1k6$3L8h3I4A6L8Y4g2^5i4K6u0V1x3W2)9J5k6e0k6Q4x3X3f1K6x3W2)9J5k6o6g2Q4x3X3b7@1K9$3y4Q4x3X3c8E0j5h3I4@1j5b7`.`.

wget -c 238K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8X3#2A6M7s2y4Q4x3V1k6V1k6h3u0A6j5h3&6Q4y4h3k6K6M7i4g2W2k6i4A6W2i4K6g2X3L8h3W2H3M7#2)9#2k6Y4y4@1j5h3&6V1j5i4u0V1i4K6u0W2M7h3y4G2N6K6t1`.
```

创建`start.sh`（NAT + 端口转发 guest:37215 -> host:37215）

```
cat > start.sh <<'EOF'

#!/bin/bash

# Start a MIPS Malta QEMU with NAT and host port forwarding for guest TCP 37215 -> host TCP 37215

# Adjust filenames if you saved kernel or qcow2 under different names/paths.

DIR="$(cd "$(dirname "$0")" && pwd)"

sudo qemu-system-mips \

  -M malta \

  -kernel "$DIR/vmlinux-2.6.32-5-4kc-malta" \

  -hda "$DIR/debian_squeeze_mips_standard.qcow2" \

  -append "root=/dev/sda1 console=tty0" \

  -net nic,macaddr=00:16:3e:00:00:01 \

  -net user,hostfwd=tcp::10022-:22,hostfwd=tcp::37215-:37215

EOF
```

赋予可执行权限

```
chmod +x start.sh
```

列出目录确认文件在位

```
ls -lh
```

返回如下图应该就是没问题了

![](https://bbs.kanxue.com/upload/attach/202511/1056642_EHZMECSXW6MJZDT.webp)

在`~/qemu-mips`目录下执行：

```
./start.sh
```

这会启动 MIPS Malta QEMU，Debian Squeeze 镜像会跑起来。

默认账号密码是：root / root。

可以加上`-nographic`（编辑`start.sh`在`qemu-system-mips`那一行最后加`-nographic`）让 QEMU 输出直接在终端显示控制台，更方便操作。

![](https://bbs.kanxue.com/upload/attach/202511/1056642_RPF2ZXH9QX4U3U2.webp)

进入qemu后接下来就是配置网卡了

```
ip addr      # 查看第一个网卡，一般是 eth1
```

```
nano /etc/network/interfaces
```

修改为：

```
allow-hotplug eth1

iface eth1 inet dhcp
```

然后执行：

```
ifup eth1
```

检查 IP：

```
ip addr
```

你会看到类似：

```
inet 192.168.192.133/24 brd 192.168.192.255 scope global eth1#如果你配置的是 tap + 自己桥接到宿主机的网卡
```

这个IP对应guest（QEMU）内部 IP，host 端通过端口转发可以直接访问 guest 的 37215。但我不太一样，是nat配置所以是10.0开头的ip。

```
2: eth1: <BROADCAST,MULTICAST,UP,LONER_UP> mtu 1500 qdisc pfifo_fast state UNKNO

HN qlen 1000

link/ether 00:16:3e:00:00:01 brd ff:ff:ff:ff:ff:ff

inet 10.0.2.15/24 brd 10.0.2.255 scope,global eth1

inet6 fec0::216:3eff:fe00:1/64 scope site dynamic

valid_lft 86396sec preferred_lft 14396sec

inet6 fe80::216:3eff:fe00:1/64 scope link

valid_ift forever preferred_lft forever
```

由于在qemu中不太好操作，建议用`ssh root@yourip`命令在host主机上连接到qemu虚拟机，这样就能在host主机的终端上操作qemu虚拟机了。

接着，我们需要将固件提取出来的文件系统拷贝到qemu的镜像文件的root目录下。

```
# 在宿主上执行（把路径换成你的实际路径）

scp -P 10022 -r ~/桌面/_HG532eV100R001C02B015_upgrade_main.bin.extracted/squashfs-root root@127.0.0.1:/root/
```

在 guest 里解包

```
cd /root

tar -xzf squashfs-root.tar.gz
```

解压完就可以在 guest 里看到`squashfs-root`目录了。

![](https://bbs.kanxue.com/upload/attach/202511/1056642_AP9M6CZRBBPZWNB.webp)

## 验证漏洞并复现

由于本人是新手菜鸟，所以漏洞的定位与分析还不是很明白这里就不过多描述了，感兴趣的师傅可以去参考winmt师傅的原文[一些经典IoT漏洞的分析与复现（新手向） - IOTsec-Zone](https://bbs.kanxue.com/elink@fdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8%4c8K6k6h3y4Q4x3X3c8*7L8$3&6W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8U0x3^5y4l9`.`.)我这里就直接进行复现了。

我们首先要更换原始镜像文件的根目录为从固件中提取的文件系统的根目录：

```
cd squashfs-root

chroot . sh
```

接着，我们按照官方报告中所说，需要打开37215端口，并向该端口下的`/ctrlt/DeviceUpgrade_1`地址发送数据包，才能启用UPnP服务。

通过

```
grep -r '37215'
```

命令可以查到，`/bin/mic`这个二进制文件中有这个端口

![](https://bbs.kanxue.com/upload/attach/202511/1056642_VMFE3GH3Y9PV2JM.webp)

我们便猜测可以通过运行`/bin/mic`文件来打开37215端口

```
sudo nmap yourip -p1-65535
```

命令扫描到qemu虚拟机中所有打开的端口，或者用

```
nc -vv yourip 37215
```

命令看看能否成功连接上37215端口。

运行`/bin/mic`文件后会报一系列错误信息并最后将卡在那里，这些都不用管，只要37215端口是正常开着的就行，此时不能ctrl+Z关掉mic文件的运行，不然37215端口肯定也会跟着被关闭。

最后，我们就可以打exp并成功复现完CVE-2017-17215这个漏洞了！

附上exp：
