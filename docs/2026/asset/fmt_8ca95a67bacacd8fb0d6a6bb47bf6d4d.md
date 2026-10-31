
# [原创]Linux 平台 OffByOne（基于栈）漏洞

前一段时间在看雪上看到 [hackyzh](http://bbs.pediy.com/user-694899.htm) 翻译的《Linux (x86) Exploit 开发系列教程之三（Off-By-One 漏洞（基于栈））》文章，后面又查看了英文原文，`d72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6k6Q4x3V1j5H3y4#2)9J5c8X3!0X3k6W2)9J5k6r3u0&6i4K6u0V1L8$3&6W2i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5X3q4K6k6h3c8Q4x3X3b7J5i4K6u0r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1#2i4K6S2q4i4K6W2r3i4@1f1$3i4K6V1$3i4K6R3%4i4@1f1@1i4@1u0p5i4K6W2o6i4@1f1^5i4K6R3H3i4K6R3#2i4@1f1%4i4K6W2m8i4K6R3@1k6i4S2H3i4@1f1^5i4@1t1%4i4K6V1I4i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1&6i4K6R3H3i4K6W2m8i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1@1i4@1t1^5i4K6V1@1i4@1f1&6i4K6R3K6i4@1p5^5i4@1f1#2i4K6R3^5i4K6R3$3i4@1f1#2i4K6W2o6i4@1t1H3i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1^5i4@1q4q4i4@1t1J5i4@1f1^5i4@1p5%4i4@1p5K6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1#2i4K6R3^5i4@1p5&6i4@1f1@1i4@1u0m8i4K6S2q4i4@1f1%4i4K6V1H3i4K6R3$3i4@1f1^5i4@1p5%4i4@1p5K6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6R3&6i4K6R3H3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1$3i4K6S2o6i4K6R3&6i4@1f1%4i4K6R3#2i4@1p5%4i4@1f1^5i4K6R3%4i4@1q4m8i4@1f1#2i4@1t1%4i4@1t1I4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4K6V1H3i4K6R3$3i4@1f1^5i4@1p5%4i4@1p5K6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1&6i4K6R3%4i4K6S2p5i4@1f1$3i4K6V1$3i4@1t1H3i4@1f1$3i4K6V1#2i4@1t1@1i4@1f1%4i4K6V1H3i4K6R3$3i4@1f1@1i4@1u0m8i4K6R3$3L8$3k6X3j5Y4W2G2L8X3g2Q4c8e0g2Q4z5f1k6Q4b7V1q4Q4c8e0c8Q4b7V1q4Q4z5p5g2Q4c8e0k6Q4b7e0m8Q4z5o6S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4b7V1y4Q4z5p5k6Q4c8e0k6Q4b7U0c8Q4z5f1g2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5e0m8Q4z5p5y4Q4c8e0k6Q4z5e0N6Q4b7U0k6Q4c8e0W2Q4z5e0W2Q4z5o6c8Q4c8e0c8Q4b7U0S2Q4z5p5q4Q4c8e0N6Q4z5f1u0Q4b7U0S2Q4c8e0g2Q4z5o6g2Q4b7U0y4W2P5s2m8Q4c8e0y4Q4z5o6m8Q4z5o6t1`。

## 基于栈的 OffByOne

### 一、漏洞原因

将源字符串复制到目标缓冲区时，可能会导致 off by one。当源字符串长度等于目标缓冲区的长度时，单个 `NULL` 字节将被复制到目标缓冲区上方。由于目标缓冲区位于堆栈中，所以单个 `NULL` 字节可以覆盖存储在堆栈中的调用者的 `EBP` 的最低有效位（`LSB`），这可能导致任意代码执行（此处 copy 的原文）。

### 二、漏洞攻击演示

漏洞演示环境：Ubuntu 12.04-32位

漏洞代码：

```c
#include <stdio.h>
#include <string.h>

void foo(char* arg);
void bar(char* arg);

void foo(char* arg) {
    bar(arg); /* [1] */
}

void bar(char* arg) {
    char buf[256];
    strcpy(buf, arg); /* [2] */
}

int main(int argc, char *argv[]) {
    if(strlen(argv[1])>256) { /* [3] */
        printf("Attempted Buffer Overflow\n");
        fflush(stdout);
        return -1;
    }
    foo(argv[1]); /* [4] */
    return 0;
}
```

在编译时，先切换到 `root`，关闭 Linux 的地址随机化（`ASLR`）特性，命令如下：

```bash
echo 0 > /proc/sys/kernel/randomize_va_space
```

（恢复）开启 `ASLR` 命令如下：

```bash
echo 2 > /proc/sys/kernel/randomize_va_space
```

编译命令如下：

```bash
gcc -fno-stack-protector -z execstack -mpreferred-stack-boundary=2 -o vuln -g offbyone.c
```

编译命令的详细解释，放到了 3.1 章节中。

上述漏洞代码的第 `[2]` 行可能是发生 off by one 溢出的地方。目标缓冲区长度为 `256`，因为长度为 `256` 字节的源字符串可能导致任意代码执行。

#### 漏洞利用

我们来看一下上述代码的堆栈布局，确定调用者的 `EBP` 是否位于目标缓冲区之上，也就是 `strcpy()` 之后，单个 `NULL` 字节是否覆盖调用者的 `EBP` 的 `LSB`？

##### `main()` 函数

```asm
.text:08048497 ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:08048497                 public main
.text:08048497 main            proc near               ; DATA XREF: _start+17o
.text:08048497
.text:08048497 var_8           = dword ptr -8
.text:08048497 argc            = dword ptr  8
.text:08048497 argv            = dword ptr  0Ch
.text:08048497 envp            = dword ptr  10h
.text:08048497
.text:08048497                 push    ebp
.text:08048498                 mov     ebp, esp
.text:0804849A                 push    edi
.text:0804849B                 sub     esp, 8
.text:0804849E                 mov     eax, [ebp+argv]
.text:080484A1                 add     eax, 4
.text:080484A4                 mov     eax, [eax]
.text:080484A6                 mov     [ebp+var_8], 0FFFFFFFFh
.text:080484AD                 mov     edx, eax
.text:080484AF                 mov     eax, 0
.text:080484B4                 mov     ecx, [ebp+var_8]
.text:080484B7                 mov     edi, edx
.text:080484B9                 repne scasb
.text:080484BB                 mov     eax, ecx
.text:080484BD                 not     eax
.text:080484BF                 sub     eax, 1
.text:080484C2                 cmp     eax, 100h
.text:080484C7                 jbe     short loc_80484EA
.text:080484C9                 mov     eax, offset format ; "attepted buffer overflow"
.text:080484CE                 mov     [esp], eax      ; format
.text:080484D1                 call    _printf
.text:080484D6                 mov     eax, ds:stdout@@GLIBC_2_0
.text:080484DB                 mov     [esp], eax      ; stream
.text:080484DE                 call    _fflush
.text:080484E3                 mov     eax, 0FFFFFFFFh
.text:080484E8                 jmp     short loc_80484FF
.text:080484EA ;   ---------------------------------------------------------------------------
.text:080484EA
.text:080484EA loc_80484EA:                            ; CODE XREF: main+30j
.text:080484EA                 mov     eax, [ebp+argv]
.text:080484ED                 add     eax, 4
.text:080484F0                 mov     eax, [eax]
.text:080484F2                 mov     [esp], eax      ; src
.text:080484F5                 call    foo
.text:080484FA                 mov     eax, 0
.text:080484FF
.text:080484FF loc_80484FF:                            ; CODE XREF: main+51j
.text:080484FF                 add     esp, 8
.text:08048502                 pop     edi
.text:08048503                 pop     ebp
.text:08048504                 retn
.text:08048504 main            endp
```

##### `foo()` 函数

```asm
.text:08048464 ; int __cdecl foo(char *src)
.text:08048464                 public foo
.text:08048464 foo             proc near               ; CODE XREF: main+5Ep
.text:08048464
.text:08048464 src             = dword ptr  8
.text:08048464
.text:08048464                 push    ebp
.text:08048465                 mov     ebp, esp
.text:08048467                 sub     esp, 4
.text:0804846A                 mov     eax, [ebp+src]
.text:0804846D                 mov     [esp], eax      ; src
.text:08048470                 call    bar
.text:08048475                 leave
.text:08048476                 retn
.text:08048476 foo             endp
```

##### `bar()` 函数

```asm
.text:08048477 ; int __cdecl bar(char *src)
.text:08048477                 public bar
.text:08048477 bar             proc near               ; CODE XREF: foo+Cp
.text:08048477
.text:08048477 dest            = byte ptr -100h
.text:08048477 src             = dword ptr  8
.text:08048477
.text:08048477                 push    ebp
.text:08048478                 mov     ebp, esp
.text:0804847A                 sub     esp, 108h
.text:08048480                 mov     eax, [ebp+src]
.text:08048483                 mov     [esp+4], eax    ; src
.text:08048487                 lea     eax, [ebp+dest]
.text:0804848D                 mov     [esp], eax      ; dest
.text:08048490                 call    _strcpy
.text:08048495                 leave
.text:08048496                 retn
.text:08048496 bar             endp
```

注意：进行 `call` 调用时，会把 Return Address 地址压入到堆栈中，然后才 `push ebp`。堆栈布局如下图 1-1 所示：

![](https://bbs.kanxue.com/upload/attach/201707/591834_lnt294uk3bzklsa.png)

图 1-1：函数栈布局图

注意：上述图中的地址是动态运行时确定的，如何知道 `foo` 的 `EBP` 的地址和 `ESP` 的地址呢？这里需要借助 `gdb`，运行命令如下所示，`l` 是显示源代码跟行号，`r` 是运行（可加参数），`break` 是增加断点，以便查看 `ebp` 和 `esp` 的寄存器值。本机上执行 `gdb` 如下所示：

![](https://bbs.kanxue.com/upload/attach/201707/591834_q9egjqimkn9z0ub.png)

![](https://bbs.kanxue.com/upload/attach/201707/591834_da9cy1nk7467yii.png)

从图 1-1 函数栈布局中，我们可以确认：该程序存在 off by one 漏洞。当我们输入 `256` 个字节之后，空字节可以覆盖 `foo` 的 `EBP` 的 `LSB`。当 `foo` 的存储在目标缓冲区 `buf` 之上的 `EBP` 被一个 `NULL` 字节覆盖时，`ebp` 从 `0xbffff588` 变成 `0xbffff500`。从图 1-1 中我们可以看到 `0xbffff500` 是目标缓冲区 `buf` 的一部分，攻击者可以控制 `0xbffff500`，从而控制指令指针 `EIP`，达到任意代码执行。可能会有人存在疑问：攻击者如何控制 `EIP` 寄存器？

**如何控制 `EIP` 寄存器：**

`bar()` 函数中的汇编代码中，存在两行标红的代码：`leave` 和 `retn` 指令。`foo()` 也存在。
