
# WindowsPE文件格式入门09.RadAsm的bug和重定位表

[586K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1M7s2y4W2L8X3c8Q4x3X3g2F1k6i4c8Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7K6z5o6y4Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@90dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1M7s2y4W2L8X3c8Q4x3X3g2F1k6i4c8Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7K6z5o6y4Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.)

## RadAsm的bug

### 创建程序

1、创建程序1：[C++](https://bbs.kanxue.com/elink@08aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1M7s2y4W2L8X3c8Q4x3X3g2F1k6i4c8Q4x3V1k6X3L8%4u0#2L8g2)9J5k6o6u0Q4x3X3b7I4i4K6u0W2K9s2c8E0L8l9`.`.)工程：

●项目选项：控制台`"hello，World"`程序，不使用预编译头

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_ETPWU6GX5AYRGGB.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_ESFJK48T49AUBDH.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_S5TAQZCSDM4R43F.jpg)

2、创建程序2：汇编工程：

●radasm工程选项：控制台

●将程序1中的`obj`拷贝至本工程中并添加至工程连接选项，用程序2调用程序1中的`TestFunc`函数

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_3BM2YG9S4MRYMNA.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_CND8FAHB2C3CYXV.jpg)

### 获取并解决BUG

1.非预期BUG：提示缺少lib，`LIBCD.LIB`、`OLDNAMES.lib`。

●解决办法：从VC库中获取并拷贝过来。

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_4DRF2ENT688AVCA.jpg)

2.预期BUG1：`LIBCD.lib` BUG：`LNK2001`：无法处理的外部符号 `_main`。

●原因：虽然汇编当中未使用此库，但是在C++程序中使用到了C库`printf`。而程序2汇编没有main函数，所以就会在连接时报预期BUG。

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_3WF39BBQ3HUB25E.jpg)

●解决方案：增加`main`，即把汇编程序的入口标号替换为`main`。

●效果：报错消失，但程序依然报其他错误。

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_Q8WEFPTNXQBXQKF.jpg)

3.预期BUG2：指定函数未定义

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_2EHTDFQQZXFAEGK.jpg)

●BUG探究：查找`GetEnvironmentStrings`函数定义（`kernel32.lib`），将lib包含后仍出现指定BUG。

●BUG再探究：从`kernel32.lib`中查询报错API，发现对应接口不分A/W，而是将`GetEnvironmentStrings`作为定义，`GetEnvironmentStringA`作为宏，违反了我们过往对于微软命名风格的认知习惯。所以造成了预期bug2。

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_XYCDHAVAFTARZDH.jpg)

●解决方案

○（1）手动修改radasm的`kernel32.inc`文件；

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_YUHH2XBWWH48M76.jpg)

```
修改前：

GetDriveTypeW PROTO :DWORD

GetEnvironmentStringsA PROTO 

GetEnvironmentStrings equ <GetEnvironmentStringsA>



修改后：

GetDriveTypeW PROTO :DWORD

GetEnvironmentStrings PROTO 

GetEnvironmentStringsA equ <GetEnvironmentStrings>
```

- （2）重新生成LIB的工具路径：`..\RadASM\masm32\tools\inc2l\inc2l.exe`

- （3）将两者拷贝出来并`cmd`命令：`inc2l.exe kernel32.inc`。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_4362PTH26GBMJ3W.jpg)

- （4）此时目录下未见新生成lib，研究一下`inc2l.exe`

#### 分析程序：`inc2l.exe`

##### 1.OD调试 引入壳

