
# [原创]随机字符串生成函数

写着玩的，见笑![](https://bbs.kanxue.com/view/img/face/16.gif)

```
#include "stdafx.h"
#include <stdio.h>
#include "windows.h"
#pragma comment(lib,"winmm.lib")

int main(int argc, char *argv[])
{
        int RanNum;
        int n1,n2;
        int count;
        char *buffer;
        char ch;

        //产生种子
        srand((unsigned(timeGetTime())));
        RanNum=rand();
        n1=RanNum%10;
        printf("随机数字%d\n",n1);
        n1++;

        buffer=(char *)VirtualAlloc(NULL,2*n1,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
        memset((void *)buffer,0,2*n1);

        for (count=0;count<(2*n1-1);count++)
        {
                RanNum=rand();
                n2=RanNum%52;
                printf("随机数字%d\n",n2);

                if (n2>=26)
                {
                        ch=n2&0x0F;
                        ch+=0x47;  //'a'------'z'
                }
                else
                {
                        ch=n2&0x0F;
                        ch+=0x41;  //'A'-----'Z'
                }
                buffer[count]=ch;
        }

        printf("生成随机字符串:%s\n",buffer);
        VirtualFree(buffer,2*n1,MEM_DECOMMIT);
        return 0;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
