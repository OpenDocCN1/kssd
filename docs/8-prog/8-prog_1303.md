
# 晒下我的第 5 题解法~

> 原文：[`www.pediy.com/kssd/pediy11/104640.html`](https://www.pediy.com/kssd/pediy11/104640.html)

## 题目分析与初步尝试

这题是个文件保护，在 `Hijack.sys` 保护 `C:\windows\system32\360safe.txt` 成功后把它删掉。

`HijackFile.sys` 的关键代码 VM 过了，所以只能黑盒之。

刚开始我的虚拟机中 `C` 盘是 `FAT32`，还得我转成 `NTFS` 的。

然后加载 `HijackFile.sys`，结果蓝掉了...

查看了一下 dump，在调用栈中发现了 `HijackFile.sys` 的踪迹，根据调用关系直接锁定 `ntfs.sys` 进行分析，也算意外收获。

其实就算没有这个意外，多检查一下基本也能发现它动手脚的地方，如果再参考一下 sudami 的[《NTFS 文件系统底层挖掘》](http://bbs.pediy.com/showthread.php?t=87741)的话，就更简单了。

## 检查 Hook

我用的是 `Windbg` 检查 Hook：

```
lkd> !chkimg -d !ntfs
    f82fd55a-f82fd55e  5 bytes - Ntfs!NtfsCheckValidAttributeAccess
  [ 8b ff 55 8b ec:e9 07 89 e0 89 ]
    f82fe1fe-f82fe201  4 bytes - Ntfs!NtfsOpenAttributeInExistingFile+b (+0xca4)
  [ 1e c1 fd ff:3c 5f ec 89 ]
9 errors : !ntfs (f82fd55a-f82fe201)
```

再细看一下：

### 第一个钩子

```
lkd> u NtfsCheckValidAttributeAccess
Ntfs!NtfsCheckValidAttributeAccess:
f82fd55a e90789e089       jmp     82105e66
f82fd55f 8b4508           mov     eax,dword ptr [ebp+8]
f82fd562 53               push    ebx
f82fd563 56               push    esi
f82fd564 0fb6700b         movzx   esi,byte ptr [eax+0Bh]

lkd> u 82105e66
82105e66 e9d509f973       jmp     f6096840
82105e6b 8bff             mov     edi,edi //可以借用这部分代码跳回去~~
82105e6d 55               push    ebp
82105e6e 8bec             mov     ebp,esp
82105e70 e9ea761f76       jmp     Ntfs!NtfsCheckValidAttributeAccess+0x5 (f82fd55f)

lkd> u f6096840 //这个就是 Fake 函数了,被 VM 过
HijackFile+0x1840:
f6096840 8bff             mov     edi,edi
f6096842 55               push    ebp
f6096843 8bec             mov     ebp,esp
```

### 第二个钩子

```
lkd> u NtfsOpenAttributeInExistingFile
Ntfs!NtfsOpenAttributeInExistingFile:
f82fe1f3 68ac000000       push    0ACh
f82fe1f8 68f0252ff8       push    offset Ntfs!\`string'+0x12c (f82f25f0)
f82fe1fd e83c5fec89       call    821c413e //被替换了，这里本应是 call Ntfs!\_SEH\_prolog
f82fe202 8365dc00         and     dword ptr [ebp-24h],0
f82fe206 8b7510           mov     esi,dword ptr [ebp+10h]

lkd> u 821c413e
821c413e e9ed28ed73       jmp     HijackFile+0x1a30 (f6096a30)
821c4143 e9d8611176       jmp     Ntfs!\_SEH\_prolog (f82da320)
```

基本上就很明确了。这两个函数调用 `NtfsCheckValidAttributeAccess` 的地方有 `NtfsOpenFcbById`, `NtFsOpenExistingPrefixFcb`, `NtfsOpenFile`, `NtfsCreateNewFile`。
`NtfsOpenAttributeInExistingFile` 被调用的地方有 `NtfsOpenFile`, `NtFsOpenExistingPrefixFcb`, `NtfsOpenFile`。

要打开存在的文件需要调用 `NtfsOpenFile`，而这个函数里有这样两个钩子，所以文件无法打开。

## 解决方法

因为不能对它的钩子做任何改动，所以方法无非是能绕就绕，绕不过就自己实现。

对于经常搞 Hook 的，想到的第一个方法就是以 Hook 绕过 Hook。

### 方法一：绕过 Hook

1. 对 `NtfsCheckValidAttributeAccess` 处的钩子，因为它已经是在函数头了，在这里已经无法再 Hook，所以直接查找所有对 `NtfsCheckValidAttributeAccess` 的 Call，也就是上面四个函数中，替换 Call 的地址到我们自己的函数中，然后执行覆盖掉的三句指令后再跳到 `NtfsCheckValidAttributeAccess+5` 处就可以了(注意看那个 Pool 的代码，也可以直接修正 call 到 `82105e6b`，借用 sudami 的代码跳回去。。。).

2. 对 `NtfsOpenAttributeInExistingFile` 处的钩子，因为采用的是替换 Call 的方法，而前面还有 10 个字节没有动，所以可以 Hook 在 `NtfsOpenAttributeInExistingFile` 头部，或者 `NtfsOpenAttributeInExistingFile+5` 的地方来跳过 `HijackFile.sys` 的钩子。

这种方法不难，写过 Hook 的基本都会，关键是需要得到几个 `NtfsXx` 函数的地址，本来自己找挺麻烦的，无非栈回溯而已。但是既然 sudami 已经帮我们找好了，为什么不用呢?看下面对部分数据的分析(`f8844000` 是 `HijackFile.sys` 的加载基址):

代码:

```
kd> //从 f8844000+5000+220 处开始才是有用数据，前面是一个反汇编引擎 LDasm 用的表
f8849200  01010101 00010101 ffff4d09 0000b2f6
f8849210  00000000 00000000 00000000 00000000
f8849220  00000174 000039d8 00000000 f830f0eb //f830f0eb 是 NtfsOpenFile 中 call NtfsCreateFcb 后的返回地址，可靠
f8849230  805427c4 00000000 f82fe6a0 f830f021 //[f8849230]=NtfsCreateNewFile，不可靠  ,f82fe6a0=NtfsOpenExistingPrefixFcb, f830f021=NtfsOpenFile
f8849240  f82fe1f3 00000000 00000001 00000001 //f82fe1f3=NtfsOpenAttributeInExistingFile
f8849250  00000000 8219cb08 00000000 00000000 //8219cb08 指向一个结构，未明确
f8849260  8001003b 00000000 00000000 00000000 //8001003b 是 OldCr0
f8849270  00000000 00000000 00000000 00000000
kd> 
f8849280  00000001 00000000 82118da8 8209d050 //82118da8 未发现有意义数据 , 8209d050 未发现有意义数据
f8849290  8206d958 8218bc08 00000000 00000000 //两个 FUNCTION_INFO 指针
f88492a0  00000000 00000000 00000000 00000000
f88492b0  00000001 0000000f f82fe1fe 00000000 //f82fe1fe 是 NtfsOpenAttributeInExistingFile 中被替换掉的 call 的位置，用于还原
f88492c0  00000000 00000000 00000000 00000000
f88492d0  00000000 00000000 00000000 00000000
f88492e0  00000000 00000000 00000000 00000000
f88492f0  00000000 00000000 00000000 00000000

[f8844000+5000+290]处的数据的意义
kd> dd 8206d958
8206d958  8211c7a8 f830491f f8845700 f830f0e6
8206d968  00000000 82136ec0 0a060004 20646156
kd> db 8211c7a8 //offset=0 处是一个指针，指向加密后的函数名字符串(或者其它编码？)
8211c7a8  44 44 43 38 43 36 42 36-43 45 42 46 44 45 46 44  DDC8C6B6CEBFDEFD  //该字符串总和 NtfsCreateFcb 相对应
8211c7b8  38 36 34 45 33 36 44 41-32 44 38 36 33 33 44 33  864E36DA2D8633D3
kd> u f830491f //offset=4 处是一个地址，指向 NtfsCreateFcb
Ntfs!NtfsCreateFcb:
f830491f 6a60             push    60h
f8304921 68c03f2ff8       push    offset Ntfs!\`string'+0x1cc (f82f3fc0)
f8304926 e8f559fdff       call    Ntfs!\_SEH\_prolog (f82da320)
kd> u f8845700 //offset=8 处是一个地址，指向 Fake 函数的地址
HijackFile+0x1700:
f8845700 8bff             mov     edi,edi
f8845702 55               push    ebp
f8845703 8bec             mov     ebp,esp
f8845705 83ec28           sub     esp,28h
kd> u f830f0e6 //offset=0xC 处是一个地址，指向 Hook 点的位置
Ntfs!NtfsOpenFile+0xee:
f830f0e6 e83458ffff       call    Ntfs!NtfsCreateFcb (f830491f)
f830f0eb 8bd8             mov     ebx,eax
f830f0ed 895db8           mov     dword ptr [ebp-48h],ebx
//offset=0x10 处是一个标志，表明是否被 Hook 过
//offset=0x14 处是一个指向同样结构的指针
```

从数据段里能分析出不少东西来~

尽管代码被 VM 过，但是数据没有，数据段前面是一个反汇编用的表，后面才是有用的数据。看来要 WS 的话还是连数据也加密了吧，用的时候再解。

### 方法二：劫持文件系统

因为知道盟主有一个山寨的 `ntfs.sys` 可以用，所以我就想劫持文件系统，不让相关调用走系统的 `ntfs.sys` 而走我们这个，这样原来的 Hook 就失效了。

不过本来文件系统这块就是我的弱项，很多东西还不熟悉，所以在如何挂载新 `ntfs` 时搞不下去了，后来偶然跟某同学聊天时得来灵感，于是演变成了这种方法，仍然是重加载 `ntfs.sys`，不过这次是加载系统原版的 `ntfs.sys`，作好重定位之后，直接替换 `ntfs` 的 `DriverObject` 中的所有 `MajorFuncion`.

这样，所有发往 `ntfs` 的调用都不再走原来的 `ntfs.sys`，而走我们重新加载的这个，可以无视 `ntfs` 内的任何 `Code Hook`，可以直接在资源管理器中右键删除，当然编程的话用 `SHFileOperation` 就行了.

详情见代码吧，其实主要就是两个函数，一个 `LoadPE`, 一个 `HookDispatchRoutine` 而已～

我实现上前面两种方法，如果宏观来看的话，也可以算是一种方法吧，代码见这里：

### 方法三：直接磁盘操作法（YY，未实现）

直接解析 `NTFS` 完成删除文件操作，然后刷新缓存。这个可以自己来或者借用一些成熟的 `ntfs` 代码来搞，比如 `ntfs-3g`，盟主似乎用这种方法。这个要搞得 `chkdsk` 检查没有问题还是挺麻烦啊。

### 方法四：底层调用（YY，未实现）

直接调用 `ntfs` 中的底层函数 `NtfsDeleteFile` 或发 `IRP` 到 `atapi`. 这个似乎也很麻烦，因为需要自己搞的东西太多了，所以只能 YY 一下。

## 遗留问题

还有一个问题不知什么原因：

在我绕过 Hook 后，本来就应该没有什么阻碍了，文件也可以打开，但是如果保护成功后打开过 `360safe.txt`，用 `DeleteFile`(其实就是 `ZwSetInformationFile`) 删除文件就会失败（函数调用成功，但是并没有立刻消失，处于 `DELETE_PENDING` 状态），如果保护成功后没有打开过 `360safe.txt` 就可以删除成功并立刻消失。也是因为这个原因我的答案才一直推迟没交，因为确实不完美。后来发现不管打没打开过，直接右键(也就是 `SHFileOperation,FO_DELETE`) 都可以删除，这个问题困扰我很久，原因未知(莫非和 `CleanCount` 之类的计数有关？)，路过的牛人指点一下。

水平有限，胡扯这么多，文件系统的知识还得恶补啊~~~

## 附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 第 5 题答案.rar  
|
