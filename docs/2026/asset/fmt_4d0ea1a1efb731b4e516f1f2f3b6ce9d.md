
# [原创]2023腾讯游戏安全大赛-安卓赛道决赛wp

## 说明

​ 本文档是进行完相关分析之后的总结回忆, 所以可能有的地方的花指令是被去除了再截图的, 函数名和数据结构被重命名了. 这决赛的混淆太恶心了,我无法做到将代码还原, 所以完全手撕汇编做的.

# flag获取分析

​ 这一部分和初赛是一样的, il2cpp那块并没有过多的加密, 还是和初赛一样从内存中将`libil2cpp.so` dump下来之后使用`il2cppdumper`得到对应的cs文件, 定位到`MouseController__CollectCoin`方法中, 找到偏移`0x4652AC`这里, 使用`gg`修改器将`CMP W0, #0x3E8`这条指令修改为`CMP W0, #0`之后随便吃一个金币得到flag.

![](upload/attach/202304/831526_GG583QED5X7ARUP.jpg)

# 反调试分析

​ 有了初赛的经验, 所以这次也是将`frida_server`的二进制文件patch掉`re.frida.server`字符串, 并且使用`./fs -l 0.0.0.0:2394`命令切换监听端口, 为了保证万无一失, 将工作目录从`tmp`文件夹移到别的地方取, 这样一来就能够成功过掉这一大类的检测.

​ 但是也是一样的, 除了文件检测和端口检测, `libsec2023.so`中还有针对代码段的`crc`检测, 使用`findcrypt`插件从`so`文件的特征中也能够发现. 意外的这里还看到了`aes`特征, 后面的算法也有可能使用到了`aes`算法, 只是说可能.

![](upload/attach/202304/831526_SRAXYU8P6YESP7Z.jpg)

​ 也是同初赛一样, 对于`crc`检测一定是不间断的读取代码段数据, 所以只需要对其代码段下一个硬件读写断点就能够定位到检测的关键点. 我依旧是使用`rwProcMem`项目中的内核硬件断点程序进行操作, 在`libsec2023.so`的内存区域中随意选择一个地址, 如下图所示成功定位到读取的关键点.

![](upload/attach/202304/831526_N2JSSJP6WWJ8W4A.jpg)

​ 此时`libsec2023.so`的内存基址是`0x7cecf00000`. 可以定位到读取内存段的代码在`libsec2023.so + 0x7d934`处, 这里因为混淆方式的改变, 所以`f5`也不能和初赛一样将其还原出来, 因此只能查看汇编代码.

![](upload/attach/202304/831526_J6736GWK3CX8JTN.jpg)

​ 这里因为被混淆了, 并且这里只是做了读取内存的操作, 所以找不到合适的`patch`点, 此时查看`lr`寄存器, 可以得到调用读取函数的返回的上一层地址是`libsec2023.so + 0x7B300`.

![](upload/attach/202304/831526_BFCXYGU5FVPAYAA.jpg)

​ 这里能够看到`0x7B2FC`的`blr x8`就是调用点, 读取完代码段的校验和之后返回值会存储在`x0`寄存器中, 因此可以按照此为线索往下面拉, 看到`0x7B320`处有一个`cmp w0, w8`. 之后一个`b`跳转到`0x7AF00`处

![](upload/attach/202304/831526_FVCVDPRU7J9TMY6.jpg)

​ 在`0x7AF04`这里会依据前面的`cmp`语句的结果对`w8`进行赋值, 而`w8`的值就会影响到之后的分支, 所以这就能够找到`patch`的关键点了, 用硬件断点工具下执行断点查看寄存器的数据可以发现正常流程中`w0`和`w8`的值应当是相等的, 所以要越过`crc`检测则应该将`cmp w0, w8`语句改成`mov w0, w8`, 同时将`csel w8, w9, w8, eq`改成`mov w8, w9`. 实践中发现后面那句不改也可以, `eq`默认成立.

​ 所以在`frida`的脚本中初始化中加入以下语句即可彻底过掉检测.

```js
function PatchCode(addr, value){
Memory.protect(addr, 4, 'rwx');
addr.writeInt(value);
}
PatchCode(libsec2023.base.add(0x7B320), 0x2a0803e0);//过crc32
```

​ 至此反调试就分析完毕, `frida`能够正常使用了.

# 算法分析

​ 相比于初赛, 决赛的混淆程度更加严重了, `ida`的伪代码功能彻底报废, 程序中函数的跳转和程序内分支语句全部变成了寄存器间接跳转, 对于函数跳转尚可使用`unidbg`这种模拟执行的方式还原, 但是条件分支具有动态性, 所以很难还原回去. 所以我选择了`frida-trace`日志静态分析 + `frida hook`动态的方式进行分析.

## 算法执行流程

​ 同样先大概看了一下流程, 同初赛一样, 刚开始会从`libil2cpp.so`中接收输入数字, 然后传入后续的算法执行, 然后生成随机的`token`.

