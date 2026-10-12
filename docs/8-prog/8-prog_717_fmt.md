
# 新年献礼! Windows 下的隐身网络连接~\[附源码\]

> 原文：[`www.pediy.com/kssd/pediy12/128887.html`](https://www.pediy.com/kssd/pediy12/128887.html)

有点标题党了：）

公司里通过路由器与行政管理方式来管理员工上网，默认 `DHCP` 会分配 2 以上的网段，这个地址是不能上网的，0 和 1 网段可以自由上网，好多同事通过手动修改 `IP` 的方式来上网，结果 `mis` 部可以通过流量监控查到上网的计算机名，所以经常有同事被处分。

基于这个原因，本人用开源的 `tcp/IP` 栈与 `Winpcap` 在本机实现了脱离 `Windows` 网络栈的上网方式，用过几个月，`mis` 部经常发现有流量，但就是找不到上网电脑（`mac` 与计算机名都能是手动指定的~：））。哈哈。

最近公司更换了路由，通过路由器直接管理上网（通过 `ip` 与计算机名等多重验证方式），这下没招了，过年了，没哈礼物，放出源码供大家评改：）

运行工程后，会在本地会打开 `127.0.0.1 8070` 的 `socks` 端口，想上网的如 `QQ` 等可以直接设置代理就可以上网了。

项目结构相对清晰，但由于起初项目只是自己用，所以没有太注重工程的组织。大家有什么编译问题留言吧。

如果在集线器网络中，可以用这个工具彻底隐身了，不过我没有这样的环境，（集线器的网络估计不会很多了），这块功能没有去实现。

注：源码中不少核心功能来源于网络（主要依赖于一个开源项目 `LwIP`），我只是在其基础之上进行的拼凑与修改。

再过几个小时就是新的一年了，祝大家新年快乐，阖家幸福，事事如意~：）

## 编译方法与使用(2011-02-04 添加)

1、

添加环境变量到系统中：

- `LWIP_DIR`: points to the main `lwip` tree (the folder where is the `CHANGELOG` file).
- `PCAP_DIR`: points to the `WinPcap` Developer's Packs (containing `'include'` and `'lib'`)

2、修改 `LwIPCVS_lib\win32\lwipcfg_msvc.h` 中的宏使其符合您的电脑配置

```c
#define PACKET_LIB_ADAPTER_NR       0              //适配器号（第几个网卡上网）

#define LWIP_PORT_INIT_IPADDR(addr)   IP4_ADDR((addr), 192,168,2,161)     //IP

#define LWIP_PORT_INIT_NETMASK(addr)  IP4_ADDR((addr), 255, 255, 255, 0)  // 掩码

#define LWIP_PORT_INIT_GW(addr)       IP4_ADDR((addr), 192,168,2,1)      //网关

#define LWIP_PORT_INIT_DNS1(addr)     IP4_ADDR((addr), 202,102,134,68)   //DNS

#define LWIP_PORT_INIT_DNS2(addr)     IP4_ADDR((addr), 202,102,154,3)
```

3、`debug` 时设置启动参数：`8070`　(设置监听端口)

4、运行~体验：）　这时您会发现您的的网络防火墙是不管用的：）

![](img/8ace80d0e64da80507da7aef01cec36e.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) |
| `lwIP_FSW.rar` |
