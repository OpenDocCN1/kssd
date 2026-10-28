
`template<typename ElfRelIteratorT>` `bool` `soinfo::relocate(const VersionTracker& version_tracker, ElfRelIteratorT&& rel_iterator,`
`const soinfo_list_t& global_group, const soinfo_list_t& local_group){`
`for` `(size_t idx` `=` `0``; rel_iterator.has_next();` `+``+``idx) {`
`const auto rel` `=` `rel_iterator.``next``();`
`if` `(rel` `=``=` `nullptr) {`
`return` `false;`
`}`
`/``/` `获取rel 的类型 ， 和符号索引`
`ElfW(Word)` `type` `=` `ELFW(R_TYPE)(rel``-``>r_info);`
`ElfW(Word) sym` `=` `ELFW(R_SYM)(rel``-``>r_info);`
`/``/``rel 内存地址`
`ElfW(Addr) reloc` `=` `static_cast<ElfW(Addr)>(rel``-``>r_offset` `+` `load_bias);`
`ElfW(Addr) sym_addr` `=` `0``;`
`const char``*` `sym_name` `=` `nullptr;`
`ElfW(Addr) addend` `=` `get_addend(rel, reloc);`
`DEBUG(``"Processing \"%s\" relocation at index %zd"``, get_realpath(), idx);`
`if` `(``type` `=``=` `R_GENERIC_NONE) {`
`continue``;`
`}`
`const ElfW(Sym)``*` `s` `=` `nullptr;`
`soinfo``*` `lsi` `=` `nullptr;`
`if` `(sym !``=` `0``) {`
`/``/``从符号表中获取到符号名称`
`sym_name` `=` `get_string(symtab_[sym].st_name);`
`const version_info``*` `vi` `=` `nullptr;`
`if` `(!lookup_version_info(version_tracker, sym, sym_name, &vi)) {`
`return` `false;`
`}`
`/``/` `这个函数可以解释LD_PRELOAD 对于相同符号名称linker的处理方式。`
`if` `(!soinfo_do_lookup(this, sym_name, vi, &lsi, global_group, local_group, &s)) {`
`return` `false;`
`}`
`if` `(s` `=``=` `nullptr) {`
`/``/` `We only allow an undefined symbol` `if` `this` `is` `a weak reference...`
`s` `=` `&symtab_[sym];`
`if` `(ELF_ST_BIND(s``-``>st_info) !``=` `STB_WEAK) {`
`DL_ERR(``"cannot locate symbol \"%s\" referenced by \"%s\"..."``, sym_name, get_realpath());`
`return` `false;`
`}`
`#elif defined(__arm__)`
`case R_ARM_ABS32:`
`#endif`
`/``*`
`*` `The sym_addr was initialized to be zero above,`
`or` `the relocation`
`*` `code below does` `not` `care about value of sym_addr.`
`*` `No need to do anything.`
`*``/`
`break``;`
`#if defined(__x86_64__)`
`case R_X86_64_PC32:`
`sym_addr` `=` `reloc;`
`break``;`
`#elif defined(__i386__)`
`case R_386_PC32:`
`sym_addr` `=` `reloc;`
`break``;`
`#endif`
`default:`
`DL_ERR(``"unknown weak reloc type %d @ %p (%zu)"``,` `type``, rel, idx);`
`return` `false;`
`}`
`}`
`else` `{`
`/``/` `We got a definition.`
`/``*`
`.......`
`*``/`
`}`
`count_relocation(kRelocSymbol);`
`}`
`/``/` `重定位运算。`
`switch (``type``) {`
`case R_GENERIC_JUMP_SLOT:`
`count_relocation(kRelocAbsolute);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO JMP_SLOT %16p <- %16p %s\n"``,` `reinterpret_cast<void``*``>(reloc),` `reinterpret_cast<void``*``>(sym_addr` `+` `addend), sym_name);`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `=` `(sym_addr` `+` `addend);`
`break``;`
`case R_GENERIC_GLOB_DAT:`
`count_relocation(kRelocAbsolute);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO GLOB_DAT %16p <- %16p %s\n"``,` `reinterpret_cast<void``*``>(reloc),` `reinterpret_cast<void``*``>(sym_addr` `+` `addend), sym_name);`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `=` `(sym_addr` `+` `addend);`
`break``;`
`case R_GENERIC_RELATIVE:`
`count_relocation(kRelocRelative);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO RELATIVE %16p <- %16p\n"``,` `reinterpret_cast<void``*``>(reloc),` `reinterpret_cast<void``*``>(load_bias` `+` `addend));`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `=` `(load_bias` `+` `addend);`
`break``;`
`case R_GENERIC_IRELATIVE:`
`count_relocation(kRelocRelative);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO IRELATIVE %16p <- %16p\n"``,` `reinterpret_cast<void``*``>(reloc),` `reinterpret_cast<void``*``>(load_bias` `+` `addend));`
`{`
`/``*` `...` `*``/`
`/``*` `...` `*``/`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `=` `ifunc_addr;`
`}`
`break``;`
`#elif defined(__arm__)`
`case R_ARM_ABS32:`
`count_relocation(kRelocAbsolute);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO ABS %08x <- %08x %s"``, reloc, sym_addr, sym_name);`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `+``=` `sym_addr;`
`break``;`
`case R_ARM_REL32:`
`count_relocation(kRelocRelative);`
`MARK(rel``-``>r_offset);`
`TRACE_TYPE(RELO,` `"RELO REL32 %08x <- %08x - %08x %s"``,` `reloc, sym_addr, rel``-``>r_offset, sym_name);`
`*``reinterpret_cast<ElfW(Addr)``*``>(reloc)` `+``=` `sym_addr` `-` `rel``-``>r_offset;`
`break``;`
`case R_ARM_COPY:`
`/``*`
`*` `ET_EXEC` `is` `not` `supported so this should` `not` `happen.`
`*`
`*` `http:``/``/``infocenter.arm.com``/``help``/``topic``/``com.arm.doc.ihi0044d``/``IHI0044D_aaelf.pdf`
`*`
`*` `Section` `4.6``.``1.10` `"Dynamic relocations"`
`*` `R_ARM_COPY may only appear` `in` `executable objects where e_type` `is` `*` `set` `to ET_EXEC.`
`*``/`
`DL_ERR(``"%s R_ARM_COPY relocations are not supported"``, get_realpath());`
`return` `false;`
`#endif`
`default:`
`DL_ERR(``"unknown reloc type %d @ %p (%zu)"``,` `type``, rel, idx);`
`return` `false;`
`}`
`}`
`return` `true;`
`}`

