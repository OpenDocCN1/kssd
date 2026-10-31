
# [原创]Docker搭建pwn环境
## Docker搭建pwn环境

> 最近有点忙，没什么时间和心思学堆了，就先利用零碎时间搭建了一个`docker`上的pwn环境。其实以前就想搭建一个`docker`环境，但是那时候想用`dockfile`来一步到位，奈何自身水平原因，反而花了很多时间，然后就放弃了。现在我直接用`Dockerfile`拉一个基础的`ubuntu`镜像，然后在容器里一个一个安装工具。

## 1. docker常用命令

### 1.1 镜像操作

1) 拉取镜像到本地
```shell
docker pull 镜像名称[:tag]
# e.g
docker pull ubuntu:16.04
```

2) 查看全部本地镜像
```shell
docker images
```

3) 删除本地的镜像
```shell
docker rmi 镜像的标识（id或者镜像名称:tag）
```

4) 镜像的导入导出
```shell
### 将本地的镜像导出
docker save -o 导出的路径 镜像id/镜像名称:tag
#e.g.
docker save -o ./ubuntu.image ubuntu:16.04
### 加载本地的镜像文件
docker load -i 镜像文件
# e.g.
docker load -i ubuntu.image
```

5) 修改镜像的名称
```shell
docker tag 镜像id 新镜像名称:版本（tag）
```

### 1.2 容器操作

1) 运行容器
```shell
# 简单操作
docker run 镜像的标识｜镜像名称[:tag]
# 常用的参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像的标识:镜像名称[:tag]
# -d：代表后台运行容器
# -p 宿主机端口：容器端口 ：为了映射当前主机的端口和容器的端口
# --name：容器名称；指定容器的名称
```

2) 查看正在运行的容器
```shell
docker ps [-qa]
# -a:查看全部的容器，包括没有运行
# -q：只查看容器的标识
```

3) 进入到容器的内部
```shell
docker exec -it 容器id bash
```

4) 停止和删除容器（删除容器前，需要先停止容器）
```shell
# 停止指定容器
docker stop 容器id
# 停止全部容器
docker stop $(docker ps -qa)
# 删除指定容器
docker rm 容器id
# 删除全部容器
docker rm $(docker ps -qa)
```

5) 启动容器
```shell
docker start 容器id
```

6) 从容器创建一个新的镜像
```shell
docker commit [OPTIONS] 容器标识 [镜像名称[:TAG]]
# OPTIONS:
# -a :提交的镜像作者；
# -c :使用Dockerfile指令来创建镜像；
# -m :提交时的说明文字；
# -p :在commit时，将容器暂停。
```

## 2. 搭建步骤

### 2.1 自定义基础镜像

- Dockerfile
```dockerfile
FROM ubuntu:16.04
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt-get clean
RUN apt-get update
RUN apt-get install \
python python3 \
python-pip python3-pip \
python-dev libffi-dev libssl-dev \
vim \
lib32ncurses5 lib32z1 \
ruby gem git \
-y
```

> - `lib32ncurses5` 和 `lib32z1` 是32位库，是为了能够运行32位程序
> - `ruby`和`gem`是为了后面安装`one_gadget`

- 制作自定义基础镜像
```shell
docker build -t pwn:latest .
```
![image-20201222231422961](upload/attach/202012/830671_H9YX5EHG7ZXKEU8.jpg)

> `apt update` 可能会失败，多试几次。

### 2.2 运行容器
```shell
docker run -it --name pwn -v xxx/challenges:/opt/challenges pwn:latest
```
- `-v`：映射数据卷。目的是为了将来把文件放入容器中，当然这里需要你自己建一个空目录，然后映射到容器中。
> 不停止运行退出：先按 `ctrl+p` ，然后按 `ctrl+q`。
![image-20201223001610054](upload/attach/202012/830671_JCFUAJ6XJ743RQ8.jpg)

### 2.3 安装gdb插件

