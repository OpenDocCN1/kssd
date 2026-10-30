
主要针对于固定语句验证的系统，使用录音设备录制目标说话人的固定语句，在验证时通过将录制的语句进行播放，来欺骗声纹识别系统。这是一种简单有效的攻击方法，通过简单的设备采集后直接播放就能够欺骗众多不包含活体检测的声纹识别系统，而使用高保真设备的采集，播放的重放攻击甚至有一定概率欺骗带有活体检测的声纹认证系统。

### 6.3 语音转换

将语音模拟到目标人物的语音特征空间，在保持语言内容不变的同时改变说话人的身份。语音转换涉及多种语音处理技术，如语音分析、频谱转换、韵律转换、说话人特征化和语音编码等，理论和实践方面的最新进展，我们现在能够产生具有高说话人相似度接近真人的语音质量。通过采集目标人物的说话的语音数据片段，提取说话人特征，然后将特征迁移，迁移到攻击人员语音数据，使得转换攻击人员身份为目标人物，进而欺骗声纹识别系统。语音转换其实是语音合成的一个研究方向，单独列出来主要是为了展示这种独特的攻击方法。下图是一个简单的语音转换示例：

![](https://bbs.kanxue.com/upload/attach/202203/937445_VWGD8KQAR3SW65C.png)

### 6.4 语音合成

语音合成，又称文语转换 `Text To Speech, TTS`，是一种可以将任意输入文本转换成相应语音的技术。是将人类语音用人工的方式所产生，能将任意文字信息实时转化为标准流畅的语音朗读出来，相当于给机器装上了人工嘴巴。它涉及声学、语言学、数字信号处理、计算机科学等多个学科技术，是信息处理领域的一项前沿技术，解决的主要问题就是如何将文字信息转化为可听的声音信息，也即让机器像人一样开口说话。使用语音合成对声纹识别攻击应该属于语音合成一个极端的应用，首先通过大量数据进行预训练，训练出效果较好的 `TTS` 系统（可能包括 `vocoder`，`synthesizer`，`encode`，`decode`），然后对目标人物语音数据进行收集，标注，制作相关的训练集，对该系统模型进行 `fine-tuning`，最终产出一个对目标人员高度拟合的 `TTS` 系统，基于此系统进行语音合成，对声纹识别系统进行攻击，该方法对声纹识别系统欺骗能达到较好的效果。下图是语音合成的一个通用结构：

![](https://bbs.kanxue.com/upload/attach/202203/937445_6BR5QK6MCM6EKRU.png)

### 6.5 对抗样本攻击

|  |
| --- |
| 1 | `对抗样本是攻击者故意设计以导致算法模型犯错误的那些输入。对抗样本攻击在深度学习算法安全对抗领域有着广泛的应用，有一定的通用性。通过对样本添加特定的扰动信息，干扰模型输出结果，输出指向错误的结果或输出结果异常。对抗样本攻击是在一段非目标人物的语音上加上扰动信息，生成对抗语音，从而使得系统将对抗语音识别为目标人物。对抗样本攻击又分为白盒攻击、黑盒攻击和半黑盒攻击。攻击者可以访问目标神经网络的所有信息，包括其架构、参数、梯度等。攻击者可以充分利用网络信息，精心制作对抗样本。在black``-``box attack中，攻击者不知道DNN模型的内部信息。攻击者只能持续提供输入数据，观测模型的输出。在semi``-``white (gray) box attack中，攻击者训练一个生成对抗样本的模型，训练这个模型时需要目标神经网络的信息。一旦生成模型被训练成功，攻击者就不再需要目标神经网络模型的信息了。语音对抗攻击是对抗样本在语音领域的应用，对抗样本攻击是一种通用的模型安全对抗的方法，下图是展示了一些对抗样本攻击的方法：` |

![](https://bbs.kanxue.com/upload/attach/202203/937445_DCQ2M9HRD9MNV95.jpg)

### 6. 结论

|  |
| --- |
| 1 | `本篇为声纹识别漏洞研究的上篇，主要对声纹识别相关技术从定义，发展到方法，应用进行了简要描述，方便大家对声纹识别建立基础的概念，主要信息，图片基本来源于声纹识别相关的论文及技术性文章，相关参考文章见文献附录。同时对声纹识别对抗方法进行了简要描述，让大家对声纹识别对抗有一个基本概念，下篇中将结合实验对声纹识别系统进行测试，分析测试结果，讨论声纹识别中的漏洞问题。` |

### References

D Sztahó，G Szaszák，A Beke.Deep learning methods in speaker recognition: a revie

Guangke Chen, Sen Chen, Lingling Fan, Xiaoning Du, Zhe Zhao, Fu Song, Yang Liu.Who is Real Bob? Adversarial Attacks on Speaker Recognition Systems

Zhongxin Bai, Xiao-Lei Zhang. Speaker Recognition Based on Deep Learning: An Overview

Rohan Kumar Das1, Xiaohai Tian1, Tomi Kinnunen2and Haizhou Li .The Attacker’s Perspective on Automatic Speaker Verification: An Overview

Zhaoxi Mu，Xinyu Yang，Yizhuo Dong. Review of end-to-end speech synthesis technology based on deep learning

Shipra J. Arora，Rishi Pal Singh. Automatic Speech Recognition: A Review

84bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4E0j5i4u0@1K9r3!0E0k6g2)9J5k6i4q4A6j5h3&6B7K9h3q4Q4x3X3g2U0L8$3#2Q4x3V1k6Z5N6r3#2D9i4K6u0r3x3U0l9J5x3g2)9J5k6o6p5H3i4K6u0r3x3e0q4Q4y4h3j5K6z5o6b7J5y4o6S2Q4x3X3g2Z5N6r3#2D9

88dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0x3$3x3U0f1@1z5e0u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3z5e0M7I4y4e0x3^5y4H3`.

b1fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5#2y4K6p5@1y4e0k6Q4x3V1k6U0j5i4c8W2k6$3!0J5P5g2)9#2k6U0V1$3x3U0f1&6z5o6g2Q4x3X3g2Z5N6r3#2D9

af2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6l9I4y4U0R3&6y4o6W2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3^5y4o6t1@1z5o6M7^5

e26K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3@1y4o6b7H3x3o6l9H3

149K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0p5&6k6o6x3@1j5U0p5&6y4e0p5%4j5R3`.

c5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5%4y4e0j5K6x3U0M7#2i4K6y4r3K9i4k6C8i4K6g2X3M7$3q4Q4x3@1b7I4x3o6t1@1x3K6t1H3N6b7`.

c74K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6j5$3y4e0x3#2x3o6g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3#2x3o6R3J5y4K6b7$3

c9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2\*7K9r3W2Z5N6g2)9J5k6h3y4G2L8g2)9J5c8Y4q4#2k6i4y4@1K9h3!0F1i4K6u0r3y4e0x3%4x3o6M7^5x3o6W2Q4x3V1k6S2L8Y4y4%4k6i4u0Q4x3V1j5K6x3e0j5&6y4o6j5@1y4U0f1`.

4aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3A6G2K9X3!0\*7K9r3q4F1k6$3A6#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5o6j5K6y4K6p5I4z5l9`.

36fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6j5$3y4e0x3#2x3o6g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3#2x3o6R3J5y4K6b7$3

d3fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8G2j5K6R3^5i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5k6o6t1H3y4K6f1&6y4e0x3%4x3U0M7$3y4e0t1^5i4K6u0W2K9s2c8E0L8l9`.

f51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2\*7K9r3W2Z5N6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8S2M7X3c8A6M7#2)9J5c8Y4y4G2k6$3!0#2i4K6u0r3M7i4g2K6i4K6u0r3x3K6l9I4y4o6p5@1y4U0l9`.

f84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2W2K9K9r3q4F1k6K6l9I4x3o6S2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3y4e0R3$3x3U0M7@1x3H3`.

560K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2B7K9h3q4Z5j5h3!0Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3@1k6A6k6q4)9K6c8o6p5$3y4U0j5@1y4U0j5%4y4U0M7J5x3o6x3%4y4e0V1H3x3e0S2Q4x3U0k6S2L8i4m8Q4x3@1u0%4k6Y4u0Q4x3@1c8K6M7r3W2V1k6i4u0Q4x3U0k6S2L8i4m8Q4x3@1u0X3L8%4u0Q4x3@1c8H3j5H3`.

d13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9h3g2T1j5g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5@1x3U0l9&6y4e0f1@1x3U0x3`.

8bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0x3&6x3o6f1&6x3o6x3I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6j5I4z5o6p5@1x3o6V1`.

欢迎加入星球一起学习探讨

![](https://bbs.kanxue.com/upload/attach/202203/937445_MNANST4E8XZV76J.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-3-15 11:05

被星舆实验室编辑

，原因：

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#汽车安全](https://bbs.kanxue.com/forum-128-1-175.htm)
