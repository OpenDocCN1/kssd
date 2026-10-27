
```text
.text:00000001401D2C30 83 E0 0F and eax, 0Fh ; 获取系统调用栈中参数的个数
.text:00000001401D2C33 0F 84 B7 00 00 00 jz KiSystemServiceCopyEnd
.text:00000001401D2C39 C1 E0 03 shl eax, 3
.text:00000001401D2C3C 48 8D 64 24 90 lea rsp, [rsp-70h] ; rsp指向\_KTrap\_Frame-70, 扩展栈, 用于填充额外参数
.text:00000001401D2C41 48 8D 7C 24 18 lea rdi, [rsp+70h+var\_58] ; 取填充位首地址
.text:00000001401D2C46 48 8B B5 00 01 00 00 mov rsi, [rbp+100h] ; \_KTrap\_Frame::Rsp
.text:00000001401D2C4D 48 8D 76 20 lea rsi, [rsi+20h] ; 用于取用户态栈帧上的参数
.text:00000001401D2C51 F6 85 F0 00 00 00 01 test byte ptr [rbp+0F0h], 1 ; \_KTrap\_Frame::SegCs
.text:00000001401D2C58 74 16 jz short loc\_1401D2C70
.text:00000001401D2C5A 48 3B 35 17 FA 1A 00 cmp rsi, cs:MmUserProbeAddress
.text:00000001401D2C61 48 0F 43 35 0F FA 1A 00 cmovnb rsi, cs:MmUserProbeAddress
.text:00000001401D2C69 0F 1F 80 00 00 00 00 nop dword ptr [rax+00000000h]
.text:00000001401D2C70
.text:00000001401D2C70 loc\_1401D2C70: ; CODE XREF: KiSystemServiceGdiTebAccess+71↑j
.text:00000001401D2C70 4C 8D 1D 79 00 00 00 lea r11, KiSystemServiceCopyEnd
.text:00000001401D2C77 4C 2B D8 sub r11, rax ; 根据栈中参数个数进行复制
.text:00000001401D2C7A 41 FF E3 jmp r11
.text:00000001401D2C7A KiSystemServiceGdiTebAccess endp
```

<a href="elink@6bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4j5h3&6Z5N6g2)9J5k6p5&6A6k6g2)9J5c8X3g2@1N6#2)9#2k6X3S2S2j5$3E0W2M7R3`.`.">etw_hacker</a> 运行截图及其调用栈如下所示：

![图片描述](upload/attach/202601/832354_QRKZX8V5HZTZGK9.webp)

![图片描述](upload/attach/202601/832354_M9UPWD9WNFAY49B.webp)

## Ref.

