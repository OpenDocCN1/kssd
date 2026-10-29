
```
Feb 01 xx:25:01 xxx CEF:0|Fortinet|FortiGate-101E|7.0.12,build0523 (GA)|xxx|system event login|4|start=Feb 01 2025 xx:25:01 logver=xxx deviceExternalId=xxx dvchost=xxx vd=root eventtime=xxx tz=-xxx logid=xxx cat=event subtype=system deviceSeverity=information logdesc=Admin login successful sn=xxx duser=watchTowr sproc=jsconsole method=jsconsole src=13.37.13.37 dst=13.37.13.37 act=login status=success reason=none profile=super_admin msg=Administrator watchTowr logged in successfully from jsconsole tz="-xxx"
```

Forescout Research还观察到使用随机生成的五个字符用户名进行初始登录的尝试，这些用户名很可能来自探测漏洞的自动脚本。这些用户名没有添加到本地管理员用户列表中。

记录的活动如下:

```
ISP Null from Null used 1 IPs (13.37.13.37)over 26 days. Total attempts: 3269 (Suspicious: 0 [0%], Success: 3252 [99.48%]). 成功登录的用户包括: hoaue,bhqlz,vuiwe,onmne,durtv,nxbww,cwkvy,ussvm,. 没有发现失败或可疑的用户账户 

在所有来源中观察到的 IP 地址还包括一个 Fortinet FortiGate 设备，共计记录了 3425 条相关事件。

首次发现该活动时间为：2025 年 1 月 30 日 15:18:55.468 UTC

最后一次发现时间为：2025 年 3 月 1 日 00:37:39.303 UTC
```

在成功利用漏洞并使用随机用户名验证访问权限后，攻击者几乎在每次事件中都创建了本地系统管理员用户。新创建的账户包括：`fortticloud-tech`、`fortigate-firewall` 和 `adnimistrator`（拼写错误的管理员）：

以下日志说明了这一活动：

```
{"itime": "xxx", "date": "2025-02-xx", "time": "xx:04:36", "devid": "xxx", "vd": "root", "type": "event", "subtype": "system", "action": "login", "devname": "xxx", "dstip": "13.37.13.37", "eventtime": "xxx", "idseq": "xx", "level": "information", "logdesc": "Admin login successful", "logid": "xxx", "logver": "xxx", "method": "jsconsole", "msg": "Administrator FortiCloud logged in successfully from jsconsole", "profile": "super_admin", "reason": "none", "sn": "xxx", "srcip": "13.37.13.37", "status": "success", "tz": "-xxx", "ui": "jsconsole", "user": "FortiCloud"}
```

```
{"itime": "xx", "date": "2025-02-xx", "time": "xx:xx:xx", "devid": "xxxx", "vd": "root", "type": "event", "subtype": "system", "action": "Add", "cfgattr": "password[*]accprofile[super_admin]", "cfgobj": "forticloud-tech", "cfgpath": "system.admin", "cfgtid": "xxx", "devname": "xxx", "eventtime": "xxx", "idseq": "xxx", "level": "information", "logdesc": "Object attribute configured", "logid": "xxx", "logver": "xxx", "msg": "Add system.admin forticloud-tech", "tz": "-xxx", "ui": "jsconsole(13.37.13.37)", "user": "FortiCloud"}
```

```
Feb 22 03:xx:10 xxx CEF:0|Fortinet|FortiGate-101E|7.0.12,build0523 (GA)|xxx|system event Add|4|start=Feb 22 2025 03:xx:10 logver=xx deviceExternalId=xxx dvchost=xxx vd=root eventtime=xxx tz=-xx logid=xx cat=event subtype=system deviceSeverity=information logdesc=Object attribute configured duser=sys_admin sproc=ha_daemon act=Add cfgtid=xx cfgpath=system.admin cfgobj=admin_support cfgattr=accprofile[super_admin]vdom[root]password[*] msg=Add system.admin admin_support tz="-xx"
```

