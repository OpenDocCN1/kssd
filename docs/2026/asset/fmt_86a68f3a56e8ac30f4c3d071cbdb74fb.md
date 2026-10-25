
# [原创]Sina登录破解[Go语言]

最近做了一个小破解，仅是练习Go语言之用，不作为其他商业用途，若有不妥欢迎大家指正...

Sina登录的大致流程如下：

1、Client发送一个登录请求：`189K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2&6i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7s2u0G2k6X3W2D9k6g2)9J5c8Y4g2F1L8r3!0Y4K9h3^5`.

2、Server响应请求，返回`Set_Cookies`字段要求Client设置下次请求的cookies(ps:这步我忽略了，实则不该省去)

3、Client请求：

`e72K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2k6$3W2F1i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7%4y4G2i4K6u0r3M7s2u0W2L8r3!0Y4K9h3&6Q4x3X3g2H3K9s2m8Q4x3@1j5`.

`entry=account&callback=pluginSSOController.preloginCallBack&su=base64用户名&rsakt=mod&checkpin=1&client=ssologin.js(v1.4.19)&_=` 请求时间戳(我有偷懒省去了)

4、Server响应请求，返回一些关键参数`servertime`、`nonce`、`pubkey`、`rsakv`、`account`,这些参数是下面发送POST请求需要用到的重要参数

5、根据 `pubkey` 给 `passwd` 加密，之后作为一个 `sp` 字段放在POST包中发送给Server

6、Client发送ＰＯＳＴ数据包

7、Server影响Client，返回一个数据包，`retcode`标志是否成功，若成功，`retcode = 0`，若失败，`retcode =` 非零值

## 一、登录sina账号链接

`0f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2&6i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7s2u0G2k6X3W2D9k6g2)9J5c8Y4g2F1L8r3!0Y4K9h3^5`.

由于我省去了cookies设置，所以此处就不放WireShark抓包的图了...

## 二、开始登录-->登录界面

