
上面这个函数就是在处理重定位相关的信息了，我们看到从Dynamic段中拿到的跟重定位相关的表，会经过这个函数来处理，将So本身的地址引用进行重定位，使其可以正常运行。其实在32位So中，需要处理的重定位类型并不是很多，就4种类型需要处理，而且还有两种处理方式相同

现在So就重定位完成了，现在So已经就可以跑起来了，下面我们就来看看从Dynamic段中拿到的各种初始化函数是怎么处理的，还记得吧

我们回到`do_dlopen`函数

```c
static int soinfo_relocate(soinfo* si, Elf32_Rel* rel, unsigned count, soinfo* needed[]) {
    // 拿到符号表和字符串表，定义一些变量
    Elf32_Sym* symtab = si->symtab;
    const char* strtab = si->strtab;
    Elf32_Sym* s;
    Elf32_Rel* start = rel;
    soinfo* lsi;
    // 遍历重定位表
    for (size_t idx = 0; idx < count; ++idx, ++rel) {
        // 拿到重定位类型
        unsigned type = ELF32_R_TYPE(rel->r_info);
        // 拿到重定位符号
        unsigned sym = ELF32_R_SYM(rel->r_info);
        // 计算需要重定位的地址
        Elf32_Addr reloc = static_cast<Elf32_Addr>(rel->r_offset + si->load_bias);
        Elf32_Addr sym_addr = 0;
        char* sym_name = NULL;

        DEBUG("Processing '%s' relocation at index %d", si->name, idx);
        if (type == 0) {
        // R_*_NONE
        continue;
        }
        if (sym != 0) {
        // 如果sym不为0，说明重定位需要用到符号，先来找符号，拿到符号名
        sym_name = (char*)(strtab + symtab[sym].st_name);
        // 下面这个函数大家有兴趣的可以看一下，就是根据符号名来从依赖so中查找所需要的符号
        s = soinfo_do_lookup(si, sym_name, &lsi, needed);
        if (s == NULL) {
        // 如果没找到，就用本身So的符号
        s = &symtab[sym];
        if (ELF32_ST_BIND(s->st_info) != STB_WEAK) {
        DL_ERR("cannot locate symbol \"%s\" referenced by \"%s\"...", sym_name, si->name);
        return -1;
        }
        switch (type) {
        // 下面是如果符号不为外部符号，就只能为以下几种类型
        #if defined(ANDROID_ARM_LINKER)
        case R_ARM_JUMP_SLOT:
        case R_ARM_GLOB_DAT:
        case R_ARM_ABS32:
        case R_ARM_RELATIVE: /* Don't care. */
        #endif /* ANDROID_*_LINKER */
        /* sym_addr was initialized to be zero above
        or
        relocation
        code below does
        not
        care about value of sym_addr.
        No need to do anything.
        */
        break;

        #if defined(ANDROID_ARM_LINKER)
        case R_ARM_COPY:
        /* Fall through.  Can't really copy
        if
        weak symbol
        is
        not
        found
        in
        run-time.
        */
        #endif /* ANDROID_ARM_LINKER */
        default:
        DL_ERR("unknown weak reloc type %d @ %p (%d)", type, rel, (int) (rel - start));
        return -1;
        }
        }
        else {
        // 如果我们找到了外部符号，取到外部符号的地址
        sym_addr = static_cast<Elf32_Addr>(s->st_value + lsi->load_bias);
        }
        count_relocation(kRelocSymbol);
        }
        else {
        // 如果sym为0，就说明当前重定位用不到符号
        s = NULL;
        }

        // 下面根据重定位类型来处理重定位
        switch(type){
        #if defined(ANDROID_ARM_LINKER)
        case R_ARM_JUMP_SLOT:
        count_relocation(kRelocAbsolute);
        MARK(rel->r_offset);
        TRACE_TYPE(RELO, "RELO JMP_SLOT %08x <- %08x %s", reloc, sym_addr, sym_name);
        // 直接将需要重定位的地方，写入获取到的符号地址
        *reinterpret_cast<Elf32_Addr*>(reloc) = sym_addr;
        break;
        case R_ARM_GLOB_DAT:
        count_relocation(kRelocAbsolute);
        MARK(rel->r_offset);
        TRACE_TYPE(RELO, "RELO GLOB_DAT %08x <- %08x %s", reloc, sym_addr, sym_name);
        // 直接将需要重定位的地方，写入获取到的符号地址，与R_ARM_JUMP_SLOT相同
        *reinterpret_cast<Elf32_Addr*>(reloc) = sym_addr;
        break;
        case R_ARM_ABS32:
        count_relocation(kRelocAbsolute);
        MARK(rel->r_offset);
        TRACE_TYPE(RELO, "RELO ABS %08x <- %08x %s", reloc, sym_addr, sym_name);
        // 先读出需要重定位地方的数据，将其和符号地址相加，写入需要重定位的地方
        *reinterpret_cast<Elf32_Addr*>(reloc) += sym_addr;
        break;
        case R_ARM_REL32:
        count_relocation(kRelocRelative);
        MARK(rel->r_offset);
        TRACE_TYPE(RELO, "RELO REL32 %08x <- %08x - %08x %s", reloc, sym_addr, rel->r_offset, sym_name);
        // 先读出需要重定位地方的数据，将其和符号地址相加，再与重定位的地址相减，重定位的写入需要重定位的地方。此处Unidbg并未处理，也可忽略，应该是用不到的
        *reinterpret_cast<Elf32_Addr*>(reloc) += sym_addr - rel->r_offset;
        break;
        #endif /* ANDROID_*_LINKER */

        #if defined(ANDROID_ARM_LINKER)
        case R_ARM_RELATIVE:
        #endif /* ANDROID_*_LINKER */
        count_relocation(kRelocRelative);
        MARK(rel->r_offset);
        if (sym) {
        DL_ERR("odd RELATIVE form...");
        return -1;
        }
        TRACE_TYPE(RELO, "RELO RELATIVE %08x <- +%08x", reloc, si->base);
        // 先读出需要重定位地方的数据，将其和So的基址相加，写入需要重定位的地方
        *reinterpret_cast<Elf32_Addr*>(reloc) += si->base;
        break;

        #ifdef ANDROID_ARM_LINKER
        case R_ARM_COPY:
        // .. 进行了一些错误处理
        break;
        #endif /* ANDROID_ARM_LINKER */

        default:
        DL_ERR("unknown reloc type %d @ %p (%d)", type, rel, (int) (rel - start));
        return -1;
        }
    }
    return 0;
}
```

