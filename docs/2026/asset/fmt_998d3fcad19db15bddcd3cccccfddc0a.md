
使用`comment`字段也可以使我们的交叉形式简单得多，它不需要重新同步。我们使用第一条记录来产生两个中心目录，然后在两个列表中交替进行，使每条记录都跳过相应的另一种实现看到的相应的记录。

```
       C++     +-+     Java

           +-- |X| --+

           |   +-+   |

           |   PAD   |

     50k 0 |   PAD   | 24.975k 24.975k

           |   PAD   |

           |   PAD   |

           |   +-+ <-+

           |   |/| --+

           |   +-+   |

           +-> +-+   | size of

           +-- |\| <-|-- next

           |   +-+   |

   size of |   +-+ <-+

    next --|-> |/| --+

           |   +-+   |

           +-> +-+   | size of

           +-- |\| <-|-- next

           |   +-+   |

   size of |   +-+ <-+

    next --|-> | | --+

           |   ...   |
```

## 跳过入口

现在我们还有两个限制——我们必须在两个列表之间共享第一个文件，同时我们在两个版本的中心目录里必须有相同数量的文件。当然，除此之外，Java版本的索引表必须与原始文件完全一致（因为它们是签名的）。

尝试使用基于前一个bug（Hendy注：Master Key）的技巧可能还是有一定吸引力的。但除了那个bug看起来比较低级以外，在很多设备上那个bug已经被修复了，现在这个bug还是可以利用的。我们现在不能使用相同的文件名来解决这个问题——zip解析器会拒绝它们。

谢天谢地，现在有一种可以加到zip文件中的入口，它不需要被签名校验，因为它们没有内容可以签名——它们就是目录入口（它同样具有不同的实现处理不相同的特性）。

```
Enumeration<JarEntry> entries = jarFile.entries();

while (entries.hasMoreElements()) {

    final JarEntry je = entries.nextElement();

    if (je.isDirectory()) continue;

    final String name = je.getName();

    if (name.startsWith("META-INF/"))

        continue;

    final Certificate[] localCerts =

        loadCertificates(jarFile, je, readBuffer);
```

创建一个目录非常容易——所需要做的只是在文件名的后面加一个斜杠。同样的代码会跳过`META-INF/`目录下的文件，这些文件是用作签名校验系统的一部分。因为这个目录级别很高，我还没有试过使用`META-INF`目录来隐藏文件。

```
public boolean isDirectory() {

    return name.charAt(name.length() - 1) == '/';

}
```

## 历史沿革

当我在Android 2.3上测试我对这个漏洞破解实现时，我发现它用不了——老版本上正确处理了文件的签名，但现在这种处理方式被改了。出于好奇，我看了一下`libcore`的修改历史，可以确定这个bug是Google在2010的时候引入的。

Android的一个设计思想是他们如何得到一些标准库。他们没有使用GNU Classpath的实现，而选择了Apach Harmony（就我理解，很大程序上是因为license的原因）。Apache Harmony在2011年已经停止更新了——现在Google自己为Android维护着一份独立演化的版本。

结果就是，Harmony的`ZipFile`的原始实现是用C写的。通过JNI，C代码创建了每个`ZipEntry`对象。在两层代码之间传递数据显然需要大量的缓冲区拷贝，这在Android手机平台上会占用太多的内存。因此，Google用Java重写了`ZipFile`。

最早的`ZipFile`的Java实现并没有此bug，它小心地处理了符号的问题。实际上，写这个代码的人已经写了他们自己的实现来特殊处理读取无符号的小头（little-endian）值。它提供了`readShortLE`（返回int）和`readIntLE`(返回long)。

然而，他们认为对于中心目录头的每个字段都调用会比较慢（这个读操作即使在需要从zip文件中解出一个小文件时也需要进行），因此他们修改了读取头的逻辑，改为一次读取然后在解析的时候进行位操作（这是正确的）。

```
/*
 * We're seeing performance issues when we call
 * readShortLE and readIntLE, so we're going to
 * read the entire header at once and then parse
 * the results out without using any function calls.
 * Uglier, but should be much faster.
 *
 * Note that some lines look a bit different, because
 * the corresponding fields or locals are long and
 * so we need to do & 0xffffffffl to avoid problems
 * induced by sign extension.
 */
```

翻译：我们看到了调用`readShortLE`和`readIntLE`的性能问题。因此我们将一次读取整个头部，然后在解析时不需要进行任何函数调用。不够优雅，但应该快得多。

注意有些行看起来会有些不同，因为对应的字段或变量是long型，因此我们需要执行`& 0xffffffffl`操作以避免由有符号扩展引起的问题。

