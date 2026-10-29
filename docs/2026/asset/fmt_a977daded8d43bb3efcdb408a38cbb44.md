
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
进入 `/path/to/xxx` 目录，找到 `xxx.json`。

### 步骤
1. `（1）导包，将SandHook中cpp文件夹下的包全部导入到项目中，并修改CMakeLists.txt中添加``native`.cpp, 修改java层导入so库为sandHook-``native``
2. `（2）配置相关的环境`  
在配置文件`build.gradle`中配置  
`externalNativeBuild {`  
`cmake {`  
`arguments`  
`'-DBUILD_TESTING=OFF'`  
`cppFlags`  
`"-frtti -fexceptions -Wpointer-arith"`  
`abiFilters`  
`'armeabi-v7a'`,`'arm64-v8a'`  
`}`  
`}`  
3. `（3）编译可以成功通过`
4. `（4）使用`  
`const` `char` `* libc =` `"/system/lib64/libc.so"`;  
`old_fopen = reinterpret_cast<``void` `*(*)(``char` `*,` `char` `*)>(SandInlineHookSym(libc,` `"fopen"``,`  
`reinterpret_cast<``void` `*>(new_fopen)));`  
`参数``2``：hook的函数 参数``3``：新的函数`  
添加原理hook旧函数的声明  
`void``* (*old_fopen)(``char``*,``char``*);`  
实现新的函数功能  
`void``* new_fopen(``char``* a,``char``* b){`  
`__android_log_print(``6``,``"windaa"``,``"I am from new open %s"``,a);`  
`return` `old_fopen(a,b);`  
`}`  
5. `（5）运行测试是否成功启动`

### 4.PLT/GOT hook实操

前面我们已经介绍了Got表hook的原理，下面我们实例操作一下导入表函数的hook

参考博客：[020K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0G2k6r3S2S2j5$3E0W2M7W2)9J5c8Y4m8Q4x3V1j5&6x3K6l9$3z5e0V1%4i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@80eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0G2k6r3S2S2j5$3E0W2M7W2)9J5c8Y4m8Q4x3V1j5&6x3K6l9$3z5e0V1%4i4K6u0W2K9s2c8E0L8l9`.`.`)

原理：

`通过解析elf格式，分析Section header table找出静态的.got表的位置，并在内存中找到相应的.got表位置，这个时候内存中.got表保存着导入函数的地址，读取目标函数地址，与.got表每一项函数入口地址进行匹配，找到的话就直接替换新的函数地址，这样就完成了一次导入表的Hook操作了`

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_ZQSAKZF2U7B5GNE.png)

首先，我们编写demo

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_QM94VN789TPJJTK.png)

我们编译后使用010Editor打开`libnative-lib.so`

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_EKYS5SE78TQUKNV.png)

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_GVKDZZSTB8VBTEA.png)

然后我们用ida打开，并直接跳转到该地址

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_V3PY8VJY2TYXEEX.png)

在got表中我们找到对应的mywin0函数

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_6WBC3YU8VG6KQMG.png)

#### <1>获得so模块的加载地址

我们可以使用`/proc/self/maps`去获得so模块的加载地址

```
char line[1024];
int *start;
int *end;
int n=1;
//1.拿到so的起始地址
//    749e5d7000-749e5db000 r--p 000f4000 103:09 441                           /system/bin/linker64
//    749e5db000-749e5dc000 rw-p 000f8000 103:09 441                           /system/bin/linker64
FILE *fd = fopen("/proc/self/maps","r");
while (fgets(line,sizeof(line),fd)){
if(strstr(line,"libnative-lib.so")){
__android_log_print(6,"windaa","%s",line);
if(n==1){
start = reinterpret_cast<int *>(strtoul(strtok(line,"-"),NULL,16));
end = reinterpret_cast<int *>(strtoul(strtok(NULL," "),NULL,16));
}
else{
strtok(line,"-");
end = reinterpret_cast<int *>(strtoul(strtok(NULL," "),NULL,16));
}
n++;
}
}
```

#### <2>找到got表的位置

我们首先根据段头找到section_header的首地址

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_Q7NVUZ5B2X57YS4.png)

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_TVKKU6G357P3HWZ.png)

然后我们遍历这个表就可以找到.got,然后根据got表地址再轮训找到函数地址

因为这种方法不能在内存中直接找到段头，内存中会抹去段头，所以我们可以通过加载so文件来定位

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_K8TVAMSKQ6KQC6U.png)

#### <3>定位到节表的地址

然后我们来获得节表的地址：

```
//读取elf文件
Elf64_Ehdr ehd;
int fp =open("/data/local/tmp/libnative-lib.so", O_RDONLY);
if(fp == -1){
__android_log_print(4,"windaa","%s","error");
}
//读取elf文件的文件头
read(fp,&ehd,sizeof(Elf64_Ehdr));
//读取节表的地址
unsigned long shof = ehd.e_shoff;
//读取节表的数量
int shnum = ehd.e_shnum;
//读取每个节表的大小
int shsize = ehd.e_ehsize;
//记录一下str表的偏移，主要是获取后面got的字符串值
int shstr = ehd.e_shstrndx;
```

我们打印一下此事shof的值，验证一下节表的地址

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_VJRHUBSV8PK6TYC.png)

这里可以发现成功读取

#### <4>定位到got表的位置和函数位置

然后我们拿到字符串的偏移值进行定位到got表，再进一步定位到函数

```
//2.拿到字符串表
Elf64_Shdr shdr;
//定位字符串，节表地址加字符串表偏移×节表个数
lseek(fp,shof+shstr*shsize,SEEK_SET);
//此时节表就定位到字符串表开头
read(fp,&shdr,shsize);
//分配一个字符串表大小
char* strtable = (char *)malloc(shdr.sh_size);
__android_log_print(6,"windaa","shdrsize %p",shdr.sh_offset);
//将字符串片指针移动到0x34104上
lseek(fp,shdr.sh_offset,SEEK_SET);
read(fp,strtable,shdr.sh_size);
//将指针移动到节表开头
lseek(fp,shof,SEEK_SET);

