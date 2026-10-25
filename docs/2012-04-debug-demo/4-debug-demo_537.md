
# 程序自校验与功能限制捆绑的双重去校验

> 原文：[`www.pediy.com/kssd/pediy10/101434.html`](https://www.pediy.com/kssd/pediy10/101434.html)

从网上下载了一个 DEMO 版的《KTV 音乐无限点歌台 V8.5》的点歌软件，酷爱之，故尝试对其限制功能进行解除，查壳为 `PECOMPACT 2.X`------基本上算最简单的压缩壳，立即脱壳、运行，发现程序能正常启动，就点歌功能不正常（出现 `Access violation at address 005D2052 in module 'Music_ktv_tk.exe、Query1: Cannot perform this operation on a closed dataset` 错误信息），最初以为是脱壳不干净，`IAT` 被加密之类的，后来到脱壳区以及 `UNPACK` 上面去求助，得到高人指点方知系程序自校验。立即着手去除程序的自校验，由于本人才学粗浅，先潜水学习之，从论坛上看到了密界高人 `laomms` 一篇文章《常见自校检分析实例》，受益非浅（在此感谢），随即参照里面的方法进行了尝试：

1、通用对比法-----跟踪了几遍，没发现蛛丝马迹，估计比较调用很深或很远，实在雷人，暂放弃！

2、跟踪退出函数------程序校验失败后并没有退出，所以不能应用该方法。

3、利用第三方软件辅助查找关键的地方------用 `PEiD` 辅助工具 `KANAL` 查看，发现程序内使用了 `MD5` 及 `DES` 多处加密，跟踪几次，还是太累，暂放弃！

4、对付校检自身大小的软件的一般方法------使用 `W32DASM` 反汇编后，未发现直接进行文件尺寸的比较，放弃！

5、对付自校验的杀手锏 -- 偷天换日-------修改程序代码，让它去验证原程序，无论如何都是 `OK` 的，这个方法虽说有效，但是并不利于破解程序的发布，毕竟玩家都不太喜欢程序被强迫安装到固定的目录下，而且还必须与破解前的程序“和平共处”。

首次调试程序时采用了“偷天换日”的方法来去除自校验，总觉得不够完美，后来调试程序的时候还发现一个怪事，程序里面的加密字符串也进行了自校验，一但修改就 `OVER`，郁闷之至！总结上一次追踪“校验比较”失败的经过，基本确定程序的自校验不属于加壳程序（`PECOMPACT`）所为，校验失败后也不退出程序，只是对程序进行出错处理(完全受作者的摆布)，猜测程序自校验与功能限制有着某种联系，好了让我们一起来看看去除双重校验的全过程：

还是从下断“`bp CreateFileA`”，开始，`F9` 运行，程序断在这里：

```
7C801A28  k>   8BFF           mov edi,edi                    //系统空间，F8 单步跟踪几步

7C801A2A      55             push ebp

7C801A2B      8BEC           mov ebp,esp

7C801A2D      FF75 08        push dword ptr ss:[ebp+8]

7C801A30      E8 CFC60000    call kernel32.7C80E104

7C801A35      85C0           test eax,eax

7C801A37      74 1E          je short kernel32.7C801A57

\========
```

`F8` 跟踪几步（或 `ALT+F9`）后返回程序空间:

```
005DCFCD      E8 3EA3E2FF    call <jmp.&kernel32.CreateFileA>        //这就是刚才读文件中断所在的系统调用

005DCFD2      8945 F4        mov dword ptr ss:[ebp-C],eax            //返回到这里

005DCFD5      837D F4 FF     cmp dword ptr ss:[ebp-C],-1

005DCFD9      0F84 D6000000  je Music_kt.005DD0B5

005DCFDF      33C0           xor eax,eax

005DCFE1      55             push ebp

005DCFE2      68 AED05D00    push Music_kt.005DD0AE

005DCFE7      64:FF30        push dword ptr fs:[eax]

005DCFEA      64:8920        mov dword ptr fs:[eax],esp

005DCFED      6A 00          push 0

005DCFEF      6A 00          push 0

005DCFF1      6A 00          push 0

005DCFF3      6A 02          push 2

005DCFF5      6A 00          push 0

005DCFF7      8B45 F4        mov eax,dword ptr ss:[ebp-C]

005DCFFA      50             push eax

005DCFFB      E8 18A3E2FF    call <jmp.&kernel32.CreateFileMappingA>   //读取文件在内存中映射，以便计算

005DD000      8945 F0        mov dword ptr ss:[ebp-10],eax

005DD003      837D F0 00     cmp dword ptr ss:[ebp-10],0

005DD007      0F84 8A000000  je Music_kt.005DD097

005DD00D      33C0           xor eax,eax

005DD00F      55             push ebp

005DD010      68 90D05D00    push Music_kt.005DD090

005DD015      64:FF30        push dword ptr fs:[eax]

005DD018      64:8920        mov dword ptr fs:[eax],esp

005DD01B      6A 00          push 0

005DD01D      6A 00          push 0

005DD01F      6A 00          push 0

005DD021      6A 04          push 4

005DD023      8B45 F0        mov eax,dword ptr ss:[ebp-10]

005DD026      50             push eax

005DD027      E8 4CA5E2FF    call <jmp.&kernel32.MapViewOfFile>       //查看文件映射

005DD02C      8945 EC        mov dword ptr ss:[ebp-14],eax

005DD02F      837D EC 00     cmp dword ptr ss:[ebp-14],0

005DD033      74 44          je short Music_kt.005DD079

005DD035      33C0           xor eax,eax

005DD037      55             push ebp

005DD038      68 72D05D00    push Music_kt.005DD072

005DD03D      64:FF30        push dword ptr fs:[eax]

005DD040      64:8920        mov dword ptr fs:[eax],esp

005DD043      6A 00          push 0

005DD045      8B45 F4        mov eax,dword ptr ss:[ebp-C]

005DD048      50             push eax

005DD049      E8 02A4E2FF    call <jmp.&kernel32.GetFileSize>         //取得文件尺寸

005DD04E      8BC8           mov ecx,eax

005DD050      8D45 94        lea eax,dword ptr ss:[ebp-6C]

005DD053      8B55 EC        mov edx,dword ptr ss:[ebp-14]

005DD056      E8 C5FDFFFF    call Music_kt.005DCE20

005DD05B      33C0           xor eax,eax

005DD05D      5A             pop edx

005DD05E      59             pop ecx

005DD05F      59             pop ecx

005DD060      64:8910        mov dword ptr fs:[eax],edx

005DD063      68 79D05D00    push Music_kt.005DD079

005DD068      8B45 EC        mov eax,dword ptr ss:[ebp-14]

005DD06B      50             push eax

005DD06C      E8 9FA5E2FF    call <jmp.&kernel32.UnmapViewOfFile>     //关闭文件映射

005DD071      C3             retn

\========
```

到这里程序已获取了文件的基本信息，并向上层逐步返回，一路跟踪来到这里：

```
005DD214      E8 6FFDFFFF    call Music_kt.005DCF88                   //这是刚才中断调用的 CALL

005DD219      8D45 EC        lea eax,dword ptr ss:[ebp-14]            //返回到这里

005DD21C      8BD3           mov edx,ebx

005DD21E      E8 C1FEFFFF    call Music_kt.005DD0E4                   //计算文件的 MD5 值

005DD223      33C0           xor eax,eax

\========
```

这时查看堆栈区：

```
0012FCBC  |00D82544  ASCII "C:\Program Files\Music_KTV_DJB\Music_ktv_tk-1.exe"     //程序名

0012FCC0  |00D826F0  ASCII "d5ced6890a02d58b280add2bf46eadbc"        //文件 MD5 加密后的代码

\========
```

算得 `MD5` 后继续向上层返回，来到这里：

```
005DEC42      E8 A5E5FFFF    call Music_kt.005DD1EC                   //这是刚才调用文件校验的主 CALL

005DEC47      8D85 BCFEFFFF  lea eax,dword ptr ss:[ebp-144]           //返回到这里

005DEC4D      50             push eax

\========
```

到这里发现程序有很长的一系列调用，向上看看程序做了些什么事，我们在 `005DEBD7` 处 `F2` 下断，重新加载程序，`F9`：

```
005DEBD7      50             push eax                                 //断在这里

005DEBD8      8D95 C4FEFFFF  lea edx,dword ptr ss:[ebp-13C]

005DEBDE      A1 B46C5F00    mov eax,dword ptr ds:[5F6CB4]

005DEBE3      8B00           mov eax,dword ptr ds:[eax]

005DEBE5      8B80 F4030000  mov eax,dword ptr ds:[eax+3F4]

005DEBEB      8B80 80000000  mov eax,dword ptr ds:[eax+80]

005DEBF1      E8 52ABE2FF    call Music_kt.00409748

005DEBF6      8B85 C4FEFFFF  mov eax,dword ptr ss:[ebp-13C]

005DEBFC      8D95 C8FEFFFF  lea edx,dword ptr ss:[ebp-138]

005DEC02      E8 89E5FFFF    call Music_kt.005DD190                  //对标识符“DEMO SOFTWARE”进行 MD5 加密

005DEC07      8B85 C8FEFFFF  mov eax,dword ptr ss:[ebp-138]

005DEC0D      B9 08000000    mov ecx,8

005DEC12      BA 08000000    mov edx,8

005DEC17      E8 A861E2FF    call Music_kt.00404DC4

005DEC1C      8D45 F4        lea eax,dword ptr ss:[ebp-C]

005DEC1F      E8 805CE2FF    call Music_kt.004048A4

005DEC24      8D95 B8FEFFFF  lea edx,dword ptr ss:[ebp-148]

005DEC2A      A1 5C715F00    mov eax,dword ptr ds:[5F715C]

005DEC2F      8B00           mov eax,dword ptr ds:[eax]

005DEC31      E8 8A8DEAFF    call Music_kt.004879C0

005DEC36      8B85 B8FEFFFF  mov eax,dword ptr ss:[ebp-148]

005DEC3C      8D95 BCFEFFFF  lea edx,dword ptr ss:[ebp-144]

005DEC42      E8 A5E5FFFF    call Music_kt.005DD1EC                  //调用文件校验的主 call,并取得 MD5 值，也是我们刚才跟到的地方

005DEC47      8D85 BCFEFFFF  lea eax,dword ptr ss:[ebp-144]

005DEC4D      50             push eax

005DEC4E      8D95 B0FEFFFF  lea edx,dword ptr ss:[ebp-150]

005DEC54      A1 886F5F00    mov eax,dword ptr ds:[5F6F88]

005DEC59      8B00           mov eax,dword ptr ds:[eax]

005DEC5B      E8 E8AAE2FF    call Music_kt.00409748

005DEC60      8B85 B0FEFFFF  mov eax,dword ptr ss:[ebp-150]

005DEC66      8D95 B4FEFFFF  lea edx,dword ptr ss:[ebp-14C]          //载入程序一个关键代码（该代码系 SQL 语句经 DES 加密后的密文）

005DEC6C      E8 1FE5FFFF    call Music_kt.005DD190                  //计算关键代码的 MD5 值

005DEC71      8B95 B4FEFFFF  mov edx,dword ptr ss:[ebp-14C]

005DEC77      58             pop eax

005DEC78      E8 EF5EE2FF    call Music_kt.00404B6C                  //把文件校验码 MD5 与关键代码 MD5 加密后的代码进行连接

005DEC7D      8B85 BCFEFFFF  mov eax,dword ptr ss:[ebp-144]

005DEC83      8D95 C0FEFFFF  lea edx,dword ptr ss:[ebp-140]
```



# Music\_kt.exe 逆向分析与自校验机制破解

## 1. 初始加密逻辑分析

```
005DEC89     E8 02E5FFFF    call Music_kt.005DD190                  //把连接后的代码再进行 MD5 加密得到“07d2b16ca04461c374f9f104d2167d45”--------------怎么样，这样的做法够 BT 吧  

005DEC8E     8B95 C0FEFFFF mov edx,dword ptr ss:[ebp-140]  

005DEC94     A1 B46C5F00    mov eax,dword ptr ds:[5F6CB4]  

005DEC99     8B00           mov eax,dword ptr ds:[eax]  

......  
```

对堆栈区的观察如下：

```
0012FCB4   00D82720  ASCII "1432CCA3CF7287C5C2740E93781B7388743406C8BF2F1F8BE56B6C026F21849795AC278931E149FECD888BAF> //关键代码  

0012FCB8   00D82A9C  ASCII "895932964c568fb566509ce3c7d51de9"  //“关键代码”MD5 加密后得到的代码  

0012FCBC   00D82544  ASCII "C:\Program Files\Music_KTV_DJB\Music_ktv_tk-1.exe"  //所调试的程序  

0012FCC0   00D82ACC  ASCII "d5ced6890a02d58b280add2bf46eadbc895932964c568fb566509ce3c7d51de9" //前半部分为所调试的程序文件 MD5 加密码，后半部分是“关键代码”MD5 加密后得到的代码，这里进行了连接  

0012FCC4   00D82C88  ASCII "07d2b16ca04461c374f9f104d2167d45"  //连接后再次进行 MD5 编码。  

0012FCC8   00D8197C  ASCII "DEMO SoftWare"  

0012FCCC   00D82514  ASCII "d824dd50127e1b3e9b8a823b79994994"   //"DEMO SoftWare"加密后的 MD5,暂时没发现有何用途  
```

=========

## 2. 中途总结

以上分析是多次、反复跟踪发现的，并非偶然。最初走入了误区，一直以为是加壳程序进行了自校验，当取得文件信息后为什么始终不进行比较，也找不出在哪儿进行的比较。更不理解的是为什么与“关键代码”进行整合（后来才发现关键代码为作者实现程序功能限制的重要 `SQL` 语句，该语句已经经过了 `DES` 加密），可见作者非常谨慎！

=========

## 3. 断点跟踪与代码对比

好，继续：

分析刚才得到的最终 `MD5` 码 `07d2b16ca04461c374f9f104d2167d45`，应该最终会参与比对，故迟早会用到的，对此在我们跟踪内存数据区 `00D82C88`，并在这里下 `DWord` 的硬件访问断点，`F9`，断在这里：

```
00409765     807C1F FF 20  cmp byte ptr ds:[edi+ebx-1],20

0040976A   ^ 76 F4         jbe short Music_kt.00409760        //断点位置，F8 继续跟踪  

0040976C     3BF3          cmp esi,ebx  

0040976E     7D 0A         jge short Music_kt.0040977A  

00409770     8BC5          mov eax,ebp  

00409772     E8 2DB1FFFF   call Music_kt.004048A4             //再次中断  

00409777     EB 17         jmp short Music_kt.00409790        //跳  

00409779     4E            dec esi  

0040977A     807C37 FF 20  cmp byte ptr ds:[edi+esi-1],20  

0040977F   ^ 76 F8         jbe short Music_kt.00409779  

00409781     55            push ebp  

00409782     8BCE          mov ecx,esi  

00409784     2BCB          sub ecx,ebx  

00409786     41            inc ecx  

00409787     8BD3          mov edx,ebx  

00409789     8BC7          mov eax,edi  

0040978B     E8 34B6FFFF   call Music_kt.00404DC4  

00409790     5D            pop ebp  

00409791     5F            pop edi  

00409792     5E            pop esi  

00409793     5B            pop ebx  

00409794     C3            retn                               //返回上层 CALL：  

```

\========

```
004049B5     89D8          mov eax,ebx  

004049B7     E8 E8FEFFFF   call Music_kt.004048A4  

004049BC     893B          mov dword ptr ds:[ebx],edi        //来到这里  

004049BE     5F            pop edi  

004049BF     5E            pop esi  

004049C0     5B            pop ebx  

004049C1     C3            retn                               //返回上层 CALL  

```

\=======

```
00404DE7     E8 A8FBFFFF   call Music_kt.00404994  

00404DEC     EB 11         jmp short Music_kt.00404DFF        //来到这里  

00404DEE     31D2          xor edx,edx  

00404DF0   ^ EB E5         jmp short Music_kt.00404DD7  

00404DF2     89D9          mov ecx,ebx  

00404DF4   ^ EB EB         jmp short Music_kt.00404DE1  

00404DF6     8B4424 08     mov eax,dword ptr ss:[esp+8]  

00404DFA     E8 A5FAFFFF   call Music_kt.004048A4  

00404DFF     5B            pop ebx  

00404E00     C2 0400       retn 4  

00404E03     C3            retn                               //返回上层 CALL  

```

\========            

```
0040978B     E8 34B6FFFF   call Music_kt.00404DC4  

00409790     5D            pop ebp                              //来到这里              

00409791     5F            pop edi  

00409792     5E            pop esi  

00409793     5B            pop ebx  

00409794     C3            retn                               //返回上层 CALL  

```

\========

```
005E70BA     E8 8926E2FF   call Music_kt.00409748  

005E70BF     8B85 78FFFFFF mov eax,dword ptr ss:[ebp-88]  //返回到这里，看看堆栈区：  

005E70C5     50            push eax  

```

\========

堆栈 `ss:[0012FB24]=00D81FA8`, (`ASCII "07d2b16ca04461c374f9f104d2167d45"`)    //这就是刚才最终 `MD5` 码（其实在首次硬件中断后，`CPU` 内存中也出现过的）

`eax=0012FB24`

\========

## 4. 关键比对逻辑

我们往上看几行：

```
005E70BA     E8 8926E2FF   call Music_kt.00409748           //这行为读取刚才最终 MD5 值 07d2b16ca04461c374f9f104d2167d45  

005E70BF     8B85 78FFFFFF mov eax,dword ptr ss:[ebp-88]     

005E70C5     50            push eax  

005E70C6     8D95 74FFFFFF lea edx,dword ptr ss:[ebp-8C]  

005E70CC     8B45 FC       mov eax,dword ptr ss:[ebp-4]  

005E70CF     8B80 04030000 mov eax,dword ptr ds:[eax+304]  

005E70D5     8B80 80000000 mov eax,dword ptr ds:[eax+80]    //载入标准的 MD5（未破解的 MD5 值 dff845376ac0cf39c12e1519b5385788）  

005E70DB     E8 6826E2FF   call Music_kt.00409748  

005E70E0     8B95 74FFFFFF mov edx,dword ptr ss:[ebp-8C]  

005E70E6     58            pop eax  

005E70E7     E8 C4DBE1FF   call Music_kt.00404CB0           //比较 MD5  

005E70EC     74 2D         je short Music_kt.005E711B       //这就是千辛万苦寻找的关键点，不跳就 OVER，把 je 改成 JMP 就一切 OK  

005E70EE     8B45 FC       mov eax,dword ptr ss:[ebp-4]  

005E70F1     8B80 F4030000 mov eax,dword ptr ds:[eax+3F4]  

005E70F7     05 80000000   add eax,80  

005E70FC     BA 88755E00   mov edx,Music_kt.005E7588  

005E7101     E8 F2D7E1FF   call Music_kt.004048F8           //这里应该是对程序进行错误处理  

005E7106     8B45 FC       mov eax,dword ptr ss:[ebp-4]  

005E7109     8B80 74030000 mov eax,dword ptr ds:[eax+374]  

005E710F     6950 30 EA490>imul edx,dword ptr ds:[eax+30],849EA  

005E7116     E8 1D87E5FF   call Music_kt.0043F838           //这里应该是对程序进行错误处理  

005E711B     8B45 FC       mov eax,dword ptr ss:[ebp-4]  

......  
```

## 5. 标准 MD5 存储位置分析

感兴趣的是我们再看看文件标准的 `MD5` `dff845376ac0cf39c12e1519b5385788` 藏在什么地方呢，首先用 `OD` 搜索程序里面的参考文本，没有发现。我们再向上看看刚才比较的那段程序，在 `005E6F77` 处下断（这个断点是经过反复确定的，后来发现此处有突破点）

```
005E6F77     8B55 88       mov edx,dword ptr ss:[ebp-78]  

005E6F7A     8D4D 8C       lea ecx,dword ptr ss:[ebp-74]    //载入一段经过 DES 加密后的代码  

005E6F7D     B8 F0745E00   mov eax,Music_kt.005E74F0        //ASCII "BAC65E17BF4EC3439371EDB7A7D2CF2B67645C5FFBF3EC34968B8370ACA2E9B74CF6B0BE8A65901221559BDEBF3687146C1C8F66F72029C5"  

005E6F82     E8 596DFFFF   call Music_kt.005DDCE0           //调用 DES 解密  

005E6F87     8B45 8C       mov eax,dword ptr ss:[ebp-74]    //得到 DES 之前的 SQL 语句：  

```

\=======

堆栈 `ss:[0012FB38]=00D81EC4`, (`ASCII "SELECT data1 as ktv1md5,data2 as ht2md5 FROM DWXXB"`)

`eax=00000000`

分析看来标准 `MD5` 数据存储在 `*.mdb` 数据库中的 `DWXXB` 表中，于是跟踪数据库的密码（很容易获得为 `MMMNNN`），打开该表一看，果然找到了标准的 `MD5`，呵呵，原来这个重要的东东居然藏在这里，至此一切疑惑均已解开（破解自校验也可以修改 `MD5` 到数据库中）！

\=======

## 6. 总结

程序自校验有很多都是加壳程序所为，但校验失败后往往都是拒绝运行程序。而本文是作者在编程时融入了程序的自校验，同时还把程序功能限制的 `SQL` 语句捆绑在一起。首先将 `SQL` 字符串进行 `DES` 加密，再把加密后的密文再次进行 `MD5`，最后再与文件校验的 `MD5` 进行连接，连接后再进行最终 `MD5` 加密。加密后的标准 `MD5` 码藏到了 `MDB` 数据库中，启动程序时进行读出，然后与当前的 `MD5` 进行比对（这个比对的地方与文件校验的地方相差很远，且藏得很隐蔽，对比跟踪法基本没戏）。一旦程序发生修改或 `SQL` 语句发生变化，这两个条件的任一个成立就 `OVER`。

自从 `2006` 年 `V6.2` 版本遭破解后，作者已经把加密提高到了一个空前的高度。呵呵，经过 `2` 周的努力，`V8.5` 的程序已经得到完美破解！

\====END====
