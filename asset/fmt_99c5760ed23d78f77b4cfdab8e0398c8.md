
# All to Wma Coverter 1.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue155.htm`](https://www.pediy.com/kssd/pediy04/kanxue155.htm)

破解 All to Wma Coverter 1.3   
作者：Unregistered/2001   
媒体工具|音频工具 本站首页   
软件名称：All To WMA Converter   
整理日期：2001.10.22   
最新版本：1.3   
文件大小：2559KB   
软件授权：共享软件   
使用平台：Win9x/Me/NT/2000   
发布公司：Home Page   
软件简介：   
All To WMA Converter 可将 MP3、OGG、WAV 格式转换成 WMA 格式，使用上相当简单快速、只需点选 MP3 音乐、利用鼠标右键快显功能即可。   
本地下载：   
`gt.onlinedown.net`

前言：   
在版主的论坛精华看过大侠们的文章，眼界大开:-)于是自己找一个软件来练手。运气还好，写的不好，各位多扔砖头#%@#$$#

文章:

工具：`si`+`id`+`peditor`+`imprec`   
原程序被加了壳，脱壳后保护解除，只有关于的对话框没搞定。

### 1、脱壳。

它所采用的保护是 Asprotect 1.2?。首先找到入口点，走许多弯路到 `E533C2 Jmp eax`，此时 `eax=419478`。不用怀疑这个就是原始的入口点。`ImageSize=A5000`，所以在 `419478` 处：`/dump 400000 A5000 dumped.exe`，脱壳第一步完成。^-^

用 `peditor` 的 `dumpfixer`、修改入口，完成文件修复，图标出现。

`imprec` 修复 `importtable`，只有一个未修复：`[4600A0]=E3C944`，那里的指令是：`mov eax, [e43650]; ret;` 那么：`bpx e43650`，重新运行，中断下来，是调用了 `Kernel32!GetCurrentProcessId`。改正它，脱壳结束。

### 2、补丁。

问了如何设置窗口标题，竟无人知道，看来还得靠自己。`bpx setwindowtexta`。 

```
:004148ED A1108F4800                          mov eax, dword ptr [00488F10]
:004148F2 50                                  push eax
:004148F3 68F81E4800                          push 00481EF8
:004148F8 8D8D2CFFFFFF                        lea ecx, dword ptr [ebp+FFFFFF2C]
:004148FE 51                                  push ecx
* Reference To: USER32.wsprintfA, Ord:029Fh
                                              |
:004148FF FF1510074600                        Call dword ptr [00460710]
:00414905 83C40C                              add esp, 0000000C
:00414908 8D952CFFFFFF                        lea edx, dword ptr [ebp+FFFFFF2C]
:0041490E 52                                  push edx
:0041490F 8B8D1CFFFFFF                        mov ecx, dword ptr [ebp+FFFFFF1C]
* Reference To: MFC42.GetTickCount, Ord:1837h
                                              |
:00414915 E8224E0000                          Call 0041973C    ;setwindowtexta，设置窗口标题
```

改动这里，去掉窗口的未注册提示：

```
:00414908 8D952CFFFFFF                        lea edx, dword ptr [ebp+FFFFFF2C]
```

改为：`lea edx, [ebp-b0]`，窗口标题设置完毕。

‘关于’的对话框是烦人的，解决不了。 `4dd466d0e1e0cc123afbe504da2462ca`
