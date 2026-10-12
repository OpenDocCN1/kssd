
## 三、总结

代码:

```
1、用户名长度不少于 2 位，注册码不少于 8 位；

2、注册码一种是验证前 8 位“52697629”，只对前 8 位验证；
第 2 种是验证注册码前 5 位，由用户名计算而来。详细可以看注释部分。

c 语言注册机代码，计算了前面 5 位，后面任意数字或字母。代码为方便固定了 3 位。

#include "stdio.h"
#include "string.h"
int sum(char name[], int n)
{
    int i, sum = 0;
    for (i = 0; i < n; i++)
        sum = sum + name[i];
    return sum;
}

void main()
{
    char name[25] = {'\0'};
    char regcode[25] = {'\0'};
    scanf("%s", name);
    if (strlen(name) >= 2)
    {
        regcode[0] = (name[0] | 0x54) % 10 + 0x30;
        regcode[1] = (name[1] | 0x49) % 10 + 0x30;
        regcode[2] = (name[0] | 0x46) % 10 + 0x30;
        regcode[3] = (name[1] | 0x46) % 10 + 0x30;
        regcode[4] = sum(name, strlen(name)) % 10 + 0x30;
        regcode[5] = regcode[6] = regcode[7] = 0x32;
        printf("%s", regcode);
    }
    else
        printf("用户名不能少于 2 位");
}
```
