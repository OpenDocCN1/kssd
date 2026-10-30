
```c
07e2b28c 00 5c 41 00 00 00 00 00 00 00 00 00 00 00 00 00 .\A.............
07e2b29c 90 7c 92 08 01 00 00 00 01 00 00 00 43 3a 5c 55 .|..........C:\U
07e2b2ac 73 65 72 73 5c 7a 6f 65 6d 5c 41 70 70 44 61 74 sers\zoem\AppDat
07e2b2bc 61 5c 4c 6f 63 61 6c 5c 54 65 6d 70 5c 4c 6f 77 a\Local\Temp\Low
07e2b2cc 5c 43 61 62 34 32 35 41 00 00 00 00 00 00 00 00 \Cab425A........
```

注意到首四个字节就是解压后大小`0x00415c00`，偏移`0x10`的位置指向了分配的`0xC`大小的空间。

`AddFile`执行完之后，程序调用`NeedFile`：

```c
int __fastcall NeedFile(int this, const CHAR *a2)
{
// [COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL-"+" TO EXPAND]
files_list = *(this + 0x10); // 0x10这里保存的就是之前提到的链表
while ( StrCmpIIA(a2, *files_list) ) // 链表遍历，找到文件名相同的位置
{
files_list = *(files_list + 4); // 0xC空间第二个位置保存的文件名指针
if ( !files_list )
goto LABEL_6;
}
if ( !*(files_list + 8) )
return 0;
LABEL_6:
if ( (*(this + 0x18) & 2) != 0 ) // 标志位，此时是1，不满足
return 1;
for ( i = *(this + 0x328); i; i = *(i + 4) ) // 0x328偏移位置，上面提到过，是0
{
if ( !StrCmpIIA(*i, a2) )
return 1;
}
return 0;
}
```

所以这次调用`NeedFile`会返回0，因此最终`fdiNotifyExtract`也会返回0。

之后`FDICopy`函数就没有什么重要操作了。

### 3.3.4 回到ExtractInfFile函数

执行完`FDICopy`函数之后，`Extract`函数调用`FDIDestroy`函数，会删除之前`FDICreate`函数创建的FDI上下文结构。`Extract`函数执行完成后，根据一开始得到的函数调用流程，程序会回到`ExtractInfFile`函数。

程序接下来会判断`GetExtnAndBaseFileName`函数的返回值是否为5，根据IDA伪代码：

```c
...
if ( !lstrcmpA(sz, "CAB") )
return 2;
if ( !lstrcmpA(sz, "OCX") )
return 4;
if ( !lstrcmpA(sz, "DLL") )
return 3;
if ( !lstrcmpA(sz, "EXE") )
return 6;
if ( !lstrcmpA(sz, "INF") )
return 5;
if ( !lstrcmpA(sz, "OSD") )
return 7;
if ( lstrcmpA(sz, "CAT") )
return v4;
return 8;
```

可以确定这个函数是在判断cab中的文件的扩展名，返回值为5时说明文件扩展名是inf。而我们测试的cab中的文件是`../msword.inf`，符合要求。

### 3.3.5 ExtractOneFile函数

之前我们提到过，在`ExtractOneFile`函数中，程序会再次调用`Extract`函数，但是除此之外，在调用`Extract`函数之前，它还执行了下面的代码：

```c
lpsz[0] = a1; // 调试发现这里的a1就是文件名../msword.inf
lpsz[1] = 0;
lpsz[2] = 0;
*(a5 + 6) = 0;
*(a5 + 0xCA) = lpsz;
```

标志位变成了0，也就是说在之后第二次传入`fdintCOPY_FILE`，程序调用`AddFile`的时候，程序会在一开始就返回1。

与此同时`*(a5 + 0xCA)`，也就是偏移值`0x328`的位置不再是0，而是传入了`lpsz`，`lpsz`的首四个字节保存的就是文件名指针。这样在调用NeedFile的时候，程序就会执行到循环体内部：

```c
for ( i = *(this + 0x328); i; i = *(i + 4) ) // 0x328偏移位置
{
if ( !StrCmpIIA(*i, a2) )
return 1;
}
```

并且由于文件名相同，函数返回1。

这次`fdiNotifyExtract`函数不会在调用`NeedFile`之后返回，而是会继续向下执行：

