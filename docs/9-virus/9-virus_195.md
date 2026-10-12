
# 昨日的病毒简单分析【附 Bin 样本】

> 原文：[`www.pediy.com/kssd/pediy12/132249.html`](https://www.pediy.com/kssd/pediy12/132249.html)

由于里面的文件比较多，我只大概分析了一下，错误之处大家分析后指出来！

我也不知道怎么感染这病毒的，幸好我电脑没联网，要不然！里面涉及技术比较多。

大家可以慢慢挖出来！注意：样本是我电脑上直接拷贝过来的！不知道是不是可以感染。

正题开始！

病毒名字好像叫：**红蜘蛛**（是拼音）。

## 病毒的工作模式

向`spoolsv.exe`注入 4 个 DLL，这 4 个 DLL 里面我只看了两个，它们都是分工明确：
- `WinIO`负责安装驱动和服务以及卸载驱动；
- 其他 3 个都负责 HOOK 消息和在 C 盘创建 2 文本文件，记录下我们的键盘操作；
- 里面的那个驱动负责干扰杀毒和杀毒辅助工具；
- 还有每个进程启动后就注入`ZkeyHOOK.Dll`。

这个里面昨天我看了下，好像是设置`WH_GETMESSAGE`和`WH_CALLWNDPROC` HOOK，然后向 C 盘的`Debug.Txt`写入键盘的操作。大家看下这是我用冰刃拷贝`debug.txt`里面复制出来的：

```
A：(Program Manager)exesys

(查找十六进制数值)49006f1.exe

(1.txt - 记事本)

ctfmon.exe 注入 zkeyhook 到进程，以便执行消息 HOOK

sstddttddt HOOK 这些函数

(OllyDbg - rellatsnI metsyS.dll - [CPU - 主线程, 模块 - rellatsn\])

--------------------------------------------------------------------------------
(Program Manager)ini1

(key.txt - 记事本)zzzxzxcz

(OllyDbg - WinIo.dll - [CPU - 主线程, 模块 - WinIo\])

--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
(新建 Microsoft Word 文档.doc - Microsoft Word)使用 CracketeateFile 创建服务通信符号和驱动，然后在 InstallInstallWinioIoDriver 调用初始化创建服务，InstallEWinIoDriver 里面删除服务 WinIo，先 iiii<<<《,Install,WinIoDriverL 里面先删除原来开的服务 WinIo 在创建服务。

(OllyDbg - hkey.dll - [CPU - 主线程, 模块 - hkey\])
```

具体的大家自己看样本文件包里面，样本我估计上面还留言了个网站还有每个进程驱动启动后就注入`ZkeyHOOK.dll`。这个里面昨天我看了下，好像是设置 HOOK 和和 HOOK，在然后向 C 盘的`DEebug.txt`写入操作。

`Dhook`这个 DLL 里面是对自己些那些`SYS`以及`DLL`以及`VXD`进行操作，好像里面有点复杂，具体自己分析。

`hKey`面是对 C 盘的`Key.txt`进行操作，在 C 盘创建`Key.Txt`，至于把在 C 盘创建的文件隐藏嘛，用了 RING3 下的`SetFileAttributes`函数，用法如下：

```
SetFileAttributes("C://key.TXT",FILE_ATTRIBUTE_HIDDEN);
```

还有 C 盘的`boot.Ini`也被替换了。每次重新启动 C 盘的文件重新记录！还有打开冰刃提示内核模块`ntrnlpa.exe`被修改了！

总之里面有好多好东西，大家需要什么可以从里面直接拿过来，随便在说几句：这病毒作者太狂了，一点保护技术都不用，这是病毒上面留下的网站：`http://www.internals.com`，大家可以把这个病毒慢慢分析哈！

## 汇编代码片段

```
00882B00  . /8E2B8800      DD hkey.00882B8E
00882B04  |> |8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]            ;  Case 30 ('0') of switch 00882AB5
00882B07  . |BA 282F8800   MOV EDX,hkey.00882F28                    ;  0
00882B0C  . |E8 CF0FFFFF    CALL hkey.00873AE0
00882B11  . |E9 85000000    JMP hkey.00882B9B
00882B16  |> |8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]            ;  Case 31 ('1') of switch 00882AB5
00882B19  . |BA 342F8800   MOV EDX,hkey.00882F34                    ;  1
00882B1E  . |E8 BD0FFFFF    CALL hkey.00873AE0
00882B23  . |EB 76          JMP SHORT hkey.00882B9B
00882B25  |> |8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]            ;  Case 32 ('2') of switch 00882AB5
00882B28  . |BA 402F8800   MOV EDX,hkey.00882F40                    ;  2
00882B2D  . |E8 AE0FFFFF    CALL hkey.00873AE0
00882B32  . |EB 67          JMP SHORT hkey.00882B9B
00882B34  |> |8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]            ;  Case 33 ('3') of switch 00882AB5
00882B37  . |BA 4C2F8800   MOV EDX,hkey.00882F4C                    ;  3
00882B3C  . |E8 9F0FFFFF    CALL hkey.00873AE0
00882B41  . |EB 58          JMP SHORT hkey.00882B9B
00882B43  |> |8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]            ;  Case 34 ('4') of switch 00882AB5
00882B46  . |BA 582F8800   MOV EDX,hkey.00882F58                    ;  4
00882B4B  . |E8 900FFFFF    CALL hkey.00873AE0
00882B50  . |EB 49          JMP SHORT hkey.00882B9B
```

## 专杀编写思路

判断`System32`里面是否有这几个文件，有的话证明中了这病毒！然后扫描进程模块，发现这几个 DLL 就强行释放，之后删除 C 盘里面的文件！再删除`System32`下的这几个文件！

当然这只是一个思路，因为我电脑没联网！所以没有必要编写专杀了！

## 手动杀毒

使用冰刃查看`spoolsv.Exe`进程里面的模块然后释放，之后删除`System32`下的文件，然后在打开`xuetr`，查看 HOOK，直接恢复。不全部用`xuetr`是因为`xuetr`被干扰了，冰刃打开不能关闭，关闭就蓝屏！

不知道他那个干扰`xuetr`怎么做到的，大家拿去慢慢分析哈，我能做的只有这么多了！等待牛人详细分析。

![](img/86601a542dd8439e2b28c8b35988bc25.jpg)

![](img/f744bd61d133b38ae34f5b7dcb278ca7.jpg)

## 上传的附件

|  |
|  ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `样本.rar`（解压密码：muma）  

 |
