
# [原创]恶意代码处理流程简介

**这篇文章根据个人的所知、经验推理而来，说的不对的地方欢迎指正。**

![](https://bbs.kanxue.com/upload/attach/201504/357400_qjkguqwggk974lq.jpg)

## 样本来源

### 厂商交换：

这个很好理解，比如McAfee和Symantec相互交换样本，VirusTotal给各杀毒厂商提供样本。

### 客户端上传：

比如360的“云安全计划”，360客户端会将可疑的文件上传并分析。

### 用户提交：

用户发现可疑文件，主动提交给安全厂商。

### 蜜罐诱捕：

安全厂商布置许多联网的、有漏洞的设备等待被感染病毒，然后捕获。

### 诱饵信箱：

厂商将邮箱散布到互联网上，被黑客收集后，邮箱就会收到恶意文件。

### 爬虫下载：

用爬虫从互联网上下载可疑文件。

### 网络流量重组还原：

这个比较高大上，监控网络流量，并把可疑的数据包重组还原成文件。

比如把所有经过动态域名“3322.org”传输的PE文件还原出来作为样本。

## 虚拟机自动化分析、信息采集

这个就是把收集到的样本投放到虚拟机中运行并生成报告，然后将这些样本信息放入“样本信息库”。

整个过程是自动化的。这种系统对外公开的有：VirusTotal、VirScan、金山火眼、腾讯哈勃、瀚海原文件B超等等，很多。

还有一个开源的“c62K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4#2j5$3E0G2L8%4y4S2L8X3c8T1L8%4S2Q4x3X3g2G2M7X3N6Q4c8e0u0Q4z5o6m8Q4z5f1c8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6c8Q4z5f1k6Q4c8e0g2Q4z5o6g2Q4b7U0c8Q4c8e0S2Q4b7U0k6Q4b7e0y4Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0N6Q4z5f1y4Q4z5p5u0Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0g2Q4b7e0g2Q4z5e0N6Q4c8e0N6Q4b7U0y4Q4b7V1u0Q4c8e0N6Q4b7V1u0Q4z5f1k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4b7U0q4Q4z5e0g2Q4c8e0N6Q4b7e0c8Q4b7V1q4Q4c8e0N6Q4b7V1c8Q4z5e0q4Q4c8e0N6Q4b7f1u0Q4z5e0W2Q4c8e0u0Q4z5o6m8Q4z5f1y4Z5N6s2c8H3M7#2)9K6b7g2)9J5c8W2)9J5c8X3#2S2L8s2N6J5i4K6u0W2j5$3!0E0i4@1f1J5i4K6R3H3i4K6W2p5i4@1f1K6i4K6R3H3i4K6R3J5”

![](https://bbs.kanxue.com/upload/attach/201504/357400_r00nmovgor1k1yr.png)

## 人工分析

依靠病毒分析工程师的经验与技术，从海量的未知样本发现可疑文件并分析之。

如果是恶意代码，就更新病毒特征库，使杀毒软件能够查杀此种病毒。

在与恶意代码的对抗中，根据新的威胁完善杀毒引擎。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [图片1.jpg](javascript:void(0);)（79.11kb，14次下载）
- [图片2.png](javascript:void(0);)（36.02kb，8次下载）
