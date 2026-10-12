
# 翻译 FLEXlm9.2 的破解教学六

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-706.htm`](https://www.pediy.com/kssd/pediy07/pediy7-706.htm)

对于 4 个 Vendor 密钥，很明显它们是初始化时在 `l_n36_buf()` 中被模糊的，在 `l_xorname()` 中进行模糊逆向，`l_xorname()` 未做别的仅进行了一些与或操作。我们都知道与或的特征：`x ^ x = 0`, `x ^ 0 = x`, `x ^ 1 = ~x`，因此进行两次相同的与或操作将相互取消。这个好特性将与或适用于编码和译码。如果我们的猜测对的话，那么 `l_n36_buf()` 将采用与 `l_good_lic_key()` 相同的方式调用 `l_xorname()`。因为 `l_n36_buf()` 驻留于 `lm_new.c`，它由 `lmnewgen.exe` 产生，因此直接检查 `lmnewgen.c` 更好。

```c
#include "l_strkey.h"

... ...

int main(int argc, char **argv)
{
    ... ...
    strcpy(outfile, "lm_new.c");
    ... ...

/* 初始化工作，能报告错误并能退出 */
if (lc_init(0, vendor_name, &vendorkeys[0], &job))
    {   

fprintf(stderr, "lc_init failed: %s\n", lc_errstring(job));
        exit(1);
    }   

/* 通过 sb_rngFIPS186Session()产生的随机数 */
l_genrand(job, lmseed1, lmseed2, lmseed3, NEWSEEDSIZ, newseeds);
    ... ... /* 把新种子转换为 seed1 – seed4,及其它 */
/* 把 4 个加密种子写进 lmseeds.h */
if (!(ofp = fopen("lmseeds.h", "w")))
    {   

perror("Can't write lmseeds.h file, exiting");
        exit(1);
    }   

fprintf(ofp, "... ...", seed1, seed2, seed3, seed4);
    fclose(ofp);

/*现在是 lm_new.c */
if (!(ofp = fopen(outfile, "w")))
    {   

perror("Can't open output file, exiting");
        exit(1);
    }   

    ... ...

/* 确定缺省的和弱的种子被排除在外 */
if (!l_reasonable_seed(seed3) || !l_reasonable_seed(seed4) ||
        !l_reasonable_seed(lmseed1) || !l_reasonable_seed(lmseed2) ||
        !l_reasonable_seed(lmseed3))
    {   

        ... ...
        fprintf(stderr, "Existing.\n");
        exit(1);
    }   

    ... ...

    fputs("#include <time.h>\n", ofp);
    do_real(); /* 写 lm_new.c 的主要内容 */
    fclose(ofp);
    return 0;
}

static void do_real()
{
    ... ... /* 产生随机的参数和函数名称 */
    while (!key5_uniqx)
    {   

key5_uniqx = our_rand(256) + (our_rand(256) << 8) +
                (our_rand(256) << 16) + (our_rand(256) << 24);
    }   

    ... ... /* 产生随机的 key5_order[] */
for (i = 0; i < 200; i += 2)
        random_garbage();
    ... ... /* 更多的随机垃圾 */
for (counter=0; counter<keysize; counter++)/* 一般 keysize=1, 只有一个 vendor */
    {   

        key5(&vendorkeys[counter]);  /* 模糊化 vendorkeys->data[] */
        l_xorname(vname, &vendorkeys[counter]);/* 模糊化 vendorkeys->keys[] */
/* 真正 VENDORCODE 的初始化 */
        do_ulong(vendorkeys[counter].data[0], "data[0]", counter);
        do_ulong(vendorkeys[counter].data[1], "data[1]", counter);
        do_ulong(vendorkeys[counter].keys[0], "keys[0]", counter);
        do_ulong(vendorkeys[counter].keys[1], "keys[1]", counter);
        do_ulong(vendorkeys[counter].keys[2], "keys[2]", counter);
        do_ulong(vendorkeys[counter].keys[3], "keys[3]", counter);
        ... ...
    }   

    l_puts_rand(ofp, fpVar, numvars); /* 以随机的顺序输出行 */
    fflush(ofp);
    uniqcode(); /* 输出 lm_new.c!l_n36_buff()的源代码 */
    fflush(ofp);
    ... ...
}

static void do_ulong(unsigned long ul, char *varname, int counter)
{
    ... ...
randvarname(b1, "b1"); bnames[0] = b1; /* 产生混乱的参数名称 */
randvarname(b2, "b2"); bnames[1] = b2;
randvarname(b3, "b3"); bnames[2] = b3;
randvarname(b4, "b4"); bnames[3] = b4;
for (i = 0; i < 4; i++)
    {   

shift = i * 8;
        CLEARV; /* 把 ul 分成 4 字节并且把它们指派为 4 个糊乱命名的参数 */
sprintf(vBuf, "static unsigned int %s = %d;\n",
bnames[i], (ul & (0xff << shift)) >> shift );
        fwrite(vBuf, sizeof(char), sizeof(vBuf), fpVar);
        CLEARF; /* 重新组装回 4 个参数，使 v->varname = ul */
sprintf(fBuf, "\tif (%s == %d) v->%s += (%s << %d);\n",
              countervar, counter, varname, bnames[i], shift);
        fwrite(fBuf, sizeof(char), sizeof(fBuf), fpFunc);
    }   

}

