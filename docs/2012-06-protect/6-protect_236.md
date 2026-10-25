
# FLEXlm9.2 的破解教学四

> 原文：[`www.pediy.com/kssd/pediy06/pediy6936.htm`](https://www.pediy.com/kssd/pediy06/pediy6936.htm)

翻译第四篇 On Software Reverse Engineering - 4  
Reverse Engineering, FLEXlm, IMSL  

因此，我们有了两个处理：在 `cmath.exe` 中进行检查和在 `lmcrypt.exe` 或 `makekey.exe` 中产生密钥（keygen）。它们都可以产生相同的正确的许可证代码，但这两个处理不是完全相同的。我们已经在前面的章节对第一个处理进行了一定深度的分析，现在按顺序列出重要的调用链接。

1. `lc_new_job()` -> `l_n36_buf()` -> `l_x77_buf()`
2. `lc_new_job()` -> `lc_init()` -> `l_init()` -> `l_sg()` -> `l_key()` -> `l_zinit()`
3. `lc_set_attr()` -> `l_set_attr()` -> `l_set_license_path()` -> `l_flush_config()` -> `l_init_file()` -> `l_allfeat()` -> `l_parse_feature_line()` -> `oldkey()` -> `l_crypt_private()` -> `real_crypt()` -> `l_string_key()`
4. `lc_checkout()` -> `l_checkout()` -> `lm_start_real()` -> `l_good_lic_key()` -> `l_xorname()`
5. `lc_checkout()` -> `l_checkout()` -> `lm_start_real()` -> `l_good_lic_key()` -> `l_sg()` -> `l_n36_buff()`
6. `lc_checkout()` -> `l_checkout()` -> `lm_start_real()` -> `l_good_lic_key()` -> `l_crypt_private()` -> `real_crypt()` -> `l_string_key()`

一个很感兴趣的问题是：为什么 `l_sg()` 在第二个链接调用 `l_key()` 而在第五个链接调用 `l_n36_buff()`？检查代码摘录，我们发现答案在于 `LM_OPTFLAG_CUSTOM_KEY5` 和 `L_UNIQ_KEY5_FUNC`。后者由 `l_x77_buf()`（也就是 `L_SET_KEY5_FUNC`）在第一个链接中设置，因此在两个调用中，`l_n36_buff` 将非空。那么原因就是 `LM_OPT_FLAG_CUSTOM_KEY5` 了：它在调用 `lc_init()` 后进行开关转换，这就是为什么 `l_key()` 在第二个链接中被调用。有趣的是，在现在的 FLEXLM 版本中，`l_key()` 是一个无用的子程序（它在引入 `l_n36_buff()` 之前用于早期的版本）。此外，它完全没有必要在初始化阶段调用译码加密种子的 `l_sg()`；那本应该在检查时进行的。

`lm_njob.c`:

```c
int lc_new_job(oldjob, l_new_job, vcode, newjobp)
{
... ...
(*L_NEW_JOB)(vendor_name, vcode, 0, 0, 0, &sign_level);
(*L_NEW_JOB)(0, 0, 0, 0, 0, 0);
if (!(ret = lc_init(oldjob, vendor_name, vcode, newjobp)))
{
(*newjobp)->options->flags |= LM_OPTFLAG_CUSTOM_KEY5;
... ...
}
return ret;
}
```

`lm_ckout.c`:

```c
void l_sg(LM_HANDLE* job, char* vendor_id, VENDORCODE* key)
{
... ...
unsigned long x = 0x6f7330b8; /* v8.x */
if (( job->options->flags & LM_OPTFLAG_CUSTOM_KEY5) && L_UNIQ_KEY5_FUNC)
{
(*L_UNIQ_KEY5_FUNC)(job, vendor_id, key);
return;
}
l_key(vendor_id, &(key->keys[0]), keys, 4); /* Pre v6.1 style */
... ... /* 在 VKEY5()中进行相同的与或操作（xor） */
}
```

`lm_init.c`:

```c
void (*L_UNIQ_KEY5_FUNC)() = 0;

void L_SET_KEY5_FUNC( void (*f)())
{
if (!L_UNIQ_KEY5_FUNC) L_UNIQ_KEY5_FUNC = f;
}
```

在检查处理的同时，我们也在密钥产生（keygen）处理中调用链接。我们将使用 `lmcrypt.exe` 用于分析，因为它比 `makekey.exe` 来得更直接（它们都执行相同的工作）。

1. `lmcrypt.c!main()` -> `lc_init()` -> `l_init()` -> `l_sg()` -> `l_key()` -> `l_zinit()`
2. `lmcrypt.c!main()` -> `dofilecrypt()` -> `dofpcrypt()` -> `lm_crstr.c!lc_cryptstr()` -> `parsefeaturelist()` -> `l_parse_feature_line()` -> `oldkey()` -> `l_crypt_private()` -> `real_crypt()` -> `l_string_key()`
3. `lmcrypt.c!main()` -> `dofilecrypt()` -> `dofpcrypt()` -> `lm_crstr.c!lc_cryptstr()` -> `cryptfeaturelist()` -> `docryptfeat()` -> `lc_crypt()` = `l_crypt_private()` -> `real_crypt()` -> `l_string_key()`

注意：对于 Vendor 和 job 的初始化，`cmath.exe` 将调用 `lc_new_job()`，（该函数）轮流调用 `lc_init()`；但是 `lmcrypt.exe` 将直接调用 `lc_init()`，因为 vendor 密钥、种子和名称都已经包含于 `lmcrypt.exe` 中了（一起通过宏放到 vendor 结构中），因此仅需要初始化工作。在这两个处理中，有对 `l_string_key()` 的两个调用；在这两种情况下，第一个返回不重要的 `oldkey()` 号为 21D5B6E8572E，仅第二个调用其实质。这两个处理在调用 `l_string_key()` 上仅有细微的差别。基本上，对于 checksum（总和检查）比较，checkout（检查）需要提供用户许可证密钥，但是 keygen 则不需要。但是，对于计算实际的杂乱信息上则是相同的。

```c
int idx = (*job->vendor) % XOR_SEEDS_ARRAY_SIZ; /* idx = V % 20 = 86 % 20 = 6 */
... ...
memset(y, 0, L_STRKEY_BLOCKSIZE); /* L_STRKEY_BLOCKSIZE = 8, in lmachdep.h */
length = (inputlen) / L_STRKEY_BLOCKSIZE;
XOR_SEEDS_INIT_ARRAY(xor_arr) /*在 l_strkey.h 中定义的替换表 */
... ... /* memcpy() 从输入到 newinput 和其他充填（值）*/
p = newinput;
for (i = 0; i < length; i++)
{
XOR(p, y, y); /*在 l_strkey.h 中定义的与或(XOR)和 L_MOVELONG */
if (i == 0)
{
if (!user_crypt_filter && !user_crypt_filter_gen
&& (job->flags & LM_FLAG_MAKE_OLD_KEY))
{
q = y; /*在 l_privat.h 中定义的 SEEDS_XOR = mem_ptr2_bytes */
L_MOVELONG(code->data[0]
^((long)(job->SEEDS_XOR[xor_arr[idx][0]])<<0)
^((long)(job->SEEDS_XOR[xor_arr[idx][1]])<<8)
^((long)(job->SEEDS_XOR[xor_arr[idx][2]])<<16)
^((long)(job->SEEDS_XOR[xor_arr[idx][3]])<<24), q)
L_MOVELONG(code->data[1]
^((long)(job->SEEDS_XOR[xor_arr[idx][0]])<<0)
^((long)(job->SEEDS_XOR[xor_arr[idx][1]])<<8)
^((long)(job->SEEDS_XOR[xor_arr[idx][2]])<<16)
^((long)(job->SEEDS_XOR[xor_arr[idx][3]])<<24), q)
}
... ...
}
if (!(job->flags & LM_FLAG_MAKE_OLD_KEY) && !demo)
our_encrypt2(y);
else
our_encrypt(y); /* our_encrypt()并不涉及代码或工作*/
p += L_STRKEY_BLOCKSIZE;
}
if (len == L_SECLEN_SHORT) /* 在 l_privat.h 中，L_SECLEN_SHORT = 0x66D8B337 */
{
... ...
y[6] = y[7] = 0;
}
```

因为它们共享了相同的代码，因此为了计算相同的杂乱信息，传入的参数必须相同。实际的追踪结果为：

`cmath.exe`:

```text
job = 00887630, input = 0012e170, inputlen = 0x16, code = 0012ee20, len = 66d8b337, license_key = 6d5c...
[00887630] - 00000066 f... 整型;
[00887634] - 0089008e .... char *mem_ptr2;
[00887638] - a06aa84e N.j. unsigned char mem_ptr2_bytes[12]; (12 是视进制)
[0088763C] - 00c3a047 G...
[00887640] - 00660000 ..f.
[00887644] - 00000000 ....
[00887648] - 00000000 ....
[0088764C] - 00000000 ....
[00887650] - 00000000 ....
[00887654] - 54414d43 CMAT
[00887658] - 00000048 H...
[0012E170] - ab370fd2 ..7. 输入串用于杂乱信息，取决于 FEATURE 行信息
[0012E174] - 414d4300 .CMA
[0012E178] - 88054854 TH..
[0012E17C] - 6a000113 ...j
[0012E180] - c5876e61 an..
[0012E184] - 000073d0 .s.. 总长度= 0x16 = 22,在 73 结束
[0012E188] - 00000000 ....
[0012EE20] - 00000004 .... 整型;
[0012EE24] - 52ed15b8 ...R 52xxxxb8, xxxx 是随机的,在每次运行是不同的
[0012EE28] - 75cf780f .x.u 75yyyy0f, yyyy 是随机的,在每次运行是不同的
[0012EE2C] - 7c2adb6a j.*| VENDOR_KEY1
[0012EE30] - b927f5a9 ..'. VENDOR_KEY2
[0012EE34] - 9cf311f8 .... VENDOR_KEY3
[0012EE38] - 0dbf7621 !v.. VENDOR_KEY4
[0012EE3C] - 00020009 .... FLEXlm 版本(这里是 9.2)
[0012EE40] - 39300020 .09
[0012EE44] - 0000302e .0..
[0012EE48] - 00000000 ....
```

`lmcrypt.exe`:

```text
job = 008C49E8, input = 0012D8D4, inputlen = 0x16, code = 004D7B48, len = 66D8B337
0x008C49E8 66 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 f...............
0x008C49F8 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x008C4A08 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x008C4A18 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x0012D8D4 d2 0f 37 ab 00 43 4d 41 54 48 05 88 13 01 00 6a Ò.7«.CMATH.....j
0x0012D8E4 61 6e 87 c5 d0 73 00 00 00 00 00 00 00 00 00 00 an.ÅÐs..........
0x0012D8F4 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x0012D904 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x004D7B48 04 00 00 00 b8 39 c1 52 0f 54 e3 75 6a db 2a 7c ....¸9ÁR.TãujÛ\*|
0x004D7B58 a9 f5 27 b9 f8 11 f3 9c 21 76 bf 0d 09 00 02 00 ©õ'¹ø.ó.!v¿.....
0x004D7B68 20 00 30 38 2e 30 00 00 c3 80 f4 83 2c c0 1c 77 .08.0..Ã.ô.,À.w
0x004D7B78 00 00 00 00 01 00 00 00 00 00 00 00 00 00 00 00 ................
```

注意：我们在这里处理诸如 `6D5C01FD71C9` 的非-CRO 的短密钥，它由 12 个 ASCII 字符组成表示 6 个十六进制位，这也就是为什么我们在上面的代码中看到“`y[6] = y[7] = 0;`”。除了 `VENDORCODE` 和 `job` 结构之外，所有其它参数是相等的`[9]`，且在这两个结构中并非所有的组成部分都是重要的。很容易勾勒出在杂乱化（hashing？）输入串中所参与的东西为：`code->data[]`、`code->keys[]` 和 `job->mem_ptr2_bytes[]`。

我们立即拷贝在检查（checkout）处理中揭示的四个 vendor 密钥到 `lm_code.h` 中（并证实在内存信息转储中），但是我们仍然缺少加密种子和神秘的 `VENDOR_LEY5`。现在，更进一步察看 n 密钥产生（keyge）处理，它在检查（checkout）的 `code->data[]` 和 `job->mem_ptr2_bytes[]`（它的 job 结构是空的）中出现不同。那是为什么呢？

```c
#include "lmprikey.h"

#include "lmclient.h"

#include "lm_code.h"

#include "lmseeds.h"

... ...

/* 设置 site_code.data = {ENCRYPTION_SEED1 ^ VENDOR_KEY5,
ENCRYPTION_SEED2 ^ VENDOR_KEY5}
```



`site_code.keys = {VENDOR_KEY1, VENDOR_KEY2, VENDOR_KEY3, VENDOR_KEY4} */`

`LM_CODE(site_code, ENCRYPTION_SEED1, ENCRYPTION_SEED2, VENDOR_KEY1,`
`VENDOR_KEY2, VENDOR_KEY3, VENDOR_KEY4, VENDOR_KEY5);`

... ...

`int main(int argc, char **argv)`

{
... ...
`/*设置 site_code.data = {ENCRYPTION_SEED1, ENCRYPTION_SEED2} */`

`LM_CODE_GEN_INIT(&site_code);`

`if (lc_init((LM_HANDLE *)0, VENDOR_NAME, &site_code, &lm_job))`

{
`lc_perror(lm_job, "lc_init failed");`

`exit(-1);`

}

... ...

`/*调用链接 dofilecrypt() -> dofpcrypt() -> lc_cryptstr() */`

`estat |= dofilecrypt(infilename, outfilename, &site_code);`

`return 0;`
}

这是 `lmcrypt.c` 的简明源代码及其自身说明。仔细注意到在 `lmclient.h` 中定义的两个宏 `LM_CODE` 和 `LM_CODE_GEN_INIT`。前者初始化 `site_code.data` 为与 `VENDOR_KEY5` 进行与或的加密种子（与 `omcode.c` 一致），但后者很快逆向它为原始的加密种子。我是否曾告诉你 `FLEXLM` 有很糟的代码类型？：）

不管怎样，原始的加密种子和零 `job->mem_ptr2_bytes` 值被用于密钥产生（`keygen`），它不同于检查（`checkout`）。很自然的事情是从检查处理拷贝 `code->data` 到 `lmseeds.h` 作为加密种子 1 和 2，并重新编译 `lmcrypt.exe`。但它并不工作，因为在表格 `52xxxxB8` 和 `75yyyy0F` 中，`code->data` 是随机的，而 `xxxx` 和 `yyyy` 在每次运行中都将改变。相同的是，对于 `job->mem_ptr2_bytes` 是真实的。我们推断，加密种子必定被模糊并存储在 `vendor` 软件中的两个地方：`code->data[]` 和 `job->mem_ptr2_bytes[]`（它们应该是关系很近的一对），因为它提供给最终用户而原始种子需要保护。相反，`lmcrypt.exe` 仅对 `vendor` 可获得，因此加密种子可以以一般形式出现。

`4dd466d0e1e0cc123afbe504da2462ca`
