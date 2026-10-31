
# [原创]house of pig详解

在复现这题之前需要了解一些前置知识：libc2.31下的`largebin_attack`、`tcache_stashing_unlink_plus`以及高版本glibc下的`IO_FILE`攻击

首先看到libc2.31下的`largebin_attack`

## 0x1. libc2.31下的`largebin_attack`

跟随how2heap项目中的`largebin_attack`以及源码调试来学习。

从libc2.30开始，`largebin`的插入代码中新增了两个检查

先看到第一个点

[![2c5gYj.jpg](upload/attach/202106/865862_MRM2GE26G88QN6J.jpg)](<mark class="encrypted">79dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5j5K6g2Y4h3h3A6Q4x3U0V1`.</mark>

将`unsortedbin`插入到`largebin`中时，且这个`unsortedbin`大于`largebin`的`size`，此时插入过程增加了双向链表完整性检查。

通常就是修改`largebin`的`bk_nextsize = target_addr - 0x20`，然后在插入一个比原有`largebin`更大的`unsortedbin`时（后面称原有的`largebin`为`largebin1`，新插入的为`largebin2`），在插入过程中，`largebin1`的`bk_nextsize`被设置为`largebin1`的`bk_nextsize`，即`target_addr - 0x20`，后续`victim->bk_nextsize->fd_nextsize = victim`这条语句，会将`target_addr - 0x20 + 0x20`位置写入`largebin2`的地址，这是第一个点。

第二个点如下

[![2chIg0.png](upload/attach/202106/865862_YYKJJJ9DVAXQW4T.jpg)](<mark class="encrypted">a89K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5j5$3S2u0k6K6m8Q4x3U0V1`.</mark>

这里的利用方式是修改`largebin1`的`bk = target_addr - 0x10`，`bck = fwd->bk; bck->fd = victim;`这两句代码执行完毕后会将`target_addr - 0x10 + 0x10`的位置写入`largebin1`的地址。

如图所示，这两处都添加了检查。

但当要插入的`unsortedbin`小于`largebin`的`size`时并没有做检查，如下图

[![2cIbb8.jpg](upload/attach/202106/865862_AF9BB2HKR5MWE8C.jpg)](<mark class="encrypted">21bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5j5@1W2T1j5U0S2Q4x3U0V1`.</mark>

在这里并没有进行检查，因此在libc2.31下这里就成了新的利用点。

demo文件如下

```c
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
/*
A revisit to large bin attack for after glibc2.30
Relevant code snippet :
if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk)){
fwd = bck;
bck = bck->bk;
victim->fd_nextsize = fwd->fd;
victim->bk_nextsize = fwd->fd->bk_nextsize;
fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
*/
int main(){
/*Disable IO buffering to prevent stream from interfering with heap*/
setvbuf(stdin,NULL,_IONBF,0);
setvbuf(stdout,NULL,_IONBF,0);
setvbuf(stderr,NULL,_IONBF,0);
size_t target = 0;
size_t *p1 = malloc(0x428);
size_t *g1 = malloc(0x18);
size_t *p2 = malloc(0x418);
size_t *g2 = malloc(0x18);
free(p1);
size_t *g3 = malloc(0x438);
free(p2);
p1[3] = (size_t)((&target)-4);
size_t *g4 = malloc(0x438);
assert((size_t)(p2-2) == target);
return 0;
}
```

我删掉了原文件中的一些描述性代码，以便于观看代码。

整体攻击思路就是申请一大一小两个`chunk`（后面称为`chunk1`，`chunk2`），先`free`掉`chunk1`，然后申请一个更大的`chunk`来将`chunk1`从`unsortedbin`中插入到`largebin`，接着将`chunk1`的`bk_nextsize`设置为`target_addr - 0x20`，这是第一步；第二步，`free`掉`chunk2`，然后申请一个更大的`chunk`来将`chunk2`从`unsortedbin`中插入到`largebin`中，由于此时插入的`chunk2`的`size`要小于`chunk1`，所以会触发新的攻击流程，这里我们采用源码调试，以便更直观地学习。

在程序执行到`size_t *g4 = malloc(0x438);`这一句时，堆的情况如下

[![2Ro900.png](upload/attach/202106/865862_YZ3DH3DFCJFCRBY.jpg)](<mark class="encrypted">6f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5f1X3)9&6x3o6m8Q4x3U0V1`.</mark>

`largebin`里放着`0x430`的`chunk`，`unsortedbin`里面则是`0x420`的

[![2Ro1hD.png](upload/attach/202106/865862_JYMNK5A98KDYQF6.jpg)](<mark class="encrypted">190K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5f1X3)9I4K9p5c8Q4x3U0V1`.</mark>

[![2RoB4S.png](upload/attach/202106/865862_FKXQ4EPRZHMB5J8.jpg)](<mark class="encrypted">577K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5f1X3!0n7y4q4y4Q4x3U0V1`.</mark>

`chunk1`的`bk_nextsize`被设置为了`target_addr - 0x20`

接下来我们将断点下在`_int_malloc`函数

[![2Roo34.png](upload/attach/202106/865862_YEFPWJHT58Y7U7W.jpg)](<mark class="encrypted">581K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5f1X3!0G2x3K6c8Q4x3U0V1`.</mark>

[![2RoLHx.png](upload/attach/202106/865862_GN7W7ERAR8V8U7Z.jpg)](<mark class="encrypted">57cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5f1X3!0x3d9s2S2Q4x3U0V1`.</mark>

然后我们运行到将`unsortedbin`插入到`largebin`的代码

[![2WoKdP.png](upload/attach/202106/865862_D7986YAH9PKBMS2.jpg)](<mark class="encrypted">8ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2$3!0w2k6q4m8Q4x3U0V1`.</mark>

首先获取要插入的`unsortedbin`对应的`largebin`的`index`，然后获取到对应的链表头

[![2WoUZq.png](upload/attach/202106/865862_KMH3KZDE38T6CUR.jpg)](<mark class="encrypted">94bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2$3!0g2h3Y4q4Q4x3U0V1`.</mark>

由于此时`largebin`中已经有了一个`chunk`，所以对应链表头的`fd`和`bk`都被设置为了这个`largebin`的地址，类似于下面这样

[![2WLTl8.png](upload/attach/202106/865862_C5DT6A34F5J2AJ9.jpg)](<mark class="encrypted">e5dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2@1I4f1L8o6S2Q4x3U0V1`.</mark>

然后进入到插入环节

[![2WoHOA.png](upload/attach/202106/865862_GXNJEUBYCXSGVVF.jpg)](<mark class="encrypted">c0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2$3!0t1e0@1q4Q4x3U0V1`.</mark>

将`bck`，也就是链表头赋值给`fwd`，将`bck->bk`（`chunk1`的地址）赋值给`bck`，进入到插入操作，首先将`chunk2`（即将插入的`chuhnk`）的`fd_nextsize`设置为`chunk1`的地址

[![RGPD3Q.png](upload/attach/202106/865862_MKHC6B7XCGRXSGK.jpg)](<mark class="encrypted">0cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9c8#2m8p5x3#2q4Q4x3U0V1`.</mark>

再将`chunk2`的`bk_nextsize`设置为`chunk1`的`bk_nextsize`，而`chunk1`的`bk_nextsize`已经被修改为了`target_addr - 0x20`，因此`chunk2`的`bk_nextsize`也会指向`target_addr - 0x20`

[![2WHyQg.png](upload/attach/202106/865862_UPVNXKFHRBNDTZU.jpg)](<mark class="encrypted">7b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2@1S2&6f1h3N6Q4x3U0V1`.</mark>

最后一行代码用于修改`chunk1`的`fd_nextsize`和`bk_nextsize`为`chunk2`的地址，由于设置`chunk1`的`fd_nextsize`是通过 `victim->bk_nextsize->fd_nextsize`来设置的，而`victim->bk_nextsize`指向的是一个错误的地址，执行完这条赋值语句后就会在`target_addr + 0x20`的位置上写入`chunk2`的地址

[![2WX5M8.png](upload/attach/202106/865862_7FP424BRMKUY56X.jpg)](<mark class="encrypted">e12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5g2#2R3#2e0e0S2Q4x3U0V1`.</mark>

至此就实现了类似于libc2.23下的`unsortedbin attack`，往任意地址写入一个堆地址。

## 0x2. `tcache_stashing_unlink_plus`

此种利用方式可以达成任意地址处分配一个`chunk`
