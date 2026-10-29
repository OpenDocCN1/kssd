
这个补丁是结合具体情况来写的，看代码就知道，是直接用指针来修改内存的。这样子做在这个例子可行，但是在别的情况就可能不行了。为什么呢，因为在正常情况，`__TEXT`段是只读可执行的，直接修改会报错的。但是我们这里为什么可以直接修改呢，因为我们前面说过，`__TEXT`的内容是被加密的，运行后将会被解密，解密回填的时候必然要对其进行写操作。于是`hd`在运行时就自行对这些内存增加了写权限。结果却没有恢复原来的权限，于是就方便了我们（要是我，我也不会恢复原来的权限的，太麻烦，没必要）。

这里仅对内存做了两处补丁，这样子破解可能还不完全，我也没发现哪里不能用，不过能用就是了。没有了注册窗口，没有了调试面板的限制，没有Demo的水印等。

最后打包的时候，将`MacOS`下的`Hopper Disassembler v3`重命名为`Hopper Disassembler v3_`，然后新建一个shell脚本，名字叫`Hopper Disassembler v3`，脚本代码如下：

```
#!/bin/bash
HD_PATH="`dirname "${0}"`"
HD_BIN="`dirname "${0}"`"/Hopper\ Disassembler\ v3_

export DYLD_INSERT_LIBRARIES="${HD_PATH}/chd.dylib"
"$HD_BIN"
```

我这里的补丁叫`chd.dylib`，将其放到`MacOS`下就OK了。简直完美。

若有不对之处敬请指正。