- [x] 追溯`addend()` ,如果`rel`类型为`R_ARM_RELATIVE` 或者`R_ARM_IRELATIVE` 则返回第二个参数，否则返回0.

```
static ElfW(Addr) get_addend(ElfW(Rel)* rel, ElfW(Addr) reloc_addr) {
if (ELFW(R_TYPE)(rel->r_info) == R_GENERIC_RELATIVE ||
ELFW(R_TYPE)(rel->r_info) == R_GENERIC_IRELATIVE) {
return *reinterpret_cast<ElfW(Addr)*>(reloc_addr);
}
return 0;
}
```

- [x] `load_bias` 的计算方法

```
load_bias_ = reinterpret_cast<uint8_t*>(start) - addr;
```

`start` :=

```
start = ReserveAligned(mmap_hint, load_size_, kLibraryAlignment);
```

`addr` : = 0

```
size_t phdr_table_get_load_size(const ElfW(Phdr)* phdr_table, size_t phdr_count,
ElfW(Addr)* out_min_vaddr,
ElfW(Addr)* out_max_vaddr){
......
ElfW(Addr) min_vaddr = 0xffffffffffffffff;
ElfW(Addr) max_vaddr = 0;
/ /遍历程序头表所有的字段
for (size_t i = 0; i < phdr_count;
++i) {
const ElfW(Phdr)* phdr = &phdr_table[i];
if (phdr->p_type != PT_LOAD) {
continue;
}
found_pt_load = true;
if (phdr->p_vaddr < min_vaddr) {
min_vaddr = phdr->p_vaddr;
}
if (phdr->p_vaddr + phdr->p_memsz > max_vaddr) {
max_vaddr = phdr->p_vaddr + phdr->p_memsz;
}
}
if (!found_pt_load) {
min_vaddr = 0;
}
min_vaddr = PAGE_START(min_vaddr);
max_vaddr = PAGE_END(max_vaddr);
......
if (out_max_vaddr != nullptr) {
*out_max_vaddr = max_vaddr;
}
......
}
load_size_ = phdr_table_get_load_size(phdr_table_, phdr_num_, &min_vaddr);
uint8_t* addr = reinterpret_cast<uint8_t*>(min_vaddr);
```
