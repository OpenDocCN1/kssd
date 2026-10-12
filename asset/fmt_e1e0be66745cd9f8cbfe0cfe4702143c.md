
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void main(void)
{
    char         UserName[128];
    char         UserString[8];
    char         a[8];
    int          index, len;
    char         k, j;
    signed long  temp, guess;
    char         SuccessCounter;

    do
    {
        printf("Input your name(at least 5 chars):");
        gets(UserName);
        len = strlen(UserName);
    } while(len < 5);

    strupr(UserName);
    index = 0;
    for(k = 0; k < 8; k++)
    {
        a[k]  = UserString[k] = UserName[index++];
        index %= len;
    }

    //下面用穷举求出变换前的 8 个字节
    SuccessCounter = 0;
    for(k = 0; k < 8; k++)
    {
        for(guess = 0; guess <= 255; guess++)
        {
            temp = 1L;
            for(j = 7; j > 0; j--)
            {
                temp *= guess;
                if (temp > 0x00008899L)
                {
                    temp %= 0x00008899L;
                }
            }
            if ((temp % 0x000000BBL) == a[k])
            {
                SuccessCounter++;
                a[k] = guess;
                break;
            }
        }
    }

    //判断 8 个字节是否全部穷举成功
    if (SuccessCounter != 8)
    {
        printf("Guess failed.\n");
        exit(-1);
    }

    for(k = 0; k < 8; k++)
    {
        a[k] ^= (k + 0x50);
    }

    printf("Your registration key is: ");
    for(k = 0; k < 4; k++)
    {
        printf("%02X", a[k] & 0xFF);
    }

    printf("-");
    for(k = 4; k < 8; k++)
    {
        printf("%02X", a[k] & 0xFF);
    }

    printf("\n");
}
```

`blowfish` `4dd466d0e1e0cc123afbe504da2462ca`
