
# SMC 技术破解 Winlicense 2.06 KEY 保护+VMP 的程序

> 原文：[`www.pediy.com/kssd/pediy10/87175.html`](https://www.pediy.com/kssd/pediy10/87175.html)

【破文标题】SMC 技术破解 Winlicense 2.06 KEY 保护+VMP 的程序  
【破文作者】wqhlgr  
【破解工具】OllyICE，PEiD v0.95  
【破解平台】WinXP sp2  
【更新时间】 2009-04-25  
【软件语言】 中文  
【应用平台】 Win2000/WinXP/VISTA  
【软件性质】 商业软件  
【软件大小】 1.7M  
【保护方式】 Winlicense 2.06 KEY 保护+VMP  
【从本文你可以学到】1. SMC 2. 过 WL2.06 反 FILI PATCH 3. PATCH WL2.06 硬件 KEY

朋友让我帮他破一个程序，程序的主要内容在 DLL 中，DLL 用了 WL key 保护，PEID 查看结果如图，有 VMP 所以脱得难度很大了。于是便试着脱了一下。

![](img/4d6a2c70d5a547bd66c085a06612ceee.jpg)

### 1.尝试脱壳

由于有 VMP，所以修复的难度很大了，DLL 脱壳还要重定位，很麻烦，换条路试试吧。于是便试着用对付一般壳的方法来试试吧，想到了 SMC 法，于是试着用 SMC 来 PATCH。PATCH 硬件 KEY 很简单，可以看 SHOOOO 的文章。这里我用了一个自动 PATCH 脚本：`TM - WL HWID & TRIAL L.B.C. BASIC Unpacker 1.0.txt` 这脚本有一个帮你找硬件 KEY PATCH 的功能，于是载入程序跑这脚本。跑完后看记录如图 `HWID WORD ADDRESS a5dd81 61f27c3a`，这就是地址和 KEY 的值，下面的 `NEW HWID` 就是你的硬件地址，只要在 `A5DD81` 硬件访问断点把最后一次变成 `61F27CA` 的时候改为你的硬件 ID 就 PATCH 成功了，找完后记下来，确实破解了，但这样离不开 OD。

![](img/cf33c45e461f6bdd7b5de9a66ab53054.jpg)

下图是最后一次这里的值变为 `61F27C3A`，在这改成你的硬件 ID 就可以搞定了，但是退出 OD 就不行了

![](img/a62f500924ae6fdfe9be966ee1355985.jpg)

#### SMC PATCH key

于是便想到了 SMC。

下面 OD 重新载入 2 次 F8 经过 `PUSHAD` 后 F9，到了 SMC 的地方

下面的指令用不到，可以替换成我们的 SMC 指令

首先让 `00A76260 ^\E9 AEEFFEFF jmp 00A65213`，让他 JMP 到 `090f0b4`，然后比较 `0a5dd81` 中的地址是否为 `61f27c3a` 是就 PATCH 成我们的硬件 ID `117be641`，由于是个 DLL 文件，所以他的基址是会变得，SMC 代码要比 EXE 的特殊一点，如下：`0090F0B4 53`

下面是我写的 SMC 代码

```asm
push    ebx                   //保护 ebx

0090F0B5    E8 00000000     call    0090F0BA

0090F0BA    5B              pop     ebx                 //得到 EIP

0090F0BB    C783 A7711600 6>mov     dword ptr [ebx+1671A7], FFE98E66     //PATCH 的地址

0090F0C5    5B              pop     ebx                     // 恢复 ebx        0090F0C6    E9 9D010000     jmp     0090F268                     //继续程序。下面的代码也差不多，不再赘述。

0090F0CB    53              push    ebx                    

0090F0CC    E8 00000000     call    0090F0D1

0090F0D1    5B              pop     ebx

0090F0D2    81BB B0EC1400 3>cmp     dword ptr [ebx+14ECB0], 61F27C3A  //比较与硬件 ID

0090F0DC    5B              pop     ebx

0090F0DD    74 05           je      short 0090F0E4

0090F0DF    E9 2F611500     jmp     00A65213

0090F0E4    53              push    ebx

0090F0E5    E8 00000000     call    0090F0EA

0090F0EA    5B              pop     ebx

0090F0EB    C783 97EC1400 4>mov     dword ptr [ebx+14EC97], 117BE641  //PATCH 你的硬件 ID

0090F0F5    5B              pop     ebx

0090F0F6    E9 18611500     jmp     00A65213
```

二进制

```text
53 E8 00 00 00 00 5B C7 83 A7 71 16 00 66 8E E9 FF 5B E9 9D 01 00 00 53 E8 00 00 00 00 5B 81 BB

B0 EC 14 00 3A 7C F2 61 5B 74 05 E9 2F 61 15 00 53 E8 00 00 00 00 5B C7 83 97 EC 14 00 41 E6 7B

11 5B E9 18 61 15 00
```

改完后就成功破解了，呵呵，然后复制到可执行文件保存，就可以了。

运行后却出现了错误框

![](img/99c8b146792dd7e3e8b382dfe6f11b7d.jpg)

有反 FILE PATCHING

### 下面是过 WL 2.06 反 FILE PATCHING

在 `GetProcAddress` 的末尾 `retn8` 处按 F2 设置断点

当得到知道 EAX 变成这样时

![](img/49b27a812526b12fd0ec3e1841fa75dd.jpg)

在 `CheckSumMappedFile` 函数地址时在该函数起始地址处按 F2 设置断点。F9 运行。

在函数 `FreeLibrary` 开始地址处按 f2 设置断点，F9 运行，此时断下来后看 ECX

![](img/111f0bf726c58a677ede55d08289b927.jpg)

把 ECX 用 `C32ASM` 写入文件最后就搞定了

### 总结：

1.要敢于尝试，不要看到是强壳+强壳就不敢动了不试怎么知道?

2.会写 SMC 代码

由于程序的 KEY 是朋友花钱买的，被作者看到后就会给 BAN 了，所以不发程序了（虽然也可以 PATCH 黑名单，但更新后作者换新 KEY 就不行了）

脚本下载

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `TM - WL HWID & TRIAL L.B.C. BASIC Unpacker 1.0.rar` |
