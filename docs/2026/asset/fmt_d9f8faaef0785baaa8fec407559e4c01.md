
这个函数出来之后的 `eax+0x70`，取内容，就是计算的内容。但有个问题，这个函数里面计算的数值很复杂，存在 `eax` 中，`eax` 经过的赋值为：

```
.text:74E1B230 FF 35 98 8D 15 75          push    ?g_dwTls@@3KA   ; dwTlsIndex
.text:74E1B236 0F BF F0                   movsx   esi, ax
.text:74E1B239 FF 15 DC 12 C2 74          call    ds:__imp__TlsGetValue@4 ; TlsGetValue(x)
.text:74E1B23F 6B F6 0C                   imul    esi, 0Ch
.text:74E1B242 8B 40 64                   mov     eax, [eax+64h]
.text:74E1B245 8B 40 30                   mov     eax, [eax+30h]
.text:74E1B248 8B 04 06                   mov     eax, [esi+eax]
```

然后 `eax+0x70` 取内容就是计算出来的值了。

现在，这个值非常不好确定，如果要自己去计算，就要跟进 `TlsGetValue` 函数。这是完全没有必要的，直接用实验法来确定。比如，`width=1`，`width=2`，看等于多少就可以确认要得到的内容了。后面 `EXP` 的地址，也可以在这里提前确认。当然，实际的值是 `width*100`，或者 `width*100 << 8 + 8`。

现在最好就取 `width*100`。其实很容易确定。当 `width=41` 时，值就是 `0x1004`，十进制就是 4100。也可以看出来就是乘以的 100。

# 4、x86平台EXP编写要点

## 4.1 堆布局

