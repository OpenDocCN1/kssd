
# [原创]强网杯线上赛Linux内核Pwn之notebook

## 强网杯线上赛内核pwn notebook详解

刚刚从强网杯的线下赛回来，想整理一下这届强网杯我觉得不错的一些题or思路。

## 题目概览

缓解：`kaslr`，`smep`，`smap`

额外加固：<a href="elink@90cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4u0V1k6h3&6W2k6r3I4A6L8Y4g2^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8U0t1H3x3e0N6Q4x3V1j5I4x3W2)9J5c8U0l9J5i4K6u0r3L8r3W2F1N6i4S2Q4y4h3k6C8k6i4u0F1k6h3I4Q4y4h3j5@1i4K6u0W2x3e0c8Q4x3U0g2q4y4#2)9J5y4e0W2m8i4K6t1#2z5o6c8e0e0p5q4n7i4K6g2X3c8W2u0q4c8f1I4u0f1#2c8Q4y4h3k6t1b7g2u0p5c8f1&6q4c8q4)9J5y4f1f1%4i4K6t1#2z5f1q4Q4x3U0f1^5y4q4)9J5y4f1f1%4i4K6t1#2b7f1g2Q4x3U0f1^5x3q4)9J5y4f1f1^5i4K6t1#2b7e0k6Q4x3U0f1^5x3g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5k6h3S2@1L8h3H3`">slab hardened</a>

之前发现了一个从`bzImage`中提取`vmlinux`并恢复部分符号的脚本，还不错：

<mark class="encrypted">01dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4u0A6L8W2)9J5k6r3#2Q4x3V1k6$3L8h3I4A6L8Y4g2^5i4K6u0V1N6r3!0Q4x3X3c8W2L8r3j5`.</mark>

### mynote_ioctl

```c
\_\_int64 \_\_fastcall mynote\_ioctl(file \*file, unsigned int cmd, unsigned \_\_int64 arg)
{
\_\_int64 v3; // rdx
userarg notearg; // [rsp+0h] [rbp-28h]
\_fentry\_\_(file);
copy\_from\_user(&notearg, v3, 0x18LL);
if ( cmd == 0x100 )
return noteadd(notearg.idx, notearg.size, notearg.buf);
if ( cmd <= 0x100 )
{
if ( cmd == 0x64 )
return notegift(notearg.buf);
}
else
{
if ( cmd == 0x200 )
return notedel(notearg.idx);
if ( cmd == 0x300 )
return noteedit(notearg.idx, notearg.size, notearg.buf);
}
printk("[x] Unknown ioctl cmd!\n", notearg.size);
return -100LL;
}
```

### noteadd

```c
\_\_int64 \_\_fastcall noteadd(size\_t idx, size\_t my\_size, void \*buf)
{
\_\_int64 v3; // rdx
\_\_int64 user\_buf; // r13
note \*entry; // rbx
size\_t orgisize; // r14
\_\_int64 ret; // rbx
\_fentry\_\_(idx);
if ( idx > 0xF ) // 无法对0x10进行add
{
ret = -1LL;
printk("[x] Add idx out of range.\n", my\_size);
}
else // idx不越界的话
{
user\_buf = v3;
entry = &notebook[idx];
raw\_read\_lock(&lock); // 读锁
orgisize = entry->size; // 首先保存原有的size（如果对应的idx中本来就有信息）
entry->size = my\_size; // 赋值为新size
if ( my\_size > 0x60 ) // 如果超过0x60，恢复size，ret
{
entry->size = orgisize;
ret = -2LL;
printk("[x] Add size out of range.\n", my\_size);
}
else // size<=0x60为合法的size
{
copy\_from\_user(name, user\_buf, 0x100LL);
if ( entry->note ) // 如果note存在的话，恢复size，ret
{
entry->size = orgisize;
ret = -3LL;
printk("[x] Add idx is not empty.\n", user\_buf);
}
else // 如果note不存在，此时可以正常进行添加
{
entry->note = \_kmalloc(my\_size, 0x24000C0LL);
printk("[+] Add success. %s left a note.\n", name);
ret = 0LL;
}
}
raw\_read\_unlock(&lock);
}
return ret;
}
```

### noteedit

