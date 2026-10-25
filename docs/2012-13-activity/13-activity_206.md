
# 利用 IDA+Hex-Rays 快速分析 ExploitMeA

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-367.htm`](https://www.pediy.com/kssd/pediy09/pediy09-367.htm)

> 很少见有人写关于 Hex-Rays 的文章，这几天摸索了一下，发现这东西相当不错，现在写篇关于 Hex-Rays 的文章，因为 Hex-Rays 是个新玩意（对我来说），有错误的地方请大家见谅。
>
> 废话结束………………
>
> 打开程序------>`exploitmeA` 发现是开 `7777` 端口，`NC` 链接下看看。打入什么返回什么。
>
> 感觉很像《网络渗透技术》那本书的 W32 溢出例程，好，打开 `IDA+Hex-Rays` 分析下。

得到

```c
int __cdecl main(int argc, const char **argv, const char *envp)
{
  SOCKET v3; // eax@1
  SOCKET v4; // ebp@1
  SOCKET v5; // ebx@6
  int v6; // esi@7
  SOCKET v8; // eax@6
  int v9; // eax@7
  SOCKET v10; // eax@10
  struct WSAData WSAData; // [sp+228h] [bp-190h]@1
  struct sockaddr name; // [sp+4h] [bp-3B4h]@4
  struct sockaddr addr; // [sp+14h] [bp-3A4h]@6
  int v14; // [sp+28h] [bp-390h]@7

  WSAStartup(0x101u, &WSAData);
  v3 = socket(2, 1, 0);
  v4 = v3;
  if ( (signed int)v3 < 0 )
  {
    ostream__operator___(v3);
    ostream__operator___("socket creating error!");
    ostream__operator___(10);
    sub_4012B0(sub_4012D0);
    exit(1);
  }
  *(_WORD *)&name.sa_data[2] = 2;
  *(_WORD *)&name.sa_data[4] = htons(0x1E61u);
  *(_DWORD *)&name.sa_data[6] = htonl(0);
  if ( bind(v4, (const struct sockaddr *)&name.sa_data[2], 16) )
  {
    ostream__operator___("binding stream socket error!");
    ostream__operator___(10);
    sub_4012B0(sub_4012D0);
  }
  ostream__operator___("************************************************");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  ostream__operator___("     exploit target server 1.0\t   ");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  ostream__operator___("************************************************");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  listen(v4, 4);
  *(_DWORD *)&name.sa_family = 16;
  v8 = accept(v4, (struct sockaddr *)&addr.sa_data[2], (int *)&name);
  v5 = v8;
  if ( v8 != -1 )
  {
    while ( 1 )
    {
      memset(&v14, 0, 0x200u);
      v9 = recv(v5, (char *)&v14, 512, 0);
      v6 = v9;
      if ( v9 < 0 )
      {
        ostream__operator___("reading stream message erro!");
        ostream__operator___(10);
        sub_4012B0(sub_4012D0);
        v6 = 0;
      }
      sub_401000(&v14);
      if ( !v6 )
      {
        closesocket(v5);
        v10 = accept(v4, (struct sockaddr *)&addr.sa_data[2], (int *)&name);
        v5 = v10;
        if ( v10 == -1 )
          break;
      }
    }
  }
  ostream__operator___("accept error!");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  return WSACleanup();
}
```

注意这里

```c
memset(&v14, 0, 0x200u);
v9 = recv(v5, (char *)&v14, 512, 0);
v6 = v9;
if ( v9 < 0 )
{
  ostream__operator___("reading stream message erro!");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  v6 = 0;
}
sub_401000(&v14);
```

这个 `v14` 定义了 `200h` 的 unsigned int

`sub_401000` 函数代码：

```c
int __cdecl sub_401000(int a1)
{
  char v2; // [sp+8h] [bp-C8h]@1

  strcpy(&v2, (const char *)a1);
  ostream__operator___("********************************");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  ostream__operator___("received:");
  ostream__operator___(10);
  sub_4012B0(sub_4012D0);
  ostream__operator___(&v2);
  ostream__operator___(10);
  return sub_4012B0(sub_4012D0);
}
```

定义出来 `a1` 后这个函数很关键 `strcpy`！！！！！！！！！！

感觉溢出就在这里了

开 `200` 个 `FUZZ` 应该再加 `JUMP+SHELLCODE` 就可以了

这是个 `FUZZ` 的小程序，不是我写的，抄到这里

```c
#include <stdio.h>
    int main(int argc,char **argv)
    {
        char *buff;
        int i,Len;

        if(argc < 3)
        {
            printf("Magic Char By CoolDiyer\n");
            printf("Usage:\n\t mc -c <count> [-v]\n");
            return -1;
        }

        Len=atoi(argv[2]);
        buff=(char *)malloc(Len+1);
        buff[Len]=0;

        for(i=0;i<Len;i++)
        {
            switch(i%4)
            {
                case 0: buff[i] = 'A'+i/(26*26*26)%26; break;
                case 1: buff[i] = 'A'+i/(26*26)%26; break;
                case 2: buff[i] = 'A'+i/26%26; break;
                case 3: buff[i] = 'A'+i%26; break;
            }
        }

        if(argc == 4)
        {
            for(i=1; i< Len+1; i++)
            {
                printf("%c",buff[i-1]);
                if(i%4 == 0)
                {
                    printf("\t%d\t0x%.2x",i,i);
                    printf("\tEIP ==> 0x%.2x%.2x%.2x%.2x\n",buff[i-1],buff[i-2],buff[i-3],buff[i-4]);
                }
            }
        } else {
            printf("%s\n",buff);
        }
        return 0;
    }
