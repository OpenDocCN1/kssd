
# [原创]新人PWN入坑总结(七)

## FSOP0xd

### 一、FILE前置知识

#### 一、`_IO_FILE_plus`：

1. 结构定义：在`libio.h`中找不到它的结构定义，但是`wiki`上说是如下的：

```
#注释头



struct _IO_FILE_plus

{

    _IO_FILE    file;

    IO_jump_t   *vtable;

}
```

虽然网上相关定义的里面包含的是`_IO_FILE`，但是实际上在内存中`_IO_FILE`却会被完善成`_IO_FILE_complete`:
![](https://bbs.kanxue.com/upload/attach/202108/904686_2FC9NS29JMGQRPT.jpg)
(1) 两条黄线之间的数据即为`struct _IO_FILE_complete`，包含`struct _IO_FILE`
(2) 第一条黄线至蓝线的即为`struct _IO_FILE`
(3) 最下面的就是`struct _IO_jump_t`

2. 符号内容：

```
#注释头



_IO_2_1_stderr_

_IO_2_1_stdout_

_IO_2_1_stdin_
```

以上三个符号就是程序被加载之后的`_IO_FILE_plus`这个结构体生成的结构体指针。

#### 二、`_IO_FILE`和`_IO_FILE_complete`(`libio.h`中)

1. `_IO_FILE`结构定义：

```
#注释头



struct _IO_FILE {

  int _flags;       /* High-order word is _IO_MAGIC; rest is flags. */

#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */

  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */

  char* _IO_read_ptr;   /* Current read pointer */

  char* _IO_read_end;   /* End of get area. */

  char* _IO_read_base;  /* Start of putback+get area. */

  char* _IO_write_base; /* Start of put area. */

  char* _IO_write_ptr;  /* Current put pointer. */

  char* _IO_write_end;  /* End of put area. */

  char* _IO_buf_base;   /* Start of reserve area. */

  char* _IO_buf_end;    /* End of reserve area. */

  /* The following fields are used to support backing up and undo. */

  char *_IO_save_base; /* Pointer to start of non-current get area. */

  char *_IO_backup_base;  /* Pointer to first valid character of backup area */

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
```

2. `_IO_FILE_complete`结构定义：

```
#注释头



struct _IO_FILE_complete

{

  struct _IO_FILE _file;

#endif

#if defined _G_IO_IO_FILE_VERSION && _G_IO_IO_FILE_VERSION == 0x20001

  _IO_off64_t _offset;

# if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T

  /* Wide character stream stuff.  */

  struct _IO_codecvt *_codecvt;

  struct _IO_wide_data *_wide_data;

  struct _IO_FILE *_freeres_list;

  void *_freeres_buf;

# else

  void *__pad1;

  void *__pad2;

  void *__pad3;

  void *__pad4;

# endif

  size_t __pad5;

  int _mode;

  /* Make sure we don't get into trouble again.  */

  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];

#endif

};
```

3. 实际内存情况：
由于程序中基本都会自动完善，所以只能查看到`_IO_FILE_complete`的实际内存结构
![](https://bbs.kanxue.com/upload/attach/202108/904686_38SD23U5QC3R28U.jpg)

4. 符号内容：

```
#注释头



_IO_stderr_

_IO_stdout_

_IO_stdin_
```

这三个符号就是`struct _IO_FILE`的结构体指针，但是实际会被完善为`_IO_FILE_complete`。
![](https://bbs.kanxue.com/upload/attach/202108/904686_MSFMSG5E2EPTHNZ.jpg)

#### 三、`IO_jump_t`结构：

1. 结构定义：

```
#注释头



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

2. 实际内存情况：
![](https://bbs.kanxue.com/upload/attach/202108/904686_JZ6HAX6M6UHDDJS.jpg)

3. 符号内容：
`_IO_file_jumps`
这个符号是全局变量符号，可以直接查看，能算出固定偏移。
`vtable`
这个符号不是之前介绍的几个符号，不是全局的，里面保存着`IO_file_jumps`的地址，只能通过前面几个全局符号来对应偏移搜索，没办法在`gdb`中直接查看。`stdin`之类的三个输入输出流如果想调用`_IO_file_jumps`里面的函数，只能通过`vtable`来调用，所以如果我们可以伪造一个`IO_jump_t`结构体，使得`vtable`指向这个伪造的结构体，根据偏移就可以调用里面伪造的函数指针。比如修改`stdin`结构中的`vtable`指向伪造的`IO_jump_t`结构体，并且将伪造的`IO_jump_t`结构体的`__overflow`劫持为`system`函数地址，那么使用`stdin`调用`__overflow`时就相当于调用`system`函数。

#### 四、`_IO_FILE_plus`中各个结构的成员含义：

1. `_IO_FILE`：这个结构体生成的指针就是我们实际上编程时用`fopen`返回值指针。
(1) `int _flags`: 给`vtable`中的各类函数指针传入的第一个参数。(之后会讲到)
(2)

```
#注释头



  char* _IO_read_ptr; /* Current read pointer */

  char* _IO_read_end; /* End of get area. */

  char* _IO_read_base; /* Start of putback+get area. */

  char* _IO_write_base; /* Start of put area. */

  char* _IO_write_ptr; /* Current put pointer. */

  char* _IO_write_end; /* End of put area. */

  char* _IO_buf_base; /* Start of reserve area. */

  char* _IO_buf_end;
```

这一系列就是对应的`stdin`、`stdout`、`stderr`中读取地址，写入地址，`buf`地址。可以通过修改`stdin`结构体中的`buf_base`和`buf_end`，改为某个`bss`段地址，来使得`scanf`之类的读取函数读到`bss`段上。其它用法类似，`ctfwiki`详解。
(3)

```
 #注释头 



  char *_IO_save_base; /* Pointer to start of non-current get area. */

  char *_IO_backup_base;  /* Pointer to first valid character of backup area */

  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;
```

这段就不知道干啥的了，应该需要具体调试才知道具体用法，网上也搜不到。
(4) `struct _IO_FILE *_chain`:
保存的是一个`_IO_FILE`指针，实际应该是`_IO_FILE_plus`指针，指向下一个`_IO_FILE_plus`结构体。
有一个全局变量`_IO_list_all`，里面保存`_IO_2_1stderr`的地址，且距离`_IO_2_1stderr`偏移不远：
![](https://bbs.kanxue.com/upload/attach/202108/904686_S3UVKGS9EXF7ERW.jpg)
这其实是一个链表，由`_IO_list_all`进行访问维护，形式如下：

```
#注释头



*_IO_list_all->_IO_2_1stderr_

_IO_2_1stderr_.chain->_IO_2_1stderr_

_IO_2_1stderr_.chain->_IO_2_1stdout_

_IO_2_1stdout_.chain->_IO_2_1stdin_

_IO_2_1stdin_.chain=0x0
```

程序会通过`_IO_list_all`来找到这三个输入输出流。
(5)

```
  #注释头  



  int _fileno;

  int _blksize;

  int _flags2;

  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

  unsigned short _cur_column;

  signed char _vtable_offset;

  char _shortbuf[1];

  _IO_lock_t *_lock;
```

这段也不知道干啥用的。
(6)

```
  #注释头



  _IO_off64_t _offset;

  struct _IO_codecvt *_codecvt;

  struct _IO_wide_data *_wide_data;

  struct _IO_FILE *_freeres_list;

  void *_freeres_buf;

  void *__pad1;

  void *__pad2;

  void *__pad3;

  void *__pad4;

  size_t __pad5;

  int _mode;

  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
```

这段在`_IO_FILE_complete`中定义的也不知道干啥用。

2. `IO_jump_t`结构，也就是`vtable`中的，里面都是一些函数指针，程序调用对应的输入输出流就会调用对应的`vtable`中的函数指针。这边挑一些常用的函数：
(1) `_IO_xsgetn_t`：实际是`__GI_IO_file_xsgetn`，会被`fread`函数调用。
(2) `_IO_xsputn_t`：实际是`__IO_new_file_xsputn`，会被`fwrite`函数调用，还有`printf`/`puts`等一些输出流函数。
(3) `fopen`和`fclose`会将三大输入输出流从`_IO_list_all`，`chain`域，链表脱链。
▲`fopen`情况下`_IO_FILE_plus`结构体会位于堆内存中

#### 五、FSOP的触发与伪造：File Stream Oriented Programming

1. 伪造：
劫持`_IO_list_all`的值，指向伪造的`_IO_FILE_`结构体`stderr`。
2. 触发：
借助`_IO_flush_all_lockp`触发，这个函数会刷新每个`_IO_FILE`结构体，同样调用里面`vtable`中的`_IO_overflow`。那么如果`_IO_overflow`为`system`，并将`flag`=`/bin/sh`，那么就可以`getshell`了。
▲而`_IO_flush_all_lockp`这个函数不需要手动触发，以下情况程序会自己触发：
(1) 当 `libc` 执行 `abort` 流程时：
(2) 当执行 `exit` 函数时：
`exit`→`__run_exit_handlers`→`_IO_cleanup`→`_IO_flush_all_lockp`
(3) 当执行流从 `main` 函数返回时
那么一旦我们伪造好了，`main`函数返回，或者`exit`，或者`abort`都会`getshell`。
3. 需要绕过的绕过检查：

```
#注释头



fake_IO_FILE_._mode <= 0

fake_IO_FILE_._IO_write_ptr > fake_IO_FILE._IO_write_base
```

#### 六、其它的劫持：

▲`libc`2.24及以上的`_IO_file_jumps`结构体不可修改，只能进行伪造，`libc`2.23及以下可以直接修改`_IO_file_jumps`的函数指针。
1. 修改`vatble`指针，伪造`_IO_file_jumps`结构体：(HCTF2018_the_end)
