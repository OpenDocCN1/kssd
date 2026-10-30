
```
ARG BUILD_VERSION



FROM ubuntu:$BUILD_VERSION



ARG DEBIAN_FRONTEND=noninteractive

ARG HUB_DOMAIN=github.com

ARG NORMAL_USER_NAME=ctf



ENV TZ=Etc/UTC

ENV LANG en_US.UTF-8

ENV LANGUAGE en_US:en

ENV LC_ALL en_US.UTF-8



WORKDIR /root



RUN apt-get update && apt-get -y dist-upgrade && apt-get install -y --fix-missing python3 python3-pip python3-dev lib32z1 \
    xinetd curl gcc gdb gdbserver g++ git libssl-dev libffi-dev build-essential tmux \
    vim iputils-ping gdb-multiarch \
    file net-tools socat ruby ruby-dev locales autoconf automake libtool make && \
    gem install one_gadget && \
    gem install seccomp-tools && \
    sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen



# 先执行容易失败的操作

RUN git clone https://${HUB_DOMAIN}/pwndbg/pwndbg && \
    cd ./pwndbg && \
    ./setup.sh



RUN git clone 0a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Q4x3U0c8Q4y4@1u0t1g2f1u0Q4y4h3k6p5e0@1#2m8d9f1&6Q4y4@1c8Q4x3V1k6z5K9i4S2a6f1#2)9J5c8Y4m8S2N6r3y4Z5k6h3I4X3i4K6u0W2k6$3W2@1 && \
    cd ./patchelf && \
    ./bootstrap.sh && \
    ./configure && \
    make && \
    make install



RUN git clone 4e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Q4x3U0c8Q4y4@1u0t1g2f1u0Q4y4h3k6p5e0@1#2m8d9f1&6Q4y4@1c8Q4x3V1k6Z5N6h3N6K6P5g2)9J5c8X3N6W2k6W2)9J5k6h3N6A6N6l9`.`. && \
    git clone f94K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Q4x3U0c8Q4y4@1u0t1g2f1u0Q4y4h3k6p5e0@1#2m8d9f1&6Q4y4@1c8Q4x3V1k6d9L8$3c8W2M7X3W2U0K9@1y4Z5j5h3&6Q4x3V1k6b7N6$3&6Y4k6r3u0Q4x3X3g2Y4K9i4b7`. && \
    git clone https://${HUB_DOMAIN}/Gallopsled/pwntools && \
    (mv /usr/lib/python3.11/EXTERNALLY-MANAGED /usr/lib/python3.11/EXTERNALLY-MANAGED.old || true) && \
    pip3 install --upgrade --editable ./pwntools && \
    git clone 655K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Q4x3U0c8Q4y4@1u0t1g2f1u0Q4y4h3k6p5e0@1#2m8d9f1&6Q4y4@1c8Q4x3V1k6d9L8$3c8W2M7X3W2U0K9@1y4Z5j5h3&6Q4x3V1k6H3N6$3&6U0L8r3W2Q4x3X3g2Y4K9i4b7`. && \
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
RUN useradd ${NORMAL_USER_NAME} -d /home/${NORMAL_USER_NAME} -m -s /bin/bash -u 1001 && \
    echo "${NORMAL_USER_NAME}:${NORMAL_USER_NAME}" | chpasswd && \
    cp -r /root/pwndbg /home/${NORMAL_USER_NAME} && \
    cp -r /root/gef /home/${NORMAL_USER_NAME} && \
    cp -r /root/pwntools /home/${NORMAL_USER_NAME} && \
    cp -r /root/Pwngdb /home/${NORMAL_USER_NAME} && \
    cp -r /root/pwncli /home/${NORMAL_USER_NAME} && \
    
    cp /root/.tmux.conf /home/${NORMAL_USER_NAME} && \
    cp /root/.gdbinit /home/${NORMAL_USER_NAME} && \
    cp /flag /home/${NORMAL_USER_NAME} && \
    cp /flag.txt /home/${NORMAL_USER_NAME} && \
    chown -R ${NORMAL_USER_NAME}:${NORMAL_USER_NAME} /home/${NORMAL_USER_NAME}

USER ${NORMAL_USER_NAME}:${NORMAL_USER_NAME}

WORKDIR /home/${NORMAL_USER_NAME}

RUN pip3 install --upgrade --editable ./pwntools && \
    pip3 install --upgrade --editable ./pwncli

# switch to root and install zsh
USER root:root
RUN apt-get install -y sudo zsh && \
    echo "${NORMAL_USER_NAME} ALL=(ALL) NOPASSWD : ALL" | tee /etc/sudoers.d/ctfsudo

# switch 2 normal user
USER ${NORMAL_USER_NAME}:${NORMAL_USER_NAME}
WORKDIR /home/${NORMAL_USER_NAME}

# install zsh
RUN curl -fsSL -O 17aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4N6Q4x3X3g2Y4K9i4c8Z5N6h3u0#2M7$3g2J5j5$3!0F1N6r3g2F1N6q4)9J5k6h3y4G2L8g2)9J5c8X3!0Z5L8i4W2*7M7$3S2Q4x3V1k6G2K9r3#2&6P5Y4y4Z5i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6A6L8Y4y4@1j5h3I4D9i4K6u0W2M7$3R3`. && \
    chmod +x  ./install.sh && \ 
    sed -i -e 's/read[[:space:]]*-r[[:space:]]*opt/opt=n/g' ./install.sh && \
    ./install.sh && \
    git clone 5c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7M7$3S2Q4x3X3c8#2M7$3g2J5M7#2)9J5c8Y4A6K6K9q4)9J5k6s2y4&6L8Y4c8S2P5q4)9J5k6r3S2A6k6$3S2D9K9h3N6Z5N6r3W2F1k6#2)9J5k6h3N6A6N6l9`.`. ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
    git clone 51dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7M7$3S2Q4x3X3c8#2M7$3g2J5M7#2)9J5c8Y4A6K6K9q4)9J5k6r3q4#2N6r3!0K6N6h3N6Y4k6i4y4@1K9h3!0F1M7H3`.`. ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

