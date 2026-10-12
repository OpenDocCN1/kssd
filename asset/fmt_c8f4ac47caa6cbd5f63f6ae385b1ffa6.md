
# 检测程序是否运行在 VMware 下（VMware has you）

> 原文：[`www.pediy.com/kssd/pediy10/57751.html`](https://www.pediy.com/kssd/pediy10/57751.html)

当你找到一个新的病毒时，我们要分析它，我们需要知道它是如何在复杂的网络环境下扩散的。如何在网络中传播，如何感染其他文件，最好的解决办法就是使用虚拟机来解决，让我们在一台机器里，虚拟出多个操作平台。这也就不会使得病毒在你自己的机器中真实的传播起来。这样也带来了一个问题：病毒如何发现自己在虚拟的机器中？

比如在虚拟机软件 VMware 中，它有一些后台端口来确定自己的虚拟设备和与真实的主机发送消息，当然有一些程序调用 API 提供给你来使用。

它是这样调用的：

```
mov     ecx, 0Ah     ; CX=function# (0Ah=get_version)
mov     eax, 'VMXh' ; EAX=magic
mov     dx,  'VX'   ; DX=magic
in      eax,  dx     ; specially processed io cmd
; output: EAX/EBX/ECX = data
cmp     ebx,  'VMXh' ; also eax/ecx modified (maybe vmw/os ver?) <===注意
je      under_VMware
```

VMware 下注册表会建立：

`HKLM\Software\VMware, Inc.\VMware for Windows NT` -- 真实主机（不在 VM 下但是机器安装了 VM）
`HKLM\Software\VMWare, Inc.\VMware Tools\` -- 虚拟主机下

VMware 的目录是

`C:\Program Files\VMware` -- 这说明机器有 VM 但无法确定是否是虚拟机下

当然还有许多其他的方法来确定，比如：incorrectly emulated ports, predetermined hardware info, special drivers 等等。

About actions to be performed under virtual OS, well, it depends on your wicked souls -- from fucking up everything。

原作者：z0mbie/29A
