
# 鹦鹉螺网络助手 1.75

> 原文：[`www.pediy.com/kssd/pediy03/forum600.htm`](https://www.pediy.com/kssd/pediy03/forum600.htm)

鹦鹉螺网络助手 1.75 破解手记

作者：孙 锋  
地址：山东省曲阜师范大学电教系 97 级 142#  
Email：`sffs@263.net`

最新的版本是：1.75，下载地址：`http://netkit.top263.net`。破解方法和以前的一样，下面是破解过程：

步骤：

1、随便输入注册信息，我的是：
- 用户名：`sunfeng`
- 注册码：`1212121212`

2、启动 `TRW2000`，OK 载入内存。`bpx hmemcpy` 万能中断，`F5` 返回程序领域。

3、点击确定按钮，被 `TRW2000` 拦截到，`BD *` 中断所有中断，`F12` 共 12 次，然后 `F10` 到下面的位置：

```
0167：0041A7BF CALL 0041A990 ；----->此处用 F8 跟踪进入 CALL 内部
```

4、`F8` 进入 `CALL` 内部，然后一直 `F10` 到下面的位置：

```
0167：0041A9F6 PUCH EDX ；-------->D EDX。这里是正确的注册码
```
得到正确的注册码是：`pibungzsle`

5、整理：
- 用户名：`sunfeng`
- 注册码：`pibungzsle`

6、另外注册码：
- 用户名：`孙锋`
- 注册码：`Sfyrkdwpib`

7、其他注册码：

| 用户名 | 注册码 |
| :--- | :--- |
| `灰松鼠` | `Lyrkdwpibu` |
| `番外地` | `Wjcvohatmf` |
| `自由用户` | `Kxqjcvohat` |
| `我就喜欢盗版` | `[ngzslexqj` |

<---------完---------> `4dd466d0e1e0cc123afbe504da2462ca`
