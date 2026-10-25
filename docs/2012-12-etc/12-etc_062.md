
# 结合 init 源码剖析 android root 提权漏洞（CVE-2010-EASY）

> 原文：[`www.pediy.com/kssd/pediy12/139738.html`](https://www.pediy.com/kssd/pediy12/139738.html)

转载请注明出处:http://hi.baidu.com/androidhacker/blog/item/59faabfda34b71f57709d707.html

这篇文章是上一篇博客的后续分析，主要介绍向 `init` 进程发送热拔插信息后 `init` 进程的处理流程

首先我们来了解一个数据结构，`uevent`，如下

```
struct uevent {                                                                                

    const char \*action;                                                                        

    const char \*path;                                                                          

    const char \*subsystem;                                                                     

    const char \*firmware;                                                                      

    int major;                                                                                 

    int minor;                                                                                 

};  
```

内核收到的信息如下，`ACTION=add` `DEVPATH=/../data/local/tmp` `SUBSYSTEM=firmware` `FIRMWARE=../../../data/local/tmp/hotplug`

通过如下函数 `parse_event` 进行解析

```
static void parse\_event(const char \*msg, struct uevent \*uevent) 

{

 

    while(\*msg) {                                                                              

        if(!strncmp(msg, "ACTION=", 7)) {                                                      

            msg += 7;                                                                          

            uevent->action = msg;                                                              

        } else if(!strncmp(msg, "DEVPATH=", 8)) {                                              

            msg += 8;                                                                          

            uevent->path = msg;                                                                

        } else if(!strncmp(msg, "SUBSYSTEM=", 10)) {                                           

            msg += 10;                                                                         

            uevent->subsystem = msg;                                                           

        } else if(!strncmp(msg, "FIRMWARE=", 9)) {                                             

            msg += 9;                                                                          

            uevent->firmware = msg;                                                            

        } else if(!strncmp(msg, "MAJOR=", 6)) {                                                

            msg += 6;                                                                          

            uevent->major = atoi(msg);                                                         

        } else if(!strncmp(msg, "MINOR=", 6)) {                                                

            msg += 6;                                                                          

            uevent->minor = atoi(msg);                                                         

        }

         while(\*msg++)；

    }           

}
```

经过解析之后，`uevent` 的结构为：

`action="add"`

`path="/../data/local/tmp"`

`subsystem="firmware"`

`firmware="../../../data/local/tmp/hotplug"`

之后来到处理 `firmware` 的核心函数

```
static void process\_firmware\_event(struct uevent \*uevent)

{

    l = asprintf(&root, SYSFS\_PREFIX"%s/", uevent->path);

    //root 为/sys/../data/local/tmp/=/data/local/tmp/

    

    l = asprintf(&loading, "%sloading", root);

    //loading 为/data/local/tmp/loading

    

    l = asprintf(&data, "%sdata", root);

    //data 为/data/local/tmp/data   其内容为指向/proc/sys/kernel/hotplug 的符号链接

    

    l = asprintf(&file, FIRMWARE\_DIR"/%s", uevent->firmware);

    //file 为/etc/firmware/../../../data/local/tmp/hotplug=/data/local/tmp/hotplug

    

    loading\_fd = open(loading, O\_WRONLY);

    

    data\_fd = open(data, O\_WRONLY); 

    

    fw\_fd = open(file, O\_RDONLY);

    

    load\_firmware(fw\_fd, loading\_fd, data\_fd)；

}
```

最后来到 `load_firmware` 函数,把 `hotplug` 中的数据写到 `/proc/sys/kernel/hotplug` 中

其内容变为 `/data/local/tmp/exploid`

```
static int load\_firmware(int fw\_fd, int loading\_fd, int data\_fd)

{

    while (len\_to\_copy > 0) {

        char buf\[PAGE\_SIZE\];

 

 

        nr = read(fw\_fd, buf, sizeof(buf));

                                                                         

        len\_to\_copy -= nr;                                                                     

        while (nr > 0) {                                                                       

                                                                                               

            nw = write(data\_fd, buf + nw, nr);                                                 

 

            nr -= nw;

        }

    }

}
```

终于 `/proc/sys/kernel/hotplug` 中写入了我们的恶意程序了，只要再次受到如 wifi 打开、`usb` 插入等热拔插信息，内核就会以 `root` 权限加载我们的程序再一次执行，从而达到提权的目的
