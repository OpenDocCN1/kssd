
# 使用`docker`调试和部署`pwn`题

[TOC]

主要内容如下：

> 1. 提供了使用`docker`构建的不同`glibc`版本的轻量级调试环境，涵盖`glibc-2.27/2.31/2.35/2.38`等主流版本

> 2. 提供了`11`种基于`docker`环境的`pwn`题部署模板

> 3. 给出了相关的`Dockerfile`用于定制化环境

# 0x0 前言

关于`docker`的基础概念不做过多的介绍。可以到[Docker : Accelerated, Containerized Application Development](elink@88cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2V1L8$3y4C8k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.)官方网站上获取更多的信息。熟悉并使用`docker`的常用命令并不需要太多的时间。

在使用`docker`指令或者`docker-compose`的指令时，尽量到官方手册上查询有关指令的详情内容。

一般来说，使用`docker-compose`来运行多个容器，也可以集成编译镜像、运行实例等流程。

# 0x1 调试环境

如果需要一个完整的、经常使用的主力调试环境，我推荐使用[skysider/pwndocker: A docker environment for pwn in ctf (github.com)](elink@581K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9%4W2K6K9h3c8W2M7W2)9J5c8Y4m8%4L8X3c8G2j5$3E0W2M7R3`.`.)，工具非常完整，可以直接拉取镜像。

本文所分享的，是临时使用的、快速的、轻量级调试环境。镜像只包含必要的程序，镜像包相对较小，下载更快。

对于不同的版本，特别是传统的`glibc`的环境，有时候使用`patchelf`改变二进制文件的`so`无法得到与远程环境一样的内存布局，并且会缺乏调试符号。而对于部署在`debian/fedora`等其他操作系统上的程序，受到的影响会更大。因此，使用`docker`搭建一个一模一样的调试环境很有必要。

目前，我针对常用的环境制作了轻量级镜像，并发布在[roderickchan/debug\_pwn\_env Tags | Docker Hub](elink@ddeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5N6h3u0Q4x3X3g2V1L8$3y4C8k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5i4K6u0r3M7X3!0V1k6i4u0A6j5$3E0U0K9r3q4F1i4K6u0r3k6r3g2T1N6h3N6Q4y4h3k6H3N6$3&6Q4y4h3k6W2L8Y4k6Q4x3V1k6@1j5h3N6K6)。

![image-20230213140845522](upload/attach/202312/956675\_5GNVRW93KFXFUJE.jpg)

每个镜像都给出了 `Ubuntu` 的版本、其使用的 `glibc libc.so.6` 的具体版本（包括小版本）和镜像编译时间。

直接使用`docker pull roderickchan/debug\_pwn\_env:22.04-2.35-0ubuntu3.1-20230213`这样的命令拉取镜像即可。

拉取镜像后，运行命令如下：

```shell
docker run -it --rm -v host\_path:container\_path -p host\_port:container\_port --cap-add=SYS\_PTRACE IMAGE\_ID # auto update
docker run -it -rm -v host\_path:container\_path -p host\_port:container\_port --cap-add=SYS\_PTRACE IMAGE\_ID /bin/bash # do not update
docker run -it --rm -v host\_path:container\_path -p host\_port:container\_port --privileged IMAGE\_ID # privileged enabled and auto update
```

可以映射宿主机端口和容器端口，映射文件或者目录。如果不带命令，容器会自动更新一些`pwn`相关的仓库和包，如果带上命令，就会执行指定的命令。

事实上这个镜像的`Dockerfile`如下：