static void uniqcode()
{
    ... ...
int idx = *vendor_name % XOR_SEEDS_ARRAY_SIZ; /* idx = V % 20 = 86 % 20 = 6 */
    XOR_SEEDS_INIT_ARRAY(xor_arr) /* l_strkey.h */
fprintf(ofp, "static void %s(job, vendor_id, key) \n\n"
            "..."
            "unsigned long x = 0x%x;\n\n"
            "struct s_tmp {int i; char *cp; unsigned char a[12];} *t, t2;\n\n"
            "if (job) t = (struct s_tmp *)job;\n\n"
            "            else t = &t2;\n\n"
            "... ...", key5_fname, key5_uniqx); /* key5_fname 和 key5_uniqx 都是随机数 */
for(i = 0; i < SEEDS_XOR_NUM; i++) /* 解码 t->a[], i.e. job->mem_ptr2_bytes[] */
    {   

        unsigned char num;
        cpp[i] = cp;
sprintf(cp, "t->cp=(char *)(((long)t->cp) ^ (time(0) ^ ((0x%x << 16) + 0x%x)));\n\n"
"t->a[%d] = (time(0) & 0xff) ^ 0x%x;\n", /* 运行时随机化 */
              our_rand(0xff), our_rand(0xff), i, our_rand(0xff));
cp += strlen(cp) + 1;
    }   

    l_puts_rand1(ofp, SEEDS_XOR_NUM, cpp); /* 以随机的顺序输出行 */
fprintf(ofp, "for (i = 0; i < %d; i++) \n\n"
            "{ \n\n"
               "if (sig[i%%SIGSIZE] != vendor_id[i%%len]) \n\n"
                 "sig[i%%SIGSIZE] ^= vendor_id[i%%len]; \n\n"
            "} \n\n"
"unsigned long y = ((((long)sig[0] << %d) \n\n"
"| ((long)sig[1] << %d) \n\n"
   "| ((long)sig[2] << %d) \n\n"
"| ((long)sig[3] << %d)) \n\n"
"^ ((long)(t->a[%d]) <<  0) \n\n"
"^ ((long)(t->a[%d]) <<  8) \n\n"
"^ ((long)(t->a[%d]) << 16) \n\n"
"^ ((long)(t->a[%d]) << 24) \n\n"
"^ x \n\n"
"^ key->keys[0] \n\n"
"^ key->keys[1]) & 0xffffffff; \n\n"
"key->data[0] ^= y; \n\n"
"key->data[1] ^= y; \n\n"
     "... ...", MAX_DAEMON_NAME, /* MAX_DAEMON_NAME = 10 在 lmclient.h 中定义 */
key5_order[0], key5_order[1], key5_order[2], key5_order[3],
xor_arr[idx][0], xor_arr[idx][1], xor_arr[idx][2], xor_arr[idx][3]));
}

static void key5(VENDORCODE *k) /* 模糊化加密种子, 即 k->data[] */
{
    ... ...
/* 和 uniqcode()中相同的 key5_uniqx, key5_order[], sig[]和 MAX_DAEMON_NAME */
for (i = 0; i < MAX_DAEMON_NAME; i++)
    {   

if (sig[i%SIGSIZE] != vname[i % len])
            sig[i%SIGSIZE] ^= vname[i % len];
    }   

y = ((((long)sig[0] << key5_order[0])
         | ((long)sig[1] << key5_order[1])
         | ((long)sig[2] << key5_order[2])
         | ((long)sig[3] << key5_order[3]))
         ^ key5_uniqx
         ^ k->keys[0]
         ^ k->keys[1]) & 0xffffffff;
    k->data[0] ^= y; /* k->data[0] = 加密种子 SEED1 ^ y */
    k->data[1] ^= y; /* k->data[1] = 加密种子 SEED2 ^ y */
}
```

源码有点长，但是它的结构一旦被我们揭开就不再非常难懂。主要的过程在 `lmseeds.h` 中花费大量时间，并且 `lm_new.c` 主要是在 `do_real()` 中产生的。在几个子程序的帮助下，`do_real()` 本身是在制造 `l_n36_buf()`，它把 `l_n36_buff()` 留给 `uniqcode()`。那么 `l_xorname()` 干什么用呢? 它虽然没有出现在 `l_n36_buf()` 中，但是在 `do_real()` 中出现了。因此这种说法是正确的：4 个 vendor key 在不同的位置被同一个异或操作来进行加密和解密，它们分别是 `do_real()` / `l_n36_buf()` 和 `l_good_lic_key()`

注意 `lm_new.c` 本身被高度模糊化：这里有大量完全无用的垃圾代码, 标识符是随机、无任何意义的， 真代码和垃圾代码看上去相似并且混在一起，真代码的行顺序被打乱……。很明显，它以这种方式设计的目的是用来迷惑读者，并且它成功了，因为理解 C 源码几乎是不可能的，更不用说反编译码了。这正是我们为什么要把注意力放在它的起源 `lmnewgen.c` 上的原因, 因为后者能给我们更多提示。例如，`l_xorname()` 在 `do_real()` 中被调用,而不是 `l_n36_buf()`。这意味着密钥的模糊化是在 vendor site 的内部进行的，并且加密后的密钥被链接到传送到终端用户的目标中，然后它们在 `l_good_lic_key()` 中被实时解码。这种方案是为了最小化真实密钥的暴露。

但是，我们对加密种子（encryption seeds）比 vendor key 更感兴趣。回想一下，`job->mem_ptr2_bytes[]` 和 `code->data[]` 在验证和密钥产生的过程中是不一样的。很可能 FLEXlm 也使用了异或进行种子加密，但是远不止这些，因为我们在运行时看到它是随机的。我们将逐步分析。
