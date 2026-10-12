
# 剑侠 2 大补贴 服务器 PHP 代码

> 原文：[`www.pediy.com/kssd/pediy11/111133.html`](https://www.pediy.com/kssd/pediy11/111133.html)

剑侠 2 大补贴 服务器 php 代码

这是一个古老而长生的外挂，存在了很久也没有被官方封闭，破解版也很少。前段时间受朋友之托完成这个软件的服务器代码逆向，开始拿郁闷了很久 PHP 以前没有接触过，看了许久开发文档。

这个软件的主要算法是 `MD5,SHA1,BLOWFISH,BASE64,CRC32`。算法过程中使用了一些随机值的 `CRC32` 值，算法最后使用 `MD5` 算出密文的 `MD5` 值，放到密文最前端。密文分为三段第一段是记录天数或点数，第二段存放第三段的解密密钥的加密密文，第三才是解密后要真正得到的数据。

下面是软件加密过程

其中变量 `atype1,username1,hashdata1` 是 `GET` 提交到服务器的数据。

变量 `data1` 保存的是第二段的加密数据结果

变量 `data2` 保存的是第三段的加密数据结果

-------------------------------------------------------------------------------------------------------

```php
$lk = dechex(crc32((string)rand()));

$rk = dechex(crc32((string)rand()));

$bk = dechex(crc32((string)rand()));

$userandpass = md5($atype1 . "/" . $username1);

$hash = sha1($lk . $rk);

$bfishkey = sha1($bk);

$dhash = sha1($lk . $hashdata1 . $rk);

$bfish1en = $userandpass . $hash . $bfishkey;

$bfish1 = encode($bfish1en, $dhash);

$data1code = strToHex($bfish1);

$data1md5 = md5($data1code);

$data1 = $data1md5 . $lk . $rk . $data1code;

$bfish2 = encode($code1,$bfishkey);

$data2code = base64_encode($bfish2);

$data2md5 = md5($data2code);

$data2 = $data2md5 . $data2code;
```

--------------------------------------------------------------------------------------------------------

代码仅供交流。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `jx2dbt 验证服务端 php 源程序.rar`  
|
