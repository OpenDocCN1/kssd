
# Rainbow Sentinel

> 原文：[`www.pediy.com/kssd/pediy03/forum920.htm`](https://www.pediy.com/kssd/pediy03/forum920.htm)

## 一、参考前面的碟子

## 二、Sentinel Driver 版本说明

Sentinel Driver 现在的版本到 `6.3` 了，但变化不大。

以 `Sx32w.dll` 为例：

- 用 `Wdasm` 反编，找出口函数
- 将 `INIT` 等找狗、初始化狗的函数
- 一律改返回值 `EAX=0`

这样，软件的检测狗的部分可以通过了。

## 三、对软件中对狗要求数据返回部分

- 仔细分析软件、尽可能找出有效数据
- 一般的加密软件来说，狗中的数据，在程序中都有对应数据
- `Sentinel Query` 函数，除返回调用正确外
- 在第三个调用参数（指针）是返回值 `response`
- 返回长度在第六个参数(`Short INT`)
- `Extend Read`, `Read` 的函数做法类似

谢谢阅读 `4dd466d0e1e0cc123afbe504da2462ca`