![img](https://bbs.kanxue.com/upload/attach/202505/865778_GPRSMXP2RM3Y5HW.jpg)

此时点击否就可以

![img](https://bbs.kanxue.com/upload/attach/202505/865778_79P7H6SSCDCTTBY.jpg)

- 可以看到,汇编代码十分诡异,怀疑是加了壳,我们现在的代码不是程序真正的入口点,而是壳的代码

- 壳的作用：对PE的保护。

- 壳的原理：在执行真正的PE前，壳会先跑一段对PE进行处理的壳代码

- 脱壳流程：1.查壳；2.有壳就去找`OEP`；3.dump进程

- 通杀`pushad`技巧（压缩壳和部分加密壳）：`ESP`定律。

##### 2.`ESP`定律

- 所谓`ESP`定律就是利用了`pushad`的设计原理，在一开始的时候，去栈上数据进行硬件读处理，当壳执行完毕要恢复PE前，即可快速锁定`OEP`定位。

##### 3.OD分析定位`OEP`

- （1）`F8`单步过`pushad`入栈，跟踪`ESP`，以`DWORD`类型，下硬件访问断点至第二组或第三组（王老师经验）。

`esp 右键,数据窗口中跟随`

![img](https://bbs.kanxue.com/upload/attach/202505/865778_AFB8R57BCVDTHKT.jpg)

![img](https://bbs.kanxue.com/upload/attach/202505/865778_QAMX2FP6VPTHRZF.jpg)

`F9`

![img](https://bbs.kanxue.com/upload/attach/202505/865778_3MP3QV554YBJSET.jpg)

`F8` 入口点

![img](https://bbs.kanxue.com/upload/attach/202505/865778_XHHNR4H8ZJT6QM7.jpg)

#### 脱壳

##### OllyDump

- 1.设置：插件 → OllyDump （）→ 脱壳在当前调试的进程 （Dump process）

- 此处幺蛾子：自版本的OD只有OllyDumpEx插件，无法同步以下操作，需要使用radasm自带的OD。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_NPS8XVGEBURYJGM.jpg)

- 2.选项：将`EIP`设为`OEP`，并确认转储为`xxx\_dump.exe`。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_CA9BPR3W2QZ25X5.jpg)

- 用cff查看,发现导入表错误

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_4923DVXRMVB8MSQ.jpg)

- 3.OD调试`xxx\_dump.exe`，发现程序无法运行。单步调试找到崩溃处，右键"长型→地址"。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_9XPVFGTX8GTQAXJ.jpg)

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_3G5MGWWE922B5D2.jpg)

- 4.思考原因：发现是导入表缺少了`IAT`，可能是由于dump后数据格式会错位，导致插件无法正确读取进行转储。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_RRGHZC7YKU8KFC2.jpg)

##### x32dbg

解决办法：使用`x32dbg`重建导入表。

- （1）定位指令于`OEF`处，并于此行设置硬件断点。这里的代码即将被改,所以下cc断点不行

![img](https://bbs.kanxue.com/upload/attach/202505/865778_Y2DMUEBGN4K35CH.jpg)

可以看到代码被改了

![img](https://bbs.kanxue.com/upload/attach/202505/865778_7FZEAQNCJ7ZTGGP.jpg)

- 设置在内存窗口中转到→常数，并将内存窗口区设置为地址。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_UAJ87YQHDKDTRN3.jpg)

- 插件 → Scylla处理 → 自动搜索 → 获取导入 → `Fix Dump` 修复转储至原dump程序（生成`xxx\_dump\_SCY.exe`）。

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_Q9N8U98UVBQ9J2M.jpg)