```

出一个 `FUZZ`--->

```
D:\>mc
Magic Char By CoolDiyer
Usage:
         mc -c <count> [-v]

D:\>mc -c 500
AAADAAAHAAALAAAPAAATAAAXAABBAABFAABJAABNAABRAABVAABZAACDAACHAACLAACPAACTAACXAADBAADFAADJAADNAADRAADVAADZAAEDAAEHAAELAAEPAAETAAEXAAFBAAFFAAFJAAFNAAFRAAFVAAFZAAGDAAGHAAGLAAGPAAGTAAGXAAHBAAHFAAHJAAHNAAHRAAHVAAHZAAIDAAIHAAILAAIPAAITAAIXAAJBAAJF AAJJAAJNAAJRAAJVAAJZAAKDAAKHAAKLAAKPAAKTAAKXAALBAALFAALJAALNAALRAALVAALZAAMDAAMHAAMLAAMPAAMTAAMXAANBAANFAANJAANNAANRAANVAANZAAODAAOHAAOLAAOPAAOTAAOXAAPBAAPFAAPJAAPNAAPRAAPVAAPZAAQDAAQHAAQLAAQPAAQTAAQXAARBAARFAARJAARNAARRAARVAARZAASDAASHAASLAASPAASTAASXAATBAATF
```

用 `ollydbg` 加一下 `exploitmea`，用 `NC` 把它发出去

程序崩掉了，哈哈，显示程序不知道如何读取地址 `56484141`

好

```
D:\>mc -c 500 -v
AAAD    4       0x04    EIP ==> 0x44414141
AAAH    8       0x08    EIP ==> 0x48414141
...
AAET    124     0x7c    EIP ==> 0x54454141
AAEX    128     0x80    EIP ==> 0x58454141
AAFB    132     0x84    EIP ==> 0x42464141
AAFF    136     0x88    EIP ==> 0x46464141
AAFJ    140     0x8c    EIP ==> 0x4a464141
AAFN    144     0x90    EIP ==> 0x4e464141
AAFR    148     0x94    EIP ==> 0x52464141
AAFV    152     0x98    EIP ==> 0x56464141
AAHZ    156     0x9c    EIP ==> 0x5a464141
AAGD    160     0xa0    EIP ==> 0x44474141
AAGH    164     0xa4    EIP ==> 0x48474141
AAGL    168     0xa8    EIP ==> 0x4c474141
AAGP    172     0xac    EIP ==> 0x50474141
AAGT    176     0xb0    EIP ==> 0x54474141
AAGX    180     0xb4    EIP ==> 0x58474141
AAHB    184     0xb8    EIP ==> 0x42484141
AAHF    188     0xbc    EIP ==> 0x46484141
AAHJ    192     0xc0    EIP ==> 0x4a484141
AAHN    196     0xc4    EIP ==> 0x4e484141
AAHR    200     0xc8    EIP ==> 0x52484141
AAHV    204     0xcc    EIP ==> 0x56484141   <========这里。
AAHZ    208     0xd0    EIP ==> 0x5a484141
AAID    212     0xd4    EIP ==> 0x44494141
...
```

`AAHV    204     0xcc    EIP ==> 0x56484141`

注意到 `204` 了吧，这就是长度了，多出来的 `4` 正式 `AAHV` 也就是 `JMP` 的位置。

剩下的分析步骤别的贴也说的很清楚了，我就不嗦了。

顺便把《网络渗透技术》那本书的 W32 溢出例程发出来，例程是原码，和 `exploitmea` 原理一模一样。

```c
/* server.cpp
*
*  《网络渗透技术》演示程序
*  作者：san, alert7, eyas, watercloud
*
*  存在缓冲区溢出的服务端程序
*/
```



```c
#include <winsock2.h>

