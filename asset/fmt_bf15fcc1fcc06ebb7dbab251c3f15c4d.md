
# 驱动中实现模拟键盘按键

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-121.htm`](https://www.pediy.com/kssd/pediy09/pediy09-121.htm)

> 在 `ring3` 中实现模拟键盘按键有 `N^N` 种方式，比如 `SendInput()`、`keybd_event()`……但在驱动中要怎么模拟呢？

### 1、写端口大法

```c
#define defI8042_DATA_PORT ((PUCHAR)0x60)
#define defI8042_CTRL_PORT ((PUCHAR)0x64)
#define defOBUFFER_FULL 0x01

BOOLEAN
WaitForKeyboardWrite(VOID)
{
    INT i;
    UCHAR c;

    for (i = 0; i < 1000; ++i)
    {
        KeStallExecutionProcessor(50);
        c = READ_PORT_UCHAR(defI8042_CTRL_PORT);
        if ((c & defOBUFFER_FULL) == defOBUFFER_FULL)
            break;
    }

    return i ? TRUE : FALSE;
}

VOID PressKeyByScanCode(
    IN CONST BYTE ScanCode
)
{
    WRITE_PORT_UCHAR(defI8042_CTRL_PORT, 0xd2);

    WaitForKeyboardWrite();

    WRITE_PORT_UCHAR(defI8042_DATA_PORT, ScanCode);
}
```

搞定，简单易行！缺点是只对 PS/2 键盘有效，USB 的就一边凉快去吧。

### 2、构造 IRP 大法

这个比较麻烦，而且由于某些原因，略……

### 3、调用 `KeyboardClassServiceCallback()` 大法

`kbdclass` 驱动是在 `I8042prt` 和 `kbdhid` 这两个驱动之上的，所以直接调用它的 `KeyboardClassServiceCallback()` 是能对 PS/2 和 USB 同时生效的，由于某些原因偶也不想给出具体的代码来。大致指点一下方向：

`KeyboaredClassServiceCallback` 函数的原型如下：

```c
typedef VOID (* PFN_KeyboardClassServiceCallback)(
    IN PDEVICE_OBJECT DeviceObject,
    IN PKEYBOARD_INPUT_DATA InputDataStart,
    IN PKEYBOARD_INPUT_DATA InputDataEnd,
    IN OUT PULONG InputDataConsumed
    );
```

第一个参数 `DeviceObject` 是 `kbdclass` 的 `DeviceObject`，怎么得到它呢？思路：先找到 `kbdclass` 的 `DriverObject`，然后从 `DriverObject` 得到 `DeviceObject`。

第二和第三个参数是 `KEYBOARD_INPUT_DATA`，自己填充，想模拟什么按键就写什么 `MakeCode` 和 `Flags` 吧。注意第三个参数应该为第二个参数的指针地址值+1。

第四个参数，随便弄个 `ULONG` 型的变量给它就行了，反正是可以丢弃的。（`I8042prt` 里面用到了它，但我们自己的模拟按键不用也行）

缺点：每个系统的 `kbdclass.sys` 中的 `KeyboardClassServiceCallback` 的 `RVA` 都不相同，要自己 `hardcode` 一下，不过写驱动本来就是那么麻烦，习惯了就好。
