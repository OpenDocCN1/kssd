
[![RnZjh9.png](upload/attach/202106/865862\_K69BPU4CGJRVSF8.jpg)](<mark class="encrypted">7cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9L8W2A6B7K9o6W2Q4x3U0V1`.</mark>
一共有五个功能，增查改删以及切换人物
点开`case 1`对应的函数，也就是`add`函数
[![RneP0O.png](upload/attach/202106/865862\_BES39DR6Y39DJBJ.jpg)](<mark class="encrypted">d1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9L8X3g2b7x3p5!0Q4x3U0V1`.</mark>
里面嵌套着一个`switch`，点开`case1`对应的函数看看
[![Rne8hj.png](upload/attach/202106/865862\_A3HS4SUTGEWCRNP.jpg)](<mark class="encrypted">d5dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9L8X3f1^5K9r3A6Q4x3U0V1`.</mark>
相当难看，很多对地址进行的操作，这种情况尽可能恢复程序的结构体方便后续分析，先大致分析一下这个函数的流程。
最多能申请20次，申请的`chunk`的大小要大于等于`0x90`小于等于`0x430`。并且每一次申请的`chunk`的`size`都要大于等于上一次所申请的。申请堆块用的是`calloc`函数，`calloc`不从`tcache`中取。
```c
\*(\_DWORD \*)(unk\_9070 + 0x150LL) = v8;
```
如果`size`合法的话，就将`size`赋值给`\*(\_DWORD \*)(unk\_9070 + 0x150LL)`，这个`unk_9070`中存放的是`mmap`出来的一块内存的地址。
```c
\*(\_DWORD \*)(a1 + 4 \* (i + 48LL)) = v8;
\*(\_BYTE \*)(a1 + i + 288) = 0;
\*(\_BYTE \*)(a1 + i + 312) = 0;
```
这一段寻址看的不清晰，我们修改一下这个函数传入的参数的类型
[![RKKhLR.png](upload/attach/202106/865862\_F7XK6V728VFBA6S.jpg)](<mark class="encrypted">cd8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9@1E0Z5e0q4u0Q4x3U0V1`.</mark>
原本是`int64`类型，我们将其修改为`char *`，在参数上按y修改类型
[![RKKoo6.png](upload/attach/202106/865862\_TMRTRVBZ6FX4HXD.jpg)](<mark class="encrypted">e15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9@1E0G2L8K6k6Q4x3U0V1`.</mark>
修改完后上面那一段就变成了如下所示
[![RKKOQH.png](upload/attach/202106/865862\_EBYH3ADBEWZ28FZ.jpg)](<mark class="encrypted">ae2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9@1E0a6f1f1S2Q4x3U0V1`.</mark>
这样子看起来就比上面那段看起来清楚一些，将`a1[4 * i + 0xC0]`赋值为`size`，将`a1[i + 0x120]`以及`a1[i + 0x138]`都赋值为0。
继续往下看
[![RKMklQ.png](upload/attach/202106/865862\_XJ96YMKD8RYAJF4.jpg)](<mark class="encrypted">6b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9@1#2C8L8q4q4Q4x3U0V1`.</mark>
将`size/0x30`，也就是将申请的`chunk`以`0x30`大小切片，每次都往切片的顶部读入`0x10`个字节的数据。
这个函数到这就算分析得差不多了，但如果我们要恢复程序使用的结构体，这些信息还不够，继续分析其他函数。
再看到`add`功能的`case 2`
`case 2`的函数与`case1`对应的函数只有几处不一样
[![RKQwEq.png](upload/attach/202106/865862\_9MGJEVE3NYDNQ6H.jpg)](<mark class="encrypted">9e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9#2q4%4c8i4q4Q4x3U0V1`.</mark>
`size`存放到`mmap`中的位置变了
[![RKQrCT.png](upload/attach/202106/865862\_PAQHD36RZNA2K6Q.jpg)](<mark class="encrypted">fafK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9#2q4J5b7#2c8Q4x3U0V1`.</mark>
将`chunk`切片之后不再是从顶部写入，而是从偏移`0x10`处开始写
`case3`对应的函数不同之处也是在这两处，就不再说了。
由`add`功能可以猜测，`add`功能传入的参数应该是一个结构体，
```c
\*(\_QWORD \*)&a1[8 \* i] = calloc(1uLL, v8); //存储堆地址
\*(\_DWORD \*)&a1[4 \* i + 0xC0] = v8; //存储chunk size
a1[i + 0x120] = 0; //意义不明
a1[i + 0x138] = 0;//意义不明
```
再看到`view`功能
[![RKzN40.png](upload/attach/202106/865862\_6JV5XD8MWM6SVSY.jpg)](<mark class="encrypted">6beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9d9%4A6z5y4o6m8Q4x3U0V1`.</mark>
首先判断`\*(int \*)(unk\_9070 + 0x404LL)`的值是否大于0，然后依然有三个函数，进入第一个函数
[![RMS0it.png](upload/attach/202106/865862\_WKY5EF2F9W5XBD5.jpg)](<mark class="encrypted">77dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2x3H3K9i4c8Q4x3U0V1`.</mark>
输入一个序号，然后会有以下三个判断
```c
\*(\_QWORD \*)&a1[8 \* v4] //判断chunk是否存在
\*(\_DWORD \*)&a1[4 \* v4 + 0xC0] //判断对应的size是否存在
!a1[v4 + 0x120]
```
满足这三个条件就会输出对应`chunk`的值
另外两个函数也基本一致
看到`edit`功能
[![RM9FjU.png](upload/attach/202106/865862\_W4TYEEYWUDZCJX6.jpg)](<mark class="encrypted">5e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0e0W2r3K9W2g2Q4x3U0V1`.</mark>
和`view`功能一样，开头会检查一个全局变量的值`\*(int \*)(unk\_9070 + 0x408LL)`是否大于0，大于0才会进入到`edit`的子函数中，查看第一个函数
[![RM9rDg.png](upload/attach/202106/865862\_V3V7MT7J9QQ7KUC.jpg)](<mark class="encrypted">359K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0e0W2J5c8r3N6Q4x3U0V1`.</mark>
依然有三个判断，和`view`的判断是相同的，通过判断的话就会重新往`chunk`中写入数据，也是先切片再写数据，和`add`功能中的逻辑一样
看到`delete`功能
[![RM9Ha9.png](upload/attach/202106/865862\_6F244M9JW4YQP43.jpg)](<mark class="encrypted">793K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0e0W2t1j5e0W2Q4x3U0V1`.</mark>
`delete`功能没有次数的限制，进入第一个子函数
[![RM9jxK.png](upload/attach/202106/865862\_J733NXXNUP8C3UC.jpg)](<mark class="encrypted">2e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0e0W2B7P5p5E0Q4x3U0V1`.</mark>
`delete`功能依然有三个检查
```c
\*(\_QWORD \*)&a1[8 \* v4]
&& !a1[v4 + 0x120]
&& !a1[v4 + 0x138]
```
检查`chunk`是否存在，检查`!a1[v4 + 0x120]` 和`!a1[v4 + 0x120]`这两个位置是否为0，通过检查的话则进行`free`，并将两个标志位置1，但并没有将堆指针清0。
前四个功能分析完毕，第五个功能稍后再谈，我们先通过这四个功能恢复出程序使用的结构体
总结如下
```
1.结构体至少需要存储20\*8=0xa0的堆地址
2.还需要存储20\*4=0x50的size数据
3.view，edit功能使用了a1[v4 + 0x120]处的标志位
4.delete功能使用了a1[v4 + 0x138]和a1[v4 + 0x120]处的标志位，也就是结构体应该有两个标志位数组，数组元素大小为1字节，因此结构体还需要存储20\*1\*2=0x28大小的标志位
```
初步猜测结构体应该如下所示
```c
struct house
{
char \*list[20];
int size[20];
char flag[20];
char flagg[20];
};
```
但这样子其实是有错误的，回到`add`功能
[![RMP9yT.png](upload/attach/202106/865862\_ZCCCGQJZTNS5PSU.jpg)](<mark class="encrypted">090K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2l9&6P5g2c8Q4x3U0V1`.</mark>
堆地址是从结构体的头部开始存储的，而`size`数据则是从结构体偏移`0xc0`开始存储的，如果只存储20个堆地址的话，那么只占用0xa0的大小，`size`应该从0xa0开始存储；要使`size`从0xc0开始存储，则堆地址存储的上限应为0xc0/0x8=0x18个，同样的，`size`，`flag`，`flagg`的存储上限也应为0x18，这样的话结构体应该如下
```c
struct house{ char \*list[0x18]; int size[0x18]; char flag[0x18]; char flagg[0x18];};
```
规划好结构体之后就可以往ida中添加了
[![RMPfnU.png](upload/attach/202106/865862\_JJ8B4THCC2ENT8Q.jpg)](<mark class="encrypted">010K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2m8X3L8W2g2Q4x3U0V1`.</mark>
点到`local types`界面
[![RMP474.png](upload/attach/202106/865862\_K7YDZDT2YNY4CB2.jpg)](<mark class="encrypted">e40K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2l9@1y4K6c8Q4x3U0V1`.</mark>
右键选择`insert`
[![RMPThR.png](upload/attach/202106/865862\_9C2ZR337AUEHGVJ.jpg)](<mark class="encrypted">3a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2m8f1K9q4u0Q4x3U0V1`.</mark>
直接输入结构体代码
[![RMPb1x.png](upload/attach/202106/865862\_S6VDAK4D4SCXBJH.jpg)](<mark class="encrypted">39fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0g2m8T1x3i4S2Q4x3U0V1`.</mark>
`local types`中会出现我们自定义的结构体类型
我们再回到`add`功能，修改参数类型
[![RMip4A.png](upload/attach/202106/865862\_4Q7VNBCNJYN4SYD.jpg)](<mark class="encrypted">4cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0h3W2H3y4p5q4Q4x3U0V1`.</mark>
将其修改为`house *`类型
[![RMiigP.png](upload/attach/202106/865862\_MDXXH8EW5TEQ5JF.jpg)](<mark class="encrypted">512K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0h3W2A6k6#2m8Q4x3U0V1`.</mark>
修改成功
我们再接着把程序中用到了这个结构体的函数的参数类型统统修改
比如我们在一开始点进去的那个一大堆赋值0的函数，修改完后就变成了下面这样
[![RMi3uT.png](upload/attach/202106/865862\_8QU7UT2KHDU2HER.jpg)](<mark class="encrypted">d99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0h3V1K6N6g2c8Q4x3U0V1`.</mark>
清楚了亿点点
其他需要修改为`house`结构体的位置我就不一一修改给大家看了，可以自行操作
但，程序中不止存在这一个结构体，在`add`功能中
[![RMisbD.png](upload/attach/202106/865862\_X29XYFWR8Q5GYHT.jpg)](<mark class="encrypted">50bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9e0h3W2K6j5V1c8Q4x3U0V1`.</mark>
很明显，`0x9070`的值指向的空间也是一个结构体，我们接下来对这个结构体进行恢复
在程序一开始，有这么一个函数
