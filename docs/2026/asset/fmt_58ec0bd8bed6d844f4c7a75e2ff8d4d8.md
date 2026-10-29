
```asm
;继续跟进
978c4 mov x1, x0 ; x1 = 0x91844490 --> 0xb400007bef75a300
978c8 add x0, sp, #0xb8 ; x0 = 0xb400007bef75a300 --> 0x7cdb8682c8
978cc add x8, x8, x27 ; x8 = 0x7c7940f228 --> 0x7cecf5ba78 (O{C)
978d0 blr x8 ; x0 = 0x7cdb8682c8 --> 0x1 x1 = 0xb400007bef75a300 --> 0x83578390830c8340 x2 = 0x284 --> 0x83598390833d83f4 x3 = 0x8614eee7 --> 0xb400007bef75a500 x4 = 0x7a5cace1 --> 0xb400007bf780a79f x5 = 0x6b8d117a --> 0xb400007bef75a584 x6 = 0x5c68331f --> 0x83258397836e83df x7 = 0x3a59490c --> 0x8326832b838b832b x8 = 0x7cecf5ba78 --> 0x29f x9 = 0x7cecf98898 --> 0xd4fcb35983838361 x10 = 0xcb036d0 --> 0xd4fcea78d4fca104 x11 = 0x10e05d66 --> 0xd4fcea7883838383 x12 = 0xf244e8a7 --> 0x830c834083598390 x13 = 0xeb3e178a --> 0x833d83f483578390 x16 = 0xa5356f4c --> 0x7ced026c80 x17 = 0xa4ff4077 --> 0x7dd3d8ec40
```

在上面这段汇编中0xb400007bef75a300是之前定义的结构体指针, `x2`参数很明显是某个比文件小的长度. 进入`sub_5AA78`看看.

![](upload/attach/202304/831526_SQXCRSVFMZN69ZR.jpg)

发现该函数的主要功能是对文件的某一段内容复制到另一块内存区域当中, 对应到上面的调用, 那含义很明显就是从`a64.dat`文件的0x1b起始, 长度为0x284的部分. 继续跟进看看其是如何使用这个区域的.

```asm
;将前面那块地址存到[sp, #0x128]栈上
979e8 str x0, [sp, #0x128] ; str = 0xb400007bef75a300
......
9745c ldr w8, [sp, #0x144] ; x8 = 0x91844490 --> 0x0 (null)
97460 ldr x9, [sp, #0x128] ; x9 = 0x7cecf9845c --> 0xb400007bef75a300
97464 mov w27, #0x23 ; x27 = 0x2d0 --> 0x23
97468 ldrb w10, [x9, x8] ; x10 = 0xcb036d0 --> 0x4
9746c eor x10, x10, x27 ; x10 = 0x4 --> 0x27
97470 adrp x27, #0x7ced00b000 ; x27 = 0x23 --> 0x7ced00b000
97474 add x27, x27, #0x536 ; x27 = 0x7ced00b000 --> 0x7ced00b536
97478 ldrb w10, [x27, x10] ; x10 = 0x27 --> 0x23
9747c strb w10, [x9, x8] ;
```

可以发现前面内存复制之后, 会马上使用到这块区域, 这是一个很明显的加解密操作, 其实现的操作是取第一个字节0x4和0x23异或得到结果0x27, 然后从原来的字节被替换为`MEM[0x10A536 + 0x27]`的值. 那么这个0x10A536偏移的区域很有可能是映射码表之类的东西.

继续查看后面的汇编可以发现果然在后面的`x8`会逐渐递增, 并重复这段操作, 也就是循环对每个字节进行这样的操作. 那么可以看看这个码表解密之后是什么样的. 这里使用hook会造成程序卡死, 那么使用以下的python脚本静态解密一下看看是什么东西.

```python
import io
a64Data = io.open("a64.dat", 'rb').read()
Mem_10A536Data = io.open("Mem_10A536.bin", 'rb').read()
data = bytearray(0x284)
for i in range(0x284):
    data[i] = Mem_10A536Data[a64Data[i+0x1b] ^ 0x23]
io.open("out.bin", "wb").write(data)
```

