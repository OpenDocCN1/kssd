

# 剑网 3Pak 文件格式分析

> 原文：[`www.pediy.com/kssd/pediy11/119731.html`](https://www.pediy.com/kssd/pediy11/119731.html)

一、首先看 package.ini 文件  
\[SO3Client\]  
11=data\_6.pak  
10=data\_5.pak  
1=ui.pak  
0=update\_1.pak  
3=maps.pak  
2=settings.pak  
5=scripts.pak  
4=represent.pak  
7=data\_2.pak  
6=data\_1.pak  
9=data\_4.pak  
Path=.\\pak  
8=data\_3.pak  
这里显示了所有 Pak 文件，索引数越小，表示等级越高，如果某个文件在 update\_1.pak 和 ui.pak 中都存在，则以 update\_1.pak 中的为准。  
二、分析每个 Pak 文件  
Pak 文件的前 16 个字节格式如下：  
\[pak 标记(Uint32)(PACK)\] + \[文件数目(Uint32)\]+\[索引数据在文件中的偏移位置(Uint32)\]+0x20000000。  
由此可以读出索引数据（在每个文件的最后面）。  
每条索引数据 16 个字节，格式为\[哈希数值(Uint32)\] + \[数据在文件中的偏移位置(Uint32)\]+\[解压后文件长度(Uint32)\] + \[文件在 pak 中的长度(Uint24)\]+\[压缩标志(1Byte)\]。  
其中，压缩标志有三种：0x20 压缩，0x00 未压缩，0x10 未压缩（是某种未知格式，不需解压）。  
三、解压注意事项  
每个文件都有唯一对应的 Hash，计算的函数在 Engine\_Lua5.dll 中的 g\_FileNameHash，注意传递的文件路径必须以“\\”开头，DWORD g\_FileNameHash(char\*)。  
解压函数也在 Engine\_Lua5.dll 中，位置是 DLL 基址+0x18020，int 函数名(char\* pSrcData, int nSrcLen, char\* pDstData, int\* pDstLen)。  
至于文件名，可以用 FileMon 得到一部分，再通过分析得到的文件内容得到更多。