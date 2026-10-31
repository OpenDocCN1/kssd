
# [原创]追赶高校生之 - 2016腾讯的游戏安全竞赛题PC第一题

逆了一遍，要达到腾讯要求的高校生水准还是不容易。。

```c
#include <stdio.h>

#include <windows.h>

char sample_str[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789@%";

int Get_Index_in_sample(UCHAR ch)

{

    char *p = (char*)memchr(sample_str, ch, sizeof(sample_str)-1);

    if (p)

        return (int)(p - sample_str);

    else

        return -1;

}

int UserName(PLONG UserNameArray)

{

    char chUserName[] = "zxxttrrun";

    ULONG UserNameLen = (ULONG)strlen(chUserName);

    char tmp[25] = {0};

    ULONG *v7;

    ULONG i;

    if ( (unsigned int)(UserNameLen - 6) > 0xE )

        return -1;

    for (i = 0; i < 16; i++)

    {

        v7 = (ULONG*)&tmp[i];

        *v7 += UserNameLen * (i + 20160126) * chUserName[i % UserNameLen];

    }

    for ( i = 0; i < 20; i += 4 )

    {

        UserNameArray[i/4] = (*(LONG *)&tmp[i]) / 10;

    }

    return 0;

}

int Password(PULONG PasswordArray)

{

    char PasswordBuf[100] = "JVeFMIdiy5HLoQcSzMzMzMzMzMz";

    ULONG PasswordLen = 27;

    ULONG tmpDWORD;

    PUCHAR p = (PUCHAR)&tmpDWORD;

    ULONG i;

    ULONG j;

    UCHAR tmp[25] = {0};

    PUCHAR p1 = tmp;

    for (i = 0, j = 0; i < PasswordLen; i++)

    {

        char ch = PasswordBuf[i];

        if ( !isalnum(ch) && ch != '@' && ch != '%' )

            break;

        p[i%4] = ch;

        if(j++ == 3)

        {

            for ( j = 0; j < 4; j++ )

            {

                p[j] = Get_Index_in_sample(p[j]);

            }

            *p1++ = (p[0]<<2) + ((p[1] >> 4) & 3);

            *p1++ = (p[1] << 4) ^ ((p[2] >> 2) & 0xF);

            *p1++ = (p[2] << 6) + p[3];

            j = 0;

        }

    }

    if (j)

    {

        memset((char *)&tmpDWORD + j, 0, 4 - j);

        for ( j = 0; j < 4; j++ )

        {

            p[j] = Get_Index_in_sample(p[j]);

        }

        *p1++ = (p[0]<<2) + ((p[1] >> 4) & 3);

        *p1++ = (p[1] << 4) ^ ((p[2] >> 2) & 0xF);

        *p1++ = (p[2] << 6) + p[3];

    }

    for ( i = 0; i < 20; i += 4 )

    {

        PasswordArray[i/4] = *(ULONG *)&tmp[i];

    }

    return 0;

}

int main()

{

    LONG U[5];  // 逆向过程是算数右移，所以必须是有符号数

    ULONG P[5];

    UserName(U);

    Password(P);

    if ( P[4] + U[0] == P[2]     &&

         P[2] + U[1] == P[4] * 2 &&

         P[3] + U[2] == P[0]     &&

         P[0] + U[3] == P[3] * 2 &&

         P[1] + U[4] == U[2] * 3)

    {

        printf("注册成功~\n");

    }

    else

        printf("注册失败!\n");

    return 0;

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

**上传的附件：**

- [`Tencent2016A.7z`](javascript:void(0);)（654.88kb，41次下载）
