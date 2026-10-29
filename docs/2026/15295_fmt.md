
# 安卓恶意程序分析First

## 前奏

- 手机病毒是一种具有传染性、破坏性的手机程序，可用杀毒软件进行清除与查杀，也可以手动卸载。其可利用发送短信、彩信，电子邮件，浏览网站，下载铃声，蓝牙等方式进行传播，会导致用户手机死机、关机、个人资料被删、向外发送垃圾邮件泄露个人信息、自动拨打电话、发短（彩）信等进行恶意扣费，甚至会损毁 SIM卡、芯片等硬件，导致使用者无法正常使用手机。历史上最早的手机病毒出现在 2000年。

- 根据病毒危害特征分为以下八大危害

  ![](https://bbs.kanxue.com/upload/attach/202006/767964_472N25VQJMUBKPT.png)

- 今天进行实战分析讲解，希望此篇文章可帮到伙伴们

## 准备阶段

- 分析工具：`GDA`
- 解包工具：`7-zip`
- 权限查看：`aapt`

## 开始分析

为照顾刚入门或还未接触安卓分析的伙伴，此篇文章会讲解些基础知识，有不当之处，望谅解。

### 对该 APK 的结构与组成进行分析

![](https://bbs.kanxue.com/upload/attach/202006/767964_PSG4ZGNNXBNKDQZ.png)

### 对该 APK 的签名进行分析

![](https://bbs.kanxue.com/upload/attach/202006/767964_RXKCMSAZNGXR33Q.png)

发现 App 签名信息均为伪造的虚假信息

### 对 APK 的访问权限进行分析

![](https://bbs.kanxue.com/upload/attach/202006/767964_PCW4FD833MJ2ZSJ.png)

权限解析如下：

```
1. uses-permission:'Android.permission.READ_EXTERNAL_STORAGE' //读取外部存储
2. uses-permission:'android.permission.READ_PHONE_STATE' //读取手机状态
3. uses-permission:'android.permission.INTERNET' //访问网络连接，可能产生GPRS流量
4. uses-permission:'android.permission.WRITE_EXTERNAL_STORAGE' //允许程序写入外部存储，如SD卡上写文件
5. uses-permission:'android.permission.CAMERA' //允许访问摄像头进行拍照
6. uses-permission:'android.permission.VIBRATE' // 允许振动
7. uses-permission:'android.permission.MOUNT_UNMOUNT_FILESYSTEMS' //挂载、反挂载外部文件系统
8. uses-permission:'android.permission.RECORD_AUDIO' //录制音频
9. uses-permission:'android.permission.SYSTEM_ALERT_WINDOW' //显示系统窗口
10. uses-permission:'android.permission.FLASHLIGHT' //允许访问闪光灯
```

### 对是否存在恶意行为进行分析

前期知道该 APK 为辅助软件，根据其用途，进行分析行为是否为必须，非必须还存在并对用户有恶意影响这里就定义为恶意行为

1. 开启了录音功能（猜测是用来录制用户声音，进行窃取用户隐私）

   ![](https://bbs.kanxue.com/upload/attach/202006/767964_R8CWGSR8SAB9X3E.png)

   ![](https://bbs.kanxue.com/upload/attach/202006/767964_JWUGGC9J9DARDA3.png)

2. 开启了视频录制功能（猜测是用来录制用户画面，进行窃取用户隐私）

   ![](https://bbs.kanxue.com/upload/attach/202006/767964_KNGMHDY3EJUJ22Q.png)

3. 调用了 `TelephonyManager` 类（猜测是用来搜集用户电话号码信息，进行窃取用户隐私）

   ![](https://bbs.kanxue.com/upload/attach/202006/767964_8C85SWXNFZPFTQ7.png)

## 总结

综上所述，该 APP 在用户不知情或未授权的情况下，获取涉及用户个人信息的行为，具有隐私窃取属性，定义为隐私窃取型手机病毒。

此篇文章到此告一段落，期待下次相见，再续前缘。
