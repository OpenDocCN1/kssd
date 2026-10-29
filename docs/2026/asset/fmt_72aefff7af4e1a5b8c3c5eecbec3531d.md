
```asm
3b9e4 csel x10, xzr, x9, lo ; x10 = 0xa2c5a --> 0x0 (null)
3b9e8 adrp x9, #0x7d03375000 ; x9 = 0x28 --> 0x7d03375000 (�fC���>)
3b9ec add x9, x9, #0xc40 ; x9 = 0x7d03375000 --> 0x7d03375c40 q3�|)
3b9f0 ldr x10, [x9, x10] ; x10 = 0x0 --> 0x7c8f337108
3b9f4 mov w11, #0x78fc ; x11 = 0xb400007bb22885a0 --> 0x78fc
3b9f8 movk w11, #0x7400, lsl #16 ; x11 = 0x78fc --> 0x740078fc
3b9fc add x10, x10, x11 ; x10 = 0x7c8f337108 --> 0x7d0333ea04 (j)
3ba00 br x10 ;必定跳转
3ba04 mov w10, #3 ; x10 = 0x7d0333ea04 --> 0x3
3ba08 mov w11, #0x18 ; x11 = 0x740078fc --> 0x18
3ba0c cmp x10, #0 ; x10 = 0x3
3ba10 mov w12, #0x10 ; x12 = 0x0 --> 0x10
3ba14 csel x11, x12, x11, ge ; x11 = 0x18 --> 0x10
3ba18 ldr x11, [x9, x11] ; x11 = 0x10 --> 0x7c8f337138
3ba1c mov w12, #0x78fc ; x12 = 0x10 --> 0x78fc
3ba20 movk w12, #0x7400, lsl #16 ; x12 = 0x78fc --> 0x740078fc
3ba24 str wzr, [sp, #0xc] ; str = 0
3ba28 add x12, x11, x12 ; x12 = 0x740078fc --> 0x7d0333ea34 (xh��3)
3ba2c mov w11, #0x18 ; x11 = 0x7c8f337138 --> 0x18
3ba30 br x12 ;必定跳转
;我称前面是第一个代码块
3ba34 ldr w12, [x0, x8, lsl #2] ; x12 = 0x7d0333ea34 --> 0x0 (null)取参数值, 也就是高32位, 即0
3ba38 add x13, sp, #0xc ; x13 = 0x6430bee2 --> 0x7cfe68653c
3ba3c mov w14, #0x18 ; x14 = 0x14da99d881a188 --> 0x18 从高到低选择字节
3ba40 lsr w12, w12, w11 ; 取字节
3ba44 eor w12, w12, w10 ; x12 = 0x0 --> 0x3 和字节序号做异或
3ba48 strb w12, [x13, x10] ; 存到[sp, #0xc]的位置去, 此时顺序还是原来
...
;我称这为第二个代码块, 主要做的事情是取666666(64位)的高32位, 和字节序号异或存回去, 省略掉的部分是其他字节相同操作
3ba74 ldrb w11, [sp, #0xf] ; x11 = 0xfffffff8 --> 0x3 第4字节
3ba78 ldrb w13, [sp, #0xd] ; x13 = 0x740078fc --> 0x1 第2字节
3ba7c mov w12, #0x86 ; x12 = 0x7d0333ea74 --> 0x86
3ba80 ldrb w14, [sp, #0xe] ; x14 = 0x18 --> 0x2 第3字节
3ba84 eor w11, w11, w12 ; x11 = 0x3 --> 0x85 byte 4和0x86异或
3ba88 mov w12, #0xd3 ; x12 = 0x86 --> 0xd3
3ba8c eor w12, w13, w12 ; x12 = 0xd3 --> 0xd2 0xd3和byte 2异或
3ba90 ldrb w13, [sp, #0xc] ; x13 = 0x1 --> 0x0 (null) 第1字节
3ba94 strb w11, [sp, #0xf] ; 也就是最高字节和0x86异或存回去 byte[3] ^= 0x86
3ba98 sub w11, w14, #0x5e ; x11 = 0x85 --> 0xffffffa4
3ba9c strb w11, [sp, #0xe] ; byte[2] -= 0x5e
3baa0 mov w11, #3 ; x11 = 0xffffffa4 --> 0x3
3baa4 sub w13, w13, #0x1c ; x13 = 0x0 --> 0xffffffe4
3baa8 strb w12, [sp, #0xd] ; byte[1] ^= 0xd3
3baac mov w12, #8 ; x12 = 0xd2 --> 0x8
3bab0 strb w13, [sp, #0xc] ; byte[0] -= 0x1c 最低字节-0x1c存回去
3bab4 mov w13, #0x20 ; x13 = 0xffffffe4 --> 0x20
3bab8 cmp x11, #0 ; x11 = 0x3
3babc str wzr, [x0, x8, lsl #2] ; str = 0
3bac0 csel x12, x13, x12, ge ; x12 = 0x8 --> 0x20
3bac4 ldr x12, [x9, x12] ; x12 = 0x20 --> 0x7c8f3371e4
3bac8 mov w13, #0x78fc ; x13 = 0x20 --> 0x78fc
3bacc movk w13, #0x7400, lsl #16 ; x13 = 0x78fc --> 0x740078fc
3bad0 mov w10, wzr ; x10 = 0xffffffffffffffff --> 0x0 (null)
3bad4 add x13, x12, x13 ; x13 = 0x740078fc --> 0x7d0333eae0 (�3)
3bad8 mov w12, #0x18 ; x12 = 0x7c8f3371e4 --> 0x18
3badc br x13 ;
;我称这为第三个代码块, 主要做的事情是每个字节分别进行对应加减法
3bae0 add x13, sp, #0xc ; x13 = 0x7d0333eae0 --> 0x7cfe68653c (�Ҥ�)
3bae4 ldrb w14, [x13, x11] ; x14 = 0x2 --> 0x85 取最高字节
3bae8 sub w14, w14, w12 ; x14 = 0x85 --> 0x6d
3baec strb w14, [x13, x11] ;最高字节-0x18存回去 byte[3] -= 0x18
3baf0 and w14, w14, #0xff ;
3baf4 lsl w14, w14, w12 ; x14 = 0x6d --> 0x6d000000
3baf8 sub x11, x11, #1 ; x11 = 0x3 --> 0x2
3bafc mov w13, #8 ; x13 = 0x7cfe68653c --> 0x8
3bb00 add w10, w14, w10 ; x10 = 0x0 --> 0x6d000000
3bb04 mov w14, #0x20 ; x14 = 0x6d000000 --> 0x20
3bb08 cmp x11, #0 ; x11 = 0x2
3bb0c str w10, [x0, x8, lsl #2] ; str = 0x6d000000 \*result = byte[3]<<24
3bb10 csel x13, x14, x13, ge ; x13 = 0x8 --> 0x20
3bb14 ldr x13, [x9, x13] ; x13 = 0x20 --> 0x7c8f3371e4
3bb18 mov w14, #0x78fc ; x14 = 0x20 --> 0x78fc
3bb1c movk w14, #0x7400, lsl #16 ; x14 = 0x78fc --> 0x740078fc
3bb20 sub w12, w12, #8 ; x12 = 0x18 --> 0x10
3bb24 add x13, x13, x14 ; x13 = 0x7c8f3371e4 --> 0x7d0333eae0 (�3)
3bb28 br x13 ;循环节
...
;我称这为第四个代码块, 主要做的事情是对每个字节分别进行byte[i] -= 8\*i;
;在这之后代码会取低32位进行相同的操作
```

