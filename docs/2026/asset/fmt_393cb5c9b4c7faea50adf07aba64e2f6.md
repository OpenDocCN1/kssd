
# [翻译]利用深度学习检测恶意PowerShell脚本

作为大型机器学习框架中的一类算法，深度学习的科研和技术进步，为最新的安全防御技术的发展提供了新的机会。在图像检测和文本分类等任务上，深度学习方法的性能明显优于传统方法。随着这些技术的发展，利用深度学习构建新型的威胁检测方法有巨大的潜力。

机器学习算法处理数字，因此需要通过*特征工程*将诸如图像、文档或电子邮件之类的对象转换为数字形式，这在传统的机器学习方法中需要大量的人力。通过深度学习，算法可以处理相对原始的数据并提取特征，而无需人工干预。

在Microsoft，我们在开拓机器学习方面投入巨大，可以通过数据为我们的安全解决方案提供可落地的认知，针对各种威胁提供智能、准确和实时的防护。本文中，我们展示了一个深度学习技术的示例，该技术最初是为自然语言处理（NLP）开发的，现在已被采用并应用于扩大我们对恶意PowerShell脚本的检测范围，恶意PowerShell脚本现在仍然是很关键的攻击向量。这些基于深度学习的检测能力已添加至Microsoft Defender Advanced Threat Protection（Microsoft Defender ATP）产品中，增强了其行业领先的终端检测和响应能力。

## 自然语言处理的词嵌入

需要注意的是，我们的目标是对PowerShell脚本进行分类，我们简要介绍一下在自然语言处理领域中如何进行文本分类。重要的一步是将单词转换为机器学习算法可以理解的向量（数字元组）。“*独热编码（one-hot encoding）*”是一种基本的方法，独热编码首先为词汇表中的每个单词分配一个唯一的整数，然后将每个单词表示为向量，其中1对应于该单词的整数索引。尽管在很多情况下好用，但是独热编码存在重大缺陷。主要问题是所有单词彼此等距，并且单词之间的语义关系没有反映在相应向量之间的几何关系中。

*上下文嵌入*是一种较新的方法，它通过在经常出现在相似上下文中的单词倾向于具有相似含义的假设下，从数据中学习单词的紧凑表示形式来克服这些限制。需要使用大型文本数据集（如Wikipedia）训练嵌入。作为此技术的一种实现，Word2vec算法因将单词的语义相似度转换为向量的几何相似度，以及保留了单词之间的极性关系而闻名。例如，在Word2vec表示中：

`Madrid – Spain + Italy ≈ Rome`

## PowerShell脚本嵌入

由于训练好嵌入需要大量数据，因此我们使用了一组由38.6万个不同的未标记PowerShell脚本组成的数据集。Word2vec算法通常应用于人类语言的处理，当应用于PowerShell语言时，也会提供类似的有意义的结果。为此，我们将PowerShell脚本拆分为token，然后我们使用Word2vec算法为每个token分配向量表示。

图1为5000个随机选择的token的向量表示的二维可视图，并高亮显示了其中一些重要token。注意语义相似的token相互距离很近。比如说，表示`-eq`、`-ne`和`-gt`的向量，即在PowerShell中分别表示“相等”、“不等”和“大于”，呈聚集状。类似地，PowerShell中`ExecutionPolicy`设置的有效值`allSigned`、`remoteSigned`、`bypass`和`unrestricted`的token，也聚集在一起。

![图1. 利用Word2vec二维可视化5000个token](upload/attach/202001/747358_H7UV4K9XDMX8XQS.jpg)

*图1 利用Word2vec二维可视化5000个token*

研究这些token的向量表示后，我们发现了一些有趣的关系。

*Token相似度*：通过分析token的Word2vec表示，我们可以识别PowerShell中具有别名的命令。在许多情况中，最接近给定命令的token是其别名。例如，token `Invoke-Expression`及其别名`IEX`的表示彼此很相近。另外两个例子是`Invoke-WebRequest`及其别名`IWR`，以及`Get-ChildItem`命令及其别名`GCI`。

