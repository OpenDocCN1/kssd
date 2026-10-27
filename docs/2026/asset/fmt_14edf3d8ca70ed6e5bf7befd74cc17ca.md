
# [原创]Wibu Codemeter 7.3学习笔记——AxProtector壳初探

前一篇 [[原创]Wibu Codemeter 7.3学习笔记](https://bbs.pediy.com/thread-275395.htm)

**看完了回个贴呗，你们的关注是我更新的动力**

***求一个Wibu AxProtector的license，或者是一个能用的AxProtector，后续研究需要***

## 0x00工具

1. Fake Wibu Codemeter Server
2. Wireshark
3. IDA Pro

## 0x01最近研究的成果，发现

1. `decrypt_telegram`函数 可以接受15s以内的通信延迟，因此可以做一个Codemeter协议解析器

   这里我修改了一点Wireshark的源码，让Wireshark将Codemeter的数据包转发给假服务器，这样可以让我们更加清晰直观的观察Codemeter的通讯过程，帮助我们在假服务器上实现那堆API![](https://bbs.kanxue.com/upload/attach/202212/786730_S8PNJFBBFJPE6GR.jpg)

2. 存在大量未公开的api 例如

   `APICODE 119`:用于帮助文档的解析

   `APICODE 32` 与Codemeter Control Center交换数据

   `APICODE 130` 添加`WibuCmLif`

### 进度

1. 已完成数据包发送函数的构建
2. 已完成数据包解析框架，实现绝大部分的授权，管理API
3. 用Qt简单写了个GUI，未完成

### TODO:

#### Fake Wibu Codemeter Server

1. 实现`CmGetBoxContents2` `CmGetInfo(ENTRYDATA ENTRYINFO)`等license相关API
2. 实现`CmCrypt`等加密相关API
3. 实现`HCMSE`句柄，及`CmContainner` `CmBox`的结构，管理

### 逆向

`CmCalculateSignature`的调用

## 0x02由CodeMeter API的调用简析Axprotector逻辑

观察受AxProtector保护的程序调用Cm API的行为
