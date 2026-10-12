
# 2 个字节的 pediy

> 原文：[`www.pediy.com/kssd/pediy10/73936.html`](https://www.pediy.com/kssd/pediy10/73936.html)

主要思路: hook `KiUserExceptionDispatcher` 异常处理函数,然后返回到自己的 打开网页 call,`OpenUlrA` 2 个字节就可以搞定,呵呵不知道是否符合题目的要求?

简单说下自己的做法:

## 1, 导出函数 `OpenUlrA` 制造一个异常,这里我用了 `CC`, 具体汇编代码如下:

```
000010D0  >    CC            int3
000010D1   .  C3            retn
```

增加导出表:

对照 `pe` 结构

`148` 导出表的地址,`14C` 导出表的大小

在 `148` 的地方 输入 导出表的地址 `21A0` 和 大小 `45`

```
00000148    A0210000    DD 000021A0          ;  Export Table address = 21A0
0000014C    45000000    DD 00000045          ;  Export Table size = 45 (69.)
```

然后在 `21A0` 的地址 对照 导出表格式,进行修改

导出表结构:

```
typedef struct _IMAGE_EXPORT_DIRECTORY {
Dword Characteristics;
Dword TimeDateStamp;
word MajorVersion;
word MinorVersion;
Dword Name;
Dword Base;
Dword NumberOfFunctions;
Dword NumberOfNames;
Dword AddressOfFunctions;  // RVA from base of image
Dword AddressOfNames;  // RVA from base of image
Dword AddressOfNameOrdinals;  // RVA from base of image
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

`IMAGE_EXPORT_DIRECTORY` 结构成员含义:

`Characteristics`: 此域没有用途,总是为 0。

`TimeDateStamp`: 程序被生成的时刻。

`MajorVersion`/`MinorVersion`: 无实际用途,0。

`Name`: 一个 `RVA` 值,指向一个 `ASCIIZ` 字串(`dll` 名称,如 `MYDLL.dll`)。模块的真实名称。本域是必须的,因为文件名可能会改变。这种情况下,`PE` 装载器将使用这个内部名字。

`Base`: 基数,加上序数就是函数地址数组的索引值了。

`NumberOfFunctions`: 模块导出的函数/符号总数。

`NumberOfNames`: 通过名字导出的函数/符号数目。该值不是模块导出的函数/符号总数,这是由上面的 `NumberOfFunctions` 给出。本域可以为 0,表示模块可能仅仅通过序数导出。如果模块根本不导出任何函数/符号,那么数据目录中导出表的 `RVA` 为 0。

`AddressOfFunctions`: 模块中有一个指向所有函数/符号的 `RVA` 数组,本域就是指向该 `RVA` 数组的 `RVA`。简言之,模块中所有函数的 `RVA` 都保存在一个数组里,本域就指向这个数组的首地址。

`AddressOfNames`: 类似上个域,模块中有一个指向所有函数名的 `RVA` 数组,本域就是指向该 `RVA` 数组的 `RVA`。

`AddressOfNameOrdinals`: `RVA`,指向包含上述 `AddressOfNames` 数组中相关函数之序数的 16 位数组。

我修改的导出表:

```
000021A0  00 00 00 00 A8 76 E3 48 00 00 00 00 D2 21 00 00  ........?..
000021B0  01 00 00 00 01 00 00 00 01 00 00 00 C8 21 00 00  .........?..
000021C0  CC 21 00 00 D0 21 00 00 D0 10 00 00 DC 21 00 00  ?..?..?..?..
000021D0  00 00 70 65 64 69 79 2E 64 6C 6C 00 4F 70 65 6E    ..pediy.dll.Open
000021E0  55 72 6C 41 00 00 00 00 00 00 00 00 00 00 00 00  UrlA............
```

## 2. 需要的增加导入函数:hook `KiUserExceptionDispatcher` 需要的 `VirtualProtectEx` 函数, 异常处理函数 `KiUserExceptionDispatcher`, 打开网页函数,这里我用 `ShellExecuteExA` 一个参数, 在这之前,提交了一个用 `ShellExecuteExA` 的 `pediy` 12 字节

修改导入表:

`150` 导入表的地址,`154` 导入表的大小

这里我在 `150` 输入 `2028` 和 大小 `78`

```
00000150    28200000    DD 00002028          ;  Import Table address = 2028
00000154    78000000    DD 00000078          ;  Import Table size = 78 (120.)
```

然后在 `2028` 的地址 对照 导入表格式,进行修改

导入表结构:

```
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
union {
Dword Characteristics;  // 0 for terminating null import descriptor
Dword OriginalFirstThunk;  // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
};
Dword TimeDateStamp;  // 0 if not bound,
                      // -1 if bound, and real date/time stamp
                      // in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                      // O.W. date/time stamp of DLL bound to (Old BIND)
Dword ForwarderChain;  // -1 if no forwarders
Dword Name;
Dword FirstThunk;  // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
```

`IMAGE_IMPORT_DESCRIPTOR` 结构成员含义:

结构第一项是一个 `union` 子结构。事实上,这个 `union` 子结构只是给 `OriginalFirstThunk` 增添了个别名,您也可以称其为"Characteristics"。该成员项含有指向一个 `IMAGE_THUNK_DATA` 结构数组的 `RVA`。

`TimeDateStamp`: 程序生成的时刻。此域通常为 0。微软的 `BIND` 程序可以将此 `IMAGE_IMPORT_DESCRIPTOR` 所对应的 `dll` 的生成时刻写到这里来。

`ForwarderChain`: 此域涉及到 `forwarding`(转交),意味着一个 `dll` 函数在调用另一个 `dll`。例如,在 `WINNT` 中,`Kernel32.dll` 将它的某些输出函数转交给 `NTDLL.dll`。应用程序可能以为它调用 `Kernel32.dll`,而事实上它调用的事 `NTDLL.dll`。这个域中含有一个索引,指向 `FirstThunk` 数组。被这个索引所指定的函数就是一个转交函数。

`Name`: 含有指向 `DLL` 名字的 `RVA`,即指向 `DLL` 名字的指针,也是一个 `ASCII` 字符串。

`FirstThunk`: 与 `OriginalFirstThunk` 非常相似,它也包含指向一个 `IMAGE_THUNK_DATA` 结构数组的 `RVA`(当然这是另外一个 `IMAGE_THUNK_DATA` 结构数组)。

我修改的导入表:

```
00002020 >                                       B0 20 00 00 00 00 00 00          .....
00002030  00 00 00 00 D6 20 00 00 10 20 00 00 B8 20 00 00  ....?..  ..?..
00002040  00 00 00 00 00 00 00 00 34 21 00 00 18 20 00 00  ........4!.. ..
00002050  C0 20 00 00 00 00 00 00 00 00 00 00 5C 21 00 00  ?..........\\!..
00002060  20 20 00 00 A0 20 00 00 00 00 00 00 00 00 00 00    ..?..........
00002070  14 21 00 00 00 20 00 00 00 00 00 00 00 00 00 00  !...  ..........
00002080  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00002090  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
000020A0  02 21 00 00 F6 20 00 00 E2 20 00 00 00 00 00 00  !..?..?......
```

## 3 我 hook `KiUserExceptionDispatcher` 代码:

```
008B10E0   > \\50            push    eax
008B10E1   .  8D4424 00     lea     eax, dword ptr [esp]
008B10E5   .  C74424 00 000>mov     dword ptr [esp], 0
008B10ED   .  50            push    eax                              ; /pOldProtect
008B10EE   .  6A 40         push    40                               ; |NewProtect = PAGE_EXECUTE_READWRITE
008B10F0   .  6A 08         push    8                                ; |Size = 8
008B10F2   .  FF35 20208B00 push    dword ptr [<&NTDLL.KiUserExcepti>; |Address = ntdll.KiUserExceptionDispatcher
008B10F8   .  6A FF         push    -1                               ; |hProcess = FFFFFFFF
008B10FA   .  FF15 08208B00 call    dword ptr [<&KERNEL32.VirtualPro>; \\VirtualProtectEx
008B1100   .  837D 0C 01    cmp     dword ptr [ebp+C], 1
008B1104   .  A1 20208B00   mov     eax, dword ptr [<&NTDLL.KiUserEx>
008B1109   .  75 22         jnz     short 008B112D                   ;  判断 dll 是否释放
008B110B   .  BB C0308B00   mov     ebx, 008B30C0
008B1110   .  8B08          mov     ecx, dword ptr [eax]
008B1112   .  890B          mov     dword ptr [ebx], ecx
008B1114   .  8B48 04       mov     ecx, dword ptr [eax+4]
008B1117   .  894B 04       mov     dword ptr [ebx+4], ecx
008B111A   .  8B1D BC308B00 mov     ebx, dword ptr [8B30BC]          ;  pediy.008B115C
008B1120   .  2BD8          sub     ebx, eax
008B1122   .  83EB 05       sub     ebx, 5
008B1125   .  C600 E9       mov     byte ptr [eax], 0E9              ;  调到自己的 Call
008B1128   .  8958 01       mov     dword ptr [eax+1], ebx
008B112B   .  EB 0F         jmp     short 008B113C
008B112D   >  BB C0308B00   mov     ebx, 008B30C0                    ;  dll 释放，恢复 KiUserExceptionDispatcher 函数头
008B1132   .  8B0B          mov     ecx, dword ptr [ebx]
008B1134   .  8908          mov     dword ptr [eax], ecx
008B1136   .  8B4B 04       mov     ecx, dword ptr [ebx+4]
008B1139   .  8948 04       mov     dword ptr [eax+4], ecx
008B113C   >  8B1C24        mov     ebx, dword ptr [esp]
008B113F   .  50            push    eax                              ; /pOldProtect
008B1140   .  53            push    ebx                              ; |NewProtect
008B1141   .  6A 08         push    8                                ; |Size = 8
008B1143   .  FF35 20208B00 push    dword ptr [<&NTDLL.KiUserExcepti>; |Address = ntdll.KiUserExceptionDispatcher
008B1149   .  6A FF         push    -1                               ; |hProcess = FFFFFFFF
008B114B   .  FF15 08208B00 call    dword ptr [<&KERNEL32.VirtualPro>; \\VirtualProtectEx
008B1151   .  58            pop     eax
008B1152   .  C2 0C00       retn    0C
```

## 4 自己的代码:

```
008B115C   .  3D D0108B00   cmp     eax, OpenUrlA                    ;  比较异常地址是否是自己的 cc
008B1161      74 12         je      short 008B1175
008B1163   .  8B4C24 04     mov     ecx, dword ptr [esp+4]
008B1167   .  8B1C24        mov     ebx, dword ptr [esp]
008B116A   .  51            push    ecx
008B116B   .  A1 20208B00   mov     eax, dword ptr [<&NTDLL.KiUserEx>
008B1170   .  83C0 08       add     eax, 8
008B1173   .  FFE0          jmp     eax
008B1175   .  68 80308B00   push    008B3080                         ;  执行打开网页的功能
```
