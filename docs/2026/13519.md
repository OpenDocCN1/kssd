
# [原创]一个简单病毒的分析

生平在看雪的第一篇帖子，本人小菜，就拿一简单病毒入手，大牛们见笑....

## 病毒信息

- 病毒类型：后门
- 文件 MD5：2A1AEF106795864CA9DB643A116807DC
- 加壳类型：UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo

## 病毒行为

### 释放文件

`c:\WINDOWS\Fonts\wuauclt.exe`

### 具体行为

1. 提升自身权限。

   ![](https://bbs.kanxue.com/upload/attach/201008/47173_eqyk7xi9tlidt23.png)

2. 加载`urlmon.dll`，得到`URLDownloadToFile`函数的地址。调用函数下载病毒文件`0e1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0x3$3x3q4)9J5k6e0q4K6i4K6u0W2k6Y4u0Q4x3V1k6H3M7#2)9J5k6h3A6H3k6#2!0q4y4g2)9^5z5q4!0n7x3r3y4Q4x3@1q4Q4y4f1y4%4K9h3&6V1N6$3!0K6i4K6g2o6k6X3!0F1N6s2y4Q4y4f1y4Y4k6i4u0F1i4K6u0W2k6X3!0F1i4@1f1%4i4K6W2n7i4@1q4q4i4@1f1#2i4@1u0p5i4K6V1#2i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1K6i4K6R3H3i4K6R3J5`。

   ![](https://bbs.kanxue.com/upload/attach/201008/47173_sjs9036lbjau4wr.png)
   ![](https://bbs.kanxue.com/upload/attach/201008/47173_xsvvfsx9myo1uyk.png)
   ![](https://bbs.kanxue.com/upload/attach/201008/47173_ih4h29rlnie5on4.png)

3. 在`c`盘根目录下生成`sa.exe`文件，设置文件为文件夹样式并设置为隐藏属性。运行`sa.exe`，然后调用`CMD`命令结束`wuault`（系统自动更新）进程，然后将病毒本身拷贝到`system`下的`font`目录中并重新命名为`wuault.exe`。病毒运行`font`目录下的`wuault`程序后，调用`cmd`命令删除自体。

   ![](https://bbs.kanxue.com/upload/attach/201008/47173_5cvl2ge6zbc7g4o.png)

   ```
   0012FBFC   0040247E  ~$@.  /CALL 到 WinExec 来自 dumped\_.0040247C
   0012FC00   0012FF20   .  |CmdLine = "cmd /c del "C:\Documents and Settings\commander\桌面\dumped\_.exe""
   0012FC04   00000000  ....  \ShowState = SW\_HIDE
   0012FC08   7C930738  8搢   ntdll.7C930738
   ```

4. 将病毒信息保存在`c:\windows\font\gern.fon`文件中，从这个文件中读取信息并设置为IE首页。

   ![](https://bbs.kanxue.com/upload/attach/201008/47173_crwrlsl5tbtf6gm.png)

5. 开启1083端口等待网络连接。

附件：样本文件及加壳后文件。因为是病毒所以加了密码，解压密码为：xiaoju![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.png](javascript:void(0);) （17.64kb，770次下载）
- [2.png](javascript:void(0);) （2.03kb，768次下载）
- [3.png](javascript:void(0);) （2.48kb，765次下载）
- [4.png](javascript:void(0);) （5.42kb，764次下载）
- [5.png](javascript:void(0);) （17.92kb，764次下载）
- [6.png](javascript:void(0);) （24.04kb，766次下载）
- [样本.rar](javascript:void(0);) （111.21kb，124次下载）
