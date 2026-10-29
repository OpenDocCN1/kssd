
# [原创]适配古河大佬的注入工具到 AndroidN+

原注入工具 https://bbs.pediy.com/thread-141355.htm

做了两个改动：

1. 获取目标进程中`dlopen`等函数地址时，先获取下导出该函数的模块名称，原代码是默认`dlxxxx`函数通过`linker`导出，有极少设备是`libdl.so`或`libc.so`

原代码：

```
int inject_remote_process(...) {

    ...

    dlopen_addr = get_remote_addr( target_pid, linker_path, (void *)dlopen );

    dlsym_addr = get_remote_addr( target_pid, linker_path, (void *)dlsym );

    dlclose_addr = get_remote_addr( target_pid, linker_path, (void *)dlclose );

    ...

}
```

修改后代码：

```
int inject_remote_process(...) {

    ...

    dlopen_addr = get_remote_addr(target_pid, (void *)dlopen);

    dlsym_addr = get_remote_addr(target_pid, (void *)dlsym );

    dlclose_addr = get_remote_addr(target_pid, (void *)dlclose);

    dlerror_addr = get_remote_addr(target_pid, (void *)dlerror);

    ...

}

void* get_remote_addr(pid_t target_pid, void* local_addr) {

    ...

    get_module_path((unsigned long)local_addr, module_path);

    local_handle = get_module_base(-1, module_path);

    remote_handle = get_module_base(target_pid, module_path);

    ...

}
```

2. `androidN`以上设备对`dlopen`的使用有权限检查，可以使用另外一个函数加载`so`，代码如下：

```
static void init_dlopen_ext_offset() {

    FILE *fp = NULL;

    if (!(fp = fopen(linker_path, "rb")))  {

        DEBUG_PRINT("[init_dlopen_ext_offset]Unable to open %s\n", linker_path);

        return;

    }

    fseek(fp, 0, SEEK_END);

    int size = ftell(fp);

    fseek(fp, 0, SEEK_SET);

    char *buffer = (char*)malloc(size);

    if (fread(buffer, 1, size, fp) != size) {

        DEBUG_PRINT("fread error\n");

        return;

    }

    fclose(fp);

    unsigned long symstr_off = 0, symtab_off = 0, symtab_size = 0;

    unsigned long symtab_entsize = 0, symtab_count = 0;

    const elf_header_t* eh  = (elf_header_t*)buffer;

    const elf_sheader_t* esh = (elf_sheader_t*)(buffer + eh->shoff);

    char* section_str = esh[eh->shstrndx].sh_offset + buffer;

    for (int i = 0; i < eh->shnum; i++) {

        char* sname = esh[i].sh_name + section_str;

        if (strcmp(sname, ".symtab") == 0) {

            symtab_off = esh[i].sh_offset; 

            symtab_size = esh[i].sh_size;

            symtab_entsize = esh[i].sh_entsize;

            symtab_count = symtab_size / symtab_entsize;

            DEBUG_PRINT("[init_dlopen_ext_offset]: symtab offset = %lx, count=%lx, index= %d\n", 

                symtab_off, symtab_count, i);

        }

        if (strcmp(sname, ".strtab") == 0) {

            symstr_off = esh[i].sh_offset;

            DEBUG_PRINT("[init_dlopen_ext_offset] symstr offset = %lx, index = %d\n", symstr_off, i);

        }
    }
    if(!symtab_off) {

        DEBUG_PRINT("[init_dlopen_ext_offset] can't find symtab from sections\n");

    }
    elf_sym_t* edt = (elf_sym_t*)(buffer + symtab_off);
    const char* name_dlopen_ext_N = "__dl__ZL10dlopen_extPKciPK17android_dlextinfoPv";

    const char* name_dlopen_ext_O = "__dl__ZL10dlopen_extPKciPK17android_dlextinfoPKv";

    const char* name_dlopen_ext_P = "__dl___loader_android_dlopen_ext";
    for(int i = 0 ; i < symtab_count; i++) {

        uint8_t st_type = ELF32_ST_TYPE(edt[i].info);

        char* st_name = buffer + symstr_off + edt[i].name;

        // DEBUG_PRINT("[init_dlopen_ext_offset] walk sym name:%s, value:%x\n", st_name, edt[i].value);

        if (st_type == STT_FUNC && edt[i].size) {

            if(strcmp(st_name, name_dlopen_ext_N) == 0) {

                dlopen_ext_offset = edt[i].value;

                DEBUG_PRINT("[init_dlopen_ext_offset] find dlopen_ext_N: %x\n", dlopen_ext_offset);

                break;

            }

            else if (strcmp(st_name, name_dlopen_ext_O) == 0) {

                dlopen_ext_offset = edt[i].value;

                DEBUG_PRINT("[init_dlopen_ext_offset] find dlopen_ext_O: %x\n", dlopen_ext_offset);

                break;

            }

            else if (strcmp(st_name, name_dlopen_ext_P) == 0) {

                dlopen_ext_offset = edt[i].value;

                DEBUG_PRINT("[init_dlopen_ext_offset] find dlopen_ext_P: %x\n", dlopen_ext_offset);

                break;

            }

        }

    }
    free(buffer);

}
```

完整代码：`a3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3&6Y4M7r3!0^5K9h3&6Q4x3V1k6A6L8X3A6W2j5%4c8Q4y4h3k6V1k6i4R3`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
