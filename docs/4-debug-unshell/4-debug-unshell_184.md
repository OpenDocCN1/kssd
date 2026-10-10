

# Patch 修复 Armadillo 的 IAT 乱序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-696.htm`](https://www.pediy.com/kssd/pediy07/pediy7-696.htm)

文章标题: Patch 修复 Armadillo 的 IAT 乱序  
目标：Fraps v2.7.2  
声明：纯属无聊  
作者：无聊的菜鸟  
时间：2005 年 12 月 7 日  17:17  
  
近日遇见一 Armadillo 的 IAT 乱序。上看雪搜索教程，看到 Fly 大大的用 ArmInline 修复，还看到一个是写代码重排 IAT。不过我太菜了，没看懂。。。。  
  
我想如果 Arm 要乱序的话，那他一定要先有正确序列的 IAT。  
  
隐藏 OD，并且忽略一切异常。  
  
我们在程序运行后中断，然后对第二区段下访问中断，断下后，我们找一个 IAT 的调用：  
00401A2B               FF15 A0327501            call dword ptr ds:\[17532A0\]  
（我们顺变记一下从 0401000 开始的第一个 IAT 调用地址 00401005）  
我们对 17532A0 下硬件写入断点，然后重来。运行后，我们断在了这里：  
01615552               890C82                   mov dword ptr ds:\[edx+eax\*4\],ecx  
01615555             ^ E9 24FFFFFF              jmp 0161547E  
  
