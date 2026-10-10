
## 特点
注入到 `explorer`，hook 函数 `NtQueryDirectoryFile` 隐藏快捷方式文件（这个时候你用 `explorer` 是看不到这个启动项的）。

## 木马行为特点
打开任务管理器，选中的一栏会不停的闪动，而且开任务管理后，木马会运行死循环来使 CPU 百分百。
