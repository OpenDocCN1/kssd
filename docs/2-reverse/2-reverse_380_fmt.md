
# `ida` 中隐藏 `nop`

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-353.htm`](https://www.pediy.com/kssd/pediy09/pediy09-353.htm)

> 在用 `IDA` 时，有些壳的花多的受不了，去花后很多的 `nop`

> 看起来很不爽，就写了个小脚本

> 不知各位老大有什么好方法 好工具 推荐一下

```c
#include <idc.idc>

static main() {
  auto ptr, i;
  ptr = 0x401000;
  while ( ptr = FindBinary(ptr, SEARCH_DOWN | SEARCH_NEXT, "CC") ) {
    for ( i = 0; i < 16; ++i ) {
      if ( Byte(ptr + i) != 0xCC
        && ((buf + i) & 15) == 0) {
        HideArea(buf, buf + i, "", "", "", -1);
        buf = buf + i;
        break;
      }
    }
  }
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