//遍历查找到got
for(int i=0;i<shnum;i++){
//从节表开头开始读取字符串，每次读取一个节表
read(fp,&shdr,shsize);
//通过节表的索引找到字符串表中对应的值
if(strcmp(&strtable[shdr.sh_name],".got")==0){
//定位到got表的地址
int* saddr = start+shdr.sh_addr/4;
//整个got表的大小
int size = shdr.sh_size;
//遍历got表中的函数
for(int j=0;j<size;j=j+8){
uint64_t  value = *(uint64_t *)(saddr + j / 4);
//找到mywind的地址
if(reinterpret_cast<uint64_t>(mywin0) == value)
{
__android_log_print(6,"windaa","value %p",value);
//替换mywind地址
// 获取当前内存分页的大小
uint64_t page_size = getpagesize();
// 获取内存分页的起始地址（需要内存对齐）
//page要保护的是函数的绝对地址，而不是相对地址
uint64_t entry_page_start = (uint64_t)(saddr+j/4) & (~(page_size - 1));
// 修改内存属性为可读可写可执行
if(mprotect((uint64_t*)entry_page_start, page_size, PROT_READ | PROT_WRITE | PROT_EXEC) == -1){
__android_log_print(6,"windaa","%s","mprotect failed");
}
value = (uint64_t)mywin1;
//将mywind0函数的地址换成mywind1函数的地址
memcpy((saddr+j/4),&value,16);
}
      }
}
}
```

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_TX7ZCATWPAERB55.png)

这里我们就可以发现成功的hook

got hook使用姿势：

1. `（1）使用``/``proc``/``self``/``maps去获得so模块的加载地址`
2. `（2）使用ElfHeader找到Section的首地址，并计算offset和size来获取StringTable`
3. `（3）找到got表位置，计算其内存位置，并指针指向got表首地址`
4. `（4）遍历got表中的函数，找到要hook的函数，使用mprotect进行hook`
5. `（5）将hook的函数地址替换为我们定义的函数地址`

### 5.Unicorn hook使用

这里我们简单了解一下基于unicorn的框架Unidbg的hook使用

开源地址：[43aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9r3E0D9x3o6t1J5z5q4)9J5c8Y4g2F1K9h3c8T1k6H3`.`.](https://bbs.kanxue.com/elink@e81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9r3E0D9x3o6t1J5z5q4)9J5c8Y4g2F1K9h3c8T1k6H3`.`.)

这里我们直接idea将项目拉取下来，然后等下项目环境配置完成

![image-20220517214320334](https://bbs.kanxue.com/upload/attach/202205/905443_KR77QZB8XUD3JQQ.png)

配置完成后，我们直接启动里面的示例代码查看hook效果

![image-20220517214506632](https://bbs.kanxue.com/upload/attach/202205/905443_JJDDRDSMTYJCX7D.png)

这里unidbg使用了xHook，xHook是一种PLT hook的方式，当然这只是unidbg强大功能其中的一种，也是hook技术中一种，这里就简单介绍到这，后续再详细讲如何使用

## 六、实验总结

本文从程序加载的原理出发，讲解了当下常用的一些基本的hook方式和手段，后续对其中一些hook方式再次深入讲解，实验的一些样本和代码会上传到知识星球和github，文章参考学了了很多大佬的文章和大佬星球的内容，参考文献放在末尾，有什么问题，就请各位大佬一一指出了。

github的地址：[github](https://bbs.kanxue.com/elink@450K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2)

## 参考文献

参考书籍：

`参考书目：《程序员的自我修养——链接、装载与库》`

GOT和PLT:

