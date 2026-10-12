
# 不用循环和 if 判断解决倒序问题(ccnyou)

> 原文：[`www.pediy.com/kssd/pediy12/143908.html`](https://www.pediy.com/kssd/pediy12/143908.html)

看了 `search` 同学的《不用循环和 if 判断解决倒序问题》 `http://bbs.pediy.com/showthread.php?t=143899`，忍不住也来几行代码

（也是递归实现）：

代码:

```c
#include <stdio.h>

int StrrevPrint(char *pString)
{
  *pString&&StrrevPrint(pString+1);
  *pString&&printf("%c",*pString);
  return 0;
}

int main(int argc, char* argv[])
{
  char szBuf[1024];
  scanf("%s",szBuf);
  StrrevPrint(szBuf);
  return 0;
}
```
