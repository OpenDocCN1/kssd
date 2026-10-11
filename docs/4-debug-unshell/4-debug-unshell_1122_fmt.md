
# 脱壳新版教程之二 upx

> 原文：[`www.pediy.com/kssd/pediy11/118478.html`](https://www.pediy.com/kssd/pediy11/118478.html)

一路走过之后发现路程很遥远，有时候也会觉得很迷茫。

但看到前辈们不辞辛饶发布教程以及心得，所以我也愿意把这举步维艰的路程走完。

希望你们在学习时候给自己定下目标 然后思考如何实现 最后则是准备用多长的时间去实现，![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

在开始脱壳之前了解下调试器，我们通常可以看到之前所找的资料中都有谈到 `SoftICE` 调试器和 `TRW2000`(only for Win 9x/Me)，不过这两个调试现在已经渐渐退出了自己的舞台，毕竟 2010 年了很少看到有人用 `window 98,window2000`。我们看到最多的则是 `xp win7 vista`

不过建议还是在 `XP` 下使用 `OD` 调试的好 （http://pediy.com/tools/Debuggers.htm） `OllyDbg[2008.1.1]`

看雪上贴出的这个版本 还是看起来很友好的，不过建议标配三个左右 `OD` 版本（其中也应该有一个英文版原因在后面的课程中讲解）

然后我们我们进入到 `OD` 的根目录，创建一个新的 `OD` 快捷方式 ，把它放到 `C:\Documents and Settings\Administrator\SendTo` ，之后就可以看到我们对目标程序反编译，就比较方便了 具体 `OD` 方面的内容你可以（http://bbs.pediy.com/showthread.php?s=&threadid=21284）

这里有 `vc6.0++ MFC` 生成的一个简单的程序，

并且用 `UPX` 加壳工具试着 `Add file` 然后点击 `compress` ，压缩之后我们对比加壳前的大小 `44.0 K`，加壳后的大小 `30.0 KB` 然后运行，程序照常加壳之后我们用 `peid` 载入

（或者用 `Fi`）就可以查壳，像一些大型的软件也有些会用到 `upx` （比如魔兽争霸）。

其实压缩壳的目的很简单，就是对目标程序进行进一步的优化，改变文件大小，提高程序运行速度。

另一方面来讲，如果你需要破解的软件是压缩壳，那么就只能恭喜你了，至少脱壳的部分你可以少走很多路，

总的来说在破解分析过程难度远小于脱壳的难度。（当然到后面的教程中会讲解到修复，以及具体的破解方法。）

这里贴出 `OD` 帮助中快捷键内容：

快捷键：

`F3`，装入程序

`F8`，单步执行，不进入 `call`

`F9` (运行)

`F7`,单步执行，进入 `call`

`CTRL+F9`,相当于 `trw2000` 的 `F12`

`ALT+F9`,相关于 `trw2000` 的 `pmodule`

`F2`,设置断点（相当于 `trw2000` 的 `F9`）

`CTRL+N`(当前模块中的名称)

`F12`(暂定)

`CTRL+F12`(重新运行)

`CTRL+F11`(跟踪进入)

`CTRL+T`(设置条件)

`CTRL+F7`(自动步入)

`CTRL+F8`(自动步过)

`CTRL+F9`(执行到返回)

用 `OD` 加载目标程序 `test.exe` 后

反汇编窗口显示：

```
0040C4B0 > 60 pushad
0040C4B1 BE 00604000 mov esi,test.00406000
0040C4B6 8DBE 00B0FFFF lea edi,dword ptr ds:[esi+FFFFB000]
0040C4BC 57 push edi
0040C4BD 83CD FF or ebp,FFFFFFFF
0040C4C0 EB 10 jmp short test.0040C4D2
0040C4C2 90 nop
0040C4C3 90 nop
0040C4C4 90 nop
0040C4C5 90 nop
0040C4C6 90 nop
0040C4C7 90 nop
0040C4C8 8A06 mov al,byte ptr ds:[esi]
0040C4CA 46 inc esi
0040C4CB 8807 mov byte ptr ds:[edi],al
0040C4CD 47 inc edi
0040C4CE 01DB add ebx,ebx
0040C4D0 75 07 jnz short test.0040C4D9
0040C4D2 8B1E mov ebx,dword ptr ds:[esi]
```

`pushad` 是一个入栈标志位，简单理解就是把数据压入堆栈当中，而相对应的则是 `popad` 出栈标志位。

现在所做的内容 就是把压缩过的程序一步一步的解压，可以理解为在程序外部包裹着一层保护膜，去除保护膜后就能到达源程序的入口点,也就是人们常说的 `OEP` ，所以带壳的程序也有一个入口点（通常叫做 `EP`），在本程序中的 `EP` 就是虚拟地址就是 `0040C4B0` （相对虚拟地址（`RVA`）=虚拟地址（`VA`）-基址（`ImageBase`））

然后我们用快捷键 `F8` 单步走，如果有指向上面的跳转就用 `F4`（运行到选定位置）

```
0040C4C7 90 nop
0040C4C8 8A06 mov al,byte ptr ds:[esi]
0040C4CA 46 inc esi
0040C4CB 8807 mov byte ptr ds:[edi],al
0040C4CD 47 inc edi
0040C4CE 01DB add ebx,ebx
0040C4D0 75 07 jnz short test.0040C4D9
0040C4D2 8B1E mov ebx,dword ptr ds:[esi]
0040C4D4 83EE FC sub esi,-4
0040C4D7 11DB adc ebx,ebx
0040C4D9 ^ 72 ED jb short test.0040C4C8
0040C4DB B8 01000000 mov eax,1
```

`jb short test.0040C4C8` 跳转指向 `0040C4C8`

如果用 `F8` 单步走，就会到 `0040C4C8 8A06 mov al,byte ptr ds:[esi]`

所以应该用 `F4`（运行到选定位置）到达 `0040C4DB B8 01000000 mov eax,1`

下面有遇到此种问题，处理是一样的。

```
0040C5D2 FF96 54C90000 call dword ptr ds:[esi+C954] ; kernel32.LoadLibraryA
0040C5D8 95 xchg eax,ebp
0040C5D9 8A07 mov al,byte ptr ds:[edi]
0040C5DB 47 inc edi
0040C5DC 08C0 or al,al
0040C5DE ^ 74 DC je short test.0040C5BC
0040C5E0 89F9 mov ecx,edi
0040C5E2 79 07 jns short test.0040C5EB
0040C5E4 0FB707 movzx eax,word ptr ds:[edi]
0040C5E7 47 inc edi
```

到这里之后我们遇到程序的第一个 `CALL` （`CALL` 就是调用子程序，可以理解为 `C` 语言中的函数概念，这里的调用的有可能是程序自己的子函数，而另一种可能就是系统 `API`，比如 `GetModuleHandleA` ,`VirtualAlloc`，`LoadLibraryA`

等）

当然也有可能是变形的 `jmp`，我们可以看到 `OD` 反汇编窗口注释（英文版本的是 `comment`）; `kernel32.LoadLibraryA` 所以这个 `CALL` 调用的是一个系统的 `API`

如果用 `F7` 跟进去的话 下一句就是：

```
7C801D77 kernel32.LoadLibraryA 8BFF mov edi,edi ; test.0040B008
7C801D79 55 push ebp
7C801D7A 8BEC mov ebp,esp
7C801D7C 837D 08 00 cmp dword ptr ss:[ebp+8],0
7C801D80 53 push ebx
7C801D81 56 push esi
7C801D82 74 14 je short kernel32.7C801D98
7C801D84 68 C0E0807C push kernel32.7C80E0C0 ; ASCII "twain_32.dll"
7C801D89 FF75 08 push dword ptr ss:[ebp+8]
7C801D8C FF15 9C13807C call dword ptr ds:[<&ntdll._strcmpi>] ; ntdll._stricmp
7C801D92 85C0 test eax,eax
7C801D94 59 pop ecx
```

我们很清除的看到地址（`address`）这行马上从 `0040C5D2` 转到 `7C801D77` ，所以以后大家在调试过程中在程序正常跟进时突然就跳转到其他区段（显示内存窗口 `(Alt+M)` 可以看到一些区段内容）上去了，这里

就涉及到领空的问题 `0040C5D2` 所在地址就是程序领空，而 `7C801D77` 是系统 `LoadLibraryA` API 所以就是系统领空，在后续的课程中则会遇到其他的加密壳中 `seh`（结构化异常处理 ），也通常的说成暗桩。

当然如果你进入了系统领空就只能慢慢的走出来，而如果只是一个 `API` 的话 可以用快捷建（`ALT+F9`）直接返回。

```
0040C5D8 95 xchg eax,ebp
```

这句就是返回的结果

```
0040C5FF ^ EB D8 jmp short test.0040C5D9
0040C601 FF96 68C90000 call dword ptr ds:[esi+C968]
0040C607 8BAE 5CC90000 mov ebp,dword ptr ds:[esi+C95C]
0040C60D 8DBE 00F0FFFF lea edi,dword ptr ds:[esi-1000]
0040C613 BB 00100000 mov ebx,1000
0040C618 50 push eax
0040C619 54 push esp
0040C61A 6A 04 push 4
0040C61C 53 push ebx
0040C61D 57 push edi
0040C61E FFD5 call ebp
0040C620 8D87 F7010000 lea eax,dword ptr ds:[edi+1F7]
```

走到这里我们可以看到 `jmp`(无条件跳转）指向 上面我们用 `F8` 跟过后就会一直的做循环，但如果我们用 `F4`（运行到选定位置） 到 `0040C601 FF96 68C90000 call dword ptr ds:[esi+C968]`

程序就直接跑飞（程序运行）， 那应该怎么办呢？ 没办法我们试试往下一行，也就是 `0040C607 8BAE 5CC90000 mov ebp,dword ptr ds:[esi+C95C]` `F4`（运行到选定位置）

呵呵果然可以 ，

然后一直，走到

```
0040C644 - E9 B789FFFF jmp test.00405000
```

后在再次一步 `F8`

```
00405000 E8 00000000 call test.00405005
00405005 5B pop ebx
00405006 81EB 05024000 sub ebx,test.00400205
0040500C 64:8B3D 3000000> mov edi,dword ptr fs:[30]
00405013 8B7F 0C mov edi,dword ptr ds:[edi+C]
00405016 8B7F 1C mov edi,dword ptr ds:[edi+1C]
00405019 8B3F mov edi,dword ptr ds:[edi]
0040501B 8B7F 08 mov edi,dword ptr ds:[edi+8]
0040501E 89BB C2034000 mov dword ptr ds:[ebx+4003C2],edi
00405024 8BF7 mov esi,edi
```

很遗憾的告诉你这个不是 `OEP` ，看样子这个版本的 `UPX` 有的不同不过并不影响 ，那我们只能继续走了，

看到 `call test.00405005` 这是就是一个变形的 `jmp` 它会跳转到 `00405005` ，不过这句你 `F8`，`F7` 都不影响最后结果 但为了有一个好习惯 ，还是建议你用 `F7`（前辈的经验变形 `CALL` 尽量用 `F7`）

继续 `F8` 走 会经过



```
004050A3       FFD7            call edi                                 ; kernel32.GetProcAddress  
004050B1       FFD0            call eax                                 ; kernel32.GetTempPathA  
004050C1       FFD7            call edi                                 ; kernel32.GetProcAddress  
004050E0       FFD7            call edi                                 ; kernel32.GetProcAddress  
004050F7       FFD0            call eax                                 ; kernel32.CreateFileA  
0040510A       FFD7            call edi                                 ; kernel32.GetProcAddress  
00405153       8932            mov dword ptr ds:[edx],esi  
00405155       83C2 04         add edx,4  
00405158     ^ E2 F9           loopd short test.00405153      \loopd 循环语句  
0040515A       BA 44000000     mov edx,44  
0040515F       891424          mov dword ptr ss:[esp],edx  
00405162       83EC 10         sub esp,10  
00405165       8BD4            mov edx,esp  
0040516C       6A 00           push 0  
0040516E       6A 00           push 0  
00405170       6A 00           push 0  
00405172       6A 00           push 0  
00405174       6A 00           push 0  
00405176       6A 00           push 0  
00405178       6A 00           push 0  
0040517A       83C2 44         add edx,44  
0040517D       52              push edx  
0040517E       FFD0            call eax  
00405180       8BE5            mov esp,ebp  
00405182     ^ E9 59C5FFFF     jmp test.004016E0  
```

到了这之后我们看到一点曙光然后 `jmp` 跟过 `F8` 来到

```
004016E0       55              push ebp  
004016E1       8BEC            mov ebp,esp  
004016E3       6A FF           push -1  
004016E5       68 F8244000     push test.004024F8  
004016EA       68 66184000     push test.00401866                     ; jmp 到  
004016EF       64:A1 00000000  mov eax,dword ptr fs:[0]  
004016F5       50              push eax  
004016F6       64:8925 000000> mov dword ptr fs:[0],esp  
004016FD       83EC 68         sub esp,68  
00401700       53              push ebx  
00401701       56              push esi  
00401702       57              push edi  
00401703       8965 E8         mov dword ptr ss:[ebp-18],esp  
00401706       33DB            xor ebx,ebx  
00401708       895D FC         mov dword ptr ss:[ebp-4],ebx  
0040170B       6A 02           push 2  
0040170D       FF15 90214000  call dword ptr ds:[402190]             ; msvcrt.__set_app_type  
```

告诉你很辛苦的你来到了，OEP（程序入口点）

note: 不能段在 `nop`（空数据）上 要不程序也会跑飞。

好了，今天就这么多了。

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `Free_UPX.zip`  

|

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `test.rar`  

|
