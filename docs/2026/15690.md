
# [原创]彻底解决Android模拟器中文短信乱码问题

给模拟器发送短信的过程：

`DDMS/Telnet -> console -> Kernel`

发送短信导致乱码的地方有两个：`DDMS` 程序和 `console`。

短信乱码问题的原因：

- 中文短信是以 `UCS2` 编码，存储在 `PDU` 的 `USD` 区。
- 而传给系统的编码非 `UCS2`。

## 第一步：修改 `DDMS` 与 `console` 的编码

`DDMS` 传给 `console` 的是 `GBK` 编码，`console` 接收后由底层统一转成 `UTF-8` 编码，至于为什么这里就不说了！

`DDMS` 最后是用 `socket` 与 `console` 进行交互，UI 操作以命令传给 `console`。

![](https://bbs.kanxue.com/upload/attach/201211/550116_789c0ad5b8d0469f.png)

默认发送时获取字节码的编码是 `ISO-8859-1`，所以我们做下修改：

![](https://bbs.kanxue.com/upload/attach/201211/550116_d25c5006030c9d0e.png)

`console` 接收后由一个命令分发表转到 `sms_send`：

![](https://bbs.kanxue.com/upload/attach/201211/550116_5e61814ead432250.png)

将字节码从 `utf-8` 流中解析出来：

![](https://bbs.kanxue.com/upload/attach/201211/550116_48a497718f303111.png)

这里就出现了一个错误。汉字的 `UTF-8` 为三个字节，在读取第二个和第三个字节前跳过了第二个，所以注释掉：

![](https://bbs.kanxue.com/upload/attach/201211/550116_e8febeef2f8eb518.png)

顺便修改 `console` 发送时候的编码函数，方法同上。

## 构建 PDU 短信

最后的步骤是用发送者号码和短信内容构建 PDU 短信：

`smspdu_create_deliver_utf8` -> `smspdu_create_deliver` -> `gsm_rope_add_sms_deliver_pdu`

我们主要关注 `gsm_rope_add_sms_deliver_pdu` 就行了。

在这个函数中，主要涉及到 PDU 编码的知识，这个也不详细叙述，想了解的可以去百度。

根据代码分析，Google 在用户数据区前面多加了一个长度信息。因为英文短信是从另外一个分支处理的，所以不受影响！

![](https://bbs.kanxue.com/upload/attach/201211/550116_7d3049b0957d4ff8.png)

修改方法如图，注释掉那段代码！

## 编译

OK，我们流程都打通了，现在就需要编译了。

编译我也说一句吧，需要编译模拟器 `emulator`，`sdk` 或者熟悉点的可以只编译 `ddms`，然后拷贝到 `sdk`。

好了，打完收工！你可以用 `telnet` 连接模拟器：

`telnet localhost 5554 -> sms send 10086 你好!`

或者在 `DDMS` 那里发送，效果是一样的！

## 涉及文件

- `external\qemu\telephony\sms.c`
- `external\qemu\android\console.c`
- `sdk\ddms\libs\ddmlib\src\com\android\ddmlib\EmulatorConsole.java`

本文是本人原创，转载请说明出处！

不好意思，可能图片挂了。大家可以看：

```
607K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6r3!0E0K9r3!0#2M7W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6t1K6y4o6R3#2y4b7`.`.
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
