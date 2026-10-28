
```c
typedef struct elf32_rel {
    Elf32_Addr    r_offset; // 要重定位地址的地址
    Elf32_Word    r_info;   // 属性(1)，符号表索引(3)
} Elf32_Rel;
```

- `r_info[0]` 三种类型

    - `R_ARM_JUMP_SLOT` 22 (**导入表中我只看到这种类型**)

      直接替换地址

    - `R_ARM_GLOB_DAT` 21

      直接替换地址

    - `R_ARM_ABS32` 2

      不是替换地址， 而是在找到符号地址的基础上， 加上原本地址存有的数值

> 实际上导入表`DT_JMPREL`导入的是函数地址， 对于其他模块变量的导入在后面的重定位表中实现

首先在动态段中定位导入表的内存偏移

![image-20220927154151417](https://bbs.kanxue.com/upload/attach/202209/890847_TDGVP7RMTY964US.jpg)

在之前的内存映射关系中， `0x51d4` < `0x 00 04 20 00`, 所以在第一个段中， 内存偏移等价于文件偏移

![image-20220926105402111](https://bbs.kanxue.com/upload/attach/202209/890847_7XGYT4VST3U78ZP.jpg)

可以发现ida的静态分析可以解析出这些内容

![image-20220927154925277](https://bbs.kanxue.com/upload/attach/202209/890847_QP49T7JHEJJPET4.jpg)

以`__libc_int`符号为例， linker首先遍历之前解析的所有导入库的导出符号，获取同名的导出符号的地址，然后替换到模块中的内存偏移地址`0x44B78` 的位置

> 这种寻找符号的方法导致了一种与windows 的PE文件格式不同的地方， ELF 文件的导入库与导入函数之间并不是强相关。
>
> 比如在linux的hook中，通过`LD_PRELOAD`环境变量的hook函数，并不需要指定模块名， 只需要函数原型相同即可完成hook。参考 87cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1#2y4o6x3^5z5l9`.`.

- 未导入符号地址， 该处的 `0x9cF0` 并不能指向函数

    ![image-20220927220935094](https://bbs.kanxue.com/upload/attach/202209/890847_KJMD6383Q7HF4KJ.jpg)

    该地址被如下代码引用， 代码结合起来类似于 `jmp [0x44B78]` , **这就是`R_ARM_JUMP_SLOT` 22类型的导入地址使用方式**

    ![image-20220927221104701](https://bbs.kanxue.com/upload/attach/202209/890847_ZC3972NFSX3WRP4.jpg)

- 导入符号地址

    ![image-20220927225447360](https://bbs.kanxue.com/upload/attach/202209/890847_RR3JBD572G8WFK4.jpg)

**关键代码解析：**

```java
public static class Elf32_Jmp {
    public byte[] r_offset = new byte[4];
    public byte[] r_info = new byte[4];

    public Elf32_Jmp() {
    }

    public String toString() {
        byte[] info = {r_info[1], r_info[2],r_info[3],0};
        return "r_offset(存放导入符号地址的地址(类似与windows的IAT（导入地址表）)， 相对于 .dynstr 的 sh_offset):" + VariableUtils.bytes2HexString(this.r_offset) +
                "\n属性(虚拟地址，指向符号所在的内存地址):" + Byte.toString(r_info[0]) + e_type_toString()+
                "\n符号(虚拟地址，指向符号所在的内存地址):" + 
                VariableUtils.bytes2HexStringReverse(info);
    }

    public String r_sym_toString(byte[] fileByteArys){
        byte[] info = {r_info[1], r_info[2],r_info[3],0};
        ElfType32.Elf32_Sym sym = AndroidElfParser.parseSymbolTableListD(fileByteArys, AndroidElfParser.type_32.symList_d,  VariableUtils.bytes2int(info));
        return sym.st_name_toString(fileByteArys);
    }

    public String e_type_toString(){
        final int R_ARM_JUMP_SLOT = 22;
        final int R_ARM_GLOB_DAT = 21;
        final int R_ARM_ABS32 = 2;

        int value = Byte.toUnsignedInt(r_info[0]);
        switch (value){
            case R_ARM_JUMP_SLOT:
                return "R_ARM_JUMP_SLOT 直接替换地址";
            case R_ARM_GLOB_DAT:
                return "R_ARM_GLOB_DAT 直接替换地址";
            case R_ARM_ABS32:
                return "R_ARM_ABS32 不是替换地址， 而是在找到符号地址的基础上， 加上原本地址存有的数值";
            default:
                return "其他类型";
        }
    }
}
```

![image-20220928003449811](https://bbs.kanxue.com/upload/attach/202209/890847_95BK4MDNMMFA8WU.jpg)

#### `DT_REL` 重定位表

解析 重定位表(类型=17 `DT_REL`), 尺寸为 (类型=18 `DT_RELSZ` ) 字节决定, 重定位表的每一项有16 字节{重定位地址， 属性(1)，符号表索引(3)}

> 重定位表的解析与导入表完全相同，导入符号的处理时，会将这两个表一起用于搜索符号, 重定位表有一部分是其他模块变量的导入表

表项结构如下

```c
typedef struct elf32_rel {
    Elf32_Addr    r_offset; // 要重定位地址的地址
    Elf32_Word    r_info;   // 属性(1)，符号表索引(3)
} Elf32_Rel;
```

- `r_info[0]` 三种类型

    - `R_ARM_RELATIVE` 23 **只有该项与`R_ARM_JUMP_SLOT` 22 数值上的不同**

      不是替换地址， 而是在找到符号地址的基础上， 加上原本地址存有的数值

    - `R_ARM_GLOB_DAT` 21

      直接替换地址

    - `R_ARM_ABS32` 2

      不是替换地址， 而是在找到符号地址的基础上， 加上原本地址存有的数值

先使用代码解析重定位表

![image-20220928013818474](https://bbs.kanxue.com/upload/attach/202209/890847_J2FQJA8XZT9PE6N.jpg)

首先定位第一个重定位的符号， 没有符号名， 重定位类型为 **`R_ARM_RELATIVE` 23**

![image-20220928014639644](https://bbs.kanxue.com/upload/attach/202209/890847_KXQGFPNRQJREEW4.jpg)

这要重定位的是全局字符串， 大多数也都是字符串需要重定位

![image-20220928014809051](https://bbs.kanxue.com/upload/attach/202209/890847_MY8MKGP5MDN39JQ.jpg)

而重定位类型 **`R_ARM_GLOB_DAT` 21** 的使用通常作为函数指针直接使用

![image-20220928015816767](https://bbs.kanxue.com/upload/attach/202209/890847_N9593YAGYGM9F88.jpg)

#### `DT_GUN_HASH` 导出/hash表

解析 导出/hash表表(类型=`0x6ffffef5` `DT_GUN_HASH`)

```c
struct{
    uint32_t nbucket; // bucket数组的个数, 每个bucket 4个字节
    uint32_t symoffset; // chain 表实际开始的符号表索引位置， 所以理论位置为chain-4*symoffset
    /* Bloom Filter用于提前停止查找丢失的符号, 本节不研究*/
    uint32_t bloom_size; // 指定bloom数组的个数, 每个bloom 4个字节
    uint32_t bloom_shift;
    uint32_t bloom[bloom_size];
    /* uint64_t for 64-bit binaries */
    /*bucket 每个bucket 4个字节,是符号表中的索引, 索引*0x10 + 符号表偏移 = 函数地址 */
    uint32_t bucket[nbucket];
    uint32_t chain[]; // chain 是符号名的hash
};
```

关于这个表的具体解析，参考： 38eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8r3q4H3k6h3&6Y4N6h3W2F1i4K6u0W2L8h3g2Q4x3V1k6W2L8r3k6Q4x3X3c8V1N6q4)9J5k6r3N6F1N6g2)9J5k6r3S2S2M7$3R3`.

下面这一套系统是用于快速索引导出表的一套结构， 理论上`chain[]`存储的是符号表的hash中的前31位， 所以项数`nchain`必须等于符号表的个数，不过由于符号表开始的一部分是导入表和重定位表的内容，为了减少空间的浪费所以`chain`表是从`sym_tab`的`symoffset`项开始的。

![image-20220923154015011](https://bbs.kanxue.com/upload/attach/202209/890847_UGQNMZBTC34P6KY.jpg)

如下所示的一个哈希函数输入一个符号名，输出一个值用于计算 `bucket` 索引。如果给出一个符号名，经哈希函数计算得到值 `x`，那么 `x%nbucket` 是 `bucket` 表内的索引，`bucket[x%nbucket]` 给出一个符号表的索引值 `y`，`y` 同时也是 `chain` 表内的索引值。如果`chain[y]`前31位与`hash`值不匹配，`chain[y]`第32位是0则继续向下匹配，否则结束匹配， 说明这个导出表中并不含有此符号。继续遍历下一个模块的导出表.

**哈希函数(hashing function)**

```c
unsigned long elf_hash(const unsigned char *name)
{
    unsigned long h = 0, g;
    while (*name)
    {
        h = (h << 4) + *name++;
        if (g = h & 0xf0000000)
            h ^= g >> 24;
        h &= ~g;
    }
    return h;
}
```

**关键解析代码:**

![image-20220928155758216](https://bbs.kanxue.com/upload/attach/202210/890847_72JQNR4TVFFNWR5.jpg)
