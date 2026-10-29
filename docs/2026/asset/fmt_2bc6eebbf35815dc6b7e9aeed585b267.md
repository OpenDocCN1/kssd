
[ehids/ecapture](https://bbs.kanxue.com/elink@f92K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2K9r3W2V1M7#2)9J5c8X3g2U0j5i4m8@1N6i4u0W2)

---

# 2 参考

- 通过**tiann/eadb**简化了手机上Debian环境的搭建，打通关键环节。

  [eadb](https://bbs.kanxue.com/elink@07bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9h3q4F1L8W2)9J5c8X3g2S2k6r3t1`.)

- 本文是对**seeflower** eBPF系列文章的copy和实践记录

  [seeflower](https://bbs.kanxue.com/elink@412K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7$3g2W2k6X3I4G2N6$3g2J5i4K6u0W2k6r3g2$3i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5I4x3e0q4Q4x3V1j5`.)

- 本文是对好友**maiyao1988/ebpf-plugin** hook脚本的学习实践

  [ebpf-plugin](https://bbs.kanxue.com/elink@123K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3W2&6j5h3)9I4z5e0R3^5i4K6u0r3k6h3u0H3k6W2)9J5k6s2m8D9N6h3N6A6L8R3`.`.)

---

# 3 在手机上布置Debian环境

- 查看手机内核对BPF支持情况

  手机为Pixel 6 / Kernel 5.10，

  需要的内核功能默认已打开，开箱即用。

  |  |  |
  | --- | --- |
  | 1 | `uname -a` |

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_ADB884DBVXBVYXB.png)

  |  |  |
  | --- | --- |
  | 1 | `zcat /proc/config.gz | grep PROBE` |

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_GXBMF867BEQR6RR.png)

- 下载eadb和debianfs-arm64-full.tar.gz

  230K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9h3q4F1L8W2)9J5c8X3g2S2k6r3u0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_QUTSMAE7QF9Z4X5.png)

- 通过eadb安装debian环境

  |  |  |
  | --- | --- |
  | 1 | `./eadb --ssh root@192.168.x.x -p xxxx prepare -a debianfs-arm64-full.tar.gz` |

  > eadb是对adeb的重新实现（rust重写的），
  > eadb的作用是可以给Android安装一个完整健全的arm debian环境。
  > （注意：配置ssh密钥的过程这里略过，可通过Magisk模块完成。）

- 安装后，通过eadb ssh连接手机即可

  |  |  |
  | --- | --- |
  | 1 | `./eadb --ssh root@192.168.x.x shell` |

---

# 4 在手机Debian中，编译运行bcc helloworld

- 像平时在ubuntu上那样布置eBPF-BCC的常规过程，

  在手机里中来一遍，没什么不一样的。

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10 | `git clone https://github.com/iovisor/bcc.git`<br>`mkdir bcc/build; cd bcc/build`<br>`cmake ..`<br>`make`<br>`sudo make install`<br>`cmake -DPYTHON_CMD=python3 .. # build python3 binding`<br>`pushd src/python/`<br>`make`<br>`sudo make install`<br>`popd` |

- 编译安装后，运行helloworld测试

  可以跑的话，

  到这就算成功了，

  缺陷是还没有IDE，

  不方便开发。

  |  |  |
  | --- | --- |
  | 1<br>2 | `cd /bcc/examples`<br>`python hello_world.py` |

- 安装proxychains

  被墙的话，在手机Debian里git和wget比较慢，

  还是要在手机Debian里配置proxychains之类的代理。

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5 | `apt-get install proxychains`<br>`vim /etc/proxychains.conf`<br>`# ---- 配置proxychains.conf ----`<br>`socks5 192.168.?.? 12345`<br>`# 填写代理ip端口`<br>`# ---- 配置proxychains.conf ----` |

- 安装pip

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4 | `proxychains apt-get install python3-pip`<br>`# 或者`<br>`proxychains wget https://bootstrap.pypa.io/get-pip.py`<br>`proxychains python get-pip.py` |

---

# 5 给手机的Debian配置独立的SSH

- 目的

  有了这个步骤，就可以不用eadb了，

  后面可以直接拿vscode ssh登录到手机的Debian上。

