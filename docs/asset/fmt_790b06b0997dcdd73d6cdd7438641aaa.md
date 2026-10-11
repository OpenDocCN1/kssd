
# 福利彩票随机号码产生器 V3.3

> 原文：[`www.pediy.com/kssd/pediy03/forum666.htm`](https://www.pediy.com/kssd/pediy03/forum666.htm)

用 `SMARTCHECK` 破解福利彩票随机号码产生器 V3.3

福利彩票随机号码产生器 V3.3

下载 `http://www.esoftware.com.cn/oload.php?url=ftp://ftp.eware.com.cn/pub/utility/otheru/numberV3.3.zip`

`smartcheck` 载入 `number.exe`

`start` ,注册码填 `123456789` ,点注册,出错,退出

```
+number_load
+showRegister.show
 +reg_click
 +register.show
   +regclick
     RegisterCode.Text
     Val returns double   ***
```
光条停在`**`行,右边你可以看到 `123456789`

点 `View` 菜单中的 `Show all events`

向下找到两个`__vbaVarCmpEq returns`

右边你可以看到 `123456789` 和 `1472654522`(真注册码)

此软件为一机一码,注册信息保存在

`HKEY_CURRENT_USER\\Software\\VB and VBA Program Settings\\Stock-Star-Website\\Number32\\Reg`

`regcodetrue=1472654522`

作者 风飘雪

主页 `http://duba.126.com`

邮件 `gd1@yeah.net`  `4dd466d0e1e0cc123afbe504da2462ca`
