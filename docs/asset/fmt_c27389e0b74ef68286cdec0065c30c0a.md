
# 修改 vmkd 支持 VMware Workstation

> 原文：[`www.pediy.com/kssd/pediy10/63777.html`](https://www.pediy.com/kssd/pediy10/63777.html)

`vmkd` 可以大大提高内核调试虚拟机的反应速度，但 `VMware server` 用起来很不方便，不支持直接从主机拖放文件，每次都得从网络上传。所以花了点时间把 `vmkd` 修改了一下，现在支持在 `VMware Workstation` 上使用。`vmkd` 的原理可以去作者网站上查看，大概是截获内核调试引擎和远程调试器通信的过程，用快速的传输数据方法替换了原来的模拟串口方法，从而加速了内核调试时的响应速度。传输数据时用到了 `VMware` 提供的 `0x12` 后门指令，直接和主机上的 `vmware-vmx.exe` 进程通信，把需要传输的数据存放到某个内存，由注入到 `vmware-vmx.exe` 进程的 `vmxpatch.dll` 读取出来后通过新的管道发给调试器。`0x12` 后门指令由 `vmware-vmx.exe` 进程中的某一段代码负责响应，`vmxpatch.dll` 注入时会通过特征码查找这一段代码，因为 `VMware Workstation` 和 `VMware server` 这一段代码的特征码不一样，所以 `vmkd` 查找失败，不支持 `VMware Workstation`。把 `vmxpatch.dll` 中的这段特征码修改一下就能实现对 `VMware Workstation` 的支持。

![](img/fec219760138db2a3cae3d350b6a1a3c.jpg)

![](img/3147e30b2e6775891e006bedd7e31c2a.jpg)

修改后的 `vmxpatch.dll` 和之前的文件有这两个差异。

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) | `vmxpatch.zip` |
| :--- | :--- |