分析完以上的汇编代码后可以还原对应的算法

```c++
void sub\_3b9d4(unsigned char\* byte)
{
//输入参数指针指向的区域是00 00 00 00 2a 2c 0a 00
//进行完修改之后数据变为e4 ca 94 6d 0e f6 9a 6d
for(int i=3;i>=0;i--)
byte[i] ^= i;
byte[0] -= 0x1c;
byte[1] ^= 0xd3;
byte[2] -= 0x5e;
byte[3] ^= 0x86;
for(int i=3;i>=0;i--)
byte[i] -= 8\*i;
byte += 4;
for(int i=3;i>=0;i--)
byte[i] ^= i;
byte[0] -= 0x1c;
byte[1] ^= 0xd3;
byte[2] -= 0x5e;
byte[3] ^= 0x86;
for(int i=3;i>=0;i--)
byte[i] -= 8\*i;
}
```

使用frida hook来验证结果是否正确, 确实无误

![](upload/attach/202304/831526\_VRZ4NB8T68J6FU9.jpg)

之后来编写对应的逆算法为

```c++
void sub\_3b9d4\_reverse(unsigned char\* byte)
{
for (int i = 3; i >= 0; i--)
byte[i] += 8 \* i;
byte[0] += 0x1c;
byte[1] ^= 0xd3;
byte[2] += 0x5e;
byte[3] ^= 0x86;
for (int i = 3; i >= 0; i--)
byte[i] ^= i;
byte += 4;
for (int i = 3; i >= 0; i--)
byte[i] += 8 \* i;
byte[0] += 0x1c;
byte[1] ^= 0xd3;
byte[2] += 0x5e;
byte[3] ^= 0x86;
for (int i = 3; i >= 0; i--)
byte[i] ^= i;
}
```

### libsec2023.so ==> sub\_3A924

在这个函数中有很多复杂且奇怪的函数, 并且也出现了函数指针+1000多的数字, 因此可以猜想这里主要做的事情是准备java层环境, 之后调用java函数, 将伪代码中JNIEnv\*参数类型标注出来. 刷新伪代码后能够很明显看到先new了一个java层的bytearray, 然后findclass, GetStaticMethodID得到函数, 最后在sub\_3ABBC中调用这个函数.

![](upload/attach/202304/831526\_79JJPP5SJ5XF6ZV.jpg)

