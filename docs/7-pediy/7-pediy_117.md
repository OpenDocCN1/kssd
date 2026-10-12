
# fldrmrkr

> 原文：[`www.pediy.com/kssd/pediy03/forum538.htm`](https://www.pediy.com/kssd/pediy03/forum538.htm)

有一个程序地址为 `http://www.newhua.com.cn/down/fldrmrkr.zip`，它可给电脑中的众多文件加上说明。先是在白菜的论坛上看到有人要求破解，于是下载下来一试，竟然……，只怪我软柿子捏多了，连手都软了，竟然没有捏动，特请各路高手帮忙。

经过一下午的研究发现问题在 `Folderem.dll` 中，关键处在：

```assembly
00406621 55                push ebp
00406622 8BEC              mov ebp, esp
00406624 8B550C            mov edx, dword ptr [ebp+0C]
00406627 8B4508            mov eax, dword ptr [ebp+08]
0040662A 8B08              mov ecx, dword ptr [eax]
0040662C 3B0A              cmp ecx, dword ptr [edx]
0040662E 7522              jne 00406652
00406630 8B4804            mov ecx, dword ptr [eax+04]
00406633 3B4A04            cmp ecx, dword ptr [edx+04]
00406636 751A              jne 00406652
00406638 8B4808            mov ecx, dword ptr [eax+08]
0040663B 3B4A08            cmp ecx, dword ptr [edx+08]
0040663E 7512              jne 00406652
00406640 8B480C            mov ecx, dword ptr [eax+0C]
00406643 3B4A0C            cmp ecx, dword ptr [edx+0C]
00406646 750A              jne 00406652
00406648 8B4010            mov eax, dword ptr [eax+10]
0040664B 8B5210            mov edx, dword ptr [edx+10]
0040664E 3BC2              cmp eax, edx
00406650 7404              je 00406656
00406652 33C0              xor eax, eax
00406654 EB05              jmp 0040665B       <-注意
00406656 B801000000        mov eax, 00000001
0040665B 5D                pop ebp
0040665C C3                ret
```

上面的一些全是比较，如全对最后 `EAX` 就为 `1`，暴破的话只要将 `EB05` 改为 `EB00` 就可以了，它是用 `ASPACK2.1` 加壳的，是用 `UNASPACK1.0.9.1` 能脱，能反汇编，就是不能用。`SMC` 我也不会。注册算法与注册名无关，是用注册码进行计算后与一常数串比较，正算法是看懂了，但逆算法却找不到。只怪我学艺不精，恳请各路高手帮忙。

另：上面的地址是反汇编来的，每次重启动 `WIN` 后 `66XX` 是不变的，但前面的会变，如 `7D66XX`。还有它是在 `WIN` 启动时加载的。不知如何脱光它！

## 声明

我写文章以交流为主，希望大家在转载时能保持文章的完整性。(这一段是从 `ljttt` 那剽窃来的 8-))

## 前言

前天到白菜乐园闲逛，在论坛看到有人请求破解 `Folder Remarker`，它可给电脑中的众多文件加上说明，象我们这种经常下载东西的人尤其用得着。于是马上架起 `jetcar` 把它装回家，抄起 `trw2000` 就剁，结果剁了半天竟然没剁开，只怪我软柿子捏多了，手上没劲，咳……。

## 工具

`IDA4.04`、`trw2000`、`Uedit7.0`

## 破解方法

`SMC`

## 正文

经过一下午的研究发现问题在 `Folderem.dll` 中，关键处在：

```assembly
00406621 55                push ebp
00406622 8BEC              mov ebp, esp
00406624 8B550C            mov edx, dword ptr [ebp+0C]
00406627 8B4508            mov eax, dword ptr [ebp+08]
0040662A 8B08              mov ecx, dword ptr [eax]
0040662C 3B0A              cmp ecx, dword ptr [edx]
0040662E 7522              jne 00406652
00406630 8B4804            mov ecx, dword ptr [eax+04]
00406633 3B4A04            cmp ecx, dword ptr [edx+04]
00406636 751A              jne 00406652
00406638 8B4808            mov ecx, dword ptr [eax+08]
0040663B 3B4A08            cmp ecx, dword ptr [edx+08]
0040663E 7512              jne 00406652
00406640 8B480C            mov ecx, dword ptr [eax+0C]
00406643 3B4A0C            cmp ecx, dword ptr [edx+0C]
00406646 750A              jne 00406652
00406648 8B4010            mov eax, dword ptr [eax+10]
0040664B 8B5210            mov edx, dword ptr [edx+10]
0040664E 3BC2              cmp eax, edx
00406650 7404              je 00406656
00406652 33C0              xor eax, eax
00406654 EB05              jmp 0040665B       <-注意
00406656 B801000000        mov eax, 00000001
0040665B 5D                pop ebp
0040665C C3                ret
```

