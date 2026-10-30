
![](https://bbs.kanxue.com/upload/attach/202508/944427_N7WY5ZMZQTE4FDT.png)

![](https://bbs.kanxue.com/upload/attach/202508/944427_48U9RENVZB33CHN.png)

打开样本文档，调试器就自动附加了`EQNEDT32.exe`进程。

## 4.2 定位栈溢出点

调试`EQNEDT32.exe`，第一次调用函数`0x443F6C`时，寄存器和栈区情况如下所示

![](https://bbs.kanxue.com/upload/attach/202508/944427_VFXQ54WMC42EY22.png)

![](https://bbs.kanxue.com/upload/attach/202508/944427_6CS36X8NJKXKA7P.png)

第二次要调用函数`0x443F6C`时，寄存器和栈区情况如下图所示

![](https://bbs.kanxue.com/upload/attach/202508/944427_8PADQABDZWQT6EH.png)

![](https://bbs.kanxue.com/upload/attach/202508/944427_3HKKXX8AB54U7AT.png)

可以看到
地址 `0x18F484` 的值`0x43A851`被覆盖成了`0x450650`
地址 `0x18F4A4` 的值`0x4`被覆盖成了`0x44C329`
是不是很眼熟

## 4.3 验证 EIP 劫持

要判断这次栈溢出是不是真的劫持了程序的执行流，就在`0x450650`处下断，果然断了下来

![](https://bbs.kanxue.com/upload/attach/202508/944427_JKT4HVQRD5ATZ6V.png)

## 4.4 执行 payload

接着单步往下走几步，可以看到跳转到了地址特别远的地方，地址从 `0x045` 开头跳转到了 `0x41E`，这是执行 payload 的典型迹象；

### 4.4.1 解密

图中显示的是 payload 的循环解密操作

![](https://bbs.kanxue.com/upload/attach/202508/944427_2XSY2D2DXHYQU8C.png)

解密的数据也是 `0x41E` 开头的地址，在内存窗口可以看到解密出的系统 dll 和函数名称

![](https://bbs.kanxue.com/upload/attach/202508/944427_PDKEAJJ5J4E7CDV.png)

### 4.4.2 创建文件 Binex.exe

![](https://bbs.kanxue.com/upload/attach/202508/944427_Z6BUBKQ373EEK4M.png)

### 4.4.3 连接网络

![](https://bbs.kanxue.com/upload/attach/202508/944427_AH6G9YHSGQB8DEV.png)

从网络下载数据并将其写入本地文件 `binex.exe`

![](https://bbs.kanxue.com/upload/attach/202508/944427_XP2UNR2KGKSDCB6.png)

接下来应该就是执行下载的恶意程序 `Binex.exe` 了。分析过程中样本 payload 始终未能完成对 `Binex.exe` 的写入，可能是 C2 地址`ejeana.co.ug`目前失效了

![](https://bbs.kanxue.com/upload/attach/202508/944427_M8EVR3C886R7MV8.png)

# 5. 参考链接

[reconstructer.org/OfficeMalScanner.zip at master · fboldewin/reconstructer.org](https://bbs.kanxue.com/elink@37bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3j5X3!0D9k6r3g2%4K9h3&6Q4x3V1k6J5k6h3y4G2L8Y4y4@1M7Y4g2U0N6r3g2J5i4K6u0W2L8%4u0Y4i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8V1!0X3k6X3W2U0k6f1#2S2L8q4y4U0j5h3&6F1k6i4u0Q4x3X3g2*7K9i4l9`.)

[恶意代码漏洞分析：CVE-2018-0798 | 云涯历险记](https://bbs.kanxue.com/elink@f3bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6N6h3&6&6j5i4N6#2i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9$3i4K6u0r3x3U0S2Q4x3V1k6Q4x3U0g2q4y4W2)9J5y4e0R3I4i4K6t1#2b7U0k6Q4x3U0g2q4y4W2)9J5y4e0R3@1i4K6t1#2z5p5k6Q4x3U0g2q4y4q4)9J5y4f1u0n7i4K6t1#2b7e0y4Q4x3U0g2q4y4#2)9J5y4f1p5H3i4K6t1#2z5o6q4Q4x3U0g2q4y4W2)9J5y4f1u0o6i4K6t1#2z5p5k6Q4x3U0g2q4y4W2)9J5y4f1t1@1i4K6t1#2z5f1g2Q4x3U0g2q4y4g2)9J5y4e0R3^5i4K6t1#2z5o6k6Q4x3U0g2q4y4W2)9J5y4e0W2q4i4K6t1#2z5e0m8Q4x3U0g2q4c8W2)9J5y4f1u0o6i4K6t1#2z5f1q4U0N6X3g2Q4x3X3b7J5x3o6p5^5i4K6u0V1x3o6M7&6z5q4)9J5c8R3`.`.)

[403 Forbidden](https://bbs.kanxue.com/elink@9d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7I4z5e0j5I4z5e0c8Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.)

[一种新升级的公式编辑器漏洞武器化工具已投入使用 – 绿盟科技技术博客](https://bbs.kanxue.com/elink@67bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8Y4y4X3L8$3y4#2M7#2)9J5k6h3&6W2N6q4)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0S2Q4x3X3b7H3y4K6V1^5i4K6u0r3)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#缓冲区溢出](https://bbs.kanxue.com/forum-150-1-156.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
