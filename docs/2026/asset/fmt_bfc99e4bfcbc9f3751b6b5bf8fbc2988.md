
## 安装步骤

### 步骤 1

下载项目后，可以重命名项目文件夹名称为`MobSf`，打开`cmd`窗口进入该项目目录。将项目内的`requirements.txt`打开，把最后一行`libsast==1.2.2`改为`libsast==1.3.4`。

### 步骤 2

安装`OpenSSL`和`wkhtmltopdf`，并配置好`wkhtmltopdf`环境后，在终端进入文件夹，然后运行`run.bat`。

### 步骤 3

打开浏览器，在地址栏输入`127.0.0.1:8000`。如果需要修改默认端口，可以在`run.bat`中修改`SET conf="0.0.0.0:8000"`中的端口号。

![image-20210903125716878](https://bbs.kanxue.com/upload/attach/202109/905443_FWP8EFJJJ73BKBS.png)

![image-20210903125748945](https://bbs.kanxue.com/upload/attach/202109/905443_M9CSZPCRUFY5ANZ.png)

一般在运行时，会出现一些报错，例如：

![image-20210903125841996](https://bbs.kanxue.com/upload/attach/202109/905443_4TR8U2GXSJSHRXN.png)

### 原因解析

由于众所周知的网络原因，导致无法访问一些域名，如`raw.githubusercontent.com`。

#### 解决办法

1. 通过`IPAddress.com`首页，输入`raw.githubusercontent.com`，查询到真实IP地址，例如：`185.199.108.133`。
2. 在本地电脑`hosts`文件中添加记录`185.199.108.133 raw.githubusercontent.com`。

例如：`IPAddress.com`首页。

![image-20210903130404834](https://bbs.kanxue.com/upload/attach/202109/905443_88UWDFEMZCJ7QK8.png)

然后进入`C:\WINDOWS\system32\drivers\etc`，修改`hosts`文件并保存。

![image-20210903130536306](https://bbs.kanxue.com/upload/attach/202109/905443_525P9XK45NYUSQY.png)

重新启动`run.bat`，再输入`127.0.0.1`，就可以正常进入，不再报错。

![image-20210903130713504](https://bbs.kanxue.com/upload/attach/202109/905443_GM2F8E96DJE6HGA.png)

同理，在后续访问过程中，如果出现类似错误，按照此解决方案即可解决。

![image-20210903131022264](https://bbs.kanxue.com/upload/attach/202109/905443_2Q6RRAD78H7MD78.png)

但是，由于这一般是网络受限导致的，所以修改后仍然可能存在报错。不过一般不会影响正常使用。要彻底解决，可以修改代码，大家可以尝试一下。

## Docker 安装

1. 下载镜像：

   ```bash
   docker pull opensecurity/mobile-security-framework-mobsf
   ```

2. 启动容器：

   ```bash
   docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
   ```

### 参考网址

`https://www.mad-coding.cn/2019/10/11/%E4%BD%BF%E7%94%A8docker%E5%AE%89%E8%A3%85%E7%A7%BB%E5%8A%A8%E5%AE%89%E5%85%A8%E6%A1%86%E6%9E%B6%EF%BC%88MobSF%EF%BC%89/#0x01-%E5%BC%80%E5%A7%8B%E5%AE%89%E8%A3%85`

## Linux 安装

### 参考网址

`https://blog.csdn.net/Alexhcf/article/details/107438583`

## Mac 安装

### 安装环境

- `Mac OS 10.14`
- `Python 3.8`
- `Java 12.0.2`
- `MobSF v3.1 beta`

### 安装步骤

参考网址：`http://www.51ste.com/share/det-5952.html`

## （4）基本使用

![image-20210903131925360](https://bbs.kanxue.com/upload/attach/202109/905443_QMA6CRB9HJ9RGTY.png)

直接拖入一个APP开始分析：

![image-20210903132056941](https://bbs.kanxue.com/upload/attach/202109/905443_SEFD987U5K6K7ET.png)

![image-20210903132027761](https://bbs.kanxue.com/upload/attach/202109/905443_PQU75PFQDCQNT9U.png)

可以看见APP正在进行逆向分析。

![image-20210903132200502](https://bbs.kanxue.com/upload/attach/202109/905443_FWD7ZGJZTBTP59A.png)

### 静态分析

点击最近扫描，就可以看见最近扫描的一些APP情况：

![image-20210903132445977](https://bbs.kanxue.com/upload/attach/202109/905443_846RYAJUAR5W9KN.png)

随便点击一个应用的静态报告，就可以看见详细的静态分析结果。

![image-20210903133349135](https://bbs.kanxue.com/upload/attach/202109/905443_V457S87RXQPXAP9.png)

### 动态分析

需要安装`Genymotion`并注册账号，创建一个模拟器。这里创建的是`7.0`模拟器。

### Genymotion 官方地址

`https://www.genymotion.com/download/`

![image-20210903133349135](https://bbs.kanxue.com/upload/attach/202109/905443_84JF444ECRPPPCU.png)

启动创建的模拟器。

![image-20210903134450994](https://bbs.kanxue.com/upload/attach/202109/905443_YC89MB6GQM7JECJ.png)

重新启动`MobSF`，在平台上点击`Dynamic Analyzer`发现新的内容，点击`MobSFy Android Runtime`给模拟器安装插件。

![image-20210903134624497](https://bbs.kanxue.com/upload/attach/202109/905443_8MYB5UYEZKR32V4.png)

![image-20210903135626110](https://bbs.kanxue.com/upload/attach/202109/905443_95QC335KTUKB5Z2.png)

然后点击动态分析，开始进行动态分析。

在这个过程中，可能会遇到各种错误，下面依次解决。

### 常见问题

#### 问题 1：Genymotion 模拟器无法联网

需要检测`Genymotion`模拟器的相关配置，具体解决方案参考：

`https://www.freebuf.com/paper/378877`

#### 问题 2：电脑上的安全管家

电脑上的`360`管家等最好关闭。虽然不一定是这个导致的，不过作为一名逆向人员，最好不要打开这些管家。

#### 问题 3：VirusBox 安装路径

安装`Genymotion`时，需要将`VirusBox`安装到默认路径下，否则会报错。安装后重启电脑。

![image-20210903172351260](https://bbs.kanxue.com/upload/attach/202109/905443_AZ7R5A42RMJWKZD.png)

![image-20210903171425268](https://bbs.kanxue.com/upload/attach/202109/905443_TAFNJ2DTPT2755W.png)

点击`Start Instrumentation`开始自动化遍历扫描。

![image-20210903172517016](https://bbs.kanxue.com/upload/attach/202109/905443_ARSHKDC9QHJXQD8.png)

还可以实时查看API监测情况：

![image-20210903172556551](https://bbs.kanxue.com/upload/attach/202109/905443_N2NWBEPW767MQS6.png)

然后生成动态分析报告。

![image-20210903172736972](https://bbs.kanxue.com/upload/attach/202109/905443_JBH28WJTK9GWGWW.png)

## 四、总结

本文针对Android APP漏洞做了一个初步的介绍，先调研了当下的一些主流漏洞，以及常见的Android APP端漏洞挖掘方式，还结合APP漏洞审计系统，详细列出了当下Android APP端漏洞挖掘的初步思路，为漏洞挖掘安全人员提供一个参考。

本文还介绍了Android APP漏洞挖掘过程中的三种常用工具：`drozer`、`Inspeckage(Xposed)`和`MobSF`，详细介绍了三种工具的安装和具体使用教程，能很好地帮助Android漏洞挖掘和渗透测试人员开展工作。

本文的知识总结来自个人学习总结和网络上大量博客的收集，所有博客都会列在参考列表中，可能还有一些总结不足，后续逐步完善，也欢迎各位大佬指正。

GitHub首页：[github](https://bbs.kanxue.com/elink@6a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6q4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3g2U0L8$3@1.)

## 五、参考网址

### Android APP 漏洞

- 《移动应用安全形势报告》（`2020`）：`https://www.cac.gov.cn/2020/08/20/c_1603845965740538.htm`
- `https://ayesawyer.github.io/2019/08/21/Android-App%E5%B8%B8%E8%A7%81%E5%AE%89%E5%85%A8%E6%BC%8F%E6%B4%9E/`
- `https://xuanxuanblingbling.github.io/ctf/android/2018/02/12/Android_app_part1/`

### drozer

- `https://labs.f-secure.com/tools/drozer/`
- `http://rui0.cn/archives/30`
- `https://www.0-sec.org/wordpress/?p=3438`

### Inspeckage

- `https://www.e-learn.cn/topic/3470422`
- `https://blog.csdn.net/tom__chen/article/details/78216732`
- `https://repo.xposed.info/module/mobi.acpm.inspeckage`

### MobSF

- `https://www.freebuf.com/article/13073244765/`
- `https://blog.csdn.net/Alexhcf/article/details/107438583`
- `https://github.com/MobSF/Mobile-Security-Framework-MobSF`
- `http://www.51ste.com/share/det-5952-3.html`
- `https://bbs.pediy.com/thread-218973.htm`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-4-12 17:07 被随风而行aa编辑，原因：`drozer`指令修改。

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
