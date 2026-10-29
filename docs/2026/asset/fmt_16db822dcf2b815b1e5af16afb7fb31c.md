
```
lldb /Library/Developer/KDKs/KDK_10.12_16A323.kdk/System/Library/Kernels/kernel.development

(lldb) kdp-remote 虚拟机IP地址

.........

Process 1 stopped

* thread #2, name = '0xffffff800b777b00', queue = '0x0', stop reason = signal SIGSTOP

frame #0: 0xffffff800520ea14 kernel.development`kdp_register_send_receive(send=(IONetworkingFamily`IOKernelDebugger::kdpTransmitDispatcher(void*, unsigned int) at IOKernelDebugger.cpp:369), receive=(IONetworkingFamily`IOKernelDebugger::kdpReceiveDispatcher(void*, unsigned int*, unsigned int) at IOKernelDebugger.cpp:353)) at kdp_udp.c:478 [opt]

475 kdp_unregister_send_receive(

476 __unused kdp_send_tsend,

477 __unused kdp_receive_treceive)

-> 478 {

479 if (current_debugger == KDP_CUR_DB)

480 current_debugger = NO_CUR_DB;

481 kdp_flag &= ~KDP_READY;

Target 0: (kernel.development) stopped.

(lldb)

```

### 编写内核扩展

环境准备好之后就可以开始编写和调试内核扩展了，编写的过程还是在自己的系统完成然后在拷贝到虚拟机里面所以最好保证主系统和虚拟机系统大版本一致，首先使用Xcode就可以通过如下方式直接创建内核扩展模块。

![](https://bbs.kanxue.com/upload/attach/202208/625107_FT72G38E9FWT8QS.jpg)

创建出来的工程中会有如下两个函数，分别是驱动加载和卸载时调用的函数，一般用于加载的时候初始化设备驱动、hook函数，卸载的时候还原hook、删除设备驱动等等:

```
#include <mach/mach_types.h>

#include <libkern/libkern.h> //printf所在头文件



kern_return_t HelloWorld_start(kmod_info_t * ki, void *d);

kern_return_t HelloWorld_stop(kmod_info_t *ki, void *d);



kern_return_t HelloWorld_start(kmod_info_t * ki, void *d)

{

asm("int $3");

printf("HelloWorld_start...\n");

return KERN_SUCCESS;

}



kern_return_t HelloWorld_stop(kmod_info_t *ki, void *d)

{

printf("HelloWorld_stop...\n");

return KERN_SUCCESS;

}
```

分别在上面两个函数中使用`printf`打印输出，在kext中`printf`函数是在`libkern`中的，所以要`include`该头文件。另外为了在加载的时候能找到符号还要在`Info.plist`中的`OSBundleLibraries`加上`com.apple.kpi.libkern16.0.0`，后面这个版本号可以在被调试的机器上面运行`kextstat`看到， 在`Build Setting`设置`Debug Information Format`为`DWARF with DSYM File` 即在Debug也生成符号文件方便源码调试， 为了让调试器主动断下来便在模块加载的时候使用`int 3`主要触发断点。

使用`scp ./HelloWorld.kext xxx@10.xx.xx.xx:/Users/xxxx/Desktop/`将生成的`HelloWorld.kext`文件拷贝到被调试的机器，在这之前需要在被调试机器上的系统偏好设置-共享开启远程登录。最后使用如下命令修改用户组并加载以及卸载:

```
sudo chown -R root:wheel ./HelloWorld.kext

sudo kextload ./HelloWorld.kext

sudo kextutil ./HelloWorld.kext //可以查看加载出错的原因

sudo kextunload ./HelloWorld.kext
```

使用如下命令可以查看在模块中的输出日志:

```
sudo dmesg | tail -n 10
```

并且在加载的时候会主动触发断点并关联到本地的符号文件对应的源码的对应行如下:

```
Loading 1 kext modules . done.

Process 1 stopped

* thread #17, name = '0xffffff801a9be1a0', queue = '0x0', stop reason = EXC_BREAKPOINT (code=3, subcode=0x0)

frame #0: 0xffffff7f95c61f21 HelloWorld`HelloWorld_start(ki=0xffffff7f95c62000, d=0x0000000000000000) at HelloWorld.c:17

14

15  kern_return_t HelloWorld_start(kmod_info_t * ki, void *d)

16  {

-> 17      asm("int $3");

18      printf("HelloWorld_start...\n");

19      return KERN_SUCCESS;

20  }

Target 0: (kernel.development) stopped.

(lldb)
```

### 内核反调试

之前在[关于反调试&反反调试那些事](https://bbs.kanxue.com/elink@815K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4D9L8$3&6W2L8h3!0F1K9$3g2&6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9#2i4K6u0r3x3U0g2Q4x3V1k6S2L8Y4c8A6j5h3&6@1K9h3c8W2j5Y4g2Y4i4K6u0r3)中讲到一些反调试的方法和过反调试的方法，但是在应用层的话需要对每个应用都进行patch hook，而通过内核扩展模块可以直接从内核模块来接管系统调用针对调试检测和禁止附加的系统调用进行修改就行了。

我们知道在Windows平台有一个叫SSDT的东西，SSDT 的全称是 System Services Descriptor Table，系统服务描述符表，里面保存 Windows 系统服务地址的数组，通过修改该数组的函数地址就可以达到hook的效果，而在XNU中也有一个系统调用的表叫`sysent`，在`bsd/sys/sysent.h`可以找到其定义:

```
struct sysent {/* system call table */

sy_call_t*sy_call;/* implementing function */

#if CONFIG_REQUIRES_U32_MUNGING || (__arm__ && (__BIGGEST_ALIGNMENT__ > 4))

sy_munge_t*sy_arg_munge32; /* system call arguments munger for 32-bit process */

#endif

int32_tsy_return_type; /* system call return types */

int16_tsy_narg;/* number of args */

uint16_tsy_arg_bytes;/* Total size of arguments in bytes for

* 32-bit system calls

*/

};
```

所以如果能够找到这个表在内存中的位置然后修改其中`ptrace`和`sysctl`在表中对应的`sy_call`就可以接管`ptrace`和`sysctl`的调用了，但是因为这个符号不导出只能从内核文件分析找到在内存在模块加载的基地址然后加上文件中对应的位置确定`sysent`。首先使用IDA打开文件`/Library/Developer/KDKs/KDK_10.12_16A323.kdk/System/Library/Kernels/kernel.development`搜索找到符号`sysent`位于`__constdata`处，根据上面的结构体将IDA显示结果手动解析如下图所示:

![](https://bbs.kanxue.com/upload/attach/202208/625107_U2UU9BZ6T24MNJC.jpg)

可以通过搜索`sysent`表中前面几个符号的地址来确定`sysent`在`__constdata`的位置，由于KASLR所以首先要找到内核模块加载后在内存中的基地址，在10.11之后可以通过`vm_kernel_unslide_or_perm_external`来计算偏移，之前的话可以在内存中搜索文件头:

```
m_offset_t func_address = (vm_offset_t) vm_kernel_unslide_or_perm_external;

vm_offset_t func_address_unslid = 0;

vm_kernel_unslide_or_perm_external(func_address, &func_address_unslid);

g_kernel_slide = func_address - func_address_unslid;
```

得到内核加载在内存中的偏移之后剩下就是macho文件格式的解析与处理，首先获取`__CONST,__constdata`并该section中找`sysent`所在的位置:

```
struct sysent * find_sysent_table(){

if (g_sysent_table) {

return g_sysent_table;

}



mach_header_t* kernel_header = find_kernel_header();



if (!kernel_header) {

return NULL;

}



// The first three entries of the sysent table point to these functions.

sy_call_t *nosys = (sy_call_t *) kernel_find_symbol("_nosys");

sy_call_t *exit = (sy_call_t *) kernel_find_symbol("_exit");

sy_call_t *fork = (sy_call_t *) kernel_find_symbol("_fork");

if (!nosys || !exit || !fork) {

return NULL;

}



const char *data_segment_name;

const char *const_section_name;

if (macOS_Sierra()) {

data_segment_name = "__CONST";

const_section_name = "__constdata";

} else {

data_segment_name = "__DATA";

const_section_name = "__const";

}



section_t* section = macho_find_section(kernel_header, data_segment_name, const_section_name);



if(!section){

return NULL;

}



vm_offset_t offset;

for (offset = 0; offset < section->size; offset += 16) {

struct sysent *table = (struct sysent *) (section->addr + offset);

if (table->sy_call != nosys) {

continue;

}

vm_offset_t next_entry_offset = sizeof(struct sysent);

if (OSX_Mavericks()) {

next_entry_offset = sizeof(struct sysent_mavericks);

}

struct sysent *next_entry = (struct sysent *)

((vm_offset_t)table + next_entry_offset);

if (next_entry->sy_call != exit) {

continue;

}

next_entry = (struct sysent *)

((vm_offset_t)next_entry + next_entry_offset);

if (next_entry->sy_call != fork) {

continue;

}



g_sysent_table = table;

return g_sysent_table;

}



return NULL;

}
```

获取到`sysent`之后就可以直接保存原来的`sy_call`然后替换`sy_call`的地址为我们自己的实现就可以接管系统调用了，由于`__CONST,__constdata`默认是只读的所以写之前要先关闭写保护写完之后再还原:

```
kern_return_t anti_ptrace(int cmd){

/* Mountain Lion (10.8+) moved sysent[] to read-only section */

kwrite_on();



/*

* we check if the syscalls had been already assigned, because we get kernel panic if we overwrite the syscall with same function

*/

if(cmd == DISABLE && g_sysent_table[SYS_ptrace].sy_call != (sy_call_t *)sys_ptrace){

if(sys_ptrace != NULL){

/* restore pointer to the original function */

g_sysent_table[SYS_ptrace].sy_call = (sy_call_t *)sys_ptrace;



/* remove the flag that indicates the hooked status */

g_hooked_functions &= ~HK_PTRACE;

}

}else if(cmd == ENABLE && !(g_hooked_functions & HK_PTRACE)){

/* save address of the real function */

sys_ptrace = (void *)g_sysent_table[SYS_ptrace].sy_call;



/* hook the syscall by replacing the pointer in sysent */

g_sysent_table[SYS_ptrace].sy_call = (sy_call_t *)ar_ptrace;



/* we set our global variable g_hooked_functions to know this function has been hooked */

g_hooked_functions |= HK_PTRACE;

}



kwrite_off();



return KERN_SUCCESS;

}
```

最后编译加载kext然后使用如下代码测试效果:
