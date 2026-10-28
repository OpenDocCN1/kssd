
```c
int fd;
void *start;
struct stat sb;
fd = open("/system/bin/linker64", O_RDONLY);
/*打开/etc/passwd */
fstat(fd, &sb);
/* 取得文件大小 */
start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
__android_log_print(6, "r0ysue", "%p", start);
Elf64_Ehdr header;
memcpy(&header, start, sizeof(Elf64_Ehdr));
int secoff = header.e_shoff;
int secsize = header.e_shentsize;
int secnum = header.e_shnum;
int secstr = header.e_shstrndx;
Elf64_Shdr strtab;
memcpy(&strtab, (char *) start + secoff + secstr * secsize, sizeof(Elf64_Shdr));
int strtaboff = strtab.sh_offset;
char strtabchar[strtab.sh_size];
memcpy(&strtabchar, (char *) start + strtaboff, strtab.sh_size);
Elf64_Shdr enumsec;
int gotoff = 0;
int gotsize = 0;
int strtabsize = 0;
int stroff = 0;
for (int n = 0; n < secnum; n++) {
    memcpy(&enumsec, (char *) start + secoff + n * secsize, sizeof(Elf64_Shdr));
    if (strcmp(&strtabchar[enumsec.sh_name], ".symtab") == 0) {
        gotoff = enumsec.sh_offset;
        gotsize = enumsec.sh_size;
        __android_log_print(6, "r0ysue", "%x", gotsize);
    }
    if (strcmp(&strtabchar[enumsec.sh_name], ".strtab") == 0) {
        stroff = enumsec.sh_offset;
        strtabsize = enumsec.sh_size;
        __android_log_print(6, "r0ysue", "%x", stroff);
    }
}
int realoff = 0;
char relstr[strtabsize];
Elf64_Sym tmp;
memcpy(&relstr, (char *) start + stroff, strtabsize);

for (int n = 0; n < gotsize; n = n + sizeof(Elf64_Sym)) {
    memcpy(&tmp, (char *)start + gotoff + n, sizeof(Elf64_Sym));
    if(tmp.st_name != 0 && strstr(relstr + tmp.st_name, "soinfo_handles_map"))
        realoff = tmp.st_value;
}
```

## 总结

这样就完成了上面4个问题的解决，比较基础，主要是对elf文件格式的一个解析，我只是提供了一个例子，没有对数据进行加密，正常情况下我觉得还应该对dex中的so信息进行加密保护效果才更强，而且我的hook貌似不稳定，又崩溃的几率，感谢大家观看

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
