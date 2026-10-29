
# 从inlinehook角度检测frida

- [从inlinehook角度检测frida](#从inlinehook角度检测frida)
- [总述](#总述)
- [frida inline hook写法](#frida-inline-hook写法)
- [检测libart是否使用inlinehook](#检测libart是否使用inlinehook)
- [检测libnative-lib是否使用inlinehook](#检测libnative-lib是否使用inlinehook)
- [从java hook 的角度检测frida](#从java-hook-的角度检测frida)
- [总结](#总结)

## 从inlinehook角度检测frida

### 总述

在使用`frida`的过程中，在我的认知里其实`frida`只做了2件事情，一件是注入，一件是`hook`，作为注入的特征我们可以通过`ptrace(PTRACE_TRACEME,NULL,0,0)`，或者从文件里面索引有关`frida`字符串这样的方式来检测`frida`，那么作为`hook`特征是否也能够检测`frida`呢？

### frida inline hook写法

答案是肯定的，既然`frida`是能够定向跳转从而更改内存中的代码，那么一定是用到了`inlinehook`的技术，那么它是怎么使用的`inlinehook`呢，我们可以使用`ida`（当然直接看`frida`源码，但是那样真的很麻烦就连我自己写的`inlinehook`都很难快速的分析清楚）去看一下，我随便`hook`了一个函数比如`art::ClassLinker::LoadMethod`，注意要先附加`frida`然后再使用`ida`链接，否则`frida`无法附加已经被`ptrace`附加的进程，然后，结果如下图

```
function main() {
var libart = Module.enumerateSymbols("libart.so");
var addr = NULL;
for (var n in libart) {
if (libart[n].name.indexOf("lassLinker10LoadMethodERKNS") >= 0) {
addr = libart[n].address;
break;
}
}
Interceptor.attach(addr, {
onEnter: function (arg) {
}
})
}
```

![](upload/attach/202110/799845_DS8CBPYPWCHXD4Q.png)

可以看到`frida`的`hook`原理就是将函数的开头改成这样的一串16进制，`0xd61f020058000050`和我之前写的`inlinehook`差不多，只是它用了`x16`寄存器我用了`x17`寄存器，那么这里就有一个思路了，我们能否通过`so`中是否有这一段来判断用户使用了`frida`呢？答案是肯定的，但是需要我们稍作调整，就是在每个函数的开头检测是否有`0xd61f020058000050`这样的一段代码，创建线程即可，那么下面就开始实现，首先试一下单个函数能不能成功，我使用了上篇文章提到的，提取符号首地址的方式`findsym`

```c
void anti1(){
while (1) {
sleep(1);
int so = findsym("/system/lib64/libart.so",
"\_ZN3art11ClassLinker10LoadMethodERKNS\_7DexFileERKNS\_21ClassDataItemIteratorENS\_6HandleINS\_6mirror5ClassEEEPNS\_9ArtMethodE");
long long as = \*(long long \*) reinterpret\_cast<long>((char \*) startr - 0x25000 + so);
if(as==0xd61f020058000050) {
\_\_android\_log\_print(6, "r0ysue", "i find frida %p",(long long \*) reinterpret\_cast<long>((char \*) startr - 0x25000 + so));
}
}
}
pthread\_create(&thread, nullptr, reinterpret\_cast<void \*(\*)(void \*)>(anti1), nullptr);
```

![](upload/attach/202110/799845_JFRYNUCYFSTQW7G.png)

### 检测libart是否使用inlinehook

可以看到当我以`attach`的方式链接上去的时候，成功的检测到了`frida`，那么其实我们就可以遍历符号表来获得一个`so`中所有的函数首地址来检测是否使用了`frida`，由于之前也说过，某些函数不在导出表中，而在符号表中，所以可能没有办法通过程序头获取，所以系统`so`这方面用节头获取，自身`so`方面用程序头获取（自身`so`路径不太好确定，所以直接用内存中的就好），这里以`libart.so`和`libnative-lib.so`来做一个例子。首先就是`libart.so`，这里我封装了2个函数一个是获得所有符号的首地址，一个是获得大小都是通过节头表索引得到的，其中`enumsym`这个函数用到了2个`int`，第一个`size`是最大值，第二个`start1`是最小值，通过外部传入，因为有些符号表的值大于文件所以容易溢出

```c
int\* enumsym(const char\* lib,int size,int start1){
int fd;
void \*start;
struct stat sb;
fd = open(lib, O\_RDONLY);
fstat(fd, &sb);
start = mmap(NULL, sb.st\_size, PROT\_READ, MAP\_PRIVATE, fd, 0);
Elf64\_Ehdr header;
memcpy(&header, start, sizeof(Elf64\_Ehdr));
int secoff = header.e\_shoff;
int secsize = header.e\_shentsize;
int secnum = header.e\_shnum;
int secstr = header.e\_shstrndx;
Elf64\_Shdr strtab;
memcpy(&strtab, (char \*) start + secoff + secstr \* secsize, sizeof(Elf64\_Shdr));
int strtaboff = strtab.sh\_offset;
char strtabchar[strtab.sh\_size];
memcpy(&strtabchar, (char \*) start + strtaboff, strtab.sh\_size);
Elf64\_Shdr enumsec;
int gotoff = 0;
int gotsize = 0;
int strtabsize = 0;
int stroff = 0;
for (int n = 0; n < secnum; n++) {
memcpy(&enumsec, (char \*) start + secoff + n \* secsize, sizeof(Elf64\_Shdr));
if (strcmp(&strtabchar[enumsec.sh\_name], ".symtab") == 0) {
gotoff = enumsec.sh\_offset;
gotsize = enumsec.sh\_size;
}
if (strcmp(&strtabchar[enumsec.sh\_name], ".strtab") == 0) {
stroff = enumsec.sh\_offset;
strtabsize = enumsec.sh\_size;
}
}
int realoff=0;
char relstr[strtabsize];
Elf64\_Sym tmp;
memcpy(&relstr, (char \*) start + stroff, strtabsize);
int\* sdc= static\_cast<int \*>(malloc(gotsize / sizeof(Elf64\_Sym)\*sizeof(int \*)));//存储返回值数组
for (int n = 0; n < gotsize; n = n + sizeof(Elf64\_Sym)) {
memcpy(&tmp, (char \*)start + gotoff+n, sizeof(Elf64\_Sym));
// \_\_android\_log\_print(6, "r0ysue", "%x",gotoff+n);
sdc[n/sizeof(Elf64\_Sym)]=tmp.st\_value;
if(tmp.st\_value>size||tmp.st\_value<start1)
sdc[n/sizeof(Elf64\_Sym)]=start1;
}
return sdc;
}
int getsymsize(const char\* lib){
int fd;
void \*start;
struct stat sb;
fd = open(lib, O\_RDONLY); /*打开/etc/passwd */
fstat(fd, &sb); /* 取得文件大小 */
start = mmap(NULL, sb.st\_size, PROT\_READ, MAP\_PRIVATE, fd, 0);
Elf64\_Ehdr header;
memcpy(&header, start, sizeof(Elf64\_Ehdr));
int secoff = header.e\_shoff;
int secsize = header.e\_shentsize;
int secnum = header.e\_shnum;
int secstr = header.e\_shstrndx;
Elf64\_Shdr strtab;
memcpy(&strtab, (char \*) start + secoff + secstr \* secsize, sizeof(Elf64\_Shdr));
int strtaboff = strtab.sh\_offset;
char strtabchar[strtab.sh\_size];
memcpy(&strtabchar, (char \*) start + strtaboff, strtab.sh\_size);
Elf64\_Shdr enumsec;
int gotoff = 0;
int gotsize = 0;
int strtabsize = 0;
int stroff = 0;
for (int n = 0; n < secnum; n++) {
memcpy(&enumsec, (char \*) start + secoff + n \* secsize, sizeof(Elf64\_Shdr));
if (strcmp(&strtabchar[enumsec.sh\_name], ".symtab") == 0) {
gotoff = enumsec.sh\_offset;
gotsize = enumsec.sh\_size;
}
if (strcmp(&strtabchar[enumsec.sh\_name], ".strtab") == 0) {
stroff = enumsec.sh\_offset;
strtabsize = enumsec.sh\_size;
}
}
return gotsize/sizeof(Elf64\_Sym);
}
```

之后把这两个函数调用写入线程函数就好了,我这里直接用`0x25000`了通过`libart.so`的程序头可以解析出来，每10秒检测一次

```c
int \*so ;
int size ;//写到全局变量里面循环就不会多次调用了
void anti1(){
while (1) {
pthread\_mutex\_lock(&mutex);
for (int n = 0; n < size; n++) {
// \_\_android\_log\_print(6, "r0ysue", "i find frida %p %x %x",(long long ) reinterpret\_cast<long>((char \*) startr - 0x25000 + so[n]),so[n],n);
long long as = \*(long long \*) reinterpret\_cast<long>((char \*) startr - 0x25000 + so[n]);
// \_\_android\_log\_print(6, "r0ysue", "i find frida %d %x %p",n,so[n],as);
if (as == 0xd61f020058000050) {
\_\_android\_log\_print(6, "r0ysue", "i find frida %p",
(long long \*) reinterpret\_cast<long>((char \*) startr - 0x25000 +
so[n]));
}
}
sleep(5);
pthread\_mutex\_unlock(&mutex);
}
}
void \_\_init(){
so = enumsym("/system/lib64/libart.so", (long) end - (long) startr,0x25000);
size = getsymsize("/system/lib64/libart.so");
}
```

看一下结果，效果不错就是检测的慢一点但是也达到了预定的目标

![](upload/attach/202110/799845_FUHGW4ASU2Z6TWH.png)

### 检测libnative-lib是否使用inlinehook

接着我们继续搞`libnative-lib`，这个由于目录特殊性所以我们很难找到它的目录从节头里面搜索符号地址，所以我们只能用程序头搜索它的导出函数，这个之前的文章也讲过，这里就直接贴代码了,这里注意一点，如何确定符号表的大小，仔细观察就知道符号表后面就是字符串表，那么用符号表偏移减去字符串表偏移就是符号表的大小