```c
\_\_int64 \_\_fastcall noteedit(size\_t idx, size\_t newsize, void \*buf)
{
\_\_int64 buf\_1; // rdx
\_\_int64 user\_buf; // r13
note \*entry; // rbx
size\_t orgisize; // rax
\_\_int64 addr; // r12
\_\_int64 ret; // rbx
\_fentry\_\_(idx);
if ( idx > 0xF ) // 无法对0x10进行edit
{
ret = -1LL;
printk("[x] Edit idx out of range.\n", newsize);
return ret;
}
user\_buf = buf\_1;
entry = &notebook[idx];
raw\_read\_lock(&lock); // 读锁
orgisize = entry->size; // 保存原本的size
entry->size = newsize; // 先把newsize赋值过去
if ( orgisize == newsize )
{
ret = 1LL;
goto editout;
}
addr = (\*krealloc.gap0)(entry->note, newsize, 0x24000C0LL);// free掉本来的chunk，然后申请新的chunk
copy\_from\_user(name, user\_buf, 0x100LL); // 从第三个参数user\_buf拷贝数据，如果此时userbuf没有初始化调入内存，那么会触发userfaultfd
if ( !entry->size ) // 如果新的size为0，等价于kfree(entry->note)
{
printk("free in fact", user\_buf);
entry->note = 0LL;
ret = 0LL;
goto editout;
}
if ( \_virt\_addr\_valid(addr) ) // 如果是有效地址的话
{
entry->note = addr; // 这里只用更新addr，因为size已经在前面更新过了
ret = 2LL;
editout:
raw\_read\_unlock(&lock);
printk("[o] Edit success. %s edit a note.\n", name);
return ret;
}
printk("[x] Return ptr unvalid.\n", user\_buf);
raw\_read\_unlock(&lock);
return 3LL;
}
```

### notegift

```c
\_\_int64 \_\_fastcall notegift(void \*buf)
{
\_\_int64 v1; // rsi
\_fentry\_\_(buf);
printk("[\*] The notebook needs to be written from beginning to end.\n", v1);
copy\_to\_user(buf, notebook, 0x100LL);
printk("[\*] For this special year, I give you a gift!\n", notebook);
return 100LL;
}
```

### mynote_write

```c
ssize\_t \_\_fastcall mynote\_write(file \*file, const char \*user\_buf, size\_t idx, loff\_t \*pos)
{
unsigned \_\_int64 idx\_1; // rdx
unsigned \_\_int64 index; // rdx
size\_t nbytes; // r13
void \*addr; // rbx
ssize\_t result; // rax
\_fentry\_\_(file);
if ( idx\_1 > 0x10 )
{
printk("[x] Write idx out of range.\n", user\_buf);
result = -1LL;
}
else
{
index = idx\_1;
nbytes = notebook[index].size;
addr = notebook[index].note;
\_check\_object\_size(addr, nbytes, 0LL);
if ( copy\_from\_user(addr, user\_buf, nbytes) )
{
printk("[x] copy from user error.\n", user\_buf);
result = 0LL;
}
else
{
printk("[\*] Write success.\n", user\_buf);
result = 0LL;
}
}
return result;
}
```

### mynote_read

```c
ssize\_t \_\_fastcall mynote\_read(file \*file, char \*user\_buf, size\_t idx, loff\_t \*pos)
{
unsigned \_\_int64 idx\_1; // rdx
unsigned \_\_int64 index; // rdx
size\_t nbytes; // r13
void \*addr; // rbx
ssize\_t result; // rax
\_fentry\_\_(file);
if ( idx\_1 > 0x10 ) // 可以read 0x10
{
printk("[x] Read idx out of range.\n", user\_buf);
result = -1LL;
}
else // 可以读取 0x10处的内容
{
index = idx\_1;
nbytes = notebook[index].size;
addr = notebook[index].note;
\_check\_object\_size(addr, nbytes, 1LL);
copy\_to\_user(user\_buf, addr, nbytes);
printk("[\*] Read success.\n", addr);
result = 0LL;
}
return result;
}
```

## 利用思路

本篇文章讲解的利用思路是依据线上赛冠军队 @长亭科技 的WP，感觉比我自己的要稳定和便利很多。

