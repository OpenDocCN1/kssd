
# [原创]PC病毒分析师所需技能和面试题

今天整理硬盘突然看到几年前做PC病毒分析师时整理的自己一些面试题和一些网上收集的面试题，还有一些可能有用的东西。放上来希望能给对病毒分析感兴趣和或者希望从事这个行业的同学一些帮助和指导。（其中的一些面试题例子我是用加密解密这本书中的例子做的解答）

---

## InLine HOOK 中 `_declspec(naked)` 使用

对于 `jmp` 类型的 hook，如果自己的过程没有使用 `_declspec(naked)`，那么系统会自动给添加一些额外的代码，控制堆栈平衡，但是这些额外的代码会破坏被 hook 函数的堆栈。

对于 `call` 类型的 hook，如果使用 `_declspec(naked)` 修饰的话，要注意自己恢复堆栈平衡。

下面是网上对 `_declspec(naked)` 的介绍：

`_declspec(naked)` 就是告诉编译器，在编译的时候，不要优化代码，通俗地说就是没代码，完全要自己写。

比如：

```c
#define NAKED        __declspec(naked)

void NAKED code(void)
{
    __asm
    {
        ret
    }
}
```

使用 `__declspec(naked)` 关键字定义函数：

1. 使用 `naked` 关键字必须自己构建 `EBP` 指针（如果用到了的话）;
2. 必须自己使用 `RET` 或 `RET n` 指令返回（除非你不返回）;

`_delcspec(naked)` 用在驱动编写，C 语言内嵌汇编完成一些特定功能。

---

## 反虚拟机检测

一般程序中检测虚拟机是通过检测虚拟机的后门（VMwar 系统存在后门，用于虚拟系统和真实系统的交互）来判断是否程序运行在虚拟机中。检测虚拟机的代码：

```cpp
bool IsInsideVMWare()
{
    bool rc = true;

    __try
    {
        __asm
        {
            push   edx
            push   ecx
            push   ebx
            mov    eax, 'VMXh'
            mov    ebx, 0                                 // any value but not the MAGIC VALUE
            mov    ecx, 10                                 // get VMWare version
            mov    edx, 'VX'                         // port number
            in     eax, dx                                  // read port
            // on return EAX returns the VERSION
            cmp    ebx, 'VMXh'                  // is it a reply from VMWare?
            setz   [rc]                                         // set return value
            pop    ebx
            pop    ecx
            pop    edx
        }
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
        rc = false;
    }

    return rc;
}
```

`eax` 里面放着 `'VMXh'` 那就是虚拟机了，注意字符串的顺序，`'V'` 是高位，`'h'` 是低位，其他字符串也是这样。

第一步要做的就是定位关键代码，我选择定位字符串 `VMXh`，搜索 `hXMV` 的 ASCII 码（注意高位低位！）

```asm
004013C6   .  52            push edx
004013C7   .  51            push ecx
004013C8   .  53            push ebx
004013C9   .  B8 68584D56   mov eax,0x564D5868
004013CE   .  BB 00000000   mov ebx,0x0
004013D3   .  B9 0A000000   mov ecx,0xA
004013D8   .  BA 58560000   mov edx,0x5658
004013DD   .  ED            in eax,dx
004013DE   .  81FB 68584D56 cmp ebx,0x564D5868
004013E4   .  0F9445 E4     sete byte ptr ss:[ebp-0x1C]
004013E8   .  5B            pop ebx
004013E9   .  59            pop ecx
004013EA   .  5A            pop edx
004013EB   .  EB 0B         jmp Xalg1.004013F8
004013ED   .  6A 01         push 0x1
004013EF   .  58            pop eax
004013F0   .  C3            retn
004013F1   .  8B65 E8       mov esp,dword ptr ss:[ebp-0x18]
004013F4   .  8365 E4 00    and dword ptr ss:[ebp-0x1C],0x0
004013F8   >  834D FC FF    or dword ptr ss:[ebp-0x4],0xFFFFFFFF
004013FC   .  8B45 E4       mov eax,dword ptr ss:[ebp-0x1C]
004013FF   .  8B4D F0       mov ecx,dword ptr ss:[ebp-0x10]
00401402   .  64:890D 00000>mov dword ptr fs:[0],ecx
00401409   .  5F            pop edi
0040140A   .  5E            pop esi
0040140B   .  5B            pop ebx
0040140C   .  C9            leave
0040140D   .  C3            retn
……
0040137F   .  C9            leave
00401380   .  C3            retn
00401381  /$  E8 0F000000   call alg1.00401395
00401386  |.  85C0          test eax,eax     //返回的关键值  根据返回值判断
00401388  |.  75 07         je Xalg1.00401391   更改指令为jne 401391   或者其它跳转指令即可越过虚拟机检测
0040138A  |.  6A 00         push 0x0
0040138C  |.  E8 8B410000   call alg1.0040551C
00401391  |>  6A 01         push 0x1
00401393  |.  58            pop eax
00401394  \.  C3            retn
00401395   $  55            push ebp
```

在它传回返回值时修改跳转指令即可越过虚拟机检测。

## 怎么定位导入函数的地址

`PE` 头位置 + `78H` 得到 `IMAGE_DATA_DIRECTORY`（数据目录表）的位置（也可直接 `PE` 头位置 + `80H` 直接得到 `Import Table` 的位置）

