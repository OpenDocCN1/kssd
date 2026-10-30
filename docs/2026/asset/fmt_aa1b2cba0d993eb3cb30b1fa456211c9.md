
# FSOP：`_IO_file_list` 中的函数是怎么被执行的？

> 2.24 开始把 `vtable` 指针设置成了只读，且会检查 `vtable` 调用的指针是否在 `vtable` 所在段，以至于该方法在 2.24 中失效

> 本段内容来自 ctf wiki[5]

FSOP 是 File Stream Oriented Programming 的缩写，根据前面对 `FILE` 的介绍得知进程内所有的 `_IO_FILE` 结构会使用 `_chain` 域相互连接形成一个链表，这个链表的头部由 `_IO_list_all` 维护。

FSOP 的核心思想就是劫持 `_IO_list_all` 的值来伪造链表和其中的 `_IO_FILE` 项，但是单纯的伪造只是构造了数据还需要某种方法进行触发。

FSOP 选择的触发方法是调用 `_IO_flush_all_lockp()`，这个函数会刷新 `_IO_list_all` 链表中所有项的文件流，相当于对每个 `FILE` 调用 `fflush()`，也对应着会调用 `_IO_FILE_plus.vtable` 中的 `_IO_overflow`。

```c
int _IO_flush_all_lockp (int do_lock)
{
  ...
  fp = (_IO_FILE *) _IO_list_all;
  while (fp != NULL)
    {
      ...
      if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base))
          && _IO_OVERFLOW (fp, EOF) == EOF)
    {
      result = EOF;
    }
      ...
    }
}
```

触发 `_IO_flush_all_lockp()` 的条件是：

- 当 `libc` 执行 `abort()` 流程时
- 当执行 `exit()` 函数时
- 当执行流从 `main()` 函数返回时

执行 `_IO_OVERFLOW` 的条件是：

- `fp->_mode <= 0`
- `fp->_IO_write_ptr > fp->_IO_write_base`

其中，`_IO_list_all` 是 `libc` 的全局变量，泄露 `libc` 地址即可拿到该指针所在地址。

利用 FSOP 需要做的是：

1. 泄露 `libc` 地址；
2. 伪造 `_IO_list_all` 结构，覆盖其 `overflow` 指针为目标函数；
3. 覆盖 `_IO_list_all` 指针为可控地址；
4. 触发程序错误或退出程序。

伪造结构：

```c
struct _IO_FILE {
    int _flags; /* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

    /* The following pointers correspond to the C++ streambuf protocol. */
    /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
    char* _IO_read_ptr; /* Current read pointer */
    char* _IO_read_end; /* End of get area. */
    char* _IO_read_base; /* Start of putback+get area. */
    char* _IO_write_base; /* Start of put area. */
    char* _IO_write_ptr; /* Current put pointer. */
    char* _IO_write_end; /* End of put area. */
    char* _IO_buf_base; /* Start of reserve area. */
    char* _IO_buf_end; /* End of reserve area. */
    /* The following fields are used to support backing up and undo. */
    char *_IO_save_base; /* Pointer to start of non-current get area. */
    char *_IO_backup_base; /* Pointer to first valid character of backup area */
    char *_IO_save_end; /* Pointer to end of non-current get area. */

    struct _IO_marker *_markers;

    struct _IO_FILE *_chain;

    int _fileno;
#if 0
    int _blksize;
#else
    int _flags2;
#endif
    _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

#define __HAVE_COLUMN /* temporary */
    /* 1+column number of pbase(); 0 is unknown. */
    unsigned short _cur_column;
    signed char _vtable_offset;
    char _shortbuf[1];

    /*  char* _save_gptr;  char* _save_egptr; */

    _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};

struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
#if 0
    get_column;
    set_column;
#endif
};
```

## 实验：hitconctf2016 - houseoforange

**还有很多重要的细节，在接下来的实例中进行分析**

### 题目分析：

菜单题：

```text
+++++++++++++++++++++++++++++++++++++
@          House of Orange          @
+++++++++++++++++++++++++++++++++++++
1. Build the house
2. See the house
3. Upgrade the house
4. Give up
+++++++++++++++++++++++++++++++++++++
Your choice :
```

首先是选项1：

```c
int sub_D37()
{
  unsigned int houseCount; // [rsp+8h] [rbp-18h]
  int color; // [rsp+Ch] [rbp-14h]
  void *house; // [rsp+10h] [rbp-10h]
  _DWORD *orange; // [rsp+18h] [rbp-8h]

  if ( count > 3u )
  {
    puts("Too many house");
    exit(1);
  }
  house = malloc(0x10uLL); // 第一次 malloc，大小固定0x10
  printf("Length of name :");
  houseCount = convertStringToInt();
  if ( houseCount > 0x1000 )
    houseCount = 4096; // 最大 0x1000
  *((_QWORD *)house + 1) = malloc(houseCount); // 第二次 malloc，大小自定
  if ( !*((_QWORD *)house + 1) )
  {
    puts("Malloc error !!!");
    exit(1);
  }
  printf("Name :");
  call_read(*((void **)house + 1), houseCount);
  orange = calloc(1uLL, 8uLL); // 第三次 calloc，大小固定0x8
  printf("Price of Orange:");
  *orange = convertStringToInt(); // 输入一个整数，可以很长
  color_menu();
  printf("Color of Orange:");
  color = convertStringToInt();
  if ( color != 56746 && (color <= 0 || color > 7) )
  {
    puts("No such color");
    exit(1);
  }
  if ( color == 56746 )
    orange[1] = 56746;
  else
    orange[1] = color + 0x1E;
  *(_QWORD *)house = orange;
  g_house = house;
  ++count;
  return puts("Finish");
}
```

主要就2个点：

1. 该选项最多用4次
2. 每次使用会申请3个chunk，其中第1和第3个大小固定，第二个可选

然后是选项2：

```c
int show()
{
  int temp; // eax
  int random_num; // eax

  if ( !g_house )
    return puts("No such house !");
  if ( *(_DWORD *)(*g_house + 4LL) == 56746 )
  {
    printf("Name of house : %s\n", (const char *)g_house[1]); // 显示信息
    printf("Price of orange : %d\n", *(unsigned int *)*g_house); // 显示信息
    temp = rand();
    return printf("\x1B[01;38;5;214m%s\x1B[0m\n", *((const char **)&unk_203080 + temp % 8));
  }
  else
  {
    if ( *(int *)(*g_house + 4LL) <= 30 || *(int *)(*g_house + 4LL) > 37 )
    {
      puts("Color corruption!");
      exit(1);
    }
    printf("Name of house : %s\n", (const char *)g_house[1]);
    printf("Price of orange : %d\n", *(unsigned int *)*g_house);
    random_num = rand();
    return printf(
      "\x1B[%dm%s\x1B[0m\n",
      *(unsigned int *)(*g_house + 4LL),
      *((const char **)&unk_203080 + random_num % 8));
  }
}
```

这个就是用来显示信息的

选项3：
