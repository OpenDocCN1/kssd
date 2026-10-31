
### 1. 修改服务配置

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
events {
worker_connections 768;
}
http {
sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
types_hash_max_size 2048;
include /etc/nginx/mime.types;
default_type application/octet-stream;
- ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
- ssl_prefer_server_ciphers on;
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
gzip on;
server {
- listen 443;
+ listen 80;
server_name libc.rocks;
- ssl on;
- ssl_certificate /etc/letsencrypt/live/libc.rip/fullchain.pem;
- ssl_certificate_key /etc/letsencrypt/live/libc.rip/privkey.pem;
location /api/ {
proxy_pass http://127.0.0.1:8000;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection keep-alive;
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}
location /download/ {
- alias /home/niklas/libc-database/db/;
+ alias /home/error404/libc-database/db/;
}
location / {
- alias /home/niklas/libc-database/searchengine/frontend/build/;
+ alias /home/error404/libc-database/searchengine/frontend/build/;
}
}
}
```

#### 修改 libc-database/searchengine/config.py

```python
import os
ES_INDEX_NAME = 'libcsearch'
# ES_HOST = 'localhost'
ES_HOST = 'es01'
# DB_DIR = os.path.dirname(os.path.abspath(__file__)) + '/../db'
DB_DIR = '/db'
DEFAULT_SYMBOLS = [
'__libc_start_main_ret',
'system',
'dup2',
'str_bin_sh',
'read',
'write',
'puts',
'printf',
]
- DOWNLOAD_URL = 'https://libc.rip/download/{}.so'
+ DOWNLOAD_URL = 'https://127.0.0.1/download/{}.so'
```

#### 修改 libc-database/searchengine/api.py

```yaml
openapi: 3.0.3
info:
version: 1.0.0
title: libc-database search engine
description: ''
paths:
/find:
post:
operationId: app.find
tags:
- libcsearch
description: |-
Look up libc by various attributes
requestBody:
content:
application/json:
schema:
$ref: '#/components/schemas/Libc'
required: true
responses:
'200':
content:
application/json:
schema:
type: array
items:
$ref: '#/components/schemas/Libc'
description: ''
/libc/{id}:
post:
operationId: app.dump
tags:
- libcsearch
parameters:
- in: path
name: id
schema:
type: string
required: true
description: |-
Dump libc symbols
requestBody:
content:
application/json:
schema:
$ref: '#/components/schemas/DumpRequest'
required: true
responses:
'200':
content:
application/json:
schema:
$ref: '#/components/schemas/Libc'
description: ''
components:
schemas:
Libc:
type: object
properties:
md5:
type: string
sha1:
type: string
sha256:
type: string
buildid:
type: string
id:
type: string
symbols:
type: object
additionalProperties:
type: string
pattern: '^(0x)?[a-fA-F0-9]+$'
download_url:
type: string
format: url
readOnly: true
DumpRequest:
type: object
properties:
symbols:
type: array
items:
type: string
servers:
- - url: https://libc.rip/api
+ - url: http://127.0.0.1/api
```

### 2. 启动 `elasticsearch` 查询服务以及查询后端 `uwsgi` 服务（守护服务）

使用命令`sudo docker-compose up -d`

启动后可以使用`docker ps`检查启动情况

![image-20200928103911098](upload/attach/202011/816358_Z3W5K4ZSP2C5TDW.jpg)

### 3. 建立 `elasticsearch` 查询服务索引

在`/libc-database/searchengine`下，执行`python3.6 -m index ../db`

建立完成后，使用`curl -XGET 127.0.0.1:9200/_cat/indices/`来测试索引建立情况

![image-20200928142936169](upload/attach/202011/816358_Q4PQW3CXJZD6VP5.jpg)

### 4. 启动本地 `Nginx` 服务

最后，在`/libc-database/searchengine`下使用`/usr/sbin/nginx -c nginx.conf`启动本地`Ngnix`服务

### 5. 测试服务状态

使用命令：

```bash
curl -X POST -H 'Content-Type: application/json' --data '{"symbols": {"strncpy": "db0", "strcat": "0x000000000d800"}}' 294K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8X3q4H3K9g2)9J5c8X3k6A6L8X3c8Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4b7U0g2Q4z5p5u0Q4c8e0S2Q4b7f1k6Q4z5e0g2Q4c8e0y4Q4z5o6m8Q4z5o6t1
```

![image-20200928143551542](upload/attach/202011/816358_3CRB3HRDTF2UPU8.jpg)

### 6. Enjoy It

断开虚拟机网络，以验证本地状态下的服务可用性

![image-20200928144129824](upload/attach/202011/816358_5U3JEA5W63TRDW5.jpg)

## 0x06 查询服务的更新（需要 ）

执行下列命令即可

```bash
cd libc-database/
./get ubuntu debian centos kali parrotsec
./get arch
./get rpm
python3.6 -m index ../db
```

## 0x07 参考链接

[【原】光盘安装ubuntu出现BusyBox不能继续安装的终极解决方法](<elink@9baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3y4I4K9h3&6@1k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0x3#2y4e0f1%4y4U0V1>)

## 0x01 前言

此处，我们以`Ubuntu 16.04`虚拟机为例讲述搭建方法，为防止环境冲突，请选择使用一台干净的虚拟机，若使用`Ubuntu Server 16.04`，可能会出现系统安装失败情况，若出现，请参考底部的参考链接。

另外，此教程中涉及到了部分文件的修改，我也对原项目进行了部分本地化，因此也可以直接获取我`fork`修改的项目进行搭建，项目地址：5cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6h3u0@1N6h3g2J5M7X3!0J5y4o6l9@1i4K6u0r3L8r3W2T1j5#2)9J5k6r3c8S2N6r3q4T1j5i4y4W2i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1$3i4@1q4o6i4@1p5J5i4@1f1^5i4@1u0r3i4K6S2q4f1%4c8S2M7W2)9%4c8b7`.`.

本文中最后生成的配置结束的PVM虚拟机已上传至百度云盘，需要的自取即可，链接: aa4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4L8h3k6s2i4K6g2X3c8g2x3@1c8%4A6n7M7V1I4&6L8@1N6d9k6i4u0K9c8r3!0m8 密码: 9vpp

## 0x02 安装环境所需的依赖（需要 ，需要较长⏱️）

### 1. 更新系统内的所有程序包（需要 ，需要较长⏱️）

使用命令`sudo apt-get update && sudo apt-get upgrade -y`

⚠️：可以使用以下命令换源以加速下载

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vi /etc/apt/sources.list
# vi 内运行以下两条命令
:%s/us.archive.ubuntu.com/mirrors.aliyun.com/g
:wq
```

### 2. 通过 `apt` 安装必要软件包（需要 ）

使用命令：

```bash
sudo apt-get install git vim wget curl python python3 python-pip python3-pip binutils file wget rpm2cpio cpio zstd nginx libffi6 libffi-dev software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6
sudo curl https://bootstrap.pypa.io/get-pip.py -o /get-pip.py
sudo python3.6 /get-pip.py
sudo ln -s /usr/local/bin/pip /usr/bin/pip
```

### 3. 通过 `pip` 安装必要软件包（需要 ）

使用命令：

```bash
sudo pip install elasticsearch
```

### 4. 安装 `docker` 以及 `docker-compose`（需要 ）

此处推荐使用以下命令直接获取`docker`

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

`docker`成功安装后可以去[阿里云官网](<https://bbs.kanxue.com/elink@2eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7W2)9J5k6h3y4G2L8Y4y4G2L8r3g2Q4x3X3g2S2L8r3W2&6N6h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8W2)9J5k6r3S2S2L8X3N6*7K9r3!0#2i4K6u0r3K9h3&6K6N6r3q4F1j5$3g2K6i4K6u0r3L8h3W2J5M7X3!0J5M7H3`.`.)>)获取镜像加速器，然后按照官网所述直接做如下配置即可。