![](upload/attach/202304/831526_QG4DB7JNTV4NZQ3.jpg)

可以看到这块区域被解密出来了, 其中也能够看到`vm_main`字符串, 所以应该是没有错误的.

时间和精力原因后面的没有分析完, 前面findcrypt插件发现了aes的特征, 所以也有可能后面用到了aes算法, 看字符串带vm字样也不排除使用了vm技术. 看汇编是真的累

# 总结

到此为止成功获取了flag, 过掉了所有反调试使得`frida`能够正常工作, 之后静态分析加动态调试还原了离散对数算法, 并且还给出了其逆算法.看汇编还是太累了, 说实在的这么高强度的混淆和前面的vm混打, 这次比赛难度真不小.

# 说明

本文档是进行完相关分析之后的总结回忆, 所以可能有的地方的花指令是被去除了再截图的, 函数名和数据结构被重命名了. 这决赛的混淆太恶心了,我无法做到将代码还原, 所以完全手撕汇编做的.

# flag获取分析

这一部分和初赛是一样的, `il2cpp`那块并没有过多的加密, 还是和初赛一样从内存中将`libil2cpp.so` dump下来之后使用`il2cppdumper`得到对应的cs文件, 定位到`MouseController__CollectCoin`方法中, 找到偏移0x4652AC这里, 使用`gg`修改器将`CMP W0, #0x3E8`这条指令修改为`CMP W0, #0`之后随便吃一个金币得到flag.

