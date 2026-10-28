
> 不过，代码中有一点很有意思：用户打开进程管理器，该样本仍弹广告而不是休眠。（有可能是我分析错了，大牛可以在评论里面指正）

### 追踪URL

> 浏览器配置代理为`127.0.0.1:8080`, 使用burpsuite拦截浏览器请求.

> 1.经过一次重定向跳转至广告页面

![one_time_request](https://bbs.kanxue.com/upload/attach/202012/738427_BZYH2W8BNN3SAPH.png)

![one_time_response](https://bbs.kanxue.com/upload/attach/202012/738427_UEJDPEBZ4KRUSBG.png)

2.经过五次重定向跳转至广告页面

![five_times](https://bbs.kanxue.com/upload/attach/202012/738427_4GZJBXS9GKHGRYP.png)

3.经过七次重定向跳转至广告页面

![seven_time](https://bbs.kanxue.com/upload/attach/202012/738427_WMCHCJ2338N4FR5.png)

![final_ad](https://bbs.kanxue.com/upload/attach/202012/738427_FQ5JW86NCZPSG2U.png)

> 总结：每次浏览器重新请求`ce4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8Z5k6h3u0W2M7%4c8G2k6X3k6W2M7Y4y4A6L8Y4c8Z5k6i4N6W2j5W2)9J5k6h3y4G2L8b7.`页面，跳转至最终广告页面的次数不是确定的。

### 总结

> 整个恶意程序大部分都是使用开源代码开发，需要修改的部分只有一个`Inno Setup`安装脚本和一个推送广告的网站。这个样本只是虚拟机感染Adware样本之一，其他的样本分析手法类似。第一次分析`Innosetup`类型的样本，顺便花一上午学习`pascal`脚本。我本身主要做逆向分析的，欢迎其他大牛补充恶意广告域名部分内容。使用微步查询`onclickmax.com`,得到如下结果：

![threatbook](https://bbs.kanxue.com/upload/attach/202012/738427_XP8QT62G4PTQR28.png)

### 如何删除该样本

**1. kill process**

> 打开进程管理器，结束进程。一般该进程处于未响应状态。

![kill_process](https://bbs.kanxue.com/upload/attach/202012/738427_UHBFVSHWQNX2EXS.png)

**2. 删除相关文件**

> 删除当前恶意样本，然后打开`%tmp%`文件夹，删除相关文件。

![delete_files](https://bbs.kanxue.com/upload/attach/202012/738427_TWEXT467M5AZXUZ.png)

**3. 注册表相关**

> 检查`HKEY_CURRENT_USER\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run`是否有存在与该样本相关的自动项。

### 相关链接

|  |  |
| --- | --- |
| 1  2  3  4  5 | idp源码: `https://github.com/WPN-XM/WPN-XM/tree/master/bin/innosetup-download-plugin`<br>itdownload源码: `https://github.com/wilbit/itdownload.git`<br>psvince源码: `https://github.com/XhmikosR/psvince`<br>Pascal脚本学习：`https://iowiki.com/pascal/`<br>Innosetup Pascal脚本： `https://blog.csdn.net/yushanddddfenghailin/article/details/17250917` |

### IOC

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | website: `http://23cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8Z5k6h3u0W2M7%4c8G2k6X3k6W2M7Y4y4A6L8Y4c8Z5k6i4N6W2j5W2)9J5k6h3y4G2L8b7./`<br>URL: `ssh://42.21.62.5`<br>URL: `http://e44e2824-6dea-452b-bca8-b2c5db1680a9.s3.amazonaws.com/Bubble/Fake/Setup.exe`<br>URL: `http://e44e2824-6dea-452b-bca8-b2c5db1680a9.s3.amazonaws.com/Bubble/USA/Setup.exe`<br>URL: `http://e44e2824-6dea-452b-bca8-b2c5db1680a9.s3.amazonaws.com/Bubble/CA/Setup.exe`<br>URL: `http://e44e2824-6dea-452b-bca8-b2c5db1680a9.s3.amazonaws.com/Bubble/SE/Setup.exe` |

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
