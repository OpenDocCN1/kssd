
# [原创][PHP]死亡exit绕过

源码：

```php
<?php
$content = '<?php exit; ?>';
$content .= $\_POST['txt'];
file\_put\_contents($\_POST['filename'],$content);
?>
```

## 1. `string.strip_tags`

无论你写入什么都会被插入一句`<?php exit; ?>`来强制结束掉PHP的运行。

假如正常尝试写入webshell：

![写入webshell](upload/attach/202201/921448_GPKBBNUTV8ZTAS4.png)

结果为：

![写入结果](upload/attach/202201/921448_XDQ2UQCYV8YQSMS.png)

如果正常的插入方法则会被插入结束语句，导致无法运行。

此时我们可以利用伪协议`php://filter`来进行绕过：

![使用伪协议](upload/attach/202201/921448_TRFZKCECY6DZPUU.png)

（虽然他说了7.3就废弃了，但是我7.4还是可以复现成功）

利用该过滤器可以去除文件中的PHP标签，如下所示：

![去除标签过程](upload/attach/202201/921448_DGEPNJMVGSD62ZP.png)

结果为：

![去除标签结果](upload/attach/202201/921448_CTMES7V4Q62PR9J.png)

成功去除了`exit;`。

但是我们直接写入webshell，会导致也被去除掉（因为也是PHP语句）。

此时我们可以使用base64加密的方法传入，利用`filter`可以接入多个过滤器。

Payload为：

```php
txt=PD9waHAgcGhwaW5mbygpOyA/Pg==&filename=php://filter/write=string.strip_tags|convert.base64-decode/resource=2.php
```

在使用`string.strip_tags`的同时使用`convert.base64-decode`（这里要注意两个过滤器的先后顺序，因为`filter`索引过滤器的时候是从左到右，如果位置调换了，也会导致无法写入）。

结果为：

![执行结果](upload/attach/202201/921448_BSESWTJFVE5EDPV.png)

可以看到`exit`成功被去除，并且写入了我们想要写入的语句。

## 2. `rot13`

首先我们需要知道`rot13`是什么：

![rot13介绍](upload/attach/202201/921448_2FJTJS28MVQHADN.png)

了解到`rot13`转换之后，我们可以get到，他只会把字母替换，而其他的数字、符号不受影响。

那么我们就可以这样来想，破坏掉`exit`来插入我们想要执行的语句。

Payload为：

```php
txt=<?cuc cucvasb(); ?>&filename=php://filter/write=string.rot13/resource=3.php
```

`<?cuc cucvasb(); ?>`为`<?php phpinfo(); ?>`的`rot13`转换。

看一下结果：

![rot13执行结果](upload/attach/202201/921448_UQRANXRQT5B453T.png)

可以看到，原本的`exit`已经被破坏，而我们想要插入的语句插入成功。

---

先记录这么多，后续会接着填坑QAQ

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