```dockerfile
ARG BUILD\_VERSION
FROM ubuntu:$BUILD\_VERSION
ARG DEBIAN\_FRONTEND=noninteractive
ARG HUB\_DOMAIN=github.com
ARG NORMAL\_USER\_NAME=ctf
ENV TZ=Etc/UTC
ENV LANG en\_US.UTF-8
ENV LANGUAGE en\_US:en
ENV LC\_ALL en\_US.UTF-8
WORKDIR /root
RUN apt-get update && apt-get -y dist-upgrade && apt-get install -y --fix-missing python3 python3-pip python3-dev lib32z1 \
xinetd curl gcc gdb gdbserver g++ git libssl-dev libffi-dev build-essential tmux \
vim iputils-ping gdb-multiarch \
file net-tools socat ruby ruby-dev locales autoconf automake libtool make && \
gem install one\_gadget && \
gem install seccomp-tools && \
sed -i '/en\_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen
# 先执行容易失败的操作
RUN git clone https://${HUB\_DOMAIN}/pwndbg/pwndbg && \
cd ./pwndbg && \
./setup.sh
RUN git clone https://${HUB\_DOMAIN}/NixOS/patchelf.git && \
cd ./patchelf && \
./bootstrap.sh && \
./configure && \
make && \
make install
RUN git clone https://${HUB\_DOMAIN}/hugsy/gef.git && \
git clone https://${HUB\_DOMAIN}/RoderickChan/Pwngdb.git && \
git clone https://${HUB\_DOMAIN}/Gallopsled/pwntools && \
(mv /usr/lib/python3.11/EXTERNALLY-MANAGED /usr/lib/python3.11/EXTERNALLY-MANAGED.old || true) && \
pip3 install --upgrade --editable ./pwntools && \
git clone https://${HUB\_DOMAIN}/RoderickChan/pwncli.git && \
pip3 install --upgrade --editable ./pwncli
COPY ./gdb-gef /bin
COPY ./gdb-pwndbg /bin
COPY ./update.sh /bin
COPY ./test-this-container.sh /bin
COPY ./.tmux.conf ./
COPY ./.gdbinit ./
COPY ./flag /
COPY ./flag /flag.txt
RUN chmod +x /bin/gdb-gef /bin/gdb-pwndbg /bin/update.sh /bin/test-this-container.sh && \
echo "root:root" | chpasswd && \
python3 -m pip install --upgrade pip && \
pip3 install ropper capstone z3-solver qiling lief
# normal user
RUN useradd ${NORMAL\_USER\_NAME} -d /home/${NORMAL\_USER\_NAME} -m -s /bin/bash -u 1001 && \
echo "${NORMAL\_USER\_NAME}:${NORMAL\_USER\_NAME}" | chpasswd && \
cp -r /root/pwndbg /home/${NORMAL\_USER\_NAME} && \
cp -r /root/gef /home/${NORMAL\_USER\_NAME} && \
cp -r /root/pwntools /home/${NORMAL\_USER\_NAME} && \
cp -r /root/Pwngdb /home/${NORMAL\_USER\_NAME} && \
cp -r /root/pwncli /home/${NORMAL\_USER\_NAME} && \
cp /root/.tmux.conf /home/${NORMAL\_USER\_NAME} && \
cp /root/.gdbinit /home/${NORMAL\_USER\_NAME} && \
cp /flag /home/${NORMAL\_USER\_NAME} && \
cp /flag.txt /home/${NORMAL\_USER\_NAME} && \
chown -R ${NORMAL\_USER\_NAME}:${NORMAL\_USER\_NAME} /home/${NORMAL\_USER\_NAME}
USER ${NORMAL\_USER\_NAME}:${NORMAL\_USER\_NAME}
WORKDIR /home/${NORMAL\_USER\_NAME}
RUN pip3 install --upgrade --editable ./pwntools && \
pip3 install --upgrade --editable ./pwncli
# switch to root and install zsh
USER root:root
RUN apt-get install -y sudo zsh && \
echo "${NORMAL\_USER\_NAME} ALL=(ALL) NOPASSWD : ALL" | tee /etc/sudoers.d/ctfsudo
# switch 2 normal user
USER ${NORMAL\_USER\_NAME}:${NORMAL\_USER\_NAME}
WORKDIR /home/${NORMAL\_USER\_NAME}
# install zsh
RUN curl -fsSL -O https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh && \
chmod +x ./install.sh && \
sed -i -e 's/read[[:space:]]\*-r[[:space:]]\*opt/opt=n/g' ./install.sh && \
./install.sh && \
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
COPY ./.zshrc ./
# expose some ports
EXPOSE 20 21 22 80 443 23946 10001 10002 10003 10004 10005
CMD ["/bin/update.sh"]
```

