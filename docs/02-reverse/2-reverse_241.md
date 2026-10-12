
# 关于 SoftICE 和 Demon-Tools 冲突的解决办法

> 原文：[`www.pediy.com/kssd/pediy12/136238.html`](https://www.pediy.com/kssd/pediy12/136238.html)

Demon-Tools 从 4.0 以后启用了 `sptd` 虚拟设备，该虚拟和 `ntice` 相同，很多人都是通过卸载 `dt` 来使用 `softice`，但是我经过研究以后发现其实这是没有必要的，只要让该设备启动的时候不加载就可以了

Windows Registry Editor Version 5.00

```
[HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\sptd\]

"Start"=dword:00000000
```

将 `start` 改成 `"Start"=dword:00000001` 就可以使用了![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")
