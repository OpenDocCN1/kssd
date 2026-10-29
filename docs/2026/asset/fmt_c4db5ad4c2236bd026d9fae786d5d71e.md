
```
int * enumsym(const char * lib,int size,int start1){
  int fd;
  void * start;
  struct stat sb;
  fd = open(lib, O_RDONLY);
  fstat(fd, &sb);
  start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
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
    }
    if (strcmp(&strtabchar[enumsec.sh_name], ".strtab") == 0) {
      stroff = enumsec.sh_offset;
      strtabsize = enumsec.sh_size;
    }
  }
  int realoff=0;
  char relstr[strtabsize];
  Elf64_Sym tmp;
  memcpy(&relstr, (char *) start + stroff, strtabsize);
  int * sdc = static_cast<int *>(malloc(gotsize / sizeof(Elf64_Sym)*sizeof(int *))); //存储返回值数组
  for (int n = 0; n < gotsize; n = n + sizeof(Elf64_Sym)) {
    memcpy(&tmp, (char *)start + gotoff + n, sizeof(Elf64_Sym));
    //        __android_log_print(6,"r0ysue","%x",gotoff+n);
    sdc[n/sizeof(Elf64_Sym)]=tmp.st_value;
    if (tmp.st_value>size||tmp.st_value<start1)
      sdc[n/sizeof(Elf64_Sym)]=start1;
  }
  return sdc;
}
int getsymsize(const char * lib){
  int fd;
  void * start;
  struct stat sb;
  fd = open(lib, O_RDONLY);
  /*打开 /etc/passwd */
  fstat(fd, &sb);
  /* 取得文件大小 */
  start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
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
    }
    if (strcmp(&strtabchar[enumsec.sh_name], ".strtab") == 0) {
      stroff = enumsec.sh_offset;
      strtabsize = enumsec.sh_size;
    }
  }
  return gotsize/sizeof(Elf64_Sym);
}
```

之后把这两个函数调用写入线程函数就好了,我这里直接用`0x25000`了通过`libart.so`的程序头可以解析出来，每10秒检测一次

```
int * so ;
int size ; //写到全局变量里面循环就不会多次调用了
void anti1(){
  while (1) {
    pthread_mutex_lock(&mutex);
    for (int n = 0; n < size; n++) {
      //    __android_log_print(6,"r0ysue","i find frida %p %x %x",(long long) reinterpret_cast<long>((char *) startr - 0x25000 + so[n]),so[n],n);
      long long as = *(long long *) reinterpret_cast<long>((char *) startr - 0x25000 + so[n]);
      //        __android_log_print(6,"r0ysue","i find frida %d %x %p",n,so[n],as);
      if (as == 0xd61f020058000050) {
        __android_log_print(6,"r0ysue","i find frida %p", (long long *) reinterpret_cast<long>((char *) startr - 0x25000 + so[n]));
      }
    }
    sleep(5);
    pthread_mutex_unlock(&mutex);
  }
}
void __init(){
    so = enumsym("/system/lib64/libart.so", (long) end - (long) startr,0x25000);
    size = getsymsize("/system/lib64/libart.so");
}
```

看一下结果，效果不错就是检测的慢一点但是也达到了预定的目标

![](https://bbs.kanxue.com/upload/attach/202110/799845_FUHGW4ASU2Z6TWH.png)

### 检测 `libnative-lib` 是否使用 `inlinehook`

接着我们继续搞 `libnative-lib`，这个由于目录特殊性所以我们很难找到它的目录从节头里面搜索符号地址，所以我们只能用程序头搜索它的导出函数，这个之前的文章也讲过，这里就直接贴代码了,这里注意一点，如何确定符号表的大小，仔细观察就知道符号表后面就是字符串表，那么用符号表偏移减去字符串表偏移就是符号表的大小