上面的一些全是比较，如全对最后 `EAX` 就为 `1`，可惜的是，正算法是看懂了，但逆算法却找不到(事实上，它用的是 `SHA` 算法，数论的高才生也搞不定，何况是我，不过如果谁对 `SHA` 算法感兴趣的话，可以分析它的原程序，由于太长我就不贴了)。没办法只有考虑暴破。暴破的话只要将 `EB05` 改为 `EB00` 就可以了，但用 `FI240` 一看，发现它是用 `ASPACK2.1` 加壳的，用 `UNASPACK1.0.9.1` 能脱，能反汇编(上面的就是脱后反汇编来的，要注意的是每次重启动 `WIN` 后地址 `66XX` 是不变的，但前面的 `0040` 会变，如可能是变为 `007D66XX`)，就是不能用(真可惜！)。对 `dll` 文件的手动脱壳我也不会，而且它是在 `WIN` 启动时加载的不知如何断下来(主要是我太菜！)。想来想去只有用 `SMC`，可怜我从没用过，这次是被逼上梁山了！废话少说，马上烧香请出 Crack 宝典——看雪论坛精华。看了半天终于有了一些头绪，于是开始动手。

分析发现 `ASPACK2.1` 加壳的软件开始不久就有一长串的 `00`，而且似乎没有用，不知它放在那干什么？(就等我来 `CRACK` 它？不会吧！)不过刚好放我的代码。

```assembly
00419001                pusha
00419002                call    sub_419579
00419007                jmp     short loc_41903C
00419009                xchg    ebx, ebx
0041900B                nop
0041900C                db      0 ; \
0041900D                db      0 ; \
0041900E                db      0 ; \好长一串 00，
0041900F                db      0 ; \刚好有用。
00419010                db      0 ; \
00419011                db      0 ; \
00419012                db      0 ; \
00419013                db      0 ; \
00419014                db      0 ; \
00419015                db      0 ; \
00419016                db      0 ; \
00419017                db      0 ; \
00419018                db      0 ; \
00419019                db      0 ; \
0041901A                db      0 ; \
0041901B                db      0 ; \
0041901C                db      0 ; \
0041901D                db      0 ; \
0041901E                db      0 ; \
0041901F                db      0 ; \
00419020                db      0 ; \
```

再看尾部:

```assembly
004194DD                mov     eax, [ebp+443A0Eh]
004194E3                push    eax
004194E4                add     eax, [ebp+444798h]
004194EA                pop     ecx
004194EB                or      ecx, ecx
004194ED                mov     [ebp+443E3Bh], eax  <-eax 为返回地址
004194F3                popa                     <-到这儿就该去执行 SMC 代码了，而且不能
004194F4                jnz     short loc_4194FE       先 popa，不然 eax 的值就丢了。
004194F6                mov     eax, 1
004194FB                retn    0Ch
004194FE                push    00000000 <-00 的地址为[ebp+443E3Bh]，执行 4194ED 一句后
00419503                retn                   原来的 00000000 就变为了返回地址
```

还有一个问题，前面说到每次重启动 `WIN` 后，执行地址都不同，有 `7D66XX`、`16766XX` 等，而且不跟踪的话我也不知道那时 `eax` 到底是多少(是不知如何跟踪吧？)，不过这也难不倒我，估计 `EAX` 的高 `4` 位和要修改的地址的高 `4` 位应该是一样的如 `007D`、`0167` 等，那我只要把它的低 `4` 位改为 `6654` 就可以了，这样两句汇编就可以办到了。

修改后反汇编如下:

头部

```assembly
00419001      pusha
00419002      call    sub_419579
00419007      jmp     short loc_41903C
00419009      db      87h ;
0041900A      db      0DBh ;
0041900B      db      90h ;
0041900C      and     eax, FFFF0000h               <-低 4 位清 0
00419012      add     eax, 6654h                  <-低 4 位变为 6654
00419017      mov     dword ptr [eax+0], 01B800EBh <-将 EB05 变为 EB00
0041901E      jmp     loc_4194FD        <-回到出口
```

尾部

```assembly
004194E3      push    eax
004194E4      add     eax, [ebp+444798h]
004194EA      pop     ecx
004194EB      or      ecx, ecx
004194ED      mov     [ebp+443E3Bh], eax
004194F3      jmp     loc_41900C        <-去执行 SMC 代码
004194F8      db      0 ;    \
004194F9      db      0 ;    \ 这些没用
004194FA      db      0 ;    /
004194FB      db      0C2h ; 0 /
004194FC      db      0Ch ;
004194FD      popa                 <-出口
004194FE      push    00000000
00419503      retn
```

收工，试一下一切 `OK`。(需要重启到 `DOS` 下，再用修改好的覆盖)

附：程序下载地址为 `http://www.newhua.com.cn/down/fldrmrkr.zip`

`hying[CCG]` `2001.1.21` `4dd466d0e1e0cc123afbe504da2462ca`
