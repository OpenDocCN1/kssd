

# 黑云压城城欲摧 - 2016年iOS公开可利用漏洞总结

作者：蒸米，耀刺，黑雪 @ Team OverSky

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_853dd308f20f94c3.jpg)

## 0x00 序

iOS的安全性远比大家的想象中脆弱，除了没有公开的漏洞以外，还有很多已经公开并且可被利用的漏洞，本报告总结了2016年比较严重的iOS漏洞（可用于远程代码执行或越狱），希望能够对大家移动安全方面的工作和研究带来一些帮助。

## 0x01 iOS 10.1.1 公开的可利用漏洞

1. `mach_portal`攻击链：该攻击链是由Google Project Zero的Ian Beer公布的。整个攻击链由三个漏洞组成：损坏的内核`port`的`uref`可导致任意进程的`port`被越权替换（CVE-2016-7637），`powerd`任意`port`替换可导致DoS（CVE-2016-7661），因为`set_dp_control_port`没有上锁导致的XNU内核UaF（CVE-2016-7644）。

攻击者先使用CVE-2016-7637将`launchd`与`com.apple.iohideventsystem`系统服务具有发送权限的`port`替换成自己控制的进程的`port`，并且攻击者还具有该`port`的接收权限。然后，攻击者利用CVE-2016-7661对`powerd`这个进程进行DoS，使其重启。在启动过程中，因为`powerd`的启动需要用到`com.apple.iohideventsystem`系统服务，于是将`task port`发送给了这个系统服务。但因为攻击者利用之前的CVE-2016-7637漏洞获取了`com.apple.iohideventsystem`系统服务`port`的接收权限，因此攻击者获得了`powerd`的`task port`，从而控制了具有root权限并且在沙盒外的`powerd`进程。攻击者随后利用`powerd`进程的`task port`获取到了`host_priv port`，然后利用`host_priv port`触发因`set_dp_control_port`没有上锁而导致的XNU内核UaF（CVE-2016-7644）漏洞，从而控制了`kernel task port`。攻击者在获取了`kernel task`以后，就可以利用系统提供的`mach_vm_read()`和`mach_vm_write()`去进行任意内核读写了。

2016年12月22日，`qwertyoruiop`在Ian Beer公布的`mach_portal`攻击链的基础上，加入了KPP的绕过、内核patch和`cydia`的安装，并在自己的twitter上发布了iOS 10.1.*的越狱。

## 0x02 iOS 9.3.4 公开的可利用漏洞

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_68e66afcd11e0475.jpg)

1. PEGASUS 三叉戟攻击链：该攻击链是在对阿联酋的一位人权活动家进行APT攻击的时候被发现。整个攻击链由三个漏洞组成：`JSC`远程代码执行（CVE-2016-4657），内核信息泄露（CVE-2016-4655），内核UAF代码执行（CVE-2016-4656）。

