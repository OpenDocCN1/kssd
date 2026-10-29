
# [翻译]Pentest-Scripts – 渗透测试脚本收集

此 GitHub repository（仓库）给我们提供了一些可以用来做渗透测试的脚本。

## 情报收集

此文件夹提供了两个脚本：

### `Automate_Enum.sh`

`Automate_Enum.sh`：`Automate_Enum` 是一个 bash 脚本，可以用来在内部或外部漏洞评估和渗透测试中自动化信息收集工作。此脚本默认运行 `Nmap` 列表和端口扫描。它也可以运行 `EyeWitness`、NetBIOS 扫描（`NBTScan`）、`Nikto`、`WhatWeb` 和 `ZAP` 这些工具【译者注：这些都是可以用于渗透测试的扫描和检测工具，包括下文中提到的 `DIRB`。`Nmap` 是一款端口扫描和网络探测工具。`NBTScan` 是扫描网络 NetBIOS 信息的工具。其他工具主要针对 Web 扫描和检测】。此脚本自动在扫描子网时创建文件夹结构，并随之会在运行 `Nmap` DNS/端口扫描、`Nikto`、`WhatWeb`、`EyeWitness` Web/VNC/RDP 扫描、`DIRB` 和 `ZAP` 时创建相应的文件夹。此[视频](<https://bbs.kanxue.com/elink@a41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2@1N6g2)9J5k6h3u0W2i4K6u0r3N6@1N6&6d9W2y4Q4x3X3c8J5k6K6q4V1z5l9>).（需翻墙）演示了使用 `Automate_Enum.sh` 对一个子网进行扫描，用到的工具有 `Nmap`、`EyeWitness`、`Nikto`、`WhatWeb` 以及 `zapmeweb`，每个工具扫描的数据都保存到了不同的文件夹。

用法：

```bash
Automate_Enum -f filename -eA/-eW/-eV/-eR/ --tellmeweb --zapmeweb --niktomeweb --dirbmeweb
```

- `-f filename`：包含目标 IP 地址或者子网地址的文件的文件名
- `-eA/-eW/-eV/-eR`：用 `EyeWitness` 扫描全部（Web、RDP、VNC），或只扫描 Web，或只扫描 VNC，或只扫描 RDP
- `--nbt`：用 `NBTScan` 扫描指定的 IP 地址或网段
- `--tellmeweb`：用 `WhatWeb` 扫描 HTTP/HTTPS 端口
- `--niktomeweb`：用 `Nikto` 扫描 HTTP/HTTPS 端口（带有侵入性，谨慎使用）
- `--dirbmeweb`：用 `DIRB` 扫描 HTTP/HTTPS 端口（带有侵入性，谨慎使用）
- `--zapmeweb`：用 `ZAP` 扫描 HTTP/HTTPS 端口（带有侵入性，谨慎使用）

### `External_Enum.sh`

`External_Enum.sh`：`External_Enum` 是一个自动化收集网上信息的 bash 脚本。它通过利用 DNS Dumpster API、`Recon-ng`、`theHarvester` 以及 `dnstwist` 来进行 WHOIS、hostsearch（主机搜索）和 dnssearch（DNS 搜索）查询，把所有结果存放在一个文件中，列举出所有找到的域名和邮箱。【译者注：DNS Dumpster API、`Recon-ng`、`theHarvester` 和 `dnstwist` 都是收集信息的工具。DNS Dumpser API 调用了 `5c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3K9g2)9J5k6h3S2S2j5$3E0W2M7Y4c8S2M7X3N6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3S2G2M7%4c8K6k6h3q4J5j5$3S2Q4x3V1k6Q4x3@1k6I4i4K6y4p5*这里填域名*`。`Recon-ng` 是一款强大的 Web 信息侦查收集工具。`theHarvester` 利用公共资源（比如搜索引擎）收集邮箱、子域名、虚拟主机、开放端口等信息。`dnstwist` 是一款可以搜索相似域名的工具。】

用法：

```bash
./External_Enum [-d domain]
```

- `-d domain`：域名

## 漏洞分析

### `Auto_msf.rc`

`Auto_msf.rc`：脚本化的 `auto-msf` 自动化执行 `Metasploit` 中各种基于端口扫描的辅助或其他模块。它还会单独地为每个模块在指定的文件夹中创建日志。

目前，该资源脚本（`.rc` 文件）可针对在任意端口上的服务运行下列模块：

- 检测 FTP 版本（`auxiliary/scanner/ftp/ftp_version`）
- 检测 FTP 匿名登陆（`auxiliary/scanner/ftp/anonymous`）
- 检测 SSH 版本（`auxiliary/scanner/ssh/ssh_version`）
- 检测 SMTP 版本（`auxiliary/scanner/smtp/smtp_version`）
- 检测 SMTP 中继（`auxiliary/scanner/smtp/smtp_relay`）
- DNS 放大检测（`auxiliary/scanner/dns/dns_amp`）
- DNS 缓存提取（`auxiliary/gather/dns_cache_scraper`）
- Finger 指纹模块（`auxiliary/scanner/finger/finger_users`）
- Tomcat HTTP 服务和 SSL/HTTP 服务管理登陆（`auxiliary/scanner/http/tomcat_mgr_login`）
- 检测 HTTP、HTTP/SSL 和 SSL/HTTP 上的 Lotus Domino 版本（`auxiliary/scanner/lotus/lotus_domino_version`）
- 检测 HTTP、HTTP/SSL 和 SSL/HTTP 上的 IIS DAV（`auxiliary/scanner/http/webdav_scanner`）
- 检测 HTTP 和 HTTPS 上的 VMvare ESXi 版本（`auxiliary/scanner/vmware/esx_fingerprint`）
- 登陆 SNMP（简单网络管理协议）（`auxiliary/scanner/snmp/snmp_login`）
- 检测 SNMP（简单网络管理协议）（`auxiliary/scanner/snmp/snmp_enum`）
- 检测 SMB（服务器信息块）版本（`auxiliary/scanner/smb/smb_version`）
- `rexec` 登陆（端口 512），`rlogin`（端口 513）和 `rsh` 登陆（端口 514）（`auxiliary/scanner/rservices/rexec_login`、`auxiliary/scanner/rservices/rlogin_login` 以及 `auxiliary/scanner/rservices/rsh_login`）
- 列举 AFP 苹果归档协议（`auxiliary/scanner/afp/afp_server_info`）
- 检测 HTTP、HTTP/SSL 和 SSL/HTTP 的 SSLv3 HTTPS SSL/TLS 版本（POODLE 漏洞）（`auxiliary/scanner/http/ssl_version`）
- 检测 HTTPS、SSL/HTTP 和 SSL/HTTPS 的心脏出血（Heartbleed）漏洞（`auxiliary/scanner/ssl/openssl_heartbleed`）
- 检测 HTTPS、SSL/HTTP 和 SSL/HTTPS 的 `openssl_css` 漏洞（`auxiliary/scanner/ssl/openssl_ccs`）
- `RSync` 扫描（`auxiliary/scanner/rsync/modules_list`）
- 列举 Java RMI（远程方法调用）`--java-rmi`（`auxiliary/scanner/misc/java_rmi_server`）
- 列举 Java RMI（远程方法调用）`--rmiregistry`（`auxiliary/scanner/misc/java_rmi_server` 和 `auxiliary/gather/java_rmi_registry`）
- 检测 Oracle TNS-LSNR 版本（`auxiliary/scanner/oracle/tnslsnr_version`）
- 检测 TNS 投毒漏洞（TNS Listener 组件漏洞）（`auxiliary/scanner/oracle/tnspoison_checker` 和 `auxiliary/scanner/oracle/sid_brute`）
- 检测 MySQL 版本（`auxiliary/scanner/mysql/mysql_version`）
- 检测 PostgreSQL 版本（`auxiliary/scanner/postgres/postgres_version`）
- 检测 Open X11 漏洞（`auxiliary/scanner/x11/open_x11`）
- 检测 PJL（打印机工作语言）JetDirect 漏洞（端口 9100）（`auxiliary/scanner/printer/printer_version_info`、`auxiliary/scanner/printer/printer_env_vars` 以及 `auxiliary/scanner/printer/printer_list_volumes`）

【译者注：该脚本需要以包含 IP 地址的文件作为输入，默认文件夹和文件名可在代码里查看，默认文件夹为 `/tmp`，默认文件名则与要扫描的服务有关，例如运行检测 FTP 版本模块，则默认文件名为 `ftp`，那么输入文件的默认路径为 `/tmp/ftp`。】

### `iscsiadm.sh`

`iscsiadm.sh` 脚本以一个包含在端口 3260 上运行 iSCSI（Internet 小型计算机系统接口）的 IP 地址的文件作为输入，然后搜索任何 iSCSI 登录入口并尝试登录（如果提供了 `--login` 参数）或者退出（如果提供了 `--logout` 参数）。

用法：

```bash
$1 [-f filename --login/ --logout]
```

- `-f filename`：包含 IP 地址或子网地址的文件的文件名
- `--login`：登陆 iSCSI
- `--logout`：退出 iSCSI

## 支持性的脚本

### `CIFWiki`

`CIFWiki`：CIF Wiki（Collective Intelligence Framework Wiki，信息集合框架维基）是一个将 [CIF Wiki](<https://bbs.kanxue.com/elink@9f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7$3W2J5N6r3N6S2k6r3N6W2N6s2y4Q4x3V1k6E0j5i4y4K6K9i4k6W2i4K6u0V1L8$3y4@1L8#2)9J5k6s2y4H3K9h3y4W2i4K6u0r3N6$3W2C8K9g2)9J5c8W2c8Z5k6g2)9J5k6p5y4u0c8W2)9J5k6p5u0G2L8$3D9>). 转换为 PDF 的简单的 Python 脚本。脚本使用 `Beautiful Soup` 解析 HTML 页面，使用 `markdown-pdf` 工具创建 PDF 文件，并且使用 `PDFtk` 工具来合并所有的 PDF 文件。（这两款工具需要安装，安装 `markdown-pdf` 使用命令 `npm install -g markdown-pdf`，安装 `PDFtk` 使用命令 `apt-get install pdftk`。）

### `DNS-Email.py`

`DNS-Email.py`：`DNS-Email.py` 是一个用于解析在 ViewDNS.info 上反向 WHOIS 查询结果的 Python 脚本。此服务可查出所有通过指定邮箱注册的域名。该脚本在 ViewDNS.info 上查询指定的邮箱，并解析出 CSV 格式的结果。

### `PSSync`

`PSSync`：`PSSync` 是一个 Python 脚本，用于在 SQLite 数据库中同步 PacketStorm 上的 News/Files（新闻/档案）。该脚本利用了 `Beautiful Soup` 来解析 [PacketStorm News](<https://bbs.kanxue.com/elink@9a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3y4C8k6i4c8K6N6r3!0J5L8i4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3L8X3g2%4M7H3>). 和 [PacketStorm Files](<https://bbs.kanxue.com/elink@82eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3y4C8k6i4c8K6N6r3!0J5L8i4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1j5>). 的初始页面。

### `niktomeweb`

`niktomeweb`：在完成 `Nmap`（带 `-oG` 和 `-sV` 参数）扫描后，`niktomeweb` 运行 `Nikto` 检测 `Nmap` 扫描结果中开放 HTTP/HTTPS 端口的主机，并将所有 `Nikto` 检测结果保存在一个文件夹中。

### `tellmeweb`

`tellmeweb`：在完成 `Nmap`（带 `-oG` 和 `-sV` 参数）扫描后，`tellmeweb` 运行 `WhatWeb` 检测 `Nmap` 扫描结果中开放 HTTP/HTTPS 端口的主机，并将所有的 `WhatWeb` 输出保存在一个文件夹中。

### `zapmeWeb`

`zapmeWeb`：在完成 `Nmap`（带 `-oG` 和 `-sV` 参数）扫描后，`zapmeweb` 运行 OWASP ZAP 检测 `Nmap` 扫描结果中开放 HTTP/HTTPS 端口的主机，并将所有结果保存在一个文件夹中。

说明：`niktomeweb` 和 `zapmeweb` 是基于 `tellmeweb` 编写的，`tellmeweb` 最初是由 YGN Ethical Hacker Group，Yangon 和 Myanmar 编写的。

**各个脚本下载地址以及原文链接：**[**cdbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9i4c8$3K9h3A6S2P5i4y4Q4x3V1k6b7k6h3&6@1k6i4y4@1i4K6u0V1f1$3y4J5K9i4m8@1M7H3](<https://bbs.kanxue.com/elink@42aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9i4c8$3K9h3A6S2P5i4y4Q4x3V1k6b7k6h3&6@1k6i4y4@1i4K6u0V1f1$3y4J5K9i4m8@1M7H3>).

**本文由 看雪翻译小组成员 SpearMint 翻译**

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
