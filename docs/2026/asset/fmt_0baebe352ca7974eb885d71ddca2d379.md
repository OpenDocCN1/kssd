
\* Reference To: `KERNEL32.lstrcmpA`, Ord:0000h

                                  |

`:0040114E E8339C0000`              Call `0040AD86`

`:00401153 85C0`                    test eax, eax

`:00401155 750D`                    jne `00401164`  ;关键跳转哟

--------------------------------------------------------------------------------

哈哈，`lstrcmp`，什么意思不用我说了吧，当然是STRingCoMPare字符串比较啦。把计算的结果与前面输入的假码比较，相等就OK了。好了，现在把上面的完整的代码过一遍，怎么样，写个注册机不难吧？

代码:
--------------------------------------------------------------------------------

```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

void main()
{
  int EAX=0,EBX=0,len;
  char name[50]={0};
  char password[50]={0};
  printf("Please input your name:");
  scanf("%s",name);
  len=strlen(name);
  for (int i=0;i<len;i++)
  {
    EAX=name[i]^(i+1);
    EBX+=EAX;
  }
  EAX*=6;
  EBX<<=7;
  EAX+=EBX;
  printf("Your password is: %lX\n",EAX);
  printf("KeyGen by RoBa  Enjoy Cracking,Newbies!\n");
}
```

--------------------------------------------------------------------------------

简单的C程序哟，你也可以用你熟悉的语言写一个。

后记：

这是一个简单的CRACKME，本来三言两语就能解决的，被我啰啰嗦嗦说了这么一大堆，就是希望能把问题真正地说明白了，希望通过这篇文章让你发现，破解软件乃至写注册机并不是多么困难的事，只要坚持下去，谁都有成为高手的那一天，也希望高手们能够抽时间发一些适合较低水平的文章或者至少对发这种文章的新手多多鼓励[609K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0H3k6h3&6U0M7X3q4U0K9$3#2W2M7#2)9J5k6h3y4J5j5h3y4C8L8h3g2K6i4K6u0W2k6r3g2Q4x3V1k6G2M7r3g2F1j5%4u0S2j5$3E0E0k6i4y4Q4x3V1k6o6L8$3I4D9k6h3y4@1K9h3!0F1M7#2)9J5c8X3E0W2P5h3N6W2L8X3&6A6L8X3M7@1L8X3g2%4j5X3W2W2M7#2)9J5c8V1y4J5j5h3y4C8L8h3g2K6i4K6u0r3K9K6c8F1i4K6u0W2P5X3W2H3](https://bbs.kanxue.com/elink@f88K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0H3k6h3&6U0M7X3q4U0K9$3#2W2M7#2)9J5k6h3y4J5j5h3y4C8L8h3g2K6i4K6u0W2k6r3g2Q4x3V1k6G2M7r3g2F1j5%4u0S2j5$3E0E0k6i4y4Q4x3V1k6o6L8$3I4D9k6h3y4@1K9h3!0F1M7#2)9J5c8X3E0W2P5h3N6W2L8X3&6A6L8X3M7@1L8X3g2%4j5X3W2W2M7#2)9J5c8V1y4J5j5h3y4C8L8h3g2K6i4K6u0r3K9K6c8F1i4K6u0W2P5X3W2H3)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