很多同学说搞不定，在此奉上传送门：[3feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4T1L8U0V1@1f1@1c8^5](https://bbs.kanxue.com/elink@11dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4T1L8U0V1@1f1@1c8^5)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

```
/*

	export DYLD_FORCE_FLAT_NAMESPACE=1
	export DYLD_INSERT_LIBRARIES=~/chd.dylib

*/

#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>
#include "pthread.h"

#include <sys/types.h>
#include <sys/ptrace.h>
#include <sys/sysctl.h>

#include <mach/mach.h>
#include <mach/mach_init.h>
#include <mach/mach_vm.h>

#include "libkern/OSCacheControl.h"

mach_vm_address_t getBasicAddress(int pid){
	mach_vm_size_t region_size = 0;
	mach_vm_address_t region = 0;
	mach_port_t task = 0;
	int ret = 0;

	ret = task_for_pid(mach_task_self(),pid,&task);
	if (ret != 0)
	{
		printf("task_for_pid() message %s!\n",mach_error_string(ret));
		return 0;
	}

	  /* Get region boundaries */
#if defined(_MAC64) || defined(__LP64__)
	vm_region_basic_info_data_64_t info;
	mach_msg_type_number_t info_count = VM_REGION_BASIC_INFO_COUNT_64;
	vm_region_flavor_t flavor = VM_REGION_BASIC_INFO_64;
	if ((ret = mach_vm_region(mach_task_self(), &region, &region_size, flavor, (vm_region_info_t)&info,
		(mach_msg_type_number_t*)&info_count, (mach_port_t*)&task)) != KERN_SUCCESS)
	{
		printf("mach_vm_region() message %s!\n",mach_error_string(ret));
		return 0;
	}
#else
	vm_region_basic_info_data_t info;
	mach_msg_type_number_t info_count = VM_REGION_BASIC_INFO_COUNT;
	vm_region_flavor_t flavor = VM_REGION_BASIC_INFO;
	if ((ret = vm_region(mach_task_self(), &region, &region_size, flavor, (vm_region_info_t)&info,
		(mach_msg_type_number_t*)&info_count, (mach_port_t*)&task)) != KERN_SUCCESS)
	{
		printf("vm_region() message %s!\n",mach_error_string(ret));
		return NULL;
	}
#endif
	return region;
}

vm_size_t readRemotoMemory(char *buf,vm_size_t len,int pid,vm_address_t address)
{
	vm_size_t outSize = 0;
	mach_port_t task = 0;

	int ret = task_for_pid(mach_task_self(),pid,&task);
	if (ret != 0)
	{
		printf("task_for_pid() message %s!\n",mach_error_string(ret));
		return 0;
	}

	ret = vm_read_overwrite(task,address,len,(vm_address_t)buf,&outSize);
	if (ret != 0)
	{
		printf("vm_read_overwrite() message %s!\n",mach_error_string(ret));
		return 0;
	}
	return outSize;
}
 //
 int FakeCode(char *addr, char code)
 {
 	mach_port_t task;
 	mach_vm_size_t region_size = 0;
 	mach_vm_address_t region = (vm_address_t)addr;

  /* Get region boundaries */
 #if defined(_MAC64) || defined(__LP64__)
 	vm_region_basic_info_data_64_t info;
 	mach_msg_type_number_t info_count = VM_REGION_BASIC_INFO_COUNT_64;
 	vm_region_flavor_t flavor = VM_REGION_BASIC_INFO_64;
 	if (mach_vm_region(mach_task_self(), &region, &region_size, flavor, (vm_region_info_t)&info, (mach_msg_type_number_t*)&info_count, (mach_port_t*)&task) != 0)
 	{
 		return 0;
 	}
 #else
 	vm_region_basic_info_data_t info;
 	mach_msg_type_number_t info_count = VM_REGION_BASIC_INFO_COUNT;
 	vm_region_flavor_t flavor = VM_REGION_BASIC_INFO;
 	if (vm_region(mach_task_self(), &region, &region_size, flavor, (vm_region_info_t)&info, (mach_msg_type_number_t*)&info_count, (mach_port_t*)&task) != 0)
 	{
 		return 0;
 	}
 #endif

  /* Change memory protections to rw- */
 	if (vm_protect(mach_task_self(), region, region_size, 0, VM_PROT_READ | VM_PROT_WRITE | VM_PROT_COPY) != KERN_SUCCESS)
 	{
 		//_LineLog();
 		return 0;
 	}

  /* Actually perform the write */
 	*addr = code;

  /* Flush CPU data cache to save write to RAM */
 	sys_dcache_flush(addr, sizeof(code));

  /* Invalidate instruction cache to make the CPU read patched instructions from RAM */
 	sys_icache_invalidate(addr, sizeof(code));

  /* Change memory protections back to r-x */
 	vm_protect(mach_task_self(), region, region_size, 0, VM_PROT_EXECUTE | VM_PROT_READ);
 	return 1;
 }

//int main(int argc, char const *argv[])
void* handler(void *p)
{
	//int pid = 16057;
	int pid = getpid();
	char buffer[512];
	mach_vm_address_t address = 0;

	address = getBasicAddress(pid);

	//printf("Target pid     : %d\n",pid);
	//printf("Base address   : %llx\n", address);

	if (address == 0)
	{
		printf("getBasicAddress() faild!\n");
		return NULL;
	}

	//Demo
	char *demo = (char*)address + 0x329a9e;
	demo[0] = ' ';
	demo[1] = ' ';
	demo[2] = ' ';
	demo[3] = ' ';

	//Demo version
	char *dv = (char*)address + 0x329E8C;
	dv[0] = 'F';
	dv[1] = 'u';
	dv[2] = 'l';
	dv[3] = 'l';

	//Waiting for decode __text
	sleep(1);

	//checkRegistrationLicense:
	//xor ebx,ebx    =>    mov     $1,%bl
	//xor edi,edi    =>    inc     %edi
	*(uint32_t*)(address + 0xb9b7) = 0xc7ff01b3;

	//checkRegistrationToken
	// xor r14d,r14d => inc r14d
	*(uint8_t*)(address + 0xb974) 	  = 0x41;
	*(uint8_t*)(address + 0xb974 + 1) = 0xff;
	*(uint8_t*)(address + 0xb974 + 2) = 0xc6;

	return NULL;
}

void __attribute__((constructor)) init()
{
	int err;
	pthread_t ntid;
    err = pthread_create(&ntid, NULL, handler, NULL);
    if (err != 0)
    {
    	printf("can't create thread: %s\n", strerror(err));
    	return ;
    }
}
```
