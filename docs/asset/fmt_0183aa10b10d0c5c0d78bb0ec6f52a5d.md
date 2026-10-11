
# 无聊看到的顺手写了个字符串转数字的代码

> 原文：[`www.pediy.com/kssd/pediy12/142948.html`](https://www.pediy.com/kssd/pediy12/142948.html)

[`bbs.pediy.com/showthread.php?t=128615`](http://bbs.pediy.com/showthread.php?t=128615)

这位面试的兄弟，哈哈，估计题目没看仔细吧？ 我就直接 show 代码了，不考虑意外和溢出啥的

TX 招我吧 哈哈~~~

代码:

```
int _hex2int(char* lpStr)
{
  unsigned int ret = 0;
  
  do {
    unsigned char c = *lpStr;
    ret = ret << 4;
    if (c >= 'a' && c <= 'f'){
      ret += 10 + c - 'a';
    } else if (c >= 'A' && c <= 'F'){
      ret += 10 + c - 'A';
    } else if (c >= '0' && c <= '9'){
      ret += c - '0';
    } else {
      printf("error _hex2int\n");
      break;
    }     
  } while(*++lpStr);

  printf("\n%x %d\n", ret, ret);

  return ret;
}

void _int2str(unsigned int i)
{
  char fmt[16] = {0};
  int bit_pos = 0;
  do {
    fmt[14 - bit_pos] = '0' + (i % 10);
    bit_pos++;
    i = i/10;
  } while(i);
  
  printf("%s\n", fmt + 15 - bit_pos);
}

int main(int argc, char** argv)
{
  char test[] = "1a12fa";
  _int2str(_hex2int(test));
}
```
