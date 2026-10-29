
```python
from unicorn import *
from unicorn.arm_const import *

table = []

def bytes2bin(bytes):
    arr = []
    for v in [m for m in bytes]:
        arr.append([(v & 128) >> 7, (v & 64) >> 6, (v & 32) >> 5, (v & 16) >> 4, (v & 8) >> 3, (v & 4) >> 2, (v & 2) >> 1, v & 1])
    return [i for j in arr for i in j]

def bin2bytes(arr):
    length = len(arr) // 8
    arr1 = [0 for _ in range(length)]
    for j in range(length):
        arr1[j] = arr[j * 8] << 7 | arr[j * 8 + 1] << 6 | arr[j * 8 + 2] << 5 | arr[j * 8 + 3] << 4 | arr[j * 8 + 4] << 3 | arr[j * 8 + 5] << 2 | arr[j * 8 + 6] << 1 | arr[j * 8 + 7]
    return bytes(arr1)

def read(name):
    with open(name, 'rb') as f:
        return f.read()

def hook_code(mu, address, size, user_data):
    if address == BASE + 0x119cc:
        arr2 = []
        for byte in mu.mem_read(PLAINTEXT_ADDR, 8):
            arr2.append(byte)
        table.append([user_data.index(1), bytes2bin(arr2).index(1)])

if __name__ == "__main__":
    key0 = b'44e715a6e322ccb7d028f7a42fa55040'
    mu = Uc(UC_ARCH_ARM, UC_MODE_THUMB)
    BASE = 0x400000
    STACK_ADDR = 0x0
    STACK_SIZE = 1024
    PLAINTEXT_ADDR = 1024 * 2
    PLAINTEXT_SIZE = 1024
    KEY_ADDR = 1024 * 3
    KEY_SIZE = 1024
    mu.mem_map(BASE, 1024 * 1024)
    mu.mem_map(STACK_ADDR, STACK_SIZE)
    mu.mem_map(PLAINTEXT_ADDR, PLAINTEXT_SIZE)
    mu.mem_map(KEY_ADDR, KEY_SIZE)
    mu.reg_write(UC_ARM_REG_SP, STACK_ADDR + STACK_SIZE - 1)
    # mu.mem_write(BASE, read("F:\\Code\\Pycharm\\JDSign\\libjdbitmapkit.so"))
    mu.mem_write(BASE, read("./libjdbitmapkit.so"))
    mu.mem_write(KEY_ADDR, key0)

    for i in range(64):
        arr1 = [0 for j in range(64)]
        arr1[i] = 1
        h = mu.hook_add(UC_HOOK_CODE, hook_code, arr1)
        mu.mem_write(PLAINTEXT_ADDR, bin2bytes(arr1))
        mu.reg_write(UC_ARM_REG_R1, KEY_ADDR)
        mu.reg_write(UC_ARM_REG_R2, 32)
        mu.reg_write(UC_ARM_REG_R3, PLAINTEXT_ADDR)
        mu.emu_start(BASE + 0x00010EA4 + 1, BASE + 0x000119D0)
        mu.hook_del(h)
    print(table)
```

可以得到映射表，例如 `[1,4]` 表示 64 个比特位在打乱后第 1 个位置的比特位会到第 4 个位置。

```python
[[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2], [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21], [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55], [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41], [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28], [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50], [63, 48]]
```

然后可轻松还原 `sub_10EA4` 算法。

```python
def sub_10EA4(input):
    table = [[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2],
             [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21],
             [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55],
             [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41],
             [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28],
             [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50], [63, 48]]
    arr = bytes2bin(input)
    arr1 = [0 for i in range(len(arr))]
    for i in range(len(table)):
        arr1[table[i][1]] = arr[table[i][0]]
    return bin2bytes(arr1)
```

### `sub_10D70` 函数分析

`sub_10D70` 函数在 IDA 反编译也比较清晰，我们来看下。

![image-20210308212310967](https://bbs.kanxue.com/upload/attach/202103/802108_5QXABCYN6RJE5UP.png)

前面有说，待加密字符串会被每 8 个字节分为一组传进 `sub_10EA4` 进行加密，`sub_10D70` 会加密最后余出来的几个字节，这 6 个 case 就是加密余出来的 1 到 7 个字节。case 0 到 case 6 对应的六个函数都是一个模板，我们以 case 0 为例来分析，即 `sub_4B7C`。

### `sub_4B7C` 初始化

`sub_4B87` 只会加密处理一个字节，首先我们来看下 `sub_4B7C` 的流程图。

![image-20210308212954647](https://bbs.kanxue.com/upload/attach/202103/802108_6B3342JYE57VS4U.png)

可以看到密密麻麻，其他五个 case 也都长这样，看到后首先第一个反应就是，这是 ollvm 吗？在经过仔细的分析以及动态调试之后，我判断这个并不是 ollvm，没有看到控制流平坦化会带有的标志性大量的常量，也没有找到不可到达的分支，虚假控制流以及指令替换的痕迹，当然也可能是我水平太低了，没有认出来这种 ollvm，总之，我还是铁着头把整个流程从头到尾看了一遍。

来看下初始化，

![image-20210308214035243](https://bbs.kanxue.com/upload/attach/202103/802108_M9RXY7VFPH6RYB4.png)

看样子有点像 `sub_10EA4`，但仔细一看又很多不一样，上半部分取了传入的一个字节的比特位，还做了些其他的计算操作，赋值给变量。

而下半部分呢，则是取一些变量的地址，然后放到另一些变量地址加偏移处，这些说起来可能很模糊，但是如果看一下 `sub_4B7C` 的栈空间就会清晰很多。

为了方便理解，我们把这样连着五个变量在一起的当作一个数组，这样的数组往下拉可以看到是有八个，每个数组的前四个位置都存放着其他数组首的地址，而第五个位置则存放着前面说的输入字节的比特位经过计算后的值，其实分析后面的 case 就会发现，有多少输入的比特位，就会有多少个这样的数组。

![image-20210308214910555](https://bbs.kanxue.com/upload/attach/202103/802108_AF7CZG9VBJPBU2C.png)

初始化就这么多，而后面就开始根据这些东西进行大量的循环计算。

### `sub_4B7C` 计算分析

![image-20210308215648511](https://bbs.kanxue.com/upload/attach/202103/802108_7WDQ7DW75GME39J.png)

整个循环是根据 `key0` 的每个字节的每个比特位作为判断条件来选择分支，然后进行循环的，所有的计算类型只有两种，就是下面的两个大方框，是两个小循环。

它们的汇编如下。
