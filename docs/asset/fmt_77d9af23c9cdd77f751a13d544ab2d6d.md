
# 反编译黑莓手机 COD 格式文件（首发看雪和 maxpda）

> 原文：[`www.pediy.com/kssd/pediy12/132659.html`](https://www.pediy.com/kssd/pediy12/132659.html)

最近接触黑莓手机，黑莓手机的软件格式是 COD 文件，先是把 JAVA 原文件打包成 JAR，然后在把 JAR 文件打包成 COD 格式。 COD 格式是 RIM 公司的专利，具体信息可以看这里

`http://appft1.uspto.gov/netacgi/nph-Parser?Sect1=PTO1&Sect2=HITOFF&d=PG01&p=1&u=%2Fnetahtml%2FPTO%2Fsrchnum.html&r=1&f=G&l=50&s1=%2220060020932%22.PGNR.&OS=DN/20060020932&RS=DN/20060020932`

下面来介绍一下我们要使用的软件。

`JAVA coddec`（国外大牛写的反编译 COD 格式文件的程序） `JDE`

## 1. JAVA 推荐安装 `JDK1.6` 版本

在 `CMD` 命令下输入 `JAVA` 查看是否安装成功

![](img/2f0acdc7b3510c6e94aaf5a314f832a3.jpg)

## 2. 安装 `JDE`

`JDE` 是黑莓公司的开发环境 这步非必须

请参考这个帖子 `http://bbs.berrymail.cn/thread-6415-1-3.html`

或者直接去官网下载

## 3. `CODDEC`

在国外牛人博客上

`http://drbolsen.wordpress.com/2008/07/14/coddec-released/`

但是不知道是网速原因还是其他原因 下载一直是 CRC 校验失败

在网上找到了国内的下载地址

`http://ishare.iask.sina.com.cn/f/14737069.html`

然后把 `CODDEC` 放到 `C` 盘根目录 如果你想放到其他目录 请修改目录里 `RUN.BAT` 的文件

文件信息如下

```
FOR %%A IN ('DIR *.cod /B') DO java.exe -classpath c:\coddec\bin\Release net.rim.tools.compiler.Compiler %%A
```

把你 `CODDEC` 的存放目录修改为你想放的目录。

下面开始来进行实战操作

下下载一个黑莓的软件 我在这个地址下载了

`http://www.51blackberry.com/rj_Show.asp?SoftID=1236`

土豆网的一个客户端

然后把下载的文件解压 把里面以 `cod` 结尾的文件放到 `CODDEC` 的目录下

如下图

![](img/dbb6e3f04ffca119532d9cd7a4c7840f.jpg)

从 `CMD` 进入 `CODDEC` 的目录 如下图

![](img/7bb293da2c6d8191aed705ea69ce5663.jpg)

在 `CMD` 下输入

`run.bat Tudou.cod`（或者其他你想反编译的文件）

然后回车就 OK 了

下面是反编译过程和结果

![](img/31fa9e7a0ba44cb921215c8486e286cf.jpg)

![](img/d012ff9ee7a20ac7511b802f121b922f.jpg)

![](img/fb15be8b45ef8ccb463ce218e6a4d6af.jpg)

过程比较简单 不足之处请多多指教：）
