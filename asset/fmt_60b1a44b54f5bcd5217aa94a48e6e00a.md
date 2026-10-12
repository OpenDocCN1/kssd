
# Windows Mobile 强制关机重启方法总结

> 原文：[`www.pediy.com/kssd/pediy10/79602.html`](https://www.pediy.com/kssd/pediy10/79602.html)

强制关机或重启有很多用处。随便总结如下：
最美妙的用处：可以吓唬一下办公室的小妹，当她哭鼻子来找你的时候你就可以英雄救美了！![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
最强悍的用处：你写的是驱动级病毒，需要用户重启，又等不及了。就强制让她重启吧！![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")  
在研究中我们总结了 3 种强制关机和一种强制重启的方法。

## 关机方式
01、`GwesPowerOffSystem` 函数没有参数很好用。  
02、`PowerOffSystem` 函数没有参数很好用。  
03、`SetSystemPowerState` 函数有 3 个参数可以关机。

## 重启方式
01、`SetSystemPowerState` 函数有 3 个参数可以重启。

## 总结
在模拟器上全部成功。在不同品牌和型号的手机上没有试过，估计不会全部成功，要看手机厂商对系统安全的设置。如果设置很严格有些 API 调用会受到限制，必须签署才能运行。没有发代码因为代码使用 Xarm 写的。查查 MSDN 应该非常容易找到这 3 个 API。发 Xarm 写的应用程序可以实验：

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 强制关机或重启示例程序.rar |
