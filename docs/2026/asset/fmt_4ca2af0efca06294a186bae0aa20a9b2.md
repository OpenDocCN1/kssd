
### 使用JTAG完成基本硬件初始化

为了确保初始化准确无误，需要保证CPU上电时完全无法找到可以引导的代码。

因为我之前清空flash时，也曾尝试立刻刷回StartCode，可惜不知什么原因，只写入了2KB的StartCode，这样仍然无法完成基本的初始化。但不清楚这2KB的代码完成了哪些操作，所以一个办法就是先将flash的`CE#`（`CE#`这种后面带#字的引脚，都是低电平有效）与3.3V短接（最好是能与输出3.3V的GPIO短接，与3.3V的VCC短接电流可能较大，不知是否有不良影响），再给板子上电。HG8245H的`CE#`引脚在板子背面电源led附近有个上拉电阻`R1542`，可以在这里短接。

CPU上电后，在OpenOCD的telnet里执行上面脚本提供的`sd5115_hwinit`函数，等看到输出

> Hardware initialization is complete!

字样的提示后，把之前dump出来的StartCode写入内存并运行：

```
halt ; load_image binary/mtd0.bin 0x82000000 ; resume 0x82000000
```

这时应该可以看见UART里输出了久违的StartCode信息：

> HuaWei StartCode 2012.02 (R13C10 Apr 22 2014 – 18:06:02)
>
> NAND: Nand(Hardware): 128 MiB
> startcode select the uboot to load
> the high RAM is :8080103c
> startcode uboot boot count:-1
> Slave struct initializtion success!!
> Use the UbootA to load first
> Start from UbootA ERROR, Change to UbootB
> Both UbootA and UbootB are wrong, load it by JTAG!

### 补充：

新版本的OpenOCD配置文件见：[c16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7$3g2J5M7$3!0X3N6q4)9J5c8X3S2A6i4K6g2X3M7$3b7#2x3e0p5#2i4K6g2X3L8%4m8W2L8X3!0U0k6q4)9#2k6X3y4G2L8X3k6A6k6H3`.`.](https://bbs.kanxue.com/elink@3b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7$3g2J5M7$3!0X3N6q4)9J5c8X3S2A6i4K6g2X3M7$3b7#2x3e0p5#2i4K6g2X3L8%4m8W2L8X3!0U0k6q4)9#2k6X3y4G2L8X3k6A6k6H3`.`.)

最新sd5115 openocd 配置文件见：[d61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7$3g2J5M7$3!0X3N6q4)9J5c8X3S2A6i4K6g2X3M7$3b7#2x3e0p5#2i4K6g2X3L8%4m8W2L8X3!0U0k6q4)9#2k6X3y4G2L8X3k6A6k6H3`.`.](https://bbs.kanxue.com/elink@706K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7$3g2J5M7$3!0X3N6q4)9J5c8X3S2A6i4K6g2X3M7$3b7#2x3e0p5#2i4K6g2X3L8%4m8W2L8X3!0U0k6q4)9#2k6X3y4G2L8X3k6A6k6H3`.`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [hi\_sd5115\_jtag.zip](javascript:void(0);)（2.61kb，32次下载）
- [blog.csersoft.net-华为HG8245H救砖番外篇之利用JTAG完成硬件初始化.pdf](javascript:void(0);)（154.44kb，34次下载）
