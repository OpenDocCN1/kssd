
```python
class DeJian:
    def __init__(self, phone):
        self.phone = phone
        self.logger = self.init_log()

    def init_log(self):
        # 创建logger实例
        logger = logging.getLogger('DeJian')
        # 设置日志级别
        logger.setLevel(logging.DEBUG)
        # 流处理器
        ch = logging.StreamHandler()
        ch.setLevel(logging.DEBUG)
        # 日志打印格式
        formatter = logging.Formatter('%(message)s')
        # 添加格式配置
        ch.setFormatter(formatter)
        # 添加日志配置
        logger.addHandler(ch)
        return logger

    def req(self, method, url, **kwargs):
        if kwargs.get("headers"):
            # 如果传递过来的请求有头信息，那么我们就在头信息中做追加
            kwargs["headers"].update = {
                "content-type": "application/x-www-form-urlencoded",
                "Host": "dj.palmestore.com",
                "user-agent": "Dalvik/2.1.0 (Linux; U; Android 8.1.0; Nexus 5X Build/OPM1.171019.011)"
            }
        else:
            # 如果传递过来的请求没有header
            kwargs["headers"] = {
                "content-type": "application/x-www-form-urlencoded",
                "Host": "dj.palmestore.com",
                "user-agent": "Dalvik/2.1.0 (Linux; U; Android 8.1.0; Nexus 5X Build/OPM1.171019.011)"
            }
        for k, v in kwargs["data"].items():
            kwargs["data"][k] = self.url_encode(v)
        kwargs["data"] = self.get_sorted_param_str(kwargs["data"])
        self.logger.debug(f"请求的参数为{method}，url为{url}， 其他参数为{kwargs}")
        r = requests.request(method=method, url=url, **kwargs)
        self.logger.info(f"响应内容为{r.json()}")
        return r.json()

    def url_encode(self, s):
        r = ['+', '/', '=']
        res = s
        for i in r:
            res = res.replace(i, '%' + hex(ord(i))[2:].upper())
        return res

    def sign(self, content):
        private_key = b""
        pri_key = RSA.importKey(private_key)
        signer = PKCS1_v1_5.new(pri_key)
        hash_obj = SHA1.new(content.encode())
        sig1 = signer.sign(hash_obj)
        signature = base64.b64encode(sig1).decode()
        res = self.url_encode(signature)
        return res

    def get_sorted_param_str(self, dic):
        content = ''.join([f'{k}={dic[k]}&' for k in sorted(dic)])[:-1]
        return content

    def sendSms(self):
        url = ''
        dic = {
            "versionId": "20005056",
            "device": "Nexus 5X",
            "flag": "1",
            "imei": "",
            "phone": self.phone,
            "times": "1",
            "sendType": "0",
            "channelId": "1240202",
            "timestamp": "1669439016670",
        }
        dic["sign"] = self.sign(self.get_sorted_param_str(dic))

        r = self.req('post', url, data=dic)
```

可以看到计算得到的`sign`值和抓包得到的`sign`值是一致的

![图片描述](https://bbs.kanxue.com/upload/attach/202212/926331_55PAH3FSDS54FWC.png)

同时返回包也是成功

![图片描述](https://bbs.kanxue.com/upload/attach/202212/926331_YX8TF4K7YADEXH7.png)

手机上也是成功收到了短信

同样地该APP在登陆时的`sign`参数也是类似的逻辑

![图片描述](https://bbs.kanxue.com/upload/attach/202212/926331_HP7SMX2RF39FQFE.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202212/926331_FKRY369Q9MJ5VV9.png)

将之前的代码添加了登录的功能，实现了从获取验证码到登录的过程

![图片描述](https://bbs.kanxue.com/upload/attach/202212/926331_G83GRQZMTGTWW95.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-12-8 18:46

被si1enceZ编辑，原因：
