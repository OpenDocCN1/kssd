
![下载文件1-pic39](upload/attach/202003/879321\_TY6Z8JKJHDYS537.jpg)

![下载文件2-pic40](upload/attach/202003/879321\_UPFC87QXK6UYWJY.jpg)

###### ⑥.删除命令

通过URL`<mark class="encrypted">529K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6A6j5$3!0D9k6h3c8G2N6s2y4G2L8W2)9J5k6h3W2U0N6g2)9J5c8X3c8W2j5X3u0&6i4K6u0r3N6$3g2S2N6r3S2W2M7X3k6G2M7X3c8Q4x3V1k6$3P5h3c8S2L8s2W2@1P5g2)9$3x3q4!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4g2)9^5z5q4!0m8x3q4!0q4z5g2)9&6z5g2!0m8y4q4!0q4y4W2)9^5b7#2)9^5y4#2!0q4y4q4!0n7b7W2!0m8y4l9`.</mark>

![获取删除指令-pic41](upload/attach/202003/879321\_6AWNPAAP2KUTUZ8.jpg)

此外我们还关联到一个与之相似的样本,诱饵文档与之相同故不再赘述

| 样本信息 | Internet in government(互联网在政府机构) |
| :----------: | :----------------------------------------------------------: |
| 样本MD5 | 20d21c75b92be3cfcd5f69a3ef1deed2 |
| 样本SHA-1 | fd20567190ef2920c5c6c449aeeb9fe75f7df425 |
| 样本SHA-256 | 23aa2347bf83127d40e05742d7c521245e51886f38b285be7227ddb96d765337 |
| 样本类型 | Win32 EXE GUI程序 |
| 样本大小 | 2.01 MB (2106880 bytes) |
| 编写语言 | Pascal |
| 编译器信息 | Free Pascal Compiler v.3.0.4 [2019/10/27] for i386 |
| 时间戳 | 1970-01-01 1:00 (100%造假) |
| 最初上传时间 | 2020-01-20 12:09:44 |

#### (2).Employee-entitlements-2020

##### a.样本信息

| 样本信息 | Employee-entitlements-2020(员工权益-2020) |
| :----------: | :----------------------------------------------------------: |
| 样本MD5 | 91f83b03651bb4d1c0a40e29fc2c92a1 |
| 样本SHA-1 | c1cfc6bbd8ce0ce03d7cd37c68ee9e694c582aef |
| 样本SHA-256 | b33f22b967a5be0e886d479d47d6c9d35c6639d2ba2e14ffe42e7d2e5b11ad80 |
| 样本类型 | MS Word 文档 带有恶意宏 |
| 样本大小 | 43.00 KB (44032 bytes) |
| 样本创造时间 | 2020-01-20 09:10:00 |
| 最后保存时间 | 2020-01-20 10:11:00 |
| 最初上传时间 | 2020-01-22 08:41:44 |

![pic z6.png](upload/attach/202003/879321\_B94G48FXRT6ZTFR.jpg)

该样本属于包含恶意`宏`的文档,我们打开可以看到其内容关于`财政部关于文职和军事雇员福利的声明`,属于`政治类`题材样本

![样本Employee-entitlements-2020.doc正文与翻译-pic](upload/attach/202003/879321\_HZWQQD52NPM66PS.jpg)

##### b.样本分析

通过使用`olevba` dump出其包含的恶意宏代码(如下图所示:)

