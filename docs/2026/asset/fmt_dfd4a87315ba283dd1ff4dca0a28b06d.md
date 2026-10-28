
```
#include <sys/types.h>

#include <sys/socket.h>

#include <netinet/in.h>

#include <stdio.h>

#include <errno.h>

 

#define VerifyErr(a, b) \

        if (a) { fprintf(stderr, "%s failed.\n", (b)); return 0; } \

        else fprintf(stderr, "%s success.\n", (b));

 

int main(void)

{

        int nSock, nSock1;

        char buf[2048];

 

        //CreateSock(&nSock, 9001, 9);

        nSock = nSock1 = 0;               // 这里只是为了调试所用

        VerifyErr(CreateSock(&nSock, 9001, 9) != 0, "Create Listen SOCKET");

        //VerifyErr(AcceptSock(&nSock1, nSock) != 0, "Link");

        AcceptSock(&nSock1, nSock);

        memset(buf, 0, sizeof(buf));

        recv(nSock1, buf, sizeof(buf), 0);

        fprintf(stderr, buf);

        close(nSock1);

        close(nSock);

 

        return 0;

}
```

运行程序, 并在在浏览器中输入`6b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0V1H3x3o6q4Q4x3V1k6Q4x3V1x3`。你将得到一条来自客户端的http报文. 但是在这里却出现了问题, 问题如下:

```
[bill@billstone Unix_study]$ make tcp1

cc     tcp1.c   -o tcp1

[bill@billstone Unix_study]$ ./tcp1

Create Listen SOCKET success.

Error received! No: 9                         // 出现错误, 为'bad file number'错误

[bill@billstone Unix_study]$
```

请大家帮忙查找一下原因！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
