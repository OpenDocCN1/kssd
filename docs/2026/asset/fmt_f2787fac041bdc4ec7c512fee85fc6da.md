
![图片描述](upload/attach/202201/767964_WK5BYBGZHMDGUYF.png)

可以看到输入的字符串已满足条件，跳转不成功，导致隐藏信息被调取

扩展分析：

`gets`函数

```
网络资料：gets 可以无限读取，不会判断上限，也就是说，gets 函数并不检查缓冲区 buffer 的空间大小，事实上它也无法检查缓冲区的空间。
```

结合分析因程序使用`gets`函数获取输入数据导致存在溢出漏洞，其中此程序根据此隐患隐藏了特定字符。

# 参考资料

* 《STACK1_VS_2017.cpp》
* 《gets和fgets函数及其区别，C语言gets和fgets函数详解》

![](https://bbs.kanxue.com/upload/attach/202109/767964_U5AHSS4QCG5XT26.png)

今天学习的内容是动静结合分析病毒，若有不当之处希望伙伴们可以指出，让我们共同完善，共同进步

注意：文章内容只是分享自身的知识储备，只是自身的理解，不一定准确，希望观看的时候只是参考，思维不要被其所局限。

开始进入正文：

![](https://bbs.kanxue.com/upload/attach/202109/767964_D46RXGG5T8TWN37.gif)

# 实战讲解

根据之前所学的静态、动态分析技术，结合起来分析测试样本

## word宏病毒

### 表层

- 打开途中连接外部地址下载文件

  ![图片描述](https://bbs.kanxue.com/upload/attach/202109/767964_UBFTAEC9J76X9SN.png)

- 打开word文档后，发现计算器程序被打开（这里代表恶意行为）

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_5ZXR7N3ACNX6KDV.png)

### 监控

- 网络监控

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_TM3TXMZBY2NFHN7.png)

  分析：

  发起程序-`WINWORD.EXE` *64位

  请求地址-`cf9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0R3$3i4K6u0W2x3e0b7@1i4K6u0r3`

  响应服务-`python3`的`SimpleHTTPServer`模块开启的web服务

  请求文件-

  |  |  |
  | --- | --- |
  | 1  2 | `http://192.168.86.144/word.cab`  解析：`cab`是windows的压缩格式，我这边使用`7-z`提取 |

- 行为监控

  寻找打开word文件主程序

  任务管理器：

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_4EWCUSHYWTWHYVR.png)

  与网络监控到发起程序相同

  进程树查看执行流程：

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_DTV64YUKA4HWF9X.png)

  分析：

  |  |  |
  | --- | --- |
  | 1  2  3 | 正常：就是打开一个word文档  可疑：调用了控制面板程序`control.exe`  并通过控制面板执行临时文件夹下名为`msword.inf`配置文件，之后通过运行`cmd`程序打开计算器程序`calc.exe`(这里代指恶意程序） |

### 本身

- 7-z查看word文档

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_VBDKGDSTGXHEKBN.png)

- 找到了是如何访问可疑地址的方式

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_ZUJVJF94VYERYZ4.png)

  结合之前网络分析访问此地址后跳转下载可疑文件

### 下载的文件

- 解包下载的文件

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_PR4VZZX2CXTP932.png)

  可直接打开提取（7-z提取也可，效果一样，但解包后的大小不同）

  找到了控制面板程序执行的配置文件`msword.inf`

- 运行测试

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_NMY7VMKQJXAY3JV.png)

  成功执行，验证是通过该文件为加载的恶意代码

- 字符串查看

  使用`cutter`查看下载的文件`word.cab`

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_GN38C8BM8Z3BMJA.png)

  看到了计算器相关内容（`calc.exe` 这里代指恶意程序）

