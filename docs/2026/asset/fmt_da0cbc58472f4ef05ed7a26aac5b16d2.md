
# [原创]由沙箱说去——2013的总结

走着走着，就踩到了13的尾巴，听见了14的召唤。

也来总结一下，拂去过往的尘埃，2014一切清零，脚踏实地的推进。

与大伙共勉。

---

之前的文档，稍加修改，涵盖内容较多，但都比较空，再多的文字也显得苍白无力。。。。。。

**Make SandBox Be Part of Security Solution**

One Target:

- To Know More about Applications

Two Things:

- Monitor its behavior
- Verify its Suspicious Action

---

目录：

```
0. 调研        2
0.1由SandBoxie+BSA切入        3
0.2 SandBox as Part of Security Solution        3
0.3 Malware Analysi        3
0.4 案例——Toward Automated Dynamic Malware Behaviors Using CWSandbox        3
1. 概述        4
2. 整体框架        4
2.1现有框架        4
2.2框架改进        5
2.3 案例参考——CWSandbox        7
3. 基础模块        7
3.1 DLL Implementation        7
3.2 Inject        9
3.3 Hook        11
3.4 API        13
3.5 Log Module——规范、统一        13
3.6 通信机制        14
3.7 PE        20
4. 可疑行为检测        23
4.1分割——功能专注        24
4.2 配置文件——灵活加载        24
4.3 可疑行为收集——原材料        24
4.4可疑行为分析        27
5. SandBox样本检测        27
6. 技术兴奋点        28
6.1 Inject,Hook——永恒的主题        28
6.2 Windows——Forensic Research        28
6.3 Secure Programming        29
6.4 Windows Data Type        29
6.5规范、设计——软实力        29
6.6环境、工具        29
6.7 来源        30
7.调研        31
7.1由SandBoxie+BSA切入        31
7.2 SandBox as Part of Security Solution        31
7.3 Malware Analysis        35
7.4 案例——Toward Automated Dynamic Malware Analysis Using CWSandbox        37
```

---

## 摘选

### 7.2 SandBox as Part of Security Solution

上面更多是以单软件、服务、产品的形式出现的，这里我们来关注一下SandBox 与安全厂商的关系。

#### 7.2.1 Trend Micro

Trend Micro Debuts Malware SandBox For Security Managers

[282K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2F1k6X3!0J5L8h3q4@1K9h3!0F1N6$3g2W2K9#2)9J5k6h3y4G2L8g2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0r3L8h3q4F1j5h3N6W2L8h3g2F1N6q4)9J5c8Y4c8J5k6h3&6V1i4K6u0V1L8h3W2U0M7X3!0Q4x3X3c8V1k6h3u0#2N6s2y4Q4x3X3c8E0j5h3I4%4j5i4u0W2i4K6u0V1M7$3q4F1k6r3u0G2P5q4)9J5k6r3k6G2M7W2)9J5k6s2y4Q4x3V1j5J5x3K6l9$3x3o6l9I4x3U0j5.](<https://bbs.kanxue.com/elink@ddfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2F1k6X3!0J5L8h3q4@1K9h3!0F1N6$3g2W2K9#2)9J5k6h3y4G2L8g2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0r3L8h3q4F1j5h3N6W2L8h3g2F1N6q4)9J5c8Y4c8J5k6h3&6V1i4K6u0V1L8h3W2U0M7X3!0Q4x3X3c8V1k6h3u0#2N6s2y4Q4x3X3c8E0j5h3I4%4j5i4u0W2i4K6u0V1M7$3q4F1k6r3u0G2P5q4)9J5k6r3k6G2M7W2)9J5k6s2y4Q4x3V1j5J5x3K6l9$3x3o6l9I4x3U0j5>).

“Antivirus vendors have long used sandboxes—benign computing environments—as a way to study captured malware and write signatures, which they push out to their customers’ antivirus scanning engines, to arrest further copies of the malware. But with their own malware sandbox, security managers could craft bespoke virtual patches to block emerging malware without waiting for a signature update.”

变集中式的Signature Producing, Update 为分布式的,可定制的，不依赖Signatue及时反应的机制。

“To make a sandbox available in an enterprise usable form is a great improvement over existing offerings.”

“Using sandboxes enables security-conscious organizations to more rapidly address outbreaks.”

“watching not just for incoming malware, but also outbreaks, is essential because no antivirus software reliably detects all malware.:

#### 7.2.2 Paloalto WildFire

Keywords:

WildFire  Site:paloaltonetworks.com

WildFire: Protection from Targeted and Unknown Malware

[7afK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8S2L8r3!0S2L8s2c8G2L8X3g2@1N6$3!0J5K9%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3I4#2N6r3W2G2L8Y4y4Q4x3V1k6%4K9h3I4V1k6X3W2J5k6g2)9J5k6h3S2@1L8h3H3.](<https://bbs.kanxue.com/elink@695K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8S2L8r3!0S2L8s2c8G2L8X3g2@1N6$3!0J5K9%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3I4#2N6r3W2G2L8Y4y4Q4x3V1k6%4K9h3I4V1k6X3W2J5k6g2)9J5k6h3S2@1L8h3H3>).

Controlling Modern Malware

Introducing WildFire: the Next-Generation of Malware Defense

[de2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2W2k6r3W2S2i4K6u0W2M7r3q4D9L8$3q4D9N6r3!0F1k6i4c8%4L8%4u0C8M7#2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1M7#2)9J5c8W2N6A6L8r3c8r3K9i4u0W2i4K6g2X3g2#2m8Q4x3X3g2H3k6r3j5.](<https://bbs.kanxue.com/elink@a0eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2W2k6r3W2S2i4K6u0W2M7r3q4D9L8$3q4D9N6r3!0F1k6i4c8%4L8%4u0C8M7#2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1M7#2)9J5c8W2N6A6L8r3c8r3K9i4u0W2i4K6g2X3g2#2m8Q4x3X3g2H3k6r3j5>).

Fully Integrated Threat Prevention

It only represents part of the threat prevention puzzle. The control of unknown malware must be integrated into an overall security strategy that incorporates application controls, known exploits and malware dangerous URLs and websites, and even dangerous file types or restricted content.

Integrated Threat Prevention

[b4aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2W2k6r3W2S2i4K6u0W2M7r3q4D9L8$3q4D9N6r3!0F1k6i4c8%4L8%4u0C8M7#2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1M7#2)9J5c8W2c8Z5M7X3g2S2N6q4)9#2k6W2m8J5k6i4k6W2L8Y4c8A6L8$3&6Q4y4h3k6V1M7#2)9J5k6i4m8V1k6R3.](<https://bbs.kanxue.com/elink@a9bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2W2k6r3W2S2i4K6u0W2M7r3q4D9L8$3q4D9N6r3!0F1k6i4c8%4L8%4u0C8M7#2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1M7#2)9J5c8W2c8Z5M7X3g2S2N6q4)9#2k6W2m8J5k6i4k6W2L8Y4c8A6L8$3&6Q4y4h3k6V1M7#2)9J5k6i4m8V1k6R3>).

---

直接上附件吧

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [由沙箱说去——2013总结.doc](javascript:void(0);)

  （0.99MB，93次下载）
