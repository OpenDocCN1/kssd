
![](https://bbs.kanxue.com/upload/attach/202002/805111_9V286T9ZZFCR7SF.png)

## Traditional Obfuscation Detection

# 传统混淆检测

Traditional obfuscation detection can be split into three approaches. One approach is to write a large number of complex regular expressions to match the most commonly abused syntax of the Windows command line. Figure 3 shows one such regular expression that attempts to match ampersand chaining with a call command, a common pattern seen in obfuscation. Figure 4 shows an example command sequence this regex is designed to detect.

传统混淆检测可以分为三类。第一是写许多复杂的正则表达式去匹配Windows命令行中最常被滥用的语法。图三是一个正则表达式样例，尝试匹配一个call命令的&链，这是混淆的一种常用模式。图四展示一个这条正则表达式负责检测的命令样本。

![](https://bbs.pediy.com/upload/tmp/805111_4884Y9PEQJU9M4Q.png)

![](https://bbs.kanxue.com/upload/attach/202002/805111_P8ACSBUJBGZ7NUG.png)

There are two problems with this approach. First, it is virtually impossible to develop regular expressions to cover every possible abuse of the command line. The flexibility of the command line results in a non-regular language, which is feasible yet impractical to express using regular expressions. A second issue with this approach is that even if a regular expression exists for the technique a malicious sample is using, a determined attacker can make minor modifications to avoid the regular expression. Figure 5 shows a minor modification to the sequence in Figure 4, which avoids the regex detection.

这个方法有两个问题。第一，基本不可能开发出匹配所有命令行滥用的正则表达式。命令行的灵活性使其就像一个[非正则的语言](https://bbs.kanxue.com/elink@26dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3z5o6V1^5y4o6R3&6i4K6u0r3N6$3S2S2N6q4)9J5k6s2m8J5L8$3N6J5j5h3#2E0K9h3&6Y4i4K6u0V1L8r3q4F1k6%4g2S2k6$3g2K6i4K6u0V1j5i4u0W2i4K6u0V1j5$3!0F1N6r3g2^5N6q4)9J5k6r3k6J5k6h3f1`.)，它的灵活性使得使用一个正则表达式表示存在恶意样本使用的技术不现实，这个方法的另一个问题就是，即使一个正则表达式适用于恶意软件样本，攻击者只需做很小的修改就可以绕过正则表达式。图五显示了对图四的一个小修改，使其避免被正则表达式检测。

![](https://bbs.kanxue.com/upload/attach/202002/805111_YUGTTNSPUGSWJ8Z.png)

The second approach, which is closer to an ML approach, involves writing complex if-then rules. However, these rules are hard to derive, are complex to verify, and pose a significant maintenance burden as authors evolve to escape detection by such rules. Figure 6 shows one such if-then rule.

第二个方法，与机器学习有点像，采用复杂的`if-then`规则。然而，这些规则很难发现，很难验证，并且随着恶意作者渐渐规避那些规则的检测将代码很大的维护开销。图6展示了一个`if-then`规则。

![](https://bbs.kanxue.com/upload/attach/202002/805111_3APT7JGVA9GPC89.png)

A third approach is to combine regular expressions and if-then rules. This greatly complicates the development and maintenance burden, and still suffers from the same weaknesses that make the first two approaches fragile. Figure 7 shows an example of an if-then rule with regular expressions. Clearly, it is easy to appreciate how burdensome it is to generate, test, maintain and determine the efficacy of such rules.

第三个方法结合正则表达式和`if-then`规则。它使得开发和维护成本很大，并也面临前两个方法相同的缺陷。图七展示一个`if-then`结合正则表达式的样例。清楚地看到，生成、测试、维护和判断这个样例的有效性需要大量的开销。

![](https://bbs.kanxue.com/upload/attach/202002/805111_YAZGZXWBNZ7MFRY.png)

## The ML Approach – Moving Beyond Pattern Matching and Rules

## 机器学习方法-超越模式匹配和规则

Using ML simplifies the solution to these problems. We will illustrate two ML approaches: a feature-based approach and a feature-less end-to-end approach.

使用机器学习简化了对这些问题的解决。我们也举两个机器学习方法的例子：一个基于特征的方法和一个非特征端到端的方法。

There are some ML techniques that can work with any kind of raw data (provided it is numeric), and neural networks are a prime example. Most other ML algorithms require the modeler to extract pertinent information, called features, from raw data before they are fed into the algorithm. Some examples of this latter type are tree-based algorithms, which we will also look at in this blog (we described the structure and uses of Tree-Based algorithms in a previous blog post, where we used a Gradient-Boosted Tree-Based Model).

有些机器学习技术可以应用于任何类型的源数据（提供数字化后的），神经网络是主要的一个。大多数其它机器学习算法需要建模者从源数据提取出相关信息，称作特征，才能提交给算法。这一类例如基于树的算法，我们也要看看这种算法（在之前的博客中我们讨论过[基于树的算法的结构和使用](https://bbs.kanxue.com/elink@823K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3K9i4u0W2k6i4W2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9$3i4K6u0r3j5Y4g2A6L8r3c8Q4x3X3c8E0j5h3y4Z5K9h3&6W2i4K6u0V1L8r3g2S2M7X3&6A6L8X3N6Q4x3X3c8E0L8$3c8W2L8s2y4Q4x3X3c8X3L8%4u0Q4x3X3c8@1K9r3g2Q4x3X3c8K6L8$3y4Q4x3X3g2Z5N6r3#2D9)，那里我们使用了一个基于树的梯度提升模型）。

## ML Basics – Neural Networks

## 机器学习基础-神经网络

Neural networks are a type of ML algorithm that have recently become very popular and consist of a series of elements called neurons. A neuron is essentially an element that takes a set of inputs, computes a weighted sum of these inputs, and then feeds the sum into a non-linear function. It has been shown that a relatively shallow network of neurons can approximate any continuous mapping between input and output. The specific type of neural network we used for this research is what is called a Convolutional Neural Network (`CNN`), which was developed primarily for computer vision applications, but has also found success in other domains including natural language processing. One of the main benefits of a neural network is that it can be trained without having to manually engineer features.

神经网络是一种最近非常流行的机器学习算法，它由许多称为神经元的元素组成。一个神经元是一个元素，它获取一个输入集合，计算这些输入的一个加权和，然后将和提供给一个非线性函数。现已证实，一个层数较浅的神经网络就可以逼近输入与输出之间的任意连续映射关系。我们在本研究中使用的神经网络类型是卷积神经网络（`CNN`），它最初被开发以应用于图像应用，但也被成功应用于许多其它领域，如自然语言识别。使用神经网络的一个优势是无需人工构造特征就可以进行训练。

##

##

## Featureless ML

## 非基于特征数据的机器学习

While neural networks can be used with feature data, one of the attractions of this approach is that it can work with raw data (converted into numeric form) without doing any feature design or extraction. The first step in the model is converting text data into numeric form. We used a character-based encoding where each character type was encoded by a real valued number. The value was automatically derived during training and conveys semantic information about the relationships between characters as they apply to `cmd.exe` syntax.

可以使用神经操作特征数据，但这种方法的真正吸引在于它可以[应用于源数据](https://bbs.kanxue.com/elink@74cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6K9%4y4A6L8X3y4Q4x3X3g2G2L8X3I4A6L8X3g2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3e0u0Q4x3V1j5I4z5q4)9J5c8X3k6W2j5i4c8#2M7X3g2Q4x3X3c8W2L8X3N6A6L8X3g2W2M7X3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3c8W2k6i4m8Q4x3X3c8D9k6h3q4J5L8X3W2F1k6#2)9J5c8R3`.`.)（转换为数字形式的）而无需进行特征设计或提取工作。第一步是将文本数据转换成数字形式。我们使用了一个字符编码，它将每一个字符类型编程成一个实数。这些值在训练过程中自动推理，并将字符间关系的语义信息转换成`cmd.exe`的语法。

#

## Feature-Based ML

# 基于特征数据的机器学习

We also experimented with hand-engineered features and a Gradient Boosted Decision Tree algorithm. The features developed for this model were largely statistical in nature – derived from the presence and frequency of character sets and keywords. For example, the presence of dozens of ‘%’ characters or long, contiguous strings might contribute to detecting potential obfuscation. While any single feature will not perfectly separate the two classes, a combination of features as present in a tree-based model can learn flexible patterns in the data. The expectation is that those patterns are robust and can generalize to future obfuscation variants.

我们也针对人工提权的特征和梯度提升决策树进行了测试。为这个模型开发的特征来源于对现实的大量统计-产生于字符集和关键字的存在及其频率。例如，存在大量或很长一段‘%’字符，附加的字符串将被用于检测潜在的混淆。当单个特征无法很好地区分两类结果是，基于树的模型中的多个特征的结合可以很好地学习数据中模式的灵活性。我们期望这些模型稳定并能包含未来混淆的变化。

## Data and Experiments

## 数据和实验
