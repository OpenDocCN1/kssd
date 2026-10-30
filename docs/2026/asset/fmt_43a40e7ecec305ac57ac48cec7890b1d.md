
# 坏字符问题

构造 exp 时有可能的坏字符：`0x20`（空格）、`0x00`（结束符）、`0x3a`（冒号）、`0x3f`（问号）、`0x3b`（分号）、`0x0a`（`\n` 换行符）等。具体还要看程序如何处理以及转义。

由于 `qemu` 用户模式下仿真执行程序，环境各种因素影响导致执行 `shell` 失败，下面我们使用 `qemu系统模式` 仿真路由器真实环境进行溢出。

# qemu系统模式

这里主要是为了在 `qemu` 虚拟机中重现 `http` 服务。通过查看文件系统中的 `/bin`、`/sbin`、`/usr/bin`、`/usr/sbin` 可以知道 `/sbin/httpd` 应该是用于监听 `web` 端口的 `http` 服务，同时查看 `/htdocs/web` 文件夹下的 cgi 文件和 php 文件，可以了解到接受到的数据通过 `php+cgi` 来处理并返回客户端。

**1. 自己按配置所需写入新建 `conf` 文件内容。**

`find ./ -name '*http*'` 找到 web 配置文件 `httpcfg.php`。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_V8XM6ZY7WMGMR45.png)

查看内容后分析出 `httpcfg.php` 文件的作用是生成供所需服务的 `配置文件` 的内容，所以我们参照里面内容，自己创建一个 `conf` 作为生成的 `配置文件`，填充我们所需的内容。

`conf` 文件内容：

```conf
Umask 026
PIDFile /var/run/httpd.pid
LogGMT On
#开启log
ErrorLog /log
#log文件

Tuning {
    NumConnections 15
    BufSize 12288
    InputBufSize 4096
    ScriptBufSize 4096
    NumHeaders 100
    Timeout 60
    ScriptTimeout 60
}

Control {
    Types {
        text/html    { html htm }
        text/xml    { xml }
        text/plain    { txt }
        image/gif    { gif }
        image/jpeg    { jpg }
        text/css    { css }
        application/octet-stream { * }
    }
    Specials {
        Dump        { /dump }
        CGI            { cgi }
        Imagemap    { map }
        Redirect    { url }
    }
    External {
        /usr/sbin/phpcgi { php }
    }
}

Server {
    ServerName "Linux, HTTP/1.1, "
    ServerId "1234"
    Family inet
    Interface eth0 #对应qemu仿真路由器系统的网卡
    Address 192.168.x.x #qemu仿真路由器系统的IP
    Port "1234" #对应未被使用的端口

    Virtual {
        AnyHost
        Control {
            Alias /
            Location /htdocs/web
            IndexNames { index.php }
            External {
                /usr/sbin/phpcgi { router_info.xml }
                /usr/sbin/phpcgi { post_login.xml }
            }
        }
        Control {
            Alias /HNAP1
            Location /htdocs/HNAP1
            External {
                /usr/sbin/hnap { hnap }
            }
            IndexNames { index.hnap }
        }
    }
}
```

**启动 qemu 仿真路由器系统：**

```bash
sudo qemu-system-mipsel -M malta -kernel vmlinux-3.2.0-4-4kc-malta -hda debian_squeeze_mipsel_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net tap -nographic
```

| 参数 | 说明 |
| --- | --- |
| `-M malta` | 指定要仿真的开发板：`malta` |
| `-kernel` | 要运行的镜像 |
| `-hda` | 指定硬盘镜像 |
| `-append cmdline` | 设置 Linux 内核命令行、启动参数 |
| `-net nic` | 为虚拟机网卡（默认为 `tap0`） |
| `-net tap` | 系统分配 `tap` 设备（默认为 `tap0`） |
| `-net nic -net tap` | 将虚拟机的网卡 `eth0` 连接真机里的 `tap0` |
| `-nographic` | 非图形化启动，使用串口作为控制台 |

**坑点4：** `8a7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8R3`. 此处下载的 `-kernel` 和 `-hda` 的文件一定要同一个目录下的，并且匹配上的程序的 `大小端序`，否则执行会出现 ![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_K8J5HNY3CNBTBSV.png) 此内容提示。

**2. 测试两台主机 ping 通网络情况。**

