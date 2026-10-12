
# 【代码之美】试题 1

> 原文：[`www.pediy.com/kssd/pediy10/76542.html`](https://www.pediy.com/kssd/pediy10/76542.html)

输入文件：`in.txt`

输出文件：`out.txt`

编译环境：`ddk 2003`

代码:

```
#include <stdio.h>
#include <stdlib.h>

//
// The max number length
//
#define NUM\_MAX\_LEN 12

//
// Hold the result.
//
int result\_array\[10\] = {0};

void count(unsigned int i)
{
    unsigned int j;
    unsigned int tmp;

    memset(result\_array, 0, sizeof(result\_array));

    if (i > 100000000)
    {
        return;
    }

    result\_array\[0\] = 1;

    for (j=1; j<=i; j++)
    {
        for (tmp=j; tmp>0;)
        {
            if (tmp != 0)
            {
                result\_array\[tmp%10\]++;

                tmp /= 10;
            }
            else
            {
                continue;
            }
        }
    }
}

int \_\_cdecl main(void)
{
    unsigned int line\_num = 0;
    int i;
    FILE \*input\_file;
    FILE \*output\_file;
    char read\_buffer\[NUM\_MAX\_LEN\];

    input\_file = fopen("in.txt", "r");

    output\_file = fopen("out.txt", "w+");

    fgets(read\_buffer, NUM\_MAX\_LEN, input\_file);

    line\_num = atoi(read\_buffer);

    while (fgets(read\_buffer, NUM\_MAX\_LEN, input\_file))
    {
        count(atoi(read\_buffer));

        for (i=0; i<9; i++)
        {
            fprintf(output\_file, "%d, ", result\_array\[i\]);
        }

        fprintf(output\_file, "%d\\n", result\_array\[9\]);
    }
}
```
