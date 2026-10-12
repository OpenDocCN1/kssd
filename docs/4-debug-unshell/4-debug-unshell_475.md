
# ASPr 1.31.04.27 另一种思路脱壳(Import Recovery)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-494.htm`](https://www.pediy.com/kssd/pediy09/pediy09-494.htm)

> 【文章标题】: ASPr 1.31.04.27 另一种思路脱壳(Import Recovery)  
> 【文章作者】: wynney  
> 【软件名称】: 某 UnpackMe  
> 【下载地址】: 附件下载  
> 【作者声明】: 这个目标程序似乎只需处理 2 个分支，寻找个处理 IAT 的思路，最后的 IAT 修复也有点不一样  
> 【详细过程】

## 一、前言

ASPr 1.31 用 volx 大狭的脚本脱不了，刚好碰到了这么个东西，试了下，没深研，关于 1.31 版本的教程以前不少大狭都有写，虽看似很复杂但是很详细，本人只是希望本人能起到抛砖之用，目标程序 PEID-Ver 0.15 查得：`Version: ASProtect 1.31 build 04.27 Beta [Extract]`。大家看教程的过程中注意看着色的地方。以免你们的操作会出现错误。

## 二、处理 IAT

忽略所有异常。`he` `GetModuleHandleA`，`Shift+F9`，中断 2 次，`Alt+F9`。找到段尾：

```
00D285C1    61                popad                             ; 直接 F4
00D285C2    75 08             jnz     short 00D285CC
00D285C4    B8 01000000       mov     eax, 1
00D285C9    C2 0C00           retn    0C
00D285CC    68 00000000       push    0
00D285D1    C3                retn
```

`Ctrl+B`：`8A 00 FF 43 08`

```
00D1555B    8A00              mov     al, byte ptr [eax]        ; 找到这里，F2，F9
00D1555D    FF43 08           inc     dword ptr [ebx+8]         ; 中断后删除断点
00D15560    33D2              xor     edx, edx
00D15562    8AD0              mov     dl, al
00D15564    8BC7              mov     eax, edi
```

`Ctrl+B`：`8B 54 24 0C 89 02 8B 44 24 0C 89 06 0F B7 44 24 04 01 43 08`

```
00D1574B    8B5424 0C         mov     edx, dword ptr [esp+C]    ; 找到这里
00D1574F    8902              mov     dword ptr [edx], eax      ; ①ebp 里是 IAT，把 eax 改成 ebp
00D15751    8B4424 0C         mov     eax, dword ptr [esp+C]
00D15755    8906              mov     dword ptr [esi], eax
00D15757    0FB74424 04       movzx   eax, word ptr [esp+4]
00D1575C    0143 08           add     dword ptr [ebx+8], eax
```

`Ctrl+B`：`8B 54 24 0C 89 02 8D 4C 24 0C 8A 54 24 07 8B C3`

```
00D15672    8B5424 0C         mov     edx, dword ptr [esp+C]    ; 找到这里
00D15676    8902              mov     dword ptr [edx], eax      ; ②ebp 里是 IAT，把 eax 改成 ebp
00D15678    8D4C24 0C         lea     ecx, dword ptr [esp+C]
00D1567C    8A5424 07         mov     dl, byte ptr [esp+7]
00D15680    8BC3              mov     eax, ebx
00D15682    E8 F9F7FFFF       call    00D14E80                  ; ③加密 Call，nop 掉
00D15687    8BC6              mov     eax, esi
00D15689    83E8 02           sub     eax, 2
00D1568C    66:8338 00        cmp     word ptr [eax], 0
00D15690    75 10             jnz     short 00D156A2            ; ④改 jmp
```

做了上述 4 次修改后，设置忽略除了内存访问和指定异常之外的所有异常。`Shift+f9`，中断异常，此时 IAT 处理完毕，撤消上述 4 处修改，以免后面出现内存保护错误。继续 `Shift+f9` 到达最后一次异常。

## 三、到达 OEP

```
00D1E56E    0156 00           add     dword ptr [esi], edx      ; 最后一次异常
00D1E571    848D 12FB0F29     test    byte ptr [ebp+290FFB12], cl
00D1E577    BC 2AA5F123       mov     esp, 23F1A52A
```

`Alt+M`，在 Code 段下断，`Shift+f9` 来到 OEP。