```
nameLen = (hdrBuf[28] & 0xff) |
         ((hdrBuf[29] & 0xff) << 8);

int extraLen = (hdrBuf[30] & 0xff) |
              ((hdrBuf[31] & 0xff) << 8);

int commentLen = (hdrBuf[32] & 0xff) |
                ((hdrBuf[33] & 0xff) << 8);
```

在2009年Harmony还没有关闭之前，Google向Harmony提交了最原始代码，这个项目后来就完全由Google来主持了，目的就是为了Android。我们因此可以在Harmony的bug tracker上找到作为[HARMONY-6346](https://bbs.kanxue.com/elink@13aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2S2M7r3q4U0K9r3g2Q4x3X3g2G2M7X3N6Q4x3V1k6B7K9i4u0S2i4K6u0r3j5Y4u0G2N6%4y4W2i4K6u0r3d9p5q4d9e0f1!0z5h3g2)9J5k6o6j5K6y4o6j5`.)问题附件的补丁，以及讨论。

一年之后，在2010年晚些时候，在项目过度（到Google）后，我们可以在Google的代码仓库里找到[引入这个bug的补丁](https://bbs.kanxue.com/elink@0deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6D9K9h3u0U0L8%4u0W2i4K6u0r3i4K6u0n7i4K6u0r3k6e0b7K6j5U0y4X3y4X3x3%4j5U0x3@1x3$3j5@1j5U0k6U0y4K6x3^5x3e0t1H3j5e0u0S2j5K6u0X3z5e0y4U0y4h3u0V1z5h3p5^5k6b7`.`.)。这个补丁是为了修改Android bug#3181430的，显然这会导致设置了CRC的高位时有符号扩展的bug（我从一个相关的[测试用例](https://bbs.kanxue.com/elink@85aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6D9K9h3u0U0L8%4u0W2i4K6u0r3i4K6u0n7i4K6u0r3x3U0M7@1z5e0f1I4z5e0N6T1y4X3j5@1x3e0q4V1k6U0p5%4x3e0p5$3x3o6t1J5j5X3b7H3k6U0f1@1y4e0S2S2k6h3y4V1z5h3x3^5y4q4)9J5y4e0g2q4i4K6t1I4i4K6u0r3)的评论中找到了这个描述）。

这个代码在使用int/long时非常危险。我不能只将字段改为int，因为它们似乎使用-1L来表示“未设置（unset）”，然而它仍然接受整个int取值范围的值（包括-1）。我们不得不查看zip的规范以确认是否是这样的。但目前来看，我们只能去避免使用有符号扩展。

我们现在知道，这个补丁引入了更多的有符号扩展问题。我也确信上面关于在为什么代码中从一开始就使用int和long这个问题上的理解是有误的——int字段实际上是用来保存16位数字（对于long，是32位），因此，上面说的“整个int取值范围”是没有用到的。

## 修复这个Bug

上面这些bug对于普通的用户来说有多大的威胁还是个问号——大部分用户得益于使用大的应用分发渠道，比如Google Play商店，（我们认为）它可以扫描这些漏洞以确保它们不是恶意软件。

然而，在[几天前Symantec发表的一篇文章](https://bbs.kanxue.com/elink@460K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4&6L8h3q4F1N6r3g2U0i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0F1L8X3g2U0N6q4)9J5c8X3u0D9L8$3N6K6i4K6u0r3k6X3W2J5M7%4c8Q4x3X3c8E0j5h3I4A6j5$3W2G2N6i4y4Q4x3X3c8#2M7$3g2Q4x3X3c8E0j5i4y4@1k6i4u0Q4x3X3c8C8k6i4W2Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6r3c8A6M7$3y4G2N6X3g2J5k6h3b7`.)中，说到在几个常用的市场上已经发现了一个中国开发者发布的利用了Master Key漏洞的恶意软件。他们提供了应用的截屏以及代码都开了些什么的详细信息。

我们在第三方应用市场上已经发现了4个由同一个攻击者制作的受感染的Android应用。这些app是一个非常流行的新闻app、一个街机游戏，一个纸牌游戏以及一个博彩app。所有这些app都是给中文用户设计的。

因此可以理解有些人可能更倾向于使用本地的修复来保护。除此之外，我认为像这样详细地给出漏洞原理而不提供可能的解决方案（比如给用户一个`hotfix`）是不负责任的。最后，我想给出这些问题是如何修复的，以及更进一步事情是如何工作的。

在我的前一篇文章中，我建议用户使用第三方的产品层面的修复。如果没有的话，我提供了一个可以修复第二种bug的补丁。然而，除了之前的警告之外，在研究这里的第二篇文章时，我发现[ReKay](https://bbs.kanxue.com/elink@1eeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2K9$3g2&6i4K6u0W2K9h3!0Q4x3V1j5`.)（我重点推荐的）并没有字修复bug#9695860（它的网站并没有明确地说明这一点）。

现在我将提供一个可以完全修改这两个bug的工具。我现在可以告诉那些希望修复的用户在`Backport`（这个名字是我放修复补丁的项目名字，意思是从Android的后续版本中“反向移植”的）就可以找到补丁。我本文的真正的目标是为了解释这个补丁是如何工作的。

在我发表这篇文章的几分钟后，Collin Muliner（他是ReKey的作者之一）联系了我，他刚发布了ReKey的新版本（在我发表这篇文章之前）。但我要说的是，它还是不能保护#9695860 bug。但我可以想像，在今后将会有可以完全修复这个bug的更新。假设真是这样的话，我将会再次推荐重点ReKey给那些希望保护自己免受此bug危害的用户（所以，用户们应该给他们一些时间）。

## Substrate扩展

要为现有设备修复这个bug，我们将使用Substrate——我为iOS以及Android提供的代码修改平台。开发者如果想要了解更多有关Substrate的东西，可以浏览[这个网站](https://bbs.kanxue.com/elink@5f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4&6k6r3W2S2M7%4g2T1M7%4c8J5j5i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3)，那里有完整的API文档。不过一些模板文件是非常简单的。

```
        public class Hook {

    public static void initialize() {

               ...

           }

}
```

第一个我们需要修改的部分是读取本地文件头的代码。它在一个叫`getInputStream`的方法中。它的参数是`ZipEntry`，返回值是用来读取内容的`InputStream`。我们的目标是检查要读取的字段，然后调用原始的实现以读取文件。

需要注意的是，我们的做法不会替换整个方法，因为可能有一些其它的基于`hook`的方法已经修复了这个bug——使用替换还不是hook补丁的方式会破坏那些可能依赖于这个类的行为。因此避免使用这种hook很重要。

为了访问文件头数据，我们使用`ZipFile`的`mRaf`字段，它是个[`RandomAccessFile`](https://bbs.kanxue.com/elink@06dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2j5%4y4Q4x3X3g2G2M7X3q4U0L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6B7j5i4k6S2M7$3g2Q4x3V1j5$3i4K6u0r3k6r3!0U0M7#2)9J5c8X3q4H3K9g2)9J5c8X3A6S2N6X3q4Q4x3V1k6A6L8#2)9J5c8W2u0S2L8X3c8G2L8f1q4U0j5$3g2K6M7@1k6A6L8r3g2Q4x3X3g2Z5N6r3#2D9)对象。`RandomAccessFile`是一个导出了一些像`readShort`等非常有用的方法的类。对`mRaf`的访问必须同步，因为调用者通常会先`seek`到它们感兴趣的位置，但扫描文件会改变它的状态。我们会检查两个修补的字段。

要替换方法的内容，我们使用Substrate的[`MS.hookMethod`](https://bbs.kanxue.com/elink@b5eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4&6k6r3W2S2M7%4g2T1M7%4c8J5j5i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6i4K6u0r3K9X3q4$3j5g2)9J5c8V1#2e0i4K6u0W2K9r3!0G2K9@1#2W2N6r3S2G2k6q4)9J5c8R3`.`.)方法。它允许我们重定向任何方法的实现。这个API使用一个叫`invoked`的functor，在我们提供的代码中，我们使用`invoke`来调用原始的实现。参数都被封装在一个数组里面。

在实际校验过程中，我们会拒绝那些符号位是1的导致解析错误的字段的zip文件（因为之前版本的Android解析这种包会失败，这是正确的。后面移除这样的逻辑是个损失。新的zip文件在老的设备上会失败可以明显说明这个问题）。因为我们读取的值是小头（little-endian）的，我们必须使用0x0080而不是0x8000来检查符号位。

```
final Field raf = ZipFile.class.

   getDeclaredField("mRaf");

raf.setAccessible(true);

final Field local = ZipFile.class.

   getDeclaredField("mLocalHeaderRelOffset");

local.setAccessible(true);

Method getInputStream = ZipFile.class.

   getDeclaredMethod("getInputStream", ZipFile.class);

MS.hookMethod(ZipFile.class, getInputStream,

   new MS.MethodAlteration<ZipFile, InputStream>() {

      public InputStream invoked(

         ZipFile thiz, Object... args

      )

         throws Throwable

      {
```
