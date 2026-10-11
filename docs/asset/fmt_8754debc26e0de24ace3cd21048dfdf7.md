
CrackZ 你好，我在制作 `VirtuoZo` 许可证里度过了快乐的时光，但是用标准的 `SEED` 和 `VENDOR` 代码回收技术没有成功生成许可证。对于 `FLEXlm` 的去保护我已经过有良好的实践，但是 `VirtuoZo` 把我给难住了。我曾经浏览过您的网站，认真读过 `Acme` 的 `FLEXlm5.1`“许可证生成又一法”中令人感兴趣的问题。你知道，这个方法对于 `v6.1` 和今后的版本已经不适用了，但是在这个的启发之下，我已经发现怎样用这个相似的方法制作许可证。

I'll describe the method in few words and probably you'll bring my ideas to more people interested in `FLEXlm` `6.1`/`7.0` license keys for 1-3 features without `Genlic32` or `Flexgen` but just with `SoftICE`. The software has just `v6.1` `FLEXlm` code implemented into about 30 executables with nothing special. I've turned on `FLEXlm` diagnostics inside registry and discovered feature name and version. Vendor name was easy to find too. Next I have played a lot with seeds and vendor code before discovering a really interesting part of code (address `.4712F0`). "It really looks like license creation", I continued with tracing this part of code. Next part appears really cool (address `.471538`) because it looks like usual text-with-binary comparison!.

我用几句话来描述这个方法，或许你可以将我的想法告诉给对 `FLEXlm` `6.1`/`7.0` 的 `license key` 制作感兴趣的人(有 1-3 个 `FEATURE`,没有 `Genlic32` 或 `Flexgen` 但只有 `SoftICE`)。这个软件用 `v6.1` `FLEXlm` 的代码大约 30 个可执行文件，没有什么特别之处。启动 `FLEXlm` 诊断发现 `FEATURE NAME` 和版本。`VENDOR NAME` 也很容易找到。下一步，在找到真正让人感兴趣的代码（地址 `.4712F0`）之前我花了很多时间在 `SEED` 和 `VENDOR` 代码上。“它真的象极了许可证的制作”，我继续跟踪这部分代码。下一步看起来真的很酷（地址 `.471538`）因为这怎么看怎么象通常的文本－二进制比较！

Voila! At address `.4715EC` you can see the best part of all `FLEXlm` code -- license number from `license.dat` and generated number comparison. That's all. You can have it directly by typing `:D DS:71E1B8` or by passing all `JNE 471613` with zero flag and wait while `FLEXlm` converts this binary to text string at `.471609`!. Another interesting thing has been revealed. This procedure have been called twice so not only one valid license number can be generated but some more :-).

瞧，那就是！地址`.4715EC`,你看到了所有 `FLEXlm` 代码中最棒的部分――从 `license.dat` 读出的 `license` 数值和生成的数值比较。一切就到此为止。你直接键入：`:D DS:71E1B8` 或用 0 标记过所有的 `JNE 471613`,你等着 `FLEXlm` 将这个二进制在地址`.471609` 转变为字符串吧。还发现了另一个有意思的事情。这个过程被调用两次，所以不会仅是一个有效的许可证数码的生成，而且还有别的东西:-)。

