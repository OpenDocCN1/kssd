
因为在这种定向攻击中，攻击者几乎不会同一个样本多次使用。且样本的hash非常容易更改，文件哪怕只更改了一个字节就会得到一个新的hash，所以攻击者也完全可以使用脚本在一个样本后面添加空字节，这样同一个样本，就很轻松的有了几十个，几百个hash值。

## 2. IP Address - IP地址

正常情况下，样本都会有通信ip，也就常说的C2，在本次分析的样本中，只有通信域名，没有ip，我们可以解析一下该域名，看看目前域名指向了哪个ip。

通过解析，我们可以知道该域名目前指向了一个韩国ip：`114.207.244.99`。

同样的，样本的通信ip也和好拿到，大多数情况下，通过沙箱就可以获取。

IP的价值比hash稍微大一些，但通常攻击者都会使用跳板、匿名网路等技术。样本通信的IP跟攻击者所在地也不一定有直接的联系。

## 3. Domain Names - 域名

此样本的通信域名是：`antichrist[.]or[.]kr`

域名和ip类似，但域名的申请相对来说要更复杂一些。

因为申请域名毕竟有成本在里面，还需要提供一些基本信息。且DNS解析的时间成本大多数时候攻击者也需要考虑进去，所以通常来讲，域名的价值比ip稍微高一些。

很多时候一些简单的样本里面只有通信ip，没有域名，这可能就是攻击者节约成本的一个方式。

## 4. Network/host Artifacts - 网络特征、主机特征

就比如样本在与C2通信时候的一些请求格式、请求路径、服务器返回格式、服务器存放的文件名等。

比如在本次样本中，我们看到在与服务器通信的时候，请求路径为：`antichrist[.]or[.]kr/data/cheditor/dir1`

那么对于本次攻击来讲，`hostname/data/cheditor/dir1` 就可以看做一条网络特征。

之前也分析了，ip和域名都靠不住，是很容易更改的特征，因为换ip、换域名对于攻击者来说都是比较简单的事情，但是与本地样本通信的服务端程序就没有那么容易更改。或者说攻击者更不愿意去更改。

对于此次分析的这个样本来看，下载路径也是很容易改变的。不容易改变的主要是样本和C2通信时候的一些特征，比如样本会收集本地的基础信息，然后通过GET/POST的方式上传到服务器，服务器接收了之后存入数据库。

## 5. Tools - 工具特征

在本次攻击中并没有发现攻击者使用开源的黑客工具。

只下载了名为Lyric的组件，但是在网上并没有找到相关的信息，所以也可以猜测是攻击者自己写的工具。

但是我们并没有大量的样本验证说KimSuky的很多样本都会使用Lyric组件，只能说以后遇到使用了Lyric组件的样本，有可能是属于KimSuky的。

之前网上曝光了APT34所使用的一系列工具，比如PoisonFrog和Glimpse，这种才属于Tools特征。

## 6. TTPS

ttps的全称是Tactics, Techniques, and Procedures (战术、技术、过程)，是对攻击者攻击战术、攻击技术、攻击过程的描述，有时候也叫做行为库。

ttps的提取非常复杂，需要大量的样本、经验和长时间的积累才能完成。

由于篇幅和本人功能有限，不能很好地把TTPS的概念及提取讲清楚，有兴趣的伙伴可以看看之前[大佬发的关于TTPs的文章](https://bbs.kanxue.com/elink@78bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0p5I4x3e0l9^5i4K6u0W2K9s2c8E0L8l9`.`.)

## IOCs提取

根据金字塔以及分析到的内容，可以提取如下信息：

- hash

  原始样本hash：`35d60d2723c649c97b414b3cb701df1c`

  释放文件hash：`e54b370d96ca0e2ecc083c2d42f05210`

- IP Address

  韩国：`114.207.244.99`

- Domain Names

  `antichrist[.]or[.]kr`（已经曝光）

- Network/host Artifacts

  `hostname/data/cheditor/dir1`

- Tools

  `Lyric` / `Lyric64`

- TTPS

  ttps提取涉及的指标非常多。在本样本中有pdb路径以及互斥体名称。

  互斥体：`Papua gloria`

  样本PDB：`E:\pc\makeHwp\Bin\makeHwp.pdb`

## 域名关联

根据提取到的内容我们分别尝试进行关联。

在VT上搜索相关的C2可以看到与该C2相关的文件一共有三个，其中前两个是本次样本中已经分析到的，还有最后一个是暂时未见的。

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_J37WJAF85KUMH3K.png)

