
# [原创]PE解析逆向LoadString

通过对 `LoadString` 的深入理解，也可加强对 PE 中资源表的理解。

熟悉 Windows 编程的朋友相信对资源都不陌生，在使用 PE 资源的时候，字符串资源也是大家经常使用的一种。下面就开始了解它的来龙去脉。

我们知道，`LoadString` 的函数原型如下：

```c
int LoadString(
  HINSTANCE hInstance,  // handle to resource module
  UINT uID,             // resource identifier
  LPTSTR lpBuffer,      // resource buffer
  int nBufferMax        // size of buffer
);
```

最后两个参数自不必说，第一个参数相信熟悉 PE 的朋友都能猜到它的意义，作为一个输入参数，它是必须的，我们需要模块基址从而定位到资源表。那么，问题来了，假如一个 PE 文件有多个字符串资源，那么怎么准确定位呢？一个 `uID` 可以吗？

以我手上一个 DEMO 程序为例，

![](https://bbs.kanxue.com/upload/attach/201508/690458_0s58zc6lgwzqm89.jpg)

定义的资源如下：

![](https://bbs.kanxue.com/upload/attach/201508/690458_o2t5ihvsbbqr3il.jpg)

两个字符串，`ID` 分别是 101 和 1000。

## 手工定位 PE 资源

那么，在我们逆向之前，不妨先手工在 PE 文件找到对应的资源，顺便复习下 PE 中的资源表。

首先，通过查询数据目录项中的第二项（从 0 开始），我们获取到资源表的 RVA（相对基址偏移），转成 FOA（相对文件偏移）后，可以在文件中定位到它。

以我的程序为例，

![](https://bbs.kanxue.com/upload/attach/201508/690458_k7a4kcbtfelpeu6.jpg)

`RVA = 0x1A000`

转成 FOA 后，在文件偏移 `7000h` 处

![](https://bbs.kanxue.com/upload/attach/201508/690458_r394rzv1oe9zidy.jpg)

我们知道，资源表主要由三层目录以及相应的详细数据构成，下图有详细说明：

![](https://bbs.kanxue.com/upload/attach/201508/690458_v9gy5po6e5ra1qv.jpg)

### 资源目录结构

目录项如下：

```c
typedef struct _IMAGE_RESOURCE_DIRECTORY {
    DWORD   Characteristics;
    DWORD   TimeDateStamp;
    WORD    MajorVersion;
    WORD    MinorVersion;
    WORD    NumberOfNamedEntries;
    WORD    NumberOfIdEntries;
//  IMAGE_RESOURCE_DIRECTORY_ENTRY DirectoryEntries[];
} IMAGE_RESOURCE_DIRECTORY, *PIMAGE_RESOURCE_DIRECTORY;
```

以及紧随其后的

```c
typedef struct _IMAGE_RESOURCE_DIRECTORY_ENTRY {
    union {
        struct {
            DWORD NameOffset:31;
            DWORD NameIsString:1;
        } DUMMYSTRUCTNAME;
        DWORD   Name;
        WORD    Id;
    } DUMMYUNIONNAME;
    union {
        DWORD   OffsetToData;  // 这里的 OFFSET 是相对资源节的偏移
        struct {
            DWORD   OffsetToDirectory:31;
            DWORD   DataIsDirectory:1;
        } DUMMYSTRUCTNAME2;
    } DUMMYUNIONNAME2;
} IMAGE_RESOURCE_DIRECTORY_ENTRY, *PIMAGE_RESOURCE_DIRECTORY_ENTRY;
```

先看第一层：

```text
ID             OffsetToData
6            80000020
18            80000040
```

字符串类型的 `ID` 编号为 `6`，取 `offset` 的低位，定位到节偏移 `0x20` 处，我们来到第二层。

![](https://bbs.kanxue.com/upload/attach/201508/690458_2il9j5dl67krqqw.jpg)

上图第一行末尾的 `02` 说明本层有两个字符串：

```text
ID            OffsetToData
07            80 00 00 58
3f            80 00 00 70
```

两个字符串编号分别为 `07` 和 `3F`，这个信息很重要，下面逆向要用到。

最终通过第三层目录定位到实际的字符串在文件偏移的地址后，可以看到如下：

![](https://bbs.kanxue.com/upload/attach/201508/690458_65wqnvvslzwwzaz.jpg)

也就是说，我们手工定位只能获取所有的字符串，但是具体哪一个，还需要研究 `LoadString` 的第二个参数意义。

## 逆向 LoadString

下面开始进入逆向。

![](https://bbs.kanxue.com/upload/attach/201508/690458_xxl29tegdmhjxlz.jpg)

我们只关心 `StringID` 这个参数，其余不做参考。

向下走，

![](https://bbs.kanxue.com/upload/attach/201508/690458_kzv4q6ojze6i6jf.jpg)

又会调用 `LoadStringBaseExW` 这个函数，继续跟。

![](https://bbs.kanxue.com/upload/attach/201508/690458_cxzen5gtbahu55e.jpg)

又将参数压栈后调用 `RtlLoadString` 这个函数，继续跟入。

![](https://bbs.kanxue.com/upload/attach/201508/690458_j63t4m9jy9kg0re.jpg)

在 `RtlLoadString` 里，代码将字符串 `ID` 右移 4 位后加一，写入到一个局部变量中，这个局部变量是我们需要关注的重点。

继续向下：

![](https://bbs.kanxue.com/upload/attach/201508/690458_tltap10dp97dohc.jpg)

刚才写入的位置是 `ebp - 38`，而 `ebp - 3c` 是字符串资源的类型 `ID`，也就是 `6`。

`ebp - 3c` 和 `ebp - 38` 在内存上连续，当做结构体传入第二个参数，进入内部 call。

![](https://bbs.kanxue.com/upload/attach/201508/690458_bhbhbdejn1j3xnp.jpg)

在这个内部 call 里，代码首先调用 `RtlImageDirectoryEntryToData` 获得资源表的 `VA`。

然后进入三次循环，因为资源表的目录就是三层，所以我们还是只关注在第二层时，代码如何将上次 `stringID` 运算的结果与之匹配。

![](https://bbs.kanxue.com/upload/attach/201508/690458_rsz86y4zi4pp93k.jpg)

代码到此，此时 `ESI = 0x0018FC98`，也就是指向 `stringID` 右移运算的结果，作为第四个参数传入，我们跟进去这个 call。

![](https://bbs.kanxue.com/upload/attach/201508/690458_eauxtlnqypqmdz3.jpg)

进入之后，又将结果作为参数，传入到下一个 call，还是一样，继续跟。

![](https://bbs.kanxue.com/upload/attach/201508/690458_wn2jpxfdus2vf8t.jpg)

到此，思路就很清晰了：函数枚举每个 `IMAGE_RESOURCE_DIRECTORY_ENTRY` 项，然后取每一项的 `ID` 值和传入的 `ID` 值做比较，相等就是找到了目标。

到了这里，离成功已经比较接近了，我们还是回到上层 `RtlLoadString` 中。

![](https://bbs.kanxue.com/upload/attach/201508/690458_jlx81ha9vqvnd0a.jpg)

此时 `edi` 保存字符串块的首地址（`0041A238`），在 PE 中可以定位到如下：

![](https://bbs.kanxue.com/upload/attach/201508/690458_gt4u1gvhdm9r2sx.jpg)

在 `0x7238` 处，似乎离目标 `"ByeBye"` 还有一段距离。

在 `RtlLoadString` 中继续向下找：

![](https://bbs.kanxue.com/upload/attach/201508/690458_5npt54c5ivjve95.jpg)

这个时候看到，`stringID` 的低四位发挥了作用。将算法简单还原如下：

```c
// edi 指向字符串块（第三层遍历之后的结果）
do
{
  edx = cx
  edx = *(word*)(edi + edx * 2)
  ecx += edx + 1
  esi -= 1
} while (esi >= 0);

char *szFinalString = (char*)edi + (ecx - edx) * sizeof(word)
```

此时 `szFinalString` 指向 `"ByeBye"`，也就是我们定义的目标字符串。

## 总结

`LoadString` 先通过第二个参数 `uID`，将之右移加一后，和资源表第二层的每一项 `ID` 做对比，成功后可以通过第三层直接定位到目标字符串块。

再通过 `uID` 的低四位获取到偏移，加上字符串块的首地址，从而精确定位。

写的匆忙，一些细节还没体现，不足和谬误之处请大家多多包涵。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（17.10kb，19次下载）
- [2.jpg](javascript:void(0);)（14.56kb，10次下载）
- [3.jpg](javascript:void(0);)（2.83kb，3次下载）
- [4.jpg](javascript:void(0);)（72.50kb，4次下载）
- [5.jpg](javascript:void(0);)（73.85kb，10次下载）
- [6.jpg](javascript:void(0);)（10.06kb，4次下载）
- [7.jpg](javascript:void(0);)（5.48kb，3次下载）
- [8.jpg](javascript:void(0);)（15.36kb，9次下载）
- [9.jpg](javascript:void(0);)（19.75kb，7次下载）
- [10.jpg](javascript:void(0);)（15.05kb，7次下载）
- [11.jpg](javascript:void(0);)（15.54kb，6次下载）
- [12.jpg](javascript:void(0);)（33.41kb，7次下载）
- [13.jpg](javascript:void(0);)（22.26kb，7次下载）
- [14.jpg](javascript:void(0);)（23.74kb，7次下载）
- [15.jpg](javascript:void(0);)（14.43kb，7次下载）
- [17.jpg](javascript:void(0);)（25.03kb，8次下载）
- [18.jpg](javascript:void(0);)（18.91kb，6次下载）
- [19.jpg](javascript:void(0);)（16.38kb，5次下载）
- [20.jpg](javascript:void(0);)（51.59kb，16次下载）
