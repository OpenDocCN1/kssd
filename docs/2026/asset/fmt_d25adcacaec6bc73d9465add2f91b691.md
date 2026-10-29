
### 使用curl发送soap请求

```
$ proxychains curl --header "Content-Type: text/xml;charset=UTF-8" --header "SOAPAction:" --data @data.xml http://127.0.0.1:8888/
$ cat data.xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="e87K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2^5L8h3I4K6L8$3q4H3i4K6u0W2L8%4u0Y4i4K6u0r3M7$3!0S2M7q4)9J5c8X3g2F1N6X3g2D9L8%4m8W2i4K6u0r3" xmlns:web="db0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5">
<soapenv:Header/>
<soapenv:Body>
<web:soapRequest>
</web:soapRequest>
</soapenv:Body>
</soapenv:Envelope>
```

### 使用nping发送payload，伪造源IP

```
sudo nping -c 1 --data hexstring --udp -p dest_port -S source_ip -g source_port dest_ip
```

### bash分组，类似SQL group by

```
cat test.txt | sort | uniq -c | sort -n
```

### 通过SSH使用Wireshark

```
ssh root@192.168.1.1 "sudo tcpdump -U -s0 -i lo -w - 'not port 22'" | wireshark -k -i -
wireshark -k -i <(ssh root@192.168.1.1 tcpdump -U -s0 -i any -w - not port 22)
```

### HEX转PCAP

```
xxd -r -p test.hex | od -Ax -tx1 | text2pcap - test.pcap
```

### 解析JSON

```
grep -Po '"field" : .*?[^\\]",' test.json
```

### tshark将过滤结果保存到文件

```
tshark -r input.pcap -Y "ip.src == 10.1.1.1" -w output.pcap -F pcap
```

### john the ripper 显卡计算, OpenCL格式, 启动会话

```
john --session=session_name --format=opencl ~/hash.txt
```

- #### john the ripper 列举显卡OpenCL格式

  ```
  john --list=formats --format=opencl
  ```

- #### john the ripper 继续会话

  ```
  john --restore=session_name
  ```

- #### john the ripper 显示破解的密码

  ```
  john ~/hash.txt --show
  ```

- #### dynamic格式

- #### edit john/JohnTheRipper/run/dynamic.conf

  ```
  john --fork=16 --session=session_dynamic --format=dynamic_xxxx hash.txt
  ```

### --------------------------本地命令--------------------------

### 有用的Linux系统本地命令

### 快速分析系统，可能有助于提升权限

### 在连接到远程系统之前，启用日志记录会话交互内容。仅在类似ssh连接之后执行。

```
script <filename>
```

### 登录

```
ssh username@hostname
```

### 检查当前shell

```
echo $0
```

### 检查当前用户

```
whoami
```

### 检查系统

```
uname -a
```

### 检查系统启动时间

```
uptime
```

### 检查系统变量

```
export
```

### 检查进程

```
ps -ef
ps auxf
ps auxfww
```

### 搜索文件

```
find . -name "*.java" -type f -exec fgrep -iHn "textToFind" {} \;
find . -regex ".*\.\(c\|java\)" -type f -exec fgrep -iHn "textToFind" {} \;
find / -maxdepth 4 -name *.conf -type f -exec grep -Hn "textToFind" {} \; 2>/dev/null
```

- #### root用户拥有的SUID文件

  ```
  find / -uid 0 -perm -4000 -type f 2>/dev/null
  ```

- #### 全局可读的root用户拥有的SUID文件

  ```
  find / -uid 0 -perm -u=s,o=r -type f -exec ls -la {} \; 2>/dev/null
  ```

- #### SUID文件

  ```
  find / -perm -4000 -type f 2>/dev/null
  ```

- #### 全局可写目录

  ```
  find / -perm -2 -type d 2>/dev/null
  ```