COPY ./.zshrc ./

# expose some ports
EXPOSE 20 21 22 80 443 23946 10001 10002 10003 10004 10005

CMD ["/bin/update.sh"]
```

内置了自使用的 `tmux` 配置文件和 `gdb-gef/gdb-pwndbg` 启动脚本以及最新的 `pwncli` 库。完整文件位于 [GitHub - RoderickChan/docker_pwn_env: Debug pwn using docker image](https://bbs.kanxue.com/elink@e74K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9L8$3c8W2M7X3W2U0K9@1y4Z5j5h3&6Q4x3V1k6V1L8$3y4C8k6i4u0Q4y4h3k6H3N6$3&6Q4y4h3k6W2L8Y4j5`.)，可根据需求更改和自定义配置文件。

![image-20231128154958586](https://bbs.kanxue.com/upload/attach/202312/956675_434NHRT48NZS5EA.jpg)

## Debian 系统
如果是 `debian` 系统，只需要安装一些基础的包即可：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31 | `#!/bin/bash`    `set` `-ex`    `apt update && apt` `install` `-y tmux gdb gdbserver wget rpm` `file` `binutils socat python3 python3-pip procps`    `# 修改tmux配置`  `cat` `> ~/.tmux.conf <<` `"EOF"`  `set` `-g prefix C-a` `#`  `unbind C-b` `# C-b即Ctrl+b键，unbind意味着解除绑定`  `bind C-a send-prefix` `# 绑定Ctrl+a为新的指令前缀`    `# 从tmux v1.6版起，支持设置第二个指令前缀`  `set``` -option -g prefix2 ` `` `` # 设置一个不常用的`键作为指令前缀，按键更快些 ``  `#set-option -g mouse on # 开启鼠标支持`  `# 修改分屏快捷键`  `unbind` `'"'`  `bind - splitw -``v` `-c` `'#{pane_current_path}'` `# 垂直方向新增面板，默认进入当前目录`  `unbind %`  `bind | splitw -h -c` `'#{pane_current_path}'` `# 水平方向新增面板，默认进入当前目录`    `# 设置面板大小调整快捷键`  `bind j resize-pane -D 10`  `bind k resize-pane -U 10`  `bind h resize-pane -L 10`  `bind l resize-pane -R 10`  `EOF`    `# 安装pwntools和pwncli`  `pip3` `install` `pwntools pwncli`  `bash` `-c` `` "$(wget 31bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4k6h3k6Q4x3X3g2T1L8r3q4Z5i4K6u0W2j5$3q4@1i4K6u0r3M7$3R3`. -O -)" `` |

这里选用 `gef` 插件，因为 `gef` 插件的依赖少，安装速度快，功能比 `peda` 强大。

## Fedora 系统
如果是 `fedora` 系统，安装软件包列表如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27 | `#!/bin/bash`    `dnf` `install` `-y tmux gdb gdb-gdbserver wget` `which` `file` `binutils socat python3 python3-pip procps`    `cat` `> ~/.tmux.conf <<` `"EOF"`  `set` `-g prefix C-a` `#`  `unbind C-b` `# C-b即Ctrl+b键，unbind意味着解除绑定`  `bind C-a send-prefix` `# 绑定Ctrl+a为新的指令前缀`    `# 从tmux v1.6版起，支持设置第二个指令前缀`  `set``` -option -g prefix2 ` `` `` # 设置一个不常用的`键作为指令前缀，按键更快些 ``  `#set-option -g mouse on # 开启鼠标支持`  `# 修改分屏快捷键`  `unbind` `'"'`  `bind - splitw -``v` `-c` `'#{pane_current_path}'` `# 垂直方向新增面板，默认进入当前目录`  `unbind %`  `bind | splitw -h -c` `'#{pane_current_path}'` `# 水平方向新增面板，默认进入当前目录`    `# 设置面板大小调整快捷键`  `bind j resize-pane -D 10`  `bind k resize-pane -U 10`  `bind h resize-pane -L 10`  `bind l resize-pane -R 10`  `EOF`    `pip3` `install` `pwntools pwncli`  `bash` `-c` `` "$(wget cebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4k6h3k6Q4x3X3g2T1L8r3q4Z5i4K6u0W2j5$3q4@1i4K6u0r3M7$3R3`. -O -)" `` |

和上面基本是一样的，不过有些包的名字不一样。

## 使用示例
使用示例如下，使用最新的版本，普通用户的用户名为 `ctf`（我的主力机器为 `Ubuntu 20.04`，但是该题目运行环境需要 `Ubuntu 22.04`）：

![623588](https://bbs.kanxue.com/upload/attach/202402/956675_BZC6V7M62V9FSBX.jpg)

这是另一个示例，使用之前的版本，普通用户名为 `roderick`：

![623673](https://bbs.kanxue.com/upload/attach/202402/956675_HHV7RCV75RA6MM6.jpg)

# 0x2 出题模板
