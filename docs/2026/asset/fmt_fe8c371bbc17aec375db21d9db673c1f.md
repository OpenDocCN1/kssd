
通过 PCIe/Thunderbolt 设备的 DMA（Direct Memory Access）能力，直接读写主机物理内存，完全绕过 CPU 的所有保护机制。不需要在目标系统上执行任何代码。

### 攻击设备

- PCIe 恶意设备（伪装成网卡等）
- Thunderbolt 外接设备
- FireWire 设备（旧系统）
- 恶意 NIC 固件（如 Intel AMT 漏洞利用）

### 实现概述

```c
// 在恶意 PCIe 设备的固件中：
void DmaHookInstall() {
// 扫描物理内存，定位 Windows 内核
PHYSICAL_ADDRESS ntBase = ScanForKernelBase();
// 定位目标函数（通过特征码匹配）
PHYSICAL_ADDRESS targetFunc = FindFunctionBySignature(ntBase);
// 通过 DMA 直接写入物理内存
// 绕过 CPU 的 MMU/EPT，没有任何权限检查
DmaWrite(targetFunc, hookCode, hookCodeSize);
// 或者更隐蔽：修改页表来实现类似 PTE Hook 的效果
PHYSICAL_ADDRESS pteAddr = CalculatePtePhysAddr(targetFunc);
DmaWrite(pteAddr, &modifiedPte, sizeof(MMPTE));
}
```

### 防御

- IOMMU (VT-d / AMD-Vi)：为 DMA 设备创建独立地址空间
- Kernel DMA Protection (Windows 10 1803+)
- Thunderbolt Security Level 设置

### 检测难度：★★★★★（无 IOMMU 时不可检测）

---

## 5.3 BIOS/UEFI Rootkit

### 原理

在 UEFI 固件中植入恶意代码。由于 UEFI 在操作系统之前执行，可以在 OS 加载前修改任何数据（包括 bootloader、内核加载器），绕过内核保护。即使重装系统、更换硬盘也无法清除。

### 已知案例

- **LoJax (2018)**：第一个野外发现的 UEFI rootkit
- **MosaicRegressor (2020)**：针对外交官的 UEFI 植入物
- **CosmicStrand (2022)**：修改 UEFI 固件的持久化攻击
- **BlackLotus (2023)**：绕过 Secure Boot 的 UEFI bootkit

### 攻击流程

```
1. 获取 SPI Flash 写入权限（利用固件漏洞或物理访问）
2. 在 UEFI DXE 阶段植入恶意模块
3. 恶意模块在 OS 加载时注入代码到 Windows Boot Manager
4. Boot Manager 加载内核时注入代码到内核加载器
5. 加载的内核带有预装 Hook（此时 Secure Boot/PatchGuard 尚未初始化）
```

### 检测难度：★★★★★

- 重装系统无法清除
- 操作系统层面几乎不可检测
- 需要专用固件扫描工具（如 CHIPSEC、UEFI Toolkit）
- Secure Boot 可以阻止未签名固件（但有时也能被绕过）

---

# 总览对比表

| # | 技术 | 层级 | 隐蔽性 | PatchGuard | 修改目标代码 | 适用场景 |
|---|------|------|--------|------------|-------------|---------|
| 1 | IAT Hook | Ring 3 | ★☆☆☆☆ | N/A | 否（改表） | 最基础的用户态拦截 |
| 2 | EAT Hook | Ring 3 | ★☆☆☆☆ | N/A | 否（改表） | 劫持动态获取的函数 |
| 3 | Inline Hook | Ring 3 | ★★☆☆☆ | N/A | **是** | 通用函数 Hook |
| 4 | VEH/HWBP Hook | Ring 3 | ★★★☆☆ | N/A | 否 | 反调试场景（4个限制） |
| 5 | PAGE_GUARD Hook | Ring 3 | ★★☆☆☆ | N/A | 否 | 概念性方案（性能极差） |
| 6 | TLS Callback | Ring 3 | ★★☆☆☆ | N/A | 否 | 早期初始化 Hook |
| 7 | Hotpatch Hook | Ring 3 | ★★☆☆☆ | N/A | 部分 | 利用微软预留空间 |
| 8 | Instrumentation Callback | Ring 3 | ★★★☆☆ | N/A | 否 | 全局 syscall 返回拦截 |
| 9 | Syscall Stub Patch | Ring 3 | ★★☆☆☆ | N/A | **是** | 对抗直接 syscall |
| 10 | SetWindowsHookEx | Ring 3 | ★☆☆☆☆ | N/A | 否 | 全局消息钩子 + DLL 注入 |
| 11 | AppInit_DLLs | Ring 3 | ★☆☆☆☆ | N/A | 否 | 注册表全局 DLL 注入 |
| 12 | IFEO | Ring 3 | ★★☆☆☆ | N/A | 否 | 映像劫持 / Verifier 注入 |
| 13 | IME 注入 | Ring 3 | ★★☆☆☆ | N/A | 否 | 输入法 DLL 加载到 GUI 进程 |
| 14 | Shim Engine | Ring 3 | ★★★☆☆ | N/A | **是**（内存） | SDB 内存补丁 / DLL 注入 |
| 15 | COM Hijacking | Ring 3 | ★★★☆☆ | N/A | 否 | 注册表 COM 对象劫持 |
| 16 | Winsock LSP | Ring 3 | ★★☆☆☆ | N/A | 否 | 全局网络流量拦截 |
| 17 | DLL Hijacking | Ring 3 | ★★☆☆☆ | N/A | 否 | DLL 搜索顺序劫持 |
| 18 | SSDT Hook | Ring 0 | ★★☆☆☆ | **必杀** | 否（改表） | XP/7 时代主流 |
| 19 | IDT Hook | Ring 0 | ★★☆☆☆ | **必杀** | 否（改表） | 拦截中断/异常 |
| 20 | IRP Hook | Ring 0 | ★★★☆☆ | 不直接 | 否（改指针） | 文件/设备过滤 |
| 21 | DKOM | Ring 0 | ★★★☆☆ | 部分检测 | 否（改链表） | 隐藏进程/驱动 |
| 22 | MSR Hook | Ring 0 | ★★☆☆☆ | **监控** | 否（改MSR） | 全局 syscall 拦截 |
| 23 | GDT/Call Gate | Ring 0 | ★★☆☆☆ | **监控** | 否（改描述符） | Ring3→Ring0 跳板 |
| 24 | Kernel Callback | Ring 0 | ★☆☆☆☆ | 不触发 | 否 | 官方合法 API |
| 25 | Infinity Hook | Ring 0 | ★★★★☆ | 不触发 | 否（改ETW指针） | 绕 PG 拦截所有 syscall |
| 26 | PG Bypass + Inline | Ring 0 | ★★★☆☆ | 绕过 | **是** | 高风险但有效 |
| 27 | WFP Callout | Ring 0 | ★☆☆☆☆ | 不触发 | 否 | 网络数据包过滤 |
| 28 | PTE Hook | Ring 0 | ★★★★☆ | 部分检测 | 否（改PTE） | 代码页重定向 |
| 29 | **EPT Hook** | **Ring -1** | **★★★★★** | **无效** | 否 | **终极内核 Hook** |
| 30 | **VMFUNC EPTP Switch** | **Ring -1** | **★★★★★+** | **无效** | 否 | 零 VM-Exit 视图切换 |
| 31 | EPT + MSR Bitmap | Ring -1 | ★★★★★ | **无效** | 否 | 代码+数据双伪装 |
| 32 | SMM Hook | Ring -2 | ★★★★★★ | **无效** | 固件级 | 固件级后门 |
| 33 | DMA Attack | 硬件 | ★★★★★★ | **无效** | 物理级 | 外部设备直接改内存 |
| 34 | UEFI Rootkit | 固件 | ★★★★★★ | **无效** | 固件级 | 持久化后门 |

