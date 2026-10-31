
# [原创]MS17-010 SMB 远程命令执行漏洞利用分析

第一次尝试分析漏洞，水平有限有不对的地方请大家指正，轻拍啊。呵呵。`MS17-010`漏洞出现在的原因是`SrvOs2FeaListSizeToNt`在计算需要分配的内存长度时存在问题，以至于可以溢出到邻近的内存块。

```c
int __stdcall SrvOs2FeaListSizeToNt(_DWORD *a1)
{
  _WORD *v1; // eax@1
  unsigned int v2; // edi@1
  unsigned int v3; // esi@1
  int v4; // ebx@3
  int v6; // [sp+Ch] [bp-4h]@1
  v1 = a1;
  v6 = 0;
  v2 = (unsigned int)a1 + *a1;
  v3 = (unsigned int)(a1 + 1);
  if ( (unsigned int)(a1 + 1) < v2 )
  {
    while ( v3 + 4 < v2 )
    {
      v4 = *(_WORD *)(v3 + 2) + *(_BYTE *)(v3 + 1);
      if ( v4 + v3 + 4 + 1 > v2 )
        break;
      if ( RtlSizeTAdd(v6, (v4 + 12) & 0xFFFFFFFC, &v6) < 0 )
        return 0;
      v3 += v4 + 5;
      if ( v3 >= v2 )
        return v6;
      v1 = a1;
    }
   *v1 = v3 - (_WORD)v1;
  }
  return v6;
}
```

按上面的逻辑如果`v3`这个缓存区中全是`0`，那么每次循环`v3`向前移动`5`个字节，`v6`就会加`0xC`（也就是`12`字节）。问题就出现在`*v1 = v3 - (_WORD)v1;`上`a1`的长度是一个四字节，但这里却是个双字节。那么如果长度为三字节以上，且执行到这里时就有可能把的长度改大。`Eternalblue`就设长度是`0x10000`，最后最后一个分片长度是`0xA8`,这时`v3`已经是偏移了`FF5D`了。加上`0xa8`就超`0x10000`了，所以就会走到了这一句，执行后长度变成了`0x1FF5D`比原来大了不少。

```c
signed int __stdcall SrvOs2FeaListToNt(char *a1, _DWORD *a2, int *a3, _WORD *a4)
{
  char *v4; // edi@1
  int v5; // eax@1
  _DWORD *v7; // eax@3
  char *v8; // ebx@9
  char *v9; // esi@9
  signed int v10; // esi@14
  __int16 v11; // [sp+8h] [bp-4h]@1
  _DWORD *v12; // [sp+14h] [bp+8h]@11
  v11 = 0;
  v4 = a1;
  v5 = SrvOs2FeaListSizeToNt(a1);
  *a3 = v5;
  if ( !v5 )
  {
    *a4 = 0;
    return -1063718657;
  }
  v7 = (_DWORD *)SrvAllocateNonPagedPool(v5, 21);
  *a2 = v7;
  if ( v7 )
  {
    v8 = &a1[*(_DWORD *)a1 - 5];
    v9 = a1 + 4;
    if ( a1 + 4 > v8 )
    {
LABEL_13:
      if ( v9 == &v4[*(_DWORD *)v4] )
      {
        *v7 = 0;
        return 0;
      }
      *a4 = v11 - (_WORD)v4;
      v10 = -1073741823;
    }
    else
    {
      while ( !(*v9 & 0x7F) )
      {
        v12 = v7;
        v11 = (signed __int16)v9;
        v7 = (_DWORD *)SrvOs2FeaToNt((int)v7, (int)v9);
        v9 += (unsigned __int8)v9[1] + *((_WORD *)v9 + 1) + 5;
        if ( v9 > v8 )
        {
          v7 = v12;
          goto LABEL_13;
        }
      }
      *a4 = (_WORD)v9 - (_WORD)v4;
      v10 = -1073741811;
    }
    SrvFreeNonPagedPool(*a2);
    return v10;
  }
  if ( *((_BYTE *)WPP_GLOBAL_Control + 29) >= 2u && *((_BYTE *)WPP_GLOBAL_Control + 32) & 1 && KeGetCurrentIrql() < 2u )
  {
    DbgPrint("SrvOs2FeaListToNt: Unable to allocate %d bytes from nonpaged pool.", *a3, 0);
    DbgPrint("\n");
  }
  return -1073741307;
}
```

