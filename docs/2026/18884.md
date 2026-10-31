
# [翻译]一个路由器SQL注入漏洞分析

本文根据devttyS0的教程Exploiting Embedded Systems – Part 2[f4cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6Y4c8@1P5i4x3H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3g2)9J5c8U0l9&6i4K6u0r3k6i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6r3g2E0j5X3g2V1k6r3g2V1i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3](https://bbs.kanxue.com/elink@a73K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6Y4c8@1P5i4x3H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3g2)9J5c8U0l9&6i4K6u0r3k6i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6r3g2E0j5X3g2V1k6r3g2V1i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3)和Exploiting Embedded Systems – Part 3[addK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6Y4c8@1P5i4x3H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3g2)9J5c8U0l9&6i4K6u0r3k6i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6r3g2E0j5X3g2V1k6r3g2V1i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7K6i4K6u0r3](https://bbs.kanxue.com/elink@952K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6Y4c8@1P5i4x3H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3g2)9J5c8U0l9&6i4K6u0r3k6i4S2H3L8r3!0A6N6r3W2F1k6#2)9J5k6r3g2E0j5X3g2V1k6r3g2V1i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7K6i4K6u0r3)复现Trendnet一个路由器的SQL注入漏洞分析过程。部门内容直接翻译引用自原文，可以看做是原文的翻译整理。

相关的环境搭建详细的路由器漏洞分析环境搭建教程<http://bbs.pediy.com/showthread.php?t=212369>。

## 静态分析

存在漏洞的固件下载地址[fe6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2N6s2u0W2L8X3c8F1k6i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6f1c8g2N6Q4x3X3b7$3y4e0c8f1f1W2)9J5c8X3k6A6M7X3#2%4j5i4u0W2i4K6u0r3](https://bbs.kanxue.com/elink@5d0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2N6s2u0W2L8X3c8F1k6i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6f1c8g2N6Q4x3X3b7$3y4e0c8f1f1W2)9J5c8X3k6A6M7X3#2%4j5i4u0W2i4K6u0r3),最新版固件下载地址[810K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8J5k6h3&6V1L8X3g2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5c8Y4y4#2M7s2m8G2M7Y4c8V1k6i4c8S2K9h3I4Q4x3X3g2S2M7%4m8Q4x3@1k6H3M7X3!0V1i4K6y4p5x3e0M7#2i4K6g2X3g2p5g2i4i4K6u0V1y4U0f1@1g2q4t1`.](https://bbs.kanxue.com/elink@b8aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8J5k6h3&6V1L8X3g2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5c8Y4y4#2M7s2m8G2M7Y4c8V1k6i4c8S2K9h3I4Q4x3X3g2S2M7%4m8Q4x3@1k6H3M7X3!0V1i4K6y4p5x3e0M7#2i4K6g2X3g2p5g2i4i4K6u0V1y4U0f1@1g2q4t1`.)
使用`binwalk`解包：

![](https://bbs.kanxue.com/upload/attach/201609/725158_810a5e10bad743f8.png)

查看登录接口的URL和参数，在解包的文件中找到`my_cgi.cgi`文件。

![](https://bbs.kanxue.com/upload/attach/201609/725158_c4f282f2030467d1.png)

在`my_cgi.cgi`中查找`user_name`和`user_pwd`两个参数。

![](https://bbs.kanxue.com/upload/attach/201609/725158_355769e82ca1f835.png)

看起来`select level from user where user_name='%s' and user_pwd='%s'`像是登录的时候查询SQL的参数，使用IDA载入`my_cgi.cgi`。查询”`select level from user where user_name`“字符串，可以定位到`do_login`函数。

![](https://bbs.kanxue.com/upload/attach/201609/725158_10eac560f9ba450d.png)

## 动态分析

分析`do_login`函数，在`exec_sql`函数执行之前下断点。

![](https://bbs.kanxue.com/upload/attach/201609/725158_64cac8c89533f8cd.png)

使用如下脚本运行`my_cgi.cgi`:
```
#!/bin/bash

INPUT="$1"
LEN=$(echo -n "$INPUT" | wc -c)
PORT="1234"
if [ "$LEN" == "0" ] || [ "$INPUT" == "-h" ] || [ "$UID" != "0" ]
then
    echo -e "\nUsage: sudo $0 <POST data>\n"
    exit 1
fi
cp $(which qemu-mipsel-static) ./qemu
echo "$INPUT" | chroot . qemu -E REQUEST_METHOD="POST" -E CONTENT_LENGTH=$LEN -E CONTENT_TYPE="multipart/x-form-data" -E REMOTE_ADDR="1.1.1.100" -g $PORT /usr/bin/my_cgi.cgi 2>/dev/null
```
需要放在解压出来的`squashfs-root`目录下执行。

`bash cgi.sh "request=login&user_name=admin&user_pwd='%20or%20'1'%3D'1"`

然后IDA连接到GDB端口进行调试，具体方法参考详细的路由器漏洞分析环境搭建教程。按F9执行程序到`exec_sql`的断点处。可以看到寄存器`$a1`的值为指向sql字符串的指针。

![](https://bbs.kanxue.com/upload/attach/201609/725158_ce98c1fa4c85e296.png)

在hex view中跳转到对应的地址，可以看到sql字符串的值为：

> `select level from user where user_name='admin' and user_pwd='' or '1'='1'`

表明我们输入的字符串已经拼接到SQL语句中。

![](https://bbs.kanxue.com/upload/attach/201609/725158_cd74dca3062e0898.png)

采用同样的方式动态调试最新版固件，可以发现漏洞已经修复。最新版对输入的参数进行了长度检查，并且检查是否含有单引号。可见这个点除了SQL，应该是也存在溢出的。

进行长度检查。

![](https://bbs.kanxue.com/upload/attach/201609/725158_b584f0e1a3b1441f.png)

检查是否存在单引号避免SQL注入。

![](https://bbs.kanxue.com/upload/attach/201609/725158_2388d8625b64b947.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
