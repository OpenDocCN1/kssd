
# [原创]浅谈系列之Object hook

![](https://bbs.kanxue.com/view/img/face/50.gif)昨天脑袋进水，睡了一晚感觉好多了。

![](https://bbs.kanxue.com/view/img/face/11.gif)怪不得昨天数据一直错，错就错在，我把 `_OBJECT_TYPE` 又算了一遍。

```
      _asm
    {
        push eax
        mov eax, pObject
        mov eax, DWORD ptr DS:[eax-10h]
        mov PP, eax
        pop eax
    }
```

关于这个汇编得到 `_object_type` 地址：

解释如下：

①. 进程的话就是 `EPROCESS-0x18=object_header` 的地址

而 `_Object_Type` 位于偏移处 `+0x08` 处，看下面代码：

```
kd> dt _object_header

nt!_OBJECT_HEADER

   +0x000 PointerCount     : Int4B

   +0x004 HandleCount      : Int4B

   +0x004 NextToFree       : Ptr32 Void

   +0x008 Type             : Ptr32 _OBJECT_TYPE <==看这个：

   +0x00c NameInfoOffset   : UChar

   +0x00d HandleInfoOffset : UChar

   +0x00e QuotaInfoOffset  : UChar

   +0x00f Flags            : UChar

   +0x010 ObjectCreateInfo : Ptr32 _OBJECT_CREATE_INFORMATION

   +0x010 QuotaBlockCharged : Ptr32 Void

   +0x014 SecurityDescriptor : Ptr32 Void

   +0x018 Body             : _QUAD
```

↑于是诞生上面那句汇编~![](https://bbs.kanxue.com/view/img/face/27.gif)

我们拿一个进程做实验~

![](https://bbs.kanxue.com/upload/attach/201308/439417_jp0e733yiwtyfvn.jpg)

输入 `dt _object_header`

得到下面内容：

```
kd> dt _object_header 81b0b7f0-18h

nt!_OBJECT_HEADER

   +0x000 PointerCount     : 0n32

   +0x004 HandleCount      : 0n1

   +0x004 NextToFree       : 0x00000001 Void

   +0x008 Type             : 0x81bbde70 _OBJECT_TYPE  （重点看这个②）

   +0x00c NameInfoOffset   : 0 ''

   +0x00d HandleInfoOffset : 0 ''

   +0x00e QuotaInfoOffset  : 0 ''

   +0x00f Flags            : 0x20 ' '

   +0x010 ObjectCreateInfo : 0x8183dc70 _OBJECT_CREATE_INFORMATION

   +0x010 QuotaBlockCharged : 0x8183dc70 Void

   +0x014 SecurityDescriptor : 0xe239f8df Void

   +0x018 Body             : _QUAD
```

现在我们大胆在 windbg 上输入：`dt _OBJECT_TYPE 0x81bbde70`

得到下面这个内容：

```
kd> dt _OBJECT_TYPE 0x81bbde70

nt!_OBJECT_TYPE

   +0x000 Mutex            : _ERESOURCE

   +0x038 TypeList         : _LIST_ENTRY [ 0x81bbdea8 - 0x81bbdea8 ]

   +0x040 Name             : _UNICODE_STRING "Process"

   +0x048 DefaultObject    : (null) 

   +0x04c Index            : 5

   +0x050 TotalNumberOfObjects : 0x15

   +0x054 TotalNumberOfHandles : 0x5a

   +0x058 HighWaterNumberOfObjects : 0x15

   +0x05c HighWaterNumberOfHandles : 0x5c

   +0x060 TypeInfo         : _OBJECT_TYPE_INITIALIZER ④看这个！

   +0x0ac Key              : 0x636f7250

   +0x0b0 ObjectLocks      : [4] _ERESOURCE
```

接着我们访问 `_object_type+60h=_obect_type_initalizer`

请再次输入 `dt _OBJECT_TYPE_INITIALIZER 0x81bbde70 +60h`

![](https://bbs.kanxue.com/view/img/face/50.gif)接着见证我们的成果：

![](https://bbs.kanxue.com/upload/attach/201308/439417_huc4bfsjnnkydut.jpg)

![](https://bbs.kanxue.com/upload/attach/201308/439417_il1nzn93nut4r79.jpg)

(～ o ～)~zZ 完成~

![](https://bbs.kanxue.com/upload/attach/201308/439417_lslc64x5k8hktrs.jpg)

最后附上我们的源代码~

![](https://bbs.kanxue.com/view/img/face/1.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [2.jpg](javascript:void(0);) （98.34kb，94次下载）
- [3.jpg](javascript:void(0);) （103.11kb，65次下载）
- [OBJECT.rar](javascript:void(0);) （6.24kb，134次下载）
- [1.jpg](javascript:void(0);) （175.31kb，11次下载）
- [4.jpg](javascript:void(0);) （67.29kb，49次下载）
