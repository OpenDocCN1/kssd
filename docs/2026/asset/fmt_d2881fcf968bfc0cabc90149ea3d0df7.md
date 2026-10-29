
```python
def sub_10EA4(input):
table = [[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2],
[12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21],
[22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55],
[32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41],
[42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28],
[53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50],
[63, 48]]
arr = bytes2bin(input)
arr1 = [0 for i in range(len(arr))]
for i in range(len(table)):
arr1[table[i][1]] = arr[table[i][0]]
return bin2bytes(arr1)
```

### `sub_10D70` 函数分析

`sub_10D70`函数在IDA反编译也比较清晰，我们来看下。

![image-20210308212310967](upload/attach/202103/802108_5QXABCYN6RJE5UP.png)

前面有说，待加密字符串会被每8个字节分为一组传进`sub_10EA4`进行加密，`sub_10D70`会加密最后余出来的几个字节，这6个case就是加密余出来的1到7个字节。case 0到case 6对应的六个函数都是一个模板，我们以case 0为例来分析，即`sub_4B7C`。

### `sub_4B7C` 初始化

`sub_4B87`只会加密处理一个字节，首先我们来看下`sub_4B7C`的流程图。

![image-20210308212954647](upload/attach/202103/802108_6B3342JYE57VS4U.png)

可以看到密密麻麻，其他五个case也都长这样，看到后首先第一个反应就是，这是ollvm吗？在经过仔细的分析以及动态调试之后，我判断这个并不是ollvm，没有看到控制流平坦化会带有的标志性大量的常量，也没有找到不可到达的分支，虚假控制流以及指令替换的痕迹，当然也可能是我水平太低了，没有认出来这种ollvm，总之，我还是铁着头把整个流程从头到尾看了一遍。

来看下初始化，

![image-20210308214035243](upload/attach/202103/802108_M9RXY7VFPH6RYB4.png)

看样子有点像`sub_10EA4`，但仔细一看又很多不一样，上半部分取了传入的一个字节的比特位，还做了些其他的计算操作，赋值给变量。

而下半部分呢，则是取一些变量的地址，然后放到另一些变量地址加偏移处，这些说起来可能很模糊，但是如果看一下`sub_4B7C`的栈空间就会清晰很多。

为了方便理解，我们把这样连着五个变量在一起的当作一个数组，这样的数组往下拉可以看到是有八个，每个数组的前四个位置都存放着其他数组首的地址，而第五个位置则存放着前面说的输入字节的比特位经过计算后的值，其实分析后面的case就会发现，有多少输入的比特位，就会有多少个这样的数组。

![image-20210308214910555](upload/attach/202103/802108_AF7CZG9VBJPBU2C.png)

初始化就这么多，而后面就开始根据这些东西进行大量的循环计算。

### `sub_4B7C` 计算分析

![image-20210308215648511](upload/attach/202103/802108_7WDQ7DW75GME39J.png)

整个循环是根据`key0`的每个字节的每个比特位作为判断条件来选择分支，然后进行循环的，所有的计算类型只有两种，就是下面的两个大方框，是两个小循环。

它们的汇编如下。

```asm
.text:00005BE2 loc_5BE2 ; CODE XREF: sub_4B7C+104↑j
.text:00005BE2 CMP R1, R2
.text:00005BE4 MOV R12, R1
.text:00005BE6 BEQ loc_5C00
.text:00005BE8 MOV R6, R1
.text:00005BEA MOV R8, R2
.text:00005BEC B loc_5BF0
.text:00005BEE ; ---------------------------------------------------------------------------
.text:00005BEE
.text:00005BEE loc_5BEE ; CODE XREF: sub_4B7C+1082↓j
.text:00005BEE MOV R12, R6
.text:00005BF0
.text:00005BF0 loc_5BF0 ; CODE XREF: sub_4B7C+1070↑j
.text:00005BF0 LDRB R6, [R6,#0x10]
.text:00005BF2 STRB.W R6, [R8,#0x10]
.text:00005BF6 MOV R8, R12
.text:00005BF8 LDR.W R6, [R12,#0xC]
.text:00005BFC CMP R6, R2
.text:00005BFE BNE loc_5BEE
.text:00005C00
```

```asm
text:00005C00 CMP R11, R2
.text:00005C02 STRB.W R4, [R12,#0x10]
.text:00005C06 MOV R4, R11
.text:00005C08 LDRB.W R8, [SP,#0xE0+var_98]
.text:00005C0C IT NE
.text:00005C0E MOVNE R12, R2
.text:00005C10 BNE loc_5C16
.text:00005C12 B loc_5C2C
.text:00005C14 ; ---------------------------------------------------------------------------
.text:00005C14
.text:00005C14 loc_5C14 ; CODE XREF: sub_4B7C+10AE↓j
.text:00005C14 MOV R4, R6
.text:00005C16
.text:00005C16 loc_5C16 ; CODE XREF: sub_4B7C+1094↑j
.text:00005C16 LDRB R6, [R4,#0x10]
.text:00005C18 RSBS.W R6, R6, #1
.text:00005C1C IT CC
.text:00005C1E MOVCC R6, #0
.text:00005C20 STRB.W R6, [R12,#0x10]
.text:00005C24 LDR R6, [R4,#4]
.text:00005C26 MOV R12, R4
.text:00005C28 CMP R6, R2
.text:00005C2A BNE loc_5C14
```