# 该日志记录了一次通过 ha_daemon 进程在防火墙上添加管理员账户的操作。新增的账户名为 admin_support，拥有超级管理员权限（super_admin），属于 root 虚拟域，并设置了密码。操作者为系统管理员 sys_admin。

# 由于是 HA 守护进程执行的操作，可能是设备高可用性机制的一部分（例如主备同步）。但若此行为非预期，则可能涉及异常配置变更或潜在的安全风险，需进一步审计相关上下文。

在某些情况下，攻击者并不依赖单一管理账户进行所有操作，而是采用了连锁方法，即利用每个新创建的管理账户来生成其他账户。这种方法很可能是为了使修复工作复杂化，增加识别和撤销所有被入侵账户的难度。

Forescout Research观察到的另一种常见利用方法是威胁者使用 `fortigate-firewall` 帐户利用 CVE-2025-24472 而不是 CVE-2024-55591。区别在于利用技术。在调查的一名受害者联系 Fortinet 确认的发现后，Fortinet 于 2 月 11 日更新了其公告，将 CVE-2025-24472 包括在内。

以下日志说明了这一活动:

```
"Local_Process_Access" "Local_Process_Access" "root" "" "" "none" [192.168.1.179]:54145 [192.168.142.132]:80
```

```
{"itime": "xxx", "date": "2025-02-xx", "time": "xx:30:44", "devid": "xxxx", "vd": "root", "type": "event", "subtype": "system", "action": "login", "devname": "xxxx", "dstip": "xxx.xxx.xxx.xx", "eventtime": "xx", "idseq": "xxx", "level": "information", "logdesc": "Admin login successful", "logid": "xxx", "logver": "xxx", "method": "https", "msg": "Administrator fortigate-firewall logged in successfully from https(80.66.8 8.90)", "profile": "super_admin", "reason": "none", "sn": "xx", "srcip": "80.66.88.90", "status": "success", "tz": "-0500", "ui": "https(80.66.88.90)", "user": "fortigate-firewall"}
```

Forescout Research还观察到 PoC 用户 `watchTowr` 的添加，该用户后来在获得访问权限后被攻击者删除。

以下日志记录说明了这一活动:

```
Feb 05 xx:38:21 xxxx CEF:0|Fortinet|FortiGate-101E|7.0.12,build0523 (GA)|xxx|system event Delete|4|start=Feb 05 2025 xx:38:21 logver=xxx deviceExternalId=xxx dvchost=PIT-FG101E-FW1 vd=root eventtime=xxx tz=-0500 logid=xxx cat=event subtype=system deviceSeverity=information logdesc=Object configured duser=watchTowr sproc=jsconsole(13.37.13.37)act=Delete cfgtid=xxx cfgpath=system.admin cfgobj=fortigate-firewall msg=Delete system.admin fortigate-firewall tz="-0500"
```

在创建本地管理员账户后，攻击者下载了防火墙配置文件，其中包含关键信息，包括策略、路由、密钥和 VPN 配置。此外，日志显示攻击者对配置进行了更改，表明系统设置已被修改。

|  |  |  |  |  |
| --- | --- | --- | --- | --- |
| notice | System configuration backed up | User admin backed up the configuration from `jsconsole(127.0.0.1)` | `jsconsole(127.0.0.1)` | `admin` |
| alert | Configuration changed | Configuration is changed in the admin session | `https(80.66.88.90)` | `fortigate-firewall` |

攻击者创建了一个脚本自动化任务，每天在指定时间用 `super_admin` 配置文件和已知密码重新同步 `forticloud-sync` 用户。这确保了即使从防火墙手动删除本地帐户，它也会通过 `system.automation-action` 自动重新创建。

以下日志条目说明了这一活动:

