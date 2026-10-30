
# [原创]TX2017游戏安全知识竞赛第二轮第一题

TX2017游戏安全知识竞赛第二轮第一题

## 一、整体轮廓

IDA载入，发现关键位置函数`sub_401694()`，如下，其中的注释已经很清楚。重点就是`sub_4011D4`、`sub_4014B8`和`sub_401634`三个函数。

[![](https://bbs.kanxue.com/upload/attach/201707/119492_78lhmt7omenc4j9.png)](https://bbs.kanxue.com/elink@128K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0p5I4z5e0b7&6x3W2)9#2k6U0M7^5L8r3S2E0N6o6N6G2L8h3g2F1j5K6c8B7z5g2)9J5k6i4m8F1k6H3`.`.)

![](https://bbs.kanxue.com/upload/attach/201707/119492_3z8ptjuaqci6pa5.png)

其中`sub_4011D4`处理通过调用`zapus_get`得到的16字节数据。

结果与如下16字节数据依次比较。

```
unsigned char fii[16] = {'G','S','L','a','b','1','7'};//对比字符串

unsigned int xy = GetCurrentProcessId();

unsigned int * fi1 = (unsigned int *)fii;

fi1[3] = xy;
```

`sub_4014B8`和`sub_401634`都是对`zapus_dll.dll`文件求hash，结果都与`0x614C5347`比较。

## 二、重点函数 `sub_4011D4` 的逆向

IDA进入该函数，

[![](https://bbs.kanxue.com/upload/attach/201707/119492_7f4ferx1n0rglcj.png)](https://bbs.kanxue.com/elink@d7cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0p5I4z5e0b7&6x3W2)9#2k6U0N6X3y4r3k6W2M7Y4R3I4L8U0m8J5k6$3I4U0K9W2)9J5k6i4m8F1k6H3`.`.)

转化为C代码如下，其中`fcode`是固定值，具体见代码，`inbuf`是测试数据，`v5`是处理结果。

```
int getTheKey1(){

       int i,j,k;

       unsigned char inbuf[] = "0123456789abcdef";

       DWORD v8 = 0x1000193;

       DWORD v7 = 0x811C9DC5;

       for (i = 0; i < 0x800; ++i ){

              v7 *= v8;

              fcode[i] ^= v7;

              v7 ^= fcode[i];

       }

       char v5[16] = {0}; 

       for (j = 0; j < 0x80; ++j ){

              unsigned char v3 = 0;

              for (k = 0; k < 0x80; ++k )   

                     v3 = ( ((signed int)fcode[16*j+k/8] >> k % 8) & ((signed int)inbuf[k/8]>> (7 - k % 8)) ^ v3) & 1;

              v5[j / 8] |= v3 << (7 - j % 8); 

       }

       int ret = 0;

       return ret;

}
```

分析易知，在已知处理结果的情况下求输入，实际上就是解异或方程组，代码如下。

```
unsigned char ut1[0x80][0x81]={0};//fcode2bit（j，k）

unsigned char ut2[0x80]={0};//inbuf2bit

unsigned char inb[0x10]={0};

 

//高斯消元法解异或方程

void Gauss ()

{

              int i,j,k;

              for (k=0;k<0x80;k++)

              {

                     //i=k;

                     for (i=k;i<0x80;i++)

                     {

                            if (ut1[i][k]==1) break;

                     }

 

                            for (j=0;j<=0x80;j++)

                            {

                                   unsigned char tmp = ut1[k][j];

                                   ut1[k][j] = ut1[i][j];

                                   ut1[i][j] = tmp;

                            }

                            

                            for (i=0;i<0x80;i++)

                            {

                                   if (i!=k && ut1[i][k])

                                   {

                                          for (j=0;j<=0x80;j++)

                                                 ut1[i][j]=ut1[k][j]^ut1[i][j];

                                   }

                            }

              }

              for (i=0;i<0x80;i++)

              {

                     ut2[i]=ut1[i][0x80];

                     inb[i/8] |= ut2[i]<<(7-i%8);

              }

}
```

```
__declspec(dllexport) int zapus_get(char * c)

{

       int i,j,k;

 

       DWORD v8 = 0x1000193;//FNVHash常量

       DWORD v7 = 0x811C9DC5;

 

       unsigned char fii[16] = {'G','S','L','a','b','1','7'};//对比字符串

       unsigned int xy = GetCurrentProcessId();

       unsigned int * fi1 = (unsigned int *)fii;

       fi1[3] = xy;

 

       for (i = 0; i < 0x800; ++i )

       {

              v7 *= v8;

              fcode[i] ^= v7;

              v7 ^= fcode[i];

       }

 

       for (j = 0; j < 0x80; ++j )

       {

              for (k = 0; k < 0x80; ++k )   

              {

                     ut1[j][k] = (fcode[16*j+k/8]>>k%8) & 1;

              }

       }

 

       for(i=0;i<0x80;i++)

       {

              ut1[i][0x80] = (fii[i/8]>> (7 - i % 8))&1;

              //            printf("%x",ut3[i]);

       }

 

       Gauss();

 

       /*

       //此题如果不要求算法分析，则可在此处算好结果后，直接传回主程序32字节，直接传全0都可满足要求

       for (int j = 0; j < 0x80; ++j )

       {

              unsigned char v3 = 0;

              for (int k = 0; k < 0x80; ++k )      

                     v3 = ( ((signed int)fcode[16*j+k/8] >> k % 8) & ((signed int)inbuf[k/8]>> (7 - k % 8)) ^ v3) & 1;

              //v3 = (((signed int)(unsigned __int8)*(&fcode[16 * j] + k / 8) >> k % 8) & ((signed int)inbuf[k/8]>> (7 - k % 8)) ^ v3) & 1;

              inbuf[16+j / 8] |= v3 << (7 - j % 8); 

       }

       memcpy(c,inbuf,32);

       */

       memcpy(c,inb,16);

       return 16;

}
```

然而，由于`zapus_get`返回值没有限制长度，且返回值紧贴预设的比较字符串，导致可以直接覆盖预设字符串达到目的，进而跳过此步验证。

由于编写的`zapus_get`和题目预设的返回方式不一样，预设为“`retn 4`”返回（可能为`delphi`或者`Borland C++`），而`VC++`默认为“`retn`”返回，导致堆栈不平衡，出现错误。

[![](https://bbs.kanxue.com/upload/attach/201707/119492_pbghgdewbdqijfl.png)](https://bbs.kanxue.com/elink@807K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0p5I4z5e0b7&6x3W2)9#2k6Y4m8T1k6$3S2Y4k6r3g2%4j5X3c8I4K9h3A6X3L8q4)9J5k6i4m8F1k6H3`.`.)

手动修改程序的返回方式为“`retn 4`”如下，即可完美实现该函数。

![](https://bbs.kanxue.com/upload/attach/201707/119492_isxbp56lyqk74ox.png)

## 二、重点函数 `sub_4014B8` 的逆向

该函数实际就是`crc32`，核心如下，下图生成CRC32使用的表。

![](https://bbs.kanxue.com/upload/attach/201707/119492_guwc2ia1oev6xg6.png)

下图处理输入

![](https://bbs.kanxue.com/upload/attach/201707/119492_s2pzumtgzlcyvhk.png)

因为表固定，不做重点分析，输入处理过程可化简为：

```
//CRC32编码

int getTheKey2(unsigned char * buf,int bufsize)

{

       DWORD ret = -1;

       DWORD * bb = (DWORD*)aa;

       for(int i=0;i<bufsize;i++)

       {

              int xt = (ret&0xff)^buf[i];

              ret = bb[1+xt]^(ret>>8);

       }

       

       return ~ret;

}
```

容易发现通过控制最后的4个字节，即可完全控制最终生成的CRC32结果，那么我们可以通过在文件后加4个字节来达到控制CRC32结果为`0x614C5347`的目的，求此4个字节的代码如下。

```
unsigned char buf[4]={0};

int get2(DWORD a)

{

       DWORD confirm1 = 0x9e; //0x9eb3acb8 == ~0x614C5347

       DWORD confirm2 = 0xb3;

       DWORD confirm3 = 0xac;

       DWORD confirm4 = 0xb8;

       DWORD tmp,x[4]={0};

       int i,y[4]={0};

       DWORD * bb = (DWORD*)aa;

       for(i=1;i<=0x100;i++){

              tmp = bb[i]>>24;

              if(tmp==confirm1){

                     x[0] = bb[i];

                     y[0] = i;

                     break;

              }

       }

       tmp = x[0]>>16;

       tmp = tmp&0xff;

       confirm2 = confirm2^tmp;

       for(i=1;i<=0x100;i++){

              tmp = bb[i]>>24;

              if(tmp==confirm2){

                     x[1] = bb[i];

                     y[1] = i;

                     break;

              }

       }

       tmp = x[0]>>8;

       tmp = tmp&0xff;

       confirm3 = confirm3^tmp;

       tmp = x[1]>>16;

       tmp = tmp&0xff;

       confirm3 = confirm3^tmp;

       for(i=1;i<=0x100;i++){

              tmp = bb[i]>>24;

              if(tmp==confirm3){

                     x[2] = bb[i];

                     y[2] = i;

                     break;

              }

       }

 

       tmp = x[0];

       tmp = tmp&0xff;

       confirm4 = confirm4^tmp;

       tmp = x[1]>>8;

       tmp = tmp&0xff;

       confirm4 = confirm4^tmp;

       tmp = x[2]>>16;

       tmp = tmp&0xff;

       confirm4 = confirm4^tmp;

       for(i=1;i<=0x100;i++)

       {

              tmp = bb[i]>>24;

              if(tmp==confirm4){

                     x[3] = bb[i];

                     y[3] = i;

                     break;

              }

       }

       DWORD ret = a;//0x32f38783;

       for(i=3;i>=0;i--){

              buf[3-i] = ((ret&0xff)^y[i]-1);

              ret = x[i]^(ret>>8);

       }

       return 0;

}
```
