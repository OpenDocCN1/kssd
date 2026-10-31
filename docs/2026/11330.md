
# [原创]无"痕"加载驱动模块之漏驱利用(下)

x64上微软增加了驱动签名机制，我们的傀儡驱动依旧需要签名，虽然泄露的sha1签名目测来看能用到微软倒闭，或者花钱买一张白签名(干坏事会被吊销)但是依然存在问题，如果能让微软的白驱动“帮忙”加载我们的功能驱动就好了。

早期驱动开发安全意识不足，导致很多微软或第三方常用驱动存在可利用漏洞，像知名的永恒之蓝漏洞或者罗技驱动利用。

`kdmapper`也是如此

项目:[38fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1K9r3g2o6M7Y4g2K9i4K6u0r3K9$3c8E0j5i4m8H3k6i4t1`.](https://bbs.kanxue.com/elink@ff2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1K9r3g2o6M7Y4g2K9i4K6u0r3K9$3c8E0j5i4m8H3k6i4t1`.)

![](https://bbs.kanxue.com/upload/attach/202507/1001878_64Y57CAFXDVSXCM.jpg)

`kdmapper`加载驱动原理与傀儡驱动一致都是内存加载，不过傀儡驱动换成了漏驱自己，自带白签。

漏驱为`iqvw64e.sys`是英特尔(Intel)网卡设备的一个驱动程序文件，被放在资源中，我们把它扒出来进行分析。

![](https://bbs.kanxue.com/upload/attach/202507/1001878_R3VU5NE4ND7423S.jpg)

![](https://bbs.kanxue.com/upload/attach/202507/M73FG4DVG9RGJE2.jpg)

标准的驱动入口，没有任何壳或者加密，就差给pdb了。

![](https://bbs.kanxue.com/upload/attach/202507/JKJSTV3T2DTEDC7.jpg)

创建设备链接，记住名字等下要在3环建立连接

![](https://bbs.kanxue.com/upload/attach/202507/1001878_4R6M9PSGZQG5X7A.jpg)

![](https://bbs.kanxue.com/upload/attach/202507/4R9B36EM2HYP8R5.jpg)

注册了派遣函数对应

```
DriverObject->MajorFunction[IRP_MJ_CREATE] = CreateDriver;
DriverObject->MajorFunction[IRP_MJ_CLOSE] = CloseDriver;
DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = ControlDriver;
```

我们利用的函数在三个中

![](https://bbs.kanxue.com/upload/attach/202507/1001878_XKYAXK2XDVQ68US.jpg)

这里有四个分支，项目中表明了是利用第一个，我们就不挨个分析了

![](https://bbs.kanxue.com/upload/attach/202507/1001878_XKR2J8YME5KY7D6.jpg)

![](https://bbs.kanxue.com/upload/attach/202507/1001878_NK3T76WN7Z4JNSF.jpg)

进去第一个函数，根据上下文这里可以猜测`a1`是一个结构体指针

![](https://bbs.kanxue.com/upload/attach/202507/1001878_3Y524NPWJ2YBTGW.jpg)

`ReadMemory`中调用了`MemCopy`

![](https://bbs.kanxue.com/upload/attach/202507/1001878_QT3TG3Y4SZ3MX4U.jpg)

![](https://bbs.kanxue.com/upload/attach/202507/1001878_ZB8UZWEF9DZ3FM5.jpg)

对应漏驱中的`0x33`我们可以看到就是一个简单的拷贝内存函数，利用此函数将想要读的内容拷贝到缓冲区。(没想到这么一个小小的内存拷贝造成了任意地址读写的漏洞)，读和写就是反着来，互换下缓冲区地址。

同理推出结构体`a1+16`是要读的地址，`a1+24`是缓冲区地址，`a1+32`是要读的长度

漏驱中还有其他可以直接调用的内核函数不过和主线无模块加载关系不紧密，本文就不放了

这里是分析的申请内存函数的调用，其他函数同理

```
uint64_t AllocatePool(HANDLE device_handle, nt::POOL_TYPE pool_type, uint64_t size);
```

这里先自写了获取内核导出函数，再进行调用

[回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)

上传的附件：

- [回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)