![](https://bbs.kanxue.com/upload/attach/202505/767964_A7BVEQDP9MR936B.jpg)

```
Feb 27 xx:13:49 xxxx CEF:0|Fortinet|FortiGate-101E|7.0.12,build0523 (GA)|xxx|system event Add|4|start=Feb 27 2025 xx:13:49 logver=700120523 deviceExternalId=xxx dvchost=xxx vd=root eventtime=xx x tz=-xxx logid=xxx cat=event subtype=system deviceSeverity=information logdesc=Object attribute configured duser=forticloud-tech sproc=jsconsole(192.248.155.218)act=Add cfgtid=xxxx cfgpath=syst em.automation-action cfgobj=Forticloud-Sync cfgattr=action-type[cli-script]minimum-interval[0]description[Automatically Syncing / Updating the device - user definition]script[config system admin\nedit f orticloud-sync\nset password #^(agT2^R96R-S_l4Y^HS#^\nset accprofile super_admin\nend\nexit]accprofile[super_admin] msg=Add system.automation-action Forticloud-Sync tz="-XX"
```

```
Feb 27 xx:13:49 xxx CEF:0|Fortinet|FortiGate-101E|7.0.12,build0523 (GA)|xxx|system event Add|4|start=Feb 27 2025 xx:13:49 logver=xxx deviceExternalId=xxx dvchost=xxx vd=root eventtime=xxx tz=-xx l ogid=xxx cat=event subtype=system deviceSeverity=information logdesc=Object attribute configured duser=forticloud-tech sproc=jsconsole(192.248.155.218)act=Add cfgtid=xx cfgpath=system.automati on-trigger cfgobj=Forticloud-Sync cfgattr=description[Automatically Syncing / Updating the device]trigger-type[scheduled]trigger-frequency[daily]trigger-hour[15]trigger-minute[30] msg=Add system.aut omation-trigger Forticloud-Sync tz="-xxx"
```

### 3、系统和网络探测

当防火墙具有 VPN 功能时，攻击者会创建本地 VPN 用户账户，账户名称与合法账户相似，但在末尾添加了一个数字。然后将这些新创建的用户添加到 VPN 用户组中，以便今后登录。这种策略很可能是为了在偶然的管理审查中逃避检测，即使最初的入口点被发现，也能保持持续访问。然后，攻击者手动为新创建的用户分配密码。

以下日志记录说明了这一活动：

```
{"itime": "xxx", "date": "2025-02-xx", "time": "18:04:01", "devid": "xxx", "vd": "root", "type": "event", "subtype": "system", "action": "Add", "cfgattr": "type[password]passwd[ENC ]", "cfgobj": "xxx1", "cfgpath": "user.local", "cfgtid": "11075652", "devname": "xxxx", "eventtime": "xxx", "idseq": "xxxx", "level": "information", "logdesc": "Object attribute configured", "logid": "xxxx", "logver": "xxxx", "msg": "Add user.local xxx1", "tz": "-xxx", "ui": "GUI(89.248.192.55)", "user": "fortigate-firewall"}
```

```
{"itime": "xxx", "date": "2025-02-xx", "time": "xx:04:01", "devid": "xxxx", "vd": "root", "type": "event", "subtype": "system", "action": "Edit", "cfgattr": "member[ xxx1]", "cfgobj": "VPN Users", "cfgpath": "user.group", "cfgtid": "xxx", "devname": "xxxx", "eventtime": "xxx", "idseq": "xxx", "level": "information", "logdesc": "Object attribute configured", "logid": "xxx", "logver": "xxx", "msg": "Edit user.group VPN Users", "tz": "-xx", "ui": "GUI(89.248.192.55)", "user": "fortigate-firewall"}
```

|  |  |  |  |
| --- | --- | --- | --- |
| FortiClient VPN disconnected | 2025-02-xxTxx:48:32Z | 2025-02-xxT00:00:00Z | xx1 |
| FortiClient VPN connected | 2025-02-xxTxx:04:15Z | 2025-02-xxT00:00:00Z | xx1 |
