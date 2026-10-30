
```
// 函数原型：
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream)

// 调用链：
fwrite(_IO_fwrite)
-> _IO_sputn (_IO_XSPUTN)
-> _IO_new_file_xsputn
// 1、首先判断输出缓冲区是否还有容量，如果有，则将目标输出数据拷贝至输出缓冲区。
// 2、如果输出缓冲区没有剩余（输出缓冲区未建立也是没有剩余）或输出缓冲区不够则调用 _IO_OVERFLOW 建立输出缓冲区或刷新输出缓冲区。
-> _IO_OVERFLOW
-> __overflow（_IO_new_file_overflow）
->
// 2.1 判断标志位是否包含 _IO_NO_WRITES，若包含，则直接返回
-> _IO_doallocbuf
-> _IO_file_doallocate
// 2.2.1 判断输出缓冲区是否为空，若空，则调用 _IO_doallocbuf 去分配
-> _IO_setg
// 2.2.2 接2.2，如果为空，分配输出缓冲区后，设置read相关的三个指针
->
// 2.3 初始化其他指针，最主要的是write相关的三个指针
-> _IO_do_write(_IO_new_do_write)
// 2.4 调用系统调用write输出输出缓冲区，输出的内容为f->_IO_write_ptr到f->_IO_write_base之间的内容
-> _IO_SYSWRITE
-> __write（_IO_new_file_write）
-> write
// 3、输出缓冲区刷新后判断剩余的目标输出数据是否超过块的size，如果超过块的size，则不通过输出缓冲区直接以块为单位，使用 _IO_new_do_write 输出目标数据。
// 4、如果按块输出数据后还剩下一点数据则调用 _IO_default_xsputn 将数据拷贝至输出缓冲区。
-> _IO_default_xsputn

// 调用vtable中的函数
// 1、_IO_fwrite 函数调用了vtable的 _IO_new_file_xsputn。
// 2、_IO_new_file_xsputn 函数调用了vtable中的 _IO_new_file_overflow 实现缓冲区的建立以及刷新缓冲区。
// 3、vtable中的 _IO_new_file_overflow 函数调用了vtable的 _IO_file_doallocate 以初始化输入缓冲区。
// 4、vtable中的_IO_file_doallocate调用了vtable中的 __GI__IO_file_stat 以获取文件信息。
// 5、new_do_write中的_IO_SYSWRITE调用了vtable的 _IO_new_file_write 最终去执行系统调用write。
```

- fclose调用链：

```
// 函数原型：
int fclose(FILE *stream)

// 调用链：
fclose(_IO_new_fclose)
-> _IO_un_link // 将FILE结构体从_IO_list_all链表中取下
-> _IO_file_close_it // 关闭文件并释放缓冲区
-> _IO_file_is_open // _IO_file_is_open宏检查该文件是否处于打开的状态
-> _IO_do_flush // _IO_do_flush 刷新此时的输出缓冲区
-> _IO_do_write // 调用系统调用将缓冲区的内容输出到文件，并刷新输出缓冲区的值。
-> _IO_SYSCLOSE
-> __close
-> _IO_setb // 设置结构体的buf指针，并释放缓冲区
-> _IO_setg // 设置read相关的指针
-> _IO_setp // 设置write相关的指针
-> _IO_un_link // 确保FILE结构体已从_IO_list_all中取下
-> _IO_FINISH // 进行最后的确认，确认FILE结构体从链表中删除以及缓冲区被释放
-> __finish
-> free // free释放IO_FILE结构体内存

// 调用vtable中的函数：
// 1、在清空缓冲区的_IO_do_write函数中会调用vtable中的函数。
// 2、关闭文件描述符_IO_SYSCLOSE函数为vtable中的__close函数。
// 3、_IO_FINISH函数为vtable中的__finish函数。
```

## 1.3 文件链表

通过`_IO_FILE *_chain`实现链表结构，头部是全局变量`_IO_list_all`

