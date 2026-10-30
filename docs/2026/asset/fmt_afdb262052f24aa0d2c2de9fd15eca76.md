
```
unsigned __int64 free_p()

{

  unsigned int _index; // [rsp+4h] [rbp-Ch]

  unsigned __int64 v2; // [rsp+8h] [rbp-8h]



  v2 = __readfsqword(0x28u);

  printf("index \n> ");

  _index = read_num();

  if ( _index > 9 || !*(_QWORD *)(16LL * _index + malloc_array) )

    exit_p();

  memset(*(void **)(16LL * _index + malloc_array), 0, *(unsigned int *)(16LL * _index + malloc_array + 8));

  free(*(void **)(16LL * _index + malloc_array));

  *(_DWORD *)(16LL * _index + malloc_array + 8) = 0;

  *(_QWORD *)(16LL * _index + malloc_array) = 0LL;

  return __readfsqword(0x28u) ^ v2;

}





unsigned __int64 put_p()

{

  unsigned int _index; // [rsp+4h] [rbp-Ch]

  unsigned __int64 v2; // [rsp+8h] [rbp-8h]



  v2 = __readfsqword(0x28u);

  printf("index \n> ");

  _index = read_num();

  if ( _index > 9 || !*(_QWORD *)(16LL * _index + malloc_array) )

    exit_p();

  puts(*(const char **)(16LL * _index + malloc_array));

  return __readfsqword(0x28u) ^ v2;



    int _sizea; // [rsp+0h] [rbp-20h]

  unsigned int size; // [rsp+4h] [rbp-1Ch]

  unsigned __int64 v5; // [rsp+8h] [rbp-18h]



  v5 = __readfsqword(0x28u);

  LODWORD(_size) = 0;

  while ( (signed int)_size <= 9 && *(_QWORD *)(16LL * (signed int)_size + malloc_array) )

    LODWORD(_size) = _size + 1;

  if ( (_DWORD)_size == 10 )                    // limited max size to 10

  {

    puts("full!");

  }

  else

  {

    m_ptr = malloc_array;

    *(_QWORD *)(m_ptr + 16LL * (signed int)_size) = malloc(0xF8uLL);

    if ( !*(_QWORD *)(16LL * (signed int)_size + malloc_array) )

    {

      puts("malloc error!");

      exit_p();

    }

    printf("size \n> ", _size);

    size = read_num();

    if ( size > 0xF8 )

      exit_p();

    *(_DWORD *)(16LL * _sizea + malloc_array + 8) = size;

    printf("content \n> ");

    read_content(*(_BYTE **)(16LL * _sizea + malloc_array), *(_DWORD *)(16LL * _sizea + malloc_array + 8));

  }

  return __readfsqword(0x28u) ^ v5;

}

}





unsigned __int64 malloc_p()

{

  __int64 m_ptr; // rbx

  __int64 _size; // [rsp+0h] [rbp-20h]

  int _sizea; // [rsp+0h] [rbp-20h]

  unsigned int size; // [rsp+4h] [rbp-1Ch]

  unsigned __int64 v5; // [rsp+8h] [rbp-18h]



  v5 = __readfsqword(0x28u);

  LODWORD(_size) = 0;

  while ( (signed int)_size <= 9 && *(_QWORD *)(16LL * (signed int)_size + malloc_array) )

    LODWORD(_size) = _size + 1;

  if ( (_DWORD)_size == 10 )                    // limited max size to 10

  {

    puts("full!");

  }

  else

  {

    m_ptr = malloc_array;

    *(_QWORD *)(m_ptr + 16LL * (signed int)_size) = malloc(0xF8uLL);

    if ( !*(_QWORD *)(16LL * (signed int)_size + malloc_array) )

    {

      puts("malloc error!");

      exit_p();

    }

    printf("size \n> ", _size);

    size = read_num();

    if ( size > 0xF8 )

      exit_p();

    *(_DWORD *)(16LL * _sizea + malloc_array + 8) = size;

    printf("content \n> ");

    read_content(*(_BYTE **)(16LL * _sizea + malloc_array), *(_DWORD *)(16LL * _sizea + malloc_array + 8));

  }

  return __readfsqword(0x28u) ^ v5;

}





unsigned __int64 __fastcall read_content(_BYTE *content, int size)

{

  unsigned int index; // [rsp+14h] [rbp-Ch]

  unsigned __int64 v4; // [rsp+18h] [rbp-8h]



  v4 = __readfsqword(0x28u);

  index = 0;

  if ( size )

  {

    while ( 1 )

    {

      read(0, &content[index], 1uLL);

      if ( size - 1 < index || !content[index] || content[index] == 0xA )

        break;

      ++index;

    }

    content[index] = 0;

    content[size] = 0;                          // off by one

  }

  else

  {

    *content = 0;

  }

  return __readfsqword(0x28u) ^ v4;

}





void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)

{

  int _opt; // eax



  setbuf_print_slogan();

  malloc_array = (__int64)calloc(0xA0uLL, 1uLL);

  if ( !malloc_array )

  {

    puts("init error!");

    exit_p();

  }

  while ( 1 )

  {

    while ( 1 )

    {

      sub_B38();

      _opt = read_num();

      if ( _opt != 2 )

        break;

      free_p();

    }

    if ( _opt > 2 )

    {

      if ( _opt == 3 )

      {

        put_p();

      }

      else if ( _opt == 4 )

      {

        exit_p();

      }

    }

    else if ( _opt == 1 )

    {

      malloc_p();

    }

  }

}

```

