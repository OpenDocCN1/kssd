
```
#include <linux/module.h>
#include <linux/version.h>
#include <linux/kernel.h>
#include <linux/types.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/device.h>
#include <linux/cdev.h>
#include <asm/uaccess.h>
#include <linux/slab.h>

//设备驱动常用变量
//static char *buffer_var = NULL;
static char *buffer_var = NULL;
static struct class *devClass; // Global variable for the device class
static struct cdev cdev;
static dev_t arbWriteModule_dev_no;

struct arbWriteNote
{
    ssize_t addr;
    ssize_t len;
    char *data;
};

static struct arbWriteNote *arbNoteObj;

static long arbWriteModule_ioctl(struct file *filp, unsigned int cmd, unsigned long arg);
static int arbWriteModule_open(struct inode *i, struct file *f);
static int arbWriteModule_close(struct inode *i, struct file *f);

static struct file_operations arbWriteModule_fops =
{
    .owner = THIS_MODULE,
    .open = arbWriteModule_open,
    .release = arbWriteModule_close,
    .unlocked_ioctl = arbWriteModule_ioctl
};

// 设备驱动模块加载函数
static int __init arbWriteModule_init(void)
{
    printk(KERN_INFO "[i] Module arbWriteModule registered");
    if (alloc_chrdev_region(&arbWriteModule_dev_no, 0, 1, "arbWriteModule") < 0)
    {
        return -1;
    }
    if ((devClass = class_create(THIS_MODULE, "chardrv")) == NULL)
    {
        unregister_chrdev_region(arbWriteModule_dev_no, 1);
        return -1;
    }
    if (device_create(devClass, NULL, arbWriteModule_dev_no, NULL, "arbWriteModule") == NULL)
    {
        printk(KERN_INFO "[i] Module arbWriteModule error");
        class_destroy(devClass);
        unregister_chrdev_region(arbWriteModule_dev_no, 1);
        return -1;
    }
    cdev_init(&cdev, &arbWriteModule_fops);
    if (cdev_add(&cdev, arbWriteModule_dev_no, 1) == -1)
    {
        device_destroy(devClass, arbWriteModule_dev_no);
        class_destroy(devClass);
        unregister_chrdev_region(arbWriteModule_dev_no, 1);
        return -1;
    }

    printk(KERN_INFO "[i] <Major, Minor>: <%d, %d>\n", MAJOR(arbWriteModule_dev_no), MINOR(arbWriteModule_dev_no));
    return 0;
}

// 设备驱动模块卸载函数
static void __exit arbWriteModule_exit(void)
{
    // 释放占用的设备号
    unregister_chrdev_region(arbWriteModule_dev_no, 1);
    cdev_del(&cdev);
}

// ioctl函数命令控制
long arbWriteModule_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
    char *chunk = NULL;
    char *buf;
    int retval = 0;
    switch (cmd) {
    case 111: //arbRead
        printk("Arbitrarily read function!---111\n");
        arbNoteObj = (struct arbWriteNote *)arg;
        copy_to_user(arbNoteObj->data,(char *)arbNoteObj->addr,arbNoteObj->len);
        break;

    case 222: //arbWrite
        printk("Arbitrarily write function!---222\n");
        arbNoteObj = (struct arbWriteNote *)arg;
        buf = kmalloc(arbNoteObj->len,GFP_KERNEL);
        copy_from_user(buf,arbNoteObj->data,arbNoteObj->len);
        memcpy((char *)arbNoteObj->addr,buf,arbNoteObj->len);
        kfree(buf);
        break;

    default:
        retval = -1;
        break;
    }

    return retval;
}

static int arbWriteModule_open(struct inode *i, struct file *f)
{
    printk(KERN_INFO "[i] Module arbWriteModule: open()\n");
    return 0;
}

static int arbWriteModule_close(struct inode *i, struct file *f)
{
    //kfree(buffer_var);
    //buffer_var = NULL;
    printk(KERN_INFO "[i] Module arbWriteModule: close()\n");
    return 0;
}

module_init(arbWriteModule_init);
module_exit(arbWriteModule_exit);

MODULE_LICENSE("GPL");
// MODULE_AUTHOR("blackndoor");
// MODULE_DESCRIPTION("Module vuln overflow");
```

可以看到传入一个结构体，包含数据指针和地址，直接任意读写。

## 不同劫持

### 1.劫持vdso

#### (1)前置知识

vdso的代码数据在内核里，当程序需要使用时，会将vdso的内核映射给进程，也就是相当于把内核空间的vdso原封不动地复制到用户空间，然后调用在用户空间的vdso代码。

如果我们能够将vdso给劫持为shellcode，那么当具有root权限的程序调用vdso时，就会触发我们的shellcode，而具有root权限的shellcode可想而知直接就可以。而vdso是经常会被调用的，所有只要我们劫持了vdso，大概率都会运行到我们的shellcode。

真实环境中`crontab`会调用vdso中的`gettimeofday`，且是root权限的调用。

而ctf题目中就通常可以用一个小程序来模拟调用。

```
#include <stdio.h>

int main(){
    while(1)
    {
        sleep(1);
        gettimeofday();
    }
}
```

将这个程序编译后放到init中，用`nohup`挂起，即可做到root权限调用`gettimeofday`：

```
nohup /gettimeofday &
```

#### (2)获取地址

由于不同版本的linux内核中的vdso偏移不同，而题目给的`vmlinux`通常又没有符号表，所以需要我们自己利用任意读漏洞来测量。(如果题目没有任意读漏洞，建议可以自己编译一个对应版本的内核，然后自己写一个具备任意读的模块，加载之后测量即可，或者进行爆破，一般需要爆破一个半字节)

例如这里给出的代码示例，就可以通过如下代码来将vdso从内核中dump下来。不过这种方式dump的是映射到用户程序的vdso，虽然内容是一样的，不过vdso在内核中的偏移却没有办法确定。

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/auxv.h>
#include <sys/mman.h>

int main(){
    int test;
    size_t result=0;
    unsigned long sysinfo_ehdr = getauxval(AT_SYSINFO_EHDR);
    result=memmem(sysinfo_ehdr,0x1000,"gettimeofday",12);
    printf("[+]VDSO : %p\n",sysinfo_ehdr);
    printf("[+]The offset of gettimeofday is : %x\n",result-sysinfo_ehdr);
    scanf("Wait! %d", test);
    /* gdb break point at 0x400A36 and then dump memory why only dump 0x1000 ??? */
    if (sysinfo_ehdr!=0){
        for (int i=0;i<0x2000;i+=1){
            printf("%02x ",*(unsigned char *)(sysinfo_ehdr+i));
        }
    }
}
```

这种方法的原理是通过寻找`gettimeofday`字符串来得到映射到程序中的vdso的内存页，如下：

![image-20211020150011934](https://bbs.kanxue.com/upload/attach/202208/904686_ZSSMYWC9UNWXKNX.jpg)

之后把下面的内容都复制，放到CyberChef中，利用From Hex功能得到二进制文件

![image-20211020150220448](https://bbs.kanxue.com/upload/attach/202208/904686_ASGSFEP8ANKWE6C.jpg)

然后就将得到的文件放到IDA中，即可自动解析到对应`gettimeofday`函数相对于vdso的函数偏移。

![image-20211020150335000](https://bbs.kanxue.com/upload/attach/202208/904686_W9UWFNV8CD3CT24.jpg)

这里就当是0xb20。

之后通过任意读，类似基于本题编写的以下代码，获取vdso相对于`vmlinux`基址的偏移。
