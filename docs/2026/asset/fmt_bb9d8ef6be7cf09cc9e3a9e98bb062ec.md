
| Lightning.Core | [fd285c2fb4d42dde23590118dba016bf5b846625da3abdbe48773530a07bcd1e](https://bbs.kanxue.com/elink@affK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3q4D9P5i4A6W2i4K6u0W2K9h3&6@1k6i4A6W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3k6A6L8r3g2K6i4K6u0r3k6X3b7J5z5o6g2U0x3X3k6T1y4r3b7@1x3X3c8V1k6e0t1K6y4e0V1H3x3e0p5^5k6r3u0S2x3o6p5$3j5X3j5#2j5U0R3@1y4U0j5J5y4h3c8S2x3$3q4T1k6r3u0W2y4o6R3%4y4K6x3#2x3K6m8S2x3o6N6T1j5$3b7I4k6b7`.`.) |

| Linux.Plugin.Lightning.Sshd | [ad16989a3ebf0b416681f8db31af098e02eabd25452f8d781383547ead395237](https://bbs.kanxue.com/elink@16dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3q4D9P5i4A6W2i4K6u0W2K9h3&6@1k6i4A6W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3k6A6L8r3g2K6i4K6u0r3j5h3b7I4y4U0V1^5z5h3p5K6k6h3u0X3x3r3t1@1x3e0j5$3z5o6q4X3z5r3c8T1x3K6q4S2k6U0l9&6z5r3f1H3x3X3g2S2j5X3b7J5y4e0b7#2x3X3j5^5k6o6M7^5x3e0x3^5x3K6f1@1y4$3g2S2k6o6x3&6y4e0t1K6y4H3`.`.) |

### Sigma 检测规则

```
title: Lightning Framework File Path
status: experimental
description: Detects creation of files related to Lightning Framework.
author: Intezer
references:
  - c9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8Y4c8W2P5X3g2J5i4K6u0W2j5$3!0E0L8r3!0Y4M7$3!0#2M7X3y4W2i4K6y4m8
  product: linux
  category: file_create
detection:
  selection1:
    TargetFilename|startswith:
      - '/usr/lib64/seahorses/'
  selection2:
    TargetFilename|contains:
      - 'kbioset'
      - 'cpc'
      - 'kkdmflush'
      - 'soss'
      - 'sshod'
      - 'nethoogs'
      - 'iftoop'
      - 'iptraof'
  condition: selection1 and selection2
falsepositives:
  - Unknown.
```

```
title: Lightning Default C2 Communication
status: experimental
description: Detects communication to default local ip for Lightning Framework
author: Intezer
references:
  - f8eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8Y4c8W2P5X3g2J5i4K6u0W2j5$3!0E0L8r3!0Y4M7$3!0#2M7X3y4W2i4K6y4m8
  category: firewall
detection:
  select_outgoing:
    dst_ip: 10.2.22.67
    dst_port: 33229
  condition: select_outgoing
falsepositives:
  - Unknown.
```

### MITRE ATT&CK

| Tactic | Technique | ID | 描述 |
| --- | --- | --- | --- |
| Persistence | Boot or Logon Initialization Scripts | [T1037](https://bbs.kanxue.com/elink@c7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9K6y4#2)9J5c8R3`.`.) | 一个`init.d`脚本被用来维持`Downloader`模块的持久性 |
| Persistence | SSH Authorized Keys | [T1098.004](https://bbs.kanxue.com/elink@c7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9&6z5q4)9J5c8U0l9H3y4q4)9J5c8R3`.`.) | `SSH`密钥可以被添加到`authorized_keys`文件中 |
| Defense Evasion | Obfuscated Files or Information | [T1027](https://bbs.kanxue.com/elink@02dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9J5y4#2)9J5c8R3`.`.) | `C2`配置文件在磁盘上被加密 |
| Defense Evasion | Deobfuscate/Decode Files or Information | [T1140](https://bbs.kanxue.com/elink@b9cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5@1x3q4)9J5c8R3`.`.) | `C2`配置文件用动态`XOR`算法进行解码 |
| Defense Evasion | Hide Artifacts | [T1564](https://bbs.kanxue.com/elink@d5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0f1$3y4q4)9J5c8R3`.`.) | 许多非常规内容被隐藏起来，包括端口、`PID`和文件时间戳。 |
| Defense Evasion | Masquerading | [T1036](https://bbs.kanxue.com/elink@36fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9K6y4W2)9J5c8R3`.`.) | 许多文件被伪装成其他文件或任务 |
| Defense Evasion | Rootkit | [T1014](https://bbs.kanxue.com/elink@f03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9I4y4q4)9J5c8R3`.`.) | 使用了`LKM`和`LD_PRELOAD` rootkits |
| Defense Evasion | Timestomp | [T1070.006](https://bbs.kanxue.com/elink@8a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9%4x3q4)9J5c8U0l9H3y4W2)9J5c8R3`.`.) | 修改`Lightning`创建的文件以与其它程序的文件相匹配 |
| Defense Evasion | File Deletion | [T1070.004](https://bbs.kanxue.com/elink@543K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9%4x3q4)9J5c8U0l9H3y4q4)9J5c8R3`.`.) | 该框架有能力删除自己 |
| Discovery | File and Directory Discovery | [T1083](https://bbs.kanxue.com/elink@284K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9^5x3#2)9J5c8R3`.`.) | 该框架可以列出被感染系统中的文件和目录 |
| Discovery | Network Service Discovery | [T1046](https://bbs.kanxue.com/elink@bf5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9@1y4W2)9J5c8R3`.`.) | 可以使用多个插件来进行网络服务探测活动 |
| Discovery | Network Sniffing | [T1040](https://bbs.kanxue.com/elink@01fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9@1x3q4)9J5c8R3`.`.) | 可使用多个插件来进行网络嗅探 |
| Discovery | System Information Discovery | [T1082](https://bbs.kanxue.com/elink@344K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9^5x3W2)9J5c8R3`.`.) | `Lightning` 可以进行系统指纹识别 |
| Command and Control | Data Encoding | [T1132](https://bbs.kanxue.com/elink@147K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5K6x3W2)9J5c8R3`.`.) | 来自`C2`的数据被加密 |
| Command and Control | Non-Application Layer Protocol | [T1095](https://bbs.kanxue.com/elink@fa2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9&6y4g2)9J5c8R3`.`.) | 与C2的通信是通过`TCP`进行的 |
| Command and Control | Proxy | [T1090](https://bbs.kanxue.com/elink@23dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9&6x3q4)9J5c8R3`.`.) | 该框架能够启动`Socks5`代理 |
| Command and Control | Exfiltration Over C2 Channel | [T1041](https://bbs.kanxue.com/elink@a91K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9@1x3g2)9J5c8R3`.`.) | 数据可以被盗取 |

[招生] 科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）

最后于 2022-7-23 21:44 被梦幻的彼岸编辑，原因：[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