有必要指出来的是， `malloc_array` 存储了各个chunk的地址以及对应存储内容的大小。

`read_content`函数有一个很明显的 *off by one* 。当我们创建输入size 248（也就是"0xf8"）时， 相邻chunk的`prev_inuse`会被抹除掉。这样，我们就可以创建overlapping chunk。 再然后，我们用 `TCache` 双重释放 overlapping chunks，最后控制了新创建的chunk地址就可以任意地址写了。

但是呢，这里所有的chunk大小都为 `0x100`，也就是说我们必须写入0x100的整倍数以创建overlapping chunk（比如0x200）。 '\x00'会被 `read_content`检测, 导致我们不能直接输入 `0x200` 到 `prev_size`。

我想了想，最后发现我们可以用 unsorted bin 改写 `prev_size`. 当unsorted合并的时候, 他会创建一个`prev_size`于块的末尾. 如果我们合并chunk A, B, C (大小都是 0x100)，会发生如下图的事:

```
合并 Chunk A:

+---------------+

|  size: 0x100  | -> unosrted bin大小

+---------------+

|     fd/bk     |

+---------------+

|               |

|               |

|               |

+---------------+

|prev size:0x100| -> 写入的prev_size

+---------------+



合并Chunk B到unsorted bins:

+---------------+

|  size: 0x200  | -> 被合并过的unsorted bin大小

+---------------+

|     fd/bk     |

+---------------+

|               |

|               |

|               |

+---------------+

|prev size:0x100| -> 原来的prev_size会保持在这里

+---------------+

|  size: 0x100  | 

+---------------+

|               |

|               |

|               |

|               |

+---------------+

|prev size:0x200| -> 新的prev_size

+---------------+



unsorted bin现在的大小是0x200, 然后添加 Chunk C

+---------------+

|  size: 0x300  | -> unosrted bin大小

+---------------+

|     fd/bk     |

+---------------+

|               |

|               |

|               |

+---------------+

|prev size:0x100|

+---------------+

|  size: 0x100  |

+---------------+

|               |

|               |

|               |

|               |

+---------------+

|prev size:0x200| -> 0x200，刚刚好可以用来overlap!

+---------------+

|  size: 0x100  | 

+---------------+

|               |

|               |

|               |

|               |

+---------------+

|prev size:0x300| -> 最终的size

+---------------+

```

当Unsorted Bin被malloc之后, 它只会改变最后的那个 `prev_size` (0x300那个）, 所以中间的 (0x200) 保持不变。

最后就是常规的TCache利用了。 由于开启了"Full RELRO"，我们只能改写 `free_hook`成`one_gadget`来RCE. btw，我们需要填充7个chunk到`TCache`，不然程序不会使用`unsorted bin`。用TCache double free的时候也是一个道理，先free掉unsorted bin里的chunk，我们才能用TCache。

攻击脚本：