`qemu` 网络配置参考：[家用路由器研究详解入门（内含仿真环境搭建）](<https://bbs.kanxue.com/elink@e8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7@1x3K6l9&6x3K6l9H3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0R3#2x3U0j5J5x3K6f1>). 两台主机互通按里面内容配置即可，如想联通外网，可按一把梭方法（硬核联网）。

**3. 将固件的提取的文件系统（在 Ubuntu 上）利用 `scp` 命令拷贝到 mipsel 虚拟机中。**

```bash
sudo scp -r squashfs-root root@192.168.x.x:/root/
```

[scp命令使用参考](<https://bbs.kanxue.com/elink@16fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5N6h3&6G2L8$3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3I4A6L8Y4g2^5i4K6u0V1j5$3!0E0L8g2)9J5k6s2y4U0M7q4)9J5k6h3S2@1L8h3H3>).

**4. 之后编写 `copy.sh` 脚本配置启动 `http` 服务需要的环境包括动态链接库，以及 `conf` 配置文件中提到的 `/usr/sbin/phpcgi`、`/usr/sbin/hnap`。**

```bash
cp conf /
cp sbin/httpd /
cp -rf htdocs/ /
rm /etc/services
cp -rf etc/ /
cp lib/ld-uClibc-0.9.30.1.so /lib/
cp lib/libcrypt-0.9.30.1.so /lib/
cp lib/libc.so.0 /lib/
cp lib/libgcc_s.so.1 /lib/
cp lib/ld-uClibc.so.0 /lib/
cp lib/libcrypt.so.0 /lib/
cp lib/libgcc_s.so /lib/
cp lib/libuClibc-0.9.30.1.so /lib/
cd /
ln -s /htdocs/cgibin /htdocs/web/hedwig.cgi
ln -s /htdocs/cgibin /usr/sbin/phpcgi
ln -s /htdocs/cgibin /usr/sbin/hnap
./httpd -f conf
```

记得启动执行（需要进入 `squashfs-root` 目录使用，脚本最后启动了 `http` 服务。）：

```bash
./copy.sh
```

**5. 在浏览器中访问 `conf` 文件中配置的 `192.168.79.143:1234/hedwig.cgi` 文件内容。**

**5.1 访问方式一：**

**坑点5：** 如果你直接在浏览器中输入以上地址，默认 `https` 访问，手动改成 `http` 协议即可看见服务被启动。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_4EXVRGSF38G7XR5.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_3E9K3X69ER572ZP.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_YFT56VKC7G3V9SZ.png)

**5.2 访问方式二：**

在宿主机（Ubuntu）中使用以下命令：其中 `-v` 显示详细信息，`-X` 指定什么指令，`-H` 自定义头信息传递给服务器，`-b` 指定 cookie 字符串。

```bash
curl http://192.168.79.143:1234/hedwig.cgi -v -X POST -H "Content-Length: 8" -b "uid=zh"
```

[curl使用：](<https://bbs.kanxue.com/elink@df5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6s2g2Z5N6h3!0Q4x3V1k6H3i4K6u0r3y4e0j5&6y4e0t1#2y4W2)9J5k6h3S2@1L8h3H3>). 在 Linux 中 `curl` 是一个利用 URL 规则在命令行下工作的文件传输工具，可以说是一款很强大的 `http` 命令行工具。它支持文件的上传和下载，是综合传输工具，但按传统，习惯称 `url` 为下载工具。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_4HY23D2DCS5F3QR.png)

# gdbserver调试

**1. 接下来尝试调试 `/htdocs/web/hedwig.cgi` 文件。**

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_3QFQZ2DTXKZ8F3B.png)

返回 `no REQUEST`，查看 IDA 静态反汇编得知没有指定环境变量 `REQUEST_METHOD` 的值（还是怼时间去逆向分析函数功能，如果能通过浏览器找到相关函数功能说明最好了，节约时间）。所以想要触发漏洞进行调试的话，还是需要通过 `export` 设置相关环境变量。

```bash
export CONTENT_LENGTH="100"
export CONTENT_TYPE="application/x-www-form-urlencoded"
export REQUEST_METHOD="POST"
export REQUEST_URI="/hedwig.cgi"
export HTTP_COOKIE="uid=1234"
```

运行成功：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_RZKK4ESNDP4J2JR.png)

**2. 接下来动态调试确定偏移但是在那之前需要关掉地址随机化，因为 `qemu` 的虚拟机内核开启了地址随机化，每次堆的地址都在变化，导致 `libc` 的基地址也不断在变，所以需要关闭地址随机化。**

```bash
echo 0 > /proc/sys/kernel/randomize_va_space
```

**3. 在 `qemu` 仿真路由器系统中，和 `gdb` 进行动态调试。**

目的：验证地址偏移位置。

**qemu仿真路由器系统中**编写调试脚本：
