
``` 
00415609  |.  F7D1          |not ecx

0041560B  |.  49            |dec ecx                            ;记数器-1

0041560C  |.  3BF1          |cmp esi,ecx                        ;循环完了吗？

0041560E  |.^ 72 C3         \jb short MultiDic.004155D3

00415610  |.  5D            pop ebp                             ;这里可以做内存注册机

00415611  |>  5E            pop esi

00415612  |.  5F            pop edi

00415613  |.  B0 01         mov al,1

00415615  |.  5B            pop ebx

00415616  |.  81C4 14020000 add esp,214

0041561C  \.  C2 0800       retn 8
```

好了，基本上算法大概就在上面的

下面用高级语言描述一下算法（顺便复习一下C ^\_^），注册机如下：

``` 
#include <stdio.h>

void main()

{

/\* KeyGen By XJTL   3c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6f1y4F1h3p5S2S2j5$3E0W2M7W2)9J5k6h3y4G2L8b7`.`. \*/

  int j,k,t=0,l=0,s[20];

  char n[20];

  printf("\*\*\*\*\*\*\*\* KeyGen For KCDICT2004 \*\*\*\*\*\*\*\*\n");

  printf("Please Input Your Serial Num:");

  scanf("%s",n);

  while(n[l]!='\0') l++;

  printf("\n Your License Num is : ");

  for(k=0;k<l;k+=2)

  {

    s[t]=n[k+1]^n[k];

    s[t]%=0x1A;

    s[t]+=0x41;

    printf("%c",s[t]);

    t++;

  }

    printf("\n");

}
```

―――――――――――――――――――――――――――――――――――――――――――

### 最后总结

这里介绍了如何找到算法核心，方便我等菜鸟们步入算法分析时代。希望各位小鸟们加油哟~

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
