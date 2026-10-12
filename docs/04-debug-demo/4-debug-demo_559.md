
# 绿鹰 PC 万能精灵 3.92

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-299.htm`](https://www.pediy.com/kssd/pediy07/pediy7-299.htm)

## 元信息

- **Cracker**：`prince`
- **时间**：2005.06.30
- **声明**：只做技术交流，不做商业用途，如果你手头宽裕并喜欢这个软件的话请支持正版软件。
- **E-mail**：`Cracker_prince@163.com`

## 软件信息

- **软件说明**：绿鹰 PC 万能精灵 3.92
- **保护方式**：序列号 + 反跟踪 + 自校验
- **限制方式**：功能限制
- **外壳保护**：`PECompact` 2.x -> `Jeremy Collake`
- **编译器/语言**：`Microsoft Visual C++` 6.0 / `C/C++`
- **下载地址**：`http://www.onlinedown.net/soft/6396.htm`
- **目的**：脱壳后自校验的去除

## 分析过程

现在的软件为了保护自身不被修改，防止脱壳，不少都加上了自身的完整性校验，一旦发现自己的关键指令被修改或被脱壳了之后，毫无提示地就拒绝运行。这种令人头疼的自校验相信难住了不少跟我一样的菜鸟！后来再遇到自校验，就只好开 2 个 OD，下断 `CreateFileA`，然后，一步一步地跟，一句一句地对照，不知道经过多少步之后，哈，发现了一个跳转不一样，激动地改掉，成功。对于这样直接比较的软件校验还可以用这种不怕脏不怕累的方法搞定，但是对于那种将自身一个字节一个字节读取出来，然后经过超级变态复杂地算法运算过后，得到了几个值，偷偷记下来，或作为后面程序运行的关键值，或者分开几个地方偷偷比较，如果不对，就异常啦！这样的校验，相信光凭耐心和运气是不行的吧？

前几天在 FLY 老大的论坛偶尔看到了 `fxyang` 大侠的文章：《浅谈 cool edit pro2.1 脱壳后解决自校验》（`http://fly.ccgforum.info/viewthread.php?tid=1339&fpage=1&highlight=%2Bfxyang`），就是这样的校验方法。`fxyang` 大侠的解决方法是：跟踪原版程序，由于校验算法复杂，不分析过程，只看得到几个结果，记下他们的值，然后在脱壳后的文件中强行将原版的值写入，最后问题解决。这种方法虽然巧妙，但是还是要分析原版计算出的几个值，并且要在脱壳文件中一个一个写入，修改字节过多，容易出错，毕竟如果有一个地方没有看到就会前功尽弃。有没有更好更简便的方法呢？呵呵，终于到主题了，呼~

静下心来仔细想一下，总结一下程序是如何校验的：首先 `GetModuleFileName` 得到自身路径和文件名，然后 `CreateFile` 打开自身，接下来，如果是简单地比较大小，就调用 `GetFileSize` 得到程序大小，和原版大小比较，`size` 大了就 `OVER`，这种比较简单；如果要进行 `CRC` 等算法校验，就会 `CreateFileMapping` 映射的一块内存中准备 `ReadFile` 读取计算，最后计算出几个值... 呵呵，注意到了吗？无论哪中方法，都要 `CreateFile` 打开自身，才能进行下一步操作，那想到了吗？既然程序要打开文件校验，我们就想办法让它不打开自身而是打开原版去计算、校验，这样无论它怎么变态，复杂，计算结果都是正确的（当然了，因为原版我们就没动过嘛）！那可能有朋友说了，怎么做啊？改动复杂不复杂啊？别急，看我举例说明。

`PECompact` 2.x 脱壳很简单（难的我也不会），`he eip`，`F9` 一步就到 `OEP`，`Ollydump` 脱之，`ImportREC` 修复，剪切掉一个垃圾指针，`FIX`，`OK`，就这么简单。由于有自校验，双击运行毫无反应，OD 加载，下断 `CreateFileA`，`F9` 来到这里：

