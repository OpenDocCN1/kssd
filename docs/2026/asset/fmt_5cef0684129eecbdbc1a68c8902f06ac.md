
# [原创]HTTP.sys UAF 漏洞 CVE-2021-31166 简要分析

最近在看`http.sys`，必然需要分析旧的漏洞的，使用网上的poc触发了漏洞后，有了个具体概念，就写一篇简要分析，帮助大家理解

# poc

通过 POC(<a href="elink@888K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3N6X3g2J5j5$3H3H3K9#2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0q4Q4x3X3b7K6x3e0p5$3y4R3`.`.">链接地址</a>) 可知，漏洞是一个UAF漏洞，栈回溯也很清晰：

``` 
# Child-SP RetAddr Call Site
00 fffff90e`a867e368 fffff804`19525382 nt!DbgBreakPointWithStatus
01 fffff90e`a867e370 fffff804`19524966 nt!KiBugCheckDebugBreak+0x12
02 fffff90e`a867e3d0 fffff804`19408eb7 nt!KeBugCheck2+0x946
03 fffff90e`a867eae0 fffff804`1941ad69 nt!KeBugCheckEx+0x107
04 fffff90e`a867eb20 fffff804`1941b190 nt!KiBugCheckDispatch+0x69
05 fffff90e`a867ec60 fffff804`19419523 nt!KiFastFailDispatch+0xd0
06 fffff90e`a867ee40 fffff804`1db3f677 nt!KiRaiseSecurityCheckFailure+0x323
07 fffff90e`a867efd0 fffff804`1daf6c05 HTTP!UlFreeUnknownCodingList+0x63
08 fffff90e`a867f000 fffff804`1dacd201 HTTP!UlpParseAcceptEncoding+0x299c5
09 fffff90e`a867f0f0 fffff804`1daa93d8 HTTP!UlAcceptEncodingHeaderHandler+0x51
0a fffff90e`a867f140 fffff804`1daa8ab7 HTTP!UlParseHeader+0x218
0b fffff90e`a867f240 fffff804`1da04c5f HTTP!UlParseHttp+0xac7
0c fffff90e`a867f3a0 fffff804`1da0490a HTTP!UlpParseNextRequest+0x1ff
0d fffff90e`a867f4a0 fffff804`1daa48c2 HTTP!UlpHandleRequest+0x1aa
0e fffff90e`a867f540 fffff804`1932ae85 HTTP!UlpThreadPoolWorker+0x112
0f fffff90e`a867f5d0 fffff804`19410408 nt!PspSystemThreadStartup+0x55
10 fffff90e`a867f620 00000000`00000000 nt!KiStartSystemThread+0x28
```

从栈回溯可以知道, 它是一个独立线程, 崩溃在解析Header里. 而poc也很简单`"Accept-Encoding: doar-e, ftw, imo, ,"`, 发送一个带`Accept-Encoding`字段的请求, 包含至少一个正常值, 最后带个空值.

# 原理分析

以下是函数`UlpParseAcceptEncoding`的伪代码. 分析参见注释和序号.

```c
while ( 1 )
{
v26[0] = 1000;
ret = UlpParseContentCoding(// 以","为分割, 解析Accept-Encoding字段的值.
v7,
v4,
(unsigned int)&v27,
(unsigned int)&v32,
(\_\_int64)&v30,
(\_\_int64)v26,
(\_\_int64)&v31);// v31指向解析的字符末尾, 比如 传入"a,b,c\r\n", 一轮下来, v31指向"b,c\r\n".
if ( ret < 0 )
{
if ( ret != 0xC0000225 ) // 1. 当最后一次的长度为0时, UlpParseContentCoding会返回 0xC0000225
goto LABEL\_46;
if ( v31 == end && !v9 )
{
ret = 0;
goto LABEL\_25;
}
}
....
if ( v31 >= end ) // 2. 当刚好是末尾的时候, v31就是消息的末尾. 所以条件成立, 跳出while循环
break;
v7 = v31;
LODWORD(v4) = end - v31;
}
Flink = v29.Flink;
if ( v29.Flink != &v29 )// 3. 当本地链表v29里有链接的时候, 链接到a3变量的结构体里.
{
v20 = v29.Blink;
if ( v29.Flink->Blink != &v29
|| v29.Blink->Flink != &v29
|| (v29.Blink->Flink = v29.Flink,
p\_encoding\_link\_off10h\_990h = &a3->encoding\_link\_off10h\_990h,
Flink->Blink = v20,
v22 = a3->encoding\_link\_off10h\_990h.Blink,
a3->encoding\_link\_off10h\_990h.Flink->Blink != &a3->encoding\_link\_off10h\_990h)
|| v22->Flink != p\_encoding\_link\_off10h\_990h
|| Flink->Flink->Blink != Flink
|| v20->Flink != Flink )
{
LABEL\_47:
\_\_fastfail(3u);
}
v22->Flink = Flink;
a3->encoding\_link\_off10h\_990h.Blink = Flink->Blink;
Flink->Blink->Flink = p\_encoding\_link\_off10h\_990h;
v23 = v28;
Flink->Blink = v22;
\*(\_WORD \*)&a3->gap88C[254] = v23; // 4. 链接完成后, 没有清理v29链表的前项和后项;
LABEL\_46:
Flink = v29.Flink;
}
if ( ret < 0 ) // 5. 因为ret=0xC0000225, 条件成立, 跳转LABEL\_33
goto LABEL\_33;
LABEL\_33:
if ( Flink != &v29 )
UlFreeUnknownCodingList(&v29);
```

