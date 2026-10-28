
`*` 同上，`vhash`由于取值只有0和1，所以

`*` 133个检出样本中，`vhash`检出5个，未检出128个

`*` 267个未检出样本中，`tlsh`检出0个，未检出267个

通过人工分析异常点对应的样本，`mmdthash`阈值取0.95，`ssdeep`阈值取0.8，`tlsh`阈值取0.8，得到统计数据如下：

|算法名称|检出数量|未检出数量|TP|TN|FP|FN|准确率ACC|召回率REC|精确率PRE|
|----|----|----|----|----|----|----|----|----|----|
|mmdthash|133|267|132|200|0|68|0.83|0.66|1.0|
|ssdeep|114|286|114|200|0|86|0.785|0.57|1.0|
|tlsh|153|247|145|192|8|55|0.8425|0.725|0.948|
|vhash|5|395|5|200|0|195|0.5125|0.025|1.0|

如图所示：

![](upload/attach/202201/467421_KSNY729DPW5XXDR.jpg)

在上述`mmdthash`阈值取0.95，`ssdeep`阈值取0.8，`tlsh`阈值取0.8的前提下，可得出如下结论：

`*` `tlsh`的准确率、召回率、误报率最高（`误报率 = 1.0 - 准确率`）

`*` `mmdthash`的准确率和召回率次之，准确率比`tlsh`低1.5%，召回率比`tlsh`低9.0%，误报率比`tlsh`低5.5%

`*` `ssdeep`的准确率和召回率再次之，准确率比`tlsh`低6.8%，召回率比`tlsh`低21.4%，误报率比`tlsh`低5.5%

`*` `vhash`情况最特殊，数据做观察对比即可

综上，基于本篇的测试结果，敏感哈希效果的综合考虑如下：

`tlsh` > `mmdthash` > `ssdeep` > `vhash`

## 其他

### 1. 400个测试文件类型分布

`*` PE文件占比96%

`*` ELF文件占比2%

`*` 其他文件占比2%

![](upload/attach/202201/467421_KJDQZAERHVPBQT4.jpg)

### 2. 400个测试文件计算耗时

为体现差异，特意在低性能的树莓派上执行，耗时如下：

|算法名称|real|user|sys|
|----|----|----|----|
|mmdthash|74.259s|69.772s|4.028s|
|ssdeep|18.095s|15.485s|2.572s|
|tlsh|17.751s|14.12s|3.558s|

![](upload/attach/202201/467421_HK2JRECYUFZTE9U.jpg)

## 反思与收获

1. `ssdeep`的论文写的真不错，通俗易懂，论证过程清晰，数学理论支撑非常充足，非常棒。

2. `tlsh`的生态做的真的好，已经出了6篇相关的paper，参与了6次会议，足以证明其应用效果。

3. 在实现`mmdthash`的过程中，过于闭门造车，有很多不成熟的想法、思路在`tlsh`相关的paper里都讲述的很清楚、很详细，应用也非常成熟了。

4. `mmdthash`的性能还要持续优化

## 前文回顾