```assembly
----------------------------------------------------------------------------------
0040538D    8B4D 08           mov ecx,dword ptr ss:[ebp+8]     ; 文件名送 ECX
00405390    57               push edi                          ; 参数入栈
00405391    68 27000008      push 8000027                      ; 参数入栈
00405396    6A 03            push 3                            ; 参数入栈
00405398    57               push edi                          ; 参数入栈
00405399    6A 01            push 1                            ; 参数入栈
0040539B    68 00000080      push 80000000                     ; 参数入栈
004053A0    51               push ecx                          ; 关键，要打开的文件名入栈
004053A1    FF15 74124600    call dword ptr ds:[<&kernel32.Cr>; KERNEL32.CreateFileA
004053A7    8BF0             mov esi,eax
004053A9    83FE FF          cmp esi,-1
004053AC    8975 DC          mov dword ptr ss:[ebp-24],esi
004053AF    75 15            jnz short UN2_.004053C6
004053B1    FF15 84124600    call dword ptr ds:[<&kernel32.Ge>; KERNEL32.GetLastError
004053B7    56               push esi
004053B8    8945 EC          mov dword ptr ss:[ebp-14],eax
004053BB    FF15 78124600    call dword ptr ds:[<&kernel32.Cl>; KERNEL32.CloseHandle
004053C1    E9 13010000      jmp UN2_.004054D9
004053C6    8D55 D4          lea edx,dword ptr ss:[ebp-2C]
...
```

文件校验算法没看：

```assembly
00405465    FF15 6C124600    call dword ptr ds:[<&kernel32.MapV>; KERNEL32.MapViewOfFile
0040546B    8BF8             mov edi,eax
0040546D    8BC6             mov eax,esi
0040546F    8BD7             mov edx,edi
00405471    8BC8             mov ecx,eax
00405473    48               dec eax
00405474    85C9             test ecx,ecx
00405476    8945 08          mov dword ptr ss:[ebp+8],eax
00405479    76 28            jbe short UN2_+.004054A3
0040547B    8B0B             mov ecx,dword ptr ds:[ebx]
0040547D    33DB             xor ebx,ebx
0040547F    8A1A             mov bl,byte ptr ds:[edx]           ；PE 文件头，表示开始计算了
00405481    8BC1             mov eax,ecx
00405483    25 FF000000      and eax,0FF
00405488    33C3             xor eax,ebx
0040548A    8B5D E0          mov ebx,dword ptr ss:[ebp-20]
0040548D    C1E9 08          shr ecx,8
00405490    8B5B 04          mov ebx,dword ptr ds:[ebx+4]
00405493    8B0483           mov eax,dword ptr ds:[ebx+eax*4]
00405496    8B5D 0C          mov ebx,dword ptr ss:[ebp+C]
00405499    33C1             xor eax,ecx
0040549B    42               inc edx
0040549C    8903             mov dword ptr ds:[ebx],eax
0040549E    8B45 08          mov eax,dword ptr ss:[ebp+8]
004054A1    ^ EB CE          jmp short UN2_+.00405471
004054A3    57               push edi
004054A4    FF15 70124600    call dword ptr ds:[<&kernel32.Unma>; KERNEL32.UnmapViewOfFile
004054AA    8B55 CC         mov edx,dword ptr ss:[ebp-34]
004054AD    8B4D D0         mov ecx,dword ptr ss:[ebp-30]
004054B0    33C0            xor eax,eax
004054B2    03D6            add edx,esi
004054B4    13C8            adc ecx,eax
004054B6    8955 CC         mov dword ptr ss:[ebp-34],edx
004054B9    8B55 D8         mov edx,dword ptr ss:[ebp-28]
004054BC    894D D0         mov dword ptr ss:[ebp-30],ecx
004054BF    8B4D D4         mov ecx,dword ptr ss:[ebp-2C]
004054C2    8B7D D0         mov edi,dword ptr ss:[ebp-30]
004054C5    2BCE            sub ecx,esi
004054C7    8B75 DC         mov esi,dword ptr ss:[ebp-24]
004054CA    1BD0            sbb edx,eax
004054CC    894D D4         mov dword ptr ss:[ebp-2C],ecx
004054CF    8955 D8         mov dword ptr ss:[ebp-28],edx
004054D2    ^ E9 5CFFFFFF   jmp UN2_+.00405433
004054D7    33FF            xor edi,edi
004054D9    8B45 E8         mov eax,dword ptr ss:[ebp-18]
004054DC    3BC7            cmp eax,edi
004054DE    74 0C           je short UN2_+.004054EC
004054E0    50              push eax
004054E1    FF15 78124600   call dword ptr ds:[<&kernel32.Clos>; KERNEL32.CloseHandle
----------------------------------------------------------------------------------
```

