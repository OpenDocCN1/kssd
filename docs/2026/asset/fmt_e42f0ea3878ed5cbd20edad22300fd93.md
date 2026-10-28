
# [原创] Windows内核不触发PG Hook `NtDeviceIoControlFile`

## 1. 前言

**注意: 请勿将本贴中所涉及的技术以及源码用于非法用途!**

兼容性: 目前经过不同系统环境下的测试,能稳定兼容win7以及win10全版本

项目地址: [CS1ime/DICHook: Hook `NtDeviceIoControlFile` with PatchGuard (github.com)](elink@aceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6f1K6q4A6L8h3g2Q4x3V1k6p5d9f1y4t1L8$3!0C8)

## 2. 思路

通过对`NtDeviceIoControlFile`函数的逆向分析,发现该函数存在这样的一条调用链:

`NtDeviceIoControlFile`会调用`IopAllocateIrpExReturn`申请irp内存

当`IopDispatchAllocateIrp = 1`时, `IopAllocateIrpExReturn`调用`IovAllocateIrp`

![](upload/attach/202110/844784_SNY4U7DSCHXQHA3.png)

当`VfIoDisabled = 0` 时, `IovAllocateIrp`调用`ViIrpAllocateLockedPacket`

![](upload/attach/202110/844784_WB582F87NTZYHVN.png)

`ViIrpAllocateLockedPacket`调用`VfPacketCreateAndLock`

![](upload/attach/202110/844784_P7NC73YKPKUPEB3.png)

`VfPacketCreateAndLock`会调用`ExAllocateFromNPagedLookasideList`,从而调用`ViPacketLookaside.L.AllocateEx`

而`ViPacketLookaside.L.AllocateEx`这个函数地址是可以被随便更改的,我们可以在这个时机拿到堆栈中调用者原先传入的参数,对参数进行任意修改

## 3. 实现过程

安装hook的过程是`DICHook.cpp`文件里的`InstallHook()`函数实现的

前面是定位各种函数和数据地址(搜的好累...)

### Hook ViPacketLookaside.L.AllocateEx

将`ViPacketLookaside.L.AllocateEx`替换成如下shellcode模板

```asm
push rax
push rbx
push rcx
push rdx
push rsi
push rdi
push rbp
push r8
push r9
push r10
push r11
push r12
push r13
push r14
push r15
pushf
lea rcx,[rsp+0x80] ;参数1是堆栈指针
movabs rax,0x10000000000 ;回调函数地址
xor rax,0x7fffffff
xchg rbx,rax
call WinAPI_EnterStack ;强制对齐堆栈
call rbx
call WinAPI_ExitStack
popf
pop r15
pop r14
pop r13
pop r12
pop r11
pop r10
pop r9
pop r8
pop rbp
pop rdi
pop rsi
pop rdx
pop rcx
pop rbx
pop rax
push rax ;这是一个不修改寄存器的jump
mov DWORD PTR [rsp],0x10000000 ;跳转地址高4字节
mov DWORD PTR [rsp+0x4],0x10000000 ;跳转地址低4字节
ret

WinAPI_EnterStack:
lea r11,[rsp+0x8]
and rsp,0xfffffffffffffff0
push r11
push r11
sub rsp,0x30
jmp QWORD PTR [r11-0x8]

WinAPI_ExitStack:
pop r11
add rsp,0x38
pop rsp
jmp r11
```

为了防止`RtlpInterlockedPopEntrySList`返回非0值,将`ViPacketLookaside.L.ListHead.Region`值设为0

![](upload/attach/202110/844784_PTXFZ8EPVHAURJC.png)

填充shellcode模板,使其触发回调函数`DICHook.cpp!DispatchCallback`

```cpp
PUCHAR pcode = (PUCHAR)ExAllocatePool(NonPagedPool, 0x500);

memcpy(pcode, shellcode, sizeof(shellcode));

*(ULONG64 *)(pcode + 0x22) = ((ULONG64)DispatchCallback) ^ 0x7fffffff; //回调函数

//ViPacketLookaside.Region=0
//防止RtlpInterlockedPopEntrySList返回非0值
*(ULONG64*)(ViPacketLookaside + 0x8) = 0;

//修改ViPacketLookaside.AllocateEx
ULONG64 pfn = *(ULONG64*)(ViPacketLookaside + 0x30);

LARGE_INTEGER Addr;

Addr.QuadPart = (ULONG64)MyAllocEx; //使ViPacketLookaside.L.AllocateEx最终返回0

*(ULONG *)(pcode + 0x5A) = Addr.LowPart;

*(ULONG *)(pcode + 0x62) = Addr.HighPart;

InterlockedExchange64((volatile LONG64*)(ViPacketLookaside + 0x30), (LONG64)pcode);
```