上面的循环中：

`v9 += (unsigned __int8)v9[1] + *((_WORD *)v9 + 1) + 5;`
`if ( v9 > v8 )`

这里虽有判断，但长度`v8`已经被上面`SrvOs2FeaListSizeToNt(a1);`改的很大了，当`v7`已经到末尾时`v9`还是小于`v8`，再执行`SrvOs2FeaToNt`就分溢出了。溢出后从`while ( !(*v9 & 0x7F) )`这句退出从而不影响其它内存。

```c
unsigned int __stdcall SrvOs2FeaToNt(int a1, int a2)
{
  int v2; // esi@1
  _BYTE *v3; // ebx@1
  unsigned int result; // eax@1
  v2 = a1;
  *(_BYTE *)(a1 + 4) = *(_BYTE *)a2;
  *(_BYTE *)(a1 + 5) = *(_BYTE *)(a2 + 1);
  *(_WORD *)(a1 + 6) = *(_WORD *)(a2 + 2);
  _memmove((void *)(a1 + 8), (const void *)(a2 + 4), *(_BYTE *)(a2 + 1));
  v3 = (_BYTE *)(*(_BYTE *)(a1 + 5) + a1 + 8);
  *v3++ = 0;
  _memmove(v3, (const void *)(*(_BYTE *)(v2 + 5) + a2 + 5), *(_WORD *)(v2 + 6));
  result = (unsigned int)&v3[*(_WORD *)(a1 + 6) + 3] & 0xFFFFFFFC;
  *(_DWORD *)v2 = ((unsigned int)&v3[*(_WORD *)(v2 + 6) + 3] & 0xFFFFFFFC) - v2;
  return result;
}
```

`Eternalblue`第一步先建立一系列链接并发送长度为`fff7`的`SMB`包，但每个连接第一次只发前`0x80`字节，然后等带完成第二步再发。为研究这个漏洞是怎么利用的，用下面的方法设置了一个断点，发现每次在`srvnet!SrvNetWskReceiveEvent`调用`afd!WskProAPIReceive`时分配的`MDL`的地址都间隔了`0x11000`字节。

`bp afd!WskProAPIReceive  ".echo ##############;dd esp l8; .echo ********************;dd poi(esp+8) l20;.echo ********************;dt poi(poi(esp+8)) _MDL;  gc;"`

而`srvnet`在接收这些包时，分配置了接收环境块及接收的内存等相关内容。其中接收环境块偏移`0x18`开的地方存了就存了调用`afd!WskProAPIReceive`时的`WSK_BUF`。

```c
typedef struct _WSK_BUF {
   PMDL  Mdl;
   ULONG  Offset;
   SIZE_T  Length;
 } WSK_BUF, *PWSK_BUF;
```

而`WSK_BUF`中的`Mdl`指向的正是环境块偏移`0x2c`的地址。如下面`86811010`是接收环境块的开始`86811028`是调用`afd!WskProAPIReceive`时的`WSK_BUF`，`8681103c`就是接收数据所用内存的`MDL`描述。

```
3: kd> dd 86811000 l100
86811000  00011000 00000000 0044006d 00740061
86811010  005c0008 0069004d 00720063 86811160
86811020  00010ea0 00000000 8681103c 00000000
86811030  0000fff7 8825f568 868110a4 00000000
86811040  10040060 00000000 86811160 86811000
86811050  00010ea0 00000160 0007f811 0007f812
86811060  0007f813 0007f814 0007f815 0007f816
86811070  0007f817 0007f818 0007f819 0007f81a
86811080  0007f81b 0007f81c 0007f81d 0007f81e
86811090  0007f81f 0007f820 0007f821 00650078
868110a0  005c0072 00000000 00000064 0065006c
868110b0  005c0073 00000000 00010ea0 00000fff
868110c0  00340030 0063002e 00000069 00690064
868110d0  c0100002 04240116 0880e96f 00000001
868110e0  977dd5f0 0044005c 00760065 00630069
868110f0  005c0065 00610048 00640072 00690064
86811100  006b0073 006f0056 0075006c 0065006d
86811110  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811120  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811130  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811140  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811150  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
```

