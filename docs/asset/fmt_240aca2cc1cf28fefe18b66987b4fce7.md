
`Real_MessageBox=(MessageBoxSys)DetourFunction((PBYTE)SystemMessageBox, (PBYTE)MessageBox_Mine);`

在 `Dll` 入口函数中的卸载 `Dll` 事件中加入：

`DetourRemove((PBYTE)Real_Messagebox, (PBYTE)MessageBox_Mine);`

### 重写二进制可执行文件

使用 `Detours` 自带的 `SetDll.exe` 重写二进制可执行文件，可以在需要截获的程序中加入一个新的 `Detours` 的 `PE` 节表。对于本文就是新建一个批处理文件调用 `SetDll.exe`。

```
@echo off
if not exist MessageBoxApp.exe (
echo 请将文件解压到 MessageBoxApp.exe 的安装目录, 然后执行补丁程序
) else (
setdll /d:ApiHook.dll MessageBoxApp.exe
)
Pause
```

调用后使用 `depends.exe`（微软 `VC6.0` 开发包的工具之一）观察 `MessageBoxApp.exe` 前后变化，可以看到 `Setdll` 已经重写 `MessageBoxApp.exe` 成功，加入了对 `ApiHook.dll` 的依赖关系。

![www.xajiahe.com/cc/crack_05.jpg](img/5e758b53cdfe1ded3699c5b09bf367d8.jpg)
&nbsp;&nbsp;&nbsp;&nbsp;（执行 `SetDll.exe` 前）&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(执行 `SetDll.exe` 后)

执行 `SetDll.exe` 重写后的 `MessageBoxApp.exe`，点击确定后可以看到结果如下：

至此，`MessageBoxApp.exe` 对 `MessageBoxA` 函数的调用已经被截获，弹出的对话框内容已经明显说明这一点。

![www.xajiahe.com/cc/crack_06.jpg](img/9871472f9498eb720dc405d1e291cd4c.jpg)

`Detours` 早知道了 很好的引擎 从 `1.5` 版就开始用
可惜 `2.2` 只是部分开源了 `X64` 和 `IA32` 的 都不是开源的
不过这个库我个人认为最大的用处还是他的指令反汇编引擎的 `ApiHook Lib` 用于 Hook `Send` 之类的函数再好不过了

`4dd466d0e1e0cc123afbe504da2462ca`
