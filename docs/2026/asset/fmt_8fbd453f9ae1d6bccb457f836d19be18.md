
以下代码展示了文件上传功能的实现逻辑（基于DVWA平台）：

```php
<?php

if (isset($_POST['Upload'])) {

            $target_path = DVWA_WEB_PAGE_TO_ROOT."hackable/uploads/";

            $target_path = $target_path . basename($_FILES['uploaded']['name']);

            $uploaded_name = $_FILES['uploaded']['name'];

            $uploaded_ext = substr($uploaded_name, strrpos($uploaded_name, '.') + 1);

            $uploaded_size = $_FILES['uploaded']['size'];

            if (($uploaded_ext == "jpg" || $uploaded_ext == "JPG" || $uploaded_ext == "jpeg" || $uploaded_ext == "JPEG") && ($uploaded_size < 100000)){

                if(!move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path)) {

                    echo '<pre>';

                    echo 'Your image was not uploaded.';

                    echo '</pre>';

                  } else {

                    echo '<pre>';

                    echo $target_path . ' succesfully uploaded!';

                    echo '</pre>';

                    }

            }

            else{

                echo '<pre>';

                echo 'Your image was not uploaded.';

                echo '</pre>';

            }

        }

?>
```

## 实验过程

为了继续拿到权限，这里制作一句话图片木马进行实验，操作步骤都一样，只需将一句话木马隐藏写入图片中即可。

### 一句话图片木马制作

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_P9WHPPN89MZY5KC.png)

继续通过`BurpSuit`的拦截功能，查看上传一句话图片木马的属性。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_4EVZSGDDPJC6R54.png)

访问上传成功后的`URL`，显示图片；目前为止图片上传成功，但木马还没有生成。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_KVUQFVDKSNJ3ZPR.png)

所有静态资源，服务器都不会执行，需要渲染。

一句话图片木马需要高安全级别上传图片，还涉及文件包含漏洞。

### 文件上传目录

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_YEEN5YAHC2EEG34.png)

`/var/www/dvwa/hackable/uploads` // 上传文件目录 `dzq.jpg`

### 文件包含目录

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_RY7VNS353AEJQ4S.png)

`/var/www/dvwa/vulnerabilities/fi` // 文件包含访问目录 `shell20.php`（参考图片木马制作脚本）

执行文件，生成后门，将上传一句话木马图片路径复制。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_Z4J72U7BQTGKUP9.png)

将复制路径放在文件包含所在`URL`参数中（这个时候安全等级为`low`）。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/780980_KKB5MBARBUBQM6K.png)

当访问后，出现大量的乱码，即执行成功，会在文件包含目录下生成后门。

接下来，使用`AntSword`进行连接，需要注意这里的地址为：`26dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0t1H3x3q4)9J5k6e0p5#2x3W2)9J5c8X3c8$3N6$3q4Q4x3V1k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8A6k6i4y4Q4x3V1k6X3K9g2)9J5c8Y4y4Z5k6h3I4D9x3U0m8Q4x3X3g2H3K9s2l9`。

后续操作同上。

### 另一种文件包含远程漏洞利用方法

只需要在`web`网站中提前将一句话木马写入`txt`文件中，通过文件包含地址参数的访问达到生成的目的（自行测试）。

## 知识点

### `webshell`

- **小马**：一句话木马也称为小马，即整个`shell`代码量只有一行，一般是系统执行函数。
- **大马**：代码量和功能比小马多，一般会进行二次编码加密，防止被安全防火墙/入侵系统检测到。

### 一句话图片木马方法总结

1. 制作一句话图片木马。
2. 上传图片木马文件。
3. 执行文件包含并生成后门。
4. 通过`AntSword`连接`webshell`。

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)
