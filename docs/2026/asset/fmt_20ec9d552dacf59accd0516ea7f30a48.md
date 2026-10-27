
# [算法]动词过去式还原程序(已更新)

文章作者：sudami

信息来源：邪恶八进制信息安全团队（[97bN6%4N6%4i4K6u0W2k6i4k6A6L8r3!0U0N6r3q4D9i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@348K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2$3K9h3I4G2j5%4c8S2L8q4)9J5k6h3y4G2L8b7`.`.)）

刚上经常逛的另一个论坛,收到一个求助者的短信,让我帮他做这个题目:

> 动词过去式还原程序：
> 
> 1.内容
> 
> 设计程序, 输入一组过去式程序,输出相应的动词原型
> 
> 2. 要求
> 
>  程序具有不规则动词表输入功能.
>  不规则动词以文件形式存储,可以从文件读入 存入安字母排序的结构体数组
>  对于不是动词过去式的输入应给予提示!

--------------------------------------------------------------------------------------------------------

大致做了,实现了以下功能:

１.过滤不是字母的字符. 比如输入:2%read#ed,会得到：read

２.具有添加不规则动词的功能，但我发现一个bug：

　我用`CFile::modeCreate | CFile::modeNoTruncate`来保证打开存在的文件时仅仅是打开而不是截去,但运行时总是把文件截断为0,很是不解.

```
// 打开文件

        CStdioFile   fText;  

        CFileException FileExc;

        UINT nOpenFlags;

        CString str;

        

        nOpenFlags = CFile::modeReadWrite | CFile::modeCreate | CFile::modeNoTruncate;

        if (!fText.Open(szBuffer, nOpenFlags, &FileExc))

        {

            FileExc.ReportError ();

            return 0;

        }
```

----------------------------(这个问题已经解决,原来问题不是出在这里,是在`wt`这个上)---------------

```
if ((fp = fopen (szBuffer, "wt")) == NULL)

    {

        ::MessageBox (NULL, "Can not open it!", "sudami", MB_OK);

        exit (1);

    }
```

这里的`wt`  应该改成 `at+`, 即以读写形式打开文件

３.＂对于不是动词过去式的输入应给予提示!＂

这个功能我想了半天都不知道如何实现，怎么能判断一个不规则的词是动词过去式还是其他的呢～～

4.

**代码优化...**

```
          

             if (temp[i-3] == temp[i-4])

            {

                i = i - 3;

                j = i - 1;

                cout << "---->Result:" ;

                while (i--)

                {

                    cout << temp[j-i];

                }

                cout << endl;

                

                cout << "Input:";

                cin.getline (g_buffer, MAX_PATH);

                Convert (g_buffer);    

            } 



            else

            {

                cout << "---->Result:" ;

                i = i - 2;

                j = i - 1;

                while (i--)

                {

                    cout << temp[j-i];

                }

                cout << endl;

                cout << "Input:";

                cin.getline (g_buffer, MAX_PATH);

                Convert (g_buffer);    

            } // if
```

这里if和else里的内容差不多,累赘.,应该改成:

```
i=(temp[i-3]==temp[i-4])?(i-3):(i-2);

                j = i - 1;

                cout << "---->Result:" ;

                while (i--)

                {

                    cout << temp[j-i];

                }

                cout << endl;

                

                cout << "Input:";

                cin.getline (g_buffer, MAX_PATH);

                Convert (g_buffer);    
```

**算法问题**

```
          

if (temp[i-1] == 'd')

       {

        if (temp[i-2] == 'e')                //注意:下面没有else与这个if对应

        {

            if (temp[i-3] == temp[i-4])

            {

             .....

            } 



            else

            {

             ....

            }

        } 

    } 

    

    else

    {

    ....//处理倒数第2个字母不是e的情况

    }
```

由于第二个if没有else对应....程序会忽略很多这种最后一个字母是d而倒数第二个字母不是e的不规则动词..

譬如:did,could,should,read....

其实,...这里的算法本身就有问题...没有考虑很多不规则动词的情况...

根据算法...

fled------->fl(应该为fleed)

bled------>bl(应该为bleed)

led-------->l(应该为lead)

added---->ad(应该为add)

loved----->lov(应该为love)

married--->marri(应该为marry)

......

还有很多例子...

由于写这个的时候把这些遗忘了,故补充进来供有兴趣的深入~

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－------

程序运行截个图:

[![](https://bbs.kanxue.com/<mark class=)c2dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4#2k6r3!0G2i4K6u0W2L8X3g2@1i4K6u0r3M7q4)9K6c8X3W2V1i4K6y4p5y4o6l9J5z5o6R3H3z5r3f1I4x3$3p5^5k6h3x3#2y4U0l9I4x3e0y4S2k6X3t1K6y4U0g2X3j5e0N6W2y4o6M7`." />](https://bbs.kanxue.com/elink@e4aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4#2k6r3!0G2i4K6u0W2L8X3g2@1i4K6u0r3M7q4)9K6c8X3W2V1i4K6y4p5y4o6l9J5z5o6R3H3z5r3f1I4x3$3p5^5k6h3x3#2y4U0l9I4x3e0y4S2k6X3t1K6y4U0g2X3j5e0N6W2y4o6M7`.)

![](http://www.qudoo.net/p?id=4028808e13b46f890113b48938e9003d&qudoo.jpg)

-----------------------------------------------------------------------------------------------------------

程序代码如下:
