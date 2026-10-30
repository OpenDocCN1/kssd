
分配大小是`disk`为`stream`保留的内存空间

实际大小为`caller`使用的实际字节数

有效数据长度(`VDL`)是根据`stream`的分配大小初始化的字节数

### Naming Coventions for Streams

当在Windows的命令行中指定文件时，一个完整的`stream`名称如下：

```
filename:stream_name:stream type

example:

myfile.dat:stream1:$DATA

```

#### 3. SMB2_SET_INFO Message

`client`发送`SMB2 SET_INFO`请求数据包以设置有关文件或基础对象存储的信息，该`message`同样包含一个标准的的`SMB2_Header`，然后包含以下数据结构：

| Offset(bytes) | Size | Field | Description |
| --- | --- | --- | --- |
| 0x00 | 0x02 | StructureSize | `client`必须设置该字段为`0x21`，表明请求结构的大小，不包括`header`部分 |
| 0x02 | 0x01 | InfoType | 信息类型，其可选值如图`InfoType`所示 |
| 0x03 | 0x01 | FileInfoClass | 为了设置文件信息，该字段必须包含以下`FILE_INFORMATION_CLASS`值之一：`FileAllocationInformation`，`FileBasicInformation`，`FileDispositionInformation`，`FileEndOfFileInformation`，`FileFullEaInformation`，`FileLinkInformation`，`FileModeInformation`，`FilePipeInformation`，`FilePositionInformation`，`FileRenameInformation`，`FileShortNameInformation`，`FileValidDataLengthInformation`；为了设置基础对象存储信息，该字段必须包含以下`FS_INFORMATION_CLASS`值之一：`FileFsControlInformation`，`FileFsObjectIdInformation`；为了设置配额和安全性信息，该字段必须为`0`。 |
| 0x04 | 0x04 | BufferLength(L) | 要设置的信息的长度（以字节为单位） |
| 0x08 | 0x02 | BufferOffset | 从`SMB2`标头的开头到要设置的信息的偏移量（以字节为单位） |
| 0x0a | 0x02 | Reserved | 保留字段 |
| 0x0c | 0x04 | AdditionalInformation | 向`server`提供其他信息，如果设置了安全信息，则该值必须包含`4`字节的标志位字段，以指示必须应用哪些安全属性。详细可选值如图`AdditionalInformation`所示 |
| 0x10 | 0x10 | FileID | 在其上执行设置的文件或命名管道的`SMB2_FILEID`标识符，基础对象存储和配额信息的设置操作将定向到文件所在的卷 |
| 0x20 | L | Buffer | 可变长度缓冲区，其中包含为请求设置的信息，如`BufferOffset`和`BufferLength`字段所述。缓冲区格式取决于`InfoType`和`AdditionalInformation`，详细的`Buffer`结构如图所示 |

`InfoType`字段的可选值：