- 调试监控

  关键函数运行完毕，恶意程序已成功加载

  |  |  |
  | --- | --- |
  | 1  2 | 关键`call:`  `00007FF67F1128E3` `| E8 C8F4FFFF              | call control.7FF67F111DB0`               `|` |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_7KBRQHQ4BKBXB9M.png)

  进入查看（已加载恶意配置文件）

  ![](https://bbs.kanxue.com/upload/attach/202202/767964_2QGC2USJRVB54YG.png)

  `ShellExecuteExW`函数（可通常此函数执行外部程序或打开文件）

  ![](https://bbs.kanxue.com/upload/attach/202202/767964_NFQRWM2RMK8DQJ9.png)

  `rundll32.exe`(`rundll32.exe`用于在内存中运行DLL文件，它们会在应用程序中被使用,可用于在内存中执行恶意dll)

  ![](https://bbs.kanxue.com/upload/attach/202202/767964_V6CHP3H3M348ZSX.png)

  |  |  |
  | --- | --- |
  | 1  2  3 | `*cmd,*bat` `//`猜测在这里准备好了dos环境  `00007FFDF9DD11ED` `|` `48:8D15 94DA4800`         `| lea rdx,qword ptr ds:[7FFDFA25EC88]     | rdx:L"*.CMD;*.BAT",` `00007FFDFA25EC88:L"*.CMD;*.BAT"`  `00007FFDF9DD11F4` `|` `48:8BC8`                  `| mov rcx,rax                             | rax:L"rundll32.exe"` |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_ZC7M2JBF94WVA8H.png)

  `syscall`（此`syscall`运行完毕，加载恶意配置文件以运行dos环境启动`calc`程序这里代指恶意程序

  `syscall`:表示系统调用）

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_J6C3P86EZ87MB3Q.png)

  反汇编定位：

  `const char Command`启动`calc.exe`

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_KYN9CCYAEQXMVYE.png)

  |  |  |
  | --- | --- |
  | 1 | 定位技巧：字符串搜索关键信息，之后双击进入反汇编页面查看 |

  到此调试程序跟踪与监控的运行流程相同。

## 远控病毒

### 已运行

- 测试机，定位到的可疑进程

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_ZKKUNU9JYWCA9E3.png)

  |  |  |
  | --- | --- |
  | 1  2  3  4 | 在模块列表看到了`RPCRT4.DLL`  文件描述：远程程序调用运行时  在`explorer.exe`进程树下  判断：可疑模块，一般情况程序有远程需求才会加载，当前测试机无已知远程，`test.exe`加载了此模块不正常，并且因此进程在`explorer.exe`进程树下判断为用户启动 |

- 攻击机已控制测试机

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_ZAN6HRRT4CW8RXZ.png)

- 进程树

  找到可疑程序位置，运行并监控，查看进程树

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_8JEUG9RRN9VFYKF.png)

  |  |  |
  | --- | --- |
  | 1  2  3  4 | `test`下调用了某些程序  `ipconfig.exe`：（运行了如下命令：`ipconfig`，行为解析查询当前IP）  `whoami.exe`：（运行了如下命令：`whoami`，行为解析显示已登录的用户信息）  `net.exe`：（运行了如下命令：`net user 123 123 add`，行为解析创建账号行为 ） |

  到此可判断`test.exe`执行了恶意行为

- 网络溯源

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_PU3GFYE8AC5ZF8U.png)

  |  |  |
  | --- | --- |
  | 1  2 | 频繁外联此IP：`192.168.86.114`  初步判断为攻击者使用的IP |

### 定位分析

找到程序，载入分析工具分析