---

# 技术演进路线图

```
2001-2006 (XP 时代 - 无保护)
│ IAT/EAT Hook → Inline Hook → SSDT Hook → IDT Hook
│ → 没安全机制，想做什么做什么
│
2006-2012 (Vista/7 - PatchGuard 登场)
│ PatchGuard v1 出现 → SSDT/IDT Hook 阵亡
│ DKOM 兴起 → MSR Hook → PG 绕过混战
│ → 猫鼠游戏开始
│
2012-2015 (Win8/8.1 - DSE + PG 强化)
│ 驱动签名强制 → BYOVD 攻击兴起
│ Infinity Hook → Kernel Callback 大行其道
│ → 合法接口开始吃香
│
2015-2018 (Win10 - HVCI + VBS)
│ HVCI 禁止动态代码执行 → EPT Hook 崛起
│ PTE Base 随机化 → PTE Hook 难度上升
│ → 军备竞赛进入虚拟化时代
│
2018-2022 (鼎盛期)
│ EPT Hook 框架成熟（HyperHide、hvpp、SimpleSvm）
│ VMFUNC EPTP Switching 实用化
│ 反作弊开始部署 Hypervisor 检测
│ → 攻守双方对等博弈
│
2022-2026 (当前)
│ VBS + Credential Guard 全面部署
│ 虚拟化嵌套检测
│ 时序侧信道对抗
│ UEFI Rootkit 商业化（BlackLotus）
│ → 战场延伸到固件层
│
未来趋势：
│ Intel TDX / AMD SEV → 硬件强制隔离
│ ARM CCA (Confidential Compute Architecture)
│ 当硬件不再信任软件 → Hypervisor Hook 的末日？
```

---

# 结语

Windows Hook 技术经过 20+ 年的进化，已经从简单粗暴的 IAT 修改，发展到需要理解 Intel VT-x 手册才能实现的 EPT 量子级技术。每一道微软加固的防线，都催生了更底层的一次突破：

- 用户态被拦 → 进内核
- 内核被 PatchGuard 监控 → 绕过 PatchGuard
- 内核结构都不能碰了 → 去 Hypervisor
- Hypervisor 被检测 → 去 SMM / UEFI

**核心公理：谁控制了更底层的抽象，谁就拥有绝对控制权。** 上层的任何检测手段都可以被下层伪造——这就是为什么 EPT Hook 之后在纯软件层面几乎不可检测的根本原因。

---

# Windows 全平台 Hook 技术完整图谱：从入门到无敌

> 从最容易被检测的用户态 Hook，到理论上无法被静态发现的 Hypervisor 级 Hook，按隐蔽性从低到高排列。每种技术均提供完整可编译的实现代码。

---

# 第一层：用户态 Hook（Ring 3）

用户态 Hook 是最基础的拦截手段，所有代码运行在应用层，任何反作弊/安全软件只要扫描进程内存就能发现。隐蔽性最低，但开发成本也最低。

---

## 1.1 IAT Hook（导入地址表 Hook）

### 原理

PE 文件加载时，Loader 会填充 IAT（Import Address Table），记录每个导入函数的实际地址。IAT Hook 直接修改这张表里的函数指针，让程序调用时跳到你的函数。

### 完整实现
