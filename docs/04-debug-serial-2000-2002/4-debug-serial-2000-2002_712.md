
# 文件管理器

> 原文：[`www.pediy.com/kssd/pediy03/forum759.htm`](https://www.pediy.com/kssd/pediy03/forum759.htm)

刚下载了文件管理器 `http://www.soft999.com/download1/flashsoft.zip`

**软件功能**

- 将硬盘上的软件分类整理，可以存储软件的大小、下载地点、软件介绍、备注等详细信息。
- 支持按软件名的模糊查询功能。
- 将软件相关信息分类输出成 `HTML` 文件。
- 支持导入一个目录下的文件。
- 支持拖拽、可以从子资源浏览器中拖拽文件。
- 多种语言支持(中文、英文)。
- 流行的 `Skins` 技术。

这个软件破解过程挺简单的，过程就免了，初学者也能轻松搞定。

给一个有较的注册码：

- 用户名：已注册
- 注册码：`776369580`

不知道作者会不会找我麻烦，注册费用`￥25.00`啊。

最近看了一些破解站，都把`文件批量设置器 V2.2a`作为新软件，其实我在 2000 初就看到这个软件了，那时我才开始学破解软件，费了很大劲才把它给破解了，现在想想当初的学习劲头真足啊，眼睛都追成了熊猫（红红的。。）通过破解它，自己的水平也提高了不少（自我感觉良好）。

其实这个软件的破解也不能，只要具备一点汇编知识就能搞定，软件的注册过程满有意思的。我认识是学习破解的一个好教材，建议初哥们都去试一试。

给一点提示吧！注册的注册码是 24 位数，分为 8 组，每组三位数。前四组好象是根据你的用户名和公司名算出来的，具体也记不得了，毕竟是一年多前的事了，后四组有的是通过加减算出来的，好象第七组必须是 100，好了不多说了，自己去体会吧。相信你也会有所提高的。

下载地址：`http://www.esoftware.com.cn/oload.php?url=http://ftp.enet.com.cn/pub/fmanager/other/fo22a.exe`

哎！现在的国产共享软件好象越来越少了，不知大家有没有同感，最后祝大家愚人节快乐。

2001.4.01

```c
#include "stdio.h"

main()
{
    char string[81];

    long d=1;

    int i,c;

    printf("******FLASHSOFT 1.0******\nThis keygen is maked by Pretender\nPlease input your name\n");

    gets(string);

    for(i=0;(c=string[i])!='\0';i++)
    {
        if(c<0) c+=256;

        d*=c;

    }

    d=d>0?d:-d;

    printf("Your name is           ");

    puts(string);

    printf("Your Register code is %ld\n",d+100);

    printf("Welcome to WWW.CRACKNOW.COM\n");
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
