
# [原创]对objc_msgSend的分析

刚接触ios，每次调试程序，都能看到很多的`objc_msgSend`。看了函数说明，感觉有点类似windows的`sendmessage`，但还是理解不深刻。所以今天花了一下午的时间，分析了一下。

## 1、基础知识

参考：`428K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2&6i4K6u0W2L8%4y4U0K9r3W2F1j5g2)9J5k6h3&6W2N6q4)9J5c8X3q4E0L8%4W2S2K9g2)9J5c8X3u0D9L8$3N6Q4x3V1j5&6x3U0p5&6x3b7`.`.

`objc_msgSend`方法含两个必要参数：`receiver`、`方法名`即：`selector`，如：

```objc
[receiver message];
```
将被转换为：`objc_msgSend(receiver, selector);`

`objc_msgSend`方法也能hold住`message`的参数，如：

`objc_msgSend(receiver, selector, arg1, arg2, …);`

每个对象都有一个指向所属类的指针`isa`。通过该指针，对象可以找到它所属的类，也就找到了其全部父类，如下图所示：

当向一个对象发送消息时，`objc_msgSend`方法根据对象的`isa`指针找到对象的类，然后在类的调度表（`dispatch table`）中查找`selector`。如果无法找到`selector`，`objc_msgSend`通过指向父类的指针找到父类，并在父类的调度表（`dispatch table`）中查找`selector`，以此类推直到`NSObject`类。一旦查找到`selector`，`objc_msgSend`方法根据调度表的内存地址调用该实现。通过这种方式，`message`与方法的真正实现在执行阶段才绑定。

为了保证消息发送与执行的效率，系统会将全部`selector`和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含有当前类自己的 `selector`以及继承自父类的`selector`。查找调度表（`dispatch table`）前，消息发送系统首先检查`receiver`对象的缓存。

注意红色字体，经过下面的分析，红色字体都是珠玑！

## 2、objc_msgSend

这个函数在`libobjc.A.dylib`中，在下面的目录中可以找到（我用的`ios7.1.1`系统）。

将它拖入`ida`中分析：（注意红线勾的地方）

## 3、整理

经过调试，我对前面的图进行了整理。

## 4、例证

分别对先后调用的`Hello`函数下断点，通过查看内存和寄存器来验证前面的结论。

```objc
- (IBAction)onBtn:(id)sender {
    //BOOL isSet = ( [[NSUserDefaults standardUserDefaults] valueForKey:@"KeyForCurrentPatternToUnlock"]) ? YES : NO;
    //CMyClass *pTest = [[CMyClass alloc] init1];
    CMyClass *pTest = [CMyClass alloc];
    [pTest Hello:4]; //断点1--第一次调用Hello
    pTest->m_nCount = 10;
    pTest->m_nLen = 15;
    //[CMyClass testClassFun];
    [pTest Hello:4]; //断点2--第二次调用Hello
    [pTest World:"Test" Num:6];
}
```

进入`objc_msgSend`后查看内存和寄存器，如下图：

### 4.1 断点1--第一次调用Hello

```lldb
(lldb) si
(lldb) re r
General Purpose Registers:
r0 = 0x16d3d610
r1 = 0x000760ea  "Hello:"
r2 = 0x00000004
r3 = 0x3aec5621  libobjc.A.dylib`objc_msgSend + 1
r4 = 0x00000004
r5 = 0x3aec5621  libobjc.A.dylib`objc_msgSend + 1
r6 = 0x0007770c  "Hello:"
r7 = 0x27d93508
r8 = 0x0000000f
r9 = 0x00000000
r10 = 0x00077724  test_msgSend`CMyClass.m_nLen
r11 = 0x0000000a
r12 = 0x3d422234  (void *)0x3b4e33c8: _platform_memset$VARIANT$CortexA9
sp = 0x27d934a0
lr = 0x00075cab  test_msgSend`-[ViewController onBtn:] + 235 at ViewController.m:36
pc = 0x3aec5620  libobjc.A.dylib`objc_msgSend
cpsr = 0x20000030
```

```lldb
(lldb) memory read -s4 -fx -c16 0x16d3d610  // receiver指针（R0）
0x16d3d610: 0x00077748 0x00000000 0x00000000 0x00000000  // isa指针
0x16d3d620: 0x00000000 0x00000000 0x00000000 0x00000000
0x16d3d630: 0x00000000 0x16d2f3c0 0x00000000 0x00000000
0x16d3d640: 0x3b686010 0x01000480 0x00090003 0x00000001
```

```lldb
(lldb) memory read -s4 -fx -c16 0x00077748  // isa指针
0x00077748: 0x00077734 0x3d3d1050 0x3aed02a8 0x00000000 // 函数结构体数组
0x00077758: 0x16d52a70 0x00077770 0x3bb0a1e4 0x17293c00
0x00077768: 0x004400ff 0x16e50fb0 0x3d3d1064 0x3bb0a1d0
0x00077778: 0x16e523f0 0x00060007 0x16e50e50 0x3d3d1064
```

```lldb
(lldb) memory read -s4 -fx -c16 0x3aed02a8  // 第一次调用，还未缓存，所以为空
0x3aed02a8: 0x00000000 0xfffffffc 0x00000003 0x00000000
0x3aed02b8: 0x00000000 0x00000000 0x00000000 0x00000000
0x3aed02c8: 0x00000000 0x00000000 0x00000000 0x00000000
0x3aed02d8: 0x00000000 0x624f0000 0x7463656a 0x495f5f00
```

