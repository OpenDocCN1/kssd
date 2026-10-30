
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 | `int` `__fastcall NeedFile(` `int` `this, const CHAR` `*` `a2)` `{` `/` `/` `[COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL` `-` `+` `TO EXPAND]` `files_list` `=` `*` `(this` `+` `0x10` `);` `/` `/` `0x10` `这里保存的就是之前提到的链表` `while` `( StrCmpIIA(a2,` `*` `files_list) )` `/` `/` `链表遍历，找到文件名相同的位置` `{` `files_list` `=` `*` `(files_list` `+` `4` `);` `/` `/` `0xC` `空间第二个位置保存的文件名指针` `if` `( !files_list )` `goto LABEL_6;` `}` `if` `( !` `*` `(files_list` `+` `8` `) )` `return` `0` `;` `LABEL_6:` `if` `( (` `*` `(this` `+` `0x18` `) &` `2` `) !` `=` `0` `)` `/` `/` `标志位，此时是` `1` `，不满足` `return` `1` `;` `for` `( i` `=` `*` `(this` `+` `0x328` `); i; i` `=` `*` `(i` `+` `4` `) )` `/` `/` `0x328` `偏移位置，上面提到过，是` `0` `{` `if` `( !StrCmpIIA(` `*` `i, a2) )` `return` `1` `;` `}` `return` `0` `;` `}` |

所以这次调用`NeedFile`会返回0，因此最终`fdiNotifyExtract`也会返回0。

之后`FDICopy`函数就没有什么重要操作了。

### 3.3.4 回到ExtractInfFile函数

执行完`FDICopy`函数之后，`Extract`函数调用`FDIDestroy`函数，会删除之前`FDICreate`函数创建的FDI上下文结构。`Extract`函数执行完成后，根据一开始得到的函数调用流程，程序会回到`ExtractInfFile`函数。

程序接下来会判断`GetExtnAndBaseFileName`函数的返回值是否为5，根据IDA伪代码：

| | |
|---|---|
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 | `...` `if` `( !lstrcmpA(sz,` `"CAB"` `) )` `return` `2` `;` `if` `( !lstrcmpA(sz,` `"OCX"` `) )` `return` `4` `;` `if` `( !lstrcmpA(sz,` `"DLL"` `) )` `return` `3` `;` `if` `( !lstrcmpA(sz,` `"EXE"` `) )` `return` `6` `;` `if` `( !lstrcmpA(sz,` `"INF"` `) )` `return` `5` `;` `if` `( !lstrcmpA(sz,` `"OSD"` `) )` `return` `7` `;` `if` `( lstrcmpA(sz,` `"CAT"` `) )` `return` `v4;` `return` `8` `;` |

可以确定这个函数是在判断cab中的文件的扩展名，返回值为5时说明文件扩展名是inf。而我们测试的cab中的文件是`../msword.inf`，符合要求。

### 3.3.5 ExtractOneFile函数

之前我们提到过，在`ExtractOneFile`函数中，程序会再次调用`Extract`函数，但是除此之外，在调用`Extract`函数之前，它还执行了下面的代码：

| | |
|---|---|
| 1 2 3 4 5 | `lpsz[` `0` `]` `=` `a1;` `/` `/` `调试发现这里的a1就是文件名..` `/` `msword.inf` `lpsz[` `1` `]` `=` `0` `;` `lpsz[` `2` `]` `=` `0` `;` `*` `(a5` `+` `6` `)` `=` `0` `;` `*` `(a5` `+` `0xCA` `)` `=` `lpsz;` |

标志位变成了0，也就是说在之后第二次传入`fdintCOPY_FILE`，程序调用`AddFile`的时候，程序会在一开始就返回1。

与此同时`*(a5 + 0xCA)`，也就是偏移值`0x328`的位置不再是0，而是传入了`lpsz`，`lpsz`的首四个字节保存的就是文件名指针。这样在调用NeedFile的时候，程序就会执行到循环体内部：

| | |
|---|---|
| 1 2 3 4 5 | `for` `( i` `=` `*` `(this` `+` `0x328` `); i; i` `=` `*` `(i` `+` `4` `) )` `/` `/` `0x328` `偏移位置` `{` `if` `( !StrCmpIIA(` `*` `i, a2) )` `return` `1` `;` `}` |

并且由于文件名相同，函数返回1。

这次`fdiNotifyExtract`函数不会在调用`NeedFile`之后返回，而是会继续向下执行：