在浏览器漏洞方面，由于iOS系统的`JavaScriptCore`库的`MarkedArgumentBuffer`类在垃圾回收的时候可能会造成内存堆破坏，导致黑客可以使用该漏洞泄露对象地址以及执行任意指令。在内核漏洞方面，由于XNU内核的`OSUnserializeBinary()`函数在反序列化用户态传入的数据时没有对`OSNumber`的长度进行校验，导致可以泄露内核的栈信息。利用精心构造的`OSString`对象，还可以触发UAF漏洞并导致内核代码执行。具体的分析可以参考我们之前的文章：[基于PEGASUS的OS X 10.11.6本地提权](https://bbs.kanxue.com/elink@b67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4q4Q4x3X3g2S2L8r3W2T1j5h3u0S2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8i4g2F1K9i4c8&6i4K6u0r3M7$3S2G2N6#2)9K6c8X3q4J5N6r3W2U0L8r3g2A6k6q4)9K6c8o6f1K6x3g2!0q4c8W2!0n7b7#2)9^5z5g2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4g2)9^5z5q4!0m8z5g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4z5q4!0m8c8W2!0m8y4g2!0q4y4W2)9&6y4q4!0n7b7W2!0q4y4g2)9^5y4#2!0n7b7W2!0q4z5g2)9&6x3#2!0n7c8g2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4g2)9^5x3g2)9&6b7g2!0q4y4g2)9^5z5q4!0n7x3r3W2a6f1#2!0q4y4q4!0n7z5q4)9^5b7g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5q4!0n7c8W2)9&6b7#2!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0m8c8g2)9^5b7#2!0q4y4#2!0n7c8g2)9^5c8g2!0q4z5q4!0n7y4W2)9^5b7g2!0q4y4#2)9^5b7W2!0n7x3g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4z5q4!0m8c8W2!0n7y4q4!0q4y4W2)9&6z5q4!0m8c8W2!0q4z5q4!0n7c8W2)9&6x3g2!0q4y4g2)9^5y4#2!0m8x3q4!0q4y4g2!0n7z5g2!0n7y4q4!0q4y4W2)9&6c8q4!0m8y4g2!0q4y4g2!0n7c8q4!0n7x3g2!0q4y4g2)9&6x3#2)9^5c8q4!0q4y4W2)9&6b7#2)9^5x3q4!0q4y4g2!0m8y4q4!0m8y4#2!0q4y4#2)9&6b7g2)9^5y4r3W2a6f1#2!0q4y4W2!0n7b7#2)9^5c8W2!0q4y4W2!0n7y4q4)9&6c8g2!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4g2!0n7z5g2!0n7y4W2!0q4y4q4!0n7z5q4)9&6y4q4!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4W2)9&6b7#2!0m8b7g2!0q4y4W2)9&6c8q4!0m8y4g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9&6c8g2)9^5x3g2!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4g2)9^5c8W2!0m8c8W2!0q4z5q4)9^5x3#2!0n7c8q4!0q4y4g2)9^5y4#2!0n7b7g2!0q4y4#2)9^5c8g2!0n7x3q4!0q4y4g2)9^5z5q4!0m8z5g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4z5q4!0m8c8W2!0m8y4g2!0q4y4W2!0n7b7#2)9^5c8W2!0q4y4W2!0n7y4q4)9&6c8g2!0q4y4#2)9&6b7g2)9^5y4r3W2a6f1#2!0q4y4g2!0m8y4q4!0m8y4#2!0q4z5g2)9&6c8q4!0m8x3W2!0q4y4#2!0m8y4#2!0m8c8W2!0q4y4W2)9^5b7#2)9^5x3W2!0q4z5g2!0m8z5g2!0m8b7#2!0q4y4q4!0n7b7g2)9^5b7W2!0q4y4q4!0n7b7W2!0n7y4W2!0q4x3#2)9^5x3q4)9^5x3R3`.`).

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_9a89f149cdb74f32.png)

利用PEGASUS对iOS 9.3.* 32位设备越狱的DEMO：

```
a6aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6Q4x3X3g2&6L8%4g2C8N6g2)9J5k6h3y4G2L8g2)9J5c8Y4k6Q4y4h3k6K6K9r3!0%4i4K6u0r3K9h3c8Q4y4h3k6j5e0g2c8Y4y4p5&6*7b7e0g2a6g2p5g2%4e0@1q4Q4x3@1c8Q4x3@1c8Q4x3X3g2Z5N6r3#2D9
```

## 0x03 iOS 9.3.3 公开的可利用漏洞

1. `IOMobileFramebuffer` Heap overflow 内核漏洞：该漏洞存在于`IOMobileFramebuffer`这个内核服务中。在`IOMobileFramebuffer::swap_submit(IOMFBSwap *)`这个函数中，因为没有对用户态传入的`IOMFBSwap`数据进行校验，从而导致内核堆溢出。利用该漏洞可以在沙盒内（不需要沙盒逃逸）直接对内核进行攻击，并完成非完美越狱。该漏洞在iOS 9.3.3盘古越狱（女娲石）中被使用。

## 0x04 iOS 9.3.2 公开的可利用漏洞

1. WebKit RCE `heapPopMin` 远程代码执行漏洞：因为WebKit模块中的`WebCore::TimerBase::heapPopMin()`存在内存破坏漏洞，利用该漏洞可以对iOS设备进行远程攻击。当用Mobile Safari浏览有恶意攻击代码的网页的时候，Safari将会被黑客控制。但要注意的是，被控制的仅仅是Safari，想要获取用户数据还需要进行沙盒逃逸，想要控制手机还需要对内核进行攻击。另外，因为WebKit不光存在于iOS中，因此该漏洞还被用于PS4、Kindle等设备的越狱。