- #### 搜索文件中的密码，忽略错误，滤出proc和其他文件夹

  ```
  find . ! -path "*/proc/*" -type f -name "*" -exec fgrep -iHn password {} \;
  find . -type f \( -iname \*.conf -o -iname \*.cfg -o -iname \*.xml -o -iname \*.ini -o -iname \*.json -o -iname \*.sh -o -iname \*.pl -o -iname \*.py \) -exec fgrep -iHn password {} \; 2>/dev/null
  ```

- #### 使用一些正则特征搜索（正则特征按行分）

  ```
  find . -type f -exec grep -iHFf patterns.txt {} \;
  ```

- #### 在小文件中搜索密码

  ```
  find . -type f -size -512k -exec fgrep -iHn password {} \;
  ```

- #### 逆向java jar文件，在其中搜索密码

  ```
  find . -name "*.jar" -type f -exec ~/jd-cli/jd-cli -oc -l -n -st {} \; | egrep -i -e "Location:" -e "password" | uniq
  ```

### 检查开放端口和监听端口的服务

```
netstat -anp
```

### 检查自定义hosts

```
cat /etc/hosts
```

### 检查本地IP地址和网口

```
ifconfig -a
```

### 检查路由

```
route -v
```

### 检查文件系统

```
$ df
```

### 检查sudo特权

```
sudo -l
```

### 检查定时任务

```
crontab -l
```

### 检查启动任务

```
cat /etc/inittab
```

### 尝试嗅探流量

```
tcpdump
tcpdump -s0 not port 22 -w trace.pcap
```

### 检查已知服务器公钥

```
cat ~/.ssh/known_hosts
```

### 尝试访问邮件

```
head /var/mail/root
```

### 列举用户组和用户

```
cat /etc/group
cat /etc/passwd
```

- #### root权限下

  ```
  cat /etc/shadow
  ```

### 检查共享内存

```
ipcs -mp
```

### 登出

```
logout
```

### 关闭脚本会话

`Ctrl + D`

### --------------------------SSH隧道和链路--------------------------

### 有助于在不同机器间跳转和隧道

### SSH代理命令

```
ssh config (~/.ssh/config)
Host _first_server
Hostname XXX.XXX.XXX.XX
Port 22
User root

Host _second_server
Hostname 127.0.0.1
Port 22
User root # password is  "XXXXXXX"
ProxyCommand ssh -v -W 127.0.0.1:22 _first_server

Host _third_server
Hostname XXX.XXX.XXX.XXX
Port 22
User XXXXX
ProxyCommand ssh _second_server -W %h:%p

Host _host_over_sshpass and ssh key
Hostname XXX.XXX.XXX.XXX
IdentityFile id_rsa
User XXXXX
ProxyCommand sshpass -pXXXXXXX ssh -Fssh_config _host_previous_in_chain -W %h:%p
```

- #### 连接

  ```
  ssh root@_second_server
  ```

### 代理链

- #### 在后台执行ssh，不执行命令

  ```
  ssh -f -N -D 9050 user@host
  proxychains telnet hosts
  ```

- #### 使用本地SSH隧道在远端服务器上访问web服务器

  ```
  sudo ssh -F ~/.ssh/ssh_config _host_definition -L 127.0.0.1:8080:127.0.0.1:8080 -L 127.0.0.1:8443:127.0.0.1:8443
  ```

- #### torsocks

  ```
  torsocks sshpass -p '********' ssh -C admin@XXX.XXX.XXX.XXX "sudo tcpdump -i any -U -s0 -w - 'not port 22'" | wireshark -k -i -
  ```

- #### 允许通过互联网及远端服务器访问本地服务器（反向SSH隧道，内网穿透）

  ```
  sudo apt install tinyproxy
  sudo systemctl disable tinyproxy
  sudo systemctl stop tinyproxy
  sudo vi /etc/tinyproxy/tinyproxy.conf
  ```

- #### 配置监听端口(例如端口3128)
