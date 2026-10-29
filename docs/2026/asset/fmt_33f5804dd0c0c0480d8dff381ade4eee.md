
PROXY protocol会让TLS重定向器保持dumb，而不会丢失客户端连接的信息。只要重定向器和目标服务器支持PROXY，那么就可以保留原始客户端地址。重定向器接收客户端的TLS连接，将header信息添加到TCP数据包之前，然后将其传输到服务器。服务器（存放TLS密钥的服务器）也必须支持PROXY protocol，否则将会报错。

## 流程图

![流程图](https://bbs.kanxue.com/upload/attach/202004/747358_6U5V3P75WNCPYCG.jpg)

## 复现步骤

我应该已经做了充分的解释，对吧？你可能只想看配置的部分，那我们直接跳到那部分，我将根据上图做标记。最近一次测试完成于2020年2月15日，测试环境为`nginx version: nginx/1.14.0 (Ubuntu)`，操作系统是`Ubuntu 18.04.4 LTS`。

### example.com

将DNS记录指向Untrusted Redirector 1的IP地址。在此例中，即`192.168.0.4`。

```
@ 300 IN A 192.168.0.4
```

### Untrusted Redirector 1

这是的入口重定向器。恶意流量触达到该重定向器时会暴露IOC。我们不想它存放任何关键信息，并希望它在需要时尽可能容易地调整。它所做的只是监听`443`端口并指向Untrusted Redirector 2。

创建文件`/etc/nginx/stream_redirect.conf`

```
stream {
    server {
        listen 443;
        proxy_pass 192.168.0.3:443;
        proxy_protocol on;
    }
}
```

在`/etc/nginx/nginx.conf`加入下面这一行，并确保它没有被包含在其他代码块中。

```
include /etc/nginx/stream_redirect.conf
```

重启nginx，到下一步的Untrusted Redirector 2。`sudo systemctl restart nginx`。

### Untrusted Redirector 2

这是我们的中点重定向器，中点重定向器不是必需的，也可将Untrusted Redirector 1直接指向Trusted Ingress。但是，如果你想将`example.com`与实际的基础设施分离，那最好还是保留中点重定向器。如果你的基础设施托管在目标环境中，也会非常有用，这对于内部蓝军来说并不罕见，“所有秘密都藏在屋子里”。

创建文件`/etc/nginx/stream_redirect.conf`

```
stream {
    server {
        listen 443;
        proxy_pass 192.168.0.2:443;
    }
}
```

然后像配置之前的重定向器一样，可以添加`include /etc/nginx/stream_redirect.conf`到`/etc/nginx/nginx.conf`。或者，可以使用`socat`或`simpleproxy`之类的dumb重定向器来代替在此层使用的nginx。该层并不需要知道代理的存在，实际上，如果你尝试在任何中间点上使用`proxy_protocol on`，可能会导致其失效。

重启nginx，到下一步的Trusted Ingress。`sudo systemctl restart nginx`。

### Trusted Ingress

这是我们的入口系统，TLS终止于此，并且我们处于受信的环境中。`example.com`的关键信息位于该系统中，并且PROXY也将在此终止。

示例文件`/etc/nginx/sites-enabled/example.com.conf`：

```
server {
    server_name example.com;

    location / {
        proxy_pass 192.168.0.1:8080;
        proxy_set_header X-Forwarded-For $proxy_protocol_addr;
    }

    listen [::]:443 ssl ipv6only=on proxy_protocol; # managed by Certbot
    listen 443 ssl proxy_protocol; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

注意在`X-Forwarded-For`中的`$proxy_protocol_addr`变量，这是PROXY protocol中的源IP地址，

也将其记录在Trusted Ingress nginx日志中。

`/etc/nginx/nginx.conf`中的示例代码片段：

```
[...]
http {
[...]
    ##
    # Logging Settings
    ##
    log_format proxy '$proxy_protocol_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent "$http_referer" ' '"$http_user_agent"';
    access_log /var/log/nginx/access.log proxy;
    error_log /var/log/nginx/error.log;
[...]
}
```

你可能想仅对Trusted Ingress上的特定站点配置使用代理，需要在`/etc/nginx/nginx.conf`中保留`access_log`，并添加`access_log /var/log/nginx/example.com.access.log proxy;`至`/etc/nginx/sites-enabled/example.com.conf`。

### Downstream Application

如果完全按照此教程操作，则应用程序需要解析`X-Forwarded-For` header。你可以在nginx层进行[客户端IP过滤](https://bbs.kanxue.com/elink@affK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3P5r3c8S2k6r3g2Q4x3V1k6K6k6i4m8Z5K9i4u0G2N6r3R3`.)，将不需要的IP重定向到其他位置，这样的话应用程序甚至不必处理IP。

还有一种方法可以解决此问题，以便应用程序不需要解析`X-Forwarded-For` header，但是需要另外安装一个nginx模块（`ngx_http_realip_module`），但它不能作为单独的可安装模块使用，我不想在此文章中涉及有关从源代码构建nginx的整个部分。也许这是另一个帖子的主题。

## 结论

我不知道蓝军成员是否可以通过其他方式解决此问题，但此文所述的方法是一个很好的解决办法，因为它不需要任何中间服务器保存关键信息或获得授权。从技术上讲，服务器之间不太可能会传输错误的客户端IP地址，但仅在中间使用dumb重定向器，不会引入任何其他问题，因为你只会获得重定向器的IP，并且无法基于此直接做决策。因此，无论如何，还是用PROXY protocol吧。

照例。。。

![Hack the planet](https://bbs.kanxue.com/upload/attach/202004/747358_MAXEDDAUH3UBNJK.jpg)

原文链接：513K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5r3c8S2i4K6u0W2k6r3g2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3M7X3g2V1i4K6u0V1N6r3g2S2L8g2)9J5k6s2m8J5L8%4S2&6i4K6u0V1M7s2u0G2N6r3!0U0L8$3I4Q4x3X3c8F1k6$3W2F1P5q4)9J5c8R3`.

翻译：看雪翻译小组 SpearMint

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-4-7 20:01

被SpearMint编辑，原因：