查看dc48.tmp，相关联的文件也如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_7J4989ZM7CR9SSD.png)

这里已经形成闭环了，看起来没有更多有效的信息，可以尝试使用搜索引擎试试。

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_AV57WJ2JBWVQQ9S.png)

这里可以看到，根据Google反馈的结果，该域名应该是跟APT组织kimsuky相关。我们继续往下找

在Twitter看到如下信息，基本可以确定是kimsuky

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_RDH7VBDDZ7YFFSC.png)

这里还拿到了一个新的hash：`2e5a70a1b4d7bd8b9a17e513efbe167d`

继续检索，最后可以发现在ests的博客上将其定义为了KimSuky的Operation Blue Estimate。

在该次攻击中所使用的诱饵如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_JX445MVSWPFUS5V.png)

可以看到，样本全是双后缀以迷惑用户。集合之前伪装为韩国KakaoTalk apk以及新闻app的攻击，这种伪装也算是KimSuky的一个常用手法。

## PDB关联

本次拿到的一系列样本中有两个样本的pdb都是：

`E:\pc\makeHwp\Bin\makeHwp.pdb`

所以尝试以makeHwp.pdb为关键字进行检索

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_JGWPSTJJGV6SN5X.png)

现在可以得知，makeHwp.pdb这个pdb文件名应该是KimSuky常用的，从19年4月份到20年2月份，都有包含该pdb的KimSuky样本。唯一不同的就是前面的路径不同，另一个路径如下：

`E:\PC\EstService\Bin32\makeHwp.pdb`

而`E:\PC\EstService\Bin32\makeHwp.pdb`是一个比较老的pdb路径了，最早追溯到19年1月份就已经在使用。

该路径关联的行动被命名为<眼镜蛇毒行动>和<假手术胶囊>。

这里也可以说明，KimSuky常会生成hwp文档作为诱饵进行样本斗投递，但是会根据伪装的内容不同而有着不同的路径。比如本次的根路径是makeHwp，可能代表是一个通用hwp制作的路径，而之前模仿estService的时候路径就是EstService，说明KimSuky会根据模仿目标的不同而改变到不同的路径下。

![图片描述](https://bbs.kanxue.com/upload/attach/202003/757351_ZRZ75G246YBEWE2.png)

## 0x06 总结

总的来说，这个样本不算是很复杂，在后面关联的时候发现了不少相关的资料，在分析该样本的时候，也尝试将更多的时间和精力用于介绍除开样本本身的内容。由于此次攻击的时间发生时间还是比较靠前，所以梳理起来还是比较容易和直观。就当做是个学习积累的过程了，希望之后可以把攻击关联和攻击事件分析的更好。

## 0x07 参考链接

ec1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0p5I4x3e0l9^5i4K6u0W2K9s2c8E0L8l9`.`.

1fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5h3I4&6j5h3y4Q4x3X3g2U0L8#2)9J5k6h3E0J5i4K6u0r3x3U0M7K6y4H3`.`.

e1bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3k6X3W2D9k6g2)9J5c8U0p5H3y4e0l9&6x3K6g2X3y4X3q4U0k6h3f1K6j5h3k6V1j5e0x3^5y4K6j5@1y4K6R3%4x3e0R3$3x3K6u0T1z5e0j5^5j5K6V1^5y4X3g2T1z5h3x3#2z5h3k6U0x3X3f1J5y4K6f1&6z5h3x3I4y4e0p5#2y4e0p5#2k6U0g2Q4x3V1k6V1k6i4c8S2K9h3I4K6

fb1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4c8W2j5%4c8Q4x3X3c8J5k6i4y4H3L8$3&6V1i4K6u0W2j5X3I4G2k6%4y4H3L8%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6y4Q4x3V1k6@1K9r3g2Q4x3X3c8H3P5i4u0S2L8h3W2V1i4K6u0V1L8$3k6Q4x3X3c8H3j5h3W2F1i4K6u0W2K9s2c8E0L8l9`.`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
