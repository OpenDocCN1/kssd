
# [原创]基于深度学习的恶意软件分类器（二）

## 一.前言

### 1.实验内容

实验内容来源于：**Deep learning based Sequential model for malware analysis using Windows exe API Calls**。虽然这只是一篇3区的论文，但是我觉得作者做的工作挺好的。在这篇论文中，作者提出了一个方法从公网上收集8种不同类型的恶意软件，并将其API调用序列保存下来。接下来，作者将获取的不同恶意软件的API调用序列作为输入数据，通过不同的机器学习算法作为模型来构建分类器。数据集以及相关代码作者也都放在github上，链接为：**Windows Malware Dataset with PE API Calls**。链接中的代码是基于Keras框架编写了一系列的分类器，其实最终的实验是表明通过SVM算法构建的分类器性能最优。

![](https://bbs.kanxue.com/upload/attach/202204/835440_GR2B9X8UBWAGAXF.jpg)

本文基于Pytorch框架实现了基于LSTM的分类器，至于基于其他算法构建的分类器，感兴趣还是去看看作者的论文和代码。在论文中，作者还是做了很多的实验，其中也包括了二分类的实验。

### 2.实验环境

- **Python版本：**3.6.13
- **Pytorch版本：**1.8.1
- **CUDA版本：**11.4

## 二.数据集

数据集的生成过程如下图所示，首先作者从github上下载一些恶意软件，接着分别通过VirusTotal和恶意软件的哈希值来获取恶意软件的类别，通过Cuckoo来获取恶意软件的API调用序列，最终将每个恶意软件的API调用序列和它所属的类别都保存起来构成数据集。

![](https://bbs.kanxue.com/upload/attach/202204/835440_Y3AHG5H8SVUJ8ME.jpg)

不过此时的数据集还不能用，因为输入数据都是如下图所示的调用的API名称：

![](https://bbs.kanxue.com/upload/attach/202204/835440_EZQM2AT8K5GYDBN.jpg)

所以作者给每一个API都指定了一个整型数字，将每个恶意软件的API调用序列都转换成一串数字:

![](https://bbs.kanxue.com/upload/attach/202204/835440_SN7U6DSU7RUY367.jpg)

最终作者共收集了8个类别，总计7107个恶意软件，数据样本分布如下：

![](https://bbs.kanxue.com/upload/attach/202204/835440_5XAP6MWA95NGAJK.jpg)

以下是数据类的参考代码，由于会用10折交叉验证来验证分类器精度，所以数据的读取是在`get_data`函数中完成的：

```
from torch.utils.data import Dataset

import pandas as pd

import torch

from keras import preprocessing





class APIDataSet(Dataset):

    def __init__(self, x_data, y_data):

        self.x_data = x_data

        self.y_data = y_data

        self.len = len(self.y_data)



    def __getitem__(self, index):

        return self.x_data[index], self.y_data[index]



    def __len__(self):

        return self.len





# 获取API调用序列和对应类别

def get_data(calls_path, types_path, max_len):

    # 读取API序列

    calls_df = pd.read_csv(calls_path, delimiter=' ', header=None, compression="zip")



    ds_tmp = calls_df.values[:, 0].tolist()

    ds = []

    for v in ds_tmp:

        ds.append(v.split(','))

    # padding成固定长度

    x = preprocessing.sequence.pad_sequences(ds, maxlen=max_len)

    x_data = torch.tensor(x)



    # 读取标签

    types_df = pd.read_csv(types_path, delimiter=' ', header=None, compression="zip")

    y_data = types_df[0].astype('category').cat.codes.values



    return x_data, y_data
```

## 三.模型

### 1.循环神经网络

下图是一个典型的神经网络示意图，虽然神经网络已经可以拟合出很复杂的曲线了，可是同一层隐藏层中的神经元直接并没有联系，也就是说同一层隐藏层中的神经元的信息并不能传递给同一层隐藏层中另一个神经元。

![](https://bbs.kanxue.com/upload/attach/202204/835440_RP7QST4GB7DQVTZ.jpg)

可是对于一些任务来说，数据的顺序是非常重要的，如手写识别，语音识别，语音翻译和时间序列分析等。在安全领域，则是如网络流量分析，行为检测和静态文件分析这些问题相关。举一个例子说明，比如一个软件API调用序列为FindFirstFile，DeleteFile，FindNextFile，DeleteFile，FindNextFile...。如果是普通的神经网络，它很有可能认为该软件只是在删除一些文件，因为它没有考虑到调用的调用DeleteFile前后调用了什么函数。

为了更好地处理这些具有序列特征地数据，就有了循环神经网络(RNN)，下图为RNN的一个示意图，此时每一层隐藏层（绿色长方形）神经元的输出信息除了会传递给下一层神经元之外，还会传递给同一层隐藏层的下一个神经元，也就是说神经元的输出会影响到同一层的隐藏层中下一个神经元的输出，这样当神经元在处理DeleteFile的时候，会考虑到调用DeleteFile前调用的函数，就很有可能会得出，此时软件正在进行格盘操作。

![](https://bbs.kanxue.com/upload/attach/202204/835440_388Q6QB8VHYAV77.jpg)

### 2.长短期记忆网络

RNN网络中引入的每个时间步长类似于前馈神经网络中的一整个额外层，在后向传播期间，由于梯度消失将会导致前面神经元的信息传递到到后面的神经元的时候，信号变得非常微弱，因此，有了长短期记忆网络（LSTM）。

LSTM是一种特殊类型的RNN网络，示意图如下，和普通的RNN网络不同的是，该网络中包含了记忆细胞和特殊神经元，它们试图决定记住哪些信息以及忘记哪些信息。剔除大部分信息极大地限制了梯度消失问题，因为这样减少了路径爆炸。

![](https://bbs.kanxue.com/upload/attach/202204/835440_6RSJJKQAZ5V5UMY.jpg)

接下来看一下论文作者给出的用于构建分类的基于LSTM的示意图：

![](https://bbs.kanxue.com/upload/attach/202204/835440_H8MHRXTV5SMCRSZ.jpg)

模型还是简单的，经过embedding层后，紧接着就是LSTM，最后通过全连接层将结果输出，以下是参考代码：

```
import torch

import torch.nn as nn





class LSTMModel(nn.Module):

    def __init__(self,

                 input_size,

                 embedding_units,

                 hidden_size,

                 output_size,

                 dropout_prob=0.2,

                 n_layers=1,

                 bidirectional=False):

        super(LSTMModel, self).__init__()



        self.embedding = torch.nn.Embedding(num_embeddings=input_size,

                                            embedding_dim=embedding_units)



        # 单层LSTM

        self.n_directions = 2 if bidirectional else 1

        self.lstm = torch.nn.LSTM(input_size=embedding_units,

                                  hidden_size=hidden_size,

                                  num_layers=n_layers,

                                  dropout=dropout_prob,

                                  bidirectional=bidirectional,

                                  batch_first=True)

        # 全连接层输出结果

        self.fc = torch.nn.Linear(hidden_size * self.n_directions, output_size)



    def forward(self, inputs):

        embedding_output = self.embedding(inputs)



        lstm_output, (h_n, c_n) = self.lstm(embedding_output)

        if self.n_directions == 2:

            hidden_cat = torch.cat([h_n[-1], h_n[-2]], dim=1)

        else:

            hidden_cat = h_n[-1]



        fc_output = self.fc(hidden_cat)



        return fc_output
```

## 四.参数

基于作者的论文和代码，本文构建的分类器的参数信息如下：

- **优化器：**Adam
- **lr:** `0.001`
- **max_len:** `342`
- **embedding_units:** `32`
- **hidden_size:** `200`
- **dropout_prob:** `0.2`
- **epochs:** `150`
- **batch_size:** `50`

因此，使用`Configure.py`来保存这些信息：

```
import os





class Configure:

    # 数据集路径

    base_path = ""

    calls_path = os.path.join(base_path, "1000_calls.zip")

    types_path = os.path.join(base_path, "1000_types.zip")



    max_len = 342

    embedding_units = 32

    hidden_size = 200

    output_size = 8

    dropout_prob = 0.2

    n_layers = 1

    bidirectional = False

    lr = 0.001



    epochs = 150

    batch_size = 50

    folder = 10
```

用来训练和测试的代码如下：
