
# 太阳镜++

> 原文：[`www.pediy.com/kssd/pediy03/forum799.htm`](https://www.pediy.com/kssd/pediy03/forum799.htm)

先备份原来的文件

运行提示为“这是〈太阳镜＋＋〉的未注册版本!”

1. 用 `FI` 分析 `sunglass.exe`，得出为 `UPX` 1.0 加的壳
2. 用 `PDUMP32` V1.6 脱壳
3. 用 `W32DASM` 反汇编
4. 用串式参考找到“这是《太阳镜++》的未注册版本！”双击

找到如下内容：

```
004B1003  MOV ECX, 004B10FC
004B1008  MOV EAX, DWORD PTR [004BEA3C]
004B100D  MOV EAX, DWORD PTR [EAX]
004B100F  CALL 004324D4
```

向上查找可以跳过此段的地方

```
004B0FE3  CMP DWORD PTR [EAX+0C], 00000000
004B0FE7  JNE 004B1014      =〉此处刚好跳过去
```

将 `JNE` 改为 `EB`

测试成功，所有限制解除

嘻嘻……我第一次用 `W32DASM` 有什么不足的地方望指正 `4dd466d0e1e0cc123afbe504da2462ca`