```c
path = v2 + 0x1C;
cchDest = (v2 + 0x120);
if ( !catDirAndFile(v2 + 0x120, 0x104, (v2 + 0x1C), fdi_ntf->psz1) || !(AddFile)(v2, fdi_ntf->psz1, fdi_ntf->cb) )
return -1;
if ( !NeedFile(v2, fdi_ntf->psz1) )
return 0;
if ( StrStrA(fdi_ntf->psz1, "\\") ) { // 因为在patch_cab进行了修改，所以不包含"\\"字符
..
}
else {
full_path = v2 + 0x120;
}
result = Win32Open(0x8302, full_path, file_name, v12, v13);
if ( result == -1 )
return -1;
return result;
```

程序会调用`Win32Open`函数，该函数会调用系统函数`CreateFileA`，按照完整路径`C:\Users\zoem\AppData\Local\Temp\Low\Cab425A\../msword.inf`创建文件

### 3.3.6 FDIGetFile函数

这次因为`fdiNotifyExtract`成功创建了要解压的文件，因此跳出回到`FDICopy`函数后，程序的流程会转到`FDIGetFile`函数的调用上：

![图片描述](upload/attach/202110/600394_4PRHDAZSSA8DVCB.png)

下面是`FDIGetFile`函数的伪代码，关键在于其中的`while`循环，`FDIGetDataBlock`函数会读取一个`CFDATA`的数据，获得其解压后的大小`curr_size_written`，每次调用`write_func`对这部分数据进行写入，然后判断尚未写入的数据`size_unwritten`是否为0，如果不为0，就继续调用`FDIGetDataBlock`读取下一个`CFDATA`。而`size_unwritten`一开始的大小等于`cbFile`，就是`0x415c00`。

```c
int __thiscall FDIGetFile(_DWORD *this) {
size_unwritten = this[0x1D]; // cbFile
if ( size_unwritten ) {
new_size_written = this[0x1E]; // 文件在文件夹中的未压缩偏移，0
old_size_written = new_size_written;
if ( new_size_written < this[0xC] )
this[0x24] = 0xFFFF;
for ( i = InitFolder(this, *(this + 62)); i; i = FDIGetDataBlock(this) ) {
if ( new_size_written < this[0xC] + *(this[0x12] + 6) ) {
while ( 1 ) {
v5 = new_size_written - this[0xC];
fdidata_size = *(this[0x12] + 6) - v5;
curr_size_written = fdidata_size;
if ( fdidata_size > size_unwritten )
{
fdidata_size = size_unwritten;
curr_size_written = size_unwritten;
}
if ( curr_size_written != (this[5])(this[0x23], v5 + this[0x10], fdidata_size) )// write_func
break;
new_size_written = curr_size_written + old_size_written;
old_size_written += curr_size_written;
size_unwritten -= curr_size_written;
if ( !size_unwritten ) // 这里判断尚未写入的数据量
goto LABEL_13;
if ( !FDIGetDataBlock(this) ) // 这里判断读取下一个CFDATA是否成功
goto LABEL_20;
}
v12 = *this;
v12[1] = 0;
* v12 = 8;
v12[2] = 1;
break;
}
}
LABEL_20:
if ( this[0x23] != -1 ) {
(this[6])(this[6], this[0x23]); // close_func 此时内容正式写入硬盘
this[0x23] = -1;
}
}
else {
LABEL_13:
...
}
return 0;
}
```

代码在这里存在一个逻辑上的漏洞，数据全部写入完成，即`size_unwritten==0`的情况，以及`CFDATA`读取失败，即`FDIGetDataBlock`函数返回值为0的情况，这两个情况的后处理方法不同。

如果是`CFDATA`读取失败，即`FDIGetDataBlock`函数返回值为0的情况，程序会跳转到`LABEL_20`，执行`close_func`关闭文件，此时对缓冲区进行flush，正式将内容写入硬盘；

如果是数据全部写入完成，即`size_unwritten==0`的情况，程序会跳转到`LABEL_13`，这部分代码我没有贴出来，有一些数据赋值的操作，我也不太清楚在干嘛，但是最终的一点在于，它调用了`fdiNotifyExtract`函数，传入的是`fdintCLOSE_FILE_INFO`参数。

也就是说两者关闭文件的函数不同，`fdiNotifyExtract`在进行关闭文件操作的时候，代码如下：

```c
if ( a1 == fdintCLOSE_FILE_INFO ) {
if ( catDirAndFile(v2 + 0x120, 260, (v2 + 28), fdi_ntf->psz1) && AdjustFileTime(fdi_ntf->time, v12, v13) ) {
CloseHandle(fdi_ntf->hf);
v4 = fdi_ntf->attribs;
v5 = v4 ? v4 & 0xFFFFFFD8 : 0x80;
if ( SetFileAttributesA(v2 + 0x120, v5) ) {
MarkExtracted(v2, v6);
return 1;
}
}
}
```