#include <stdio.h>

#pragma comment(lib,"ws2_32")

char Buff[2048];
void overflow(char * s,int size)
{
    char s1[50];
    printf("receive %d bytes",size);
    s[size]=0;
    strcpy(s1,s);
}

int main()
{
    WSADATA wsa;
    SOCKET listenFD;
    int ret;
    char asd[2048];

    WSAStartup(MAKEWORD(2,2),&wsa);

    listenFD = WSASocket(2,1,0,0,0,0);

    struct sockaddr_in server;

    server.sin_family = AF_INET;
    server.sin_port = htons(8888);
    server.sin_addr.s_addr = INADDR_ANY;
    ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
    ret=listen(listenFD,2);

    int iAddrSize = sizeof(server);
    SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize);
    unsigned long lBytesRead;
    while(1)    {
            lBytesRead=recv(clientFD,Buff,2048,0);
            if(lBytesRead<=0)    break;

            printf("\nfd = %x\n", clientFD);

            overflow(Buff,lBytesRead);

            ret=send(clientFD,Buff,lBytesRead,0);
            if(ret<=0)    break;

    }
    WSACleanup();
    return 0;
}
```

/* server.cpp

  《网络渗透技术》演示程序
  作者：san, alert7, eyas, watercloud

  存在缓冲区溢出的服务端程序
*/

```c
#include <winsock2.h>
#include <stdio.h>

#pragma comment(lib,"ws2_32")

char Buff[2048];
void overflow(char * s,int size)
{
    char s1[50];
    printf("receive %d bytes",size);
    s[size]=0;
    strcpy(s1,s);
}

int main()
{
    WSADATA wsa;
    SOCKET listenFD;
    int ret;
    char asd[2048];

    WSAStartup(MAKEWORD(2,2),&wsa);

    listenFD = WSASocket(2,1,0,0,0,0);

    struct sockaddr_in server;

    server.sin_family = AF_INET;
    server.sin_port = htons(8888);
    server.sin_addr.s_addr = INADDR_ANY;
    ret=bind(listenFD,(sockaddr *)&server,sizeof(server));
    ret=listen(listenFD,2);

    int iAddrSize = sizeof(server);
    SOCKET clientFD=accept(listenFD,(sockaddr *)&server,&iAddrSize);
    unsigned long lBytesRead;
    while(1)    {
            lBytesRead=recv(clientFD,Buff,2048,0);
            if(lBytesRead<=0)    break;

            printf("\nfd = %x\n", clientFD);

            overflow(Buff,lBytesRead);

            ret=send(clientFD,Buff,lBytesRead,0);
            if(ret<=0)    break;

    }
    WSACleanup();
    return 0;
}
```

4dd466d0e1e0cc123afbe504da2462ca