此时我们的`find_library`函数已经处理完了，So已经被装载且链接过了，最后一步它调用了`soinfo`的`CallConstructors`函数，我们来看看这个函数处理了什么

```c
soinfo* do_dlopen(const char* name, int flags) {
    if ((flags & ~(RTLD_NOW|RTLD_LAZY|RTLD_LOCAL|RTLD_GLOBAL)) != 0) {
        DL_ERR("invalid flags to dlopen: %x", flags);
        return NULL;
    }
    set_soinfo_pool_protection(PROT_READ | PROT_WRITE);
    soinfo* si = find_library(name);
    if (si != NULL) {
        si->CallConstructors();
    }
    set_soinfo_pool_protection(PROT_READ);
    return si;
}
```

```c
void soinfo::CallConstructors() {
    if (constructors_called) {
        return;
    }
    constructors_called = true;

    if ((flags & FLAG_EXE) == 0 && preinit_array != NULL) {
    // The GNU dynamic linker silently ignores these, but we warn the developer.
    PRINT("\"%s\": ignoring %d-entry DT_PREINIT_ARRAY in shared library!", name, preinit_array_count);
    }

    // 如果Dynamic段不为空，先处理依赖库的初始化
    if (dynamic != NULL) {
        for (Elf32_Dyn* d = dynamic; d->d_tag != DT_NULL; ++d) {
            if (d->d_tag == DT_NEEDED) {
                const char* library_name = strtab + d->d_un.d_val;
                TRACE("\"%s\": calling constructors in DT_NEEDED \"%s\"", name, library_name);
                find_loaded_library(library_name)->CallConstructors();
            }
        }
    }
    TRACE("\"%s\": calling constructors", name);
    // 我们来看下面一句英文注释，非常重要。他说如果DT_INIT和DT_INIT_ARRAY都存在，DT_INIT应该在DT_INIT_ARRAY之前被调用
    // DT_INIT should be called before DT_INIT_ARRAY if both are present.
    // 下面就是在调用两者，CallArray只是在循环调用CallFunction，我们看一下CallFunction
    CallFunction("DT_INIT", init_func);
    CallArray("DT_INIT_ARRAY", init_array, init_array_count, false);
}
```