![59.png](https://bbs.kanxue.com/upload/attach/202202/945611_VAZQ8KQS6Y69SPQ.png)

图2 堆溢出前堆布局

![59.png](https://bbs.kanxue.com/upload/attach/202202/945611_HCAYGNQVYH4NBYW.png)

图3 溢出后堆布局

这两个图来源于《漏洞战争》，很重要。首先分配 `E`、`A`、`B`、`CButtonLayout` 的 4 个堆，然后释放其中的 `E`，用 `vulheap` 替代，也就是我们之前的溢出堆来替代。

下面我来详细说明这里的编写、调试思路。

## 4.2 为什么堆大小是 `0x100`

运行 `EXP`，在堆布局完成之后，断下。

首先，用：`!heap -stat`

![](https://bbs.kanxue.com/upload/attach/202202/945611_AFHMNHVGMDNADMU.png)

然后，用：`!heap -stat -h 00340000`

![](https://bbs.kanxue.com/upload/attach/202202/945611_VKUBN3F9VA9Z83W.png)

接着，用 `!heap -a 00340000`，搜索大小为 `fc` 的堆：

![](https://bbs.kanxue.com/upload/attach/202202/945611_E9B2CJTEPQWW9BF.png)

注意：这个命令输出的大小为 `fc` 的堆数量不全，只有 6 个，但实际上是 250 个，所以要用 `!heap -flt s fc` 命令，但这里只是为了找对象大小。后面确认时，再用 `!heap -flt s [size]` 命令。

再用：`!heap -p -a 40e860`

![](https://bbs.kanxue.com/upload/attach/202202/945611_9Y3JYVRR99QKNRY.png)

最后，用 `!heap -flt s fc`

![](https://bbs.kanxue.com/upload/attach/202202/945611_EWTWADC496CCX2Q.png)

可知，`CButtonLayout` 的 `Size=0x21*8=0x108` 字节。`UserSize=0xFC`，但因为内存对齐关系，实际大小为 `0x108` 字节。

现在来看看 `E`、`A`、`B` 堆大小：

![](https://bbs.kanxue.com/upload/attach/202202/945611_4CADZFRJD6CHFKE.png)

可以看到，`Size` 是 108 字节大小，和上面 `CButtonLayout` 的大小 108 字节一样。这就是为什么要设置成 `0x100` 大小。当然 `UserSize` 不重要，一个 `0x100`，一个 `0xfc`，都是数据大小，但是整块堆的大小就要看 `0x21`，也就是 108 字节了。

这就是为什么堆大小是 `0x100` 的原因，当然这里的 `0x100`，指的是 `UserSize`。

记住，总共 3 个长度：

1. 堆的总长 `0x108`
2. 用户长度 `0x100`
3. `BSTR` 长度 `0xfa`

## 4.3 `EXP` 的 `span` 为什么等于 9

因为 `9*1C = FC`，刚好等于 `CButtonLayout` 的大小，而通过 `CButtonLayout` 我们知道，它是 `FC` 大小的堆，但实际 `size` 是 `0x108`。

特别注意：这儿 `usersize` 是 `0xFC` 大小，但 `size` 是 `0x108`，这和 `free` 的 `"EEEE"` 堆的大小是一样的。堆占用，要看 `size`，而不是 `usersize`。

## 4.4 覆盖虚表指针

### 4.4.1 第一阶段（覆盖字符串 `B` 长度）

在 `EXP` 代码 `CollectGarbage` 之后，再重新赋值 `ID=132` 的 `span` 之前，`alert` 一下，然后进去看内存布局。因为取 `ID=132` 时，`E` 和 `A` 之间有垃圾数据，所以选择 `ID=131` 的 `span`。也就是日志 `vulheap` 倒数第二个，就是我们要的内存地址。但是因为断点是刚刚打在分配时，还没有赋值，所以还可以看到 `E` 的存在。内存如下：

![](https://bbs.kanxue.com/upload/attach/202202/945611_RSUKGVVSZQMMF23.png)

![](https://bbs.kanxue.com/upload/attach/202202/945611_FN9PG7NQNZNHZ2U.png)

### 4.4.2 第二阶段（字符串 `B` 的长度被覆盖，地址在 `20de680`），内存如下

![](https://bbs.kanxue.com/upload/attach/202202/945611_5WP6QXWEZBGVW3G.png)

### 4.4.3 第三阶段：覆盖虚表指针

之前其他 `EXP` 的 `span` 等于 44，这个值可能也可以，但没必要覆盖这么大区域，直接等于 29 即可，内存如下：

未覆盖之前：

![](https://bbs.kanxue.com/upload/attach/202202/945611_XVSKFNB7Y4C3J6B.png)

覆盖之后：

![](https://bbs.kanxue.com/upload/attach/202202/945611_RMQ34K9ZFFNU8AQ.png)

![](https://bbs.kanxue.com/upload/attach/202202/945611_2F7H657SWTMVVQS.png)

### 4.4.4、触发

![](https://bbs.kanxue.com/upload/attach/202202/945611_YG7H933JHTQ5QRY.png)

可见，系统现在调用的地址是：

`call [0707002c]`

只要把堆喷射到这个地址，就可以了。

# 5、x64平台EXP编写要点

## 5.1 堆布局

堆布局和 `x86` 平台一样，只是堆的大小要重新修改，一些参数要重新确认，这在后面分析。

## 5.2 POC验证

![](https://bbs.kanxue.com/upload/attach/202202/945611_MWWSAJ3A3W85RJ7.png)

## 5.3 CButtonLayout大小确认

![](https://bbs.kanxue.com/upload/attach/202202/945611_92YWZKK2XPHVQPR.png)

上面红色是 `user size`。所以，`CButtonLayout` 的实际大小是 `0x148+0x8(8字节头)=0x150`，但注意，可能有绿色的 8 字节头填充。那就是 `0x158` 字节了。下面的堆，是堆大小还等于 `0x100` 时的图。现在，`x64` 要更改大小了。

![](https://bbs.kanxue.com/upload/attach/202202/945611_64GVW7QGK3NWZY5.png)

所以，可以看到，`Size = 0x12*0x10 = 0x120`，刚好是 `UserSize + 0x10` 字节头长度。`x64` 平台下，堆头是 `0x10` 字节。

## 5.4、x64堆代码修改及堆分布

```
for (var i = 0 ; i < 500 ; i += 2) {
    fr[i] = free.substring(0,(0x150-0x10-4-4-2)/2);
    al[i] = string1.substring(0,(0x150-0x10-4-4-2)/2);
    bl[i] = string2.substring(0,(0x150-0x10-4-4-2)/2);
    var obj = document.createElement("button");
    div_container.appendChild(obj);
}
```

其中，`0x150` 是分配的总大小，`-0x10` 是因为 8 字节头 + 8 字节填充，`-4-4` 是因为填充 + `BSTR` 长度，`-2` 是因为最后 `NULL` 字符。

![](https://bbs.kanxue.com/upload/attach/202202/945611_3MQSE9JX54Y4VHT.png)

![](https://bbs.kanxue.com/upload/attach/202202/945611_2MP85DXQK3SKUZ3.png)

注意：蓝色是虚表指针。

不管大小是 `0x158`，还是 `0x150`，内存的堆分配都是根据 `Size` 来的，只要 `Size` 的数值一样，堆就会被分配在相邻的内存。

## 5.5 确认堆分配地址

同样，在 `CalculateMinMax` 函数里，反汇编有：

```
.text:000007FF7DCA2E9B              lea     rcx, [r14+0D0h] ; this
.text:000007FF7DCA2EA2              mov     r8d, r9d        ; int
.text:000007FF7DCA2EA5              mov     edx, 20h ; ' ' ; unsigned __int64
.text:000007FF7DCA2EAA              call    ?EnsureSizeWorker@CImplAry@@AEAAJ_KJ@Z ; CImplAry::EnsureSizeWorker(unsigned __int64, long)
.text:000007FF7DC0CE00              mov     rdi, rcx
.text:000007FF7DC0CE72              lea     rcx, [rdi+10h]  ; void **
.text:000007FF7DC0CE76              mov     edx, ebp        ; unsigned __int64
.text:000007FF7DC0CE78              call    ?_HeapRealloc@@YAJPEAPEAX_K@Z ; _HeapRealloc(void **, unsigned __int64)
```

所以堆指针在：

`r14+0xd0+0x10`

`bu mshtml!CTableLayout::CalculateMinMax+0x1df ".echo vulheap;dd poi(r14+0xd0+0x10) l4;g"`

这个断点的最后一个，也就是 `id=132` 的 `vulheap`，就是要被覆盖的堆的地址，后面的截图，就是这样找的地址。

## 5.6 修改 `span` 长度以让 `vulheap` 等于 `0x150`

通过上面断点分析，发现分配的大小是 `span*0x20`。

所以让其等于 `10*32=320=0x140`，再加 `0x10` 字节头，让其等于 `0x150`。

那么需要 `span` 等于 10。

## 5.7 覆盖虚表指针

### 5.7.1、第一阶段：覆盖字符串 `B` 长度确认

`bu mshtml!CTableLayout::CalculateMinMax+0x1df ".echo vulheap;dd poi(r14+0xd0+0x10) l4;g"`

这个断点的最后一个，也就是 `id=132` 的 `vulheap`，就是要被覆盖的堆的地址，下面的截图，就是这样找的地址。

![](https://bbs.kanxue.com/upload/attach/202202/945611_7YCU4ZE8ZCAMCN7.png)

所以要覆盖到 `32c9cd0`，长度就是 `0x2a0`。

通过前面知道，一个 `span` 是 `0x20`，所以这里的长度是：`0x2a0 / 0x20 = 0x15 = 21`，实际可以再加 1 个 1，成为 22。

### 5.7.2、第二阶段：覆盖 `B` 字符串长度（有重大改进）

采用 `x86` 堆结构，无法覆盖虚表，需要修改

![](https://bbs.kanxue.com/upload/attach/202202/945611_45P3CGZ77NSN8N5.png)

![](https://bbs.kanxue.com/upload/attach/202202/945611_D9QG39N9ZBZ6A7P.png)

修改之后，加一个块 `C`，为后面虚表指针被覆盖做准备。

注意：上面字符串 `B` 的长度被替换了。

### 5.7.3、获取 `mshtml` 基址代码说明

```
var leak = bl[i].substring((0x140-4-6+2+16+0x150)/2,(0x140-4-6+2+16+8+0x150)/2);
```

`bl` 是块 `BBBB`。

说明：

- `0x140` 是字符串的长度
- `-4` 是填充
- `-6` 是 `BSTR` 的 4 字节长度和末尾 2 字节

所以，当使用 `substring` 函数的时候，这个 `index`，就是字符串 `BBBB` 块的最后一个字符。

然后用这个 `index`（`0x140-4-6`）继续计算：

- `+2`，是 2 字节 `NULL`
- `+0x16`，是 `CButtonLayout` 块的头
- `+0x150`，是块 `C` 的大小（包括头+填充+内容，是整个大小）

然后就可以获取 `47d25f0` 这个地址的 `mshtml` 基址了。

### 5.7.4、第三阶段：覆盖虚表指针

![](https://bbs.kanxue.com/upload/attach/202202/945611_FRMUBR7BSYVB98S.png)

### 5.7.5 触发

![](https://bbs.kanxue.com/upload/attach/202202/945611_RUSBAXNR3MBUAFY.png)

这里的指针地址是 `0707002407070024`，要修改这个地址，让地址能够被堆喷射。

但是，这里 `x64` 是 `0707002407070024`，怎么办呢？堆喷不过去。

长度写成 `1`，地址最小也是 `0x0000006400000064`，也堆喷不过去。

所以 `x64` 平台我只是成功控制了喷射地址，而没有实现利用的效果。

# 6、堆喷射和ROP

`ROP` 代码是通过 `mona.py` 实现的，你可以自己搜索资料实现。也可以参考我 `EXP` 的代码。

![](https://bbs.kanxue.com/upload/attach/202202/945611_MDUZSEBATQ7M4PT.png)

1. 先观察喷射的内存分布，本来喷射最后 4 位都是 `0018` 结尾，如果是在 `0c0c` 结尾的内存的话，就要 `padding` 数据，最后让 `eip` 对准 `rop` 地址，但是这里，直接把位置定准在了 `0024`，也就是说 `0018+8字节头+4字节BSTR长度`，刚好就是 `24`，所以就没有 `padding`，就让 `width*100` 直接指向 `24`。具体 `padding` 的技巧，参考下面网址：
