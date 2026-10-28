
我们可以看到每个重定位信息都对应一个`Elf32_Sym`符号信息，并且`r_info`提供了重定位类型和对应的符号表索引。

如果有兴趣，可以自己查看每个符号是怎么修复的。

接下来我们看一个查找符号地址的函数`soinfo_do_lookup`。

## 查找符号地址：`soinfo_do_lookup`

```
static Elf32_Sym* soinfo_do_lookup(soinfo* si, const char* name, soinfo** lsi, soinfo* needed[]) {

    unsigned elf_hash = elfhash(name);

    Elf32_Sym* s = NULL;



    if (si != NULL && somain != NULL) {



        /*

         * Local scope is executable scope. Just start looking into it right away

         * for the shortcut.

         */



        if (si == somain) {

            s = soinfo_elf_lookup(si, elf_hash, name);

            if (s != NULL) {

                *lsi = si;

                goto done;

            }

        } else {

            /* Order of symbol lookup is controlled by DT_SYMBOLIC flag */



            /*

             * If this object was built with symbolic relocations disabled, the

             * first place to look to resolve external references is the main

             * executable.

             */



            if (!si->has_DT_SYMBOLIC) {

                DEBUG("%s: looking up %s in executable %s",

                      si->name, name, somain->name);

                s = soinfo_elf_lookup(somain, elf_hash, name);

                if (s != NULL) {

                    *lsi = somain;

                    goto done;

                }

            }



            /* Look for symbols in the local scope (the object who is

             * searching). This happens with C++ templates on i386 for some

             * reason.

             *

             * Notes on weak symbols:

             * The ELF specs are ambiguous about treatment of weak definitions in

             * dynamic linking.  Some systems return the first definition found

             * and some the first non-weak definition.   This is system dependent.

             * Here we return the first definition found for simplicity.  */



            s = soinfo_elf_lookup(si, elf_hash, name);//首先在自身模块地址获取

            if (s != NULL) {

                *lsi = si;

                goto done;

            }



            /*

             * If this object was built with -Bsymbolic and symbol is not found

             * in the local scope, try to find the symbol in the main executable.

             */



            if (si->has_DT_SYMBOLIC) {

                DEBUG("%s: looking up %s in executable %s after local scope",

                      si->name, name, somain->name);

                s = soinfo_elf_lookup(somain, elf_hash, name);//在somain获取

                if (s != NULL) {

                    *lsi = somain;

                    goto done;

                }

            }

        }

    }



    /* Next, look for it in the preloads list */

    for (int i = 0; gLdPreloads[i] != NULL; i++) {//在预加载的模块中查找

        s = soinfo_elf_lookup(gLdPreloads[i], elf_hash, name);

        if (s != NULL) {

            *lsi = gLdPreloads[i];

            goto done;

        }

    }



    for (int i = 0; needed[i] != NULL; i++) {//在加载进来的模块中查找

        DEBUG("%s: looking up %s in %s",

              si->name, name, needed[i]->name);

        s = soinfo_elf_lookup(needed[i], elf_hash, name);

        if (s != NULL) {

            *lsi = needed[i];

            goto done;

        }

    }



done:

    if (s != NULL) {

        TRACE_TYPE(LOOKUP, "si %s sym %s s->st_value = 0x%08x, "

                   "found in %s, base = 0x%08x, load bias = 0x%08x",

                   si->name, name, s->st_value,

                   (*lsi)->name, (*lsi)->base, (*lsi)->load_bias);

        return s;

    }



    return NULL;

}
```

符号会被`elfhash`经过 HASH 计算返回一个整数，然后通过函数`soinfo_elf_lookup`返回符号的地址。

我们可以看到其查找的步骤：

1. 在自身模块获取符号信息
2. 在`somain`模块获取符号信息
3. 在`gLdPreloads`数组中的模块获取符号信息
4. 在`DT_NEEDED`加载进来的模块数组中查找

其中查找的函数为`soinfo_elf_lookup`。

```
unsigned elfhash(const char* _name) {
    const unsigned char* name = (const unsigned char*) _name;

    unsigned h = 0, g;



    while(*name) {

        h = (h << 4) + *name++;

        g = h & 0xf0000000;

        h ^= g;

        h ^= g >> 24;

    }

    return h;

}



static Elf32_Sym* soinfo_elf_lookup(soinfo* si, unsigned hash, const char* name) {

    Elf32_Sym* symtab = si->symtab;

    const char* strtab = si->strtab;



    TRACE_TYPE(LOOKUP, "SEARCH %s in %s@0x%08x %08x %d",

               name, si->name, si->base, hash, hash % si->nbucket);



    for (unsigned n = si->bucket[hash % si->nbucket]; n != 0; n = si->chain[n]) {

        Elf32_Sym* s = symtab + n;

        if (strcmp(strtab + s->st_name, name)) continue;



            /* only concern ourselves with global and weak symbol definitions */

        switch(ELF32_ST_BIND(s->st_info)){

        case STB_GLOBAL:

        case STB_WEAK:

            if (s->st_shndx == SHN_UNDEF) {

                continue;

            }



            TRACE_TYPE(LOOKUP, "FOUND %s in %s (%08x) %d",

                       name, si->name, s->st_value, s->st_size);

            return s;

        }

    }



    return NULL;

}
```

这里就是通过`DT_HASH`对应的那张表来查找指定符号了。

这里我列一下`DT_HASH`动态节表对应的结构如下。

```
{
    unsigned nbucket;
    unsigned nchain;
    unsigned bucket[nbucket];
    unsigned chain[nchain];
}
```

`dlsym` 其实最终就是通过这个函数返回符号地址的。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-2-22 17:54 被我是七叔编辑，原因：修改标题