- 在手机Debian中执行以下操作，配置ssh

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8 | `apt-get update`<br>`apt-get install ssh`<br>`vim /etc/ssh/sshd_config # 编辑详细服务配置`<br>`vim id_rsa.pub # 手动粘贴导入公钥`<br>`cat id_rsa.pub >> ~/.ssh/authorized_keys`<br>`chmod 600 ~/.ssh/authorized_keys`<br>`chmod 700 ~/.ssh`<br>`service ssh restart # 手机每次重启后，第一次进入debian，可能都需要手动启动下ssh服务` |

- sshd_config的详细配置

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8 | `vim /etc/ssh/sshd_config`<br>`AuthorizedKeysFile .ssh/authorized_keys`<br>`Port 11111`<br>`PubkeyAuthentication yes`<br>`PermitRootLogin yes`<br>`PasswordAuthentication yes`<br>`GSSAPIAuthentication no # 加速SSH`<br>`UseDNS no # 加速SSH` |

- 修改Debian的root密码

  需要重新修改Debian root密码，登ssh时需要。

  |  |  |
  | --- | --- |
  | 1 | `passwd root` |

- PC客户端上配置一下私钥，然后连接测试

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15 | `$ ssh-add # 密钥文件添加到ssh-agent`<br>`$ ssh-add -L # 检查`<br>`$ ping 192.168.x.x # 测试网络`<br>`$ ssh root@192.168.x.x -p 11111`<br>`Enter passphrase for key '/home/ccc/.ssh/id_rsa':`<br>`root@192.168.x.x's password:`<br>`Last login: Fri Nov 11 12:25:33 2022 from 192.168.x.x`<br>`.`<br>`The programs included with the Debian GNU/Linux system are free software;`<br>`the exact distribution terms for each program are described in the`<br>`individual files in /usr/share/doc/*/copyright.`<br>`.`<br>`Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent`<br>`permitted by applicable law.`<br>`root@localhost:~#` |

---

# 6 通过VSCODE SSH登陆到手机Debian，进行开发

- 安装vscode ssh插件

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_5UQ4977487SPZSR.png)

- 客户端ssh配置

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_788WDAP4K9HYQN2.png)

- 连接后，可访问debian服务端上的文件，这个就是资源管理器

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_63UBZ33EWW6DBWX.png)

- 打开bash

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_JKAQDPZUGZQYMF2.png)

- 安装Remote Development插件，即远程开发插件

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_Y4CK6BBNH7BJXRW.png)

- 安装python插件

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_EZ5G7GP52NCDW92.png)

- 如果要debug启动python脚本，想传递参数的话，需要配置一个启动文件。

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_JEZ89CZTYJN622V.png)

- 配置中，你可能还需要补充一下环境变量

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_RJ5UNW99PJDR485.png)

  |  |  |
  | --- | --- |
  | 1<br>2 | `"cwd": "${fileDirname}",`<br>`"env": {"PYTHONPATH": "${workspaceFolder}${pathSeparator}${env:PYTHONPATH}"},` |

- 最后用这个配置文件，调试启动python脚本。

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_G8K5E3299EMPWHF.png)

---

# 7 hook all syscall

- 项目推荐

  通过**maiyao1988/ebpf-plugin**项目，

  可一键一次性的HOOK全部syscall。

  代码逻辑清晰，

  可根据需求自由修改。

  （本文只关注eBPF亮点，无痕hook系统调用）

- 项目地址

  f0bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3W2&6j5h3)9I4z5e0R3^5i4K6u0r3k6h3u0H3k6W2)9J5k6s2m8D9N6h3N6A6L8R3`.`.

- 在手机Debian上运行一下，看看效果

  |  |  |
  | --- | --- |
  | 1<br>2<br>3<br>4<br>5 | `# mkdir bcc/my`<br>`# cd bcc/my`<br>`# git clone 169K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3W2&6j5h3)9I4z5e0R3^5i4K6u0r3k6h3u0H3k6W2)9J5k6s2m8D9N6h3N6A6L8R3`.`.<br>`# cd ebpf-plugin`<br>`# python btrace.py -m64 -n com.android.bankabc` |

  ![](https://bbs.kanxue.com/upload/attach/202211/760871_2JNQT8458ENUD5U.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-11-16 15:56 被爱吃菠菜编辑，原因：

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