关键在于`MarkExtracted`函数：

```c
int __thiscall MarkExtracted(_DWORD *this, int a2) {
v2 = this[4];
while ( StrCmpIIA(v4, v5) ) {
...
}
*(v2 + 8) = 0; // 注意这里的标志位清零
return 1;
}
```

可能从伪代码看不出来这个标志位是什么。通过windbg调试，我发现它修改的就是3.3.3小结中提到的调用`AddFile`的时候分配的0xC空间的第三位，当时是设置成了1，我还不知道指的是什么，现在看来这个第三位应该是**是否解压**的标志。

### 3.3.7 DeleteExtractedFiles函数

`FDIGetFile`执行完之后，没有什么重要的操作，一直跳出，知道到达`GetSupportedInstallScopesFromFile`函数（调用`ExtractInfFile`的位置），然后就执行到了`DeleteExtractedFiles`函数。

```c
void __thiscall DeleteExtractedFiles(_DWORD *this) {
v2 = this[4]; // 这里就是0xC的那个空间
while ( v2 ) {
if ( !v2[2] && catDirAndFile(FileName, 260, (this + 7), *v2) && SetFileAttributesA(FileName, 0x80u) )
DeleteFileA(FileName);
CoTaskMemFree(*v2);
v3 = v2;
v2 = v2[1];
CoTaskMemFree(v3);
}
this[4] = 0;
}
```

这个函数在`if`条件语句中检查了上面提到的**是否解压**的标志标志位，只有在这个标志位是0的情况下，才会继续往下执行其他函数，包括`DeleteFileA`函数。

因为在`FDIGetFile`函数中，程序没有正确的对这个标志位清零，因此`DeleteFileA`函数不会被执行，释放的mfword.inf保留了下来，导致后面可以通过`.cpl:../../../AppData/Local/Temp/Low/msword.inf`路径执行释放的文件。

### 3.3.8 小结

到目前为止，我们已经完整分析了整个漏洞利用流程，同时2.4小结提出的四个问题也都得到了解答。

经过分析，漏洞主要存在于两个位置：

1. `urlmon!fdiNotifyExtract`函数中对于分隔符的判断，只判断了`\\`，而没有考虑`/`的情况，导致目录遍历情况的出现

2. `cabinet!FDIGetFile`，当`cbFile`大于所有`CFDATA`的大小，或者其他情况导致`FDIGetDataBlock`调用失败的时候，后处理步骤缺少标志位清除操作

# 4. 补丁比较

我没找到适用于1709的补丁版本，不过主机也是win10的，所以直接比较了主机中和虚拟机中的文件，发现微软并没有对上面提到的两个函数进行修改，而是修改了`catDirAndFile`函数：

![图片描述](upload/attach/202110/600394_47FTMZXUK99XQVK.png)

新添加的部分查找了路径中的`/`字符，并替换成了`\\`：

![图片描述](upload/attach/202110/600394_AM2WQ57DSZZASXM.png)

至于标志位的清除操作则没有进行修改，可能是因为这个问题无伤大雅吧，因为缺少了目录遍历的条件，释放的文件位于一个随机的文件夹下，攻击者无法定位到释放的文件，因此即使释放的文件没有被正确删除，也不会造成危害。

# 5. 总结

这篇文章从poc代码，cab文件格式以及IDA静态代码分析+windbg调试三个方面对cve-2021-40444漏洞进行了比较全面的分析。其实漏洞分析本身并不比之前更难，但是明显能够感觉到，因为不像之前有详细的资料可以参考（虽然也有分析文章），不确定哪些地方是真正和漏洞相关的内容，我的个人探索会比较多，再加上本身也想要把整个执行流程弄清楚，因此这篇文章也就比较长。

# 6. 参考资料

1. <a href="elink@20bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6T1j5U0b7I4y4K6x3@1x3#2)9J5z5s2k6Q4x3@1c8E0M7$3c8F1i4K6u0W2x3e0l9`.">Microsoft Cabinet Format</a>)

2. <a href="elink@387K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0p5%4x3e0S2Q4x3V1k6Q4x3U0x3H3P5o6l9I4i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9J5x3g2)9J5k6o6b7H3y4o6b7@1">CVE-2021-40444 漏洞深入分析</a>
