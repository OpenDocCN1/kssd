
# 某 App `deviceinfo` 分析

## 1. 加密函数

`com.***.***.***.***.***.JNICLibrary#doCommandNative` 方法，其中当序号为 `10400` 时为加密算法。

## 2. 快速分析

使用自制的 `trace` 工具，对序号为 `10400` 的执行流程进行 `trace`。在 `trace` 文件中输入 `libart` 查看所有的 `libart` 调用。

![图片描述](https://bbs.kanxue.com/upload/attach/202210/847002_W68MEM4DQS6W3JD.png)

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_VV4M5R3XBA483UP.jpg)

可以看到输入是 `protobuf`，输出是 `byte array`。并且输入和输出相隔了 40 多万条指令。

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_YHAMGGAZZFPF6RT.jpg)

## 3. 从输入流开始追踪

在 `trace` 文件中搜索 `0x2a`。

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_GFYWJ8WAKMJ69RX.jpg)

对应的 C 语言代码：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_QMKYYVGKF9D66HA.jpg)

经分析，该函数功能是对输入的 `byte arr` 进行字节转换。再往下：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_KSWPMGZ3XBACCDY.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_RWA7A7N5WCT4GDS.jpg)

这是具体的加密函数，对输入的数据 16 位一组进行加密，末尾补 `pad` 个 `pad`。

```python
pad = 0x10 - (input % 16)
```

使用 Python 还原一下：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_DGUYRSPG6FNVSKE.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_82Q89YNFEPG3334.jpg)

为减少干扰，删除已还原数据，搜索目标数据：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_J7UN2KSXYVS8GAX.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_32AHU2G3EDBKGCE.jpg)

经过分析，该函数为计算 `hash`，调用了两次，计算的 `hash` 值为：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_VV8ECVASVAMR9KG.jpg)

第二次计算的 `hash` 为结果串的前 8 个字节中的后四位。输入为：`002a7b0a-7ae6-4000-93af-60c2d63176fe`。

继续搜索，可以发现在 `0x13714` 位置，对数据进行了 `eor` 运算，`eor` 结果即为输出的数据。

从 `0x20` 起，即为 `eor` 第一步中加密算法的结果。

前 32 位数据为：

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_5G9BZAWFXVZZ4XC.jpg)

未加注释的地方，由于 `trace` 工具对内存读写这方面的支持还不是很好，就鸽了，感兴趣的可以自己分析，基本上在这儿。

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_TQPQTYCXS7BVWP3.jpg)

## 4. 结果串前 8 位分析

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_SF4WDMHFJ3RJT8T.jpg)

前面说到后四位是由第二次计算结果的值获得。那么前 4 位呢？

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_CE89FA7J66UGR8P.jpg)

前两位固定，后两位是由一个数 `^ CDEF` 而来。

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_TK53CSB5DE3PJQQ.jpg)

由于 `trace` 工具对内存读写还不太完善，具体没追这个值，不过大胆猜测这是该加密函数的调用次数，可以用 `frida` 验证下结果。

## 5. 最终结果异或值分析

![image.png](https://bbs.kanxue.com/upload/attach/202210/847002_MCXEFVX5V4QEVAP.jpg)

`0x35 0x79.......0x32` 是 `0x10` 长度的 `salt`，多次对 App 进行清空等操作，发现只有两组 `salt`，分别为：

- `5yPDTSC3rzXfpwN2`
- `fGqSL6alaNcUyV9W`

同样由于 `trace` 工具对内存读写访问的问题，不太容易跟踪，在此不得不感慨，`unidbg` 的内存访问断点是真的好用。哎，有时间根据加密算法，还原下解密吧。

---

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-10-15 21:57 被我只是没人要编辑。

原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
