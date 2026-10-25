
# Yoda's Protector V1.03.3 静态脱壳机(附源码)

> 原文：[`www.pediy.com/kssd/pediy11/117070.html`](https://www.pediy.com/kssd/pediy11/117070.html)

稀里糊涂过了一年,找了份工作.`pe` 格式忘得差不多了.所以拿此壳来练手![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

给新手朋友们一点参考,希望大牛们多拍砖与指点.可能以后的工作会跟壳有关了![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

## 说下大概流程

1. 首先解密 `shell` 代码的两处加密,由于 `yp` 的加密方式有了一点小动态.所以采取了动态抠代码的方式来解密的.(直接用 `opcode` 比较效率高一些,人懒了点.带了个反汇编引擎.见谅)
2. 对加密的节区进行三次解密.
3. 如有资源节,则还原.并替换掉 `.x01` 节为 `.rsrc`
4. 还原 `IAT`,将 `.yp` 节删掉
5. 修复 `OEP`,`sizeofImage` 等等信息.
6. `dump2file`.

`ps`:写完后才发现此壳开源的![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心"),有意向的朋友可看源码学习

## Yoda's Protector v1.03.3 主程序下载地址

`http://www.pediy.com/tools/PACK/Protectors/yodap/yp1.03.3.zip`

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `bin.rar`  

|

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `src.rar`  

|
