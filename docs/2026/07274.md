
# [翻译]使用有序马尔可夫链和用户信息加快密码破解速度

我一直在寻找加快密码破解的速度的方法。有很多关于这个话题的研究，但是有一个团队在这方面已经做了一些了不起的工作。这里有一篇论文 [OMEN: Faster Password Guessing Using an Ordered Markov Enumerator](https://bbs.kanxue.com/elink@cdbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5h3I4Q4x3X3g2S2M7X3y4Z5K9i4k6W2M7#2)9J5k6r3!0#2N6X3g2J5N6r3g2K6i4K6u0W2k6Y4u0Q4x3V1k6Z5j5h3I4Q4x3X3b7H3x3e0p5I4x3U0p5J5y4q4)9J5c8X3k6A6L8r3g2Q4x3V1k6G2L8h3g2F1i4K6u0W2M7r3c8X3)，就是我写这篇文章的灵感来源。虽然这篇文章的想法来自上面的论文，但是所有的工作都是我手动完成的，没有复制代码和其它任何东西。让我们开始吧！

## 使用有序马尔可夫链和用户信息加快密码破解速度

*比John the ripper 的马尔可夫和增强模式提高22.5%的准确性*

阅读完上面提到的论文后，我想要一个类似的工具，但是这些家伙制作的工具已经不可用了。因此，我花了几天时间编写代码来实现这个工具，它足够灵活，使用一个破解的密码列表来实现基于n-gram和马尔可夫链的密码生成。我想修改并编写我自己的工具给了我这样做的自由。这个工具仍处于开发阶段，一旦完成，我将会把它分享出来。

编写脚本后，我想看看我们是否可以利用用户信息来加快密码破解速度。这里有一个Fling泄漏的数据，Fling是一个成人交友网站。他们泄漏的数据库可以公开获取到，我下载了这个数据并进行了一些分析。让我们来看看吧。

## 用户在其密码中使用个人信息

通过一个简单的查询，我们可以看到用户密码与他们邮箱/用户名/用户代码/昵称有相同的前三个字母的用户占多大比例。在Fling泄漏的数据库中共有4993276个密码。让我们看看有相同起始三元组的占多大比例。

![](https://bbs.kanxue.com/upload/attach/201703/735423_f52fhr5ty9q5loe.jpg)

总密码的8% 意味着大约有386894个密码。这是一个很大的数字。

让我们看看1-grams。

![](https://bbs.kanxue.com/upload/attach/201703/735423_zok7tw9j040n87q.jpg)

17%的用户的密码与其邮箱、用户名、昵称有相同的首字母。

让我们玩玩生日和加入网站日期。我想看看有多少用户在他们密码结尾使用他们的生日。查询显示2%的用户这样做。下面是一些结果。

![](https://bbs.kanxue.com/upload/attach/201703/735423_3yg5x2vgz0izc43.jpg)

在做了一些非常基础的分析之后，我们已经知道大约有20%的用户在他们密码中使用其非常基本的信息。

如果这还不够，我还有另一个泄漏的数据（ClixSense），包含更多的用户信息，例如用户名、名字、姓氏、电子邮件、国家、城市、生日、加入日期、安全问题等。我精心编写了一个查询，来看看有多少密码的前`n-grams`同样出现在他们的社交信息中。结果令人惊讶，让我们看看它们。

使用2-grams和更多的社交信息例如生日，我们发现32.5%的用户在密码中使用他们的社交信息。下面是这个查询：

```
select count(password) from users where lower(substring(password,1,2)) = lower(substring(username,1,2))
or substring(password,1,2) = substring(first_name,1,2)
or substring(password,1,2) = substring(email,1,2)
or substring(password,1,2) = substring(last_name,1,2)
or substring(password,1,2) = substring(city,1,2)
or substring(password,1,2) = substring(security_answer,1,2)
or substring(password,1,2) = substring(address1,1,2)
or substring(password,1,2) = substring(security_answer,1,2)
or substring(password,1,2) = substring(zip,1,2)
or substring(password,1,2) = substring(payable_to,1,2)
or lower(substring(password,1,2)) = lower(substring(country,1,2))
or substring(password,length(password)-3,4) = substring(date_of_birth,1,4)
or substring(password,length(password)-4,4) = substring(date_of_birth,1,4)
or substring(password,length(password)-3,4) = substring(register_date,1,4)
or substring(password,length(password)-3,2) = substring(username,length(username)-3,2)
or substring(password,length(password)-2,1) = substring(username,length(username)-2,1)
```

结果是在200万密码中有65万密码使用社交信息。

我们可以使用这个来加速密码破解吗？是的，我们可以。让我们深入了解马尔可夫链（Markov Chain）。

## 马尔可夫链（Markov Chains）

简单来说，马尔可夫链所做的是，告诉我们一个字母在一个n-gram之后出现的概率。假设，我们有一个4-gram 的 'ilov'，马尔可夫链只会告诉我们下个字母出现的概率。在这个例子中，大部分时间，出现最高概率的字母是'e'，使它成为'ilove'。这就是你需要知道的关于马尔可夫链的所有知识。如果我们对这些概率进行排序，我们按照从最高到最低概率的顺序排列密码，故名有序马尔可夫链（ordered markov chains）。

John the ripper 使用未排序的马尔可夫链，从某种意义上来说，它完全遍历一个'n-grams'之后才到下一个'n-grams'。我的脚本的做法是，使用线程同时遍历所有高概率的'n-grams'。我希望已经解释清楚，如果没有的话，请在评论中提出任何问题。你也可以阅读文章开头给出的OMEN论文。

![](https://bbs.kanxue.com/upload/attach/201703/735423_whqh6lt89nbc7ok.png)

## 破解密码

一旦脚本完成，我将会分享这个处理数据的脚本。现在，我只是把结果和数据公布出来。

### 数据集

我破解所使用的数据来自于 MuslimMatch 数据库泄漏的数据。密码使用MD5加密存储，同时泄漏的数据库中还有用户名、用户代码、用户邮箱和用户国家。我现在忽略了用户国家，只是使用用户名、用户代码和用户邮箱。大约有77000个哈希值需要破解。

### John the ripper 增强模式

每种模式我想尝试1亿个长度在6-12位之间的密码。第一种模式是增强模式，结果是15000个哈希值被破解，占总数的20%。

### John the ripper 马尔可夫模式

这次我依然尝试了1亿个密码，同时设置`level=210`并保持密码长度不变。结果是有20000个哈希值被破解，占总数的25%。

### 用户信息的前三个字母作为`n-grams`的有序马尔可夫链

在这里，我从用户信息中获取 tri-grams, tetra-grams, penta-grams and hexagrams，并把它们作为密码的起始字母。接下来的字母使用了可以公开获取的[mate1](https://bbs.kanxue.com/elink@00aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2S2N6r3f1I4i4K6u0W2j5$3!0E0)密码列表。我只使用了500万个泄漏的密码数据，为了证明这种方法即使使用了更少的密码也可以执行的更好，因为John the ripper 的马尔可夫模型是基于1300万的rockyou密码训练而成的。这样，我使用一个的密码列表（mate1）进行训练，另一个不同的密码列表（muslimmatch）进行测试。

我使用n-grams用户信息方法生成了6000万个密码，使用简单的有序Markov链方法并使用mate1密码作为训练列表生成了4000万个密码。

结果是25000个哈希值被破解。 这大约是32％，多于增强模式和马尔可夫模式。

## 结论

为什么它会发挥作用？它之所以起作用，是因为大量用户在密码中使用他们邮箱、用户名或其它详细信息；如果我们首先检查以部分用户名、用户邮箱开始的n-grams密码，我们可以直观的加快准确性，而这个实验证明了这种直觉是正确的。准确性提高的另一个原因是：我使用了相同类别的单词训练列表，都是成人社交网站。这两个因素是准确性提高的主要原因。

为什么这样做是有意义的？在我的笔记本电脑上，John the ripper 在不到2分钟时间内就产生了1亿个密码，如果我们能够在不到2分钟内破解32%的密码，这是很棒的事情，不是吗？

**PS**：我还做了一些试验，在密码结尾添加生日，而且结果是非常理想的。

这就是我所做的工作。我希望这个想法可以被你们进一步的拓展，让我们看到一些令人惊讶的结果。出于隐私的原因，我不能上传整理后的泄漏数据。

**注意**：这绝不意味着我已经打破了john the ripper的速度和准确性记录。John the ripper是一些了不起的人开发的一个伟大的工具。我只是认为，使用用户社交信息作为`n-grams`的增强马尔可夫模型可以显著提高密码破解速度。欢迎批评指正。

**数据**: 60cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3j5h3W2\*7j5h3&6F1x3U0c8Q4x3V1k6g2M7$3W2F1k6#2)9J5k6p5!0J5k6r3g2J5k6h3c8Q4x3X3c8y4j5i4u0C8L8%4k6Q4x3X3c8o6K9r3q4A6L8Y4y4Q4x3X3c8S2L8X3c8Q4x3X3c8g2M7$3g2J5i4K6u0V1d9h3&6X3L8%4u0E0j5i4c8A6L8$3&6Q4x3X3c8@1L8#2)9J5k6q4y4H3k6h3g2V1i4K6u0V1N6i4m8Q4x3X3c8b7j5i4y4K6N6$3!0J5k6q4)9J5k6p5y4J5j5h3y4C8K9h3&6Y4

原文链接：[Using Ordered Markov Chains and User Information to speed up Password Cracking](https://bbs.kanxue.com/elink@b6dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6K6k6h3y4#2M7X3W2X3P5g2)9J5k6h3y4G2L8g2)9J5c8Y4g2K6K9h3&6Y4i4K6u0V1L8%4u0V1k6i4u0W2k6q4)9J5k6r3#2S2M7X3E0G2N6W2)9J5k6r3y4Z5j5h3W2F1M7#2)9J5k6r3q4F1k6q4)9J5k6s2g2K6k6i4u0Q4x3X3c8A6L8X3k6G2M7X3#2S2N6r3W2G2L8W2)9J5k6s2c8G2i4K6u0V1M7%4m8W2k6h3c8Q4x3X3c8#2M7q4)9J5k6s2m8S2M7%4y4%4L8%4u0V1i4K6u0V1j5%4u0S2j5$3E0A6L8X3M7`.)

本文由看雪翻译小组 FlamePeak 编译

LinkMe：[FlamePeak.com](https://bbs.kanxue.com/elink@a3fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6D9j5h3#2W2M7r3g2S2K9#2)9J5k6h3y4G2L8b7`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
