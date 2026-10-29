
```c
// injector_mips64.c - MIPS64 ptrace 注入工具
// 用法: ./injector_mips64 <pid> <so_absolute_path>
// 需要 root 权限
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/ptrace.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/mman.h>
#include <dlfcn.h>
#include <asm/ptrace.h>
#define mips64_regs pt_regs
// 寄存器别名定义
#define REG_V0 2 // 返回值
#define REG_A0 4 // 参数0
#define REG_A1 5 // 参数1
#define REG_A2 6 // 参数2
#define REG_A3 7 // 参数3
#define REG_A4 8 // 参数4 (MIPS64 N64 ABI)
#define REG_A5 9 // 参数5 (MIPS64 N64 ABI)
#define REG_SP 29 // 栈指针
#define REG_RA 31 // 返回地址
#define REG_T9 25 // 跳转目标 (PIC 代码需要)
#define DUMMY_RETURN_ADDRESS 0x0
static void dump_regs(const char *tag, const struct mips64_regs *regs) {
printf("[*] %s pc=0x%lx sp=0x%lx ra=0x%lx t9=0x%lx v0=0x%lx\n",
tag,
(unsigned long long) regs->cp0_epc,
(unsigned long long) regs->regs[REG_SP],
(unsigned long long) regs->regs[REG_RA],
(unsigned long long) regs->regs[REG_T9],
(unsigned long long) regs->regs[REG_V0]);
printf("[*] %s a0=0x%llx a1=0x%llx a2=0x%llx a3=0x%llx a4=0x%llx a5=0x%llx\n",
tag,
(unsigned long long) regs->regs[REG_A0],
(unsigned long long) regs->regs[REG_A1],
(unsigned long long) regs->regs[REG_A2],
(unsigned long long) regs->regs[REG_A3],
(unsigned long long) regs->regs[REG_A4],
(unsigned long long) regs->regs[REG_A5]);
}
// 获取寄存器
static int get_regs(pid_t pid, struct mips64_regs *regs) {
if (ptrace(PTRACE_GETREGS, pid, NULL, regs) < 0) {
perror("PTRACE_GETREGS");
return -1;
}
return 0;
}
// 设置寄存器
static int set_regs(pid_t pid, struct mips64_regs *regs) {
if (ptrace(PTRACE_SETREGS, pid, NULL, regs) < 0) {
perror("PTRACE_SETREGS");
return -1;
}
return 0;
}
static long get_module_base(pid_t pid, const char *module_name) {
char path[256];
char line[512];
long base = 0;
if (pid == 0) {
snprintf(path, sizeof(path), "/proc/self/maps");
} else {
snprintf(path, sizeof(path), "/proc/%d/maps", pid);
}
FILE *fp = fopen(path, "r");
if (!fp) {
perror("fopen maps");
return 0;
}
while (fgets(line, sizeof(line), fp)) {
// 查找行中的路径部分（空格后的部分）
char *path_start = strchr(line, '/');
if (path_start) {
// 检查是否以 module_name 结尾
size_t path_len = strlen(path_start);
size_t name_len = strlen(module_name);
// 移除换行符
if (path_start[path_len-1] == '\n') {
path_start[path_len-1] = '\0';
path_len--;
}
// 精确匹配：路径以 module_name 结尾
if (path_len >= name_len && strcmp(path_start + path_len - name_len, module_name) == 0) {
base = strtol(line, NULL, 16);
printf("[*] Found module %s at base 0x%lx (pid=%d)\n", module_name, base, pid);
break;
}
}
}
fclose(fp);
return base;
}
// 通过地址查找其所在的模块名和模块基址
static int find_module_by_addr(void *addr, char *module_name, size_t name_len, long *base) {
char line[512];
FILE *fp = fopen("/proc/self/maps", "r");
if (!fp) return -1;
unsigned long target = (unsigned long)addr;
char found_module[256] = {0};
while (fgets(line, sizeof(line), fp)) {
unsigned long start, end;
if (sscanf(line, "%lx-%lx", &start, &end) != 2) continue;
if (target >= start && target < end) {
char *path = strrchr(line, '/');
if (path) {
char *nl = strchr(path, '\n');
if (nl) *nl = '\0';
strncpy(found_module, path, sizeof(found_module) - 1);
}
break;
}
}
fclose(fp);
if (found_module[0] == '\0') return -1;
*base = get_module_base(0, found_module);
strncpy(module_name, found_module, name_len - 1);
module_name[name_len - 1] = '\0';
return 0;
}
// 计算目标进程中某函数的地址（自动检测所在模块）
static long get_remote_func_addr(pid_t pid, void *local_func, const char **out_module) {
char module_name[256];
long local_base;
if (find_module_by_addr(local_func, module_name, sizeof(module_name), &local_base) < 0) {
fprintf(stderr, "Failed to find module for addr %p\n", local_func);
return 0;
}
long remote_base = get_module_base(pid, module_name);
if (!local_base || !remote_base) {
fprintf(stderr, "Failed to get module base for %s (local: 0x%lx, remote: 0x%lx)\n",
module_name, local_base, remote_base);
return 0;
}
if (out_module) *out_module = strdup(module_name);
long offset = (long)local_func - local_base;
long remote_addr = remote_base + offset;
printf("[*] %s: local_func=%p, local_base=0x%lx, remote_base=0x%lx, offset=0x%lx\n",
module_name, local_func, local_base, remote_base, offset);
printf("[*] 计算出的远程地址: 0x%lx\n", remote_addr);
return remote_addr;
}
// 向目标进程写入数据（以 long 为单位，按需补齐）
static int ptrace_write_data(pid_t pid, unsigned long addr, const void *data, size_t len) {
const unsigned char *src = (const unsigned char *)data;
size_t i = 0;
for (i = 0; i + sizeof(long long) <= len; i += sizeof(long long)) {
long long val;
memcpy(&val, src + i, sizeof(long long));
if (ptrace(PTRACE_POKEDATA, pid, (void *)(addr + i), (void *)val) < 0) {
perror("PTRACE_POKEDATA");
return -1;
}
}
if (i < len) {
long long val = ptrace(PTRACE_PEEKDATA, pid, (void *)(addr + i), NULL);
memcpy(&val, src + i, len - i);
if (ptrace(PTRACE_POKEDATA, pid, (void *)(addr + i), (void *)val) < 0) {
perror("PTRACE_POKEDATA tail");
return -1;
}
}
return 0;
}
// 在远程进程中调用一个函数（支持6个参数，用于mmap）
// MIPS64 N64 ABI: 前8个整数参数通过 $a0-$a7 (gpr[4]-gpr[11]) 传递
// 关键: MIPS PIC 代码需要设置 $t9 寄存器为目标地址 (gpr[25])
// 返回值在 $v0 (gpr[2])
static long call_remote_func(pid_t pid, struct mips64_regs *orig_regs,
long func_addr,
long arg0, long arg1, long arg2, long arg3,
long arg4, long arg5) {
struct mips64_regs regs;
memcpy(&regs, orig_regs, sizeof(regs));
// MIPS64 N64 ABI: 前8个参数都通过寄存器传递
regs.regs[REG_A0] = arg0; // $a0
regs.regs[REG_A1] = arg1; // $a1
regs.regs[REG_A2] = arg2; // $a2
regs.regs[REG_A3] = arg3; // $a3
regs.regs[REG_A4] = arg4; // $a4 (第5个参数)
regs.regs[REG_A5] = arg5; // $a5 (第6个参数)
// ★ 关键: 设置 $t9 为目标地址 (MIPS PIC 代码需要)
// MIPS 的位置无关代码(PIC)使用 $t9 来访问全局偏移表(GOT)
regs.regs[REG_T9] = func_addr;
// 设置 pc 为目标函数地址
regs.cp0_epc = func_addr;
regs.regs[REG_RA] = DUMMY_RETURN_ADDRESS;
// 关键：保存原始 ra 和 pc，用于后续判断
unsigned long orig_ra = orig_regs->regs[31];
unsigned long orig_pc = orig_regs->cp0_epc;
printf("[*] call6: pc=0x%lx, t9=0x%lx, sp=0x%lx, ra=0x%lx\n",
regs.cp0_epc, regs.regs[REG_T9], regs.regs[REG_SP], regs.regs[REG_RA]);
printf("[*] args: a0=0x%lx, a1=0x%lx, a2=0x%lx, a3=0x%lx, a4=0x%lx, a5=0x%lx\n",
arg0, arg1, arg2, arg3, arg4, arg5);
printf("[*] orig: pc=0x%lx, ra=0x%lx\n", orig_pc, orig_ra);
dump_regs("before-set", &regs);
if (set_regs(pid, &regs) < 0) {
perror("set_regs");
return -1;
}
struct mips64_regs verify_regs;
if (get_regs(pid, &verify_regs) < 0) {
perror("get_regs after set");
return -1;
}
dump_regs("after-set", &verify_regs);
printf("[*] 调用 PTRACE_CONT...\n");
if (ptrace(PTRACE_CONT, pid, NULL, NULL) < 0) {
perror("PTRACE_CONT");
return -1;
}
// 读取当前寄存器
if (get_regs(pid, &regs) < 0) return -1;
printf("[*] current: pc=0x%lx, ra=0x%lx, v0=0x%lx\n",
regs.cp0_epc, regs.regs[REG_RA], regs.regs[REG_V0]);
// 正常返回时 pc == DUMMY_RETURN_ADDRESS
if (regs.cp0_epc == DUMMY_RETURN_ADDRESS) {
printf("[+] 函数正常返回（pc=0x%lx）\n", regs.cp0_epc);
printf("[*] return value v0=0x%lx\n", regs.regs[REG_V0]);
return (long) regs.regs[REG_V0];
}
printf("[!] 远程调用未正常返回，按失败处理\n");
return -1;
}
// 在远程进程中直接执行 MIPS64 syscall (最多6个参数)
static long call_remote_syscall(pid_t pid, struct mips64_regs *orig_regs,
long syscall_no,
long arg0, long arg1, long arg2,
long arg3, long arg4, long arg5) {
struct mips64_regs regs;
memcpy(&regs, orig_regs, sizeof(regs));
regs.regs[REG_V0] = syscall_no;
regs.regs[REG_A0] = arg0;
regs.regs[REG_A1] = arg1;
regs.regs[REG_A2] = arg2;
regs.regs[REG_A3] = arg3;
regs.regs[REG_A4] = arg4;
regs.regs[REG_A5] = arg5;
unsigned long long saved_word = ptrace(PTRACE_PEEKDATA, pid, (void *)orig_regs->cp0_epc, NULL);
unsigned long long patched_word = (saved_word & 0xffffffff00000000ULL) | 0x0000000cULL;
if (ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)patched_word) < 0) {
perror("PTRACE_POKEDATA syscall");
return -1;
}
regs.cp0_epc = orig_regs->cp0_epc;
regs.regs[REG_RA] = DUMMY_RETURN_ADDRESS;
dump_regs("before-syscall", &regs);
if (set_regs(pid, &regs) < 0) {
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
if (ptrace(PTRACE_SYSCALL, pid, NULL, NULL) < 0) {
perror("PTRACE_SYSCALL enter");
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
int status;
if (waitpid(pid, &status, WUNTRACED) < 0) {
perror("waitpid syscall enter");
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
if (ptrace(PTRACE_SYSCALL, pid, NULL, NULL) < 0) {
perror("PTRACE_SYSCALL exit");
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
if (waitpid(pid, &status, WUNTRACED) < 0) {
perror("waitpid syscall exit");
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
struct mips64_regs result_regs;
if (get_regs(pid, &result_regs) < 0) {
ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word);
return -1;
}
if (ptrace(PTRACE_POKEDATA, pid, (void *)orig_regs->cp0_epc, (void *)saved_word) < 0) {
perror("PTRACE_POKEDATA restore");
}
dump_regs("after-syscall", &result_regs);
if (result_regs.regs[REG_A3] != 0) {
printf("[!] 远程 syscall 失败: errno=%ld (%s)\n",
(long) result_regs.regs[REG_V0],
strerror((int) result_regs.regs[REG_V0]));
return -1;
}
return (long) result_regs.regs[REG_V0];
}
// MIPS64 Linux syscall number for mmap
#define __NR_MMAP_MIPS64 5009
int main(int argc, char *argv[]) {
if (argc != 3) {
fprintf(stderr, "用法: %s <pid> <so_absolute_path>\n", argv[0]);
return 1;
}
pid_t pid = atoi(argv[1]);
const char *so_path = argv[2];
int status;
printf("[*] 目标进程: %d\n", pid);
printf("[*] 注入 so: %s\n", so_path);
// 1. 附加目标进程
if (ptrace(PTRACE_ATTACH, pid, NULL, NULL) < 0) {
perror("PTRACE_ATTACH");
return 1;
}
waitpid(pid, &status, WUNTRACED);
printf("[+] 已附加到目标进程\n");
// 2. 保存原始寄存器
struct mips64_regs orig_regs;
if (get_regs(pid, &orig_regs) < 0) {
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[+] 已保存寄存器现场\n");
printf("[*] 原始寄存器: pc=0x%lx, ra=0x%lx, sp=0x%lx\n",
orig_regs.cp0_epc, orig_regs.regs[REG_RA], orig_regs.regs[REG_SP]);
// 检查 PC 是否有效
if (orig_regs.cp0_epc == 0 || orig_regs.cp0_epc > 0xfffffffffffff000) {
fprintf(stderr, "[-] 警告: 目标进程 PC 无效 (0x%lx)，进程可能已崩溃\n", orig_regs.cp0_epc);
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
// 3. 获取远程 mmap 地址 (自动检测所在模块)
// 尝试使用 __mmap（内部实现）而不是 mmap（可能是包装函数）
long remote_mmap = get_remote_func_addr(pid, (void *)mmap, NULL);
if (!remote_mmap) {
fprintf(stderr, "[-] 获取远程 mmap 地址失败\n");
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[*] 远程 mmap 地址: 0x%lx\n", remote_mmap);
// 4. 优先直接使用 MIPS64 mmap syscall，绕过 libc wrapper
long mmap_result = call_remote_syscall(pid, &orig_regs,
__NR_MMAP_MIPS64,
0, // addr = NULL
0x1000, // length = 4096
PROT_READ | PROT_WRITE, // prot
MAP_PRIVATE | MAP_ANONYMOUS, // flags
(unsigned long)-1, // fd = -1
0); // offset
// mmap 失败时返回 0 / MAP_FAILED / 异常高地址
if (mmap_result == 0 || mmap_result == -1 || (unsigned long)mmap_result > 0xfffffffffffff000) {
fprintf(stderr, "[-] 远程 mmap 失败, 返回: 0x%lx\n", mmap_result);
set_regs(pid, &orig_regs);
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[+] 远程 mmap 成功, 地址: 0x%lx\n", mmap_result);
// 5. 将 so 路径写入 mmap 分配的内存
size_t path_len = strlen(so_path) + 1;
if (ptrace_write_data(pid, (unsigned long)mmap_result, so_path, path_len) < 0) {
fprintf(stderr, "[-] 写入 so 路径失败\n");
set_regs(pid, &orig_regs);
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[+] so 路径已写入远程内存 @ 0x%lx\n", mmap_result);
// 6. 获取远程 dlopen 地址 (自动检测所在模块)
void *local_dlopen = dlsym(RTLD_DEFAULT, "dlopen");
if (!local_dlopen) {
fprintf(stderr, "[-] 无法获取 dlopen 地址\n");
set_regs(pid, &orig_regs);
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[*] 本地 dlopen 地址: %p\n", local_dlopen);
const char *dlopen_module = NULL;
long remote_dlopen = get_remote_func_addr(pid, local_dlopen, &dlopen_module);
if (!remote_dlopen) {
fprintf(stderr, "[-] 获取远程 dlopen 地址失败\n");
set_regs(pid, &orig_regs);
ptrace(PTRACE_DETACH, pid, NULL, NULL);
return 1;
}
printf("[+] 远程 dlopen 地址: 0x%lx (模块: %s)\n", remote_dlopen,
dlopen_module ? dlopen_module : "unknown");
// 7. 在远程进程中调用 dlopen(so_path, RTLD_NOW)
long dlopen_ret = call_remote_func(pid, &orig_regs, remote_dlopen,
mmap_result, // $a0 = so 路径地址 (mmap 分配的)
RTLD_NOW, // $a1 = flags
0, 0, // $a2, $a3
0, 0); // padding
printf("[+] dlopen 返回值: 0x%lx\n", dlopen_ret);
if (dlopen_ret == 0) {
printf("[-] dlopen 失败！\n");
} else {
printf("[+] 注入成功！handle = 0x%lx\n", dlopen_ret);
}
// 8. 恢复寄存器现场并分离
if (set_regs(pid, &orig_regs) < 0) {
perror("set_regs restore");
}
ptrace(PTRACE_DETACH, pid, NULL, NULL);
printf("[+] 已恢复现场并分离目标进程\n");
return 0;
}
```