其主要逻辑为:下载该URL`<mark class="encrypted">09cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6L8X3c8S2i4K6u0V1j5$3q4D9L8r3q4Y4K9r3q4F1i4K6u0W2K9h3y4#2i4K6u0r3e0h3W2F1K9$3!0%4M7$3E0A6i4K6u0r3j5Y4u0G2N6$3&6Q4y4U0m8Q4c8e0c8Q4b7U0S2Q4z5p5q4Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5o6k6Q4z5o6g2Q4c8e0g2Q4b7f1g2Q4b7U0W2Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0k6Q4z5f1y4Q4b7f1y4Q4c8e0g2Q4z5p5k6Q4b7U0m8Q4c8e0k6Q4z5f1y4Q4b7V1q4Q4c8e0g2Q4z5e0W2Q4b7e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4y4U0m8Q4x3U0g2b7M7X3!0Y4M7X3q4E0c8r3q4@1j5g2)9J5y4g2)9#2b7@1W2F1N6r3g2Y4M7X3q4@1k6h3c8a6k6X3k6A6j5$3g2Q4x3X3g2@1P5s2c8Q4y4U0m8Q4x3U0S2Q4c8e0k6Q4b7f1c8Q4b7e0c8Q4c8e0k6Q4z5e0N6Q4b7U0k6Q4c8e0g2Q4b7U0W2Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4z5p5c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0g2Q4z5o6g2Q4b7U0k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0W2Q4z5e0N6Q4b7e0S2Q4x3V1y4Q4c8e0S2Q4z5o6m8Q4z5p5y4Q4c8e0c8Q4b7U0S2Q4z5e0c8Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0W2Q4z5e0N6Q4b7e0S2Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4y4U0m8T1j5i4y4W2y4U0c8Q4y4U0m8Q4c8e0N6Q4b7V1y4Q4z5e0k6Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0N6Q4b7V1u0Q4z5e0y4Q4c8e0k6Q4z5f1g2Q4z5f1y4Q4x3U0W2Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0W2Q4z5o6m8Q4z5f1q4Q4c8e0S2Q4b7V1k6Q4z5o6N6Q4c8e0S2Q4b7f1k6Q4b7V1u0Q4c8e0g2Q4z5p5k6Q4z5e0k6Q4y4U0m8u0L8Y4c8W2k6%4u0S2N6r3g2V1e0$3k6X3K9h3y4W2i4K6u0W2N6s2S2@1i4K6j5H3i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6R3&6i4K6R3H3i4@1f1$3i4K6W2o6i4K6R3&6i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1#2i4@1q4q4i4@1t1&6i4@1f1#2i4@1t1H3i4K6R3$3i4@1f1#2i4K6R3#2i4@1t1$3i4@1f1^5i4@1p5%4i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1#2i4K6V1H3i4K6S2q4i4K6u0o6i4@1f1$3i4K6S2m8i4K6S2m8i4@1f1$3i4K6V1#2i4@1t1H3i4@1f1$3i4K6S2p5i4@1q4q4i4@1f1$3i4@1t1#2i4K6R3I4i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1#2i4K6R3#2i4@1p5#2i4K6j5H3i4K6t1#2f1s2u0G2k6%4u0S2L8f1c8S2N6r3q4Q4x3U0g2Q4y4f1y4u0L8Y4c8W2k6%4u0S2N6r3g2V1e0$3k6X3K9h3y4W2i4K6u0W2k6i4S2W2i4K6j5H3i4@1f1@1i4@1t1^5i4@1q4p5i4K6u0o6i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1@1i4@1t1^5i4K6V1@1i4@1f1#2i4@1u0n7i4@1t1$3i4@1f1^5i4@1u0r3i4K6W2r3i4@1f1^5i4@1u0r3i4K6V1H3i4@1f1^5i4@1p5I4i4K6S2o6i4K6j5H3i4K6t1#2f1s2u0G2k6%4u0S2L8f1c8S2N6r3q4Q4x3U0g2Q4y4f1y4u0L8Y4c8W2k6%4u0S2N6r3g2V1e0$3k6X3K9h3y4W2i4K6u0W2k6i4S2W2i4K6j5H3i4@1f1#2i4K6R3^5i4@1p5H3i4@1f1&6i4K6V1&6i4@1p5@1i4K6j5H3i4K6t1#2f1s2u0G2k6%4u0S2L8f1c8S2N6r3q4Q4x3U0g2Q4y4f1y4u0L8Y4c8W2k6%4u0S2N6r3g2V1e0$3k6X3K9h3y4W2i4K6u0W2N6s2S2@1i4K6j5H3</mark>

![样本Employee-entitlements-2020.doc中的恶意宏文件主要代码(带注释)-pic](upload/attach/202003/879321\_UUKCBAAKQ2NSYN2.jpg)

| 样本信息 | IntegratedOffice.exe |
| :----------: | :----------------------------------------------------------: |
| 样本MD5 | e8effd3ad2069ff8ff6344b85fc12dd6 |
| 样本SHA-1 | 417e60e81234d66ad42ad25b10266293baafdfc1 |
| 样本SHA-256 | 80fb33854bf54ceac731aed91c677d8fb933d1593eb95447b06bd9b80f562ed2 |
| 样本类型 | Win32 EXE GUI程序 |
| 样本大小 | 1.95 MB (2047488 bytes) |
| 编写语言 | Pascal |
| 编译器信息 | Free Pascal Compiler v.3.0.4 [2019/10/27] for i386 |
| 时间戳 | 1970-01-01 1:00 (100%造假) |
| 最初上传时间 | 2020-01-22 12:29:16 |

![样本IntegratedOffice.exe文件信息(图片)-pic](upload/attach/202003/879321\_FJJ2X77MC4UTGPH.jpg)

该样本属于上一个样本中的`下载者(Downloader)`部分,其还是通过创建`GUID`.bin标记感染机器

![创建guid.bin-pic](upload/attach/202003/879321\_6ZEWNHGF6XQGYAG.jpg)

并且创建指向自身的快捷方式于自启动文件夹中

![在自启动文件夹创建指向自身的快捷方式-pic](upload/attach/202003/879321\_JY5D7W4D7M5R3GA.jpg)

剩下的收集信息并且等待回显数据的操作都与上文中提到的相同故此不再赘述

#### (3).Brochure-Jerusalem\_26082019\_pdf

##### a.样本信息

| 样本信息 | Brochure-Jerusalem\_26082019\_pdf(手册-耶路撒冷) |
| :----------: | :----------------------------------------------------------: |
| 样本MD5 | 46871f3082e2d33f25111a46dfafd0a6 |
| 样本SHA-1 | f700dd9c90fe4ba01ba51406a9a1d8f9e5f8a3c8 |
| 样本SHA-256 | 284a0c5cc0efe78f18c7b9b6dbe7be1d93da8f556b432f03d5464a34992dbd01 |
| 样本类型 | Win32 EXE GUI程序 |
| 样本大小 | 2.27 MB (2376192 bytes) |
| 编写语言 | Pascal |
| 编译器信息 | Free Pascal Compiler v.3.0.4 [2019/10/27] for i386 |
| 时间戳 | 1970/1/1 1:00 (100%造假) |
| 最初上传时间 | 2020-02-16 07:08:10 |

![pic z7](upload/attach/202003/879321\_5XU2S8ANHGA2SFW.jpg)

##### b.样本分析

通过`FindResource`函数查找资源`MYDATA`,通过下图我们可以看出该资源是一个`PDF`文件

![FindResource函数查找资源MYDATA-pic](upload/attach/202003/879321\_ZU3WHFRWNXFMDJX.jpg)

通过`CreateFile`函数将文件源数据写入`%Temp%\Brochure-Jerusalem\_26082019.pdf`(诱饵文件)中

![通过CreateFile函数将文件源数据写入Brochure-Jerusalem\_26082019.pdf-pic](upload/attach/202003/879321\_KTE45XZWHVKH5QF.jpg)

通过`ShellExecute`函数将`%Temp%\Brochure-Jerusalem\_26082019.pdf`打开

![打开Brochure-Jerusalem\_26082019.pdf-pic](upload/attach/202003/879321\_BJ28K438877WKG3.jpg)

该样本关于`耶路撒冷`的话题,属于`政治类`诱饵文档

![诱饵文件Brochure-Jerusalem\_26082019.pdf内容以及翻译-pic](upload/attach/202003/879321\_2HPKH4VQDE6H2NT.jpg)

之后的行为就和之前的如出一辙了,在此就不必多费笔墨。

#### (4).Congratulations\_Jan-7\_78348966\_pdf

##### a.样本信息

| 样本信息 | Congratulations\_Jan-7\_78348966\_pdf(恭喜7月) |
| :----------: | :----------------------------------------------------------: |
| 样本MD5 | 09cd0da3fb00692e714e251bb3ee6342 |
| 样本SHA-1 | 82d425384eb63c0e309ac296d12d00fe802a63f1 |
| 样本SHA-256 | 4be7b1c2d862348ee00bcd36d7a6543f1ebb7d81f9c48f5dd05e19d6ccdfaeb5 |
| 样本类型 | Win32 EXE GUI程序 |
| 样本大小 | 2.17 MB (2270720 bytes) |
| 编写语言 | Pascal |
| 编译器信息 | Free Pascal Compiler v.3.0.4 [2019/10/27] for i386 |
| 时间戳 | 1970-01-01 1:00 (100%造假) |
| 最初上传时间 | 2020-01-22 19:22:17 |

![pic z8.png](upload/attach/202003/879321\_2S8RW7P5AP4FY44.jpg)

##### b.样本分析

通过`FindResource`函数查找资源`MYDATA`,通过下图我们可以看出该资源是一个`PDF`文件

![FindResource函数查找资源MYDATA-pic](upload/attach/202003/879321\_4FY3X868GVGZN64.jpg)

通过`CreateFile`函数将文件源数据写入`%Temp%\Congratulations\_Jan-7.pdf`(诱饵文件)中

![通过CreateFile函数将文件源数据写入Congratulations\_Jan-7.pdf-pic](upload/attach/202003/879321\_YY7XT6YB7S3WE2E.jpg)

通过`ShellExecute`函数将`%Temp%\Congratulations\_Jan-7.pdf`打开

![打开Scholarships in Serbia 2019-2020.pdf-pic](upload/attach/202003/879321\_T6AMJG6N782TWP4.jpg)

该样本关于`耶路撒冷归属`的话题,属于`政治类`诱饵文档

![诱饵文件Congratulations\_Jan-7.pdf内容以及翻译-pic](upload/attach/202003/879321\_Z5JDNJRW92G8BSS.jpg)

之后的行为就和之前的如出一辙了,在此就不必多费笔墨。

#### (5).Directory of Government Services\_pdf

##### a.样本信息