- 进入容器，在`/opt`目录下创建`tools`目录
![image-20201223001742858](upload/attach/202012/830671_GXUBAWMHTKXA37F.jpg)

#### 2.3.1 pwndbg
> pwndgb： <mark class="encrypted">dceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6H3N6$3&6V1j5X3M7`.</mark>

- 官方安装过程：
```shell
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

- 我的安装过程
```shell
git clone https://hub.fastgit.org/pwndbg/pwndbg.git # 高速下载油猴脚本：https://zhuanlan.zhihu.com/p/262905603
cd pwndbg
./setup.sh
```

- 安装成功
![image-20201223005215043](upload/attach/202012/830671_Y9B68BBNUQQ5T87.jpg)

> 期间会安装python库，可能因为源和网络的原因会安装失败，这时候有两个选择：
> 1. 重试`./setup.sh`
> 2. 哪个python库因为网络原因安装失败，那就自己指定源，先用`pip3`安装它。

#### 2.3.2 安装peda-heap
> 这里我就不安装`peda`了，因为`peda-heap`是在`peda`的基础上添加了一些调试堆的命令。
> 而且`peda`的安装过程也和`peda-heap`基本一样。
> `peda`： <mark class="encrypted">721K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3&6Y4L8r3c8Q4x3V1k6H3k6h3c8S2</mark>
> `peda-heap`：<mark class="encrypted">363K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9i4m8#2z5e0c8Q4x3V1k6H3k6h3c8S2i4K6u0V1K9r3g2S2M7l9`.`.</mark>

- 官方安装过程：
```
git clone git://github.com/Mipu94/peda-heap.git ~/peda-heap
echo "source ~/peda-heap/peda.py" >> ~/.gdbinit
```

- 我的安装过程：
```
git clone https://hub.fastgit.org/Mipu94/peda-heap.git
echo "source /opt/tools/peda-heap/peda.py" >> ~/.gdbinit
```
![image-20201223005859989](upload/attach/202012/830671_JT848HEQYUVW8HV.jpg)

> 图中还有dbg的影子，如果不想要的话，那么可以注释掉`~/.gdbinit`（这里也就是`/root/.gdbinit`）里关于`pwndbg`的那一行。

#### 2.3.3 安装gef
> 我在`docker`里安装`gef`失败，暂时没找到原因，但是我在虚拟机里是安装成功的。
> PS：`gef`我用得少，就没管了。如果有哪位大佬知道为啥，请告诉我。

1) 安装wget
```shell
apt install wget
```

2) 安装gef
```shell
#下载 `gef.sh` 并执行
wget -q -O- https://github.com/hugsy/gef/raw/master/gef.sh | sh
# 下载 `gef.py`, 并将其 `source` 写入 `.gdbinit`
wget -q -O ~/.gdbinit-gef.py https://github.com/hugsy/gef/raw/master/gef.py
echo source ~/.gdbinit-gef.py >> ~/.gdbinit
```

#### 2.3.4 安装pwngdb
> 我安装 `pwngdb` 主要也是为了调试堆。
```shell
git clone https://github.com/scwuaptx/Pwngdb.git
cp Pwngdb/.gdbinit ~/
```
安装完成后，修改~/.gdbinit里面的内容
![image-20201223012816413](upload/attach/202012/830671_QKSFTBNU77VTT7Q.jpg)
![image-20201223012749737](upload/attach/202012/830671_AGVJUXZR9DQPVU6.jpg)

#### 2.3.5 命令切换
```shell
pwndbg：
echo "source /opt/tools/pwndbg/gdbinit.py" > ~/.gdbinit
peda-heap：
echo "source /opt/tools/peda-heap/peda.py" > ~/.gdbinit
gef：
echo "source ~/.gdbinit-gef.py" > ~/.gdbinit
```

### 2.4 安装LibcSearcher
```shell
# git clone https://github.com/lieanu/LibcSearcher.git 官方
git clone https://gitclone.com/github.com/lieanu/LibcSearcher.git # 加速
cd LibcSearcher
python setup.py develop
```
![image-20201223013821499](upload/attach/202012/830671_FFGUHZCZ9C28KR4.jpg)

