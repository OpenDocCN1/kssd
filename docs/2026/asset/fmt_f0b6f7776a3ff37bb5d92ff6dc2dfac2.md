
```c
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
static dev_t condition_dev_no;

struct flagStruct
{
    int len;
    char* flagUser;
};

static struct flagStruct* flagObj;
static char* flag = "flag{PIG007NBHH}";

static long condition_ioctl(struct file *filp, unsigned int cmd, unsigned long arg);
static int condition_open(struct inode *i, struct file *f);
static int condition_close(struct inode *i, struct file *f);
static bool chk_range_not_ok(ssize_t v1, ssize_t v2);

static struct file_operations condition_fops =
{
    .owner = THIS_MODULE,
    .open = condition_open,
    .release = condition_close,
    .unlocked_ioctl = condition_ioctl
};

// 设备驱动模块加载函数
static int __init condition_init(void)
{
    printk(KERN_INFO "[i] Module condition registered");
    if (alloc_chrdev_region(&condition_dev_no, 0, 1, "condition") < 0)
    {
        return -1;
    }
    if ((devClass = class_create(THIS_MODULE, "chardrv")) == NULL)
    {
        unregister_chrdev_region(condition_dev_no, 1);
        return -1;
    }
    if (device_create(devClass, NULL, condition_dev_no, NULL, "condition") == NULL)
    {
        printk(KERN_INFO "[i] Module condition error");
        class_destroy(devClass);
        unregister_chrdev_region(condition_dev_no, 1);
        return -1;
    }
    cdev_init(&cdev, &condition_fops);
    if (cdev_add(&cdev, condition_dev_no, 1) == -1)
    {
        device_destroy(devClass, condition_dev_no);
        class_destroy(devClass);
        unregister_chrdev_region(condition_dev_no, 1);
        return -1;
    }

    printk(KERN_INFO "[i] <Major, Minor>: <%d, %d>\n", MAJOR(condition_dev_no), MINOR(condition_dev_no));
    return 0;
}

// 设备驱动模块卸载函数
static void __exit condition_exit(void)
{
    // 释放占用的设备号
    unregister_chrdev_region(condition_dev_no, 1);
    cdev_del(&cdev);
}

// ioctl函数命令控制
long condition_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
    int retval = 0;
    printk(KERN_INFO "Ioctl Get!\n");
    switch (cmd) {
    case 111: //doubleCon //get flag_addr
        printk("Your flag is at %llx! But I don't think you know it's content\n", flag);
        break;

    case 222: //doubleCon //print flag
        flagObj = (struct flagStruct*)arg;
        ssize_t userFlagAddr = flagObj->flagUser;
        ssize_t userFlagObjAddr = (ssize_t)flagObj;

        if(chk_range_not_ok(userFlagAddr, flagObj->len) && chk_range_not_ok(userFlagObjAddr, 0) && (flagObj->len == strlen(flag)))
        {
            if(!strncmp(flagObj->flagUser, flag, strlen(flag)))
                printk("Looks like the flag is not a secret anymore. So here is it %s\n", flag);
            else
                printk("Wrong!");
            break;
        }
        else
        {
            printk("Wrong!\n");
            break;
        }
    default:
        retval = -1;
        break;
    }

    return retval;
}

static bool chk_range_not_ok(ssize_t v1, ssize_t v2)
{
    if((v1 + v2) <= 0x7ffffffff000)
        return true;
    else
        return false;
}

static int condition_open(struct inode *i, struct file *f)
{
    printk(KERN_INFO "[i] Module condition: open()\n");
    return 0;
}

static int condition_close(struct inode *i, struct file *f)
{
    kfree(buffer_var);
    //buffer_var = NULL;
    printk(KERN_INFO "[i] Module condition: close()\n");
    return 0;
}

module_init(condition_init);
module_exit(condition_exit);

MODULE_LICENSE("GPL");
// MODULE_AUTHOR("blackndoor");
// MODULE_DESCRIPTION("Module vuln overflow");
```

### (1)前置知识

主要是线程的知识。

就是如果一个变量是全局的，那么在没有锁的情况就会导致一个程序的不同线程对该全局变量进行的竞争读写操作。就像这里给出的代码，首先`flag`在内核模块中是静态全局的：

```c
static char *flag = "flag{PIG007NBHH}";
//-----------------------------------------------
if(chk_range_not_ok(userFlagAddr, flagObj->len) && chk_range_not_ok(userFlagObjAddr, 0) && (flagObj->len == strlen(flag)))
{
    if(!strncmp(flagObj->flagUser, flag, strlen(flag)))
        printk("Looks like the flag is not a secret anymore. So here is it %s\n", flag);
    else
        printk("Wrong!");
    break;
}
else
{
    printk("Wrong!\n");
    break;
}
```

### (2)检测：

先检测传入的数据的地址是否是在用户空间，长度是否为`flag`的长度，传入的所有数据是否处在用户空间。如果都是，再判断传入的数据与`flag`是否一致，一致则打印`flag`，否则打印Wrong然后退出。

这么一看好像无法得到`flag`，首先`flag`我们不知道，是硬编码在内核模块中的。其次就算无法传入内核模块中的地址，意味着就算我们获得了内核模块中`flag`的地址和长度，传进去也会判定失败。

### (3)漏洞

但是这是建立在一个线程中的，如果是多个线程呢。在我们传入用户空间的某个数据地址和长度之后，先进入程序的if语句，也就是进入如下if语句

```c
if(chk_range_not_ok(userFlagAddr, flagObj->len) && chk_range_not_ok(userFlagObjAddr, 0) && (flagObj->len == strlen(flag)))
```

然后在检测`flag`数据之前，也就是如下if语句之前，启动另外一个线程把传入的该数据地址给改成内核模块的`flag`的数据地址，这样就能成功打印`flag`了。

```c
if(!strncmp(flagObj->flagUser, flag, strlen(flag)))
```

那么就直接给出POC，这里涉及到一些线程的操作，可以自己学习一下。

### (4)POC
