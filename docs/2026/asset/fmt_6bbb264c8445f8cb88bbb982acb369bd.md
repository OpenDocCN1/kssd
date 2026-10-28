

# [原创] 通过暴搜 `DRIVER_OBJECT` 枚举驱动

通过暴搜 `DRIVER_OBJECT` 枚举驱动。

技术可能有点老了，不过希望能够给和我一样的初学者一点参考。

## 背景

先来捣鼓几句，几年前，在内核中战争的焦点总是检测与隐藏。通过内核的特权，我们能够肆意妄为，想改什么就改什么，似乎能想到的，就能够做到。

对于进程的隐藏，方法多得已经数不过来了。曾经，修改内核对象是一件很时髦的事，无论是做隐藏还是检测，无论是好人还是坏人，无论是男人还是女人（哦，说漏嘴了），无论是……是……？但是不管多牛 X![](https://bbs.kanxue.com/view/img/face/50.gif)，而仅仅 `few years` 的时间，一切好像都过时了。

后来又出现了暴力搜索这个东西。当进程被通过修改各种各样的内核对象隐藏时，各种依赖于这些对象的检测方法也渐渐失效。暴力搜索 `EPROCESS` 或 `KPROCESS`、`EHTREAD` 或 `KTHREAD`、`SECTION` 对象等，只要能跟进程扯上关系的，一个也不放过，研究这些成为了大家的最爱。

而后驱动也想通过类似的方法来隐藏自己，即通过断掉 `LDR_DATA_TABLE_ENTRY` 的驱动加载链表，以使利用 `ZwQuerySystemInformation` 的 11 号功能枚举驱动失效。但是，又可以打开驱动对象目录查找驱动的方法来检测到断这种链表的隐藏驱动。

而邪恶者们又能够通过断掉目录对象的 hash 链来自我隐身。当然，一旦这种技术爆料后，我们又有新的方法来搜索——搜索 `TypeList` 来搜索隐藏驱动。不过这种方法有个缺点，需要当 `NtGlobalFlag` 设置了 `MaintainTypeList` 标志时，此链表才有效。到这里，你自然能够想到断掉此链来攻击。我们也不能够束手就擒呀，战争仍在继续![](https://bbs.kanxue.com/view/img/face/27.gif)……

最后我们自然想到了暴搜，暴力搜索 `DRIVER_OBJECT`。这种方法能够很有效地检测出以上的各种方法隐藏的驱动。当然还可以通过搜索内存中的 PE 镜像来搜索，不过没有绝对有效的方法。暴搜驱动对象和 PE 镜像仍然不能够检测到通过抹掉驱动对象和抹掉 PE 头，或者伪造驱动对象和 PE 头的 rootkit。自然还有一些好的方法来检测，如内存搜索 `FileObject`、`SectionObject` 等等。

好了，废话完了，回到主题吧。百度、谷歌上找了一下暴搜驱动，可是没有找到一个完整的，于是就自己来实现了。原来也是如此的简单，估计大牛们要笑话了。不过为了学习交流，我也讲讲吧。不过我是初学者，可能有错误的地方，为了不至于误人子弟，望大牛尽快指出。

## `DRIVER_OBJECT` 结构

要暴力搜索驱动对象，自然关注的焦点是驱动对象的结构。只要会驱动编程的人，对此也再熟悉不过了。`NTDDK.H` 中有，我从 WinDbg 上 dump 下来这个结构：

```text
// nt!\_DRIVER_OBJECT

// +0x000 Type             : Int2B

// +0x002 Size             : Int2B

// 0x004 DeviceObject     : Ptr32 \_DEVICE_OBJECT

// 0x008 Flags            : Uint4B

// +0x00c DriverStart      : Ptr32 Void

// +0x010 DriverSize       : Uint4B

// +0x014 DriverSection    : Ptr32 Void

// +0x018 DriverExtension  : Ptr32 \_DRIVER_EXTENSION

// +0x01c DriverName       : \_UNICODE_STRING

// +0x024 HardwareDatabase : Ptr32 \_UNICODE_STRING

// +0x028 FastIoDispatch   : Ptr32 \_FAST_IO_DISPATCH

// +0x02c DriverInit       : Ptr32     long

// +0x030 DriverStartIo    : Ptr32     void

// +0x034 DriverUnload     : Ptr32     void

// +0x038 MajorFunction    : [28] Ptr32     long
```

既然我们要在内存中搜索驱动对象，那么我们必须找出几乎所有驱动的 `_DRIVER_OBJECT` 都具备的共有特征，以此来准确地识别出此内存段是否是合法的驱动对象。以上用红色标记的是我们要作为驱动对象标记的成员，我们来看看这些成员：

- `Type`：标识对象的类型，其值为 `04`。
- `Size`：驱动对象的大小，其值为 `0xA8`。
- `DriverStart`：其值表示驱动 PE 映像的基地址。因为驱动的映像必定处于高端内存，所以不能小于 `0x80000000` 或为空，同时它的值必须是 `8` 的倍数。
- `DriverSize`：表示驱动的大小，其值不能为 `0`。
- `DriverSection`：驱动节，其实是一个指向 `LDR_DATA_TABLE_ENTRY` 结构的指针，此结构也必在高端内存，也不能小于 `0x80000000`。

就这几条，足够我们唯一标示驱动对象了。我想也应该不难写出辨别驱动对象的代码来，代码如下：

```c
BOOLEAN IsRealDriver( ULONG pAddr )
{
    PDRIVER_OBJECT pTempDriver;

    pTempDriver = (PDRIVER_OBJECT)pAddr;

    if (*(ULONG *)pAddr != 0x00A80004) // 将类型和大小一起验证
    {
        return FALSE;
    }

    if (pTempDriver->DriverSize == 0)
    {
        return FALSE;
    }

    if (/*pTempDriver->DriverStart == NULL || 没有必要*/
        (ULONG)(pTempDriver->DriverStart) < 0x80000000)
    {
        return FALSE;
    }

    if ((ULONG)pTempDriver->DriverSection < 0x80000000)
    {
        return FALSE;
    }

    if (((ULONG)(pTempDriver->DriverStart) & 7) != 0)
    {
        return FALSE;
    }

    return TRUE;
}
```

很简单吧，接下来我们应该确定如何来搜索内存，自然我们要从高端内存开始搜了。

关于搜索范围，根据我的观察，驱动对象几乎都在 `0x8000000` 到 `0x82000000` 之间。当然，如果要想快速的话，就只搜索这段内存；如果想要保险的话，还是 `0x8000000-0xfffff000` 都搜索下吧。这里我还是把所有高端内存都搜一下。

为了能尽快地进行搜索，关于地址的验证，我用了网上的一段用于搜索隐藏进程的地址验证代码来验证。另外，由于我们的对象的地址是 `8` 字节对齐的，因而可以以 `8` 为增量搜索。

当我们搜到了对象，那就好办了。我原本想直接通过对象中的 `DriverName` 来显示驱动名，可是好像大部分的驱动名要么没有后缀，要么名字后面出现乱码（在我的另一个查看系统内核对象的工具中也出现类似的情况，希望哪位大牛有好的解决方法）。但是这里无关紧要，我们还有 `DriverSection` 呢，它指向一个 `LDR_DATA_TABLE_ENTRY` 的结构，大家也再熟悉不过了，定义在代码中，我也不废话了，估计大家也嫌我啰嗦，再说，就要飞砖头了![](https://bbs.kanxue.com/view/img/face/33.gif)（就这点东东，还这么多废话，我仍……![](https://bbs.kanxue.com/view/img/face/50.gif)）。

基本的思路就这么简单。与菜鸟们共同学习交流，我也是菜菜儿，不足之处望大家指出。

## `IsValidAddr` 函数

其中的函数：

```c
ULONG IsValidAddr( ULONG uAddr );
```

是完全照抄过来的，实现代码如下（在 XP SP3 上测试通过）：

```c
#include <ntddk.h>

#define  VALID_PAGE 1
#define  INVALID_PAGE 0
#define  PDEINVALID 2
#define  PTEINVALID 3

typedef struct _LDR_DATA_TABLE_ENTRY {

    LIST_ENTRY InLoadOrderLinks;

    LIST_ENTRY InMemoryOrderLinks;

    LIST_ENTRY InInitializationOrderLinks;

    PVOID DllBase;

    PVOID EntryPoint;

    ULONG SizeOfImage;

    UNICODE_STRING FullDllName;

    UNICODE_STRING BaseDllName;

    ULONG Flags;

    USHORT LoadCount;

    USHORT TlsIndex;

    union {

        LIST_ENTRY HashLinks;

        struct {

            PVOID SectionPointer;

            ULONG CheckSum;

        };

    };

    union {

        struct {

            ULONG TimeDateStamp;

        };

        struct {

            PVOID LoadedImports;

        };

    };

    // struct _ACTIVATION_CONTEXT * EntryPointActivationContext;

    PVOID EntryPointActivationContext;

    PVOID PatchInformation;

} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

typedef struct _DRIVER_ARRAY
{
    PVOID pDriverStart;

    PVOID DllBase;

    ULONG SizeOfImage;

} DRIVER_ARRAY, *PDRIVER_ARRAY;

int number = 0;

BOOLEAN IsRealDriver( ULONG pAddr );

ULONG IsValidAddr( ULONG uAddr );

void ShowHidedDriver();

void UnLoad( PDRIVER_OBJECT pDriverObject )
{
    DbgPrint("Driver Unload..\n");
}

NTSTATUS DriverEntry(
    PDRIVER_OBJECT pDriverObject,
    PUNICODE_STRING pRegPath
)
{
    NTSTATUS Status = STATUS_SUCCESS;

    pDriverObject->DriverUnload = UnLoad;

    KdPrint(("Driver object Address:0x%08x\n", pDriverObject));

    KdPrint(("CURRENT DRIVER %S\n", pDriverObject->DriverName.Buffer));

    ShowHidedDriver();

    return Status;
}

// 以下这个函数抄的网上的，直接拿来主义....

ULONG IsValidAddr( ULONG uAddr )
{
    ULONG uInfo;

    ULONG uCr4;

    ULONG uPdeAddr;

    ULONG uPteAddr;

    _asm
    {
        cli

        push eax

        _emit 0x0F

        _emit 0x20

        _emit 0xE0 // mov eax, cr4

        mov [uCr4], eax

        pop eax
    }

    _asm sti

    uInfo = uCr4 & 0x20;

    if ( uInfo != 0 )
    {
        uPdeAddr = (uAddr >> 21) * 8 + 0xC0600000;
    }
    else
    {
        uPdeAddr = (uAddr >> 22) * 4 + 0xC0300000;
    }

    if ( (*(PULONG)uPdeAddr & 0x1) != 0 )
    {
        if ( (*(PULONG)uPdeAddr & 0x80) != 0 )
        {
            return VALID_PAGE;
        }
        else
        {
            if ( uInfo != 0 )
            {
                uPteAddr = (uAddr >> 12) * 8 + 0xC0000000;
            }
            else
            {
                uPteAddr = (uAddr >> 12) * 4 + 0xC0000000;
            }

            if ( (*(PULONG)uPteAddr & 0x1) != 0 )
            {
                return VALID_PAGE;
            }
            else
            {
                return PTEINVALID;
            }
        }
    }
    else
    {
        return PDEINVALID;
    }
}

// 判定是否是真正的驱动结构

BOOLEAN IsRealDriver( ULONG pAddr )
{
    PDRIVER_OBJECT pTempDriver;

    pTempDriver = (PDRIVER_OBJECT)pAddr;

    if (*(ULONG *)pAddr != 0x00A80004) // 类型和大小一起验证了
    {
        return FALSE;
    }

    if (pTempDriver->DriverSize == 0) // 驱动大小不能为 0
    {
        return FALSE;
    }

    if (/*pTempDriver->DriverStart == NULL || 可以不要*/
        (ULONG)(pTempDriver->DriverStart) < 0x80000000)
    {
        return FALSE;
    }

    if ((ULONG)pTempDriver->DriverSection < 0x80000000)
    {
        return FALSE;
    }

    if (((ULONG)(pTempDriver->DriverStart) & 7) != 0)
    {
        return FALSE;
    }

    return TRUE;
}

void ShowHidedDriver()
{
    ULONG uStartAddr = 0x80000000;

    ULONG PEStart;

    PDRIVER_OBJECT TempDriver;

    PLDR_DATA_TABLE_ENTRY Ldr = NULL;

    ULONG Count = 0;

    KdPrint(("show Information.....\n"));

    for (;
         uStartAddr <= (ULONG)0xfffff000;
         uStartAddr += 8)
    {
        ULONG uRet = IsValidAddr(uStartAddr);

        if (uRet == VALID_PAGE)
        {
            if (IsRealDriver(uStartAddr))
            {
                KdPrint(("Driver object Address:0x%08x\n", uStartAddr));

                TempDriver = (PDRIVER_OBJECT)uStartAddr;

                Ldr = (PLDR_DATA_TABLE_ENTRY)TempDriver->DriverSection;

                if (MmIsAddressValid(Ldr))
                {
                    DbgPrint("DriveName : %S\n", Ldr->FullDllName.Buffer);

                    DbgPrint("ImageBase : 0x%08X.\n", Ldr->DllBase);

                    DbgPrint("ImageSize : 0x%08X.\n", Ldr->SizeOfImage);

                    DbgPrint("EntryPoint : 0x%08X.\n", Ldr->EntryPoint);

                    DbgPrint("@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\n");

                    Count++;
                }

                uStartAddr += sizeof(DRIVER_OBJECT);
            }
        }
        else if (uRet == PDEINVALID)
        {
            uStartAddr -= 8;

            uStartAddr += 0x400000;
        }
        else
        {
            uStartAddr -= 8;

            uStartAddr += 0x1000;
        }
    }

    DbgPrint("Driver Number %d", Count);
}
```

