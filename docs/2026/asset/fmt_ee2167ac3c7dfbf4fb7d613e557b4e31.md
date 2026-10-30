
```
7cf9407e ff5020          call    dword ptr [eax+20h]  ds:0023:7cf78b24={quartz!CTransformInputPin::CheckMediaType (7cf9de49)}
```

后来发现这办法太那个了，直接下写入断点就行了，就可以直达了，上面就换乘了

```
0:002> ba w1 003f8610  

0:002> g

Breakpoint 1 hit

eax=000000c3 ebx=003f82b0 ecx=00000004 edx=000000f7 esi=0017b8f8 edi=003f8614

eip=7d0706d0 esp=024ff650 ebp=024ff658 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202

quartz!ParseSequenceHeader+0x114:

7d0706d0 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

我们看看此时的源地址和目的地址，目的堆块的size已经被覆盖成4141了

```
0:002> !heap -p -a esi

    address 0017b8f8 found in

    _HEAP @ 150000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        0017b7e0 0027 0000  [05]   0017b7e8    00123 - (busy)

0:002> !heap -p -a edi

    address 003f8614 found in

    _HEAP @ 3f0000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        003f8608 4141 0000  [41]   003f8610    20a07 - (busy)
```

我们看看`edi`的前一个堆的位置（随便找个小于003f8608 ，被4141覆盖的就行了），可以看到本来是003f82a8堆块的数据，最后确实是覆盖到了下一个堆003f8608了，htc也没有检查出来

```
0:002> !heap -p -a 003f8570 

    address 003f8570 found in

    _HEAP @ 3f0000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        003f82a8 006c 0000  [05]   003f82b0    00350 - (busy)

          quartz!CMpegVideoCodec::`vftable'

0:002> dd 003f8570 l50

003f8570  41414141 41414141 41414141 41414141

003f8580  41414141 41414141 41414141 41414141

003f8590  41414141 41414141 41414141 41414141

003f85a0  41414141 41414141 41414141 41414141

003f85b0  41414141 41414141 41414141 41414141

003f85c0  41414141 41414141 41414141 41414141

003f85d0  41414141 41414141 41414141 41414141

003f85e0  41414141 41414141 41414141 41414141

003f85f0  41414141 41414141 41414141 41414141

003f8600  41414141 41414141 41414141 41414141

003f8610  41414141 003f8610 00000000 7cf73ed0

003f8620  7cf730a0 003f8700 00000000 00000000

003f8630  003f830c 00000000 003f82b0 00000000

003f8640  00000001 00000000 00000000 00000000

003f8650  00000000 00000000 00000000 00000000

003f8660  00000000 00000001 00000000 00000001

003f8670  00000000 00000000 00000000 00000000

003f8680  00000000 00000000 00000000 00000000

003f8690  00000000 00000000 ffffffff 7fffffff

003f86a0  00000000 3ff00000 7cf74678 00000000
```

## 最后说说

之后回到家用自己的电脑又可以了，无语~（但htc还是不行），可能一个细微的差别都会导致调试结果不尽相同，软件版本，系统，还有虚拟机毕竟是虚拟出来的。

```
(7c0.1a4): Access violation - code c0000005 (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=000000c3 ebx=027c8cb0 ecx=00000009 edx=000000f7 esi=14f02fd4 edi=027c9000

eip=7d0706d0 esp=0d90f650 ebp=0d90f658 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202

quartz!ParseSequenceHeader+0x114:

7d0706d0 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]

0:001> !gflag

Current NtGlobalFlag contents: 0x02000000

    hpa - Place heap allocations at ends of pages
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
