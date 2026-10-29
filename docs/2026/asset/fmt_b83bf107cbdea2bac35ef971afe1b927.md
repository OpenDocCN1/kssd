
# 6. ida+unidbg分析sign的具体如何生成的

ida导入so包 export导出中找到sign生成方法 截图如下:

![](https://bbs.kanxue.com/upload/attach/202203/884888_DCM9ETUV3XSB8DZ.jpg)

F5生成伪代码

![](https://bbs.kanxue.com/upload/attach/202203/884888_E2629KVAW7GT2M5.jpg)

根据方法的具体逻辑修改了下方法名称 方便直观查看。初步观察是讲传入参数通过key=value的方式拼接起来

通过unidbg的执行日志中也可以发现这点

![](https://bbs.kanxue.com/upload/attach/202203/884888_FP86F2ZVM5DBYW7.jpg)

接下来开始从函数返回的地方开始往上分析,单独调用执行返回如下:

`st=1648374072802&sign=8102aefd41782d405174725cd433c5a0&sv=111`

从上可知要分析的为`st`、`sign`、`sv`这个三个值

1. `st`生成分析: 生成一个时间戳 长度为13位的 用于计算`sign`

2. `sign`的生成:

分析伪代码找到`sign`的生成方法为`sub_126AC`,该方法上面的伪代码都是一些拼接字符串的操作,就是把入参拼接起来。

![](https://bbs.kanxue.com/upload/attach/202203/884888_W8PWT6RVUGFE2HY.jpg)

> 接下来开始unidbg hook该方法

![](https://bbs.kanxue.com/upload/attach/202203/884888_XVJD4N8J9U2GADD.jpg)

> 然后开启无尽的S 单步执行往下走当程序走到`switch`判断时 发现`bl`指令跳转到`sub_10de4`函数 多次在这个位置debugger 到 发现 到 `sv =111` 走的是 `case 2` 还遇到过`sv = 110` 走的是`case 1`、这部分我们只分析为2的情况也就是走`sub_10DE4`,至于为啥呢？就是因为这后面走的加密算法看起来简单些 像`md5`

![](https://bbs.kanxue.com/upload/attach/202203/884888_3Q7GMUWH8RBQ9EK.jpg)

> 查看下函数`sub_10de4`入参,hook该函数

> 打印`mr1`,为一串固定的字符串

![](https://bbs.kanxue.com/upload/attach/202203/884888_PXF3WK4J8HH8QK4.jpg)

> `mr2` 为`0x1` 固定值

> `mr3` 打印如下，为前面拼接的参数串 长度为`0x106`

![](https://bbs.kanxue.com/upload/attach/202203/884888_ZDSNWPYPCWVFE7K.jpg)

> 接着分析`sub_10de4` 这个函数,点击进入`sub_12ECC`函数 ,这里为啥不分析`sub_12FF0` 呢 因为具体看了下没啥特别的。

![](https://bbs.kanxue.com/upload/attach/202203/884888_RRDX4H4FMGSFKAN.jpg)

hook `sub_12ECC`函数分析下入参

> 参数1:打印看看 ,看不太出来是什么 但是是64位 看后面的伪代码这个值主要是 跟 `md5`魔数有关系

![](https://bbs.kanxue.com/upload/attach/202203/884888_RKXR6PD94UZF3DQ.jpg)

![](https://bbs.kanxue.com/upload/attach/202203/884888_6PXJZYCETMQ7CED.jpg)

> 参数2: 固定字符串 `80306f4370b39fd5630ad0529f77adb6`

![](https://bbs.kanxue.com/upload/attach/202203/884888_2S2RAGQSSQK8SVT.jpg)

> 参数3: `0x1`

> 参数4: 为明文的参数拼接串

`functionId=personinfoBusiness&body={"callCJH":"1","callNPS":"1","closeJX":"0","headTaskRefresh":"1","locationArea":"0_0_0_0","menuStaticSource":"0","menuTimeStamp":"1631586010000"}&uuid=55a9c688729bb118&client=android&clientVersion=10.4.6&st=1648455172760&sv=111`

![](https://bbs.kanxue.com/upload/attach/202203/884888_2D2XT65V83TXTJQ.jpg)

> 参数5：为长度 `0x106`

## 开始分析代码

![](https://bbs.kanxue.com/upload/attach/202203/884888_MRK5HS98Y6R5C3V.jpg)

> 从上面分析 `v27` 应该就是 `md5` 魔数 数组咯 大致为

> `v27[1]= 0x68449237;`

> `v27[2]= 0x7FCC3DA5;`

> `v27[3] = 0x88D90FBB;`

> `v27[4] = 0x5AE99AEE;`

> 继续往下分析

![](https://bbs.kanxue.com/upload/attach/202203/884888_PP58JB3579HS3JW.jpg)

这个区域是具体的实现的位置，`flag` 是之前传入的`0x1` 所以走的是`if` 判断 。这个`do-while` 大致是 从`v27`中取具体魔数 `s` 往下走 `v18 = 0x37` 猜测应该是不断从`v27`中取值 还原算法时`v27` 就是这个下面这个数组

```
{0x37, 0x92, 0x44, 0x68, 0xA5, 0x3D, 0xCC, 0x7F, 0xBB, 0x0F, 0xD9, 0x88, 0xEE, 0x9A,
0xE9, 0x5A};
```

![](https://bbs.kanxue.com/upload/attach/202203/884888_TVPEC52YUBCFME7.jpg)

> 继续S 往下走 `0x38` 就是传入固定字符串的第一个字符 `80306f4370b39fd5630ad0529f77adb6`

![](https://bbs.kanxue.com/upload/attach/202203/884888_3MDSXNAJVKXP99Z.jpg)

> `r0 = 0x66` 是从拼接的字符串中取出的第一个

> `r0=0x66` `r1=0x0` `r2=0x37` `r3=0x1` `r4=0x38`

> `R0 = R0 ^ R2 = 0x51`

> `R0 = R0 ^ R4 = 0x69`

![](https://bbs.kanxue.com/upload/attach/202203/884888_7CXV4Q5GHENQDV9.jpg)

![](https://bbs.kanxue.com/upload/attach/202203/884888_EUCBBM847KC6AUY.jpg)

> 按照汇编 分析猜测 后面就是 将前面异或的结果 加上`R2` `0x37` 然后在 `R2 = R2 ^ R0`

```
*(v5 - 1) = md5_init_key_index ^ *(_BYTE *)(key + v17);// EOR.W R2, R2, R1    STRB.W R2, [R10,#-1]
```

> 根据ida 上伪代码 结和汇编 大致可知 这部分是每执行一次`do while` 就修改一次`v5`的索引对应的值 一直循环到循环到`v5`数据长度，所以结合伪代码 用C还原 `sign` 的算法 如下：

```c
 void encryption(char *data) {

     uint32_t v1, v2, v3 = 0;

     unsigned char init_key[60] = {0x37, 0x92, 0x44, 0x68, 0xA5, 0x3D, 0xCC, 0x7F, 0xBB,    0x0F, 0xD9, 0x88, 0xEE, 0x9A,

                                   0xE9, 0x5A};

     char *sign = "80306f4370b39fd5630ad0529f77adb6";

     int data_length = strlen(data);

     unsigned char md5_init_key = 0;

     unsigned char sign_key = 0;

     do {

         v1 = v3 & 0xF;

         v2 = v3 & 7;

         v3 = v3 + 1;

         sign_key = sign[v2];

         md5_init_key = init_key[v1];

         md5_init_key = md5_init_key ^ ((md5_init_key ^ *data ^sign_key) + md5_init_key);

         *data = md5_init_key ^ sign_key;

         data++;

     } while (v3 != data_length);

 }
```

> 用python 来调用C代码 跟unidbg 黑盒调用结果比对如下:

![](https://bbs.kanxue.com/upload/attach/202203/884888_Y476G4U963YD4X2.jpg)

> 总结: 某东的`sign`有三个算法 这边只还原了其中比较简单的 `sv =111` 的这种 。还原这部分算法也花的时间挺多的，结合unidbg 来分析算法 真的方便。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-3-30 17:16

被那年没下雪编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)
