
# 第四层：Hypervisor 级 Hook（Ring -1）— 极致隐蔽

这是 Windows 平台上隐蔽性最强的一层。Hypervisor 运行在所有软件的下面（包括 Windows 内核），拥有对物理内存、CPU 状态、I/O 的完全控制权。在这个层面实施的 Hook，操作系统本身看到的所有信息都可以被伪造。

---

## 4.1 EPT Hook（扩展页表 Hook）— 读写/执行分离

### 原理

EPT（Extended Page Table）是 Intel VT-x 提供的第二层地址翻译。Guest 的物理地址（GPA）通过 EPT 映射到实际的主机物理地址（HPA）。EPT 的每个条目都有独立的 Read/Write/Execute 权限位。

核心思想：**对同一个 GPA，让读写操作映射到干净原始页，让执行操作映射到包含 Hook 代码的页**。

### EPT 条目结构

```
typedef union _EPT_PTE {

    ULONG64 Value;

    struct {

        ULONG64 ReadAccess : 1;

        ULONG64 WriteAccess : 1;

        ULONG64 ExecuteAccess : 1;

        ULONG64 MemoryType : 3;        // 0=UC, 6=WB

        ULONG64 IgnorePat : 1;

        ULONG64 LargePage : 1;         // 2MB/1GB 大页

        ULONG64 Accessed : 1;

        ULONG64 Dirty : 1;

        ULONG64 UserModeExecute : 1;   // MBEC

        ULONG64 Reserved1 : 1;

        ULONG64 PhysicalAddress : 40;  // 物理页帧号

        ULONG64 Reserved2 : 11;

        ULONG64 SuppressVE : 1;        // #VE 抑制

    };

} EPT_PTE, *PEPT_PTE;

typedef union _EPTP {

    ULONG64 Value;

    struct {

        ULONG64 MemoryType : 3;        // EPT 页表自身的内存类型

        ULONG64 PageWalkLength : 3;    // 页表遍历深度-1 (3 = 4级)

        ULONG64 DirtyAndAccessEnabled : 1;

        ULONG64 Reserved1 : 5;

        ULONG64 PML4PhysicalAddress : 40;

        ULONG64 Reserved2 : 12;

    };

} EPTP;
```

### 完整实现（从 Hypervisor 初始化到 Hook 安装）
