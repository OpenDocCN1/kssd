
# PHPwind7.5GBK 版漏洞简单分析！

> 原文：[`www.pediy.com/kssd/pediy12/120124.html`](https://www.pediy.com/kssd/pediy12/120124.html)

网上已经公布了这个漏洞的 exp 了！所以就看了一下 phpwind 的代码！

今天对简单的看了一下 PHPwind 的代码，漏洞主要是在 `pw_ajax` 里面,问题存在于该文件的 2010 行,变量 `fieldname` 在 19 多行的时候获取后判断不为空就直接进入了 sql 查询,但是与一般的注入不同的是,该变量在字段中,而不是 where 后,所以需要构造闭合一下,也就是字段名后要使用 `from`,后面跟一个 `union select` 语句就好了!还有一个条件是 `action` 必须等于 `pcdelimg`。当然这个需要我们登陆后才可而已。还有一个漏洞是需要跟这个漏洞配合的,那个漏洞我还没看.

…………省略 N 行代码

```php
} elseif ($action == 'pcdelimg') {

InitGP(array('fieldname','pctype')); //这里未过滤

InitGP(array('tid','id'),GP,2);

if (!$tid || !$id || !$fieldname || !$pctype) {
  echo 'fail';ajax_footer();
}
```

……………………省略 N 行 代码

```php
$path = $db->get_value("SELECT $fieldname FROM $tablename WHERE tid=". pwEscape($tid));
```

//这里进行了 sql 的查询，产生注入

群里有位朋友说呢，这个不适于批量获取 webshell，当然了公布的那个工具是不适合批量的获得，但是我们可以通过抓包分析，自动获得 cookie 的功能，批量获取 webshell 也是可以的。只不过这需要自己费一些时间去抓包，然后写代码！

注：我的 phpwind 的版本是 7.5gbk 的。我也是一个 PHP 初学者，欢迎各位一起交流！本人 QQ；704622720（BaCde）
