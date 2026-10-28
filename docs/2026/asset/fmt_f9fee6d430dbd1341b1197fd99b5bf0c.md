
```c
//SysDefine.h
#pragma once
#define FILE_DEVICE_LY 0xD431
#define IOCTL_CREATE_CALL_GATE (ULONG)CTL_CODE(FILE_DEVICE_LY, 0x801, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define IOCTL_AllOC_SHELLCODE_MEMORY (ULONG)CTL_CODE(FILE_DEVICE_LY, 0x802, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define IOCTL_CHANGE_MEMORY_ATTR (ULONG)CTL_CODE(FILE_DEVICE_LY, 0x803, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define IOCTL_CREATE_INT_GATE (ULONG)CTL_CODE(FILE_DEVICE_LY, 0x804, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define DEVICE_NAME L"\\Device\\lalalasssssss"
#define DRIVER_LINKD L"\\??\\lalala"
#define DRIVER_LINK L"\\\\.\\lalala"
struct CREATE_CALL_GATE
{
void* pFunc;
int argc;
};
```

# 0x01 环境

windows xp

wdk7600

2-9-9-12分页

# 0x02 实现功能

替换`MessageBoxA`的第二个参数

# 0x03 流程

1.在 R0 层申请一块内存，用来放置`shellcode`

2.R3 修改`shellcode`的页属性+us、+rw，写入`shellcode`，`shellcode`末尾的代码需要还原`MessageBoxA`的指令

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26 | `__shellcode:`  `/``/``int` `3`  `mov eax, __end`  `mov[esp` `+` `0xc``], eax`  `pop eax`    `/``*`  `.text:``77D507EA` `8B` `FF                                           mov     edi, edi`  `.text:``77D507EC` `55`                                              `push    ebp`  `.text:``77D507ED` `8B` `EC                                           mov     ebp, esp`  `.text:``77D507EF` `83` `3D` `BC` `14` `D7` `77` `00`                            `cmp`     `_gfEMIEnable,` `0`  `*``/`  `_emit` `0x8B`  `_emit` `0xFF`  `_emit` `0x55`  `_emit` `0x8B`  `_emit` `0xEC`  `_emit` `0x83`  `_emit` `0x3D`  `_emit` `0xBC`  `_emit` `0x14`  `_emit` `0xD7`  `_emit` `0x77`  `_emit` `0x00`  `jmp eax`  `__end :` |

3.创建中断门，负责在调用`shellcode`之前提权修改`shellcode`页属性+us，-NX

4.R3 修改`MessageBoxA`的页属性+us、+rw

5.R3 对`MessageBoxA`进行`inlineHook`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_AKWP9FX2TA4AD4W.png)

替换 4 条指令（12 个字节）：

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `int`     `0``/``/``0``替换成步骤``3``的中断门`  `nop`  `mov     eax,` `0DEADBEEFh``/``/``地址替换成shellcode的地址`  `call    eax`  `nop`  `nop` |

# 0x04 实现效果

测试程序：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `int` `main()`  `{`  `MessageBoxA(``0``,` `"123"``,` `"456"``,` `0``);`  `return` `0``;`  `}` |

运行主程序：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_KARS33YAEUFKYRR.png)

运行测试程序：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_SNSH72DGUDCHXEK.png)

调试测试程序看看指令：

其中 `CD 20` 是 `int 20h`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_PV7PX4ZF99FUNDY.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_NNYVGXS7AB4KW5W.png)

继续运行主程序，取消`hook`：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_X9MHGEFSQCWTJST.png)

运行测试程序：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_68C9CTEM7MXNSME.png)

看到代码已还原：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/861996_6K6AZFFAQQC9J69.png)

# 0x05 源代码