- [python_mmdt:一种基于敏感哈希生成特征向量的python库(一)](https://bbs.pediy.com/thread-265211.htm)我们介绍了一种叫`mmdthash`（敏感哈希）生成方法，并对其中的概念做了基本介绍。

- [python_mmdt:从0到1--实现简单恶意代码分类器(二)](https://bbs.pediy.com/thread-265499.htm)我们介绍了基于`mmdthash`的一种简单恶意代码分类器应用。

- [python_mmdt:从1到2--实现基于KNN的机器学习恶意代码分类器(三)](https://bbs.pediy.com/thread-265860.htm)我们介绍基于`mmdthash`的机器学习恶意代码分类器应用。

- [python_mmdt:mmdthash的在线使用(四)](https://bbs.pediy.com/thread-271243.htm)我们介绍如何使用`mmdthash`实现在线恶意文件检测。

- [python_mmdt:KNN机器学习分类结果测试分析(五)](https://bbs.pediy.com/thread-271265.htm)我们对基于KNN机器学习算法的分类结果进行统计测试，评估分类模型。

- 本篇，我们对`ssdeep`、`tlsh`、`vhash`、`mmdthash`这四类敏感哈希算法效果进行对比

## 项目地址

- github代码地址：[python_mmdt](https://bbs.kanxue.com/elink@9adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2x3U0x3J5x3K6p5&6y4K6M7&6i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6l9`.`.)

## 对比结论

- 准确率ACC：`tlsh` > `mmdthash` > `ssdeep` > `vhash`

- 召回率REC：`tlsh` > `mmdthash` > `ssdeep` > `vhash`

- 精确率PRE：`mmdthash` = `ssdeep` = `vhash` > `tlsh`

![](https://bbs.kanxue.com/upload/attach/202201/467421_SKTNHRPB3W9BCDP.jpg)

基于本篇的测试结果，在`mmdthash`阈值取0.95，`ssdeep`阈值取0.8，`tlsh`阈值取0.8的前提下，敏感哈希效果的综合排序如下：

`tlsh` > `mmdthash` > `ssdeep` > `vhash`

## 敏感哈希介绍

四类敏感哈希一览：

- [CTPH(ssdeep)](https://bbs.kanxue.com/elink@1dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7$3c8W2k6i4m8Q4x3X3c8H3M7X3!0B7k6h3y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6K6M7$3c8W2k6i4m8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.)：Context Triggered Piecewise Hashes(CTPH)，又叫模糊哈希，最早由Jesse Kornblum博士在2006年提出，论文地址点击[这里](https://bbs.kanxue.com/elink@25aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7$3c8W2k6i4m8Q4x3X3c8H3M7X3!0B7k6h3y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6K6M7$3c8W2k6i4m8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.)。CTPH可用于文件/数据的**同源性判定**。据官方文档介绍，其计算速度是`tlsh`的两倍（测试了一下，好像并没有）。

- [tlsh](https://bbs.kanxue.com/elink@f69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8s2y4Z5i4K6u0W2L8%4u0Y4i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9)：是趋势科技开源的一款模糊哈希计算工具，将50字节以上的数据计算生成一个哈希值，通过计算哈希值之间的相似度，从而得到原始文件之间的同源性关联。据官方文档介绍，`tlsh`比`ssdeep`和`sdhash`等其他模糊哈希算法更难攻击和绕过。

- [vhash](https://bbs.kanxue.com/elink@622K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2N6X3W2J5N6i4y4@1L8%4c8S2L8q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3k6A6L8r3g2K6)：（翻遍了整个virustotal的文档，就找到这么一句话）“an in-house similarity clustering algorithm value, based on a simple structural feature hash allows you to find similar files”，大概就是说是个内部相似性聚类算法，允许你通过这个简单的值，找到相似的样本。

- [mmdthash](https://bbs.kanxue.com/elink@a66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2x3U0x3J5x3K6p5&6y4K6M7&6i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6l9`.`.)：是开源的一款模糊哈希计算工具，将任意数据计算生成一个模糊哈希值，通过计算模糊哈希值之间的相似度，从而判断两个数据之间的关联性。详情前文1-5篇。

## 对比思路

基于[python_mmdt:KNN机器学习分类结果测试分析(五)](https://bbs.pediy.com/thread-271265.htm)文章中的`mmdthash`测试数据及测试结果，对`ssdeep`、`tlsh`、`vhash`进行对比测试。即计算`mmdthash`关联出的两个样本之间的`ssdeep`、`tlsh`、`vhash`相似度，并统计分析相关的**异常值**、**精确度**、**召回率**、**精确率**，从而得出敏感哈希算法之间的对比结果。

## 对比过程

说明：`ssdeep`和`tlsh`在Windows上的安装比较折腾，测试直接在树莓派的Linux环境上进行。

### 1. ssdeep计算

ssdeep安装：

1. 安装模糊哈希库：`sudo apt-get -y install libfuzzy-dev ssdeep`

2. 安装`ssdeep`的python库：`pip install ssdeep`

或者，如果linux的编译环境比较全（包含automake等工具），可以直接通过`pip`安装`ssdeep`的模糊哈希库：`BUILD_LIB=1 pip install ssdeep`

利用python的`ssdeep`库计算785个测试文件的`ssdeep`值，并使用`json`格式保存在文件中，代码如下：

```
# -*- coding: utf-8 -*-
import os
import sys
import hashlib
import json
import ssdeep
# 遍历目录
def list_dir(root_dir):
    files = os.listdir(root_dir)
    for f in files:
        file_path = os.path.join(root_dir, f)
        yield file_path
# 生成sha1
def gen_sha1(file_name):
    with open(file_name,'rb') as f:
        s = f.read()
        _s = hashlib.sha1()
        _s.update(s)
    return _s.hexdigest()
def main():
    file_path = sys.argv[1]
    ssdeep_dict = dict()
    for file_name in list_dir(file_path):
        file_sha1 = gen_sha1(file_name)
        ssdeep_hash = ssdeep.hash_from_file(file_name)
        print('%s,%s'%(file_sha1, ssdeep_hash))
        ssdeep_dict[file_sha1] = ssdeep_hash
    with open('ssdeep_test.json','w') as f:
        f.write(json.dumps(ssdeep_dict, indent=4))
if __name__ == '__main__':
    main()
```

`ssdeep`计算结果示例如下：

```
cat ssdeep_test.json
{
  "0ec279513e9e8a0e8f6e7c170b9462b60d9888c6": "6144:w9qaZ5E6fCvH5H42SUiTV2MTb54y94HTFboTWhmzeOws:w9d96yeKV2MTb5X4zZQWhmqd",
  "0ad6db9128353742b3d4c8a5fc1993ca8bf399f1": "1536:NxiIXeGNc0BL0IFx34bPMkG/KsrKlEqjjPWUJ7h/dbZkv13t43O:eIXeGNtV0KIQjr5ehlbSv13t43O",
  ...
}
```

### 2. tlsh计算

`tlsh`安装：`pip install py-tlsh`

利用python的`tlsh`库计算785个测试文件的`tlsh`值，并使用`json`格式保存在文件中，代码如下：
