
# 如何在 Windows 程序中读取 BIOS 内容

> 原文：[`www.pediy.com/kssd/pediy05/pediy50415.htm`](https://www.pediy.com/kssd/pediy05/pediy50415.htm)

**标题**：如何在 Windows 程序中读取 BIOS 内容（物理内存内容）  
**作者**：火翼\[CCG\]  
**组织**：\[CCG\] (China Cracking Group)

**源程序下载**

今天和夜月兄讨论了一下在 Windows NT/2000/XP 下如何读取 BIOS 信息，现在把结果向大家汇报一下。

大家都知道，Windows 接管了对物理内存的直接存取，而 BIOS 信息存在物理内存的 `f000:0000` 处，关键就是如何读取物理内存。

查阅了 MSDN 的文章后，发现以下有几个函数和物理内存访问有关：

```c
NTSTATUS ZwOpenSection(OUT PHANDLE SectionHandle, IN ACCESS_MASK DesiredAccess, IN POBJECT_ATTRIBUTES ObjectAttributes);

NTSTATUS ZwMapViewOfSection(
    IN HANDLE SectionHandle,
    IN HANDLE ProcessHandle,
    IN OUT PVOID *BaseAddress,
    IN ULONG ZeroBits,
    IN ULONG CommitSize,
    IN OUT PLARGE_INTEGER SectionOffset OPTIONAL,
    IN OUT PSIZE_T ViewSize,
    IN SECTION_INHERIT InheritDisposition,
    IN ULONG AllocationType,
    IN ULONG Protect
);

NTSTATUS ZwUnmapViewOfSection(IN HANDLE ProcessHandle, IN PVOID BaseAddress);
```

用到的结构定义如下：

```c
typedef struct _UNICODE_STRING {
    USHORT Length; // 长度
    USHORT MaximumLength; // 最大长度
    PWSTR Buffer; // 缓存指针，访问物理内存时，此处指向 UNICODE 字符串 "\\device\\physicalmemory"
} UNICODE_STRING, *PUNICODE_STRING;

typedef struct _OBJECT_ATTRIBUTES {
    ULONG Length; // 长度 18h
    HANDLE RootDirectory; // 00000000
    PUNICODE_STRING ObjectName; // 指向对象名的指针
    ULONG Attributes; // 对象属性 00000040h
    PVOID SecurityDescriptor; // Points to type SECURITY_DESCRIPTOR, 0
    PVOID SecurityQualityOfService; // Points to type SECURITY_QUALITY_OF_SERVICE, 0
} OBJECT_ATTRIBUTES;

typedef OBJECT_ATTRIBUTES *POBJECT_ATTRIBUTES;
```

### 函数说明

*   第一个函数 `ZwOpenSection` 用来打开 section，第一个参数是指向 `HANDLE` 变量的指针，第二个是访问参数，第三个是指向 `OBJECT_ATTRIBUTES` 的指针。
*   第二个函数 `ZwMapViewOfSection` 用来建立物理内存和当前进程的一段物理内存的联系，参数很多，一会儿在例程里再详细解释。
*   第三个函数 `ZwUnmapViewOfSection` 用来断开物理内存和当前进程中的映射联系，第一个参数是进程句柄，必须调用第二个函数时一样，第二个是当前进程中映射的基址，由 `ZwMapViewOfSection` 返回。

这三个函数都在 `ntdll.dll` 中，MSDN 里的帮助说这几个函数用在驱动编制上。

### 例程

```c
// 结构定义
typedef struct _UNICODE_STRING {
    USHORT Length; // 长度
    USHORT MaximumLength; // 最大长度
    PWSTR Buffer; // 缓存指针
} UNICODE_STRING, *PUNICODE_STRING;

typedef struct _OBJECT_ATTRIBUTES {
    ULONG Length; // 长度 18h
    HANDLE RootDirectory; // 00000000
    PUNICODE_STRING ObjectName; // 指向对象名的指针
    ULONG Attributes; // 对象属性 00000040h
    PVOID SecurityDescriptor; // Points to type SECURITY_DESCRIPTOR, 0
    PVOID SecurityQualityOfService; // Points to type SECURITY_QUALITY_OF_SERVICE, 0
} OBJECT_ATTRIBUTES;

typedef OBJECT_ATTRIBUTES *POBJECT_ATTRIBUTES;

// 函数指针变量类型声明
typedef DWORD (__stdcall *ZWOS)(PHANDLE, ACCESS_MASK, POBJECT_ATTRIBUTES);
typedef DWORD (__stdcall *ZWMV)(HANDLE, HANDLE, PVOID, ULONG, ULONG, PLARGE_INTEGER, PSIZE_T, DWORD, ULONG, ULONG);
typedef DWORD (__stdcall *ZWUMV)(HANDLE, PVOID);

// 以上在程序开始定义全局变量处定义

// 以下在程序的主函数里

// 变量声明
UNICODE_STRING struniph;
OBJECT_ATTRIBUTES obj_ar;
ZWOS ZWopenS;
ZWMV ZWmapV;
ZWUMV ZWunmapV;
HANDLE hSection;
DWORD ba;
LARGE_INTEGER so;
SIZE_T ssize;

so.LowPart = 0x000f0000; // 物理内存的基址，就是 f000:0000
so.HighPart = 0x00000000;
ssize = 0xffff;
wchar_t strPH[30] = L"\\device\\physicalmemory";

// 变量初始化
ba = 0; // 联系后的基址将在这里返回
struniph.Buffer = strPH;
struniph.Length = 0x2c; // 注意大小是按字节算
struniph.MaximumLength = 0x2e; // 也是字节
obj_ar.Attributes = 64; // 属性
obj_ar.Length = 24; // OBJECT_ATTRIBUTES 类型的长度
obj_ar.ObjectName = &struniph; // 指向对象的指针
obj_ar.RootDirectory = 0;
obj_ar.SecurityDescriptor = 0;
obj_ar.SecurityQualityOfService = 0;

// 读入 ntdll.dll, 得到函数地址
HINSTANCE hinstLib = LoadLibrary("ntdll.dll");
ZWopenS = (ZWOS)GetProcAddress(hinstLib, "ZwOpenSection");
ZWmapV = (ZWMV)GetProcAddress(hinstLib, "ZwMapViewOfSection");
ZWunmapV = (ZWUMV)GetProcAddress(hinstLib, "ZwUnmapViewOfSection");

// 调用函数，对物理内存进行映射
ZWopenS(&hSection, 4, &obj_ar);

ZWmapV(
    (HANDLE)hSection,      // 打开 Section 时得到的句柄
    (HANDLE)0xffffffff,    // 将要映射进程的句柄
    &ba,                   // 映射的基址
    0,                     // 没怎么看明白, 设为 0 就好了
    0xffff,                // 分配的大小
    &so,                   // 物理内存的地址
    &ssize,                // 指向读取内存块大小的指针
    1,                     // 子进程的可继承性设定
    0,                     // 分配类型
    2                     // 保护类型
);

// 执行后会在当前进程的空间开辟一段 64k 的空间，并把 f000:0000 到 f000:ffff 处的内容映射到这里
// 映射的基址由 ba 返回, 如果映射不在有用, 应该用 ZwUnmapViewOfSection 断开映射
```

BTW: 思路主要是来之上次跟踪的联想的安装验证程序，真的要感谢联想的技术人员了：-）。

### 汇编代码示例

随便用汇编写了点代码，懒的解释，功能同 2000 下读取 BIOS：

```asm
void* ptrmem = malloc(0xffff);
_asm
{
    PUSHAD
    PUSH 0
    SIDT [ESP-2]
    POP  EDI           ; EDI -> IDT base
    ADD  EDI, 32       ; EDI -> target Int Entry
    MOV  ECX, [EDI+4]
    MOV  CX, WORD PTR [EDI]    ; ECX -> original Int Handler
    MOV  ESI, offset pNewIntHandler  ; ESI -> new Int Handler
    
    ; make the Int Handler address point to pNewIntHandler
    MOV  [EDI], SI
    SHR  ESI, 16
    MOV  [EDI+6], SI
    
    INT  4             ; "call" hooked interrupt
    
    ; restore the original Int Handler address
    MOV  [EDI], CX
    SHR  ECX, 16
    MOV  [EDI+6], CX
    
    POPAD
    jmp ring0_out

pNewIntHandler:
    PUSHAD
    MOV  ECX, 0FFFFh
    MOV  EAX, 0F0000h
    MOV  EBX, ptrmem

CONTINUE:
    MOV  DL, SS:[EAX]
    MOV  [EBX], DL
    INC  EBX
    INC  EAX
    DEC  ECX
    JNZ  CONTINUE
    POPAD
    iretd

ring0_out:
}
```

### 其他发现

我发现每次 2000 启动后 `1408-140a` 偏移处代码都不一样。

搜寻相异处：

1. `C:\pll20001.tmp`: 65,535 字节
2. `C:\pll2000.tmp`: 65,535 字节

```text
1408:   90   E9
1409:   47   3E
140A:   E8   A8
```

3 处差别发现。

为什么会变化呢？有知道这几个字节的意思的兄弟么？

`pll621 4dd466d0e1e0cc123afbe504da2462ca`