这是我们看看内存窗口：  
01753280  016065DF  遝\`  
01753284  016065E6  鎒\`  
01753288  01606654  Tf\`  
0175328C  016065AC  琫\`  
01753290  016065C1  羍\`  
01753294  016064F4  鬱\`  
01753298  016065E4  鋏\`  
0175329C  016066EB  雈\`  
017532A0  016066BA  篺\`  
017532A4  00000000  ....  
017532A8  00000000  ....  
017532AC  00000000  ....  
  
像什么？是不是很想准备填充 IAT？我们继续运行，断在了这里：  
01616A1C               8B85 14DBFFFF            mov eax,dword ptr ss:\[ebp-24EC\]  
01616A22               83C0 04                  add eax,4  
01616A25               8985 14DBFFFF            mov dword ptr ss:\[ebp-24EC\],eax  
  
再看看内存窗口：  
01753280  77D2DF46  F 咭 w   USER32.SetWindowPlacement  
01753284  77D2FEC2  漫襴    USER32.MapVirtualKeyA  
01753288  77D202DD  ?襴    USER32.SetWindowRgn  
0175328C  77D1BDC8  冉褀    USER32.ScreenToClient  
01753290  77D1B5F5  醯褀    USER32.InvalidateRect  
01753294  77D1B6D4  远褀    USER32.GetWindowRect  
01753298  77D1DBEC  燠褀    USER32.MoveWindow  
0175329C  77D1C01B  姥 w   USER32.SetWindowPos  
017532A0  016065D1  裡\`  
017532A4  016065CA  蔱\`  
017532A8  01606625  %f\`  
017532AC  016066BE  緁\`  
  
我们向下慢慢走：  
01616C7A             ^\\E9 C7F6FFFF              jmp 01616346  
  
到了这里程序开始向上跳了，我们猜测，程序已经完成了对 User32.dll 中需要的函数的处理。所以我们也向上跳去找处理的起始地点，找到了这里：  
01616346               6A 01                    push 1  
01616348               58                       pop eax  
01616349               85C0                     test eax,eax  
0161634B               0F84 2E090000            je 01616C7F  
  
继续向下走：  
016163DB               FFB5 38D6FFFF            push dword ptr ss:\[ebp-29C8\]    //GDI32.dll  
016163E1               E8 2202FFFF              call 01606608   //相当于 GetModuleHandleA  
016163E6               8985 54D7FFFF            mov dword ptr ss:\[ebp-28AC\],eax  
....  
016167DD               FF15 84E26101            call dword ptr ds:\[161E284\]                  ; kernel32.GetTickCount   //又看见了这个玩意,估计快到了。  
...  
01616A61               FFB5 14ABFFFF            push dword ptr ss:\[ebp+FFFFAB14\]  
01616A67               FFB5 54D7FFFF            push dword ptr ss:\[ebp-28AC\]  
01616A6D               E8 EBEEFEFF              call 0160595D  //相当于 GetProcAddress  
...  
比较加密与未加密的运行流程之后发现，关键跳转在这里：  
01616A30               83BD 1CCDFFFF 00         cmp dword ptr ss:\[ebp-32E4\],0  //判断加密后地址是否存在  
01616A37               75 42                    jnz short 01616A7B  //不跳的话就不加密。  
  
我们在这里下个硬件执行断点，然后重来：  
01616A30               83BD 1CCDFFFF 00         cmp dword ptr ss:\[ebp-32E4\],0  
01616A37               75 42                    jnz short 01616A7B  //nop  
  
然后这里：  
01616B97             ^\\E9 4DFCFFFF              jmp 016167E9  
01616B9C               FF15 84E26101            call dword ptr ds:\[161E284\]                  ; kernel32.GetTickCount  
01616BA2               2B85 58D7FFFF            sub eax,dword ptr ss:\[ebp-28A8\]  
01616BA8               8B8D 5CD7FFFF            mov ecx,dword ptr ss:\[ebp-28A4\]  
01616BAE               6BC9 32                  imul ecx,ecx,32  
01616BB1               81C1 D0070000            add ecx,7D0  
01616BB7               3BC1                     cmp eax,ecx  
01616BB9               76 28                    jbe short 01616BE3  //这种东西怎么能让他不跳呢？修改成 EB 28  
  
01616C7A             ^\\E9 C7F6FFFF              jmp 01616346  
01616C7F               8B85 FCD9FFFF            mov eax,dword ptr ss:\[ebp-2604\]  
01616C85               8985 B4AEFFFF            mov dword ptr ss:\[ebp+FFFFAEB4\],eax  
01616C8B               FFB5 B4AEFFFF            push dword ptr ss:\[ebp+FFFFAEB4\]  
01616C91               E8 F0620000              call 0161CF86                                ; jmp to msvcrt.operator delete  
看情况来说，到 1616C7F 的时候事情该做的都应该完成了。F4 到那里。  
现在我们看看对应的数据区，完美的 IAT。我们把这里记为关键点 1  
  
现在我们来思考一下，下面他肯定要乱序，乱序后一定会写入乱序后的地址。而且不出意外的话肯定是从代码段的起始开始写。那么我们对 0401007 下硬件写入断点。  
016174B4               8908                     mov dword ptr ds:\[eax\],ecx  
016174B6             ^ E9 4DFFFFFF              jmp 01617408  
我们把这里记为关键点 2。  
  
好现在我们就来 Patch：  
关键点 1：  
nop 01616A37 那里的跳转，EB 01616BB9 那里的跳转。  
01616C9E               60                       pushad  //保护现场  
01616C9F               9C                       pushfd  
01616CA0               B8 00005000              mov eax,500000  //新的 IAT 地址，这里大家要看实际情况，只要程序运行中用不到的地方都行。  
01616CA5               BB 08307501              mov ebx,1753008  //现在 IAT 所处的位置  
01616CAA               8B0B                     mov ecx,dword ptr ds:\[ebx\]  
01616CAC               8908                     mov dword ptr ds:\[eax\],ecx  //搬家  
01616CAE               83C0 04                  add eax,4  
01616CB1               83C3 04                  add ebx,4  
01616CB4               81FB F8327501            cmp ebx,17532F8 //现在的 IAT 结束地址  
01616CBA             ^ 72 EE                    jb short 01616CAA  
01616CBC               9D                       popfd  
01616CBD               61                       popad  
01616CBE             ^ EB DE                    jmp short 01616C9E  //这里跳上去之后我们使用 OD 的“撤销选择”功能恢复原有代码继续运行。  
  
关键点 2：  
016174B4               8908                     mov dword ptr ds:\[eax\],ecx  
016174B6               E9 05000000              jmp 016174C0  
016174BB               A0 348F6201              mov al,byte ptr ds:\[1628F34\]  //我们在这里 F4  
016174C0               60                       pushad  //保护现场  
016174C1               9C                       pushfd  
016174C2               BB 00005000              mov ebx,500000  //IAT 搬家后的地址  
016174C7               8B09                     mov ecx,dword ptr ds:\[ecx\]  //去乱序后的 API 存储位置中的 API  
016174C9               390B                     cmp dword ptr ds:\[ebx\],ecx  //定位我们需要的储存位置  
016174CB               74 05                    je short 016174D2  
016174CD               83C3 04                  add ebx,4  
016174D0             ^ EB F7                    jmp short 016174C9  
016174D2               8918                     mov dword ptr ds:\[eax\],ebx  //写入  
016174D4               9D                       popfd  
016174D5               61                       popad  
016174D6             ^ E9 2DFFFFFF              jmp 01617408  //回去继续  
  
在 016174BB 断下后同样撤销选择，然后我们再到 OEP 的时候，看见的 IAT 就比较美好了。  
  
不过 Fraps 还有代码搬迁，所以这里我只是对付了一下 IAT 乱序。 

 4dd466d0e1e0cc123afbe504da2462ca