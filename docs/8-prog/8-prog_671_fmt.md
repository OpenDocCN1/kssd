
# 娱乐,扩展一下 ProcessNotify~~

> 原文：[`www.pediy.com/kssd/pediy10/80109.html`](https://www.pediy.com/kssd/pediy10/80109.html)

这不是一个什么新话题了,但是 DIY 一下还是蛮有意思,纯属娱乐~~

大家都知道,`Win2000`,`WInXp`,`Win2003` 下都只能安装 8 个 `Notify`(`Vista` 下听说是 32 个,看来够用了),多了不行.因为有个先来后到的问题,后来的坑满了就蹲不进去了,所以有时候我们为了跟别人和平共处,扩展一下还是有必要的.本代码只在 `WinXP SP2` 下测试通过,因为测试条件有限,所以没打算支持 `Vista`,重在思路,重在实践!我把代码附上,如果谁愿意改成支持的也行~

上张图,27 个 `Notify` 都正常工作:

![](img/365b11ae13a744a299e89590b109538c.jpg)

附件中的 `testNotify.sys` 是专们用来测试的,它会注册 9 个 `NotifyRoutine`~~

你可以把它复制多份来加载进行测试,打印的消息前面会有一个数字作为标识来区分

你可以先加载 `ExtendNotify.sys` 来完成扩展,然后注册多少个都可以~~卸载当然也是可以的~

或者你先加载 `testNotify.sys` 把 8 个坑占满再加载 `ExtendNotify.sys`,依然可以正常扩展~~

实际上,上面那张图就是在虚拟机中 8 个坑全满的情况下再进行扩展的结果.

代码附上,有兴趣的自己看,有错误请指出~~

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 扩展 ProcessNotify\_achillis.rar  
|