![](https://bbs.kanxue.com/upload/attach/202109/767964_ZEEKW76RP638M2Y.png)

- `VirtualAlloc`函数

  |  |  |
  | --- | --- |
  | 1 | Windows API函数，用于申请内存空间 |

- `ExitProcess`函数

  |  |  |
  | --- | --- |
  | 1 | Windows API函数，用于结束调用进程及其所有线程。 |

- 关键函数

  此`call`会调用`syscall`发送网络数据到攻击机

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_QNKHH6N935TQ4T6.jpg)

  |  |  |
  | --- | --- |
  | 1  2  3  4  5  6  7  8 | 关键`call:00007FF8054B2A59` `|` `48:FF15 301F0500`         `| call qword ptr ds:[<&NtDeviceIoControlF |`  进入`call:`  `00007FF8086CCEF0` `|` `4C:8BD1`                  `| mov r10,rcx                             |`  `00007FF8086CCEF3` `| B8 07000000`              `| mov eax,7`                               `|`  `00007FF8086CCEF8` `| F60425 0803FE7F 01`       `| test byte ptr ds:[7FFE0308],1`           `|`  `00007FF8086CCF00` `|` `75 03`                    `| jne ntdll.7FF8086CCF05`                  `|`  `00007FF8086CCF02` `|` `0F05`                     `| syscall                                 |`  `NtDeviceIoControlFile`函数解析：为提供的缓冲区构建描述符，并将非类型化数据传递给与文件句柄关联的设备驱动程序。 |

## Linux病毒查杀

### 网络相关

- `ifconfig`命令

  可查询网卡发送数据情况，对比日期所需，若发送数据过大或接收发送对比差异过段（未下载数据）则说明可疑，存在异常，需深入排查

  |  |  |
  | --- | --- |
  | 1  2  3  4 | `RX Packets` `/` 接收到的数据包数量  `TX Packets` `/` 发送数据包的数量  `RX Bytes` `/` 接收到的字节数  `TX Bytes` `/` 发送出去的字节数 |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_6KHDZZ6JNUCX76X.png)

- `nethogs`命令（正常情况需安装）

  列出进程的网络情况

  |  |  |
  | --- | --- |
  | 1  2  3 | `dev`        网卡  `sent`       发送  `received`   接收 |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_WRUQCB5YJ4W4NSN.png)

  看到进程1在发送与接收数据，查看详情

- `top`命令

  查询指定进程资源占用情况

  |  |  |
  | --- | --- |
  | 1 | `top -p 7132`  `//7132为进程pid` |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_3UP2EFF4XU6YYJ5.png)

  开始监控资源占用不到，后期逐步占用过高，可能为远控木马驻留，后收到命令执行攻击行为如:搜索指定文件

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_D3PQF4CQB54YF3Z.png)

- 根据PID查找进程路径

  |  |  |
  | --- | --- |
  | 1  2  3  4  5  6  7  8  9  10 | `cd /proc/7132`  `//7132为进程pid`  `ls -ail` `//exe对应的就是可执行文件路径信息`  补充知识：  Linux系统上的`/proc`目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，`/proc`是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。  `ls -ail [目录名]：打印当前/指定目录的目录文件`  组合查询：`ls -ail /proc/7132`  若非root权限会导致有些信息查询不到，如提示如下信息：  `ls: 无法读取符号链接'/proc/7132/cwd': 权限不够`  `ls: 无法读取符号链接'/proc/7132/root': 权限不够`  `ls: 无法读取符号链接'/proc/7132/exe': 权限不够` |

  ![](https://bbs.kanxue.com/upload/attach/202109/767964_VNSRBXSCBBAPNQM.png)

- 猜测状态查询外联IP

  |  |  |
  | --- | --- |
  | 1 | `netstat -na|grep ESTABLISHED` |

  ![](https://bbs.kanxue.com/upload/attach/202110/767964_JHHUHSRFJXQPQJE.png)

- 根据端口反查进程

  |  |  |
  | --- | --- |
  | 1 | `netstat -nap | grep 47074` |

  ![](https://bbs.kanxue.com/upload/attach/202110/767964_RY48BF6KFXGW3FA.png)

- 根据pid查看进程内存情况

  |  |  |
  | --- | --- |
  | 1 | `pmap -d 3885` |

  ![](https://bbs.kanxue.com/upload/attach/202110/767964_XF2YW872QQMURHN.png)

- 根据pid关闭进程

  |  |  |
  | --- | --- |
  | 1 | `kill -HUP 3885` |