![](upload/attach/202304/831526_RJJUJY6EKXBJW76.jpg)

​ 通过`frida hook`可以得知这里的跳转会跳转到`libsec2023.so + 0x70E74`.

![](upload/attach/202304/831526_MEQ2QBFBASUHP8Y.jpg)

​ 上图的`sub_94368`便是加密算法的主体部分, `v5`跳转则会跳回`libil2cpp.so + 0x465994`, 参数`v3`是返回的`int64`值.

![](upload/attach/202304/831526_EB83F7YYTUR4K3F.jpg)

​ 返回的值如果满足低`32`位等于`token`, 高`32`为`0`, 则开启`mod`. 不同于初赛, 这里最后并没有其他校验, 所以算法的全部实现都在`libsec2023.so`中.

## 离散对数

​ 首先假设输入的数字是`666666`, 对应的十六进制表达是`0xa2c2a`. 先进入`sub_94368`函数, 从这里开始的所有代码都被严重混淆了, 截图为我部分修复之后的伪代码. **其中要满足`strcmp`结果一致之后才会进入离散对数算法之后的后续逻辑**

![](upload/attach/202304/831526_VWWB6627U8N784J.jpg)

​ 在这个函数中主要是初始化了两个大数数组, 之后进入`sub_9e93c`中进行某种运算, 其中第一个参数是输入的数字和`0xffff00ffffff`进行按位与运算的结果. 对于输入数字数字`666666`, 其计算的结果是`1db29b949927d377f0270e1161964bb0b9fc004f7125b6956d057e09a7c2fadf77df04bbcae93aa0000`字符串, 之后将`s2`的字符串与前面初始化的某个数组使用`strcmp`函数进行比较, 经过`hook`获取发现该数组是一个恒定不变的字符串`25f6b048b4f32e3ce9175bb64930f65101a706ae74988a4ec87b4d5ec7feb9223ab782bcf1ec9d7fee750`. 刚开始看到这个字符串想到了哈希, 但是具体是什么还需要进入`sub_9e93c`函数查看.

![](upload/attach/202304/831526_55CX3VZC8CY3DQP.jpg)

​ `sub_9e93c`函数首先会进行一系列初始化, 然后进入`sub_9F0A8`函数和`sub_9f704`中, 使用`frida hook`也能够发现输入的数字是在最后执行完`sub_9f704`出现了字符串. 理所应当先进入`sub_9f704`看看这个字符串是如何产生的, 查看`frida-trace`的`log`能够发现其在某处开始会每`8`个一组产生`0`, 一直产出了`168`个`0`之后才生成想看到的字符串.

```asm
......
9f934 mov x27, x1 ; x27 = 0x81d8ae82 --> 0x7ce2ef7850
9f938 adrp x3, #0x7ced00b000 ; x3 = 0x3762f4b5 --> 0x7ced00b000
9f93c add x1, x1, x8 ;
9f940 ldr x8, [sp, #8] ; x8 = 0x0 --> 0x7ce2ef7470
9f944 add x0, sp, #0x20 ; x0 = 0x767ac01f --> 0x7ce2ef73e0
9f948 mov x2, #-1 ; x2 = 0x471527a8 --> 0xffffffffffffffff
9f94c add x3, x3, #0x64b ; x3 = 0x7ced00b000 --> 0x7ced00b64b (%.08x)
9f950 ldr w4, [x8, x9, lsl #2] ; x4 = 0x22cd9db --> 0x0 (null)
9f954 ldr w8, [sp, #0x24] ; x8 = 0x7ce2ef7470 --> 0xeecf7326
9f958 adrp x9, #0x7ced030000 ; x9 = 0x1f --> 0x7ced030000 (U!3})
9f95c ldr x9, [x9, #0xa68] ; x9 = 0x7ced030000 --> 0x7cb537824c
9f960 str w8, [sp, #0x20] ; str = 0xeecf7326
9f964 mov w8, #0x89f0 ; x8 = 0xeecf7326 --> 0x89f0
9f968 movk w8, #0x37c2, lsl #16 ; x8 = 0x89f0 --> 0x37c289f0
9f96c add x8, x9, x8 ; x8 = 0x37c289f0 --> 0x7cecfa0c3c
9f970 blr x8 ; x0 = 0x7ce2ef73e0 --> 0x8 x1 = 0x7ce2ef7850 --> 0x7ce2ef6b83 x2 = 0xffffffffffffffff --> 0x9 x3 = 0x7ced00b64b --> 0x7ce2ef7260 x4 = 0x0 --> 0x7ce2ef6b84 x5 = 0x50 --> 0x7ce2ef7858 x6 = 0x110 --> 0x30 x7 = 0x108 --> 0x30 x8 = 0x7cecfa0c3c --> 0xd55c7b4c8b012ca8 x9 = 0x7cb537824c --> 0xd55c7b4c8b012ca8 x10 = 0xcd53af93 --> 0x0 x11 = 0x7cecfa092c --> 0x2 x12 = 0x48 --> 0x0 x13 = 0x400 --> 0x7ffffff7 x14 = 0xc8 --> 0xeecf7326 x15 = 0xf0fc4d01 --> 0x0 x16 = 0xd1993aaf --> 0x7dd3e069a0 x17 = 0xb952cbee --> 0x7dd3d4bb94
9f974 ldr w8, [sp, #0x28] ; x8 = 0xd55c7b4c8b012ca8 --> 0x0 (null)
9f978 ldr w9, [sp, #0x2c] ; x9 = 0xd55c7b4c8b012ca8 --> 0x1f
9f97c ldr w10, [sp, #0x24] ; x10 = 0x0 --> 0xeecf7326
9f980 mov w11, #0x8ff8 ; x11 = 0x2 --> 0x8ff8
9f984 mov w4, #0xd9db ; x4 = 0x7ce2ef6b84 --> 0xd9db
9f988 mov w3, #0xf4b5 ; x3 = 0x7ce2ef7260 --> 0xf4b5
9f98c mov w2, #0x27a8 ; x2 = 0x9 --> 0x27a8
9f990 mov w0, #0xc01f ; x0 = 0x8 --> 0xc01f
9f994 mov w17, #0xcbee ; x17 = 0x7dd3d4bb94 --> 0xcbee
9f998 mov w16, #0x3aaf ; x16 = 0x7dd3e069a0 --> 0x3aaf
9f99c mov w15, #0x4d01 ; x15 = 0x0 --> 0x4d01
9f9a0 mov x1, x27 ; x1 = 0x7ce2ef6b83 --> 0x7ce2ef7850 (00000000)
;这里能看到生成了一串0, 这个0就是上面的br跳转得到的
9f9a4 mov w27, #0xae82 ; x27 = 0x7ce2ef7850 --> 0xae82
......
```