```asm
IMAGE_DATA_DIRECOTRY STRUC
        VirtualAddress        DWORD                ?                ;数据块起始地址
        Size                                DWORD                ?                ;数据块的长度
IMAGE_DATA_DIRECOTRY ENDS
```

假设 `PE` 文件头的起始地址为 `B0h`，则输入表地址为 `B0h + 80h = 130h`，在 `130h` 处发现四字节指针 `40 20 00 00`，倒过来为 `2040` 即输入表在内存中偏移量。

`IMAGE_DATA_DIRECTORY` 第二个元素为 `Import Table`，输入表以一个 `IMAGE_IMPORT_DESCRIPTOR`（简称 `IID`）数组开始。

假设 `2040` 位于 `.rdata` 块中，其虚拟偏移为 `2000h`，物理偏移为 `600h`，因此 `k = 2000h - 600h = 1A00h`。相对虚拟地址 `2040h` 转化为物理地址 `2040 - 1A00h = 640h`。`640h` 处为 `IMAGE_IMPORT_DESCRIPTOR`。

```asm
IMAGE_IMPORT_DESCRIPTOR STRUC
        union
                Characteristics                                DWORD
                OriginalFirstThunk                        DWORD                ?指向输入名称表(简称INT)
        ends
TimeDateStamp                                DWORD                ?时间标志 可以忽略
ForwarderChain                                DWORD                ?第一个被转向的API索引，一般为0
Name                                                DWORD                ?DLL名字的指针
        FirstThunk                                        DWORD                ?指向输入地址表(简称IAT)
IMAGE_IMPORT_DESCRIPTOR ENDS
```

`FirstThunk` 和 `OriginalFirstThunk` 非常类似，指向相同的数组 `IMAGE_THUNK_DATA` 结构数组。

假设 `Name` 的值为 `7421 0000`，倒过来为虚拟偏移地址 `2174h`，物理偏移地址 `2174h - 1A00h = 774h`，查看文件中 `774h` 的字符就是调用的 DLL 名字。

`OriginalFirstThunk` 处的值为 `8C20 0000`，倒过来为 `208Ch`，物理地址为 `208Ch - 1A00h = 68Ch`，`68Ch` 处就是 `IMAGE_THUNK_DATA` 数组。

```asm
IMAGE_THUNK_DATA STRUC
        union  u1
                ForwarderString                DWORD                ?指向一个转向者字符串的RVA
                Function                                DWORD                ?被输入的函数的内存地址
                Ordinal                                DWORD                ?被输入的API的序数值
                AddressOfData                DWORD                ?指向IMAGE_IMPORT_BY_NAME
        ends
IMAGE_THUNK_DATA ENDS

IMAGE_IMPORT_BY_NAME STRUC
        Hint                                                WORD                ?
        Name                                         BYTE                ?
IMAGE_IMPORT_BY_NAME ENDS
```

假设的其中一个 `IMAGE_THUNK_DATA` 值是 `1021 0000`，倒过来即为虚拟偏移地址 `2110h`，物理偏移地址 `2110h - 1A00h = 710h`，偏转到 `710` 处位置就会看到函数名字字符串。

- 注入方式怎么实现
- 反调试技术
- 反虚拟机检测
- 花指令解决办法
- IDC 脚本
- 网络数据分析
- 调试方法

---

## 汇编知识

```asm
Mov dl,0dh        回车控制字符0Dh
Mov dl,0ad        换行控制字符0Ah
Mov al,[bx]
or al,al                判断选取的字符是否为零
jz done
or al,20h                转化小写字母为大写字母（使用D5=1）
```

串操作指令 `LODSB`/`LODSW` 是块装入指令，其具体操作是把 `SI` 指向的存储单元读入累加器，其中 `LODSB` 是读入 `AL`，`LODSW` 是读入 `AX` 中，然后 `SI` 自动增加或减小 1 或 2 位。当方向标志位 `D=0` 时，则 `SI` 自动增加；`D=1` 时，`SI` 自动减小。

```asm
test   eax,   eax
je   00401083
```

`test` 指令操作是目的操作数和源操作数按位逻辑“与”运算，结果不送回目的操作数然后根据结果设置 `SF`、`ZF`、和 `PF` 标志位，并将 `CF` 和 `OF` 标志位清零。而 `JE` 是当 `ZF=1` 时跳转。

即，当 `eax` 的值等于 0 时跳转。因此说，这里的 `test` 就是检测 `eax` 的值是不是 0。

```asm
mov eax,0a
text eax,eax
je  xxxx
```

这个跳会跳吗？能详细解释下 `test eax，eax` 这是什么意思？

`test eax,eax` 基本上和 `And eax,eax` 是一样的，不同的是 `test` 不改变 `eax` 的结果，只是改变 `FLAG` 寄存器的状态，也就是改变进位标志，零标志，溢出标志等。

举一个例子，如果 `eax = 01h`，`test eax, eax` 就是两个 `01h` 作与操作，所以结果还是 `01h`，不是 0 的话，就不会跳转 `je xxxx`。所以要跳转 `je xxxx`，只有一种可能就是 `eax = 0h`。所以现在 `eax = 0a` 则不会跳转 `je xxxx`。

```asm
push    offset LibFileName ; "Raining.dll"
call    ds:LoadLibraryA ; 加载Raining.dll
test    eax, eax        ; 检测Raining.dll是否加载成功
jnz     short loc_401012
```

---

## 题目1：写出DLL劫持原理，并写出哪些DLL 不能被劫持

