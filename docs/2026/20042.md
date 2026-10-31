
# 湖湘杯CTF大赛reverse(逆向)题分享

如题，本人大一学生，前段时间参加湖湘杯CTF大赛在复赛中遇到了一道有趣的逆向题目，不过最终没能做出来。已在52某论坛发帖但至今无解，希望求看雪大神前来讨论。

## 难点分析

1.此`crackme`为`win32 console`控制台程序（控制台程序的窗口是系统创建的，所以`OD`载入会自动弹出一个窗口，此时程序并没有跑飞），所以断点的设置比较困难。
2.此程序是用`python`语言编写，再把`python`脚本文件打包成`EXE`。程序运行后实际上是运行的`python`虚拟机，然后将解压的`python`字节码在虚拟机上运行，所以直接跑`OD`不能定位到关键代码(仅个人观点，不一定正确）
3.此题的目的是获取`flag`，而`flag`在哪，如何得到`falg`主办方没有任何提示，也就是说爆破破解后不一定会出现`flag`。

## 提示

1.最后的`flag`一般是以`flag{......}`的形式出现（仅供参考）
2.本程序需要安装`VS2015`支持库才能正常运行，所以最好在实体机或者`win7 32位`环境下破解。

## 本人的思路

1.直接逆向，不过没能解决代码定位问题。2.解包逆向`pyc`二进制文件，但是貌似没有得到有用的信息。(应该是正解，但是没能成功，仅供参考)

![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

## 原题截图

![](https://bbs.kanxue.com/upload/attach/201701/589933_3l2evvp29utm76z.png)

## 附件在最后

[`login.rar`](https://bbs.kanxue.com/attach-download-109708.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [`login.png`](javascript:void(0);)
  （21.03kb，90次下载）
- [`login.rar`](javascript:void(0);)
  （3.79MB，290次下载）