​ 而上面的那个`br`跳转则是进入了`sub_9FC3C`这个函数

![](upload/attach/202304/831526_E47WSUA9FJSYKH6.jpg)

​ 进去查看发现这个函数实现的功能是将某个数字按照十六进制生成字符串, 不足`8`位补齐, 所以前面的`00000000`就是这么产生的, 经过`hook`发现最终生成的一长串数字也是在这里一组一组生成最终拼接而成. 经过详细的比对, 最终确定`sub_9f704`函数只是一个类似于显示层的函数, 计算出的数字是在`sub_9F0A8`中生成出来的.

​ `sub_9F0A8`函数存在有控制流混淆, 使用`OBPO`插件去除混淆后能够得到这样伪代码, 其中注释和函数名称是我经过大量`hook`与对应`trace log`比对后得到其对应功能后命名的.

![](upload/attach/202304/831526_97ZDHVG5S4AH4W6.jpg)

​ 这个函数最终实现的功能是对于输入数字`x`, 输出结果`y = pow(x, 17) mod key`, 其中`key`的值是已知的, 也就是前面初始化的某个数组之一, 经过`hook`后得到`key`为`0x028a831a5bf4b902e95318e50c2075259f91094d08d84409e1b76eadfa0865d1278acc90fa7c6cf6acb375`. 这个算法是离散对数问题, `y`为已知的`25f6b048b4f32e3ce9175bb64930f65101a706ae74988a4ec87b4d5ec7feb9223ab782bcf1ec9d7fee750`, 其逆算法也就是求得输入数字`x`. 其中`x`的范围已知是和`0xffff00ffffff`按位与之后的结果, 那么最简单的逆算法就是爆破枚举.

```c++
#include <iostream>
#include <gmpxx.h>
//这里使用了gmp库来计算大整数, 会暴力输出所有满足的解
//这里得到的解是原始输入int64 & 0xffff00ffffff之后应当满足的部分
void GetAns() {
mpz_class z("0x28a831a5bf4b902e95318e50c2075259f91094d08d84409e1b76eadfa0865d1278acc90fa7c6cf6acb375", 0);
mpz_class y("0x25f6b048b4f32e3ce9175bb64930f65101a706ae74988a4ec87b4d5ec7feb9223ab782bcf1ec9d7fee750", 0);
mpz_class j, x;
mpz_ui_pow_ui(j.get_mpz_t(), 2, 32); // j = 2^32
for (int i = 0x0; i <= 0xffff; i++) {
x = i;
x *= j; // x = i * 2^32
for (int k = 0; k <= 0xffffff; k++) {
x += k;
mpz_class result;
mpz_powm_ui(result.get_mpz_t(), x.get_mpz_t(), 17, z.get_mpz_t());
if (result == y) {
std::cout << x << std::endl;
}
}
}
return 0;
}
```

## a64

​ 在满足前面的`strcmp`条件之后, 程序执行流会进入到`sub_99EF4`中.

![](upload/attach/202304/831526_5TANUG4M4BYRZHC.jpg)

​ 在上图中能够看到修复后伪代码, 主要是会初始化一些参数. 使用`frida-trace`对其进行`trace`能够看到一些奇怪的字符串