这里我首先对findclass进行hook, 发现类名是空的, 静态函数名称能得到是encrypt, 基本可以确定无误. 用jadx打开dex发现并没有这个函数, 因此怀疑是走RegisterNatives动态注册的. 使用frida对RegisterNatives进行hook, 发现了一条记录

```c++
[RegisterNatives] java\_class: com.tencent.games.sec2023.Sec2023Application name: initialize sig: ()I fnPtr: 0x7cfe0d5abc fnOffset: 0x7cfe0d5abc libsec2023.so!0xfabc callee: 0x7cfe0d5ed0 libsec2023.so!JNI\_OnLoad+0x3b0
```

但是经过排查很明显不是上面的encrypt函数. 思考一下, 除了动态注册以外, 有可能是动态加载了dex, 于是使用frida-dexdump将内存中的dex dump下来. 果然在某个dex中发现了该函数.

![](upload/attach/202304/831526\_C8YED6W979TA64Z.jpg)

但是其控制流也被混淆了, 好在函数本身并不复杂, 因此可以将伪代码直接复制新建一个java工程, 测试功能完好之后对其进行调试分析, 将函数逻辑梳理一下得到其逻辑代码如下

```c++
char\* encrypt(char\* var0)
{
//输入109,-108,-54,-28
//输出-6,23,-40,16
//输入109,-102,-10,14
//输出46,23,-52,119
int var5 = 0;
int var6 = 0;
var6 = (var0[3] & 0xff) + (var0[0] << 24 & -0x1000000) + (var0[1] << 16 & 0xff0000) + (var0[2] << 8 & 0xff00);
var5 = ((unsigned int)var6 >> 7) | (var6 & 0x7f) << 25;
char \*var1 = new char[4];
var1[0] = (unsigned char)(var5 >> 24 & 255);
var1[1] = (unsigned char)(var5 >> 16 & 255);
var1[2] = (unsigned char)(var5 >> 8 & 255);
var1[3] = (unsigned char)(var5 & 255);
unsigned char data[] = { 50, -51, -1, -104, 25, -78, 124, -102 };
for (int i = 0; i < 4; i++)
{
var1[i] ^= data[i % 8];
var1[i] = (unsigned char)(var1[i] + i);
}
return var1;
}
```

经过分析也能得出对应的逆算法为如下

```c++
char\* decrypt(char\* var0) {
unsigned char data[] = { 50, -51, -1, -104, 25, -78, 124, -102 };
for (int i = 0; i < 4; i++) {
var0[i] = (unsigned char)(var0[i] - i);
var0[i] ^= data[i % 8];
}
int var5 = (var0[3] & 0xff) + (var0[2] << 8 & 0xff00) + (var0[1] << 16 & 0xff0000) + (var0[0] << 24 & -0x1000000);
var5 = ((unsigned int)var5 << 7) | ((unsigned int)var5 >> 25);
char\* var1 = new char[4];
var1[0] = (unsigned char)(var5 >> 24 & 255);
var1[1] = (unsigned char)(var5 >> 16 & 255);
var1[2] = (unsigned char)(var5 >> 8 & 255);
var1[3] = (unsigned char)(var5 & 255);
return var1;
}
```

使用frida hook得到的结果与上面算法计算得到的结果一致

![](upload/attach/202304/831526\_2WYMZ6VQR62BDKX.jpg)

梳理完这个函数那么算法在libsec2023.so的部分就已经结束了, 按照动态流程分析执行完sub\_31164后会跳回到libil2cpp.so + 0x465AB4中, 从java层这个encrypt返回的结果与0x465AB4所拦截的结果是一致的. 所以libsec2023.so部分到此结束.

### libil2cpp.so ==> sub\_465AB4

这个函数是从libsec2023.so返回所进入的函数, 首先根据伪代码观察其流程, 主要做的事情是初始化了一些数组和对象, 将上一个encrypt的结果传入sub\_46AD44函数中进行加密, 然后在一个do\_while循环中进行第二次加密. 最后如果v28等于0并且v27与token的值相等就开启mod成功, 这里手动patch判断语句可以实现作弊功能, 所以整体思路是对的.

![](upload/attach/202304/831526\_GMSHCWNK9UY485B.jpg)

### libil2cpp.so ==> sub\_46AD44

首先来分析第一个sub\_46AD44函数, 这个函数非常复杂, 主要是重要的字段名信息还有函数名信息都被混淆掉了.

![](upload/attach/202304/831526\_3WQJW93M9PHR2QU.jpg)

整个类中做了什么事情, 首先会初始化一些参数, 最重要的是某个迭代值(类内偏移0x2c), action词典(类内偏移0x38), 码表(类内偏移0x10). 该类在初始化的时候会注册0x1-0x16的值与某个操作函数对应关系在action词典中, 然后在一个大循环中随着迭代值的变化在码表中选择不同的action, 类似于状态机的实现. 下面这个sub\_46AD44函数内的循环即是主要逻辑.

![](upload/attach/202304/831526\_SFR9EFYEGHJ4E6W.jpg)