我们还测量了多个token集合的距离。例如，四个token `$i`、`$j`、`$k`和`$true`（见图2右侧）。前三个通常用于表示数值变量，最后一个表示布尔常量。正如所料，`$true` token与其他不相匹配——它是距离集合中心欧氏距离最远的。

更具体地说，针对PowerShell在网络安全中的语义，我们检查了token的表示形式：`bypass`、`normal`、`minimized`、`maximized`和`hidden`（参见图2左侧）。第一个token是PowerShell中`ExecutionPolicy`的合法值，而其余token是`WindowStyle`的合法值。正如预期的那样，`bypass`的向量表示距离其他四个token的向量的中心很远。

![图2. 选取token的三维可视化1](upload/attach/202001/747358_J6DART53XV5U3DQ.jpg)

![图2. 选取token的三维可视化2](upload/attach/202001/747358_KCN22A8SWJ94UYN.jpg)

*图2 选取token的三维可视化*

*线性关系*：

由于Word2vec保留了线性关系，因此计算向量表示的线性组合会产生在语义上有意义的结果。以下是我们发现的一些有趣的关系：

```
high – $false + $true ≈' low
'-eq' – $false + $true '≈ '-neq'
DownloadFile – $destfile + $str ≈' DownloadString '
Export-CSV' – $csv + $html '≈ 'ConvertTo-html'
'Get-Process' - $processes + $services '≈ 'Get-Service'
```

在上述每个表达式中，符号≈表示右侧的向量约等于（在词汇表中表示token的所有向量中）左侧向量的计算结果。

## 深度学习检测恶意PowerShell脚本

在上一节中，我们使用PowerShell语言的Word2vec嵌入来训练能够检测恶意PowerShell脚本的深度学习模型。使用标记为“无风险”或“恶意”的大量PowerShell脚本数据集对分类模型进行定型和验证，并且是用未标记的数据训练嵌入的。流程如图3所示：

![图3. 训练模型流程图](upload/attach/202001/747358_VFXCM6GDV8B7AFF.jpg)

*图3 训练模型流程图*

利用Microsoft Azure的GPU算力，我们尝试了各种深度学习和传统的机器学习模型。性能最佳的深度学习模型比传统的机器学习模型可提高22%的覆盖率（误检率控制在0.1%）。图4所示的模型结合了多个深度学习模块，如卷积神经网络（CNN）和长短期记忆循环神经网络（LSTM-RNN）。神经网络是由生物神经系统（如人脑）启发的机器学习算法。除了此处介绍的预训练嵌入之外，该模型还提供字符级的脚本嵌入。

![图4. 最佳性能模型的网络结构](upload/attach/202001/747358_2JWD4AF8QZSA74A.jpg)

*图4 最佳性能模型的网络结构*

## 利用深度学习检测恶意PowerShell脚本的实际应用

使用Microsoft ML.Net技术和ONNX格式的深度神经网络，将性能最佳的深度学习模型大规模应用到Microsoft Defender ATP通过AMSI接口捕获到的PowerShell脚本。该模型强化了Microsoft Defender ATP的机器学习模型和启发式工具套件，以及对恶意脚本的检测能力。

自从首次部署以来，该深度学习模型就可以高精度地检测到许多恶意和蓝军团队的PowerShell活动，其中有些活动是其他方法发现不了的。通过PowerShell获得的信息与各种机器学习模型以及Microsoft Defender ATP的信息结合在一起以检测网络攻击。

以下是恶意PowerShell脚本的例子，深度学习可以准确地检测到这些恶意脚本，但对于其他检测方法而言是很有挑战性的：

![图5. 高度混淆的恶意脚本](upload/attach/202001/747358_XKNZ582RQ5PD6PN.jpg)

*图5 高度混淆的恶意脚本*

![图6. 下载并运行payload的混淆脚本](upload/attach/202001/747358_VQE42GUXN4UVFDA.jpg)

*图6 下载并运行payload的混淆脚本*

![图7. 解密并执行恶意代码的脚本](upload/attach/202001/747358_QY9NU544Q9BJ2XE.jpg)

*图7 解密并执行恶意代码的脚本*

## 通过深度学习强化Microsoft Defender ATP
