
### 1、 `WM_DPICHANGED` 消息传入的建议窗口大小及位置有偏差

引入 DWM 开始，顶层窗口的阴影部分也被计入窗口区域（平时我们使用不同的窗口截图工具有的会多截取边缘有的不会，就是因为专业一点的截图工具考虑到了这个 DWM 阴影，详情参考 [MSDN: `DWMWA_EXTENDED_FRAME_BOUNDS`](https://bbs.kanxue.com/elink@52dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3c8%4L8h3q4H3K9g2)9J5c8X3&6W2i4K6u0V1k6s2N6E0j5i4m8A6i4K6u0V1k6s2N6E0N6$3W2F1k6r3!0%4j5i4c8@1M7X3W2T1N6i4c8W2)）。系统给我们 `WM_DPICHANGED` 消息时，`lParam` 指向的 `Rect` 未计入此偏差，按这个 `Rect` 来 `SetWindowPos` 就会出现这个偏差，所以新的位置和大小我们得自己算：

![](https://bbs.kanxue.com/upload/attach/202206/853701_FE8JSE6PSBUTBPD.jpg)

### 2、缩放后高度总是有一点偏差

在 Visual Studio 等资源编辑器设计窗口时，并未计入菜单高度。如果在程序运行时通过 `SetMenu` 动态加了菜单，那么按照原来的设计，高度肯定会出现偏差，偏差等于菜单高度。如果菜单一定要窗口创建完后动态增加，那么可以在资源编辑器设计窗口时增加一个空菜单，窗口创建完成后动态替换为新菜单。

## 实现参考

[`NTAssassin/NTADPI.c` at master · KNSoft/NTAssassin](https://bbs.kanxue.com/elink@688K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6z5g2p5q4K6M7$3q4K6M7$3W2F1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2y4G2N6i4u0U0k6g2)9J5c8V1&6f1b7i4y4K6j5i4y4K6K9h3&6Q4x3V1k6z5g2p5q4p5f1p5W2Q4x3X3g2U0)

[`KNSoft/AlleyWind`: An advanced Win32-based and open-sourced utility that helps you to manage system's windows](https://bbs.kanxue.com/elink@198K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6m8L8r3I4W2P5g2N6A6L8X3b7`.)

## 参考资料

- 《MSDN - DPI and Device-Independent Pixels》
  https://bbs.kanxue.com/elink@a2cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3I4W2j5i4u0F1N6$3W2F1x3K6u0Q4x3V1k6V1M7r3W2Q4x3X3c8S2L8X3c8Q4x3X3c8V1k6i4k6A6j5$3g2Q4x3X3c8A6L8X3c8W2M7r3g2F1k6r3g2F1N6q4)9J5k6s2m8A6P5r3g2D9M7H3`.`.)

- 《MSDN - High DPI Desktop Application Development on Windows》
  https://bbs.kanxue.com/elink@07fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3S2A6k6s2m8A6i4K6u0r3K9r3W2Y4K9q4)9J5k6r3c8H3K9g2)9J5k6r3c8W2M7$3E0@1L8%4m8Q4x3X3c8S2M7s2m8D9K9h3y4S2N6r3W2G2L8W2)9J5k6r3c8W2N6X3g2D9L8%4m8E0k6h3&6@1i4K6u0V1L8$3&6Q4x3X3c8%4K9h3&6V1L8%4N6K6)

- 《Windows Blogs - Improving the high-DPI experience in GDI based Desktop Apps》
  https://bbs.kanxue.com/elink@78aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6i4N6A6L8X3c8G2N6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6V1L8%4N6K6k6r3g2$3k6h3I4G2M7r3g2J5i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9#2i4K6u0r3x3e0W2Q4x3V1k6A6L8i4m8J5L8%4k6A6L8X3N6Q4x3X3c8Z5K9h3N6Z5i4K6u0V1k6s2m8A6i4K6u0V1k6i4S2H3k6i4u0A6k6h3&6U0k6g2)9J5k6r3N6V1K9g2)9J5k6r3u0S2M7$3g2V1i4K6u0V1k6r3g2K6K9%4c8G2M7q4)9J5k6r3q4H3M7s2y4Q4x3V1j5`.)

尚有不足，或有谬误，欢迎指正，以上实现也将继续改进。若需要进一步研究，建议参考上述 MSDN 资料，非常详细、全面。

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://bbs.kanxue.com/elink@4e9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4J5k6h3q4@1K9i4k6W2j5$3!0E0L8h3!0F1M7#2)9J5k6h3!0J5k6#2)9J5c8X3I4A6j5$3g2F1M7$3g2K6i4K6u0r3j5Y4W2Q4x3V1j5@1i4K6u0W2x3q4)9J5c8R3`.`.).

Ratin/ratin@knsoft.org

国家认证 系统架构设计师

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-6-25 16:34

被 Ratin 编辑，原因： 勘误

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)