1. [[原创] 剖析 InfinityHook 原理 掀起一场更激烈的攻与防恶战](https://bbs.kanxue.com/thread-253450.htm)

2. <a href="elink@56eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3K9g2W2t1k6i4u0Q4x3V1k6u0L8X3k6A6L8X3W2@1P5f1S2G2L8$3E0b7M7X3!0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8R3`.`.">InfinityHookPro</a>

# 1. Silo

Windows内核中引入了 **Silo（服务隔离对象，Service Isolation Object）** 机制，用于实现细粒度资源隔离。Silo 最初在 Windows Server 2019 和 Windows 10 后期版本中逐步引入，并在后续版本（如 Windows 11 和 Windows Server 2022）中进一步完善。

- Silo 是 Windows 内核中用于隔离进程、线程、对象命名空间、安全上下文等的一种容器机制。
- 每个 Silo 拥有独立的全局状态（如对象目录、注册表视图、安全令牌策略等）。
- 主机 Silo（Host Silo）是默认的系统 Silo，而容器化场景（如 Windows 容器、WSL2、Azure Host OS）会创建额外的 Silo。

## 1.1 PspHostSiloGlobals

全局变量 `PspHostSiloGlobals` 指向一个类型为 `_ESERVERSILO_GLOBALS` 的结构体，描述主操作系统 Silo（即传统 Windows 环境）。其中，`PspHostSiloGlobals::EtwSiloState` 为描述 ETW 子系统的 Silo。

```text
nt!_ESERVERSILO_GLOBALS
+0x000 ObSiloState      : _OBP_SILODRIVERSTATE // 对象管理器（Object Manager）的 Silo 状态
+0x2e0 SeSiloState      : _SEP_SILOSTATE // 安全子系统（Security Reference Monitor）的 Silo 状态
+0x310 SeRmSiloState    : _SEP_RM_LSA_CONNECTION_STATE // LSA（Local Security Authority）连接状态
+0x360 EtwSiloState     : 0xffffa908459c1000 _ETW_SILODRIVERSTATE // ETW
+0x368 MiSessionLeaderProcess : 0xffffa908478ad400 _EPROCESS // 内存管理器（Mm）的会话领导者进程: smss.exe（Session Manager Subsystem）
// 作为会话内存配额和页面文件分配的参考点
+0x370 ExpDefaultErrorPortProcess : 0xffffa90847c424c0 _EPROCESS // 默认硬错误（Hard Error）接收进程: 默认硬错误（Hard Error）接收进程
// csrss.exe 或 svchost.exe (DcomLaunch)
// 当应用崩溃时：错误对话框由该进程弹出
+0x378 ExpDefaultErrorPort : 0xffffa90847593d30 Void // 指向 LPC（Local Procedure Call）端口对象
// 内核通过此端口向 ExpDefaultErrorPortProcess 发送崩溃通知
+0x380 HardErrorState   : 1 // 硬错误处理状态 0:禁用硬错误（如无头服务器）; 1:启用（桌面系统默认）
// 1 → 系统会弹出“xxx 已停止工作”对话框
+0x388 WnfSiloState     : _WNF_SILODRIVERSTATE // Windows Notification Facility (WNF) 的 Silo 状态
+0x3c0 DbgkSiloState    : _DBGK_SILOSTATE // 内核调试器（Debugging Subsystem）的 Silo 状态
+0x3e0 PsProtectedCurrentDirectory : _UNICODE_STRING "C:\Windows" // 受保护进程（Protected Process Light, PPL）的当前目录
// 防止恶意 DLL 注入到 PPL（如 lsass.exe, MsMpEng.exe）
+0x3f0 PsProtectedEnvironment : _UNICODE_STRING "Path=C:\Windows\System32" // 受保护进程的环境变量: 限制 PPL 加载非系统路径的 DLL，提升安全性
+0x400 ApiSetSection    : (null)
+0x408 ApiSetSchema     : (null)
+0x410 OneCoreForwardersEnabled : 0 '' // 是否启用 OneCore API 转发（用于通用 Windows 平台）
+0x418 NtSystemRoot     : _UNICODE_STRING "C:\Windows" // 系统根目录
+0x428 SiloRootDirectoryName : _UNICODE_STRING ""
+0x438 Storage          : 0xffffa90845874a00 _PSP_STORAGE // Silo 的私有存储区
+0x440 State            : 1 ( SERVERSILO_STARTED ) // Silo 生命周期状态
+0x444 ExitStatus       : 0n259 // Silo 退出码
+0x448 DeleteEvent      : (null) // Silo 销毁完成事件
+0x450 UserSharedData   : 0xffffa908458742b0 _SILO_USER_SHARED_DATA // 用户模式共享数据区（User-Shared Data）的 Silo 版本
+0x458 UserSharedSection : (null) // 用户共享数据的节对象（Section Object）
+0x460 TerminateWorkItem : _WORK_QUEUE_ITEM // Silo 终止时的延迟工作项
```

## 1.2 EtwpHostSiloState

`_ETW_SILODRIVERSTATE` 用于记录**Silo（Windows 容器/Job 隔离环境）中的 ETW 全局状态**，包含该 Silo 内的所有 GUID 注册表、Logger 列表等。

```text
nt!_ETW_SILODRIVERSTATE EtwpHostSiloState
+0x000 Silo             : Ptr64 _EJOB // 主操作系统 Silo（即非容器、非 Job 隔离的传统 Windows 环境）
+0x008 SiloGlobals      : 0xfffff80022a47540 _ESERVERSILO_GLOBALS // 全局内核对象表（如进程、线程、注册表等命名空间）, 即 PspHostSiloGlobals
+0x010 MaxLoggers       : 0x40 // 当前 Silo 允许的最大 ETW Logger 数量
+0x018 EtwpSecurityProviderGuidEntry : _ETW_GUID_ENTRY
+0x1a8 EtwpLoggerRundown : 0xffffa908459c2900 -> 0xffffa90845850510 _EX_RUNDOWN_REF_CACHE_AWARE // 用于 Logger 引用计数与安全析构
+0x1b0 EtwpLoggerContext : 0xffffa908459c2b00 -> 0xffffa90849a4c040 _WMI_LOGGER_CONTEXT // 指向 LoggerTable[LoggerId]
+0x1b8 EtwpGuidHashTable : [64] _ETW_HASH_BUCKET // 可通过 hash_table 计算 guid 快速获取 GuidEntry, 例如函数 EtwpFindGuidEntryByGuid
+0xfb8 EtwpSecurityLoggers : [8] 3
+0xfc8 EtwpSecurityProviderEnableMask : 0x3 ''
+0xfcc EtwpShutdownInProgress : 0n0 // Silo 关闭标志
+0xfd0 EtwpSecurityProviderPID : 0x344 // lsass.exe （Local Security Authority Subsystem Service） 是当前注册安全提供者的进程
// LSASS 负责处理本地安全和登录管理等关键任务
+0xfd8 PrivHandleDemuxTable : _ETW_PRIV_HANDLE_DEMUX_TABLE // 私有句柄多路复用表. 允许用户态通过 OpenTrace 获取 Logger 句柄; 维护句柄到 _WMI_LOGGER_CONTEXT 的映射.
+0xff8 EtwpCounters     : _ETW_COUNTERS // ETW 性能计数器集合
+0x1008 LogfileBytesWritten : _LARGE_INTEGER 0x17d1800
+0x1010 ProcessorBlocks  : 0xffffa908459c2240 _ETW_SILO_TRACING_BLOCK
+0x1018 ContainerRestoreWnfSubscription : 0xffffe2832a98ac40 _EX_WNF_SUBSCRIPTION
+0x1020 PartitionId      : _GUID {00000000-0000-0000-0000-000000000000}
+0x1030 ParentId         : _GUID {00000000-0000-0000-0000-000000000000}
+0x1040 QpcOffsetFromRoot : _LARGE_INTEGER 0x0
+0x1048 PartitionType    : 0 // Silo 分区类型
// 0: 主系统; 1:WSL2; 2:Windows Server Container; 3: Job Object Isolation.
+0x104c SystemLoggerSettings : _ETW_SYSTEM_LOGGER_SETTINGS // 系统级 Logger 全局配置
+0x11c0 EtwpStartTraceMutex : _KMUTANT
```

# 2. ETW 机制

## 2.1 Logger

Logger 是一个 ETW tracing session（跟踪会话），负责收集来自一个或多个 Provider 的事件，实现日志的收集与存储。Logger 对象的关键成员为 `LoggerId/InstanceGuid/SiloState`。其中，`LoggerId` 描述当前 Logger 位于 `EtwpLoggerContext` 中的索引——`EtwpHostSiloStat::EtwpLoggerContext[LoggerId]`；`SiloState` 指向当前所属 Silo 对象。
