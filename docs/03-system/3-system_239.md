
# SSDT Hook 的妙用－对抗 ring0 inline hook

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-84.htm`](https://www.pediy.com/kssd/pediy09/pediy09-84.htm)

> `****`
>
> `*标题:【原创】SSDT Hook 的妙用－对抗 ring0 inline hook *`
>
> `*作者:堕落天才*`
>
> `*日期:2007 年 3 月 10 号*`
>
> `*声明:本文章的目的仅为技术交流讨论*`
>
> `****`

## 1, SSDT

SSDT 即系统服务描述符表，它的结构如下(参考《Undocument Windows 2000 Secretes》第二章):

```c
typedef struct _SYSTEM_SERVICE_TABLE
{
  PVOID   ServiceTableBase;        //这个指向系统服务函数地址表
  PULONG  ServiceCounterTableBase;
  ULONG   NumberOfService;         //服务函数的个数,NumberOfService*4 就是整个地址表的大小
  ULONG   ParamTableBase;
}SYSTEM_SERVICE_TABLE,*PSYSTEM_SERVICE_TABLE;

typedef struct _SERVICE_DESCRIPTOR_TABLE
{
  SYSTEM_SERVICE_TABLE   ntoskrnel;  //ntoskrnl.exe 的服务函数
  SYSTEM_SERVICE_TABLE   win32k;     //win32k.sys 的服务函数,(gdi.dll/user.dll 的内核支持)
  SYSTEM_SERVICE_TABLE   NotUsed1;
  SYSTEM_SERVICE_TABLE   NotUsed2;
}SYSTEM_DESCRIPTOR_TABLE,*PSYSTEM_DESCRIPTOR_TABLE;
```

内核中有两个系统服务描述符表,一个是 `KeServiceDescriptorTable`(由 `ntoskrnl.exe` 导出),一个是 `KeServieDescriptorTableShadow`(没有导出)。两者的区别是，`KeServiceDescriptorTable` 仅有 `ntoskrnel` 一项，`KeServieDescriptorTableShadow` 包含了 `ntoskrnel` 以及 `win32k`。一般的 Native API 的服务地址由 `KeServiceDescriptorTable` 分派，`gdi.dll`/`user.dll` 的内核 API 调用服务地址由 `KeServieDescriptorTableShadow` 分派。还有要清楚一点的是 `win32k.sys` 只有在 GUI 线程中才加载，一般情况下是不加载的，所以要 Hook `KeServieDescriptorTableShadow` 的话，一般是用一个 GUI 程序通过 `IoControlCode` 来触发(想当初不明白这点，蓝屏死机了 N 次都想不明白是怎么回事)。

## 2，SSDT HOOK

SSDT HOOK 的原理其实非常简单，我们先实际看看 `KeServiceDescriptorTable` 是什么样的。

```text
lkd> dd KeServiceDescriptorTable
8055ab80  804e3d20 00000000 0000011c 804d9f48
8055ab90  00000000 00000000 00000000 00000000
8055aba0  00000000 00000000 00000000 00000000
8055abb0  00000000 00000000 00000000 00000000
```

在 `windbg.exe` 中我们就看得比较清楚，`KeServiceDescriptorTable` 中就只有第一项有数据，其他都是 0。其中 `804e3d20` 就是 `KeServiceDescriptorTable.ntoskrnel.ServiceTableBase`，服务函数个数为 `0x11c` 个。我们再看看 `804e3d20` 地址里是什么东西：

```text
lkd> dd 804e3d20
804e3d20  80587691 805716ef 8057ab71 80581b5c
804e3d30  80599ff7 80637b80 80639d05 80639d4e
804e3d40  8057741c 8064855b 80637347 80599539
804e3d50  8062f4ec 8057a98c 8059155e 8062661f
```

如上,`80587691` `805716ef` `8057ab71` `80581b5c` 这些就是系统服务函数的地址了。比如当我们在 ring3 调用 `OpenProcess` 时，进入 `sysenter` 的 ID 是 `0x7A`(XP SP2)，然后系统查 `KeServiceDescriptorTable`，大概是这样 `KeServiceDescriptorTable.ntoskrnel.ServiceTableBase(804e3d20) + 0x7A * 4 = 804E3F08`,然后 `804E3F08` -> `8057559e` 这个就是 `OpenProcess` 系统服务函数所在,我们再跟踪看看:

```text
lkd> u 8057559e
nt!NtOpenProcess:
8057559e 68c4000000      push    0C4h
805755a3 6860b54e80      push    offset nt!ObReferenceObjectByPointer+0x127 (804eb560)
805755a8 e8e5e4f6ff      call    nt!InterlockedPushEntrySList+0x79 (804e3a92)
805755ad 33f6            xor     esi,esi
```

原来 `8057559e` 就是 `NtOpenProcess` 函数所在的起始地址。

嗯，如果我们把 `8057559e` 改为指向我们函数的地址呢？比如 `MyNtOpenProcess`，那么系统就会直接调用 `MyNtOpenProcess`，而不是原来的 `NtOpenProcess` 了。这就是 SSDT HOOK 原理所在。

## 3, ring0 inline hook

ring0 inline hook 跟 ring3 的没什么区别了，如果硬说有的话，那么就是 ring3 发生什么差错的话程序会挂掉，ring0 发生什么差错的话系统就挂掉，所以一定要很小心。inline hook 的基本思想就是在目标函数中 JMP 到自己的监视函数，做一些判断然后再 JMP 回去。一般都是修改函数头，不过再其他地方 JMP 也是可以的。下面我们来点实际的吧:

```text
lkd> u nt!NtOpenProcess
nt!NtOpenProcess:
8057559e e95d6f4271      jmp     f199c500
805755a3 e93f953978      jmp     f890eae7
805755a8 e8e5e4f6ff      call    nt!InterlockedPushEntrySList+0x79 (804e3a92)
...
```

同时打开“冰刃”跟“Rootkit Unhooker”我们就能在 `NtOpenProcess` 函数头看到这样的“奇观”,第一个 jmp 是“冰刃”的，第二个 jmp 是“Rootkit Unhooker”的。他们这样是防止被恶意程序通过 `TerminateProcess` 关闭。当然“冰刃”还 Hook 了 `NtTerminateProcess` 等函数。

`××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××`

好了，道理就说完了，下面就进入本文正题。

对付 ring0 inline hook 的基本思路是这样的，自己写一个替换的内核函数，以 `NtOpenProcess` 为例，就是 `MyNtOpenProcess`。然后修改 SSDT 表，让系统服务进入自己的函数 `MyNtOpenProcess`。而 `MyNtOpenProcess` 要做的事就是，实现 `NtOpenProcess` 前 10 字节指令，然后再 JMP 到原来的 `NtOpenProcess` 的十字节后。这样 `NtOpenProcess` 函数头写的 JMP 都失效了，在 ring3 直接调用 `OpenProcess` 再也毫无影响。

`****`

```c
#include<ntddk.h>

