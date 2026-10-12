
# CMailServer 3.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1173.htm`](https://www.pediy.com/kssd/pediy03/forum1173.htm)

`CMailServer` Ver `3.2`

主要功能

1. 支持通用邮件客户端软件 `Outlook Express`、`Foxmail` 等收发邮件。
2. 支持 Web 页面收发邮件。
3. 支持 Internet 收发邮件。
4. 支持用户通过 Web 浏览器申请邮箱、修改邮箱密码、修改用户信息等资料。
5. 支持管理员新建、删除、禁用用户邮箱和设置用户邮箱大小。
6. 支持管理员同时向多个用户发送邮件。
7. 支持管理员生成 `HTML` 和 `INI` 格式的用户邮箱信息报表。
8. 支持作为 `NT` 服务运行。

未注册限制：

1. 只支持 10 个用户
2. 能享受免费升级服务。
3. 能获得免费的技术支持服务。

下载地址： [`http://microserver.top263.net/cmailserver/cmailserver32cr2.exe`](http://microserver.top263.net/cmailserver/cmailserver32cr2.exe)

注册机：

```c
#include "stdio.h"

/* Key Generator for CMailServer Ver 3.2 */
/* tcc -mt -lt crk-ccp */

main()
{
    char *str1="microserver1";
    char *str2="cmailserver3";
    char setup[13];
    char rn[13];
    char sn[13];
    int i=0;
    int temp;

L1:
    puts("\n --= CMailServer Ver 3.2 注册机 Programmed by 西风瘦马=--\r\n 请输入您的特征码(共 12 位)：");
    gets(setup);
    if(strlen(setup)-12)goto L1;

L2:
    puts("请输入您的序列号(共 12 位 16 进制数字)：");
    gets(rn);
    if(strlen(rn)-12)goto L2;

    for(i=0;i<12;i++)
    {
        temp=(str1[i]+str2[i]+setup[i]+rn[i])%0x3e;
        sn[i]=temp>=0x1a?(temp>=0x34?temp-4:temp+0x27):(temp+0x61);
    }
    sn[i]=0;
    printf("\n 您的注册码是: %s\n 多谢使用本程序！\n",sn);
    getch();
}
```

北京遥志软件注册算法都是一样的，只是参与运算的字符串不一样而已。哈哈哈……

`4dd466d0e1e0cc123afbe504da2462ca`