在 `0040538D` 处就是要打开的文件名赋值，我们就是要改这里，让 `ECX` 的值为我们指定的原版程序的路径。首先做好准备工作，比如原版的路径为：`D:\Program Files\绿鹰 PC 万能精灵\adam.exe`，我们的目的就是要将这个字符串赋值给 `ECX`，所以要把这个字符串写到程序里面。由于 `PE` 文件 `Section` 之间的对齐，文件中通常都会有一些间隙，用 `UE` 打开就是一堆 `00` 的地方，我们就用利用这些空地写我们的原版路径字符串。用 `UE` 打开脱壳后的文件，拉动滚动条到文件的结尾，找到空地，将上面的字符串复制拷贝上去，记住我们的字符串的起始地址，我的是 `001B1F80`，后面用的到，然后保存修改后的文件。回到 OD，下断 `0040538D`，重新断下，修改代码为：

```assembly
----------------------------------------------------------------------------------
0040538D  - E9 1ECC1A00       jmp UN2_+.005B1FB0
00405392    90                nop
00405393    90                nop
00405394    90                nop
00405395    90                nop
00405396    6A 03             push 3
00405398    57                push edi
00405399    6A 01             push 1
0040539B    68 00000080       push 80000000
004053A0    51                push ecx
004053A1    FF15 74124600     call dword ptr ds:[<&kernel32.Cr>; KERNEL32.CreateFileA
----------------------------------------------------------------------------------
```

因为这里空间不够写入我们的代码，所以要跳到空地写我们要的代码，然后再跳回来继续执行，就好象什么都没有发生过一样，:) `005B1FB0` 是我们写好的字符串后面的空地：

```assembly
----------------------------------------------------------------------------------
005B1FA8    0000             add byte ptr ds:[eax],al ；空地
005B1FAA    0000             add byte ptr ds:[eax],al ；空地
005B1FAC    0000             add byte ptr ds:[eax],al ；空地
005B1FAE    0000             add byte ptr ds:[eax],al ；空地
005B1FB0    B9 801F5B00      mov ecx,UN2_+.005B1F80   ；将写好的原版文件地址赋值给 ECX
005B1FB5    57               push edi                  ；继续压栈参数
005B1FB6    68 27000008      push 8000027              ；继续压栈参数
005B1FBB  - E9 D633E5FF     jmp UN2_+.00405396         ；跳回去继续执行
005B1FC0    0000             add byte ptr ds:[eax],al
```



OK，保存修改，完工，运行试试，呵呵，运行了~ :)

`AntiDebug` 的去除：下断 `SetUnhandledExceptFilter`，共有两处，`NOP` 掉；程序循环调用 `CreateToolhelp32Snaps`

`hots` 进行父进程检测，将 `00418807` 处改为绝对跳转即可。

总结一下：

用这种偷天换日，偷梁换柱，偷龙转凤的手段可以轻易对付利用 `CreateFilaA` 进行自校验的程序，代码修改量小，成功率高，真乃居家旅行必备之良药，^O^ 以后再见到自校验，就又多了一种对付的方法啦~

看来遇到难以解决的问题的时候，不妨转换一下思路，避其锋芒，找到像打太极一样，四两拨千斤的方法，复杂问题就变得简单了。

呵呵，小弟菜鸟，有不妥的地方请各位大侠见谅、赐教，另外多谢 `fxyang` 大侠的文章。

菜鸟写菜文~

补充：以上方法不适用于破解文件的发布，因为要依赖原版的绝对路径，我的目的只是为了让它运行，从而使我们可以跟踪出它的注册码或者注册算法，毕竟写出注册机才是我们的最终目标。

prince 2005.06.30

任何问题可至：`Cracker_prince@163.com`

`4dd466d0e1e0cc123afbe504da2462ca`