### 2.5 安装pwntools

- 官方安装
```shell
apt-get update
apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade pwntools
```

- 我的安装（依赖我在dockerfile安装了，pwntools习惯用python2）
```shell
python -m pip install --upgrade pip
# 豆瓣源
python -m pip install --upgrade pwntools -i https://pypi.douban.com/simple/
```

> `pwntools`：<mark class="encrypted">7eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2j5h3I4D9L8%4m8K6L8r3g2V1i4K6u0r3M7s2N6F1N6r3!0G2L8s2x3`.</mark>

- 测试是否安装成功
![image-20201223015427942](upload/attach/202012/830671_77ZCRP6FJS895SA.jpg)

但是`shellcraft`使用报错：语法错误。初步猜测问题出在`Pygments`这个库，可能版本过高或者过低？
![image-20201223015557536](upload/attach/202012/830671_X5EXZPK923ET2WC.jpg)

`pip list` 查看到`Pygments`的版本是2.7.3，然后故意写错安装命令查看`Pygments`有哪些版本，发现提供的最高是2.5.2，安装2.5.2版本后发现使用`shellcraft`成功。
![](upload/attach/202012/830671_746R95VGM8BGXCA.jpg)
![](upload/attach/202012/830671_QGK3CJRZMMWXMV2.jpg)

### 2.6 安装ROPgadget
```shell
# 官方
git clone https://github.com/JonathanSalwan/ROPgadget.git
# 高速
git clone https://gitclone.com/github.com/JonathanSalwan/ROPgadget.git
cd ROPgadget
python setup.py install
```
![image-20201223020716641](upload/attach/202012/830671_95WUD9UTE9758F9.jpg)

### 2.7 安装one_gadget
```shell
# 移除https://rubygems.org源
gem sources --remove https://rubygems.org/
# 增加http://gems.ruby-china.org/源
gem sources -a http://gems.ruby-china.com/
# 安装
gem install one_gadget
```
![image-20201223021035885](upload/attach/202012/830671_E9WJWSN7K7A4325.jpg)

### 2.8 安装glibc-all-in-one
```shell
# 官方
# git clone https://github.com/matrix1001/glibc-all-in-one.git
# 加速
git clone https://gitclone.com/github.com/matrix1001/glibc-all-in-one.git
cd glibc-all-in-one
./update_list #更新最新版本的glibc
cat list #查看可下载的glibc
./download glibc #glibc为你想要下载glibc的名字
ps：查看当前glibc版本号方法
/lib/x86_64-linux-gnu/libc.so.6
```
![image-20201223022100675](upload/attach/202012/830671_PSRZ47K5MB8QZ9P.jpg)

### 2.9 安装patchelf
```shell
# 官方
# git clone https://github.com/NixOS/patchelf.git
# 加速
git clone https://gitclone.com/github.com/NixOS/patchelf.git
cd patchelf
./bootstrap.sh # 这里报错了，解决方法看下面
./configure
make
make install
make check
```

- 报错：
![image-20201223022513545](upload/attach/202012/830671_MG4WEJDYZWR6ESB.jpg)

- 解决：
```shell
apt-get install autoconf automake libtool
apt-get install libffi-dev
```

- 安装完成
![image-20201223022815773](upload/attach/202012/830671_57REY6BN9F655KZ.jpg)

### 2.10 给程序换glibc

#### 2.10.1 生成所需的符号链接
```shell
cd /lib64 #进入64位的目录 glibc 32位就 cd /lib
# 在root态下链接
ln -s /opt/tools/glibc-all-in-one/libs/2.27-3ubuntu1.4_amd64/ld-2.27.so ./27_14-linux.so.2
#27代表glibc版本,14代表ubuntu后面的数字(单纯为了好记)
```
![image-20201223023216246](upload/attach/202012/830671_H7HN8HRCJGREM8H.jpg)

#### 2.10.2 更改elf文件的ld和libc