![](https://bbs.kanxue.com/upload/attach/202210/830671_8Z7FAGKUCFZXXSJ.png)

# 2 `_IO_FILE`攻击

## 2.1 虚函数表vtable劫持

如果能够控制`_IO_FILE_plus`结构体，实现对`vtable`指针的修改，使得**`vtable`指向可控的内存，在该内存中构造好`vtable`，再通过调用相应IO函数，触发`vtable`函数的调用，即可劫持程序执行流**。

劫持最关键的点在于修改IO `FILE`结构体的`vtable`指针，指向可控内存。一般来说有两种方式：**一种是只修改内存中已有`FILE`结构体的`vtable`字段；另一种则是伪造整个`FILE`结构体。**当然，两种的本质最终都是**修改了`vtable`字段**。

- **攻击条件**

  - **有可控内存**（视情况而定） -> 伪造`FILE`结构体

  - **任意地址写** -> 修改`vtable`指针

例子可参考：

- [零基础要如何破除 `IO_FILE` 利用原理的迷雾](<https://bbs.kanxue.com/elink@675K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6s2c8S2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3e0M7@1x3W2)9J5c8W2)9J5x3%4c8G2j5#2)9#2k6R3`.`.)>)

- [IO FILE 之劫持vtable及FSOP](<https://bbs.kanxue.com/elink@de8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4W2Q4x3X3c8U0M7q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5i4u0U0K9r3W2$3k6i4u0K6i4K6u0r3d9f1!0Q4y4h3k6r3d9f1I4q4i4K6g2X3N6Y4c8S2j5X3I4W2i4K6g2X3K9r3q4B7j5h3y4C8i4K6g2X3j5h3&6V1i4K6g2X3k6Y4y4G2M7l9`.`.)>)

## 2.2 FSOP

FSOP（File Stream Oriented Programming）的核心思想就是**劫持`_IO_list_all`的值来伪造链表和其中的`_IO_FILE`项**，但是单纯的伪造只是构造了数据，还需要某种方法进行触发。FSOP 选择的触发方法是**调用`_IO_flush_all_lockp`**，这个函数会刷新`_IO_list_all`链表中所有项的文件流，相当于对每个`FILE`调用`fflush`，也对应着会调用`_IO_FILE_plus.vtable`中的`_IO_overflow`。

- **`_IO_flush_all_lockp`被系统调用的时机**

  - 当 `libc` 执行 `abort` 流程时

  - 当执行 `exit` 函数时

  - 当执行流从 `main` 函数返回时

- **`_IO_flush_all_lockp`中调用`_IO_OVERFLOW`的条件**，根据短路原理可知需满足：

  - `fp->_mode <= 0`

  - `fp->_IO_write_ptr > fp->_IO_write_base`

```
if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base))
    && _IO_OVERFLOW (fp, EOF) == EOF)
{
    result = EOF;
}
```

- **FSOP攻击条件**

  - **能泄露出 libc 的基址** -> 泄露出`_IO_list_all`的地址

  - **有可控内存** -> 伪造`_IO_FILE`和`vtable`

  - **任意地址写** -> 将`_IO_list_all`的内容改为指向可控内存的指针

> [ctf-wiki-FSOP](<https://bbs.kanxue.com/elink@820K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3W2G2i4K6u0V1k6X3W2D9k6g2)9J5c8X3k6K6L8%4m8Q4x3V1j5.)>)

### 2.2.1 ctf实例 - `ciscn_2019_n_7`

```
lzx@ubuntu16x64:~/pwn/heap/IO_FILE/ciscn_2019_n_7$ ls
ciscn_2019_n_7  log.txt
lzx@ubuntu16x64:~/pwn/heap/IO_FILE/ciscn_2019_n_7$ ./ciscn_2019_n_7

1.add page
2.edit page
3.show page
4.exit
Your choice->
Alarm clock
```

#### ida分析

- `main`

```
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  int v3; // eax
  __int64 v4; // rdx
  __int64 v5; // rcx
  bool v6; // zf
  bool v7; // sf
  unsigned __int8 v8; // of

  sub_CA0();
LABEL_2:
  while ( 2 )
  {
    while ( 1 )
    {
      v3 = sub_D80(a1, a2);
      v8 = __OFSUB__(v3, 3);
      v6 = v3 == 3;
      v7 = v3 - 3 < 0;
      if ( v3 != 3 )
        break;
LABEL_7:
      show(); // 3 - show
    }
    while ( (unsigned __int8)(v7 ^ v8) | v6 )
    {
      if ( v3 == 1 ) // 1 - add
      {
        add();
        goto LABEL_2;
      }
      if ( v3 != 2 )
        goto LABEL_11;
      edit(); // 2 - edit
      v3 = sub_D80(a1, a2);
      v8 = __OFSUB__(v3, 3);
      v6 = v3 == 3;
      v7 = v3 - 3 < 0;
      if ( v3 == 3 )
        goto LABEL_7;
    }
    if ( v3 == 4 )
      exit_(); // 4 - exit_
    if ( v3 == 666 )
    {
      sub_C50(a1, (__int64)a2, v4, v5); // 666 - 打印puts函数地址
      continue;
    }
    break;
  }
LABEL_11:
  puts("NO, Please continue! ");
  return 0LL;
}
```

- `main` 开头的 `sub_CA0`

```
unsigned int sub_CA0()
{
  FILE *v0; // rbx
  char v1; // al
  unsigned int result; // eax
  unsigned __int64 v3; // rt1
  unsigned __int64 v4; // [rsp+8h] [rbp-20h]

  v4 = __readfsqword(0x28u);
  v0 = fopen("log.txt", "r");
  while ( 1 )
  {
    v1 = fgetc(v0);
    if ( v1 == -1 )
      break;
    IO_putc(v1, stdout);
  }
  fclose(v0);
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 1, 0LL);
  setvbuf(stderr, 0LL, 1, 0LL);
  global = malloc(0x18uLL); // malloc了一个chunk给全局变量
  v3 = __readfsqword(0x28u);
  result = v3 ^ v4;
  if ( v3 == v4 )
    result = alarm(0x3Cu);
  return result;
}
```

- `add` - 溢出
