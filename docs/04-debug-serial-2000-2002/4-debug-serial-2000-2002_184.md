
# Acrobat Reader 5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1221.htm`](https://www.pediy.com/kssd/pediy03/forum1221.htm)

Acrobat Reader 5.0 的反跟踪代码部分

HI，大家好！已经很久没有帖东西了！由于比较忙！（其实是懒^\_^）！

今天想用 Acrobat Reader 5.0 看看书！没想到启动后显示了一个位图，做一些初使化后就不见了踪影！一连试了几次！嗯？？我的机子不会烂到连这个软件都不能运行的地步吧！想了想。。。。。噢！是不是又是 `SICE` 在捣鬼！由于我平时总喜欢挂着它！总会引起一些不必要的麻烦！唉！没办法，运行 `frogsice` 挡一挡吧！咦！我倒~~~~~~~~不管用！气死我了！追追它！看来是怎么发现我的宝贝的！

经过一番跟踪，发现反 `SICE` 的代码在 `DocBox.api` 这个插件中！

好吧。继续...........

```
:37020654 0F014DD6                  sidt [ebp-2A]           ************
:37020658 8A45CC                    mov al, byte ptr [ebp-34]
:3702065B 84C0                      test al, al
:3702065D 0F84D8430100              je 37034A3B

:37034A3B 33C0                      xor eax, eax
:37034A3D 8B55D0                    mov edx, dword ptr [ebp-30]
:37034A40 397A20                    cmp dword ptr [edx+20], edi
:37034A43 0F95C0                    setne al
:37034A46 3BC7                      cmp eax, edi
:37034A48 740A                      je 37034A54
:37034A4A 6A02                      push 00000002
:37034A4C E8DE9FFDFF                call 3700EA2F
:37034A51 83C404                    add esp, 00000004
:37034A54 33C0                      xor eax, eax
:37034A56 8B4DD8                    mov ecx, dword ptr [ebp-28]     得到中断描述符表的基地址
:37034A59 668B5144                  mov dx, word ptr [ecx+44]       得到偏移值为 44h(68d)的两个字节
:37034A5D 6681F20005                xor dx, 0500
:37034A62 F6C61F                    test dh, 1F
:37034A65 7505                      jne 37034A6C
:37034A67 B801000000                mov eax, 00000001
:37034A6C 3BC7                      cmp eax, edi
:37034A6E 740A                      je 37034A7A
:37034A70 6A03                      push 00000003
:37034A72 E8B89FFDFF                call 3700EA2F
:37034A77 83C404                    add esp, 00000004
```

当在读取中断描述符表偏移值为 `44h` 的两个字节时，值为 `8500h`!不管它，继续跟下去！已经离死不远了！`F10` 带过 `call 3700EA2F` 时！就退出程序了！里面调用了 `EXITPROCESS`！那么只有向上找找看了！

其实有经验的人早就应该知道！在执行 `SIDT` 这条命令的时候，就应该和咱们的目标不远了！一般的程序不可能用到这个指令！`TEST` 指令后，强制跳跃！避开那个 `CALL` 指令后来到了下面

.........

```
:37034AE1 6A03                      push 00000003
:37034AE3 53                        push ebx
:37034AE4 53                        push ebx
:37034AE5 53                        push ebx
:37034AE6 8B4D8C                    mov ecx, dword ptr [ebp-74]
:37034AE9 51                        push ecx          <== //./SICE
:37034AEA FF1524210537              call dword ptr [37052124]   <===CreateFilea
:37034AF0 8BF0                      mov esi, eax
:37034AF2 8B558C                    mov edx, dword ptr [ebp-74]
:37034AF5 85D2                      test edx, edx
:37034AF7 7433                      je 37034B2C
:37034AF9 8BFA                      mov edi, edx
:37034AFB 83C9FF                    or ecx, FFFFFFFF
:37034AFE 33C0                      xor eax, eax
:37034B00 F2                       repnz
:37034B01 AE                       scasb
:37034B02 F7D1                      not ecx
:37034B04 49                       dec ecx
:37034B05 B85A7DBDBC               mov eax, BCBD7D5A
:37034B0A 8D80623FFFFF             lea eax, dword ptr [eax+FFFF3F62]
:37034B10 8BFA                      mov edi, edx
:37034B12 8BD1                      mov edx, ecx
:37034B14 C1E902                   shr ecx, 02
:37034B17 F3                       repz
:37034B18 AB                       stosd
:37034B19 8BCA                      mov ecx, edx
:37034B1B 83E103                   and ecx, 00000003
:37034B1E F3                       repz
:37034B1F AA                       stosb
:37034B20 8B458C                   mov eax, dword ptr [ebp-74]
:37034B23 50                       push eax
:37034B24 E810030000               call 37034E39
:37034B29 83C404                   add esp, 00000004
:37034B2C 83FEFF                   cmp esi, FFFFFFFF    <======R U OK?
:37034B2F 740C                     je 37034B3D      <==jump is OK!
:37034B31 56                       push esi

* Reference To: KERNEL32.CloseHandle, Ord:001Bh
                                  |
:37034B32 FF156C330637             Call dword ptr [3706336C]
:37034B38 BBCA104000               mov ebx, 004010CA
:37034B3D 85DB                     test ebx, ebx
```

大家已经看到了！这又是一个检测 `SICE` 的方法！也是最常用的！但如果你直接用 `CREATEFILEA` 来设断拦截是没用的！这个程序是用 `GETPROCADDRESS` 来找入口地址来调用的！玩了一个小把戏！有些病毒就是这么做的！

好了，到此也就差不多了！剩下的事就简单了！改改它就 `OK` 了！可有个问题还不清楚！读中断描术符表第 68 的两个字节有什么用？？问我？？我也不清楚！请高手明示吧！！！！！！

```
.386

.model flat, stdcall

option casemap :none

include    /masm32/include/windows.inc
include   /masm32/include/user32.inc
includelib   /masm32/lib/user32.lib
include    /masm32/include/kernel32.inc
includelib   /masm32/lib/kernel32.lib

.data
format db "%x",0

.data?
char  db  4 dup(?)

.code
start:
 sidt   [esp-2]
 pop    eax
 xor     ebx,ebx
 mov     bx,word ptr [eax+44h]
 invoke wsprintf,addr char,addr format,ebx
 invoke MessageBox,0,addr char,0,MB_OK
 invoke ExitProcess,0
end start
```

不过我做了个实验：用 `MASM32` 编了上面的程序！来实现读出 `IDT` 的第 68 处的两个字节！我试过了，在没挂 `SICE` 的情况下是 `8E00h`,在挂 `SICE` 的情况下是 `8500h`！难不成这是 `SICE` 驻留的标志？？有意思！！请哪位高手回答一下吧！

全文完！

`garfield`

`China Cracking Group`

`2001.8.14` `4dd466d0e1e0cc123afbe504da2462ca`
