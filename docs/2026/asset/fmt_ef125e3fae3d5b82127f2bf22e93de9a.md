
在`WinDbg`快捷方式的目标选项中追加 `` -y SRV*C:\symbol*6faK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3`. -b -k com:port=//./pipe/com_1,baud=115200,pipe ``

![image-20210603141131658](https://bbs.kanxue.com/upload/attach/202106/867232_VKHWZ9QCWZCP9RA.jpg)

## 3.5 系统环境变量配置

增加名为`_NT_SYMBOL_PATH`的变量，值为`SRV*C:\symbol* 167K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8D9i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6K6P5h3#2T1L8$3I4K6`

![image-20210603134806180](https://bbs.kanxue.com/upload/attach/202106/867232_5YHSAMHREDWNAJP.jpg)

## 3.6 验证

选择Debug系统，管理员运行`windbg`，如下图断下就说明双机调试已经配置成功了。

![image-20210603141520390](https://bbs.kanxue.com/upload/attach/202106/867232_UW2T37M4QVYYBM4.jpg)

接下来`windbg`输入`lm`验证符号配置，如下图则说明符号配置成功

![image-20210603141837381](https://bbs.kanxue.com/upload/attach/202106/867232_8NKFM3BQE5QW9PM.jpg)

输入`ld *`可以强制加载所有符号，包括目前暂未使用的符号也会被事先加载。第一次加载时需要从网络上下载符号文件，因此会加载的慢一点，耐心等待全部加载完毕即可。

![image-20210603143027117](https://bbs.kanxue.com/upload/attach/202106/867232_ME2CUYC6AF49VG2.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-6-3 14:51 被SSH山水画编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