这两个小循环做了什么呢？其实十分简单，就是不断地查刚才我们定义的数组的前四个位置存放的变量地址值，判断是否等于另一个变量的地址值，只不过判断的过程中会不断地移动这些数组的第五个位置的值。

### 使用unicorn还原case0——`sub_4B7C`算法

这个算法似乎很难搞，确实很难搞，不同于`sub_10EA4`的对输入字节的比特位进行简单的交换，在获取了比特位后又进行了很多计算。

该怎么办呢，我们来看下`sub_4B7C`算法的最后部分。

![image-20210308220953273](upload/attach/202103/802108_V7AWBUNK72Q5VYS.png)

可以看到和`sub_10EA4`函数最后的比特位还原部分几乎是一样的，这时候我们进行下大胆的猜测，`sub_4B7C`函数其实也是打乱输入字节的比特位进行了还原，只不过比特位在打乱后还进行了计算，而且每个比特位进行计算的规则都是一样的。即输入的字节第`x`个比特位是0的话，打乱计算还原后，第`y`个比特位会是`m`；如果输入的字节第`x`个比特位是1的话，第`y`个比特位则会是`n`，`(x, 0)——>(y, m)` `(x, 1)——>(y, n)`。后面经过实践，证明了这样的猜想是正确的。

然后我们还是可以借助unicorn来找到所有的映射关系，还原`sub_4B7C`算法。思路我们进行下改变，我们可以unicorn控制输入字节的比特位，用八个比特位只有一个1的计算结果和八个比特位全是0的计算结果进行对比从而得到所有的映射关系。

```python
from unicorn import *
from unicorn.arm_const import *
table = []
table1 = []
def bytes2bin(bytes):
arr = []
for v in [m for m in bytes]:
arr.append(
[(v & 128) >> 7, (v & 64) >> 6, (v & 32) >> 5, (v & 16) >> 4, (v & 8) >> 3, (v & 4) >> 2, (v & 2) >> 1,
v & 1])
return [i for j in arr for i in j]
def bin2bytes(arr):
length = len(arr) // 8
arr1 = [0 for _ in range(length)]
for j in range(length):
arr1[j] = arr[j * 8] << 7 | arr[j * 8 + 1] << 6 | arr[j * 8 + 2] << 5 | arr[j * 8 + 3] << 4 | arr[
j * 8 + 4] << 3 | arr[j * 8 + 5] << 2 | arr[j * 8 + 6] << 1 | arr[j * 8 + 7]
return bytes(arr1)
def read(name):
with open(name, 'rb') as f:
return f.read()
def hook_code(mu, address, size, user_data):
if address == BASE + 0x5284:
table.append(bytes2bin(mu.mem_read(PLAINTEXT_ADDR, 1)))
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
for i in range(9):
arr1 = [0 for j in range(8)]
if i != 0:
arr1[i-1] = 1
h = mu.hook_add(UC_HOOK_CODE, hook_code, arr1)
mu.mem_write(PLAINTEXT_ADDR, bin2bytes(arr1))
mu.reg_write(UC_ARM_REG_R0, KEY_ADDR)
mu.reg_write(UC_ARM_REG_R1, 32)
mu.reg_write(UC_ARM_REG_R2, 1)
mu.reg_write(UC_ARM_REG_R3, PLAINTEXT_ADDR)
mu.emu_start(BASE + 0x0004B7C + 1, BASE + 0x0005288)
mu.hook_del(h)
for i in range(8):
for j in range(8):
arr3 = []
if table[0][j] != table[i+1][j]:
table1.append([i, j, table[0][j], table[i+1][j]])
print(table1)
```

就结果为

```python
[[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0], [7, 7, 0, 1]]
```

我们可以得到为`[[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0], [7, 7, 0, 1]]`，里面的列表每个即为`[x, y, m, n]`，比如`[6, 1, 0, 1]`意味，第6个比特位在打乱计算后会放到第1个比特位上，如果第6个比特位是0，则在打乱计算后会放到第6个比特位上是0，反之是1。

然后可轻松还原`sub_4B7C`算法。

```python
def sub_4B7C(input):
table = [[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0],
[7, 7, 0, 1]]
arr = bytes2bin(input)
arr1 = [0 for i in range(8)]
for i in range(8):
if arr[i] == 0:
arr1[table[i][1]] = table[i][2]
else:
arr1[table[i][1]] = table[i][3]
return bin2bytes(arr1)
```

### 使用unicorn一步求出所有case的映射关系

好了，现在我们已经还原出第一个case，我们再会过头来看下`sub_10D70`，可以看到一共有七个case，分别对应的七个函数不同的只有函数起始结束地址，以及要处理的字节数，开拓下思维，这时候我们完全可以all in one，一次得到所有case的映射关系。

![image-20210308212310967](upload/attach/202103/802108_5QXABCYN6RJE5UP.png)

代码如下：
