
# [原创]C/C++技巧 变长数据

比如在进行网络收发数据时，数据的长度不可变的，一般我们是这样定义数据的。

```
typedef struct _PROTOCOL_STRUCT
{
int nLen;
unsinged char pBuffer[1]; //gun linux可定义为0的数组
} PROTOCOL_STRUCT,*PPROTOCOL_STRUCT;
int nTotal = GetOffset(PROTOCOL_STRUCT,pBuffer[0]) + sizeof(UCHAR)*n;
```

这里计算数据的长度使用了 C/C++技巧 结构体成员偏移 的内容。

由于windows使用了标准的c/c++,故无法声名0个数的数组，gun linux可以的。

使用这种方法，可以获取每一个实际数据字节。

例如：

```
PROTOCOL_STRUCT pProtocolData = NULL;
....
for(int i=0;i<pProtocolData->nLen;i++)
{
unsigned char data = pProtocolData->pBuffer[i];
//do something
}
```

这里使用了数组与指针转换的技巧。

其实我们在c/c++定义的数组，结构体，类等各种我们认为有意义的数据类型，其实只是一种连续内存的数据组织方式，而编译器自动会进行偏移计算获取其数据。

所以说，这种有意义的结构体只是对我们编程者来说有界限的，但对编译器来说，其都只是数据内存，只是偏移不同而已。