```
00401000 >  6A 00             push    0                           ; OEP
00401002    E8 0D010000      call    00401114                    ; Enter 进去
00401007    A3 00304000      mov     dword ptr [403000], eax
```

```
0040110E  - FF25 0804D700    jmp     dword ptr [D70408]          ; kernel32.ExitProcess
00401114    0000             add     byte ptr [eax], al          ; 二进制编辑为 FF25
00401116    5C               pop     esp
00401117    04 D7            add     al, 0D7
00401119    00FF             add     bh, bh
0040111B    25 6804D700      and     eax, 0D70468
00401120    0000             add     byte ptr [eax], al          ; 二进制编辑为 FF25
00401122    74 04            je      short 00401128
00401124    D7               xlat    byte ptr [ebx+al]
00401125    00FF             add     bh, bh
00401127    25 8004D700      and     eax, 0D70480
0040112C  - FF25 9003D700    jmp     dword ptr [D70390]          ; user32.EndPaint
00401132  - FF25 9C03D700    jmp     dword ptr [D7039C]          ; user32.LoadBitmapA
00401138  - FF25 7C3FD700    jmp     dword ptr [D73F7C]          ; user32.SendMessageA
0040113E  - FF25 0843D700    jmp     dword ptr [D74308]          ; GDI32.BitBlt
00401144  - FF25 4445D700    jmp     dword ptr [D74544]          ; GDI32.CreateCompatibleDC
0040114A  - FF25 E845D700    jmp     dword ptr [D745E8]          ; GDI32.DeleteDC
00401150  - FF25 6C48D700    jmp     dword ptr [D7486C]          ; GDI32.DeleteObject
00401156  - FF25 344CD700    jmp     dword ptr [D74C34]          ; GDI32.SelectObject
```

上述 2 处二进制编辑后，真实的 IAT 面貌出现了：）

```
0040110E  - FF25 0804D700    jmp     dword ptr [D70408]          ; kernel32.ExitProcess
00401114  - FF25 5C04D700    jmp     dword ptr [D7045C]          ; kernel32.GetModuleHandleA
0040111A  - FF25 6804D700    jmp     dword ptr [D70468]          ; user32.BeginPaint
00401120  - FF25 7404D700    jmp     dword ptr [D70474]          ; user32.DialogBoxParamA
00401126  - FF25 8004D700    jmp     dword ptr [D70480]          ; user32.EndDialog
0040112C  - FF25 9003D700    jmp     dword ptr [D70390]          ; user32.EndPaint
00401132  - FF25 9C03D700    jmp     dword ptr [D7039C]          ; user32.LoadBitmapA
00401138  - FF25 7C3FD700    jmp     dword ptr [D73F7C]          ; user32.SendMessageA
0040113E  - FF25 0843D700    jmp     dword ptr [D74308]          ; GDI32.BitBlt
00401144  - FF25 4445D700    jmp     dword ptr [D74544]          ; GDI32.CreateCompatibleDC
0040114A  - FF25 E845D700    jmp     dword ptr [D745E8]          ; GDI32.DeleteDC
00401150  - FF25 6C48D700    jmp     dword ptr [D7486C]          ; GDI32.DeleteObject
00401156  - FF25 344CD700    jmp     dword ptr [D74C34]          ; GDI32.SelectObject
```

## 四、修复

大家看到了，IAT 所在的区段是在 `D70000`，我们打开 `ImportREC`，`OEP` 输入 `1000`，点自动查找 IAT，竟然提示 “在此 OEP 处未找到可用的信息”。于是，我们需要特殊处理下了：（

按照下图示操作后，修复脱壳文件，运行看看，脱壳成功

![](img/bf375d42305fee0c44e7b1b52faa8efd.jpg)

![](img/c8337a80b2e1157acfa55da4992022c5.jpg)

![](img/3f0b0994ad11af1d2a3f90dcf957344e.jpg)

用 `FixRes` 和 `Resource Binder` 优化下吧。

> 【经验总结】
> 1、引申到某些壳，如果 IAT 被加密了，请先确定一个 IAT 的地址，对这个地址进行下断，可以找到 IAT 处理的点。
> 2、ASPr 1.31 的有几个分支处理，本目标程序似乎只需处理 2 个分支，而且使用的是很懒的法子，并非某些大狭使用的 Patch 代码的法子，如果有兴趣大家可以看看 fly 原先写的教程。

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!
> 2007 年 04 月 11 日 wynney