![](https://bbs.kanxue.com/upload/attach/201703/694771_40z48qk3yvvt9nm.png)

## 三、登录过程

使用截获数据包软件WireShark/BurpSuite，认证成功POST包内容如下：

![](https://bbs.kanxue.com/upload/attach/201703/694771_5h8u7ztnn6ncmzm.png)

可以很容易的得出，发送POST包的地址: `8feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2k6$3W2F1i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7%4y4G2i4K6u0r3L8r3!0Y4K9h3&6Q4x3X3g2H3K9s2m8Q4x3@1k6U0L8r3W2W2L8Y4c8Q4x3@1c8K6M7$3!0D9L8$3N6A6L8W2)9J5k6h3A6K6i4K6t1^5N6U0q4Q4x3X3f1@1i4K6u0W2x3e0W2Q4x3U0W2Q4x3U0k6S2L8i4m8Q4x3@1u0Q4y4h3k6Q4x3@1b7`.

这里是个时间戳，也可以省略，它的效果图是：

![](https://bbs.kanxue.com/upload/attach/201703/694771_z8kk7xhzbk78j9n.png)

解析关键参数：

1）`su` ： base64加密的用户名

2）`sp` ： 经过服务器返回的公钥(后面提到)加密的密码

3）`servertime` ：请求时间戳

4）`nonce` : 计算加密密码时候使用

5）`rsakv` ：其实我也不知道是嘛

6）`sr` ：当前使用机器分辨率，这里根据自己的机器分辨率修改就可以了

其中，`servertime`、`nonce`、`rsakv`可以从服务器返回，从分析截获数据包可知请求的地址，先上个数据包截图：

========>>>请求包 WireShark抓包内容如下：

![](https://bbs.kanxue.com/upload/attach/201703/694771_u23ruhrxgvao3rv.png)

========>>>对应的Wireshark抓到的响应包如下：

![](https://bbs.kanxue.com/upload/attach/201703/694771_8bonk5s4wabxk6e.png)

得出请求地址：

`95bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2k6$3W2F1i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7%4y4G2i4K6u0r3M7s2u0W2L8r3!0Y4K9h3&6Q4x3X3g2H3K9s2m8Q4x3@1j5`.

`entry=account&callback=pluginSSOController.preloginCallBack&su=base64用户名&rsakt=mod&checkpin=1&client=ssologin.js(v1.4.19)&_=` 

上图中`Pubkey`是比较重要的，是用于加密密码的公钥

## 四、填充POST包参数

下面就是 填充POST包里的参数，发送POST包验证了，没有圈出的POST包参数是固定的，那么最重要的是获取 `servertime`、`nonce`、`pubkey`、`rsakv`、`account`,填充到POST包中的`su`、`sp`、`servertime`、`nonce`、`rsakv`

### 111.获取参数 servertime、nonce、pubkey、rsakv 源代码

1）首先是计算`su`(PS:请求链接需要这个参数)

看一下，网页自带js`[ssologin.js]`用户名 `su` 加密的代码截图：

![](https://bbs.kanxue.com/upload/attach/201703/694771_sj1ndur0hrl92ik.png)

Golang中这样实现：===============================================

```go
func encryptUname(uname string) string {  // 获取username base64加密后的结果
    urlEncode := url.QueryEscape(uname)  // html转义uname
    return base64.StdEncoding.EncodeToString([]byte(urlEncode))
}
```

2）抓取网页内容

```go
//url的取值为
// url = f4eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2k6$3W2F1i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7%4y4G2i4K6u0r3M7s2u0W2L8r3!0Y4K9h3&6Q4x3X3g2H3K9s2m8Q4x3@1k6Q4x3U0k6F1j5Y4y4H3i4K6y4n7
//entry=account&callback=pluginSSOController.preloginCallBack&su=base64用户名 
//&rsakt=mod&checkpin=1&client=ssologin.js(v1.4.19)&_=

// proxy_addr = "" 这里赋值为空即可
func fetch(url, proxy_addr *string) (html string, oHeader http.Header) {
    client := &http.Client{}
    req, err := http.NewRequest("GET", *url, nil)
    if err != nil {
        log.Fatal(err.Error())
    }
    resp, err := client.Do(req)
    if err != nil {
        log.Fatal(err.Error())
    }
    oHeader = resp.Header
    if resp.StatusCode == 200 {
        robots, err := ioutil.ReadAll(resp.Body)
        resp.Body.Close()
        if err != nil {
            log.Fatal(err.Error())
        }
        html = string(robots)
    } else {
        html = ""
    }
    return
}
```

### 222.填充POST包，发送给服务器

1）上述过程中已经成功获取`servertime`、`nonce`、`pubkey`、`rsakv` 重要参数了

2）计算加密密码

看一下，网页自带js密码 `sp` 加密的代码：

![](https://bbs.kanxue.com/upload/attach/201703/694771_t9puosxo3hux4j2.png)

下面是Golang中实现计算密码RSA加密的代码：

```go
//把字符串转换bigint
func string2big(s string) *big.Int {
    ret := new(big.Int)
    ret.SetString(s, 16)  // 将字符串转换成16进制
    return ret
}

//加密密码
func encryptPassword(context urljson, password string) string {
    pub := rsa.PublicKey{
        N: string2big(context.Key),
        E: 65537,                       // 65537 是上述图片的 10001 的16进制形式
    }
    // servertime、nonce之间加\t，然后在\n ,和password拼接 
    encryString := strconv.Itoa(context.Stime) + "\t" + context.Noce + "\n" + password
    // 拼接字符串加密
    encryResult, _ := rsa.EncryptPKCS1v15(rand.Reader, &pub, []byte(encryString))
    return hex.EncodeToString(encryResult)
}
```

### 333.发送POST包 Golang代码

```go
resp, _ := client.PostForm("91fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2k6$3W2F1i4K6u0W2M7$3W2F1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3M7%4y4G2i4K6u0r3L8r3!0Y4K9h3&6Q4x3X3g2H3K9s2m8Q4x3@1k6U0L8r3W2W2L8Y4c8Q4x3@1c8K6M7$3!0D9L8$3N6A6L8W2)9J5k6h3A6K6i4K6t1^5N6U0q4Q4x3X3f1@1i4K6u0W2x3e0W2Q4x3U0W2Q4x3U0k6S2L8i4m8Q4x3@1u0Q4y4h3k6Q4x3@1c8Q4x3U0k6I4N6h3!0@1i4K6y4n7i4K6u0o6
url.Values{
    "entry":       {"account"},      //固定值
    "gateway":     {"1"},            //固定值
    "from":        {},
    "savestate":   {"30"},           //固定值     
    "qrcode_flag": {"true"},         //固定值
    "useticket":   {"0"},            //固定值
    "pagerefer":   {},
    "vsnf":        {"1"},            //固定值
    "su":          {ssu},            // rsa公钥加密后的密码 =======
    "service":     {"sso"},          //固定值
    "servertime":  {sertime},        //============= 服务器返回值
    "nonce":       {content.Noce},   //============= 服务器返回值
    "pwencode":    {"rsa2"},         //固定值
    "rsakv":       {content.Kv},     //============= 服务器返回值  
    "sp":          {ssp},            // base64加密后的username =====
    "sr":          {"1395*822"},     //屏幕分辨率
    "encoding":    {"UTF-8"},        //固定值
    "cdult":       {"3"},            //固定值
    "domain":      {"sina.com.cn"},  //固定值 
    "prelt":       {"48"},           //下面有解释
    "returntype":  {"TEXT"},         //固定值 
})

b, _ := ioutil.ReadAll(resp.Body)
fmt.Println(string(b))
}
```

解析：`"prelt"` 这个值可以随机构造，js计算代码

`preloginTime = (new Date()).getTime() - preloginTimeStart - (parseInt(result.exectime, 10) || 0)`

然后将 `preloginTime` 赋值给 `prelt`

最后对比一下抓包返回验证通过的 和 程序实现返回结果 验证效果图:

1）服务器返回表示

![](https://bbs.kanxue.com/upload/attach/201703/694771_tqw4qxaqlrl2gh1.png)

2）Golang程序实现的返回结果

![](https://bbs.kanxue.com/upload/attach/201703/694771_2135k7dt7g7zg1r.png)

可以看到 `retcode` 也是 `0`，代表成功，若失败 `retcode`为非零值，还会有一串字符提示 `reason`

小结：主要sina登录流程并不复杂，主要是对于passwd的计算要花点时间，passwd的加密过程就是：

11）处理`pubkey`，处理方式在`ssologin.js`中可以分析得出

22）`servertime`、`nonce`之间加`\t`，然后在`\n` ,和`password`拼接

33）使用处理过的秘钥 对 拼接后的字符串加密

加密流程可以通过调用它自带的`ssologin.js`文件实现，感觉有点麻烦，不如自己写出来呢，还有一点值得