```assembly
.004712CF: push esi  
.004712D0: call .0048EDA8 -------- (1)  
.004712D5: add esp,00C ;""  
.004712D8: jmps .004712DD -------- (2)  
.004712DA: mov esi,\[ebp\]\[0000C\]  
.004712DD: mov d,\[ebp\]\[-0004\],0 ;"  
.004712E4: cmp d,\[ebp\]\[-0024\],0 ;" "  
.004712E8: jle .004714C9 -------- (3)  
.004712EE: xor eax,eax  
.004712F0: mov cl,\[eax\]\[esi\] <-- 制作许可证的数码  
.004712F3: xor \[eax\]\[0071E1B8\],cl  
.004712F9: inc eax  
.004712FA: cmp eax,8 ;""  
.004712FD: jl .004712F0 -------- (4)  
.004712FF: cmp d,\[ebp\]\[-0004\],000 ;" "  
.00471303: jne .004714AA -------- (5)  
.00471309: mov ecx,\[ebp\]\[00008\]  
.0047130C: cmp d,\[ecx\]\[00000020C\],000 ;" "  
.00471313: jne .00471454 -------- (6)

代码继续 :-

.00471521: mov d,\[ebp\]\[-0008\],000000008 ;"  
.00471528: cmp d,\[ebp\]\[00018\],066D8B337 ;  
.0047152F: jne .00471538 -------- (1)  
.00471531: mov d,\[ebp\]\[-0008\],000000006 ;"  
.00471538: xor esi,esi <-- 开始比较  
.0047153A: cmp \[ebp\]\[-0008\],esi  
.0047153D: jle .00471601 -------- (2)  
.00471543: lea edi,\[ebp\]\[-0020\]  
.00471546: mov bl,\[edi\]  
.00471548: call \_\_p\_\_\_mb\_cur\_max ;MSVCRTD.dll  
.0047154E: cmp d,\[eax\],001 ;""  
.00471551: jle .00471564 -------- (3)  
.00471553: movsx eax,bl  
.00471556: push 004  
.00471558: push eax  
.00471559: call \_isctype ;MSVCRTD.dll

.004715D6: je .004715EC -------- (1)  
.004715D8: movzx eax,bl  
.004715DB: push eax  
.004715DC: lea edx,\[esi\]\[00071E1B8\]  
.004715E2: push esi  
.004715E3: push edx  
.004715E4: push d,\[ebp\]\[00008\]  
.004715E7: call ecx  
.004715E9: add esp,010 ;""  
.004715EC: cmp \[esi\]\[00071E1B8\],bl <- 猜猜这是什么?  
.004715F2: jne .00471613 -------- (2)  
.004715F4: add edi,002 ;""  
.004715F7: inc esi  
.004715F8: cmp \[ebp\]\[-0008\],esi  
.004715FB: jg .00471546 -------- (3)  
.00471601: push d,\[ebp\]\[00018\] <-- 在这里，数码转化为字符串  
.00471604: push 0071E1B8 ;" qá¸"  
.00471609: call .004716AC -------- (4)  
.0047160E: add esp,008 ;""  
.00471611: jmps .00471615 -------- (5)
```

Needless to say, you should be able to find something useful amongst this snippet to search for with your hex editor.

毋庸置言，用十六进制编辑器搜索你能在这个代码段里找到有用的东西。

## `v7.2` 摘录

对 `FLEXlm` `v7.2x` 的绪言性评论来自两个地方。

“`v7.2` 有几个变化：（a）4 个 `vendor seed`;(b)`CRO` key。我试着用特定的 seed 和 key 做了个 deamon,编译了一个新的 `demo.exe` 和 `lmcrypt.exe`。然而，`demo.exe` 不认 `lmgrypt` 做出来的 `license`。我想问题主要出在 `SEED3` 和 `SEED4` 是我自己指定的”。

“不幸的是，该 `SEED` 并没有保存在 `deamon` 里。`ECC` 专用的 `SEED3` 和 `SEED4` 用来制作公匙和私匙。`DEAMON` 和/或应用程序从许可证文件读取到的 `SIGN=`只是用来验证签名（`signature`）,并不是真正的 key。私匙是用来制作签名并仅编译在 `lmcrypt` 的二进制文件里。回收 `SEED3` 和 `SEED4` 是 `ECC` 制作关键的第一步，然后，一旦私匙确定下来，你就得做逆向工程：私匙是如何从 `SEED` 里产生。希望这对你有用。”

所以在早期，看起来我们很像又回到了补丁 ;-)。

| **文档标题** | **说明** | **日期** |
| :--- | :--- | :--- |
| [Ansoft Serenade v8.5/v8.7](http://www.woodmann.com/crackz/Tutorials/Serenade.htm) | FLAIR 辅助 FLEXlm 许可证的制作 | 30/12/01 |
| [Crypt Filters](http://www.woodmann.com/crackz/Tutorials/Nbufilt.htm) | 承蒙 Nolan Blender 好意,描述了加密滤波器（crypt filter）如何执行并如何用标准工具破解 | 21/11/00 |
| [ECC FLEXlm](http://www.woodmann.com/crackz/Tutorials/Flexecc.htm) | 讨论了早期 FELXlm 用来提高性能的 ECC 的弱点 | Dec 2001 |
| ["How to crack a PC-based license manager"](http://www.woodmann.com/crackz/Tutorials/Flexpilg.htm) | Pilgrim 的 FLEXcrypt 和 FLEXlm 破解文章（2 篇合并在一块） | 30/10/98, 07/01/99 |
| ["FlexLock ...less secure than the rest of FLEXlm"](http://www.woodmann.com/crackz/Tutorials/Flexlock.htm) | FlexLock 破解, pilgrim 的第三篇友情贡献. | June 1999 |
| [IMSL](http://dev.csdn.net/user/zxg32) | FLEXlm 逆向拓展,7 个教程. | April 2004 |
| [Information hiding methods used by FLEXlm targets](http://www.woodmann.com/crackz/Tutorials/Nolflex2.htm) | 描述了新版的 FLEXlm 如何隐藏重要的 SEED 代码。FLEXlm 专家 Nolan Blender 提供。 | October 1999 |
| [lc\_new\_job() FLEXlm v6.1 by dan](http://www.woodmann.com/crackz/Tutorials/Danflex.htm) | 很棒的分析文章，描述了 GlobeTrotter 所用迷乱法的逆向来回收 key | September 1999 |
| [Reversing GlobeTrotter's FLEXcrypt](http://www.woodmann.com/crackz/Tutorials/Nolflexc.htm) | Nolan Blender 的 Key 提取与加密算法逆向 | 17/09/99 |
| [SDS2 v6.112](http://www.woodmann.com/crackz/Tutorials/Sdsflex.htm) | 简单的例子演示了怎样生成 FLEXlm 的许可证 | 28/08/99 |
| [Siul+Hacky's FLEXlm Linux Cracking](http://www.woodmann.com/crackz/Tutorials/Siulflex.htm) | 一个非常棒的文章，描述了 Linux 的调试与反汇编和 FLEXlm 的弱点（开山鼻主之作） | July 1999 |
| [UGFLEX - modified FLEXlm by Unigraphics](http://www.woodmann.com/crackz/Tutorials/Ugmaci1.htm) | Macilaci's first foray inside Unigraphics. | 15/11/99 |
| [UGFLEX2 - let UGFLEX generate the keys for you](http://www.woodmann.com/crackz/Tutorials/Ugmaci2.htm) | Macilaci's second Unigraphics tutorial, this time to generate the correct keys. | 16/11/99 |
| [Using FLEXlm Internal Diagnostics](http://www.woodmann.com/crackz/Tutorials/Acmeflex.htm) | 利用 FLEXlm 内诊断来找到一,Acme 倾情奉献。 | Jul. 1999 |
| [Vendor Defined Encryption (locating and reversing)](http://www.woodmann.com/crackz/Tutorials/Amante1.htm) | 开发商的自定义保护，Amante4 倾情奉献。 | 08/01/00 |
| [Zendenc](http://www.woodmann.com/crackz/Tutorials/Nolflex3.htm) | 出自 Nolan Blender 之手，提供更多的 FLEXlm 技巧. | June. 2001 |

## FLEXlm 加密滤波器和其他的问题

Most of this is reworked from posts I saw at Fravia's Message Board (it may however be useful even if the questions are target related) :-

大部分是我从 Fravia 的信息公告栏里的帖子整理出来（即使问题是个案相关，但是可能有用）:-

### Q1

I have read most all the essays I could get my hands on and the API, header files, observed `lc\_set\_attr` etc, etc. Yet I still can't seem to generate correct codes with the keys/seeds I extract. The target is Pixar Renderman, found a copy and thought it would be fun to play around with. At any rate, I'm not positive that I have the correct vendor key 5, although from previous posts, I gather that the only thing used to make the keys, is the seeds. Has this changed in Flex 6.1?.

### 问 1

我看完了我手头能有的所有文章,API,头文件, 观察 `lc\_set\_attr` 等等、等等。但我似乎仍不能产生正确的 `key`/`seed`。 目标是 `pixarRenderMan`,找到一个拷贝,以为会玩地很好. 无论如何,我不认为我得到正确的 `vendor key 5`,尽管有以前的帖子， 我推测, 唯一用来制造 `key` 的是 `SEED`. 这在 `Flex` `6.1` 里难道有不这样吗? 