内置了自使用的`tmux`配置文件和`gdb-gef/gdb-pwndbg`启动脚本以及最新的`pwncli`库。完整文件位于[GitHub - RoderickChan/docker\_pwn\_env: Debug pwn using docker image](elink@3fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9L8$3c8W2M7X3W2U0K9@1y4Z5j5h3&6Q4x3V1k6V1L8$3y4C8k6i4u0Q4y4h3k6H3N6$3&6Q4y4h3k6W2L8Y4j5`.)，可根据需求更改和自定义配置文件。

![image-20231128154958586](upload/attach/202312/956675\_434NHRT48NZS5EA.jpg)

如果是`debian`系统，只需要安装一些基础的包即可：

```shell
#!/bin/bash
set -ex
apt update && apt install -y tmux gdb gdbserver wget rpm file binutils socat python3 python3-pip procps
# 修改tmux配置
cat > ~/.tmux.conf << "EOF"
set -g prefix C-a #
unbind C-b # C-b即Ctrl+b键，unbind意味着解除绑定
bind C-a send-prefix # 绑定Ctrl+a为新的指令前缀
# 从tmux v1.6版起，支持设置第二个指令前缀
set-option -g prefix2 ` # 设置一个不常用的`键作为指令前缀，按键更快些
#set-option -g mouse on # 开启鼠标支持
# 修改分屏快捷键
unbind '"'
bind - splitw -v -c '#{pane\_current\_path}' # 垂直方向新增面板，默认进入当前目录
unbind %
bind | splitw -h -c '#{pane\_current\_path}' # 水平方向新增面板，默认进入当前目录
# 设置面板大小调整快捷键
bind j resize-pane -D 10
bind k resize-pane -U 10
bind h resize-pane -L 10
bind l resize-pane -R 10
EOF
# 安装pwntools和pwncli
pip3 install pwntools pwncli
bash -c "$(wget https://gef.blah.cat/sh -O -)"
```

这里选用`gef`插件，因为`gef`插件的依赖少，安装速度快，功能比`peda`强大。

如果是`fedora`系统，安装软件包列表如下：

```shell
#!/bin/bash
dnf install -y tmux gdb gdb-gdbserver wget which file binutils socat python3 python3-pip procps
cat > ~/.tmux.conf << "EOF"
set -g prefix C-a #
unbind C-b # C-b即Ctrl+b键，unbind意味着解除绑定
bind C-a send-prefix # 绑定Ctrl+a为新的指令前缀
# 从tmux v1.6版起，支持设置第二个指令前缀
set-option -g prefix2 ` # 设置一个不常用的`键作为指令前缀，按键更快些
#set-option -g mouse on # 开启鼠标支持
# 修改分屏快捷键
unbind '"'
bind - splitw -v -c '#{pane\_current\_path}' # 垂直方向新增面板，默认进入当前目录
unbind %
bind | splitw -h -c '#{pane\_current\_path}' # 水平方向新增面板，默认进入当前目录
# 设置面板大小调整快捷键
bind j resize-pane -D 10
bind k resize-pane -U 10
bind h resize-pane -L 10
bind l resize-pane -R 10
EOF
pip3 install pwntools pwncli
bash -c "$(wget https://gef.blah.cat/sh -O -)"
```

和上面基本是一样的，不过有些包的名字不一样。

使用示例如下，使用最新的版本，普通用户的用户名为`ctf`（我的主力机器为`Ubuntu 20.04`，但是该题目运行环境需要`Ubuntu 22.04`）：

![623588](upload/attach/202402/956675\_BZC6V7M62V9FSBX.jpg)

这是另一个示例，使用之前的版本，普通用户名为`roderick`：

![623673](upload/attach/202402/956675\_HHV7RCV75RA6MM6.jpg)

# 0x2 出题模板
