
# [原创][代程]远程线程详解(一):无DLL远程线程注入

## 1. 题记

有一些小心得和大家分享一下，希望能帮助有需要的朋友。

## 2. 概要

无DLL远程线程注入获得演示程序的密码。

## 3. 关键词

远程线程注入 星号密码 初级shellcode

## 4. 具体细节

### 4.1 目的

得到`TargetForGetPass`中的星号密码。

### 4.2 方法一

send `WM_GETTEXT`。无效。

### 4.3 方法二

`CreateRemoteThread`-->inject shellcode-->将密码保存于`c:\pass.txt`中-->`GetPassword`打开--->显示密码。

## 5. 另

1. 文件在vs2003.net上编译时会出现`_rmtGetPass`符号未定义问题，第二个附件为解决该问题的源代码及可执行文件。（其他平台未测试）
2. 若出现`multiple symbol`，请在链接时使用`/FORCE:MULTIPLE`。

## 6. 最后

感谢月夜同学的讨论。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [`C3_RemoteThread.rar`](javascript:void(0);)（35.83kb，188次下载）
- [`C5_CoffEditor_a.rar`](javascript:void(0);)（31.64kb，169次下载）
