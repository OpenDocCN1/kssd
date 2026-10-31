
# [原创]SMC技术破解Winlicense 2.06 KEY保护+VMP的程序

- 【破文标题】SMC技术破解Winlicense 2.06 KEY保护+VMP的程序
- 【破文作者】wqhlgr
- 【破解工具】OllyICE，PEiD v0.95
- 【破解平台】WinXP sp2
- 【更新时间】2009-04-25
- 【软件语言】中文
- 【应用平台】Win2000/WinXP/VISTA
- 【软件性质】商业软件
- 【软件大小】1.7M
- 【保护方式】Winlicense 2.06 KEY保护+VMP
- 【从本文你可以学到】1. SMC     2. 过WL2.06 反FILI PATCH    3. PATCH WL2.06硬件KEY

朋友让我帮他破一个程序，程序的主要内容在DLL中，DLL用了WL key保护，PEID查看结果如图，有VMP所以脱得难度很大了。于是便试着脱了一下。

![](https://bbs.kanxue.com/upload/attach/200904/222765_yovxolxzeuz1uvx.jpg)

## 1. 尝试脱壳

由于有VMP，所以修复的难度很大了，DLL脱壳还要重定位，很麻烦，换条路试试吧。于是便试着用对付一般壳的方法来试试吧，想到了SMC法，于是试着用SMC来PATCH。PATCH硬件KEY很简单，可以看SHOOOO的文章。这里我用了一个自动PATCH脚本：`TM - WL HWID & TRIAL L.B.C. BASIC Unpacker 1.0.txt`，这脚本有一个帮你找硬件KEY PATCH的功能，于是载入程序跑这脚本。跑完后看记录如图 HWID WORD ADDRESS `a5dd81` `61f27c3a`，这就是地址和KEY的值，下面的NEW HWID 就是你的硬件地址，只要在`A5DD81`硬件访问断点把最后一次变成`61F27CA`的时候改为你的硬件ID就PATCH成功了，找完后记下来，确实破解了，但这样离不开OD。

![](https://bbs.kanxue.com/upload/attach/200904/222765_0naelg8fjsszaym.jpg)

下图是最后一次这里的值变为`61F27C3A`，在这改成你的硬件ID就可以搞定了，但是退出OD就不行了。

![](https://bbs.kanxue.com/upload/attach/200904/222765_xwgeqwyps5t01hj.jpg)

## SMC PATCH key

于是便想到了SMC。

下面OD重新载入2次F8经过PUSHAD后F9，到了SMC的地方。下面的指令用不到，可以替换成我们的SMC指令。

首先让`00A76260`处的`E9 AEEFFEFF`（`jmp 00A65213`）跳到`0090F0B4`，然后比较`0A5DD81`中的地址是否为`61F27C3A`，是就PATCH成我们的硬件ID`117BE641`。由于是个DLL文件，所以它的基址是会变的，SMC代码要比EXE的特殊一点。

下面是我写的SMC代码：

```
0090F0B4    53              push    ebx                             //保护ebx
0090F0B5    E8 00000000     call    0090F0BA
0090F0BA    5B              pop     ebx                             //得到EIP
0090F0BB    C783 A7711600 6>mov     dword ptr [ebx+1671A7], FFE98E66     //PATCH的地址
0090F0C5    5B              pop     ebx                             //恢复ebx
0090F0C6    E9 9D010000     jmp     0090F268                        //继续程序。下面的代码也差不多，不再赘述。
0090F0CB    53              push    ebx
0090F0CC    E8 00000000     call    0090F0D1
0090F0D1    5B              pop     ebx
0090F0D2    81BB B0EC1400 3>cmp     dword ptr [ebx+14ECB0], 61F27C3A  //比较与硬件ID
0090F0DC    5B              pop     ebx
0090F0DD    74 05           je      short 0090F0E4
0090F0DF    E9 2F611500     jmp     00A65213
0090F0E4    53              push    ebx
0090F0E5    E8 00000000     call    0090F0EA
0090F0EA    5B              pop     ebx
0090F0EB    C783 97EC1400 4>mov     dword ptr [ebx+14EC97], 117BE641  //PATCH你的硬件ID
0090F0F5    5B              pop     ebx
0090F0F6    E9 18611500     jmp     00A65213
```

二进制：

```
53 E8 00 00 00 00 5B C7 83 A7 71 16 00 66 8E E9 FF 5B E9 9D 01 00 00 53 E8 00 00 00 00 5B 81 BB
B0 EC 14 00 3A 7C F2 61 5B 74 05 E9 2F 61 15 00 53 E8 00 00 00 00 5B C7 83 97 EC 14 00 41 E6 7B
11 5B E9 18 61 15 00
```

改完后就成功破解了，呵呵，然后复制到可执行文件保存，就可以了。

运行后却出现了错误框：

![](https://bbs.kanxue.com/upload/attach/200904/222765_p62ivt80nyyn78x.jpg)

有反FILE PATCHING。

下面是过WL 2.06 反FILE PATCHING：

在`GetProcAddress`的末尾`retn8`处按F2设置断点。

当得到知道`EAX`变成这样时：

![](https://bbs.kanxue.com/upload/attach/200904/222765_umbpzpb5eqaicb6.jpg)

在`CheckSumMappedFile`函数地址时，在该函数起始地址处按F2设置断点，F9运行。

在函数`FreeLibrary`开始地址处按F2设置断点，F9运行，此时断下来后看`ECX`：

![](https://bbs.kanxue.com/upload/attach/200904/222765_r3gbvhkx6nbhfs7.jpg)

把`ECX`用C32ASM写入文件最后就搞定了。

## 总结

1. 要敢于尝试，不要看到是强壳+强壳就不敢动了，不试怎么知道？
2. 会写SMC代码。

由于程序的KEY是朋友花钱买的，被作者看到后就会给BAN了，所以不发程序了（虽然也可以PATCH黑名单，但更新后作者换新KEY就不行了）。

## 脚本下载

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

## 上传的附件

- [1.jpg](javascript:void(0);)（24.47kb，1360次下载）
- [未命名.jpg](javascript:void(0);)（27.85kb，1354次下载）
- [2.jpg](javascript:void(0);)（48.98kb，1352次下载）
- [4.jpg](javascript:void(0);)（6.59kb，1344次下载）
- [5.jpg](javascript:void(0);)（61.81kb，1363次下载）
- [6.jpg](javascript:void(0);)（41.40kb，1352次下载）
- [TM - WL HWID & TRIAL L.B.C. BASIC Unpacker 1.0.rar](javascript:void(0);)（10.75kb，250次下载）
