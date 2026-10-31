
# [原创]浅谈SQL注入

## 一：目录

1. 如何判断SQL注入
2. 获得我们想要的数据的思路
3. 绕过防御

## 二：开始

在开始之前我先谈谈我对SQL注入的三种分类。看过很多文章，每个人基于不同的原理把SQL注入分为很多类，在这其中，我最赞成的是《SQL注入与防御》中的那种分类方法。上面把SQL注入分为这三类：基于错误、基于时间、基于内容。这同时也是我最赞成的观点。下面的一些叙述都是按照这三个类别来分别叙述的。

### 1：如何判断SQL注入

判断是否是注入，总的思路在于参数后面的内容我们是否可以控制。

我认为注入分为这三类，那么判断当然也按照这三类的思路。

**基于错误：** 当我们在参数后面加个单引号，例如`id=1'`，可能页面会报错，是直接显示出数据库错误的那种报错。那么可能这里就是一个注入点。

**基于内容：** 用基于内容的思路来判断，就是看参数后的内容我们是否可控制。当我们用`id=1`可以获取一个数据页面的时候，我们可以修改一下参数，我们用`id=2-1`，或者`id=0+1`（`+`号需要编码，`%2b`）来试一试，看看返回的页面的内容是不是跟我们`id=1`返回的内容一致。如果返回的内容一致，那么说明`id`后面的参数我们是可以控制的，那么也说明这是一个注入点。

**基于时间：** 我们一样用`id=1`来讨论一下吧。如果我们在`id=1`后面加上一段代码，例如这样`and if(1=1,sleep(5),1)`，这是一段`if`语句，就是如果`1=1`，则延迟5秒，否则返回`1`。那么当我们要访问的页面过了一段时间才可以被访问，这就说明了这是个注入点，我们在`id=1`后面有可以控制的参数。

### 2：获得我们想要数据的思路

**基于错误：** 个人认为基于错误是最简单的，因为他会返回数据库中的错误信息，来让我们省下很多时间。如果可以`order by`等，这些那么就更简单了。

下面提供一些报错语句：

通过`floor()`报错：

```
b5eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3L8X3q4E0k6g2)9K6c8q4)9J5y4#2)9J5b7X3!0J5i4K6u0n7i4K6t1^5M7$3g2D9k6h3y4@1i4K6u0n7x3g2)9J5b7X3k6J5L8$3#2Q4x3U0S2K6k6h3I4W2j5%4c8Q4x3V1u0U0L8%4g2F1N6q4)9J5z5q4)9J5b7g2)9J5z5g2)9J5b7$3y4G2L8X3y4S2N6q4)9J5z5s2g2K6k6i4u0Q4x3U0S2Q4x3U0W2Q4x3V1x3H3P5o6N6W2i4K6u0o6k6X3I4G2L8%4u0Q4x3U0S2J5j5h3&6V1i4K6t1^5x3q4)9J5z5g2)9J5b7e0u0Q4x3U0W2Q4x3U0W2^5i4K6u0n7k6Y4u0G2L8g2)9J5b7X3W2F1k6X3!0J5L8h3q4@1K9h3!0F1i4K6g2X3M7$3y4Z5k6h3#2S2i4K6u0W2N6r3q4T1L8r3g2K6i4K6u0n7k6%4u0G2N6i4m8Q4x3V1u0T1P5g2)9J5b7Y4S2Q4x3U0W2S2i4K6t1&6i4K6u0n7i4K6t1#2x3U0y4Q4x3U0k6S2L8i4m8Q4x3@1u0H3j5i4y4K6i4K6y4p5x3b7`.`.
```

通过`extractvalue()`报错：

```
294K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3L8X3q4E0k6g2)9K6c8q4)9J5y4#2)9J5b7X3!0J5i4K6u0n7k6i4S2@1M7X3q4U0N6s2k6S2L8s2g2W2i4K6t1^5x3g2)9J5b7$3y4G2L8X3y4S2N6q4)9J5z5s2g2K6k6i4u0Q4x3U0S2Q4x3U0W2Q4x3V1x3H3P5o6N6W2i4K6u0o6N6X3g2J5M7$3W2G2L8W2)9J5z5q4)9J5z5g2)9J5z5g2)9J5z5g2)9J5z5g2)9J5b7W2)9J5y4e0t1K6i4K6t1$3j5h3#2H3i4K6y4n7M7r3q4K6M7#2)9K6c8o6p5`.
```

通过`updatexml()`报错：

```
a98K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3L8X3q4E0k6g2)9K6c8q4)9J5y4#2)9J5b7X3!0J5i4K6u0n7N6i4m8V1j5i4c8W2P5r3#2D9i4K6t1^5x3g2)9J5b7$3y4G2L8X3y4S2N6q4)9J5z5s2g2K6k6i4u0Q4x3U0S2Q4x3U0W2Q4x3V1x3H3P5o6N6W2i4K6u0o6N6X3g2J5M7$3W2G2L8W2)9J5z5q4)9J5z5g2)9J5z5g2)9J5b7K6q4Q4x3U0W2Q4x3V1u0Q4x3U0f1J5x3#2)9J5y4X3q4E0M7q4)9K6b7Y4m8S2M7%4y4Q4x3@1b7I4
```

通过`exp()`报错：

```
bfeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3L8X3q4E0k6g2)9K6c8q4)9J5y4#2)9J5b7X3!0J5i4K6u0n7c8g2S2b7i4K6t1^5i4K6N6q4i4K6t1^5f1@1g2x3c8f1y4f1 * from(select user())a))+%23&pass=1
```

通过`NAME_CONST`（适用于低版本）报错：

```
153K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3L8X3q4E0k6g2)9K6c8q4)9J5y4#2)9J5b7X3!0J5i4K6u0n7i4K6t1^5M7$3g2D9k6h3y4@1 * from (select NAME_CONST(version(),1),NAME_CONST(version(),1)) as x)+%23&pass=1
```

**基于内容：** 当我们遇到这种`id=1`，我们可以用`id=2-1`来获得相同页面的时候，我们可以这样构造语句：`id=2-1/(if(user='root'),1,0)`。当`user`是`root`的时候，`if`函数返回的值为`1`，`id`的值也就是`1`，返回的页面也就相同；当`user`不是`root`，`if`函数返回的值就是`0`，`2-0=2`，返回的页面跟我们`id=1`的页面是不同的，`user='root'`也就是我们可以修改数据来判断的地方。

**基于时间：** 基于时间是最花费时间的，因为我们用基于时间的时候时间是不可以设置太小的，例如`sleep(2)`只延迟2秒。如果网站访问速度本来就不是怎么快的话，会影响我们对数据的判断的。

基于时间的找我们可以修改数据的方法是在函数后面加判断语句。我们可以在函数后面加`if(user='root',sleep(5),1)`，这样当`user='root'`的时候，就会延迟5秒，那么我们也就可以慢慢来判断我们需要的数据了。

**总的来说：** 就选择上面`基于错误 > 基于内容 > 基于时间`。毕竟基于时间的方法是花费最久的，但是当前两个方法都不可用，也就是报错统一`404`，返回内容统一一个页面（可能是过滤了敏感参数）的时候，这时候如果是一个注入点，我们就只能用基于时间的方法了。

### 3：绕过防御

**内联注释：**

```
id=1/*!UnIoN*/+SeLeCT+1,2,concat(/*!table_name*/)+FrOM /*information_schema*/.tables /*!WHERE */+/*!TaBlE_ScHeMa*/+like+database()-- -
```

**编码绕过：**

如`URLEncode`编码，`ASCII`，`HEX`，`unicode`编码绕过。

**空格绕过：**

两个空格代替一个空格，用`Tab`代替空格：`%20` `%09` `%0a` `%0b` `%0c` `%0d` `%a0` `/**/`

括号绕过空格：在`MySQL`中，括号是用来包围子查询的。因此，任何可以计算出结果的语句，都可以用括号包围起来：`select(user())from dual where 1=1 and 2=2;`

**换行符绕过：**

`%0a`、`%0d`

**宽字节绕过：**

过滤单引号时，可以试试宽字节：`%bf%27` `%df%27` `%aa%27`

**反引号绕过：**

select `version()`，可以用来过空格和正则，特殊情况下还可以将其做注释符用。

**等价函数绕过：**

`hex()`、`bin()` ==> `ascii()`

`sleep()` ==> `benchmark()`

`concat_ws()` ==> `group_concat()`

`mid()`、`substr()` ==> `substring()`

`@@user` ==> `user()`

`@@datadir` ==> `datadir()`

举例：`substring()`和`substr()`无法使用时：

```
?id=1+and+ascii(lower(mid((select+pwd+from+users+limit+1,1),1,1)))=74
```

或者：

```
substr((select 'password'),1,1) = 0x70
strcmp(left('password',1), 0x69) = 1
strcmp(left('password',1), 0x70) = 0
strcmp(left('password',1), 0x71) = -1
```

个人博客：`ac5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0&6N6$3q4D9K9%4y4Q4x3X3g2U0L8$3@1`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
