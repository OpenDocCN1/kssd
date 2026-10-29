
这是一个用于解析 ELF 文件（可执行和可链接格式）的程序头（Program Header）信息的 C 语言函数。它接收一个指向 ELF 文件内存映射的缓冲区指针，然后解析并打印出文件类型、入口点、程序头表以及段到节（Section to Segment）的映射关系。

以下是该函数的代码实现：

```c
void programheader(const char *pbuff)
{
    Elf64_Ehdr* pfilehead = (Elf64_Ehdr*)pbuff;
    Elf64_Phdr* pproheader = (Elf64_Phdr*)(pbuff + pfilehead->e_phoff);
    Elf64_Shdr* psecheader = (Elf64_Shdr*)(pbuff + pfilehead->e_shoff);
    Elf64_Shdr* pshstr = (Elf64_Shdr*)(psecheader + pfilehead->e_shstrndx);
    char* pstrbuff = (char*)(pbuff + pshstr->sh_offset);

    printf("Elf 文件类型是");
    switch(pfilehead->e_type)
    {
        case 0:
            printf(" No file type\r\n");
            break;
        case 1:
            printf(" Relocatable file\r\n");
            break;
        case 2:
            printf(" Executable file\r\n");
            break;
        case 3:
            printf(" Shared object file\r\n");
            break;
        case 4:
            printf(" Core file\r\n");
            break;
        default:
            printf(" ERROR\r\n");
            break;
    }

    printf("入口点位置 0X%0lX\r\n", pfilehead->e_entry);
    printf("共有 %d 程序头, 偏移位置 %lu\r\n\r\n", pfilehead->e_phnum, pfilehead->e_phoff);

    printf("Program Headers:\r\n");
    printf("  %-14s  %-16s  %-16s  %-16s\r\n", "Type", "Offset", "VirtAddr", "PhysAddr");
    printf("  %-14s  %-16s  %-16s  %-6s  %-6s\r\n", "", "FileSiz", "MemSiz", "Flags", "Align");

    for(int i=0;i<pfilehead->e_phnum;++i)
    {
        //type
        switch(pproheader[i].p_type)
        {
            case PT_NULL:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_LOAD:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "LOAD", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_DYNAMIC:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "DYNAMIC", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_INTERP:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "INTERP", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_NOTE:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "NOTE", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_SHLIB:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "SHLIB", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_PHDR:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "PHDR", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_TLS:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "TLS", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_NUM:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "NUM", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_GNU_EH_FRAME:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "GNU_EH_FRAME", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_GNU_RELRO:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "GNU_RELRO", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            case PT_GNU_STACK:
                printf("  %-14s  %016lX  %016lX  %016lX\r\n  %-14s  %016lX  %016lX  ", "GNU_STACK", pproheader[i].p_offset, pproheader[i].p_vaddr,
                       pproheader[i].p_paddr, "", pproheader[i].p_filesz, pproheader[i].p_memsz);break;
            default:
                break;
        }

        //
        switch(pproheader[i].p_flags)
        {
            case PF_X:
                printf("%-6s  %-lX\r\n", "  E", pproheader[i].p_align);break;
            case PF_W:
                printf("%-6s  %-lX\r\n", " W ", pproheader[i].p_align);break;
            case PF_R:
                printf("%-6s  %-lX\r\n", "R  ", pproheader[i].p_align);break;
            case PF_X|PF_W:
                printf("%-6s  %-lX\r\n", " WE", pproheader[i].p_align);break;
            case PF_X|PF_R:
                printf("%-6s  %-lX\r\n", "R E", pproheader[i].p_align);break;
            case PF_W|PF_R:
                printf("%-6s  %-lX\r\n", "RW ", pproheader[i].p_align);break;
            case PF_X|PF_R|PF_W:
                printf("%-6s  %-lX\r\n", "RWE", pproheader[i].p_align);break;
            default:
                printf("\r\n");
                break;
        }

        if(PT_INTERP == pproheader[i].p_type)
            printf("      [Requesting program interpreter: %s]\r\n", (char*)(pbuff + pproheader[i].p_offset));
    }

    printf("\r\n Section to Segment mapping:\r\n");
    printf("  段节...\r\n");
    for(int i=0;i<pfilehead->e_phnum;++i)
    {
        printf("   %-7d", i);
        for(int n = 0;n<pfilehead->e_shnum;++n)
        {
            Elf64_Off temp = psecheader[n].sh_addr + psecheader[n].sh_size;
            if((psecheader[n].sh_addr>pproheader[i].p_vaddr && psecheader[n].sh_addr<pproheader[i].p_vaddr + pproheader[i].p_memsz)  ||
                    (temp > pproheader[i].p_vaddr && temp<=pproheader[i].p_vaddr + pproheader[i].p_memsz))
            {
                printf("%s ", (char*)(psecheader[n].sh_name + pstrbuff));
            }
        }
        printf("\r\n");
    }
}
```

## 效果展示

![](https://bbs.kanxue.com/upload/attach/202006/856851_4R6KCVGPA42SJBC.jpg)

## readelf 对比

![](https://bbs.kanxue.com/upload/attach/202006/856851_KH3RYDZJUQMEKU9.jpg)

最后还有一个重定位表的打印函数，暂时不介绍了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [jtElf](javascript:void(0);)（37.49kb，240次下载）
- [jtElf.c](javascript:void(0);)（23.03kb，380次下载）
- [testForargv](javascript:void(0);)（18.65kb，207次下载）