2. `GasGauge` 条件竞争内核漏洞：该漏洞存在于`GasGauge`这个内核服务中，因为在`free`内存的时候没有进行加锁操作，黑客可以开多个线程进行`free`操作，当竞争成功的时候可以造成`double free`的漏洞，随后可以转化为任意zone的UAF并控制内核，并完成非完美越狱。需要注意的是，该内核服务并不能在沙盒内直接访问，所以想要利用该漏洞，需要先做到沙盒逃逸。

## 0x05 iOS 9.3.1 公开的可利用漏洞

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_ea765598dd39a771.jpg)

1. `inpuTbag` Heap overflow 内核漏洞：该漏洞是阿里移动安全的OverSky团队发现并公布的，该漏洞存在于`IOHIDDevice`这个内核服务中，因为没有对Input report的size做检测从而造成内核堆溢出。利用该漏洞可以对内核进行攻击，并完成非完美越狱。需要注意的是，该内核服务需要在沙盒外并拥有`com.apple.hid.manager.user-access-device`这个entitlement才能访问，所以想要利用该漏洞，需要先做到沙盒逃逸，然后绕过entitlement的检测才能利用。

## 0x06 iOS 9.1 公开的可利用漏洞

1. CVE-2015-7037 Photos 沙盒逃逸漏洞：该漏洞存在于`com.apple.PersistentURLTranslator.Gatekeeper`这个系统服务中，在盘古越狱中被使用，通过利用该漏洞，一个在沙盒内的App可以做到mobile权限的沙盒外任意文件读写，配合`dyld`的漏洞可以做到沙盒外的任意代码执行。

2. CVE-2015-7084 `IORegistryIterator` 内核漏洞：该内核漏洞存在于IOKit中，因为`IORegistryIterator`对象没有线程互斥的保护，导致对成员进行操作的时候可能出现错误。该漏洞可以在沙盒内直接通过`race condition`触发，随后转化为内核信息泄露以及内核的代码执行，并做到非完美越狱。

## 0x07 iOS 9.0 公开的可利用漏洞

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_95ddef44de51e2b9.png)

1. CVE-2015-6974 `IOHIDFamily` 内核漏洞：该漏洞存在于`IOHIDResource`这个内核服务中，在`terminateDevice`后，系统没有将`device`设置为`NULL`，从而造成UAF漏洞。该漏洞在盘古iOS 9.0越狱中被使用，利用该漏洞可以做到内核的任意读写，并完成非完美越狱。需要注意的是，该内核服务并不能在沙盒内直接访问，所以想要利用该漏洞，需要先做到沙盒逃逸。

## 0x08 总结

可以看到2016年的公开可利用的漏洞数量是非常巨大的，相对2015年可以说是有了一个指数级的增长。虽然苹果更新系统的速度非常快并且无法降级，但随着老设备（iPhone 4s及以下已无法升级iOS 10）越来越多，并且用户对新系统期望越来越低，iOS设备的更新率已经变得非常缓慢。

![配图](https://bbs.kanxue.com/upload/attach/201612/666663_116004e3e6d49d3f.png)

根据某专业移动分析平台2016年12月的数据可以看到，仅有3.28%的设备更新了最新版的iOS 10.2。这意味着96.72%的设备都有被最近刚发布的`mach_portal`漏洞攻击的风险。我们相信，在新的一年，iOS的漏洞数量还会持续增加，并且随着漏洞利用技术的公开，黑灰产也极有可能利用漏洞对用户进行攻击，希望广大用户一定要注意自己iOS设备的安全。

最后，对本文提到的漏洞感兴趣的同学可以在我们的GitHub上学习相关的资料：

```
6cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9r3g2F1k6$3#2A6L8U0p5&6z5o6W2Q4x3V1k6s2M7X3g2S2N6r3W2a6f1@1A6S2K9h3I4T1M7X3g2S2K9@1#2S2N6r3g2J5K9h3q4D9
```

更多阿里内部安全类技术文章及报告，请访问[阿里聚安全博客](https://bbs.kanxue.com/elink@b67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4q4Q4x3X3g2S2L8r3W2T1j5h3u0S2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8i4g2F1K9i4c8&6i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3@1`.`)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