然后将`IopDispatchAllocateIrp`赋值为1,`VfIoDisabled`赋值为0.

此时已经实现了系统中每次调用`NtDeviceIoControlFile`函数会经过如下路径来触发`DICHook.cpp!DispatchCallback`

`NtDeviceIoControlFile->IopXxxControlFile->IopAllocateIrpExReturn->IovAllocateIrp->ViIrpAllocateLockedPacket->VfPacketCreateAndLock->ExAllocateFromNPagedLookasideList->ViPacketLookaside.L.AllocateEx->DICHook.cpp!DispatchCallback`

### 寻找参数

`NtDeviceIoControlFile`函数原型如下

```c
__kernel_entry NTSTATUS NtDeviceIoControlFile(
    [in] HANDLE FileHandle,
    [in] HANDLE Event,
    [in] PIO_APC_ROUTINE ApcRoutine,
    [in] PVOID ApcContext,
    [out] PIO_STATUS_BLOCK IoStatusBlock,
    [in] ULONG IoControlCode,
    [in] PVOID InputBuffer,
    [in] ULONG InputBufferLength,
    [out] PVOID OutputBuffer,
    [in] ULONG OutputBufferLength
);
```

Windows x64调用约定是前4个参数分别用寄存器`rcx`,`rdx`,`r8`,`r9`存放,第5个开始参数存放在堆栈里, 也就是前4个参数在函数被调用时不会直接出现在参数堆栈里,而`NtDeviceIoControlFile`前4个参数里最重要的是`FileHandle`参数

`IopXxxControlFile`有一处是调用`ObReferenceObjectByHandle`将输入的`FileHandle`转换为对应的`Object`并保存在堆栈里,所以拿到堆栈里的`Object`就行啦

![](upload/attach/202110/844784_46SCKFXKHTXMSDF.png)

那么问题来了..如何获取每个系统的堆栈中参数偏移?

经过几天的思考后,我想到了一个可行的方案:

**模拟调用一次`NtDeviceIoControlFile`,触发`DICHook.cpp!DispatchCallback`时通过搜索传入的参数来得到每个系统的堆栈中参数偏移**

编写以下代码来测试参数偏移:

```cpp
VOID TestDeviceIoControl() {

    HANDLE FileHandle = 0;

    UNICODE_STRING name;

    RtlInitUnicodeString(&name, L"\\??\\C:");

    OBJECT_ATTRIBUTES oa;

    InitializeObjectAttributes(&oa, &name, OBJ_CASE_INSENSITIVE, 0, 0);

    IO_STATUS_BLOCK iosb;

    RtlZeroMemory(&iosb, sizeof(IO_STATUS_BLOCK));

    NTSTATUS stats = ZwCreateFile(&FileHandle, FILE_GENERIC_READ, &oa, &iosb, 0, FILE_ATTRIBUTE_NORMAL, FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE, FILE_OPEN_IF, FILE_SYNCHRONOUS_IO_NONALERT | FILE_NON_DIRECTORY_FILE, 0, 0);

    RtlZeroMemory(&iosb, sizeof(IO_STATUS_BLOCK));

    PFILE_OBJECT obj = 0;

    OBJECT_HANDLE_INFORMATION objhandle = { 0 };

    RtlZeroMemory(&objhandle, sizeof(objhandle));

    stats = ObReferenceObjectByHandle(FileHandle, 0, *IoFileObjectType, KernelMode, (PVOID *)&obj, &objhandle);

    if (!NT_SUCCESS(stats)) {

        ZwClose(FileHandle);

        KeBugCheck(0x56009);

    }

    ObDereferenceObject(obj);

    Search_Object = (ULONG64)obj;

    typedef NTSTATUS

    (*NTAPI fnNtDeviceIoControlFile)(

        _In_ HANDLE FileHandle,

        _In_opt_ HANDLE Event,

        _In_opt_ PIO_APC_ROUTINE ApcRoutine,

        _In_opt_ PVOID ApcContext,

        _Out_ PIO_STATUS_BLOCK IoStatusBlock,

        _In_ ULONG IoControlCode,

        _In_reads_bytes_opt_(InputBufferLength) PVOID InputBuffer,

        _In_ ULONG InputBufferLength,

        _Out_writes_bytes_opt_(OutputBufferLength) PVOID OutputBuffer,

        _In_ ULONG OutputBufferLength

    );

    fnNtDeviceIoControlFile pNtDeviceIoControlFile = (fnNtDeviceIoControlFile)KGetProcAddress(KGetNtoskrnl(), "NtDeviceIoControlFile");

    UCHAR Input[4] = { 0 };

    UCHAR Output[4] = { 0 };

    ULONG64 Magic[2];

    Magic[0] = 0x1122334455667788; //Magic1

    Magic[1] = 0x8877665544772299; //Magic2

    pNtDeviceIoControlFile(FileHandle, 0, 0, 0, &iosb, IOCTL_NDIS_QUERY_GLOBAL_STATS, Input, 4, Output, 4);

    ZwClose(FileHandle);

}
```

