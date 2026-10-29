
```
read = 0;
while (count > 0) {
    sz = size_inside_page(p, count);
    kvaddr = ioremap(p, sz);
    if (!kvaddr)
    {
    printk(KERN_ERR "ioremap failed\n");
    return -ENOMEM;
    }
    if (buf) {
        if (cmd_type == READ_MEMORY) {
    memcpy_fromio(buf, kvaddr, sz);
    printk("Read To Buffer:%s Size:%d, Src:%s", buf, sz, kvaddr);
    }
    if (cmd_type == WRITE_MEMORY) {
    printk("Before Write To Addr:%p Content:%s", kvaddr, buf);
    memcpy_toio(kvaddr, buf, sz);
    printk("After Write To Addr:%p Content:%s", kvaddr, buf);
    }
    }
    iounmap(kvaddr);
    kvaddr = NULL;
    buf += sz;
    count -= sz;
    read += sz;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