![](https://bbs.kanxue.com/upload/attach/202304/831526_GG583QED5X7ARUP.jpg)

# 反调试分析

有了初赛的经验, 所以这次也是将`frida_server`的二进制文件patch掉`re.frida.server`字符串, 并且使用`./fs -l 0.0.0.0:2394`命令切换监听端口, 为了保证万无一失, 将工作目录从`tmp`文件夹移到别的地方取, 这样一来就能够成功过掉这一大类的检测.

但是也是一样的, 除了文件检测和端口检测, `libsec2023.so`中还有针对代码段的`crc`检测, 使用`findcrypt`插件从`so`文件的特征中也能够发现. 意外的这里还看到了`aes`特征, 后面的算法也有可能使用到了`aes`算法, 只是说可能.

![](https://bbs.kanxue.com/upload/attach/202304/831526_SRAXYU8P6YESP7Z.jpg)

也是同初赛一样, 对于`crc`检测一定是不间断的读取代码段数据, 所以只需要对其代码段下一个硬件读写断点就能够定位到检测的关键点. 我依旧是使用`rwProcMem`项目中的内核硬件断点程序进行操作, 在`libsec2023.so`的内存区域中随意选择一个地址, 如下图所示成功定位到读取的关键点.

![](https://bbs.kanxue.com/upload/attach/202304/831526_N2JSSJP6WWJ8W4A.jpg)

此时`libsec2023.so`的内存基址是0x7cecf00000. 可以定位到读取内存段的代码在`libsec2023.so + 0x7d934`处, 这里因为混淆方式的改变, 所以`f5`也不能和初赛一样将其还原出来, 因此只能查看汇编代码.

![](https://bbs.kanxue.com/upload/attach/202304/831526_J6736GWK3CX8JTN.jpg)

这里因为被混淆了, 并且这里只是做了读取内存的操作, 所以找不到合适的`patch`点, 此时查看`lr`寄存器, 可以得到调用读取函数的返回的上一层地址是`libsec2023.so + 0x7B300`.

![](https://bbs.kanxue.com/upload/attach/202304/831526_BFCXYGU5FVPAYAA.jpg)

这里能够看到0x7B2FC的`blr x8`就是调用点, 读取完代码段的校验和之后返回值会存储在`x0`寄存器中, 因此可以按照此为线索往下面拉, 看到0x7B320处有一个`cmp w0, w8`. 之后一个`b`跳转到0x7AF00处.

![](https://bbs.kanxue.com/upload/attach/202304/831526_FVCVDPRU7J9TMY6.jpg)

在0x7AF04这里会依据前面的`cmp`语句的结果对`w8`进行赋值, 而`w8`的值就会影响到之后的分支, 所以这就能够找到`patch`的关键点了, 用硬件断点工具下执行断点查看寄存器的数据可以发现正常流程中`w0`和`w8`的值应当是相等的, 所以要越过`crc`检测则应该将`cmp w0, w8`语句改成`mov w0, w8`, 同时将`csel w8, w9, w8, eq`改成`mov w8, w9`. 实践中发现后面那句不改也可以, `eq`默认成立.

所以在`frida`的脚本中初始化中加入以下语句即可彻底过掉检测.

```javascript
function PatchCode(addr, value){
    Memory.protect(addr, 4, 'rwx');
    addr.writeInt(value);
}

PatchCode(libsec2023.base.add(0x7B320), 0x2a0803e0); // 过crc32
```

至此反调试就分析完毕, `frida`能够正常使用了.

# 算法分析

相比于初赛, 决赛的混淆程度更加严重了, `ida`的伪代码功能彻底报废, 程序中函数的跳转和程序内分支语句全部变成了寄存器间接跳转, 对于函数跳转尚可使用`unidbg`这种模拟执行的方式还原, 但是条件分支具有动态性, 所以很难还原回去. 所以我选择了`frida-trace`日志静态分析 + `frida` hook动态的方式进行分析.

## 算法执行流程

同样先大概看了一下流程, 同初赛一样, 刚开始会从`libil2cpp.so`中接收输入数字, 然后传入后续的算法执行, 然后生成随机的`token`.

![](https://bbs.kanxue.com/upload/attach/202304/831526_RJJUJY6EKXBJW76.jpg)

通过`frida` hook可以得知这里的跳转会跳转到`libsec2023.so + 0x70E74`.

![](https://bbs.kanxue.com/upload/attach/202304/831526_MEQ2QBFBASUHP8Y.jpg)

上图的`sub_94368`便是加密算法的主体部分, `v5`跳转则会跳回`libil2cpp.so + 0x465994`, 参数`v3`是返回的`int64`值.

![](https://bbs.kanxue.com/upload/attach/202304/831526_EB83F7YYTUR4K3F.jpg)

返回的值如果满足低32位等于`token`, 高32为0, 则开启`mod`. 不同于初赛, 这里最后并没有其他校验, 所以算法的全部实现都在`libsec2023.so`中.

## 离散对数

首先假设输入的数字是666666, 对应的十六进制表达是0xa2c2a. 先进入`sub_94368`函数, 从这里开始的所有代码都被严重混淆了, 截图为我部分修复之后的伪代码. **其中要满足`strcmp`结果一致之后才会进入离散对数算法之后的后续逻辑**

![](https://bbs.kanxue.com/upload/attach/202304/831526_VWWB6627U8N784J.jpg)

在这个函数中主要是初始化了两个大数数组, 之后进入`sub_9e93c`中进行某种运算, 其中第一个参数是输入的数字和0xffff00ffffff进行按位与运算的结果. 对于输入数字数字666666, 其计算的结果是`1db29b949927d377f0270e1161964bb0b9fc004f7125b6956d057e09a7c2fadf77df04bbcae93aa0000`字符串, 之后将`s2`的字符串与前面初始化的某个数组使用`strcmp`函数进行比较, 经过hook获取发现该数组是一个恒定不变的字符串`25f6b048b4f32e3ce9175bb64930f65101a706ae74988a4ec87b4d5ec7feb9223ab782bcf1ec9d7fee750`. 刚开始看到这个字符串想到了哈希, 但是具体是什么还需要进入`sub_9e93c`函数查看.

![](https://bbs.kanxue.com/upload/attach/202304/831526_55CX3VZC8CY3DQP.jpg)

`sub_9e93c`函数首先会进行一系列初始化, 然后进入`sub_9F0A8`函数和`sub_9f704`中, 使用`frida` hook也能够发现输入的数字是在最后执行完`sub_9f704`出现了字符串. 理所应当先进入`sub_9f704`看看这个字符串是如何产生的, 查看`frida-trace`的`log`能够发现其在某处开始会每8个一组产生0, 一直产出了168个0之后才生成想看到的字符串.
