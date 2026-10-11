
# 争分夺秒背单词 V1.5.01.12.11

> 原文：[`www.pediy.com/kssd/pediy04/kanxue305.htm`](https://www.pediy.com/kssd/pediy04/kanxue305.htm)

```c
#include "stdafx.h"
#include "stdio.h"
#include "string.h"
#include "conio.h"

void main()
{
    char name[80];
    char sn[80];
    int namelen, snlen, i = 0;
    long febx = 1;

    puts("\n 用户名:");
    scanf("%s", name);
    namelen = strlen(name);

    puts("\n 机器码:");
    scanf("%s", sn);
    snlen = strlen(sn);

    namelen = namelen + snlen;

    puts("\n 注册码:");
    strcat(name, sn);

    for(i = 0; i < namelen; i++) febx = (febx * 0x22d6) + name[i];

    _asm {
        mov eax, febx
        cdq
        xor eax, edx
        sub eax, edx
        mov febx, eax
    }

    printf("%d", febx);
    getch();
}
```

```asm
_asm {
    mov eax, febx
    cdq
    xor eax, edx
    sub eax, edx
    mov febx, eax
}
```

分析：
若 `febx >= 0`（最高位是 `0`）：
`cdq` 后，`edx==0`。`eax xor 0 == eax`，`eax - 0 == eax`。

否则若 `febx < 0`（最高位是 `1`）：
`cdq` 后，`edx==0xffffffff`。
(1) `eax xor 0xffffffff == eax 取反 == 0x100000000 - eax - 1`。
(2) `sub eax, edx == sub eax, 0xffffffff == sub eax, -1 == add eax, 1`。
由 (1) 和 (2) 得 `eax = (0x100000000 - eax - 1) + 1 = 0x10000000 - eax`，在 `32` 位寄存器中即 `eax = -eax`。

因此上述过程是将 `eax` 取绝对值。用 `abs(febx)`［忘了是不是这样的了：)］可搞定。或者用一句：`(febx >= 0 ? febx : -febx)`。
`4dd466d0e1e0cc123afbe504da2462ca`