// 运行到`_objc_msgSend`里这一行
// `__text:37D0963A                 TEQ.W           R12, R1 ; 与传入的函数名进行比较`
// 此时在lldb中查看寄存器

```lldb
(lldb) re read
General Purpose Registers:
r0 = 0x16d3d610
r1 = 0x000760ea  "Hello:"
r2 = 0x00000004
r3 = 0x3aec5621  libobjc.A.dylib`objc_msgSend + 1
r4 = 0x00000004
r5 = 0x3aec5621  libobjc.A.dylib`objc_msgSend + 1
r6 = 0x0007770c  "Hello:"
r7 = 0x27d93508
r8 = 0x0000000f
r9 = 0x3aed02a8  libobjc.A.dylib`_objc_empty_cache // 这里显示空缓存
r10 = 0x00077724  test_msgSend`CMyClass.m_nLen
r11 = 0x0000000a
r12 = 0x00000000
sp = 0x27d934a0
lr = 0x00075cab  test_msgSend`-[ViewController onBtn:] + 235 at ViewController.m:36
pc = 0x3aec563e  libobjc.A.dylib`objc_msgSend + 30
cpsr = 0x20000030
```

从上面的可以看出，第一次调用，系统并未缓存。最后会进入函数`__objc_msgSend_uncached`，查找并缓存函数地址。

### 4.2、断点2--第二次调用Hello

```lldb
(lldb) si
(lldb) re read
General Purpose Registers:
r0 = 0x16d3d610
r1 = 0x000760ea  "Hello:"
r2 = 0x00000004
r3 = 0x00077724  test_msgSend`CMyClass.m_nLen
r4 = 0x00000004
r5 = 0x3aec5621  libobjc.A.dylib`objc_msgSend + 1
r6 = 0x0007770c  "Hello:"
r7 = 0x27d93508
r8 = 0x0000000f
r9 = 0x0000000f
r10 = 0x00077724  test_msgSend`CMyClass.m_nLen
r11 = 0x0000000a
r12 = 0x0007770c  "Hello:"
sp = 0x27d934a0
lr = 0x00075cdb  test_msgSend`-[ViewController onBtn:] + 283 at ViewController.m:41
pc = 0x3aec5620  libobjc.A.dylib`objc_msgSend
cpsr = 0x60000030
```

```lldb
(lldb) memory read -s4 -fx -c16 0x16d3d610 // receiver指针（R0）
0x16d3d610: 0x00077748 0x0000000f 0x0000000a 0x00000000 // isa指针
0x16d3d620: 0x00000000 0x00000000 0x00000000 0x00000000
0x16d3d630: 0x00000000 0x16d2f3c0 0x00000000 0x00000000
0x16d3d640: 0x3b686010 0x01000480 0x00090003 0x00000001
```

```lldb
(lldb) memory read -s4 -fx -c16 0x00077748 // isa指针
0x00077748: 0x00077734 0x3d3d1050 0x16e47b20 0x00010003  // 函数结构体数组
0x00077758: 0x16d52a70 0x00077770 0x3bb0a1e4 0x17293c00
0x00077768: 0x004400ff 0x16e50fb0 0x3d3d1064 0x3bb0a1d0
0x00077778: 0x16e523f0 0x00060007 0x16e50e50 0x3d3d1064
```

```lldb
(lldb) memory read -s4 -fx -c16 0x16e47b20  // 函数结构体数组
0x16e47b20: 0x00000000 0x00000000 0x00000000 0x00000000
0x16e47b30: 0x000760ea 0x00075aed 0x00000000 0x00000000 // 有数据了。这里我有点迷糊，因为有时
0x16e47b40: 0x00000001 0x16e47b18 0x00000000 0x00000000 // 候数据是在地址0x16e47b28，很奇怪！
0x16e47b50: 0x00000028 0x30814000 0x031bc000 0x00000000
```

```lldb
(lldb) memory read 0x000760ea  // 显示函数名字
0x000760ea: 48 65 6c 6c 6f 3a 00 57 6f 72 6c 64 3a 4e 75 6d  Hello:.World:Num
0x000760fa: 3a 00 6d 5f 6e 4c 65 6e 00 6d 5f 6e 43 6f 75 6e  :.m_nLen.m_nCoun
```

```lldb
(lldb) di -s 0x00075aed  // 显示函数地址
test_msgSend`-[CMyClass Hello:] + 1 at CMyClass.m:25:
0x75aed:  ldr    r5, [r6, #0x78]
0x75aef:  strh   r6, [r0, #0x1a]
```

第二次调用，系统有缓存。就不会进入函数`__objc_msgSend_uncached`，而是直接取出函数地址并调用。

`word:`

[对objc_msgSend的分析.rar](https://bbs.kanxue.com/attach-download-97376.htm)

`src:`

[src.rar](https://bbs.kanxue.com/attach-download-97375.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [src.rar](javascript:void(0);)
  （28.42kb，79次下载）
- [对objc_msgSend的分析.rar](javascript:void(0);)
  （356.36kb，210次下载）
