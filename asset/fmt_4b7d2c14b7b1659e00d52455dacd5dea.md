
# SQL 注入漏洞测试工具比较

> 原文：[`www.pediy.com/kssd/pediy12/144626.html`](https://www.pediy.com/kssd/pediy12/144626.html)

SQL 注入漏洞测试工具比较

Sql 注入测试一定要使用工具。原因一：工作效率；原因二：人工很难构造出覆盖面广的盲注入的 sql 语句。例如当一个查询的 where 字句包含了多个参数，or and 的关系比较多时，简单的 `or 1=1`, `and 1=2` 是很难发现注入点的。

Sql 注入的工具很多(Top 15 free SQL Injection Scanners)，我最近使用的有 `Sqlmap`，`SqliX`，`JbroFuzz`，`Sql Power Injector`, 网站啄木鸟.现将他们的使用方法和比较结果贴于此：

## Sqlmap

Sqlmap 是 python 开发的 SQL 注入漏洞测试工具。没有 UI 界面的命令行工具。虽说是命令行工具，可他的使用比网站啄木鸟，Sql Power injector 容易多了，并且有很详细的帮助文档。从下面 2 个地址获得相关程序包：

Python 下载，推荐下载 2.7 版本，兼容性好：[`www.python.org/ftp/python/2.7.2/python-2.7.2.msi`](http://www.python.org/ftp/python/2.7.2/python-2.7.2.msi)

SqlMap 下载：[`sqlmap.sourceforge.net/`](http://sqlmap.sourceforge.net/)

SqlMap 根目录下 `Sqlmap.py` 是主程序，`sqlmap.conf` 是配置文件。Sqlmap 的使用有 2 种方式：

1 在 `cmd` 中直接输入命令行。
2 在 `sqlmap.conf` 中配置命令行参数，然后在 `cmd` 中用 `sqlmap.py -c sqlmap.conf` 发起攻击。

我比较喜欢第二种方式。这里仅列出 `sqlmap.conf` 种几个命令。更详细的命令描述参考 `doc` 目录下的 `readme.pdf`, 

# Target URL.

# Example: http://192.168.1.121/sqlmap/mysql/get_int.php?id=1&cat=2

`url` = 

指定攻击的 url 地址。

# Data string to be sent through POST.

`data` = 

如果是 POST 命令的话，请在 `data` 字段填上 POST 的数据。

# HTTP Cookie header.

`cookie` = 

如果网站需要登录的话，请在 `cookie` 字段填上 cookie 数据。cookie 数据可以用 `wireshark` 抓包得到。

# Alert with audio beep when sql injection found.

`beep` = True

建议把 `beep` 设为 `True`，当发现注入点时 Sqlmap 会嘀的一声，很提神的声音。

相关命令设置好以后，在命令行中输入 `sqlmap.py -c sqlmap.conf`，就可以开始 Sql 注入测试啦。发现注入点(嘀的一声响)后，就可以通过设置其他命令来尝试获取更多的后台数据库信息。

Sqlmap 功能非常多，获取数据用户名/密码/角色，数据库表，查询数据库表内容，上传/下载文件文件，修改注册表 deng。当然有些功能是在当前数据库用户用一定权限的前提下才能完成。

## SqliX

`SqliX` 是由 OWASP 用 Perl 开发的 SQL 注入漏洞测试工具。跟 Sqlmap 一样是命令行驱动，也和 sqlmap 一样的好用。没有详细的帮助文档，项目主页上有简单的安装和使用介绍。

Perl 下载地址：[`www.activestate.com/activeperl/downloads`](http://www.activestate.com/activeperl/downloads)

SqliX 下载地址：[`www.owasp.org/index.php/Cate..._SQLiX_Project`](https://www.owasp.org/index.php/Category:OWASP_SQLiX_Project)

`SqliX` 依赖 2 个包：首先使用下面的命令安装依赖包：
```
perl -MCPAN -e 'install WWW::CheckSite'
perl -MCPAN -e 'install Tie::CharArray'
```
如果 `CheckSite` 安装失败，出现如下错误信息：
```
make test had returned bad status, won't install without force
```
使用下面的命令强制安装：
```
perl -MCPAN -e "CPAN::Shell->force(qw(install WWW::CheckSite));"
```
安装成功后就可以在 `cmd` 中开始 `SqliX` 的注入测试了。`SqliX` 的命令相对较少：

`-url` 给定测试的 URL 地址
`--post_content` 如果是 POST 命令的话，请加上 POST 数据
`-crawl` 这是一个唯一 Sqlmap 没有的功能。他指定一个 url 的根目录，爬虫原理爬出所有的链接地址，然后对每个地址进行注入测试。
`-v` 显示信息级别，值越大，显示的信息越多。
`-all`
`-method_taggy`
`-method_error`
`-method_blind`
`-method_blind_integer`
`-method_blind_string`
`-method_blind_statement`
`-method_blind_comment`

以上 这些是根据不同的注入原理得出的不同的注入方法，一般选择 `-all` 或者 `-method_blind`，应用所有的注入方法。

例如：`sqlix.pl -url http://target1.edu/extensioninfonet/article.asp?id=3602 -method_blind -exploit -v=2`

几秒钟后，显示信息如下：
```
Analysing URL [http://target1.edu/extensioninfonet/article.asp?id=3602]
  http://target1.edu/extensioninfonet/article.asp?id=3602
         [+] working on id
                 [+] Method: SQL Blind Integer Injection
                         [FOUND] Blind SQL Injection: Integer based
                         [INFO] Current function:
                         [INFO] length: 0
                         [FOUND] SQL Blind Integer Injection
RESULTS:
The variable [id] from [http://extension.missouri.edu/extensioninfonet/article.asp?id=3602] is vulnerable to SQL Injection [Integer without quote -].
```
“is vulnerable” 表明这个 url 是有注入漏洞的。

SqliX 发现漏洞的能力确实不错，不比 Sqlmap 差。而漏洞利用的能力远不如 Sqlmap，他只能执行一些函数和 sql 语句。事实上在不知道后台数据库表名的情况下很难执行 sql 语句。SqliX 这样设计是可以理解的，他的目的只是为了进行 Sql 注入漏洞测试发现漏洞，才不希望被用来干坏事呢。

## JbroFuzz

`JbroFuzz`，是 OWASP 开发 Web 应用程序模糊(Fuzz)测试工具. 有 UI 界面，但真的不好用，很多常用操作都没有快捷键的支持。他提供 Sql 注入相关的有效载荷(Payload,这个词真没有合适的中文翻译)。直接看看 Sql 注入的 payload 吧：

`' or 1=1--`

`and 12=21`

`or '7659'='7659`

`union all select @@version--`

`union all select 1,2,3`

`1 and user_name() = 'dbo'`

等等 payload 有很多。JBroFuzz 用这些预定义的 payload 去替换 http 请求中的参数值(可以是请求的任意字段)，保存 http 返回数据，并对返回数据做统计分析，形成统计图表，通过人工分析统计图表来判断是否有注入漏洞。很费劲，建议不要用 JbroFuzz 进行 Sql 注入漏洞测试。他的价值在于很多的 payload，基本上每个 payload 对应一种注入方法，可以作为 Sql 注入学习和手工测试的参考。

除了 Sql 注入，JRoFuzz 还可以进行其他多种 Web 应用程序相关的测试例如：XSS，LDAP 注入，XML 注入等。

## SqlPowerInjector

悲惨，真不好用。没有其他评价。

## 网站啄木鸟

下载地址：[`www.skycn.com/soft/45424.html`](http://www.skycn.com/soft/45424.html)

能发现简单的注入漏洞。`wireshark` 抓包可见，他构造的 sql 注入 payload 仅仅是简单的`"and 1=1--"`, `"and 1>1--"`等。 在发现漏洞后啄木鸟猜表面的能力也很弱，只是依靠查询 MS-sql 和 Oracle 的 user table 来获取用户表名。

## 实际测试结果

以下是 8 个漏洞 url，用于测试 Sqlmap，Sqlix，JBroFuzz，网站啄木鸟。即使我人工分析 JBroFuzz 统计数据，没有发现注入漏洞的线索。具体结果看下表：

`Sqlmap` `啄木鸟` `SqliX` `http://www.Target.com/news.php?id=92`

`Sqlmap` `啄木鸟` `SqliX` `http://www.Target.com/article.asp?ID=102&title=Fast%20food%20marketing%20for%20children%20is%20on%20the%20rise`

`Sqlmap` `啄木鸟` `SqliX` `http://www.Target.com/article.asp?id=1969`

`Sqlmap` `No` `SqliX` `http://extension.Target.edu/extensioninfonet/article.asp?id=3602`

`Sqlmap` `No` `SqliX` `http://school.Target.com/tools/nutrition-education/view.php?id=23945657`

`Sqlmap` `No` `SqliX` `http://www.Target.com/ebookprint.php?id=245`

`Sqlmap` `No` `SqliX` `http://www.Target.com/******-nutrition-facts.php?id=21019`

`Sqlmap` `No` `SqliX` `http://www.Target.com/******/artinfo.php?id=6534`

最后结果：`Sqlmap` 是个很好的 Sql 注入测试工具。

见笑了，给自己的总结，供大家参考。
