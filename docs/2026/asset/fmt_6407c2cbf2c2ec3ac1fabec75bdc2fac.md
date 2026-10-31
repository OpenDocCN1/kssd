
此时申请走7个`tcachebin` `chunk`，再次申请的时候，满足了`tcachebin`没满，`fastbin`有多个`chunk`的条件，会进入`Tcache Reverse Into Fastbin`的流程

此时`fastbin`中，假定`chunk`链表为：`A->B->C`，则`A`会被分配走，然后`B`和`C`会依次填入`tcachebin`

如果`C`的`fd`被我们控制，指向了我们指定的地方（2.41下，目标地址需要其`fd`解密结果是0），那么就会将指定地方作为`tcache` `chunk`存入`tcachebin`中

### 源码分析：2.41 下的 fastbin dup 技巧

2.41下`fastbin dup`和之前没有什么区别，其流程发生在`free`过程中：

```
unsigned int idx = fastbin_index(size);
fb = &fastbin (av, idx);
// 取出 fastbin 第一个 chunk
/* Atomically link P to its fastbin: P->FD = *FB; *FB = P;  */
mchunkptr old = *fb, old2;

if (SINGLE_THREAD_P)
{
  /* Check that the top of the bin is not the record we are going to
     add (i.e., double free).  */
  if (__builtin_expect (old == p, 0))
    // 检查和fastbin第一个chunk是否相同，相同则认为发生了 double free
    malloc_printerr ("double free or corruption (fasttop)");
  p->fd = PROTECT_PTR (&p->fd, old);
  // 加密指针，存入
  *fb = p;
}
```

只检查在`fastbin`链表中第一个`chunk`，和释放的`chunk`是否相同

只需要申请两个`chunk`，`A`和`B`

按照`A`，`B`，`A`的顺序释放

`fastbin`的链就会变成`A->B->A`

### Tcache Reverse Into Fastbin 组合 fastbin dup 技巧

如果说，在之前设想的`Tcache Reverse Into Fastbin`场景里，我的`fastbin` `chunk`的链通过`fastbin dup`之后变成了`A->B->A`

此时申请相同大小的`chunk`，触发`Tcache Reverse Into Fastbin`的流程，就会将`A`分配走，然后将`B`插入`tcachebin`，再将`A`插入`tcachebin`

程序流程中会向`A`中写入数据，此时写入目标地址（需要其`fd`处解密结果是0），那么`tcache`链就会由`A->B`变成`A->目标`

最终可以得到一个任意地址分配的原语，如果`fd`处解密结果不为0，就会向`tcache`链表写入一个不可访问的地址，以至于该`size`的`tcahebin`不能再使用，题目只能用2个`size`的`chunk`

### 任意地址分配原语

这里是按照流程走下来之后，完成`fastbin dup`之后，准备触发`Tcache Reverse Into Fastbin`的时候，此时我在第一个`chunk`处准备了经过解密可以得到`0`的值：

```
0x5643909ab380  0x0000000000000000      0x0000000000000061      ........a.......
0x5643909ab390  0x00000005643909ab      0x00000005643909ab      ..9d......9d....
0x5643909ab3a0  0x00000005643909ab      0x00000005643909ab      ..9d......9d....
0x5643909ab3b0  0x00000005643909ab      0x00000005643909ab      ..9d......9d....
0x5643909ab3c0  0x00000005643909ab      0x00000005643909ab      ..9d......9d....
0x5643909ab3d0  0x00000005643909ab      0x00000005643909ab      ..9d......9d....
0x5643909ab3e0  0x00000005643909ab      0x0000000000000061      ..9d....a.......
...
0x5643909ab620  0x4242424242424242      0x0000000000000061      BBBBBBBBa.......         <-- fastbins[0x60][0], fastbins[0x60][0]
0x5643909ab630  0x00005646f4a3bf2b      0x4141414141414141      +...FV..AAAAAAAA
0x5643909ab640  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab650  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab660  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab670  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab680  0x4141414141414141      0x0000000000000061      AAAAAAAAa.......         <-- fastbins[0x60][1]
0x5643909ab690  0x00005646f4a3bf8b      0x4141414141414141      ....FV..AAAAAAAA
0x5643909ab6a0  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab6b0  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab6c0  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab6d0  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab6e0  0x4141414141414141      0x0000000000000061      AAAAAAAAa.......
0x5643909ab6f0  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab700  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab710  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab720  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab730  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab740  0x4141414141414141      0x0000000000000061      AAAAAAAAa.......
0x5643909ab750  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab760  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab770  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab780  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab790  0x4141414141414141      0x4141414141414141      AAAAAAAAAAAAAAAA
0x5643909ab7a0  0x4141414141414141      0x000000000001f861      AAAAAAAAa.......         <-- Top chunk
```

此时，我再申请一个`0x60`的`chunk`，触发`Tcache Reverse Into Fastbin`：