| | |
|---|---|
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 | `path` `=` `v2` `+` `0x1C` `;` `cchDest` `=` `(v2` `+` `0x120` `);` `if` `( !catDirAndFile(v2` `+` `0x120` ,`0x104` , (v2` `+` `0x1C` `), fdi_ntf` `-` `>psz1) || !(AddFile)(v2, fdi_ntf` `-` `>psz1, fdi_ntf` `-` `>cb) )` `return` `-` `1` `;` `if` `( !NeedFile(v2, fdi_ntf` `-` `>psz1) )` `return` `0` `;` `if` `( StrStrA(fdi_ntf` `-` `>psz1,` `"\\"` `) ) {` `/` `/` `因为在patch_cab进行了修改，所以不包含“\\”字符` `..` `}` `else` `{` `full_path` `=` `v2` `+` `0x120` `;` `}` `result` `=` `Win32Open(` `0x8302` `, full_path, file_name, v12, v13);` `if` `( result` `=` `=` `-` `1` `)` `return` `-` `1` `;` `return` `result;` |

程序会调用`Win32Open`函数，该函数会调用系统函数`CreateFileA`，按照完整路径`C:\Users\zoem\AppData\Local\Temp\Low\Cab425A\../msword.inf`创建文件

### 3.3.6 FDIGetFile函数

这次因为`fdiNotifyExtract`成功创建了要解压的文件，因此跳出回到`FDICopy`函数后，程序的流程会转到`FDIGetFile`函数的调用上：

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_4PRHDAZSSA8DVCB.png)

下面是`FDIGetFile`函数的伪代码，关键在于其中的`while`循环，`FDIGetDataBlock`函数会读取一个`CFDATA`的数据，获得其解压后的大小`curr_size_written`，每次调用`write_func`对这部分数据进行写入，然后判断尚未写入的数据`size_unwritten`是否为0，如果不为0，就继续调用`FDIGetDataBlock`读取下一个`CFDATA`。而`size_unwritten`一开始的大小等于`cbFile`，就是`0x415c00`。

| | |
|---|---|
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 | `int` `__thiscall FDIGetFile(_DWORD` `*` `this) {` `size_unwritten` `=` `this[` `0x1D` `];` `/` `/` `cbFile` `if` `( size_unwritten ) {` `new_size_written` `=` `this[` `0x1E` `];` `/` `/` `文件在文件夹中的未压缩偏移，` `0` `old_size_written` `=` `new_size_written;` `if` `( new_size_written < this[` `0xC` `] )` `this[` `0x24` `]` `=` `0xFFFF` `;` `for` `( i` `=` `InitFolder(this,` `*` `(this` `+` `62` `)); i; i` `=` `FDIGetDataBlock(this) ) {` `if` `( new_size_written < this[` `0xC` `]` `+` `*` `(this[` `0x12` `]` `+` `6` `) ) {` `while` `(` `1` `) {` `v5` `=` `new_size_written` `-` `this[` `0xC` `];` `fdidata_size` `=` `*` `(this[` `0x12` `]` `+` `6` `)` `-` `v5;` `curr_size_written` `=` `fdidata_size;` `if` `( fdidata_size > size_unwritten )` `{` `fdidata_size` `=` `size_unwritten;` `curr_size_written` `=` `size_unwritten;` `}` `if` `( curr_size_written !` `=` `(this[` `5` `])(this[` `0x23` `], v5` `+` `this[` `0x10` `], fdidata_size) )` `/` `/` `write_func` `break` `;` `new_size_written` `=` `curr_size_written` `+` `old_size_written;` `old_size_written` `+` `=` `curr_size_written;` `size_unwritten` `-` `=` `curr_size_written;` `if` `( !size_unwritten )` `/` `/` `这里判断尚未写入的数据量` `goto LABEL_13;` `if` `( !FDIGetDataBlock(this) )` `/` `/` `这里判断读取下一个CFDATA是否成功` `goto LABEL_20;` `}` `v12` `=` `*` `this;` `v12[` `1` `]` `=` `0` `;` `*` `v12` `=` `8` `;` `v12[` `2` `]` `=` `1` `;` `break` `;` `}` `}` `LABEL_20:` `if` `( this[` `0x23` `] !` `=` `-` `1` `) {` `(this[` `6` `])(this[` `6` `], this[` `0x23` `]);` `/` `/` `close_func 此时内容正式写入硬盘` `this[` `0x23` `]` `=` `-` `1` `;` `}` `}` `else` `{` `LABEL_13:` `...` `}` `return` `0` `;` `}` |

代码在这里存在一个逻辑上的漏洞，数据全部写入完成，即`size_unwritten==0`的情况，以及`CFDATA`读取失败，即`FDIGetDataBlock`函数返回值为0的情况，这两个情况的后处理方法不同。

如果是`CFDATA`读取失败，即`FDIGetDataBlock`函数返回值为0的情况，程序会跳转到`LABEL_20`，执行`close_func`关闭文件，此时对缓冲区进行flush，正式将内容写入硬盘；

如果是数据全部写入完成，即`size_unwritten==0`的情况，程序会跳转到`LABEL_13`，这部分代码我没有贴出来，有一些数据赋值的操作，我也不太清楚在干嘛，但是最终的一点在于，它调用了`fdiNotifyExtract`函数，传入的是`fdintCLOSE_FILE_INFO`参数。

也就是说两者关闭文件的函数不同，`fdiNotifyExtract`在进行关闭文件操作的时候，代码如下：

| | |
|---|---|
| 1 2 3 4 5 6 7 8 9 10 11 | `if` `( a1` `=` `=` `fdintCLOSE_FILE_INFO ) {` `if` `( catDirAndFile(v2` `+` `0x120` ,`260` , (v2` `+` `28` `), fdi_ntf` `-` `>psz1) && AdjustFileTime(fdi_ntf` `-` `>time, v12, v13) ) {` `CloseHandle(fdi_ntf` `-` `>hf);` `v4` `=` `fdi_ntf` `-` `>attribs;` `v5` `=` `v4 ? v4 &` `0xFFFFFFD8` `:` `0x80` `;` `if` `( SetFileAttributesA(v2` `+` `0x120` `, v5) ) {` `MarkExtracted(v2, v6);` `return` `1` `;` `}` `}` `}` |

关键在于`MarkExtracted`函数：

| | |
|---|---|
| 1 2 3 4 5 6 7 8 | `int` `__thiscall MarkExtracted(_DWORD` `*` `this,` `int` `a2) {` `v2` `=` `this[` `4` `];` `while` `( StrCmpIIA(v4, v5) ) {` `...` `}` `*` `(v2` `+` `8` `)` `=` `0` `;` `/` `/` `注意这里的标志位清零` `return` `1` `;` `}` |

可能从伪代码看不出来这个标志位是什么。通过windbg调试，我发现它修改的就是3.3.3小结中提到的调用`AddFile`的时候分配的0xC空间的第三位，当时是设置成了1，我还不知道指的是什么，现在看来这个第三位应该是**是否解压**的标志。

### 3.3.7 DeleteExtractedFiles函数

`FDIGetFile`执行完之后，没有什么重要的操作，一直跳出，知道到达`GetSupportedInstallScopesFromFile`函数（调用`ExtractInfFile`的位置），然后就执行到了`DeleteExtractedFiles`函数。

| | |
|---|---|
| 1 2 3 4 5 6 7 8 9 10 11 12 | `void __thiscall DeleteExtractedFiles(_DWORD` `*` `this) {` `v2` `=` `this[` `4` `];` `/` `/` `这里就是` `0xC` `的那个空间` `while` `( v2 ) {` `if` `( !v2[` `2` `] && catDirAndFile(FileName,` `260` `, (this` `+` `7` `),` `*` `v2) && SetFileAttributesA(FileName,` `0x80u` `) )` `DeleteFileA(FileName);` `CoTaskMemFree(` `*` `v2);` `v3` `=` `v2;` `v2` `=` `v2[` `1` `];` `CoTaskMemFree(v3);` `}` `this[` `4` `]` `=` `0` `;` `}` |

这个函数在`if`条件语句中检查了上面提到的**是否解压**的标志标志位，只有在这个标志位是0的情况下，才会继续往下执行其他函数，包括`DeleteFileA`函数。

因为在`FDIGetFile`函数中，程序没有正确的对这个标志位清零，因此`DeleteFileA`函数不会被执行，释放的mfword.inf保留了下来，导致后面可以通过`.cpl:../../../AppData/Local/Temp/Low/msword.inf`路径执行释放的文件。

### 3.3.8 小结

到目前为止，我们已经完整分析了整个漏洞利用流程，同时2.4小结提出的四个问题也都得到了解答。

经过分析，漏洞主要存在于两个位置：

1. `urlmon!fdiNotifyExtract`函数中对于分隔符的判断，只判断了`\\`，而没有考虑`/`的情况，导致目录遍历情况的出现
