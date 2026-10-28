
# [原创]VC控制台修改文件时间属性

最近对VC实现修改文件创建、访问、修改时间属性的方法产生了兴趣，我便在网上查了这方面的教程，但是却总是看不明白，有的不够简洁。例如在一些文章中提到了使用`FindFirstFile()`函数和`WIN32_FIND_DATA`结构，我认为完全没有必要，因为到最后时我们需要用到`SetFileTime()`函数，而这个函数和`FindFirstFile()`是无关的，我们只需用到`SYSTEMTIME`和`FILETIME`结构体。如果感兴趣的话，你可以先看看微软的官方帮助和支持文档：`FILETIME`结构使用信息，里面讲的不仅仅是`FILETIME`结构的使用，很值得一看。

```c
struct _FILETIME {     //结构体定义
    DWORD dwLowDateTime;
    DWORD dwHighDateTime;}
 FILETIME;
//FILETIME 结构表示 1601 年 1 月 1 日以来 100 纳秒为间隔数。结构包含的这两个DWORD值组合在一起
//形成一个 64 位值的 32 位值。
struct _SYSTEMTIME {     //这个结构体的成员实际上查查字典就知道什么意思了，呵呵
    WORD wYear ;   //年
    WORD wMonth ;//月
    WORD wDayOfWeek ;
     //星期，0=星期日，1=星期一...
    WORD wDay ;     //日
    WORD wHour ;    //时
    WORD wMinute ;    //分
    WORD wSecond ;    //秒
    WORD wMilliseconds ;   //毫秒
}SYSTEMTIME;
```

头文件：`windows.h`

接着定义变量。

```c
    FILETIME Filetime;
    SYSTEMTIME Systemtime;
```

接着，设置日期和时间，我们这里设置成1999年1月1日 01:01:01：

```c
    Systemtime.wDay = 1;
    Systemtime.wYear = 1999;
   //一定要大于1601，因为FILETIME的时间是从1601年1月1日开始
   //如果不大于1601，SetFileTime()会返回false，GetLastError()
   //返回78 （无效参数）
    Systemtime.wMonth = 1;
    Systemtime.wHour = 1;
    Systemtime.wMinute =1;
    Systemtime.wDayOfWeek = 1;
    Systemtime.wSecond = 1;
    Systemtime.wMilliseconds = 1;
```

转换`SystemTime`为`Filetime`结构：

```c
    SystemTimeToFileTime( &Systemtime, &Filetime );
```

由于`SetFileTime()`需要文件句柄，所以只好用`CreateFile()`打开这个文件（微软官方实例）

总算快完成了，但是还有最关键的一步（写入）：

```c
     HANDLE hFile = CreateFile( "C:\\测试.txt",GENERIC_WRITE, FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, OPEN_EXISTING, 0, NULL );
    if (hFile != INVALID_HANDLE_VALUE)
    {
         SetFileTime( hFile, &Filetime, &Filetime, &Filetime );
        CloseHandle(hFile);
    }
```

注：`SetFileTime`函数函数原型：

```c
BOOL SetFileTime(   //返回值：TRUE表示成功，FALSE表示失败
     HANDLE hFile,                     // 文件句柄
     CONST FILETIME *lpCreationTime,   // 文件创建时间
     CONST FILETIME *lpLastAccessTime, // 最后访问时间
     CONST FILETIME *lpLastWriteTime   // 最后修改时间
   );
```

你运行试试看，是不是OK了？不，远远不是。我们来看看程序运行后文件的创建时间：

你会问，怎么是1999年1月1日，9:01:01？我们明明设置的是1999年1月1日，1:01:01啊！这个问题我也纠结了好半天，最后总算找到了答案：时区! 我国标准北京时间是GMT+8:00，所以你只要把时间GMT标准之间就可以了，不过还得再说明一下，如果你调成GMT（格林威治时间）的话，还是会有1小时的误差，也就是说Microsoft Windows的文件时间不是按的GMT（格林威治时间）来计算的。我们只要把时区调成GMT（协调世界时）就可以了。或者，在程序中这样修改一下：

```c
     Systemtime.wDay = 31;
     Systemtime.wYear = 1998;
    Systemtime.wMonth = 12;
     Systemtime.wHour = 17;
     Systemtime.wMinute = 1;
     Systemtime.wDayOfWeek = 1;
     Systemtime.wSecond = 1;
     Systemtime.wMilliseconds = 1;
```

就可以了。

范例下载地址：[b36K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8Y4m8S2L8W2)9J5k6h3y4F1i4K6u0r3f1e0g2$3g2W2k6y4M7g2y4n7L8X3u0K6g2q4)9J5y4#2)9J5y4X3N6@1i4K6y4n7x3K6j5H3i4@1f1@1i4@1u0m8i4K6V1I4i4@1f1%4i4K6W2n7i4K6V1^5i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6S2i4K6t1$3k6%4c8Q4x3@1u0Q4c8f1k6Q4b7V1y4Q4z5o6V1](<b36K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8Y4m8S2L8W2)9J5k6h3y4F1i4K6u0r3f1e0g2$3g2W2k6y4M7g2y4n7L8X3u0K6g2q4)9J5y4#2)9J5y4X3N6@1i4K6y4n7x3K6j5H3i4@1f1@1i4@1u0m8i4K6V1I4i4@1f1%4i4K6W2n7i4K6V1^5i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6S2i4K6t1$3k6%4c8Q4x3@1u0Q4c8f1k6Q4b7V1y4Q4z5o6V1>).

转载请注明：[af3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8X3&6&6M7$3q4T1i4K6u0W2j5X3I4G2k6#2)9J5k6e0p5$3x3#2)9J5k6h3y4G2L8g2)9J5c8R3`.`](https://bbs.kanxue.com/elink@1f6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8X3&6&6M7$3q4T1i4K6u0W2j5X3I4G2k6#2)9J5k6e0p5$3x3#2)9J5k6h3y4G2L8g2)9J5c8R3`.`.)

实际上这篇文章是我原创的，sunnysab.blog.163.com是我博客

更好的排版，请访问：[505K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8X3&6&6M7$3q4T1i4K6u0W2j5X3I4G2k6#2)9J5k6e0p5$3x3#2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6K6N6r3q4@1K9h3y4Q4x3V1j5I4z5o6l9K6y4K6f1H3x3o6V1J5x3o6p5K6y4o6t1J5x3e0t1&6y4e0b7J5y4g2)9J5c8R3`.`](https://bbs.kanxue.com/elink@916K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2L8X3&6&6M7$3q4T1i4K6u0W2j5X3I4G2k6#2)9J5k6e0p5$3x3#2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6K6N6r3q4@1K9h3y4Q4x3V1j5I4z5o6l9K6y4K6f1H3x3o6V1J5x3o6p5K6y4o6t1J5x3e0t1&6y4e0b7J5y4g2)9J5c8R3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
