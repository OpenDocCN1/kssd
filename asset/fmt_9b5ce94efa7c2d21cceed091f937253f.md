
# FoxMAIL 口令

> 原文：[`www.pediy.com/kssd/pediy03/forum670.htm`](https://www.pediy.com/kssd/pediy03/forum670.htm)

再狡猾的狐狸也逃不过猎人的眼睛 (手把手教你破 FoxMAIL 口令)

版本 Foxmail3.11 正式版 Build0108

下载 `http://www.esoftware.com.cn/oload.php?url=ftp://ftp.eware.com.cn/pub/network/email/fm311ch0108.exe`

引子：办公室新分配来一小妹，漂亮绝顶，凤中极品。从此人迹罕至的办公室车水马龙，人丁兴旺。无不想讨美女一笑，小弟当然也不例外，极力搜索情报。只因此 MM 用 Foxmail 收信，且加了访问口令。经不起所诱，只得拿此优秀国软开刀，多有得罪，小龙哥不要骂我哦！

## 方法一（推荐使用）

启动 `FOXMAIL`，新建一帐户，加访问口令 `abc369def` (够复杂吧，字母数字都有)。

点击带有锁标志的此帐户，随便输入 `12345678`，不要点确定。

启动 `trw2000`，`ctrl-n` 激活，下万能中断（小小虾，只知道此中断）：

```
bpx hmemcpy
go
```

点确定，被断后执行：

```
bc *
pmodule
```

按 7 下 `F12`，3 下 `F10`，来到 `*` (其他版本好象也是^_^)：

```
0167:00511A11 8B45F4            MOV       EAX,[EBP-0C]
0167:00511A14 8B5628            MOV       EDX,[ESI+28]
0167:00511A17 E81824EFFF        CALL      00403E34    *****
```

执行：

```
d eax  eax=12345678
d edx  edx=abc369def
```

看看表，两分钟，小龙兄弟真照顾咱初学者。

## 方法二

删除 `C:\Program Files\Foxmail\mail\zhangsan(某人)\Account.stg`

此法不推荐（原因，自己试试吧:-))

作者 风飘雪

主页 `http://duba.126.com`

邮件 `gd1@yeah.net`  `4dd466d0e1e0cc123afbe504da2462ca`
