
# [翻译]构建安全AI——加密深度学习教程

**TLDR：**在这篇博客中，我们要训练一个训练期间完全加密的神经网络（使用未加密数据）。这个神经网络将会有两个有益的特性。首先，该神经网络智被保护不受那些想要偷窃它的人的伤害，即允许在不安全的环境下训练贵重的AI而无被盗窃的风险。第二，该神经网络可以只作出加密的预测（这对外部世界可能没有什么影响，因为外部世界无法理解没有密钥的预测）。这就在用户和超级智能之间创造出一个非常重要的权利不平衡。如果这个AI被同态加密，那么在它看来，整个外部世界也被同态加密了。控制该密钥的人就可以选择解锁这个AI（将它公之于众）或者仅仅个人拥有这个AI做出的预测（似乎更安全）

当我在[@iamtrask](https://bbs.kanxue.com/elink@db9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9h3q4E0N6s2u0S2M7$3D9`.)上完成一篇新博客后，通常会发推特。如果你有兴趣阅读更多的博客的话，可以在twitter关注我，在此感谢所有的反馈。

## 超级智能

许多人担心超级强大的AI有一天会选择伤害人类。最近，斯蒂芬·霍金呼吁[新世界政府](https://bbs.kanxue.com/elink@c8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3N6i4c8#2M7X3W2K6L8g2)9J5k6h3y4G2L8g2)9J5c8Y4y4@1k6i4m8Z5k6h3&6Q4x3X3c8Z5j5i4N6C8K9h3&6Y4i4K6u0V1k6X3W2F1j5h3I4D9P5g2)9J5k6s2u0W2N6X3g2S2L8r3g2V1i4K6u0V1K9r3W2K6i4K6u0V1M7r3I4S2L8W2)9J5k6r3k6G2M7W2)9J5k6s2m8J5k6i4k6W2L8Y4c8A6L8X3N6Q4x3X3c8S2L8W2)9J5k6r3q4A6i4K6u0V1j5i4m8G2j5$3q4D9P5i4m8K6k6g2)9J5c8R3`.`.)控制我们给予人工智能的能力，这样人工智就不会反过来毁灭我们。这是相当大胆的言论，我认为这反映出了科学界和社会的一种普遍担忧。在这篇博客中，就这个问题我想给出一种可能的技术解决方案，并用一些示例代码来证明该方法。

目标很简单。我们想要构建AI科技——可以变得极度聪明（聪明到可以治愈癌症，解决世界饥饿问题等等），但是它的智能被一个持有密钥的人所控制，因此智能的应用被限制了。不受限的学习固然很棒，但是那种知识的不受限制的应用可能是危险的。

为了介绍这个想法，我会非常快速地描述两个令人兴奋的研究领域：深度学习和同态加密。

## Part 1：什么是深度学习？

深度学习是为了智能自动化的一套工具，主要利用神经网络。作为计算机科学的一个领域，深度学习和近来AI科技的繁荣有着密不可分的关系，因为它在许多智能任务中已经超过之前的质量记录。例如，它在[DeepMind的AlphaGo](https://bbs.kanxue.com/elink@ff2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6h3g2H3L8h3W2F1k6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3j5h3I4H3K9r3q4Y4L8#2)9J5c8R3`.`.)系统，最近击败了世界围棋冠军李世石，中扮演了很重要的角色。

### 问题：神经网络如何学习？

神经网络基于输入作出预测。它学着通过试验和错误有效地作出预测。一开始它先作出一个预测（最初有很大的随机性），然后收到一个"错误信号(error signal)"——表明它预测的太高或者太低（通常可能性）。在这个循环重复几百万次甚至几千万次后，神经网络开始作出合理的预测。若想要了解更多关于此过程的细节，请看[A Neural Network in 11 Lines of Python](https://bbs.kanxue.com/elink@340K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2S2L8i4c8J5j5i4y4C8i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6N6Q4x3V1j5I4x3W2)9J5c8X3u0S2M7$3W2U0i4K6u0V1M7s2W2@1K9r3!0F1i4K6u0V1L8X3g2@1N6$3!0J5K9#2)9J5c8R3`.`.)

其中的关键点就是这个错误信号。要是神经网络不被告知它的预测的好坏程度，它就无法学习。这一点很重要，牢记。

## Part 2：什么是同态加密？

人如其名，[同态加密](https://bbs.kanxue.com/elink@847K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4K9i4u0W2k6q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0c8Q4x3V1j5I4x3g2)9J5c8X3S2S2j5$3E0W2M7W2)9J5k6r3I4W2P5r3W2U0L8$3&6Q4x3X3c8Z5L8$3#2G2L8h3!0J5M7r3S2A6j5#2)9J5k6r3g2F1j5%4u0&6M7s2c8A6L8$3&6Q4x3V1j5`.)是一种加密的方式。在非对称情形下，它可以使用公钥（public key）完美地将可阅读文本（明文）转换为一堆乱七八糟的东西（密文）。更重要的是，它还能使用私钥（secret key）将那些密文还原为相应的明文。理论上，除非你拥有私钥，否则你是无法破译密文的。

同态加密是加密方法中的一种特殊类型。它允许有人以特定方式修改加密的信息而不必能读懂那些信息。例如，数字可以做同态加密，比如对加密的数值可以执行乘法和加法而无需解密它们。这里有一些简单的例子。

![](https://bbs.kanxue.com/upload/attach/201704/747240_b59565332e2fc902.png)

如今，有关同态加密的方案越来越多，每种方案都有不同的特性。这是一个相对年轻的领域，有一些重要的问题还正在研究之中，但是我们以后再说。

接下来我们只讨论以下方案。基于乘法和加法同态的整数公钥加密方案，此外可以执行上面图片中的运算。更进一步，由于公钥允许"单向"加密，你甚至可以在未加密数字和（通过单向加密的）已加密数字之间执行运算，例如上面的`2 * Cypher A`（有些加密方案甚至连那个也不需要……但是……我们以后再说。）

## Part 3：我们能将深度学习和同态加密一起使用吗？

或许深度学习和同态加密之间最常见的交点已经证明是数据隐私方面。事实证明，当你同态加密数据后，虽然你无法读懂它的含义，但是你仍然保留了大部分有趣的统计结构。这就允许人们使用加密的数据([CryptoNets](https://bbs.kanxue.com/elink@a42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4S2A6N6W2)9J5k6h3!0J5k6#2)9J5c8X3q4T1M7#2)9J5c8U0p5@1x3e0u0Q4x3X3f1$3x3e0R3I4))训练模型。此外，一个新成立的对冲基金[Numer.ai](https://bbs.kanxue.com/elink@b8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6h3#2W2M7X3q4A6i4K6u0r3k6h3&6U0M7Y4W2H3N6r3g2V1i4K6u0V1k6r3q4@1j5g2)9J5k6r3k6G2M7W2)9J5k6r3g2X3k6X3W2U0K9h3g2F1N6q4)9J5k6r3#2S2M7X3E0W2N6s2y4Q4x3X3c8X3k6X3k6T1k6e0V1%4y4o6y4T1j5e0R3`.)加密了昂贵的，专有的数据，并允许任何人尝试去训练机器学习模型以此来预测股市。正常情况下他们无法这样做，因为这要放弃极其珍贵的信息（正常的加密不可能训练模型）。

但是，这篇博客将要做的是相反的事情，加密神经网络并使用未加密数据训练它。

一个神经网络，看起来惊人的复杂，实际可分解为非常小的、简单的一遍又一遍地重复的移动单元。事实上，许多最先进的神经网络只能使用下面的运算来创建：

- 加法
- 乘法
- 除法
- 减法
- [`Sigmoid`](https://bbs.kanxue.com/elink@1bfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2N6r3S2%4L8%4u0D9k6q4)9J5k6i4N6G2L8r3k6J5j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9h3N6E0L8$3W2V1c8Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2Z5N6r3#2D9)
- 双曲正切([`Tanh`](https://bbs.kanxue.com/elink@f12K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2N6r3S2%4L8%4u0D9k6q4)9J5k6i4N6G2L8r3k6J5j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6t1P5i4m8W2M7X3u0G2L8r3W2U0g2r3q4F1k6$3g2F1N6q4)9J5k6h3S2@1L8h3H3`.))
- 指数运算([`Exponential`](https://bbs.kanxue.com/elink@286K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4P5s2m8G2L8X3g2F1N6r3W2S2L8q4)9#2k6X3k6#2L8X3y4@1K9h3!0F1))

现在，问一个明显的技术性问题，我们能否同态加密神经网络？事实证明，做一些保守的近似后，这是可以做到的。

- 加法－现成可用的
- 乘法－现成可用的
- 除法－现成可用的？－简单地１／乘法
- 减法－现成可用的？－简单地加法的反面
- [`Sigmoid`](https://bbs.kanxue.com/elink@e77K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2N6r3S2%4L8%4u0D9k6q4)9J5k6i4N6G2L8r3k6J5j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9h3N6E0L8$3W2V1c8Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2Z5N6r3#2D9)－呃...有点难
- 双曲正切([`Tanh`](https://bbs.kanxue.com/elink@1e9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2N6r3S2%4L8%4u0D9k6q4)9J5k6i4N6G2L8r3k6J5j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6t1P5i4m8W2M7X3u0G2L8r3W2U0g2r3q4F1k6$3g2F1N6q4)9J5k6h3S2@1L8h3H3`.)－呃...有点难
- 指数运算([`Exponential`](https://bbs.kanxue.com/elink@7c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4P5s2m8G2L8X3g2F1N6r3W2S2L8q4)9#2k6X3k6#2L8X3y4@1K9h3!0F1))－呃...有点难

看起来似乎我们可以十分容易地得到除法和减法，但是这些更复杂的函数...好吧...比起简单的加法和乘法要复杂得多。为了同态加密一个深度神经网络，我们还需要一个秘密武器。

## Part 4：泰勒级数展开

或许你可从小学的记忆中找到它。泰勒级数([Taylor Series](https://bbs.kanxue.com/elink@bd9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6f1j5i4W2D9L8%4u0Q4y4h3k6K6k6i4u0A6k6i4x3`.))允许人们利用无穷级数的加、减、乘、除来计算一个复杂的（非线性）函数。这很完美！（除了无穷那部分）。幸运的是，如果你只计算了精确的泰勒级数展开的前面有限项，你仍可以得到该函数的一个精度相当高的结果。这里有一些常用的借助泰勒级数近似的函数([Source](https://bbs.kanxue.com/elink@e61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2&6M7r3g2J5M7r3S2&6M7$3W2U0M7#2)9J5k6i4m8Z5P5g2)9J5k6r3q4K6N6s2u0Q4x3X3g2Y4M7%4g2Q4x3X3g2W2k6s2g2Q4x3V1k6Z5j5X3q4K6k6g2)9J5c8Y4c8S2P5i4y4W2M7W2)9J5k6h3S2@1L8h3H3`.))。

等一下！这里有指数运算！别担心。指数运算只不过是重复的乘法，我们可以计算。玩一下吧，这里有个利用泰勒级数近似计算`sigmoid`函数的python实现代码。（你可以在网站[Wolfram Alpha](https://bbs.kanxue.com/elink@83fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2N6r3S2%4L8%4u0D9k6q4)9J5k6i4N6G2L8r3k6J5j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9h3N6E0L8$3W2V1c8Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2Z5N6r3#2D9)中查看公式）。我们会计算此泰勒级数的前几项并看看得到的结果和真正`sigmoid`函数相差多少。

```python
import numpy as np



def sigmoid_exact(x):

  return 1 / (1 + np.exp(-x))



# using taylor series

def sigmoid_approximation(x):

  return (1 / 2) + (x / 4) - (x**3 / 48) + (x**5 / 480)



for lil_number in [0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9,1.0]:

  

  print("\nInput:" + str(lil_number))

  print("Exact Sigmoid:" + str(sigmoid_exact(lil_number)))

  print("Approx Sigmoid:" + str(sigmoid_approximation(lil_number)))
```

运行结果：

```text
Input:0.1

Exact Sigmoid:0.524979187479

Approx Sigmoid:0.5249791875



Input:0.2

Exact Sigmoid:0.549833997312

Approx Sigmoid:0.549834



Input:0.3

Exact Sigmoid:0.574442516812

Approx Sigmoid:0.5744425625



Input:0.4

Exact Sigmoid:0.598687660112

Approx Sigmoid:0.598688



Input:0.5

Exact Sigmoid:0.622459331202

Approx Sigmoid:0.6224609375



Input:0.6

Exact Sigmoid:0.645656306226

Approx Sigmoid:0.645662



Input:0.7

Exact Sigmoid:0.668187772168

Approx Sigmoid:0.6682043125



Input:0.8

Exact Sigmoid:0.689974481128

Approx Sigmoid:0.690016



Input:0.9

Exact Sigmoid:0.710949502625

Approx Sigmoid:0.7110426875



Input:1.0

Exact Sigmoid:0.73105857863

Approx Sigmoid:0.73125
```

仅仅计算了该泰勒级数的前四项，我们得到的结果就非常接近`sigmoid`函数了。既然我们已经有了总体策略，是时候选择一种同态加密算法了。

## Part 5：选择一种加密算法