以下是`UlFreeUnknownCodingList`函数:

```c
while ( 1 )
{
Flink = a1->Flink;
if ( a1->Flink == a1 )// 7. 当链表第一个项被释放后, 此条件还是成立
break;
v3 = Flink->Flink;
if ( Flink->Flink->Blink != Flink || (v4 = Flink->Blink, v4->Flink != Flink) )// 8. 当链表第一个项被释放后, 链表已经错乱, 这个条件就成立了, 最后触发崩溃
\_\_fastfail(3u);
v4->Flink = v3;
v3->Blink = v4;// 6. 此处也没有重置Flink的前项和后项字段.
ExFreePoolWithTag(&Flink[-1], 0);
}
```

通过上述注释, 可以明显看到问题出在`v29`的链表管理上.

下面是释放前后的关系:

释放前的链表关系:

![图片描述](upload/attach/202301/674429_3MKJMZSVAXZ9BFD.png)

步骤7时的链表关系:

![图片描述](upload/attach/202301/674429_SKJHF9DUJ2CVHEP.png)

# 补丁

知道了根本原因, 修补也很简单, 在步骤3和步骤4之间, 把`v29`的指针重置一下即可. 而官方补丁也是这样做的.

# CVE-2022-21907

根据测试来看, 在本来win10 2004已经补了CVE-2021-31166, 它的继任者 win 10 21H1, 又重新引入了这个bug, 看起来在6月(或者5月?)的补丁里又补了它, 导致有的人测试还以为CVE-2022-21907就是CVE-2021-31166, 因此github上也有很多拿CVE-2021-31166的poc当作CVE-2022-21907的poc的情况.

事实上CVE-2022-21907属于另一个未初始化的问题. 参考<a href="elink@0a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2i4K6u0V1L8r3q4T1M7#2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3M7s2u0G2L8$3k6Q4x3X3c8U0L8$3&6U0k6i4m8@1i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9J5x3W2)9J5k6o6t1I4z5e0l9%4i4K6u0V1K9s2c8@1M7q4)9J5k6s2m8J5L8%4c8G2j5$3!0D9i4K6u0V1M7%4c8S2j5$3E0Q4x3X3c8J5k6h3#2G2N6r3g2Q4x3X3c8U0L8$3c8W2i4K6u0V1k6i4S2W2j5%4g2@1K9h3!0F1">此处链接</a>

问题出在`UlFastSendHttpResponse`函数内, 某种条件下, 让第一次`UlpAllocateFastTracker`申请就被释放, 然后走第二次申请,让申请的内存没有初始化重要的+80字段的内容, 然后在后续的`UlGenerateFixedHeaders`中失败, 直接跳转到`MmUnmapLockedPages`去调用未初始化的数据, 导致崩溃. 具体成因比较复杂, 就不深入分析了.

# 总结

可以看到, 问题主要还是在于`v29`在链接了其它链表时, 没有清理自身的链表, 导致有可能UAF, 如果及时清理了, 其实也就不会造成安全问题. 所以我们分析软件安全问题时, 也要多注意这种类似点, 提高敏感度

最近在看`http.sys`，必然需要分析旧的漏洞的，使用网上的poc触发了漏洞后，有了个具体概念，就写一篇简要分析，帮助大家理解

# poc

通过 POC([链接地址](<https://bbs.kanxue.com/elink@634K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3N6X3g2J5j5$3H3H3K9#2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0q4Q4x3X3b7K6x3e0p5$3y4R3`.`.`.)>) 可知，漏洞是一个UAF漏洞，栈回溯也很清晰：

|  |  |
|---|---|

