
# [原创]SQL注入实例分析

dareyourmind网站SQL exploit分析，challenge地址：

[cf2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8S2M7X3g2&6L8%4g2J5L8h3W2F1k6q4)9J5k6h3&6W2N6q4)9J5c8X3#2W2L8Y4g2Q4x3X3g2H3K9s2m8Q4x3@1k6H3j5h3N6W2i4K6y4p5M7%4q4D9k6i4S2H3L8r3!0A6N6o6t1`.](https://bbs.kanxue.com/elink@df2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8S2M7X3g2&6L8%4g2J5L8h3W2F1k6q4)9J5k6h3&6W2N6q4)9J5c8X3#2W2L8Y4g2Q4x3X3g2H3K9s2m8Q4x3@1k6H3j5h3N6W2i4K6y4p5M7%4q4D9k6i4S2H3L8r3!0A6N6o6t1`.)

## 0x01 工具准备

SQL 自动注入的工具就算了，主要练习手动注入的分析能力。

（1）选用Burp suite拦截包，然后修改提交的内容即可（需要设置代理，仅分析个SQL略麻烦）；

（2）Firefox的Live HTTP headers插件，可以修改HTTP包的头信息（本文选用这个工具）

## 0x02 SQL注入分析

首先，测试下是否过滤特殊字符如 `'`、`"`、`,`、`---`、`/*`、`#` 等，输入 `1'`，如下图所示

![](https://bbs.kanxue.com/upload/attach/201407/410891_mtlbbb2bbmklzxz.png)

提交数据后，可看到爆出出错信息：对单引号没有过滤

![](https://bbs.kanxue.com/upload/attach/201407/410891_t33rgv1imby1p6u.png)

猜测查询语句应该为 `select column_name from table_name where user='输入数据'` 的形式。

那么修改输入数据，使用注释符屏蔽掉后续的单引号，试用`---`、`/*` 和`#` 后发现`#`可以起到注释作用。如下图所示，可以列出所有的GirlFriends：

![](https://bbs.kanxue.com/upload/attach/201407/410891_azbh0uei19smuxi.png)

题目中要求分析密码，那么我们下来需要看看这个table有几列？使用`order by`分析,在live http headers内修改数据包：`user='or  1=1 order by 3 #` ，发现弹出`User Name too long !!!`，oh oh 对输入的数据长度有限制。

![](https://bbs.kanxue.com/upload/attach/201407/410891_0i26rf8bcngaphc.png)

既然限制了输入长度，那直接猜测下列名吧：pass ? passwd? password?

输入`user='or  length(pass)>0#` [节约字符空间，能省得空格就尽量省],居然列出了所有的GrilFrinds，ok知道了列名为`pass`

接下来解析`pass`的长度：修改`length(pass)>`后的数字，当返回的列表为空时，就表示这个条件为false，那可以确定这个具体值：

`user='or length(pass)>6#` 返回所有列表

`user='or length(pass)>10#` 返回所有列表[注意多余的空格，占用有限的字符问题]

`user='or length(pass)>15#` 返回空列表

`user='or length(pass)>14#` 返回null

`user='or length(pass)>13#` 返回null

`user='or length(pass)>12#` 返回所有列表，那么说明`pass`的长度为13

`user='or length(pass)=13#` 返回所有列表，证明猜测正确

密码分析？

可以使用`substr(pass,1,1)>1 and substr(pass,1,1)<9`的形式 + 二分法 可以确定一个字符值，可是对字符长度做了限制，那就使用`mid` 代替？

使用`user='or mid(pass,1,1)>1#` 测试，发现可以返回所有的列表，那么我们可以使用其一一分析出各个字符。一个个的试感觉好麻烦的样子，是不是可以试试`python` 发包，分析的功能？

## 0x03 python半自动分析

从上面的分析发现，结果主要分为两种：1、输入的字符过长 2、返回表格列表，列表中有数据的话，表格的行数大于1

那么，可以用下面的代码自动获取每个字符的值。

![](https://bbs.kanxue.com/upload/attach/201407/410891_tixcamvk7qavm3w.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [sql-0.png](javascript:void(0);)

  （171.70kb，105次下载）

- [sql-1.png](javascript:void(0);)

  （8.89kb，55次下载）

- [sql-2png.png](javascript:void(0);)

  （37.43kb，59次下载）

- [sql3.png](javascript:void(0);)

  （35.65kb，33次下载）

- [sql4.png](javascript:void(0);)

  （35.76kb，96次下载）