R3

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90 | `/``/``main.cpp`  `#include "Driver.h"`  `#include <stdio.h>`  `#include <windows.h>`    `int` `main(``int` `argc, char``*` `argv[])`  `{`  `Driver d;`  `if` `(!d.loadDriver())`  `{`  `system(``"pause"``);`  `return` `0``;`  `}`  `printf(``"loadDriver suc\n"``);`    `int` `a` `=` `*``(``int``*``)MessageBoxA;`  `printf(``"MessageBoxA: %08x\n"``, a);`    `auto pShellCode` `=` `d.AllocMemory();`  `if` `(NULL` `=``=` `pShellCode)`  `{`  `printf(``"AllocMemory error\n"``);`  `system(``"pause"``);`  `return` `0``;`  `}`  `printf(``"pMem:0x%08x\n"``, pShellCode);`  `d.ChangeMemoryAttr(pShellCode);`  `printf(``"pMem: %s\n"``, pShellCode);`  `char``*` `pShellCodeBegin` `=` `NULL;`  `char``*` `pShellCodeEnd` `=` `NULL;`    `_asm`  `{`  `mov eax, __shellcode`  `mov pShellCodeBegin, eax`  `mov eax, __end`  `mov pShellCodeEnd, eax`  `jmp __end`  `__shellcode:`  `/``/``int` `3`  `mov eax, __end`  `mov[esp` `+` `0xc``], eax`  `pop eax`    `/``*`  `.text:``77D507EA` `8B` `FF                                           mov     edi, edi`  `.text:``77D507EC` `55`                                              `push    ebp`  `.text:``77D507ED` `8B` `EC                                           mov     ebp, esp`  `.text:``77D507EF` `83` `3D` `BC` `14` `D7` `77` `00`                            `cmp`     `_gfEMIEnable,` `0`  `*``/`  `_emit` `0x8B`  `_emit` `0xFF`  `_emit` `0x55`  `_emit` `0x8B`  `_emit` `0xEC`  `_emit` `0x83`  `_emit` `0x3D`  `_emit` `0xBC`  `_emit` `0x14`  `_emit` `0xD7`  `_emit` `0x77`  `_emit` `0x00`  `jmp eax`  `__end :`  `}`  `memcpy(pShellCode, pShellCodeBegin, pShellCodeEnd` `-` `pShellCodeBegin);`    `memcpy(pShellCode, pShellCodeBegin, pShellCodeEnd` `-` `pShellCodeBegin);`  `char``*` `str_pos` `=` `pShellCode` `+` `(pShellCodeEnd` `-` `pShellCodeBegin);`  `memcpy(str_pos,` `"hack"``,` `5``);`  `memcpy(pShellCode` `+` `1``, &str_pos,` `4``);`    `char intGate` `=` `d.CreateIntGate();`  `printf(``"intGate:%d\n"``, intGate);`  `d.ChangeMemoryAttr(MessageBoxA);`  `char hook[]` `=` `{` `"\xCD\x00\x90\xB8\xEF\xBE\xAD\xDE\xFF\xD0\x90\x90"` `};`  `hook[``1``]` `=` `intGate;`  `memcpy(MessageBoxA, hook,` `12``);`  `memcpy((char``*``)MessageBoxA` `+` `4``, &pShellCode,` `4``);`    `/``/``__debugbreak();`  `system(``"pause"``);`    `char MsgBox_old[]` `=` `{``"\x8B\xFF\x55\x8B\xEC\x83\x3D\xBC\x14\xD7\x77\x00"``};`  `memcpy(MessageBoxA, MsgBox_old,` `12``);`  `printf(``"已还原\n"``);`  `system(``"pause"``);`    `return` `0``;`  `}` |

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `/``/``Driver.h`  `#pragma once`  `#include "../xpDrv1/SysDefine.h"`    `class` `Driver`  `{`  `public:`  `bool` `loadDriver();`  `~Driver();`  `short CreateCallGate(CREATE_CALL_GATE``*` `data);`    `char CreateIntGate();`  `char``*` `AllocMemory();`  `bool` `ChangeMemoryAttr(const void``*` `data);`  `};` |

|  |  |

| --- | --- |
