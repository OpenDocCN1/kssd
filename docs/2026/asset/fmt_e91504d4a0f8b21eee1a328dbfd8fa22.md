
```c
#include <windows.h>
#include <stdint.h>
// x64 指令长度解析器（简化版，覆盖常见指令）
// 完整版应使用 Zydis/distorm 等反汇编库
typedef struct \_INSTRUCTION {
uint8\_t length;
BOOL isRipRelative; // 是否包含 RIP 相对寻址
int32\_t ripOffset; // RIP 偏移在指令中的位置
int32\_t ripDisplacement; // 原始 displacement 值
} INSTRUCTION;
// 解析单条指令长度（简化版核心逻辑）
INSTRUCTION ParseInstruction(const uint8\_t\* code) {
INSTRUCTION inst = {0};
const uint8\_t\* p = code;
// 跳过前缀 (REX, LOCK, REP, segment override 等)
while (\*p == 0xF0 || \*p == 0xF2 || \*p == 0xF3 ||
\*p == 0x26 || \*p == 0x2E || \*p == 0x36 || \*p == 0x3E ||
\*p == 0x64 || \*p == 0x65 || \*p == 0x66 || \*p == 0x67 ||
(\*p >= 0x40 && \*p <= 0x4F)) { // REX prefix
p++;
}
uint8\_t opcode = \*p++;
// 处理双字节操作码 (0F xx)
if (opcode == 0x0F) {
uint8\_t op2 = \*p++;
// ModRM
if (op2 >= 0x80 && op2 <= 0x8F) {
// Jcc rel32 (条件跳转)
inst.length = (int)(p - code) + 4;
inst.isRipRelative = TRUE;
inst.ripOffset = (int)(p - code);
inst.ripDisplacement = \*(int32\_t\*)p;
return inst;
}
// 其他 0F xx 指令处理...
if ((op2 & 0xC0) != 0xC0) { // 有 ModRM
uint8\_t modrm = \*p++;
uint8\_t mod = (modrm >> 6) & 3;
uint8\_t rm = modrm & 7;
if (mod == 0 && rm == 5) { // RIP-relative
inst.isRipRelative = TRUE;
inst.ripOffset = (int)(p - code);
inst.ripDisplacement = \*(int32\_t\*)p;
p += 4;
} else if (mod == 0 && rm == 4) { p++; } // SIB
else if (mod == 1) { if (rm == 4) p++; p++; }
else if (mod == 2) { if (rm == 4) p++; p += 4; }
}
inst.length = (int)(p - code);
return inst;
}
// 单字节操作码处理
switch (opcode) {
case 0xE8: // CALL rel32
case 0xE9: // JMP rel32
inst.length = (int)(p - code) + 4;
inst.isRipRelative = TRUE;
inst.ripOffset = (int)(p - code);
inst.ripDisplacement = \*(int32\_t\*)p;
return inst;
case 0xEB: // JMP rel8
inst.length = (int)(p - code) + 1;
inst.isRipRelative = TRUE;
inst.ripOffset = (int)(p - code);
inst.ripDisplacement = (int8\_t)\*p;
return inst;
// ... 其他操作码
}
// 通用 ModRM 解析
// (这里省略完整的操作码表映射，实际项目应使用 Zydis)
inst.length = (int)(p - code);
if (inst.length == 0) inst.length = 1; // 兜底
return inst;
}
// Trampoline 构建器：将被覆盖的原始指令复制到 trampoline，并修正 RIP 相对引用
#define HOOK\_STUB\_SIZE 14 // x64 绝对跳转: FF 25 00 00 00 00 [8字节地址]
#define TRAMPOLINE\_MAX 64
typedef struct \_HOOK\_CONTEXT {
void\* pTarget; // 原始函数地址
void\* pDetour; // Hook 函数地址
uint8\_t trampoline[TRAMPOLINE\_MAX]; // Trampoline 缓冲区
uint8\_t originalBytes[HOOK\_STUB\_SIZE]; // 备份的原始字节
uint32\_t stolenLength; // 实际偷取的字节数
void\* pTrampoline; // Trampoline 可执行内存
} HOOK\_CONTEXT;
BOOL BuildTrampoline(HOOK\_CONTEXT\* ctx) {
uint8\_t\* src = (uint8\_t\*)ctx->pTarget;
uint8\_t\* dst = ctx->trampoline;
uint32\_t totalCopied = 0;
// 需要至少偷取 HOOK\_STUB\_SIZE 字节的完整指令
while (totalCopied < HOOK\_STUB\_SIZE) {
INSTRUCTION inst = ParseInstruction(src + totalCopied);
if (inst.isRipRelative) {
// RIP 相对指令需要重定位
// 计算原始目标地址
uint8\_t\* originalRip = src + totalCopied + inst.length; // 执行完该指令后的 RIP
void\* absoluteTarget = originalRip + inst.ripDisplacement;
// 计算新的 displacement（从 trampoline 中的新位置到同一个绝对目标）
uint8\_t\* newRip = dst + inst.length;
int64\_t newDisp = (int64\_t)((uint8\_t\*)absoluteTarget - newRip);
if (newDisp > INT32\_MAX || newDisp < INT32\_MIN) {
// 距离超过 ±2GB，需要用绝对跳转间接寻址
// 这种情况在 trampoline 分配在远处时会发生
// 解决：在 trampoline 末尾放跳转表
memcpy(dst, src + totalCopied, inst.length);
// 将 displacement 改为指向 trampoline 内的跳转表
// （此处简化处理，实际需要跳转表机制）
return FALSE; // 需要更复杂的处理
}
// 复制指令，修改 displacement
memcpy(dst, src + totalCopied, inst.length);
\*(int32\_t\*)(dst + inst.ripOffset) = (int32\_t)newDisp;
} else {
// 非 RIP 相对指令，直接复制
memcpy(dst, src + totalCopied, inst.length);
}
dst += inst.length;
totalCopied += inst.length;
}
ctx->stolenLength = totalCopied;
// 在 trampoline 末尾追加绝对跳转回原始函数（跳过被偷取的部分）
// FF 25 00 00 00 00 [目标地址 8字节]
\*dst++ = 0xFF;
\*dst++ = 0x25;
\*(uint32\_t\*)dst = 0; dst += 4;
\*(uint64\_t\*)dst = (uint64\_t)(src + totalCopied); dst += 8;
// 分配可执行内存给 trampoline
size\_t trampolineSize = (size\_t)(dst - ctx->trampoline);
ctx->pTrampoline = VirtualAlloc(NULL, trampolineSize,
MEM\_COMMIT | MEM\_RESERVE, PAGE\_EXECUTE\_READWRITE);
if (!ctx->pTrampoline) return FALSE;
memcpy(ctx->pTrampoline, ctx->trampoline, trampolineSize);
return TRUE;
}
// 安装 Hook
BOOL InstallInlineHook(HOOK\_CONTEXT\* ctx) {
// 备份原始字节
memcpy(ctx->originalBytes, ctx->pTarget, HOOK\_STUB\_SIZE);
// 构建 Trampoline
if (!BuildTrampoline(ctx)) return FALSE;
// 写入跳转到 Detour
DWORD oldProtect;
VirtualProtect(ctx->pTarget, HOOK\_STUB\_SIZE, PAGE\_EXECUTE\_READWRITE, &oldProtect);
uint8\_t\* p = (uint8\_t\*)ctx->pTarget;
p[0] = 0xFF;
p[1] = 0x25;
\*(uint32\_t\*)(p + 2) = 0;
\*(uint64\_t\*)(p + 6) = (uint64\_t)ctx->pDetour;
VirtualProtect(ctx->pTarget, HOOK\_STUB\_SIZE, oldProtect, &oldProtect);
FlushInstructionCache(GetCurrentProcess(), ctx->pTarget, HOOK\_STUB\_SIZE);
return TRUE;
}
// 卸载 Hook
BOOL RemoveInlineHook(HOOK\_CONTEXT\* ctx) {
DWORD oldProtect;
VirtualProtect(ctx->pTarget, HOOK\_STUB\_SIZE, PAGE\_EXECUTE\_READWRITE, &oldProtect);
memcpy(ctx->pTarget, ctx->originalBytes, HOOK\_STUB\_SIZE);
VirtualProtect(ctx->pTarget, HOOK\_STUB\_SIZE, oldProtect, &oldProtect);
FlushInstructionCache(GetCurrentProcess(), ctx->pTarget, HOOK\_STUB\_SIZE);
if (ctx->pTrampoline) {
VirtualFree(ctx->pTrampoline, 0, MEM\_RELEASE);
ctx->pTrampoline = NULL;
}
return TRUE;
}
// 线程安全 Hook 安装（暂停其他线程避免竞态）
BOOL SafeInstallHook(HOOK\_CONTEXT\* ctx) {
// 挂起所有其他线程
HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS\_SNAPTHREAD, 0);
THREADENTRY32 te = { .dwSize = sizeof(te) };
DWORD currentThread = GetCurrentThreadId();
DWORD currentProcess = GetCurrentProcessId();
HANDLE suspendedThreads[256];
int suspendCount = 0;
if (Thread32First(hSnap, &te)) {
do {
if (te.th32OwnerProcessID == currentProcess && te.th32ThreadID != currentThread) {
HANDLE hThread = OpenThread(THREAD\_SUSPEND\_RESUME, FALSE, te.th32ThreadID);
if (hThread) {
SuspendThread(hThread);
suspendedThreads[suspendCount++] = hThread;
}
}
} while (Thread32Next(hSnap, &te) && suspendCount < 256);
}
CloseHandle(hSnap);
// 安装 Hook
BOOL result = InstallInlineHook(ctx);
// 恢复所有线程
for (int i = 0; i < suspendCount; i++) {
ResumeThread(suspendedThreads[i]);
CloseHandle(suspendedThreads[i]);
}
return result;
}
```

### 检测难度：★★☆☆☆

读取函数头几字节，对比磁盘上的原始 DLL 即可发现。大部分反作弊都会做完整性校验。

### 优点

- 通用性最强，可以 Hook 任何函数
- 不管调用方式（IAT/EAT/动态获取）都能拦截
- Microsoft Detours 库提供了工业级实现

### 局限

- 函数头部被直接修改，任何完整性校验都能发现
- 需要处理多线程竞态（Hook 安装瞬间其他线程正在执行目标函数）
- x64 绝对跳转指令长达 14 字节，可能覆盖多条原始指令
- 指令重定位是最复杂的部分，生产环境建议使用 Zydis/distorm

---

## 1.4 VEH Hook（向量化异常处理 Hook）

### 原理

利用 Windows 向量化异常处理机制（VEH），通过设置硬件断点或内存保护异常，在目标函数执行时触发异常，在异常处理器中劫持执行流。

### 完整实现（含多线程 DR 设置）
