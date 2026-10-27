
# [原创]XAntiDenbug的检测逻辑与基本反调试

可能存在错误 大佬们可以的话给提示

## 入口相关代码

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_8HBTPKMZ86QB2XC.webp)

`XAntiDebug` 初始化获取当前模块地址 与设置了一个标识

```c
#define FLAG_FULLON (FLAG_CHECKSUM_NTOSKRNL | FLAG_CHECKSUM_CODESECTION | \
FLAG_DETECT_DEBUGGER | FLAG_DETECT_HARDWAREBREAKPOINT)
```

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_EDWX7V89G7F9MQ9.webp)

获取当前系统相关信息

```text
_isArch64 代表是64位架构
_isWow64  代表是64位环境运行32位程序
```

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_P7QAAJ6VNKHNA4Z.webp)

```text
NtSetInformationThread  设置线程的优先级
句柄-1 是进程的句柄 -2是当前线程的句柄
0x11 ThreadHideFromDebugger = 17,
wrk源码
```

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_J3PCKSEFVA3HWF9.webp)

会设置线程标识

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_HPKFMMUV5Z8JUTU.webp)

把调试端口置空

```text
然后继续调用但是传递了错误的地址 如果没调试器应该返回错误 调试器可能处理全部设置为0 证明其存在
```

## 运行`XAD_Initialize`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_BY552SKKJM6XK4J.webp)

添加调加校验`crc32`的初始值

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_73D9JU94E6SJ5PB.webp)

获取64位 `ZwQueryInformationProcess`的地址 然后减去`ntdll`64位模块地址 算偏移=`0x000000000009d370`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_GJRUSD2CBVANTSJ.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_8Z3EGSXCF3DVB7Z.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_RHHMGQ93D2MRDEC.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_R7K2EKNTDCESV7P.webp)

这段代码是直接分析ntdl pe结构 然后计算这个偏移 得到代码 直接得到文件中的代码

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_QE64JEKCGSD7BJJ.webp)

为了得到映射的下标 构造系统调用 代码目保证应用层的`NtQueryInformationProcess`正确

并使用`crc`保护自己的代码

## 然后调用`XAD_ExecuteDetect`进行检测

### 1 `crc` 检测 节表 我们的构造的系统调用 可以防止下代码段被修改

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_MHXS9JK5CZJH2QW.webp)

### 2 `IsDebuggerPresent` 检测当前是否正在被调试

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_JVGT95QGY6ZEWRE.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_668C754KRSF2FRE.webp)

`fs:[30h]`指向`PEB`的基址

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_A239T6YB2FAPRR5.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_HCD94MRA4PT4JNR.webp)

检测`BeingDebugged`的值

### 3 `CheckRemoteDebuggerPresent`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_8QHGR8HD2B4JXRE.webp)

实际上是查询`NtQueryInformationProcess` `ProcessDebugPort`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_VW9QP3AZQZ24QBC.webp)

如果`DebugPort`存在返回 `true`

### 4 关闭一个无效句柄 如果被调试会进入异常

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_ZSQWU3XRPKJ858N.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_7M699V67CPW52NY.webp)

`CloseHandle->NtClose` 如果存在`debuginfo`与`debugPort`抛出异常 `0xC0000008L`被调试器接管

### 5 使用`DuplicateHandle`进行检测

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_P4VKKSZ9QRB8G8P.webp)

我这边追流程 好像还是`DuplicateHandle...->ObDuplicateObject` 调用了`NtClose`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_EWKBC6CS5UXJ86B.webp)

### 6 检测`StrongOD`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_YXCAFQAPGZQF8GE.webp)

### 7 `NtQueryInformationProcess`检测 `0x1E` `ProcessDebugObjectHandle`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_ZNMGU78UQUTZVH8.webp)

`0x1E` `ProcessDebugObjectHandle`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_WPUN4V7EY3PMBQZ.webp)

`DbgkOpenProcessDebugPort`

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_QS6HP7Z7ZXPENN3.webp)

如果`DebugPort`未存在 返回 `STATUS_PORT_NOT_SET` 端口未设置

### 8 内核二次覆盖的BUG来检测反调试

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_YMQSNMZVS49AKRQ.webp)

这个没看懂和上面有区别吗

### 9 使用`GetThreadContext`检测 检测当前线程是否使用硬件断点

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_CC4J79GD6RWNXCB.webp)

### 10 使用`VEH`检测

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_K75CSVVD9X3Y2G4.webp)

添加`VEH` 到最前面

调用`HardwareBreakpointRoutine`触发异常 检测硬件断点

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_DSPKGK7HRNRK7Q3.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202403/941362_JUBGSUHDB33PCDY.webp)

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2024-3-2 16:54 被tian_chen编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm) [#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm) [#VM保护](https://bbs.kanxue.com/forum-4-1-4.htm)