当运行以上代码时,触发`DICHook.cpp!DispatchCallback`时的堆栈的数据情况是这样子的

![](upload/attach/202110/844784_DMQKZ7EF5ZJCTEN.png)

回调中先获取`Magic1`和`Magic2`的位置,从该位置向下遍历找到`NtDeviceIoControlFile`调用`IopXxxControlFile`时的返回地址,从该返回地址再向下遍历找到`Object`的偏移

实现代码:

```cpp
for (int i = 0; (ULONG64)Rsp < StackHighLimit - 8; Rsp++, i++) {

    if (RspOffset == 0) {

        if (Rsp[0] == 0x1122334455667788) {

            if (Rsp[1] == 0x8877665544772299) {

                //搜索栈上Object偏移
                ULONG64 OLRSP = (ULONG64)Rsp;

                for (int j = 0; OLRSP > pRsp && j < 0x1000; OLRSP -= 8, j += 8) {

                    if (*(ULONG64*)OLRSP == NtDeviceIoControlFileRet) {

                        RspOffset = OLRSP - pRsp;

                        ULONG64 OOLRSP = OLRSP;

                        for (int p = 0; OOLRSP > pRsp && p < 0x1000; OOLRSP -= 8, p += 8) {

                            if (NtDevice_Offset_Object)

                                break;

                            if (NtDevice_Offset_Object == 0) {

                                if (*(ULONG64*)OOLRSP == Search_Object) {

                                    NtDevice_Offset_Object = OOLRSP - pRsp;

                                    continue;

                                }

                            }

                        }

                        break;

                    }

                }

            }

        }

    }

    if (RspOffset)break;

}
```

由于每个系统中`NtDeviceIoControlFile`函数的申请的堆栈大小都是`0x68`,所以可以用`IopXxxControlFile`返回地址的偏移加上固定偏移找到后7个参数

但是此时的`OutputBuffer`是没有数据的,要到`IopXxxControlFile`返回的时候才能修改`OutputBuffer`里的数据 , 修改堆栈里的`IopXxxControlFile`返回地址即可

### 调用回调

申请一个pool来存放获取到的参数,用以下两个shellcode模板来传递pool指针并调用`DICHook.cpp!DICPreCallback`来传递参数给回调

shellcode模板1,用来传递参数

```asm
movabs rcx,0x100000000000 ;原先返回地址
push rcx
movabs rcx,0x100000000000 ;参数
push rax ;jump到shellcode模板2
mov DWORD PTR [rsp],0x10000000
mov DWORD PTR [rsp+0x4],0x10000000
ret
```

shellcode模板2,用来调用`DICHook.cpp!DICPreCallback`

```asm
push rax
push rbx
push rcx
push rdx
push rsi
push rdi
push rbp
push r8
push r9
push r10
push r11
push r12
push r13
push r14
push r15
pushf
movabs rax,0x10000000000 ;填入DICHook.cpp!DICPreCallback函数地址
xor rax,0x7fffffff
xchg rbx,rax
call WinAPI_EnterStack ;强制对齐堆栈
call rbx ;调用函数
call WinAPI_ExitStack
popf
pop r15
pop r14
pop r13
pop r12
pop r11
pop r10
pop r9
pop r8
pop rbp
pop rdi
pop rsi
pop rdx
pop rcx
pop rbx
pop rax
ret

WinAPI_EnterStack:
lea r11,[rsp+0x8]
and rsp,0xfffffffffffffff0
push r11
push r11
sub rsp,0x30
jmp QWORD PTR [r11-0x8]

WinAPI_ExitStack:
pop r11
add rsp,0x38
pop rsp
jmp r11
```
