
# [原创]基于深度学习的恶意软件分类器（五）

## 一.前言

### 1.实验内容

参考论文：**IMCFN: Image-based Malware Classification using Fine-tuned Convolutional Neural Network Architecture** https://bbs.kanxue.com/elink@0e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3W2W2L8X3y4W2k6r3W2J5k6h3y4@1i4K6u0W2j5$3!0E0i4K6u0r3M7$3y4A6k6h3&6U0k6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6H3K9h3W2Q4x3V1k6e0x3e0x3^5z5e0p5J5z5o6j5I4z5e0x3H3y4o6M7K6y4R3`.`.)构建的恶意软件分类器，本文所用的数据集为在**基于深度学习的恶意软件分类器（一）** https://bbs.pediy.com/thread-271558.htm文中介绍过的微软公开数据集，与论文中所用数据集并不相同。

下图为分类器的构建过程，首先通过`applyColorMap`将灰度图转为彩图作为输入数据，接着用这些来**微调** `fine-tuning` 加载了预训练模型的`VGG16`模型来完成分类器的训练。此外，还通过**数据增强** `Data Augmentation, DA` 来缓解数据不足的问题，增加训练集样本多样性。

![](https://bbs.kanxue.com/upload/attach/202205/835440_ZFB7NTKRE52HFQZ.jpg)

### 2.实验环境

- **Python版本：**3.6.13
- **Pytorch版本：**1.8.1
- **CUDA版本：**11.4

## 二.数据处理

下图显示了将一张恶意软件可视化为灰度图后，resize成（224*224）大小后在应用`applyColor`转换为彩图的结果，从转换结果可以看出转换的彩图依然具有清晰的图像纹理。

![](https://bbs.kanxue.com/upload/attach/202204/835440_A9HWQBZGFUN9A45.jpg)

在深度学习的任务中，都会希望样本数量可以足够多，且类型丰富。这样的数据集训练出来的模型具有更好的泛化能力，分类器的性能将会更高。但是，实际训练过程中，很有可能无法得到满意的样本数量，因此，需要通过数据增强来丰富样本数。数据增强的方法有很多，本文选用了其中3种，分别是垂直翻转，水平翻转和随机旋转一定角度(-45度到45度)，下面的图图分别对应三种数据增强方法后生成的图片。

![](https://bbs.kanxue.com/upload/attach/202204/835440_C28ES88EUNWXUWK.jpg)

![](https://bbs.kanxue.com/upload/attach/202204/835440_2YKD6XPCBYEJY7C.jpg)

通过数据增强，此时拥有了类别更丰富，数量更多的训练样本来构建分类器，此时的数据类代码如下：

```
import os

from torch.utils.data import Dataset

import glob

import pandas as pd

import torch

from torchvision import transforms

import cv2

from torchvision.transforms import InterpolationMode

from PIL import Image





class MalwareDataset(Dataset):

    def __init__(self, file_path, is_train):

        self.is_train = is_train

        self.file_path = glob.glob(os.path.join(file_path, "*.png"))

        self.len = len(self.file_path)



        self.transforms_data = transforms.Compose([transforms.ToTensor()])



        # 判断是否是获取训练数据集标志

        if is_train:

            train_label_path = os.path.join(file_path, "..", "trainLabels.csv")

            df = pd.read_csv(train_label_path)

            self.y_data = get_train_label(self.file_path, df)

            self.y_data = torch.Tensor(self.y_data)



            # 随机选择三种数据增强方法中的一种

            transforms_choice = transforms.RandomChoice([transforms.RandomRotation(degrees=45,

                                                                                   interpolation=InterpolationMode.NEAREST,

                                                                                   expand=True),

                                                         transforms.RandomHorizontalFlip(p=0.4),

                                                         transforms.RandomVerticalFlip(p=0.4)])

            # 数据增强以后在缩放到(224, 224)后转成tensor

            self.transforms_data = transforms.Compose([transforms_choice,

                                                       transforms.Resize((224, 224), interpolation=InterpolationMode.NEAREST),

                                                       transforms.ToTensor()])



    def __getitem__(self, index):

        image = cv2.imread(self.file_path[index])

        image = cv2.resize(image, (224, 224))

        image = cv2.applyColorMap(image, cv2.COLORMAP_RAINBOW)

        image = Image.fromarray(image)

        image = self.transforms_data(image)



        if self.is_train:

            return image, self.y_data[index]

        else:

            file_name = get_file_name(self.file_path[index])

            return image, file_name



    def __len__(self):

        return self.len





# 根据文件路径得出不带后缀的文件名

def get_file_name(file_path):

    file_name_begin = file_path.rfind("/") + 1

    file_name_end = file_path.rfind(".")

    return file_path[file_name_begin:file_name_end]





# 从trainLabels.csv中获得文件名对应的类别

def get_train_label(file_path, df):

    train_label = []

    for fp in file_path:

        file_name = get_file_name(fp)

        train_label.append(df[df["Id"] == file_name]["Class"].astype(int).values[0] - 1)

    return train_label
```

## 三.模型

### 1.卷积神经网络

**卷积神经网络** `Convolutional Neural NetWorks, CNN` 是一类包含卷积计算且具有深度结构的前馈神经网络，具有良好的表征学习能力。卷积层和池化层是卷积神经网络的关键内容，这两个层都是通过滑动窗口实现的。

下图是一个卷积层的实例，此时输入的图像是一个5*5大小的矩阵，卷积核的大小为3 * 3，此时滑动窗口的大小就是3 * 3，滑动窗口的步长为1，该滑动窗口会向下或向右移动，对应的窗口中的数值与卷积核的数值进行卷积运算(对应相乘后相加)作为输出结果。

以下图为例，第一个输出结果就是输入图像深色部分核卷积核中的每一个对应数字相乘得到的结果，也就是0 * 0 + 1 * 1 + 2 * 2 + 5 * 1 + 6 * 2 + 7 * 0 + 0 * 2 + 1 * 1 + 0 * 0 =23。第一个结果计算完成后，滑动窗口会向右移动一个步幅，第二个输出结果的计算就是1 * 0 + 2 * 1 + 3 * 2 + 6 * 1 + 7 * 2+ 8 * 0 +1 * 2 + 2 * 1 + 3 * 0 = 32。以此类推，最终输入图像经过运算以后会得到最终的输出结果。

![](https://bbs.kanxue.com/upload/attach/202204/GENMD6683HQ5ZWY.jpg)

卷积层的输出结果会在经过池化层处理，如下图所示，池化层分别平均池化层和最大池化层两种。和卷积层一样，池化层也是通过滑动窗口来实现的，图中对应的滑动窗口大小为2*2，每次也是移动一个步长。池化层的计算很简单，最大池化层就是对窗口中的数值取最大值作为输出，平均池化层就是对窗口中的数值求平均数后最为输出。通过池化层可以减少输入数据，从而减少特征的尺寸以便更快地计算，同时池化层可以保留最重要的信息。

![](https://bbs.kanxue.com/upload/attach/202205/835440_99BJY3CXGWGN929.jpg)

通常，一个卷积神经网络架构具有一组或多组卷积和池化层，最后通过一组全连接层输出结果。在卷积层和池化层的支持下，`CNN`可以进行局部的特征学习，专注于输入数据的局部部分，抽取出图像中的有效特征。全连接层就会接收到这些高级别的特征作为输入，因此能够准确地处理复杂数据，在图像识别和分类方面有良好的性能。

### 2.VGG16

下图是`VGG16`的网络架构，该网络共有5个Block，包含13个卷积层和5个池化层来抽取图像特征，最终通过全连接层来输出分类结果。最后的一层全连接层为任务的类别数，这里的1000是ImageNet数据集的类别数，需要修改为本文的分类任务数。

![](https://bbs.kanxue.com/upload/attach/202205/835440_RUP5X74PB7DP4Z9.jpg)

### 3.微调模型

`CNN`的卷积层和池化层提取的特征一般都是比较通用的特征，一个图像分类任务训练得到的参数可以迁移到其他任务中去使用。而后面的全连接层是用来输出分类结果，这些层的参数往往因为任务的不同有较大差别。因此，可以将模型在其他任务中训练得到的参数迁移到本任务中，通过冻结前面的一些卷积层，仅调整后面的全连接层的参数来微调模型构建分类器。此时，需要训练的参数量就会减少，模型训练速度会有所加快。

本文的分类器是通过使用`VGG16`模型在`ImageNet`数据集预训练得到的参数，进行微调得到的，模型代码依然如下：
