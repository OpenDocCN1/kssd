
# [原创]UXXX?毒专家 单机版 内存注册机制作+算法分析

## 前言

以前一直喜欢搞爆破，前几天还看到一个帖子说的是"UXXX?毒专家"的破解，一看过程真的是简单到爆！除了那个作者说的可以搜索字符串“已注册”的方法以外，打开它的语言ini文件还可以看到一些重要的信息：这个软件用数字代替了字符串，所以只要在语言文件里面搜索想要的内容，然后按照它的对应数字就可以到程序里面搜到代码了！本来想补充方法的，但是后来一想别人都已经发过了，那我就不重复提爆破了吧，来看看内存注册机的制作和算法分析。

因为是第一次搞算法，所以弄的程序是比较简单(是我看到程序注册码简单的样子我才敢去尝试)，高手勿喷!

## 破解

### Part 1 第一部分，编写内存注册机

由于这个软件的注册码在内存里是会明码出现的(不会吧...但确实是这样的...)，所以我们可以做内存注册机.

首先，我们为达到目的，最快方法先查找字符串，找到如下：

```
Ultra 字符串参考，项目 3800
 地址=0057FD32
 反汇编=mov edx,XXXXXXXX.0057FE14
 文本字符串=已注册
```

好的，双击看看：

```
0057FCA0  /$  55            push ebp
0057FCA1  |.  8BEC          mov ebp,esp
0057FCA3  |.  33C9          xor ecx,ecx
0057FCA5  |.  51            push ecx
0057FCA6  |.  51            push ecx
0057FCA7  |.  51            push ecx
0057FCA8  |.  51            push ecx
0057FCA9  |.  51            push ecx
0057FCAA  |.  51            push ecx
0057FCAB  |.  51            push ecx
0057FCAC  |.  53            push ebx
0057FCAD  |.  56            push esi
0057FCAE  |.  57            push edi
0057FCAF  |.  8BD8          mov ebx,eax
0057FCB1  |.  33C0          xor eax,eax
0057FCB3  |.  55            push ebp
0057FCB4  |.  68 FBFD5700   push XXXXXXXX.0057FDFB
0057FCB9  |.  64:FF30       push dword ptr fs:[eax]
0057FCBC  |.  64:8920       mov dword ptr fs:[eax],esp
0057FCBF  |.  B2 01         mov dl,1
0057FCC1  |.  A1 3CEB4D00   mov eax,dword ptr ds:[4DEB3C]
0057FCC6  |.  E8 99EFF5FF   call XXXXXXXX.004DEC64
0057FCCB  |.  8BF0          mov esi,eax
0057FCCD  |.  8D55 F4       lea edx,dword ptr ss:[ebp-C]
0057FCD0  |.  8BC6          mov eax,esi
0057FCD2  |.  E8 D9F9F5FF   call XXXXXXXX.004DF6B0
0057FCD7  |.  8B55 F4       mov edx,dword ptr ss:[ebp-C]
0057FCDA  |.  8B83 3C040000 mov eax,dword ptr ds:[ebx+43C]
0057FCE0  |.  E8 3B3BEDFF   call XXXXXXXX.00453820
0057FCE5  |.  8BC6          mov eax,esi
0057FCE7  |.  E8 00F6F5FF   call XXXXXXXX.004DF2EC                     ;  关键Call
0057FCEC  |.  2C 01         sub al,1                                   ;  Switch (cases 0..3)
0057FCEE  |.  72 0D         jb short XXXXXXXX.0057FCFD
0057FCF0  |.  2C 02         sub al,2
0057FCF2  |.  0F84 B5000000 je XXXXXXXX.0057FDAD
0057FCF8  |.  E9 DC000000   jmp XXXXXXXX.0057FDD9
0057FCFD  |>  8B83 1C040000 mov eax,dword ptr ds:[ebx+41C]             ;  Case 0 of switch 0057FCEC
0057FD03  |.  E8 7C71EDFF   call XXXXXXXX.00456E84
0057FD08  |.  8BF8          mov edi,eax
0057FD0A  |.  4F            dec edi
0057FD0B  |.  85FF          test edi,edi
0057FD0D  |.  7C 23         jl short XXXXXXXX.0057FD32
0057FD0F  |.  47            inc edi
0057FD10  |.  C745 F8 00000>mov dword ptr ss:[ebp-8],0
0057FD17  |>  8B55 F8       /mov edx,dword ptr ss:[ebp-8]
0057FD1A  |.  8B83 1C040000 |mov eax,dword ptr ds:[ebx+41C]
0057FD20  |.  E8 2371EDFF   |call XXXXXXXX.00456E48
0057FD25  |.  33D2          |xor edx,edx
0057FD27  |.  8B08          |mov ecx,dword ptr ds:[eax]
0057FD29  |.  FF51 64       |call dword ptr ds:[ecx+64]
0057FD2C  |.  FF45 F8       |inc dword ptr ss:[ebp-8]
0057FD2F  |.  4F            |dec edi
0057FD30  |.^ 75 E5         \jnz short XXXXXXXX.0057FD17
0057FD32  |>  BA 14FE5700   mov edx,XXXXXXXX.0057FE14                  ;  已注册 //我们到了这里
0057FD37  |.  8B83 34040000 mov eax,dword ptr ds:[ebx+434]
0057FD3D  |.  E8 DE3AEDFF   call XXXXXXXX.00453820
0057FD42  |.  8D55 F0       lea edx,dword ptr ss:[ebp-10]
0057FD45  |.  8BC6          mov eax,esi
0057FD47  |.  E8 ACFDF5FF   call XXXXXXXX.004DFAF8
0057FD4C  |.  8B55 F0       mov edx,dword ptr ss:[ebp-10]
0057FD4F  |.  8B83 40040000 mov eax,dword ptr ds:[ebx+440]
0057FD55  |.  E8 C63AEDFF   call XXXXXXXX.00453820
0057FD5A  |.  C645 FF 01    mov byte ptr ss:[ebp-1],1
0057FD5E  |.  8D55 EC       lea edx,dword ptr ss:[ebp-14]
0057FD61  |.  8BC6          mov eax,esi
0057FD63  |.  E8 00FDF5FF   call XXXXXXXX.004DFA68
0057FD68  |.  8B55 EC       mov edx,dword ptr ss:[ebp-14]
0057FD6B  |.  8B83 40060000 mov eax,dword ptr ds:[ebx+640]
0057FD71  |.  E8 AA3AEDFF   call XXXXXXXX.00453820
0057FD76  |.  8D55 E8       lea edx,dword ptr ss:[ebp-18]
0057FD79  |.  B8 24FE5700   mov eax,XXXXXXXX.0057FE24                  ;  ASCII "66"
0057FD7E  |.  E8 FD05F9FF   call XXXXXXXX.00510380
0057FD83  |.  8B55 E8       mov edx,dword ptr ss:[ebp-18]
0057FD86  |.  8B83 98050000 mov eax,dword ptr ds:[ebx+598]
0057FD8C  |.  E8 8F3AEDFF   call XXXXXXXX.00453820
0057FD91  |.  33D2          xor edx,edx
0057FD93  |.  8B83 9C050000 mov eax,dword ptr ds:[ebx+59C]
0057FD99  |.  E8 7239EDFF   call XXXXXXXX.00453710
0057FD9E  |.  33D2          xor edx,edx
0057FDA0  |.  8B83 A0050000 mov eax,dword ptr ds:[ebx+5A0]
0057FDA6  |.  E8 6539EDFF   call XXXXXXXX.00453710
0057FDAB  |.  EB 2C         jmp short XXXXXXXX.0057FDD9
0057FDAD  |>  C645 FF 00    mov byte ptr ss:[ebp-1],0                  ;  Case 3 of switch 0057FCEC
0057FDB1  |.  8D55 E4       lea edx,dword ptr ss:[ebp-1C]
0057FDB4  |.  B8 30FE5700   mov eax,XXXXXXXX.0057FE30                  ;  ASCII "67"
0057FDB9  |.  E8 C205F9FF   call XXXXXXXX.00510380
0057FDBE  |.  8B55 E4       mov edx,dword ptr ss:[ebp-1C]
0057FDC1  |.  8B83 98050000 mov eax,dword ptr ds:[ebx+598]
0057FDC7  |.  E8 543AEDFF   call XXXXXXXX.00453820
0057FDCC  |.  B2 01         mov dl,1
0057FDCE  |.  8B83 9C050000 mov eax,dword ptr ds:[ebx+59C]
0057FDD4  |.  E8 3739EDFF   call XXXXXXXX.00453710
0057FDD9  |>  8BC6          mov eax,esi                                ;  Default case of switch 0057FCEC
0057FDDB  |.  E8 4843E8FF   call XXXXXXXX.00404128
0057FDE0  |.  33C0          xor eax,eax
0057FDE2  |.  5A            pop edx
0057FDE3  |.  59            pop ecx
0057FDE4  |.  59            pop ecx
0057FDE5  |.  64:8910       mov dword ptr fs:[eax],edx
0057FDE8  |.  68 02FE5700   push XXXXXXXX.0057FE02
0057FDED  |>  8D45 E4       lea eax,dword ptr ss:[ebp-1C]
0057FDF0  |.  BA 05000000   mov edx,5
0057FDF5  |.  E8 DE51E8FF   call XXXXXXXX.00404FD8
0057FDFA  \.  C3            retn
```

如图我们到达的位置，向上不难找到一个关键可能包含算法的`Call`.(如所示)

于是我们在那个`Call`上`F2`下断重新来过。

`F7`进入`Call`，得到：
