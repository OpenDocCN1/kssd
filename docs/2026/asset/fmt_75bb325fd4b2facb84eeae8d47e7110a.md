
# [原创]恶意样本分析学习—GlobeImposter3.0勒索病毒分析

## 一、样本信息

- 文件名：`svhost.exe`
- 文件大小：51712 字节
- MD5：C120F323C78D046C991F0EFE45F3819C
- SHA1：C6CBF8DD6DDA8388A6B5860A62091808E2B4D2BF

## 二、病毒介绍

GlobeImposter3.0是GlobeImposter家族的变种之一，2018年8月份该勒索病毒变种出现并攻击勒索国内多家大型医院和政企事业单位，GlobeImposter3.0的加密后缀为十二生肖英文名+4444，所以GlobeImposter3.0勒索病毒又被称为"十二生肖勒索病毒"。勒索病毒采用RSA加密文件，在加密目录下会生成勒索信息提示文件`HOW_TO_BACK_FILES.txt`，包含攻击者联系方式和受害者ID等信息。目前该勒索病毒无解密工具。

![](https://bbs.kanxue.com/upload/attach/202006/795506_452PJC4JCYTRX7J.png)

## 三、样本分析

1. 病毒会将自身文件复制到`AppData\Local`目录下，若该目录不存在则复制到`AppData\Roaming`目录下

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_FB46T37HDHUZ3HW.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_KJV6FRX6ZS8UMJQ.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_753J6SA95F8J2XA.png)

2. 病毒在`C:\Users\Public`目录下创建文件

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_PZKGNYG6V2C675A.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_S6X4BP98YHA9NZT.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_7XJKTPEN5EJ2U8P.png)

    文件内容包含RSA公钥和解密所需的用户ID

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_N3F674Z8QX67Q9R.png)

3. 在`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce`下创建`BrowserUpdateCheck`自启动项，实现开机自启动

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_SQ9QB55APZ8NN6C.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_S3MEYD6V93JVAWZ.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_47T8D4TQ2D2Y8WY.png)

4. 获取当前磁盘类型，该勒索病毒共支持三种磁盘类型，其中3为本地硬盘，2为软盘，4为网络磁盘

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_7RF44YYB3XVJ5JE.png)

5. 针对每个类型的磁盘创建一个子线程

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_GEFAPV8VK822USW.png)

    遍历磁盘下的文件并对文件进行加密

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_TH9NXF28BUJKFS3.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_TV2PDRDSTMKM34V.png)

    在加密目录下生成勒索信息提示文件`HOW_TO_BACK_FILES.txt`

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_95NWMCHSJEHBDBM.png)

    加密后的文件后缀为`.Snake4444`

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_NJ3Q47YBYF2BN4E.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_NM5DWQ8W864DP4J.png)

6. 在`tmp`目录下生成`.bat`脚本文件

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_9C3XFJKS4P7RGTC.png)

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_9KEZCWQFWQY2XMT.png)

    bat脚本内容如下，用于清除文件的卷影副本和rdp远程连接记录

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_63EWAVYJKV4EV3H.png)

7. 病毒执行完成后会运行`%COMSPEC% /c del filename > nul`，将自身删除

    ![](https://bbs.kanxue.com/upload/attach/202006/795506_R4QASB6CYJDVSDW.png)

## 四、安全建议

1. 勒索病毒常伪装为钓鱼邮件和常用软件，避免下载和运行可疑文件。
2. 及时更新系统补丁，防止攻击者通过进行漏洞攻击。
3. 不要使用弱口令密码，防止攻击者进行暴力破解。
4. 关闭不必要的，如139、445、3389等高危端口。
5. 安装主机防护软件并及时更新病毒库。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