因为前面的步骤发送了一系列的包，让`srvnet`驱动分配了一系列的内存块。内存块的分配规律就如同前面说描述的那样。这时`Eternalblue`再发送一个超大的`SMB`包, `srvnet`驱动接收后会走到 `SrvOs2FeaListToNt` 函数处理。就像一开始分析的那样这个函数里触发那个字段操作错误，把相邻的内存块覆盖掉。而被覆盖的这块内存恰恰正好是`srvnet`驱动之前接收`SMB`包的接收环境块，并且`MDL`早已经投递给网路驱动接收数据了。

```
3: kd> dd 86811000 l100
86811000  00000000 00000000 0000ffff 00000000
86811010  0000ffff 00000000 00000000 00000000
86811020  00000000 00000000 ffdff100 00000000
86811030  00000000 ffdff020 ffdff100 ffffffff
86811040  10040060 00000000 ffdfef80 00000000
86811050  ffd00010 ffffffff ffd00118 ffffffff
86811060  00000000 00000000 00000000 00000000
86811070  10040060 00000000 00000000 00000000
86811080  ffcfff90 ffffffff 00000000 00000000
86811090  00001080 00000000 00000000 00000000
868110a0  005c0054 00000000 00000064 0065006c
868110b0  005c0073 00000000 00010ea0 00000fff
868110c0  00340030 0063002e 00000069 00690064
868110d0  c0100002 04240116 0880e96f 00000001
868110e0  977dd5f0 0044005c 00760065 00630069
868110f0  005c0065 00610048 00640072 00690064
86811100  006b0073 006f0056 0075006c 0065006d
86811110  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811120  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811130  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811140  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
86811150  4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
```

再看`MDL`描述的内存地址变成了什么。

```
0: kd> dt 8681103C _MDL
ntdll!_MDL
   +0x000 Next             : 0xffffffff _MDL
   +0x004 Size             : 0n96
   +0x006 MdlFlags         : 0n4100
   +0x008 Process          : (null)
   +0x00c MappedSystemVa   : 0xffdfef80 Void
   +0x010 StartVa          : (null)
   +0x014 ByteCount        : 0xffd00010
   +0x018 ByteOffset       : 0xffffffff
```

`MDL`描述的内存地址变成了`0xffdfef80`因为链接之前已经接收了先发送的`0x80`个字节,等待接收下面的数据呢，要再接收的话会把数据`COPY`到`0xffdfef80+0x80`的地方，即`0xffdff000`位置。而这个位置是可写可执行的。

```
2: kd> !pte 0xffdff000
                    VA ffdff000
PDE at C0603FF0            PTE at C07FEFF8
contains 000000000018A063  contains 00000000001E2163
pfn 18a       ---DA--KWEV  pfn 1e2       -G-DA--KWEV
```

下面只要发送`SHELLCODE`就会被写到这个位置。

当接收完成后会调用`srvnet!SrvNetWskReceiveComplete`即接收`IRP`的完成函数，同时会把接收环境块传给它。而接收环境块在偏移`0x24`的位置存储着一个地址，它指向了一块内存，这块内存偏移`0x16c`存了一个地址，这个地址是一个函数指针数组，里面分别存的是`SrvConnectHandler`，`SrvReceiveHandler`，`SrvDisconnectHandler`，`SrvCredentialHandler`。很明显在覆盖后接受环境块，偏移`0x24`的位置变成了`ffdff020`，指向了存放`SHELLCODE`的内存区，完全可被攻击者控制。从而能很轻松就转到攻击代码部份执行。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [MS17-010 SMB 远程命令执行漏洞利用分析.docx](javascript:void(0);)

  （22.27kb，354次下载）