typedef struct _SERVICE_DESCRIPTOR_TABLE
{
  PVOID   ServiceTableBase;
  PULONG  ServiceCounterTableBase;
  ULONG   NumberOfService;
  ULONG   ParamTableBase;
}SERVICE_DESCRIPTOR_TABLE,*PSERVICE_DESCRIPTOR_TABLE; //由于 KeServiceDescriptorTable 只有一项,这里就简单点了

extern PSERVICE_DESCRIPTOR_TABLE    KeServiceDescriptorTable;//KeServiceDescriptorTable 为导出函数

/////////////////////////////////////
VOID Hook();
VOID Unhook();
VOID OnUnload(IN PDRIVER_OBJECT DriverObject);
////////////////////////////////////
ULONG JmpAddress;//跳转到 NtOpenProcess 里的地址
ULONG OldServiceAddress;//原来 NtOpenProcess 的服务地址
////////////////////////////////////

__declspec(naked) NTSTATUS __stdcall MyNtOpenProcess(PHANDLE ProcessHandle,
              ACCESS_MASK DesiredAccess,
              POBJECT_ATTRIBUTES ObjectAttributes,
              PCLIENT_ID ClientId)
{
  DbgPrint("NtOpenProcess() called");
  __asm{
    push    0C4h
    push    804eb560h  //共十个字节
    jmp     [JmpAddress]
  }
}
/////////////////////////////////////////////////
NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject,PUNICODE_STRING RegistryPath)
{
  DriverObject->DriverUnload = OnUnload;
  DbgPrint("Unhooker load");
  Hook();
  return STATUS_SUCCESS;
}
/////////////////////////////////////////////////////
VOID OnUnload(IN PDRIVER_OBJECT DriverObject)
{
  DbgPrint("Unhooker unload!");
  Unhook();
}
/////////////////////////////////////////////////////
VOID Hook()
{
  ULONG  Address;
  Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + 0x7A * 4;//0x7A 为 NtOpenProcess 服务 ID
  DbgPrint("Address:0x%08X",Address);

  OldServiceAddress = *(ULONG*)Address;//保存原来 NtOpenProcess 的地址
  DbgPrint("OldServiceAddress:0x%08X",OldServiceAddress);

  DbgPrint("MyNtOpenProcess:0x%08X",MyNtOpenProcess);

  JmpAddress = (ULONG)NtOpenProcess + 10; //跳转到 NtOpenProcess 函数头＋10 的地方，这样在其前面写的 JMP 都失效了
  DbgPrint("JmpAddress:0x%08X",JmpAddress);

  __asm{//去掉内存保护
    cli
        mov  eax,cr0
    and  eax,not 10000h
    mov  cr0,eax
  }

  *((ULONG*)Address) = (ULONG)MyNtOpenProcess;//HOOK SSDT

  __asm{//恢复内存保护
        mov  eax,cr0
    or   eax,10000h
    mov  cr0,eax
    sti
  }
}
//////////////////////////////////////////////////////
VOID Unhook()
{
  ULONG  Address;
  Address = (ULONG)KeServiceDescriptorTable->ServiceTableBase + 0x7A * 4;//查找 SSDT

  __asm{
    cli
        mov  eax,cr0
    and  eax,not 10000h
    mov  cr0,eax
  }

  *((ULONG*)Address) = (ULONG)OldServiceAddress;//还原 SSDT

  __asm{
        mov  eax,cr0
    or   eax,10000h
    mov  cr0,eax
    sti
  }
}
```



```
DbgPrint("Unhook");
}
×××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××

就这么多了，或许有人说，没必要那么复杂，直接恢复`NtOpenProcess`不就行了吗？对于象“冰刃”“Rookit Unhooker”这些“善良”之辈的话是没问题的，但是象`NP`这些“穷凶极恶”之流的话，它会不断检测`NtOpenProcess`是不是已经被写回去，是的话，嘿嘿，机器马上重启。这也是这种方法的一点点妙用。
```

`4dd466d0e1e0cc123afbe504da2462ca`
