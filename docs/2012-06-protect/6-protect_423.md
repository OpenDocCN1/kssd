
# 一个小花招 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-534.htm`](https://www.pediy.com/kssd/pediy08/pediy8-534.htm)

简单却有用，`themida` 的小花招，单击下载附件。

将 API 的代码复制到新申请到的内存空间去执行，再下断点当然无效啦。

```
代码:

00A2CCDE    8BFF            mov     edi, edi
00A2CCE0    55              push    ebp
00A2CCE1    8BEC            mov     ebp, esp
00A2CCE3    833D 1821E777 0 cmp     dword ptr [77E72118], 0
00A2CCEA    74 24           je      short 00A2CD10
00A2CCEC    64:A1 18000000  mov     eax, fs:[18]
00A2CCF2    6A 00           push    0
00A2CCF4    FF70 24         push    dword ptr [eax+24]
00A2CCF7    68 442FE777     push    77E72F44
00A2CCFC    FF15 D012E177   call    [<KERNEL32.InterlockedCompareEx>; kernel32.InterlockedCompareExchange
00A2CD02    85C0            test    eax, eax
00A2CD04    75 0A           jnz     short 00A2CD10
00A2CD06    C705 402FE777 0 mov     dword ptr [77E72F40], 1
00A2CD10    6A 00           push    0
00A2CD12    FF75 14         push    dword ptr [ebp+14]
00A2CD15    FF75 10         push    dword ptr [ebp+10]
00A2CD18    FF75 0C         push    dword ptr [ebp+C]
00A2CD1B    FF75 08         push    dword ptr [ebp+8]
00A2CD1E    E8 4D040000     call    00A2D170
00A2CD23    5D              pop     ebp
00A2CD24    C2 1000         retn    10
```

恩，先取 `MessageBoxA` 的代码，然后放到其他内存地址然后去 `CALL`。

恩，我的方法是：

1. 找到他申请空间的地方然后执行访问中断。
2. 是将 `MessageBoxA` 代码复制部分，然后再他执行到窗口出来的时候，进行下代码查找，就可以知道他的代码复制到哪个地方，然后下硬件执行，关闭再装载就可以断下来了。

**问题：**

这么说 `user32` 应该不会被重定位吧？

**菜鸟教学：**

1. 根据 dll 大小分配内存，把 dll 读入。
2. 获得 api 的地址，根据 dll 块表找到 `physoffset`。
3. 调用 `空间地址+physoffset`。
4. 释放内存。

`forgot` 终于浮出来了![](img/d96007cf245c7669301e5df8beb41bf5.jpg) 。

`themida` 在查找函数地址的时候用的是 `gzgzlxg` 文章介绍的

方式。我用 `GetProcAddress` 只是图省事。

我想这个玩艺能工作是用了绑定输入

引用: *最初由 Lenus 发布*

请问一下...`q3` 兄用的是什么工具得到上面的数据？![](img/de82a00003a7d4a29fabed7b826fae7a.jpg)

感觉这样 `GetFileSize` 不是很保险...访问数据段，重定位等等都存在问题吧！

数据段指向真正的 `user32.dll`，

如果有绝对地址都会访问真的 dll

相对地址比如 `jmp`/`call` 因为都是固定偏移，也不存在重定位的问题。

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

to `softworm`

这招真的不错啊，用来做 `krnl`/`usr`/`gdi`/`adv` 等系统 dll 的 IAT 加密还能防断点。

windows 肯定都有 bound imports

甚至可以在写 `myGetProcAddr` 的时候判断一下 timestamp，把 bound 的都用这种方式，只是浪费一点点内存:)

运行无 key 的 `Themida` 会跳出如下的对话框：

![](img/133777f3b7605db21d36872b9877ce2b.jpg)

图一

这个对话框是调用 `MessageBox` 函数显示的，但你用 `MessageBox` 下断是拦不住的，因为外壳将 `MessageBox` 函数代码搬到自己的地址空间执行了。

感兴趣的可以看看这个实例：http://bbs.pediy.com/showthread.php?s=&threadid=23721

**对策：**

在这用 `Themida1.3.3.0` 为例演示一下。

先用 `OD` 加载 `Themida`，运行，直到出现图一那个对话框。这里最终调用 `MessageBoxExW` 显示对话框的，在 `OD` 里查看 `MessageBoxExW` 函数：

```
77D50538  USER32.MessageBoxExW   8BFF            mov     edi, edi
77D5053A                                 55              push    ebp
77D5053B                                 8BEC            mov     ebp, esp
77D5053D                                 6A FF           push    -1
77D5053F                                 FF75 18         push    dword ptr [ebp+18]
77D50542                                 FF75 14         push    dword ptr [ebp+14]
77D50545                                 FF75 10         push    dword ptr [ebp+10]
77D50548                                 FF75 0C         push    dword ptr [ebp+C]
77D5054B                                 FF75 08         push    dword ptr [ebp+8]
77D5054E                                 E8 EE590100     call    MessageBoxTimeoutW
77D50553                                 5D              pop     ebp
77D50554                                 C2 1400         retn    14
```

用二进制复制，将 `MessageBoxExW` 开始一段的机器码复制出来：`8B FF 55 8B EC 6A FF FF 75 18 FF 75 14 FF 75 10 FF 75 0C FF 75 08`

在 `OD` 里按 `Alt+M` 打开内存窗口，查找刚才复制出来的机器码：

![](img/236d5b3d919cda1dc37242a4439b0d51.jpg)

找到的数据：

```
013FF938  8B FF 55 8B EC 6A FF FF 75 18 FF 75 14 FF 75 10  ?U 嬱 j??u??u??u?
013FF948  FF 75 0C FF 75 08 E8 EE 59 01 00 5D C2 14 00 90  ?u.?u ? Y.\]?..
```

然后在反汇编窗口跳到 `013FF938` 代码处：

```
013FF938                 8BFF            mov     edi, edi
013FF93A                 55              push    ebp
013FF93B                 8BEC            mov     ebp, esp
013FF93D                 6A FF           push    -1
013FF93F                 FF75 18         push    dword ptr [ebp+18]
013FF942                 FF75 14         push    dword ptr [ebp+14]
013FF945                 FF75 10         push    dword ptr [ebp+10]
013FF948                 FF75 0C         push    dword ptr [ebp+C]
013FF94B                 FF75 08         push    dword ptr [ebp+8]
013FF94E                 E8 EE590100     call    01415341
013FF953                 5D              pop     ebp
013FF954                 C2 1400         retn    14
013FF957                 90              nop
013FF958                 90              nop
013FF959                 90              nop
013FF95A                 90              nop
013FF95B                 90              nop
013FF95C                 8BFF            mov     edi, edi
013FF95E                 55              push    ebp
013FF95F                 8BEC            mov     ebp, esp
013FF961                 6A FF           push    -1
013FF963                 FF75 18         push    dword ptr [ebp+18]
013FF966                 FF75 14         push    dword ptr [ebp+14]
013FF969                 FF75 10         push    dword ptr [ebp+10]
013FF96C                 FF75 0C         push    dword ptr [ebp+C]
013FF96F                 FF75 08         push    dword ptr [ebp+8]
013FF972                 E8 4D5A0100     call    014153C4                 //这里显示图 1 的窗口
013FF977                 5D              pop     ebp                        //这里下断就可断下
013FF978                 C2 1400         retn    14
```

在 `013FF977` 处下断，单击图一对话框的 `OK` 后，就能中断。

> `4dd466d0e1e0cc123afbe504da2462ca`
