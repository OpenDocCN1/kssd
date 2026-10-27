
# [原创]启用PAE后虚拟地址到物理地址的转换

## 由常规的两级页表转换得不到物理地址的引发的思考？

可能你会发现一个郁闷的事情——按照通常的两级页表的转化得不到正确物理地址，到底是哪个环节出了问题咧，其实一切都是 PAE 惹的祸！

## PAE 是神马？

PAE：全称 Phyiscal Address Extension，物理地址扩展。

## 如何判断 PAE 是否开启？

一般有两个特点：

- “我的电脑”=》属性 =》常规面板中有“物理地址扩展”的字样。

  ![](https://bbs.kanxue.com/upload/attach/201311/603227_1z70e9lv9tgeoch.png)

- `CR4` 寄存器的第 5 位为 1，说明开启了 PAE。

  ![](https://bbs.kanxue.com/upload/attach/201311/603227_85sovex3blnr5li.png)

说明当前系统启用了 PAE。

## 新的转化规则

转换示意图：

![](https://bbs.kanxue.com/upload/attach/201311/603227_e672urgwk0ojqxf.png)

### 规则

- 每个项的长度为 8。
- `VA` 为：
  - 2 位（30—31）：页目录指针表的索引
  - 9 位（21—29）：页目录表索引
  - 9 位（12—20）：页表索引
  - 12 位（0—11）：页内偏移

例如：开启 PAE 的情况下，将计算器进程的入口点 `0x01012475` 转化为物理地址。

|  | 页目录指针表的索引 | 页目录表索引 | 页表索引 | 页内偏移 |
| `0x01012475` | `00` | `000001000` | `000010010` | `010001110101` |
|  | `0x0` | `0x8` | `0x12` | `0x475` |

### 1. `!process 0 0` 获取计算器进程的信息

![](https://bbs.kanxue.com/upload/attach/201311/603227_1745qtr5yssry8l.png)

页目录指针表物理地址：`0x17f02c0`。

### 2. 定位页目录指针表并获取页目录表物理页地址

![](https://bbs.kanxue.com/upload/attach/201311/603227_qz1yqg2zn8onilg.png)

页目录指针表项的下标为 0，所以就是 `0x06d13801`，因此 `0x06d13000` 是页目录表物理页面的首地址。

### 3. 定位页表项

![](https://bbs.kanxue.com/upload/attach/201311/603227_06c5dw72huvjrat.png)

`0x4250867` 就是要找的页目录表项，因此页表物理内存页面首地址为 `0x4250000`。

### 4. 定位物理页面

![](https://bbs.kanxue.com/upload/attach/201311/603227_gux3gpr5tpzmch7.png)

`0x56b7000` 就是物理页面首地址，`0x457` 是偏移，所以 `0x056b7475` 就是我们要转化的物理地址了。

![](https://bbs.kanxue.com/upload/attach/201311/603227_6p1tc7acyogp0jz.png)

正好就是虚拟地址对应的内容。

### 修改物理地址处首字节对比虚拟地址处显示内容

![](https://bbs.kanxue.com/upload/attach/201311/603227_t7h04d8nmvh1ka9.png)

![](https://bbs.kanxue.com/upload/attach/201311/603227_v3b628yhfk69lmd.png)

虚拟地址处内容也随之改变了。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [`1.png`](javascript:void(0);)（34.53kb，17次下载）
- [`2.png`](javascript:void(0);)（4.31kb，9次下载）
- [`3.png`](javascript:void(0);)（65.41kb，8次下载）
- [`4.png`](javascript:void(0);)（16.56kb，7次下载）
- [`6.png`](javascript:void(0);)（3.12kb，6次下载）
- [`7.png`](javascript:void(0);)（3.86kb，6次下载）
- [`8.png`](javascript:void(0);)（59.61kb，6次下载）
- [`9.png`](javascript:void(0);)（0.62kb，7次下载）
- [`10.png`](javascript:void(0);)（61.02kb，7次下载）
- [`11.png`](javascript:void(0);)（3.90kb，7次下载）
