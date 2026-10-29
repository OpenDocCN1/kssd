
# [原创]SpringBoot-MyBatis-luckwheel-master开源代码审计

[TOC]

# 前言

比较简单的代码！

# 环境搭建

Github地址: [GitHub - s6056826/luckwheel: 国产开源幸运大转盘管理系统，积分，倍率，奖品兑换](https://github.com/s6056826/luckwheel)

CSDN介绍地址: [开源大转盘抽奖源码，带后台管理，可管理奖品和奖品中奖概率，java语言实现\_飞吧菜鸟了的博客-CSDN博客\_转盘抽奖源码](https://blog.csdn.net/qq_43645782/article/details/125518307)

下载之后用Idea打开，首先导入数据库

![](upload/attach/202207/809626\_Z2GCV8YJ4Z373SP.jpg)

这个数据库里面只有表，不会自动创建库。所以需要手动创建一个数据库

![](upload/attach/202207/809626\_J6UJGVFSZ6GHFY9.jpg)

## 1、导入数据库

看一眼spring的配置库名叫`youyoudb`

![](upload/attach/202207/809626\_PJXR2CYUNXCMCRU.jpg)

创建跟这个一样的名称，然后导入表即可

![](upload/attach/202207/809626\_CRBBB9Q8DCKZZ2G.jpg)

> 这里的`codepay_order`是我自己创建的，后边会说到为什么

然后idea maven会自动添加下载依赖，等待全部下好之后，启动即可

![](upload/attach/202207/809626\_NCZQSRT7YQW27K9.jpg)

## 2、访问页面

直接访问会提示404

![](upload/attach/202207/809626\_HDFZKEX8TG4WSR5.jpg)

看一眼静态文件配置

![](upload/attach/202207/809626\_NPFF3TXP5K9F78E.jpg)

访问`index.html`和`luckmanager.html`也会提示404

![](upload/attach/202207/809626\_XQCAKZTWZTH6MKU.jpg)

看一眼spring配置

![](upload/attach/202207/809626\_5TPGRDG2P77FDWS.jpg)

注意这里的`server.servlet.context-path`

> `server.servlet.context-path`
>
> 1、`server.servlet.context-path=` # Context path of the application. 应用的上下文路径，也可以称为项目路径，是构成url地址的一部分。
>
> 2、`server.servlet.context-path`不配置时，默认为 `/` ，如：`localhost:8080/xxxxxx`
>
> 3、当`server.servlet.context-path`有配置时，比如 `/demo`，此时的访问方式为`localhost:8080/demo/xxxxxx`

所以在访问的时候需要加上`/luck`来访问

![](upload/attach/202207/809626\_3WG6HA78SK5FEJR.jpg)

## 3、解决参数错误的报错

然后就一直会提示参数错误，看一眼JS

![](upload/attach/202207/809626\_W7D454KDUE8U6K8.jpg)

```javascript
var uid= getQueryString("pnum");
if(uid==""||uid==null||uid===undefined){
alert("非法参数错误，请重新打开页面!");
window.location.reload();
}
```

这里会判断`uid`的值，上面的变量可以看到`uid`来自另外一个函数，这里函数这个`pnum`是GET参数，跟过去看一眼

![](upload/attach/202207/809626\_SJ74PKHU4P9X3UC.jpg)

```javascript
function getQueryString(name) {
var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
var r = window.location.search.substr(1).match(reg);
if (r != null) return unescape(r[2]); return null;
}
```

这里正则匹配我们不需要管，只需要返回值不等于`NULL`即可，所以我们直接在前端构造一个`pnum`参数试试

![](upload/attach/202207/809626\_MVE9TJRTNE4H3WK.jpg)

现在不报错了，这里为什么会不报错主要是因为在数据库中有`id`为1的用户，可以看一眼数据库

![image-20220714161738766](upload/attach/202207/809626\_NJCQPUQH57MY8EA.jpg)

这里的值是我自己加的，所以就不会报错了

> 这里已经任意用户登录了吧

然后在`common.js`中可以看到，其实之前可能有登录功能

![](upload/attach/202207/809626\_DN78WKWQFVMXEH5.jpg)

但是下载下来的源码并没有，开始以为是作者删了，我又去看了一遍commit，发现根本就没有上传登录的前端源码

![](upload/attach/202207/809626\_XS44V9CURNS8UAJ.jpg)

看一眼后台的样子

![](upload/attach/202207/809626\_NZDHFQPCPP2AM2W.jpg)

目前所有的搭建都已经完成了

# 代码审计

## 1、SQL注入一处

我这里是直接搜关键字找执行SQL语句的地方

![](upload/attach/202207/809626\_RDEJR3EPV9XTW5N.jpg)

![](upload/attach/202207/809626\_TFMR2P5D4G7AJXN.jpg)

一眼望去就可以看到SQL注入一个

> `test`接口，参数`uid`

![](upload/attach/202207/809626\_5VDX25Q64BD3PZ4.jpg)

直接构造参数去请求接口

![](upload/attach/202207/809626\_WWS8HWJUD2F6NH7.jpg)

这里直接查询会显示没有这个表，所以按照这里SQL语句的参数去构造表字段即可

![](upload/attach/202207/809626\_HCENCPWK348ZQ5N.jpg)

随后再请求接口

![](upload/attach/202207/809626\_PDGM4JSP2HVSMSH.jpg)

这里只会返回`FALSE`或者`TURE`

![](upload/attach/202207/809626\_DFDQCRZ8T3XDF94.jpg)

但是这里的是会回显报错的，所以构造报错语句的参数

```mysql
select * from codepay_order where pay_id='1'' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0
```

看一下这里报错的语句，首先需要闭合前面的单引号，然后构造`and`后面跟报错的函数最后注释即可，payload如下

```http
GET /luck/wheel/test?uid=1'and+extractvalue(1,concat(0x5c,user()))--' HTTP/1.1
```

![](upload/attach/202207/809626\_5MBJDD2ZAMCWAZM.jpg)

返回了当前用户，这里注释后面需要加上单引号，因为如果不加上，后面那个单引号无法闭合，可以看下图

![](upload/attach/202207/809626\_CJCNRQ2GY2J8FZ6.jpg)

```mysql
select * from codepay_order where pay_id='1'and extractvalue(1,concat(0x5c,user()))--' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0'
```

这里可以看到注释后面的单引号跟`up_time`字段的单引号合在一起了，导致了报错，所以需要加上单引号就可以成功

![](upload/attach/202207/809626\_XX7XP9RJ64EEHQS.jpg)

## 2、SQL注入二处

![](upload/attach/202207/809626\_9ZDC9FKFE68N3QB.jpg)

那么想一下，有没有方法调了`canDog`方法，鼠标放在`canDog`方法上按下Command（Windows下是alt或者ctrl）可以看到有哪些方法调用了该方法

![](upload/attach/202207/809626\_39X27T649TAG9QG.jpg)

`lottery`接口，那么肯定也是有sql注入的，因为这里将`UID`传给了下面

```java
@RequestMapping("lottery")
public JSONObject wheelLottery(String uid){
boolean flag=canDog(uid);
if(!flag){
return fail("没有可抽奖次数");
}
LuckUser luckUser = new LuckUser();
luckUser.setUid(uid);
LuckUser luckUser1 = luckUserService.queryOne(luckUser);
if(luckUser1==null){
luckUserService.add(luckUser);
}
```

![](upload/attach/202207/809626\_WX252A6HK8UD7AM.jpg)

那么这里用同样的payload也可以注入

![](upload/attach/202207/809626\_PZYGP2EAMVYFN2R.jpg)

严格来说这里算是一处SQL注入，因为在业务上只需要修`canDog`这一个接口就可以了

## 3、后台存储XSS

![](upload/attach/202207/809626\_4TBXDH7TS5X4AEC.jpg)

没什么好说的

![](upload/attach/202207/809626\_FZT48Y7B2K9N8MV.jpg)

直接取这里的返回数据展示在前端了，并且在`list`接口也没过滤

![](upload/attach/202207/809626\_JUFPFRKD2VAJV3N.jpg)

![](upload/attach/202207/809626\_2RUJW489VVRNZ27.jpg)

## 4、FastJson 1.2.41 RCE

![](upload/attach/202207/809626\_CJ4YBMHE43Q3F33.jpg)

看到请求中有JSON，马上想到会不会有FastJson，在`pom.xml`中搜了一下，笑嘻了，1.2.41，全局搜一下开没开`autype`

![](upload/attach/202207/809626\_N2HSHTEGAR77ETT.jpg)

![](upload/attach/202207/809626\_XJTM6B2UXKAZZ44.jpg)

但是有个通杀payload可以试试 <1.2.47的双键绕过

> 这里插件也扫到了

![](upload/attach/202207/809626\_YDNSZ8YVD6RPRKU.jpg)

这里的payload用了Unicode编码，解码之后如下

![](upload/attach/202207/809626\_G4Z6EN3WZ6K9B8N.jpg)

看上去和常规的payload差不多，这里只是变成了两个键值对，打`ldap`的话，那就尝试复现一下

![](upload/attach/202207/809626\_YNHBHNRNDD5BQTT.jpg)

![](upload/attach/202207/809626\_UJWMBRWTGHM66GC.jpg)

成功RCE

### FastJson 1.2.41

推荐文章: [FastJson<=1.2.47RCE细枝末节详细分析 - 安全客，安全资讯平台](https://www.anquanke.com/post/id/180725)

## 5、SQL注入三处

![](upload/attach/202207/809626\_WQSNXHKPJHQFYAW.jpg)

![](upload/attach/202207/809626\_P5CKBUZPEYCJRUP.jpg)

跟过去可以看到这里的接口是`/luck/lup/update`

在前端页面中可以看到

![](upload/attach/202207/809626\_9BS6TZERQ4WG4C7.jpg)

构造参数

```json
{"id":1,"exchange":1,
"pname":1,
"uid":1}
```

![](upload/attach/202207/809626\_B2MXAJJ9P7PWSHX.jpg)

这里报错了，看一下具体的接口实现代码

![](upload/attach/202207/809626\_HD9D5HNSHNFD7DQ.jpg)

```java
@Override
protected void afterUpdate(Map map) {
if(map!=null){
Integer exchange = (Integer) map.get("exchange");
if(exchange!=null){
String pname = (String) map.get("pname");
if(pname.contains("元")){
String money = pname.substring(0, pname.indexOf("元"));
int anInt = Integer.parseInt(money);
String uid = (String) map.get("uid");
jdbcTemplate.update("update login_user set user_money=user_money+"+anInt+" where app_login_id='"+uid+"'");
}
}
}
super.afterUpdate(map);
}
```

首选从`map`中get出来`exchange`的值，随后如果`exchange`如果不为空的话，就从`map`中get出来`pname`的值，并赋值给`pname`，Java是强类型的语言这里定义接收`pname`的值使用的是`String`类型，我们POST的参数是`int`类型，就会导致出现报错，并且在下面中会通过`contains`方法来判断`元`这个字是否在`map.get('pname')`这里获取出来的值中，如果不在，也是不会走下面的逻辑的

> Tips: Java `contains()`方法
>
> `contains()` 方法用于判断字符串中是否包含指定的字符或字符串。
>
> 函数原形: `public boolean contains(CharSequence chars)`
>
> - `chars`参数
> - 需要判断的字符
> - Demo举例
>
> ​    `String name = 'UzJu';`
>
> ​    `name.contains("U")`
>
> 这时就会判断，`name`这个字符串中是否存在`U`这个字符，并且返回`True`和`False`

所以这里需要将`pname`的值更改一下

![](upload/attach/202207/809626\_X58J9AMYHFEMPPA.jpg)

可以打上一个断点，随后再次请求这个接口，然后还是会报错`String`类型的错误，这时候DEBUG看一眼，会发现漏了一个点

![](upload/attach/202207/809626\_E3SKB4V8XYRCM3H.jpg)

这里的`UID`需要是`String`类型，我们传入的是`int`，所以修改为`String`再次请求

![](upload/attach/202207/809626\_JQPSV74BH3U7YTS.jpg)

随后发现这里没有这个表，自己去新增

![image-20220714162155169](upload/attach/202207/809626\_K2FW6VAKRJXP99U.jpg)

![image-20220714162215259](upload/attach/202207/809626\_ED9FKP8ECE9T2ZJ.jpg)

添加了6个字段才成功，还是一样白盒模式下可以打印SQL语句
