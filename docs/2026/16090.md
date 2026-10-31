
# [原创]安卓 APP 抓包之双向认证突破

## APP 抓包问题

APP 抓包问题已经是老生常谈的一个问题了，今天正好碰到这个问题，经过一番折腾最终解决了这个问题。

## 先解决安卓 7 手机或模拟器正常抓包问题

### 1. 导出 Burp 的 DER 证书

![](https://bbs.kanxue.com/upload/attach/202101/777761_3MS9CDA8ZR6PS8N.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/777761_JMRJNWD8GBKMWVU.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/777761_U43BU86QK9A4NTB.jpg)

### 2. 使用 OpenSSL 对证书进行配置

将 DER 转换成 PEM，命令如下：

```bash
openssl x509 -inform DER -in cacert.der -out cacert.pem
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_JSTXVMYCNHWXYFC.jpg)

### 3. 查看 PEM 证书的 Hash 值并记录

命令如下：

```bash
openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_3YQERB2RGJSXJPY.jpg)

### 4. 将 PEM 证书改名为“Hash 值.0”

```bash
mv cacert.pem 9a5ba575.0
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_82TJBCR4WR3MY4P.jpg)

### 5. 将证书上传到手机或模拟器

使用 Root 权限启动：

```bash
adb root
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_UX524SY93KQ5FB7.jpg)

### 6. 重新安装分区读写

```bash
adb remount
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_UTZJXHGA9HE4QGT.jpg)

### 7. 复制证书并设置权限

把 `9a5ba575.0` 证书文件复制到手机或模拟器的系统证书文件夹，并设置 `644` 权限。设置完成后重启手机或模拟器。

以下路径与手机或模拟器路径一致，例如：`/sdcard/Download/9a5ba575.0`。

```bash
adb shell

mv /sdcard/Download/9a5ba575.0 /system/etc/security/cacerts/

chmod 644 /system/etc/security/cacerts/9a5ba575.0
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_3BNSZTG6GKWA2C5.jpg)

### 8. 查看证书是否为系统级信任证书

重启完成后，查看证书是否为系统级信任证书。

![](https://bbs.kanxue.com/upload/attach/202101/777761_A2NY2DV3CG9ZY4V.jpg)

至此，安卓 7 的 Burp 证书已被系统信任。上述演示的模拟器为逍遥模拟器，这个模拟器我也只是偶尔用一下，其他手机或模拟器的操作步骤差不多。

但是经过以上一顿乱搞后，你会发现抓手机或模拟器浏览器的包时还是会弹出证书的问题。被测 APP 如果不是证书绑定（SSL Pinning）和双向认证的问题，还是可以正常进行抓包测试的。

我安卓 7 用得少，因为证书问题有点麻烦，基本上都是用安卓 5 进行测试，除非一些 APP 只能安卓 7 以上版本才能运行，才会用到安卓 7 进行测试。

## 开始进入正题：突破双向认证

之前有看到过双向认证证书的突破思路，今天正好在项目中碰到了，而且 APP 又正好没有加固，免除了脱壳烦恼，因此有了以下突破过程。

刚开始不知道 APP 做了双向证书校验，结果杯具就发生了。

![](https://bbs.kanxue.com/upload/attach/202101/777761_Y54J797BUM2YJDW.jpg)

然后开始寻找之前某大佬提供的突破思路。

### 搜索证书相关关键字

因为 APP 没有进行加固，可以直接用 `jadx` 反编译出来，然后全局搜索 `.cer`、`.crt`、`.pfx`、`PKCS12`、`keyStore` 等等关键字，我这里搜索的是 `client.pfx`。

![](https://bbs.kanxue.com/upload/attach/202101/777761_NVQBZHBKMK498XT.jpg)

转到代码位置查看详情，包括证书安装密码、其他密码等信息。可以看到，PFX 证书没有设置密码。

本地测试安装时，看到有需要输入密码。第一次尝试过输入密码，结果提示密码错误；第二次尝试不输入密码却安装成功了，于是开启了全局搜索之旅，看到没有设置证书安装密码。

![](https://bbs.kanxue.com/upload/attach/202101/777761_4GQ4TGRYU2DEQ22.jpg)

### 1. 解压 APP

将 APP 以压缩包形式解压出来。

![](https://bbs.kanxue.com/upload/attach/202101/777761_HRF9AT7KV2BYWCD.jpg)

### 2. 搜索证书文件

进入解压出来的目录，可以搜索一些证书的后缀文件，例如 `cer`、`p12`、`pfx` 等。一般安卓下的为 `bks`，也可以先去 `assets` 或 `res` 目录下找找。

我碰到的 APK 就在 `assets` 目录下存放：

![](https://bbs.kanxue.com/upload/attach/202101/777761_Q6AUA7NB5F2W3CE.jpg)

### 3. 导出密钥

本来以为还得在文件夹里面继续找 key，找了一圈无果，然后咨询了某大佬，大佬让我直接把 key 导出来。因为之前没遇到过，觉得有点新鲜，就刚了一波。

![](https://bbs.kanxue.com/upload/attach/202101/777761_CMS7A2PGQ96VZRG.jpg)

```bash
openssl pkcs12 -in client.pfx -nocerts -nodes -out client.key
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_VUNGWSSAN43XBM3.jpg)

#### 注意事项

导出 key 的时候需要输入密码，也就是下图所示的位置：

![](https://bbs.kanxue.com/upload/attach/202101/777761_879NRQHYBNCRMN4.jpg)

因为我这里没有设置密码，所以不用输入，直接回车导出；如果设置了证书密码，这里需要输入证书密码才能把 key 导出来。

### 4. 合并证书和密钥

有了 key 剩下的就好办了，将 CRT 证书和 key 文件合并成 `.p12` 证书文件。合并的时候记得对证书进行加密，也就是加一个证书密码；不加密码，Burp Suite 是无法导入的。

合并证书命令如下：

```bash
openssl pkcs12 -export -inkey client.key -in client.crt -out client.p12
```

![](https://bbs.kanxue.com/upload/attach/202101/777761_JHY7VSM5R43W24H.jpg)

### 5. 将证书导入 Burp Suite

![](https://bbs.kanxue.com/upload/attach/202101/777761_ATG4GQKPQ7H6GDR.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/777761_78SPUAXPCEXX8KB.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/777761_4U99NXWTEQG7SFR.jpg)

### 6. 确认证书已启用

证书导入成功，并且已启用。

![](https://bbs.kanxue.com/upload/attach/202101/777761_79SWN98GSN43TRA.jpg)

### 7. 重启手机或模拟器并抓包

接下来就是见证奇迹的时刻了。为了保险起见，先重启一下手机或模拟器，最好是彻底关闭然后再打开，我是这样操作的。

最后成功抓到该 APP 的数据包。

![](https://bbs.kanxue.com/upload/attach/202101/777761_RP82QUPHST7YFYE.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/777761_9D6YCF48A8JBH2Z.jpg)

## 总结

以上就是本次突破双向认证成功抓包的全过程，有问题可共同探讨、学习。本人技术也不是很好，可能有很多地方描述得不是很好，有问题的地方还请各位指正。

## 参考链接

- [604K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0p5%4x3e0V1@1i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@21dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0p5%4x3e0V1@1i4K6u0W2K9s2c8E0L8l9`.`.)

- [331K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4e0b7H3x3U0N6Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@739K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4e0b7H3x3U0N6Q4x3X3g2Z5N6r3#2D9)

- [[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-7-7 11:03 被丶空心编辑。

原因：

- [#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
- [#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
