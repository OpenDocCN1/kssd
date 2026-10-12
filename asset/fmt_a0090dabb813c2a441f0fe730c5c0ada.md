
# Rootkit 系统调用表防护与检测欺骗

## 系统调用表重建与 Trampolining

有一些 `anti-rootkit` 应用程序可以重建系统调用表。可以通过从原始文件 `ntoskrnl.exe` 重新初始化内核内存的方法来实现。如果系统调用表在我们的 `rootkit` 安装后被重建，那么所有的钩子将会失效。为了防止这种可能性发生，新的 `rootkit` 跟着表入口来到实际函数并修复函数自身使他们跳转到各自的 `rootkit` 子程序。这个技术被叫做“trampolining（蹦床）”，并被用作进程注入，第四章将详细介绍。

## 欺骗检测与挂钩策略

当然，新的 `anti-rootkit` 程序也可以跟随表入口来到真实函数并恢复被更改的函数，因此你的 `rootkit` 需要一个方法来欺骗系统调用表的检查，或者欺骗 `ntoskrnl.exe` 的加载。因为在真实内核函数中增加跳转会使 `anti-rootkit` 软件更难清除你的挂钩，增加更多的挂钩来阻止 `anti-rootkit` 软件查看真实的系统调用表（或真实的 `ntoskrnl.exe` 内容）是唯一被认可的方法。

当然，欺骗系统调用表检测的方法是被推荐的。这是因为 `anti-rootkit` 软件可能只是基于 Windows 操作系统众多版本中某一个固定版本的 `ntoskrnl.exe`。如果 `anti-rootkit` 所调用相应版本的 `ntoskrnl.exe` 的文件名改变了，那么挂钩 `ZwOpenFile` 来查找 `ntoskrnl.exe` 将找不到它！

## 实现欺骗检测的具体步骤

为了欺骗系统调用表的差检测，你需要挂钩 `MmCreateMdl` 并查找系统调用表(`KeServiceDescriptorTable.ServiceTableBase`)的基地址。你还应该挂钩 `ZwOpenSection` 并使用 `\device\physicalmemory` 查找 `OBJECT_ATTRIBUTES`。如果有进程打开物理内存并写入系统调用表，那么可以确定你的挂钩将存在被清除的危险。当然也有可能是另外一个 `rootkit` 在运行。

## 总结

我们现在已经拥有了一个具有如下功能的 `rootkit`：

- 隐藏自身设备驱动条目
- 隐藏自身配置文件
- 挂钩操作系统内核

内核钩子可以提供 `rootkit` 所需要的大部分函数功能，但是实现一个完整功能的 `rootkit` 还需要一些其他的技术参与。下章增加另一个 `rootkit` 中至关重要的部分：进程注入。