- ![img](https://bbs.kanxue.com/upload/attach/202505/865778_SJPW3RXBT9GQBTH.jpg)

- 在用cff查看 此时查看`IA`表已经导入成功。

![img](https://bbs.kanxue.com/upload/attach/202505/865778_GKFUGN638QYKMWR.jpg)

##### ImportREC

一般32位程序在32位系统中修复

![img](https://bbs.kanxue.com/upload/attach/202505/865778_632VMZJBB9ERXP6.jpg)

#### 修复`inc2l.exe`

`inc2l.exe`里面编译和链接用的绝对路径,换成相对路径就可以了,还有改字符串对应长度

修改编译选项

![img](https://bbs.kanxue.com/upload/attach/202505/865778_JWMYMX65WC6W78N.jpg)

![img](https://bbs.kanxue.com/upload/attach/202505/865778_E3773VS4S4E7MFR.jpg)

修改链接选项

![img](https://bbs.kanxue.com/upload/attach/202505/865778_WNWNX9GAJJDPRBG.jpg)

![img](https://bbs.kanxue.com/upload/attach/202505/865778_Z4BUGP3QMMBX7SB.jpg)

## 重定位表

定义：记录需要绝对地址修正的表，大多数绝对地址如果`imagebase`变化的话就无法使用，需要修正程序所调用的那些绝对地址。

- 修正方法：需要重定位的地址 + 偏移（当前基址 - PE的基址）

- 开了随机基址的程序才需要重定位，而DLL通常都有重定位表，因为不一定能够加载到DLL指定的`ImageBase`上。

### OS如何判定是否重定位？

- 先查看随机地址标志，标志开启，地址重定位

- 再查看数据目录项 5 是否为`NULL`，不为`NULL`，基址重定位。

我们现在都是玩固定基址的PE，随机基址涉及到要修代码，如果有重定位信息，就可以在内存中随便申请一块内存，把代码放进去跑。

我们知道随机基址需要重定位表来修代码， 那么是修什么代码呢。

实际上我们修的是使用绝对地址的代码，例如API的调用，通过`IAT`调用，这里就是使用的绝对地址，当模块基址改变时，原`VA`地址并没有保存API函数地址，所以就需要修正到正确的位置去获取API地址。

### img设计思路

假设以下 RVA 地址需要进行修正,最简单的方法是把下面的地址都记录下来,加载的时候直接去修正

`00001023`

`00001028`

`00001128`

`00001228`

`00001328`

`00001428`

但是直接保存所有地址,那么数组的体积就会变得很大,那么如何减少体积呢

可以按照 分页地址 +分页偏移 的方式记录,因为分页偏移只需要 2个字节就可以了

`00001000` 分页基址

`0000014` 总大小

`0023` 分页偏移

`0028`

`0128`

`0228`

`0328`

`0428`

### 重定位表的结构

- 重定位表的位置：在数据目录的[5]项，`IMAGE_BASE_RELOCATION`，共8+N字节。

```
// IMAGE_BASE_RELOCATION 重定位结构体，以8字节全0结尾

typedef struct _IMAGE_BASE_RELOCATION {                 

    DWORD   VirtualAddress; ;+0x00, 分页基址            

    DWORD   SizeOfBlock;    ;+0x04, 对应重定位数据块的大小，以字节为单位  

    // WORD TypeOffset[1]   ;+0x08, 重定项位数组，个数=(SizeOfBlock-8)/2

    // TypeOffset解析：高4位两个取值--0无需重定位(多用于对齐)，3需要重定位；低12位是页内偏移；

} IMAGE_BASE_RELOCATION;                    

typedef IMAGE_BASE_RELOCATION ，* PIMAGE_BASE_RELOCATION;
```

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_8E2NK2R7VVS288V.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_GFUVCC856EDY3P6.jpg)

`1F000` 对应文件偏移 `9200`

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_2E2GAG9GQSSZ4SE.jpg)

所有分页的数据大小合计就是 `3B0` 跟 结构体里面的总大小一致

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_HZMHFAH7FDME8XB.jpg)

偏移的最高位是 `3` 表示需要被重定位 , `0` 表示不需要,代表要对齐

OD中,有下划线的是代表修正后的地址

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_NCCVSUW8UNPAZWE.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_5YEB6TVUQ6BRQP8.jpg)

`3538` 开头是`3` 表示是一个有效重定位项 偏移值是 `538` ,分页是 `12000`

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_KKCWMVHVEJYNCA7.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_QMECXXP7XSZ8RXU.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_DGHMBJ6MUTWGFTA.jpg)

```
原基址: 10000000
新基址: 78b80000
偏移 : 68b80000
地址 1001788c + 68b80000 = 78 B9 78 8C
8c 78 b9 78
```

![image.png](https://bbs.kanxue.com/upload/attach/202505/865778_W9AYZDXUEBB6HXY.jpg)

### `LoadDll`

一个PE 有重定位表, 那么我们就可以把它加载到任意地址,然后修复重定位表即可,就可以模拟 `LoadLibrary` 了