### 思路一(非race泄漏地址)

**Step1:** 首先我们多次打开`ptmx`结构体，在打开的时候会在`alloc_tty_struct`函数中调用`tty = kzalloc(sizeof(*tty), GFP_KERNEL);`分配空间（大小`0x2e0`），对应的是`kmalloc-1024`。
**Step2:** 然后系统会将`tty struct`中的`struct tty_operations`初始为全局内核变量`ptm_unix98_ops`，它是可以从`/proc/kallsyms`中获取相应地址/偏移。
**Step3:** 之后将喷射后的`tty struct`通过`close`给`free`出去。然后调用`noteedit`调整`note`大小到`0x2e0`，申请多个`0x2e0`的`note`，那么就会把我们`free`出去的`tty struct`拿回来。输出`((uint64_t *)note)[3]`位置的值减去偏移即可泄漏`kernel base`。

### 思路二(race构造UAF泄漏地址)

我们把目光放在两个用了读锁的函数。发现：
- 对于`noteedit`，首先会赋值`entry->size = new_size`，然后执行`krealloc`调整大小，然后会有一个`copy_from_user(name, user_buf, 0x100LL)`，在这之后才会更新`notelist`对应`entry->note`
- 对于`noteadd`，首先会赋值`entry->size = size`，如果`size`大于`0x60`则恢复为原有的`size`，之后先进行`copy_from_user(name, user_buf, 0x100LL);`然后再`kmalloc`更新`note entry`地址。
**于是这就产生了一个问题，如果将`user_buf`设置为一个为没有初始化的内存区域。并且不在`userfault`中起额外的`handler`做内存恢复。那么这两个函数就会一直卡在`copy_from_user`之前。**

**而`krealloc`又可以产生一个`free`操作，也就是说现在我们可以通过`userfault`来制造出一个任意大小的`UAF`。（因为被卡住的`thread`无法执行`copy_from_user`之后的更新`notelist`中相关地址的代码）**

**Step1:** 首先我们注册自己的`userfaultfd`，监控一块未初始化的内存，而在`add`和`edit`的时候由于其只拿了读写锁中的读锁，而不是互斥的写锁，那么这两个函数其实是可以并发的。我们注意到在`noteedit`和`noteadd`中都有`copy_from_user(name, user_buf, 0x100LL);`这个从用户态的`user_buf`拷贝的操作，那么我们设置`user_buf`为`mmap`后未初始化的内存，在这里触发`userfault`，当缺页发生时，程序会阻塞掉。我们将`notelist`填满`0x2e0`大小的`chunk`，然后开启`0x10`个`edit`线程，`0x10`个`add`线程。通过`edit`线程`krealloc`对应`note`为一个很大的`size`，此时会触发`krealloc`的`kfree`，将我们`0x2e0`的`chunk`进行`free`操作（但此时由于线程被卡在`copy_from_user`，`notelist`中的`addr`并没有被修改，仍是已经`free`后的`chunk addr`，但`size`此时变成了一个极大值，这里就是`UAF`的位置）；接下来喷射大量的`tty_struct`把我们`free chunk`中填成`tty_struct`。然后通过`add`线程修改`notelist`中的`chunksize`为正常大小以通过`__check_object_size`的检查。
**Step2:** 第一步结束之后，`notelist`上的一些`note`已经被放置成了`tty_struct`，我们通过`read`操作读取对应的`notelist`上的`tty_struct`中的成员，泄漏`kernel base`（参见思路一，实际情况下这里可能是`ptm_unix98_ops`或者`pty_unix98_ops`）
**Step3:** 接下来，我们通过伪造一个`fake_tty_ops`放到`notelist`的其中项，在`fake ops`中，我们让`ioctl`函数指向`work_for_cpu_fn`。

```c
__int64 __fastcall work_for_cpu_fn(_QWORD *a1)
{
_QWORD *v1; // rbx
__int64 (*v2)(void); // rax
__int64 v3; // rdi
__int64 result; // rax
_fentry_(a1);
v1 = a1;
v2 = a1[4]; // a1+32
v3 = a1[5]; // a1+40
result = _x86_indirect_thunk_rax(v2);
v1[6] = result; // a1+48
return result;
}
```