![](https://bbs.kanxue.com/upload/attach/202008/779730_H6BUXRVMU8ZE6NZ.png)

`AdditionalInformation`字段可选值如下：

![](https://bbs.kanxue.com/upload/attach/202008/779730_754BDHQTTTY7GV8.png)

`Buffer`字段详细可选值：

![](https://bbs.kanxue.com/upload/attach/202008/779730_RZWKTSEATUJBPD3.png)

### 3. 详细分析

当前面`Buffer`字段包含一个`FileRenameInformation`类时，`InfoType`字段的值为`SMB2_0_INFO_FILE(0x01)`，`FileInfoClass`值为`0x0a`。`FileRenameInformation`类用于重命名一个文件，其格式如下所示：

| Offset(bytes) | Size | Field | Description |
| --- | --- | --- | --- |
| 0x00 | 0x01 | ReplaceIfExists | `boolean`值，设置为`TRUE`表示如果具有给定名称的文件已经存在，则应将其替换为给定文件。设置为`FALSE`表示如果给定名称的文件已经存在，重命名操作必须失败。 |
| 0x01 | 0x07 | Reserved | 为进行对齐保留的字段，可以包含任意值，但会被忽略处理 |
| 0x08 | 0x08 | RootDirectory | `64`位无符号整数，其中包含文件新名称相对于的目录的文件句柄。对于网络操作，该值必须始终为零。 |
| 0x10 | 0x04 | FileNameLength(K) | `32`位无符号整数，`FileName`字段数据长度，字节为单位 |
| 0x0e | K | FileName | 包含文件新名称的`Unicode`字符序列。使用此字段时，使用`FileNameLength`来确定文件名的长度，而不要假定存在结尾的空定界符。 |

漏洞出现原因主要为由于对`filename`长度验证不完善造成。其主要过程如下：

1. 在通过`SMBv2`重命名文件时，原有文件命名为`OldFileName`，首先通过发送`SMB2_CREATE`请求消息到`server`，如果请求的文件存在则`server`会返回一个`SMB2_CREATE`响应消息，该消息中包含名为`FileGUID`的文件标识符；
2. `client`发送一个`SMB2_SET_INFO`请求消息，其中`FileID`字段为`FileGUID`，`Buffer`字段为`FileRenameInformation`中的新文件名。一旦该请求被`server`接收，`server`就会通过`Smb2ExecuteSetInfoReal()`函数调用`Smb2UpdateLeaseFileName()`函数；
3. `Smb2UpdateLeaseFileName()`函数会检查`FileNameLength`字段是否小于`0xffff`。如果小于，会检查`FileName`字段的值是否以`:`开头，以确认字段中是否包含`stream`。如果包含`stream`，该函数遍历`OldFileName`并检查`OldFileName`的最后一个路径名是否包含`stream`。如果发现`OldFileName`包含一个`stream`，函数会移除`stream`字符串，并将剩余的前缀字符串存储到名为`prefixFileName`的变量中，其长度为`prefixFileNameLen`；
4. `Smb2UpdateLeaseFileName()`函数没有对`prefixFileNameLen`进行验证，直接按照以下公式计算新文件名的长度：

```
FileLengthVar = prefixFileNameLen * 2 + FileNameLength
```

`FileLengthVar`类型为无符号`short`类型。

5. `Smb2UpdateLeaseFileName()`函数检查计算得到的`FileLengthVar`是否大于`OldFileName`的`unicode`形式的长度，称为`unicodeFileLen`。如果大于，函数创建一个大小为`FileLengthVar`的`buffer`并将`prefixFileName`和`FileName`拷贝到`buffer`中；如果`FileLengthVar`小于等于`unicodeFileLen`，函数创建的`buffer`大小变为`unicodeFileLen`，并进行复制操作。

以上函数处理过程可简单描述为如下流程图：

![](https://bbs.kanxue.com/upload/attach/202008/779730_CXSFV68PAPE4JMR.png)

函数在计算`FileLengthVar`的长度之前并没有对`prefixFileNameLen`进行验证，如果`prefixFileNameLen`为一个任意大小的值，那么就可能导致`FileLengthVar`大于`0xffff`，从而发生整数溢出。在这种情况下，`FileLengthVar`会小于`unicodeFileLen`，那么在后续进行内存分配时分配的内存大小为`unicodeFileLen`。但是在进行复制操作时，复制的数据为`prefixFileName`和`FileName`，其长度和大于`unicodeFileLen`，导致在进行`copy`操作时发生溢出。

一个`stream`允许的最多字符数量为`0x200`（包含`":"`符号在内），那么`FileNameLength`字段的最大值为`0x400 = 0x200 * 2`，为触发该漏洞，`SMB2_CREATE`消息中的`NameLength`字段的值必须大于`0xfbff(0xffff - 0x400)`。

#### 1. 静态分析

反汇编存在漏洞的文件：`srv2.sys`，来自`Windows Server 2012 R2 X64`，版本为`6.3.9600.17396`。

##### 1. Smb2ExecuteSetInfoReal()函数

首先处理相关参数和信息：

![](https://bbs.kanxue.com/upload/attach/202008/779730_E47M2P3RSX9MF2D.png)

然后调用存在漏洞的`Smb2UpdateLeaseFileName()`函数：

![](https://bbs.kanxue.com/upload/attach/202008/779730_WPNG4STRNQ7HMDD.png)

##### 2. Smb2UpdateLeaseFileName()函数

首先检查`FileNameLength`是否小于`0xFFFF`：

![](https://bbs.kanxue.com/upload/attach/202008/779730_Q3J4EZG8XN6PW52.png)

如果小于`0xFFFF`，则进行循环搜索`:`符号：

![](https://bbs.kanxue.com/upload/attach/202008/779730_FYMBRNQ6E7WDWQU.png)

然后按照之前的公式进行计算，而且可以明显看出，在进行计算之前并没有进行任何的数据验证，导致漏洞存在：

![](https://bbs.kanxue.com/upload/attach/202008/779730_SUTJ9U6S4QTJT4V.png)

最后，进行缓冲区分配和`copy`操作：

![](https://bbs.kanxue.com/upload/attach/202008/779730_U8AQCE82ZR6FDZT.png)

简单总结以上过程，真正的触发点在于`FileLengthVar`没有进行数据检查，如果超过`0xFFFF`，则未通过`if`判断，从而分配的缓冲区大小为`unicodeFileLen`，但`copy`的数据为`prefixFileName`和`FileName`，其长度和大于`unicodeFileLen`，导致在进行`copy`操作时发生溢出：

![](https://bbs.kanxue.com/upload/attach/202008/779730_XHKEDNRFH956NYC.png)

#### 2. 动态调试

因为确认了出问题的函数，所以在`Smb2ExecuteSetInfoReal()`函数下断，发送`poc`，断下：

![](https://bbs.kanxue.com/upload/attach/202008/779730_XY722J88YS6K8S5.png)

找到调用`Smb2UpdateLeaseFileName()`函数处，下断，执行：

![](https://bbs.kanxue.com/upload/attach/202008/779730_QETGC7D625ZBPH5.png)

`Smb2UpdateLeaseFileName()`函数有`3`个参数，第一个参数`rcx`中存放的是`FileInfoClass`，偏移为`0x03`，主要用于定位栈中数据使用（猜测）；第`2`个参数`rdx`中存放的是我们`poc`中发送的用于修改的`FileName`的具体内容，以`:`开头；第`3`个参数`r8`中存放的是`FileNameLength`，恰好为我们`poc`中构造的`512`个字节(包含`:`)。自此，进入函数后将开始进行文件重命名过程：

![](https://bbs.kanxue.com/upload/attach/202008/779730_5N9JCYB8DZEP3UP.png)

这里对新的`FileName`的长度做了校验，要求小于`0xFFFF`。然后，小写转大写：

![](https://bbs.kanxue.com/upload/attach/202008/779730_WYHRWYB8B882M33.png)

转换完成：

![](https://bbs.kanxue.com/upload/attach/202008/779730_RQX5D8RFQET4JDD.png)

检查新的`FileName`是否以`:`开头：

![](https://bbs.kanxue.com/upload/attach/202008/779730_5XBQU8M24QCTKY4.png)

因为在新的`FileName`中找到了`:`字符，所以转而在`OldFileName`中查找`:`字符，以便进行后续处理：

![](https://bbs.kanxue.com/upload/attach/202008/779730_UCP5XPA7TZ9YTST.png)

![](https://bbs.kanxue.com/upload/attach/202008/779730_BGPC8GT7BYBVFK6.png)

这里获取到`OldFileName`的`:`字符前面部分的长度`prefixFileNameLen = 0x7f04`：

![](https://bbs.kanxue.com/upload/attach/202008/779730_Z62APNB9JGYGC3E.png)

然后按照公式

```
FileLengthVar = prefixFileNameLen * 2 + FileNameLength

# prefixFileNameLen * 2还是因为unicode长度
```

计算重命名后新的完整的文件名的长度：

![](https://bbs.kanxue.com/upload/attach/202008/779730_3GTKGPBE3A2HGU6.png)

在进行计算时，使用的寄存器为`bp`，`prefixFileNameLen`长度为`0x7f04`，`FileNameLength`长度为`0x200`，所以最终的计算结果为`0x10008`。但是因为使用`bp`进行最后计算结果的存储，所以发生整数溢出，导致最后结果变为`0x8`。

然后，进行需要分配的缓冲区大小的计算：

![](https://bbs.kanxue.com/upload/attach/202008/779730_UTPVA4ADRPZMPYP.png)

在进行内存分配时，根据不同的计算结果有`2`种不同的内存分配的路径：

![](https://bbs.kanxue.com/upload/attach/202008/779730_TT9BBUQPMKJV9BK.png)

开始进行`copy`操作，使用的函数为`memcpy()`：

![](https://bbs.kanxue.com/upload/attach/202008/779730_KZ7Z3SY6WDP6ZTQ.png)

执行到这里的前提是计算得到的`FileNameLengVar`小于`unicodeFileNameLen`，但是原来的`OldFileName`的`stream`长度为`1`，现在新增的`stream`长度为`0x200`，明显会发生溢出，会覆盖掉`rcx`指向的地址下面的数据。

进行`copy`前`rcx`指向的地址空间的数据布局如下：

![](https://bbs.kanxue.com/upload/attach/202008/779730_FSG5QAYG2TCMDUK.png)

而在执行完`copy`操作后，`rcx`指向的地址的内存布局变成了如下所示，多余的`0x1fe`个字节覆盖了原来的一些地址和数据，很明显会影响到后续程序的正常运行：

![](https://bbs.kanxue.com/upload/attach/202008/779730_XH5CJRSM3QBRG28.png)

然后直接运行，程序发生`crash`，根据栈回溯可以获取一些信息：

![](https://bbs.kanxue.com/upload/attach/202008/779730_THGY96ZPDFXTCN6.png)

因为在重命名文件时发生了缓冲区溢出，覆盖掉了一些关键的地址或者参数，而这些被覆盖的数据可能正是后续函数需要使用的数据。根据栈回溯信息，初步判断是影响了`CloseFile`的相关函数。

### 4. 利用思路

##### 1. 利用条件

1. 基本条件

   - 可以通过网络访问到目标`host`
   - 具有可以访问目标的`share`的`username`、`password`

2. 触发过程

   - `attacker`首先创建一个恶意的过长文件名的文件，然后发起重命名文件`stream name`的请求

##### 2. 利用过程

1. `attacker`首先建立`SMBv2`连接，包括 `Negotiate Protocol`，`Session Setup`， `Tree Connect messages`的交换：
