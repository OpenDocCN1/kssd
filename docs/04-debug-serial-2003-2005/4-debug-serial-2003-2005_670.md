
# 3dlink 1.6 sp2 控件注册算法详解及注册机（手把手教你学注册）

> 原文：[`www.pediy.com/kssd/pediy06/pediy6661.htm`](https://www.pediy.com/kssd/pediy06/pediy6661.htm)

**【破解作者】** `jackily`

**【作者邮箱】** `jackily_zhang@msn.com`；`jackily_zhang@yahoo.com.cn`

**【作者主页】** `http://estudy.ys168.com`

**【使用工具】** `ollydbg` `1.10`

**【破解平台】** `Win9x/NT/2000/XP`

**【软件名称】** `3dlink` `1.6` `sp2` 控件

**【下载地址】**
`http://www.3dlinx.com/downloads/3dlinx/3dlinx16sp2/3dlinx1_6sp2.zip`
`http://estudy.ys168.com` 其它目录中的 `3dlintrail1.5.exe`

**【软件简介】** 这个软件是个控件，被 `VC`，`C++`，`DELPHI` 来调用，可以非常简单地开发三维程序，这个是 `3dlinx` 最新版，安装完毕后全部是 `DLL` 文件。未注册时，加载控件时会有一个 `nag` 窗口，并有 30 天使用限制和 10 个设计窗口限制。`3dlintrail1.5.exe` 是前一版本的测试版，带注册程序，名字是 `LinXRegister.exe`。先安装 `1.6` 再安装 `1.5` 版本，不要安装在一起，就可以有一个独立的程序来注册了。

**【破解声明】** 本破解纯属以学习为目的，偶得一点心得，愿与大家分享！感谢 `Shooo` 提供了注册机的算法思路，本人进行了修改（在总结中有说明）。

---

## 破解内容

经分析，其实真正的注册代码在 `1.6` `sp2` 中的 `poppages.dll` 中。运行 `ollydbg`，加载 `LinxRegister.exe`。点右键—“搜索”—“字符参考”，找到 `ASCII` `"Registration succeeded!"`。地址是 `00401719`，在 `00401717` 跳转处下断，而这段代码源于 `00401650`。按 `F9` 调试，出现注册画面，分四个输入栏：
1.  第一个是 `3dlink` 产品名称（可选）；
2.  第二个是 `computer code entry`（已自动生成，但可以修动）；
3.  第三个是 `ID` 号（根据电脑配置和操作系统自动生成，灰色不可修）；
4.  第四个是注册码（要求用户输入）。

第一栏是 `3DLINK`，第二栏是 `"274925803"`，第三栏是 `"481920"`，在第四栏中，我们输入 `"1234567"`。点注册，在刚才下断的地方停下来。代码分析如下：

```asm
00401650    > 51                push ecx
00401651    .  56                push esi
00401652    .  57                push edi
00401653    .  8BF1              mov esi,ecx
00401655    .  6A 01             push 1
00401657    .  BF 05400080       mov edi,80004005
0040165C    .  E8 7DC60000       call LinXRegi.0040DCDE
00401661    .  8B8E B0000000     mov ecx,dword ptr ds:[esi+B0]
00401667    .  85C9              test ecx,ecx
00401669    .  0F84 F9000000     je LinXRegi.00401768
0040166F    .  8B86 B4000000     mov eax,dword ptr ds:[esi+B4]
00401675    .  8B11              mov edx,dword ptr ds:[ecx]
00401677    .  8D0440            lea eax,dword ptr ds:[eax+eax*2]
0040167A    .  C1E0 02           shl eax,2
0040167D    .  8BB8 C0814100     mov edi,dword ptr ds:[eax+4181C0]
00401683    .  8B80 BC814100     mov eax,dword ptr ds:[eax+4181BC]
00401689    .  57                push edi
0040168A    .  50                push eax
0040168B    .  51                push ecx
0040168C    .  FF52 24           call dword ptr ds:[edx+24]             ;  poppages.01EBA5A5
0040168F    .  8BF8              mov edi,eax
00401691    .  85FF              test edi,edi
00401693    .  /0F8C CF000000    jl LinXRegi.00401768
00401699    .  |8B86 B0000000    mov eax,dword ptr ds:[esi+B0]
0040169F    .  |50                push eax
004016A0    .  |8B08              mov ecx,dword ptr ds:[eax]
004016A2    .  |FF51 0C           call dword ptr ds:[ecx+C]             ;  分析是否已注册，是则到 004016A9，退出。
004016A5    .  |85C0              test eax,eax
004016A7    .  |75 18             jnz short LinXRegi.004016C1           ;  否则跳至 004016c1 继续注册程序
004016A9    .  |50                push eax
004016AA    .  |68 90814100       push LinXRegi.00418190                 ;  ASCII "3DLinX Registration"
004016AF    .  |68 60854100       push LinXRegi.00418560                 ;  ASCII "This product is already registered."
004016B4    .  |8BCE              mov ecx,esi
004016B6    .  |E8 C7BE0000       call LinXRegi.0040D582
004016BB    .  |5F                pop edi
004016BC    .  |33C0              xor eax,eax
004016BE    .  |5E                pop esi
004016BF    .  |59                pop ecx
004016C0    .  |C3                retn
004016C1    > |8B86 A8000000      mov eax,dword ptr ds:[esi+A8]       ;  ASII "1234567"
004016C7    .  |8B96 B0000000     mov edx,dword ptr ds:[esi+B0]       ;  ASII "274925803"
004016CD    .  |8BBE A0000000     mov edi,dword ptr ds:[esi+A0]       ;  ASII "481920"   ,这三个数我们很熟悉吧。
004016D3    .  |53                push ebx
004016D4    .  |8B9E 98000000     mov ebx,dword ptr ds:[esi+98]
004016DA    .  |8D4C24 0C         lea ecx,dword ptr ss:[esp+C]
004016DE    .  |55                push ebp
004016DF    .  |8B2A              mov ebp,dword ptr ds:[edx]
004016E1    .  |51                push ecx
004016E2    .  |50                push eax
004016E3    .  |E8 160E0000       call LinXRegi.004024FE             ;  将 ASII 码变成 HEX 十六进制数
004016E8    .  |83C4 04           add esp,4
004016EB    .  |50                push eax
004016EC    .  |57                push edi
004016ED    .  |E8 0C0E0000       call LinXRegi.004024FE             ;  将 ASII 码变成 HEX 十六进制数
004016F2    .  |83C4 04           add esp,4
004016F5    .  |50                push eax
004016F6    .  |53                push ebx
004016F7    .  |E8 020E0000       call LinXRegi.004024FE             ;  将 ASII 码变成 HEX 十六进制数
004016FC    .  |8B96 B0000000     mov edx,dword ptr ds:[esi+B0]
00401702    .  |83C4 04           add esp,4
00401705    .  |50                push eax
00401706    .  |52                push edx
00401707    .  |FF55 20           call dword ptr ss:[ebp+20]        ;  关键调用，动态加载 poppages.dll 地址：01EBA253，跟进。
0040170A    .  |5D                pop ebp
0040170B    .  |8BF8              mov edi,eax
0040170D    .  |5B                pop ebx
0040170E    .  |85FF              test edi,edi                      ;  比较标志，注册失败跳至 00401750。成功则出现成功画面。这里不
00401710    .  |6A 00             push 0
00401712    .  |68 90814100       push LinXRegi.00418190             ;  ASCII "3DLinX Registration"
00401717    .  |7C 37             jl short LinXRegi.00401750          ;  刚才下断的地方
00401719    .  |68 48854100       push LinXRegi.00418548               ;  ASCII "Registration succeeded!"
0040171E    .  |8BCE              mov ecx,esi
00401720    .  |E8 5DBE0000       call LinXRegi.0040D582
00401725    .  |68 34854100       push LinXRegi.00418534               ;  ASCII "Product Registered"
0040172A    .  |8D8E A4000000     lea ecx,dword ptr ds:[esi+A4]
00401730    .  |E8 72D30000       call LinXRegi.0040EAA7
00401735    .  |6A 00             push 0
00401737    .  |8BCE              mov ecx,esi
00401739    .  |E8 A0C50000       call LinXRegi.0040DCDE
0040173E    .  |8BB6 B0000000     mov esi,dword ptr ds:[esi+B0]
00401744    .  |56                push esi
00401745    .  |8B06              mov eax,dword ptr ds:[esi]
00401747    .  |FF50 28           call dword ptr ds:[eax+28]
0040174A    .  |8BC7              mov eax,edi
0040174C    .  |5F                pop edi
0040174D    .  |5E                pop esi
0040174E    .  |59                pop ecx
0040174F    .  |C3                retn
00401750    > |68 04854100        push LinXRegi.00418504           ;  ASCII "Registration failure. Check input for validity."
00401755    .  |8BCE              mov ecx,esi
00401757    .  |E8 26BE0000       call LinXRegi.0040D582
0040175C    .  |8BB6 B0000000     mov esi,dword ptr ds:[esi+B0]
00401762    .  |56                push esi
00401763    .  |8B06              mov eax,dword ptr ds:[esi]
00401765    .  |FF50 28           call dword ptr ds:[eax+28]
00401768    > \8BC7              mov eax,edi
0040176A    .  5F                pop edi
0040176B    .  5E                pop esi
0040176C    .  59                pop ecx
0040176D    .  C3                retn
```

---

由 `00401707` 跟进。

注意：在实际调试时，尽管在同一机器上，该调用地址会随内存中进程而经常变动。在 `poppages.dll` 中的 `01EBA293` 处有对控件文件的处理，否则只会出现成功画面，但下次还提示注册。



```
01EBA253 55  push ebp
01EBA254 8BEC  mov ebp,esp
01EBA256 83EC 30  sub esp,30
01EBA259 56  push esi
01EBA25A 8B75 08  mov esi,dword ptr ss:[ebp+8]
01EBA25D 57  push edi
01EBA25E 807E 1C 00  cmp byte ptr ds:[esi+1C],0
01EBA262 0F84 B0000000  je poppages.01EBA318
01EBA268 8B7D 10  mov edi,dword ptr ss:[ebp+10]
01EBA26B 393D B02FEF01  cmp dword ptr ds:[1EF2FB0],edi
01EBA271 0F85 A1000000  jnz poppages.01EBA318 ；以下三行是刚才转换的三个 HEX 值
01EBA277 FFB6 68020000  push dword ptr ds:[esi+268] ；ESI+268 是"123456"的 HEX 值“1E240”
01EBA27D 57  push edi ；EDI 为 ID 的 HEX 值“75A80”。
01EBA27E FF75 0C  push dword ptr ss:[ebp+C] ；这里是 CODE ENTRY 的 HEX 值“106308EB”
01EBA281 FF75 14  push dword ptr ss:[ebp+14] ；这里是 0x69930,程序约定。
01EBA284 E8 B6680100  call poppages.01ED0B3F ；关键，调用算法程序，跟进。
01EBA289 8B4D 18  mov ecx,dword ptr ss:[ebp+18]
01EBA28C 83F8 01  cmp eax,1 ；EAX 是否为 1，注册码比较标志。
01EBA28F 8901  mov dword ptr ds:[ecx],eax
01EBA291 75 14  jnz short poppages.01EBA2A7 ；EAX 不为 1,注册码不正确。跳至 poppages.01EBA2A7
01EBA293 6A 04  push 4 ；正确的话，程序对控件进处理，解除注册限制。
01EBA295 58  pop eax
01EBA296 3946 14  cmp dword ptr ds:[esi+14],eax
01EBA299 74 79  je short poppages.01EBA314
01EBA29B 68 90C6EE01  push poppages.01EEC690
01EBA2A0 8946 14  mov dword ptr ds:[esi+14],eax
01EBA2A3 6A 09  push 9
01EBA2A5 EB 59  jmp short poppages.01EBA300
01EBA2A7 83F8 31  cmp eax,31
01EBA2AA 75 28  jnz short poppages.01EBA2D4
01EBA2AC 8D45 08  lea eax,dword ptr ss:[ebp+8]
01EBA2AF 6A 00  push 0
01EBA2B1 50  push eax
01EBA2B2 8D45 10  lea eax,dword ptr ss:[ebp+10]
01EBA2B5 50  push eax
01EBA2B6 8D45 0C  lea eax,dword ptr ss:[ebp+C]
01EBA2B9 50  push eax
01EBA2BA E8 42680100  call poppages.01ED0B01
01EBA2BF FF75 08  push dword ptr ss:[ebp+8]
01EBA2C2 FF75 10  push dword ptr ss:[ebp+10]
01EBA2C5 FF75 0C  push dword ptr ss:[ebp+C]
01EBA2C8 6A 05  push 5
01EBA2CA FF76 18  push dword ptr ds:[esi+18]
01EBA2CD E8 48660100  call poppages.01ED091A
01EBA2D2 EB 40  jmp short poppages.01EBA314
01EBA2D4 83F8 32  cmp eax,32
01EBA2D7 75 31  jnz short poppages.01EBA30A
01EBA2D9 8B86 68020000  mov eax,dword ptr ds:[esi+268]
01EBA2DF 8B8E 6C020000  mov ecx,dword ptr ds:[esi+26C]
01EBA2E5 50  push eax
01EBA2E6 8D9401 40E20100  lea edx,dword ptr ds:[ecx+eax+1E240]
01EBA2ED 52  push edx
01EBA2EE 51  push ecx
01EBA2EF 50  push eax
01EBA2F0 8D45 D0  lea eax,dword ptr ss:[ebp-30]
01EBA2F3 57  push edi
01EBA2F4 50  push eax
01EBA2F5 E8 2E650100  call poppages.01ED0828
01EBA2FA 8D45 D0  lea eax,dword ptr ss:[ebp-30]
01EBA2FD 50  push eax
01EBA2FE 6A 0A  push 0A
01EBA300 FF76 18  push dword ptr ds:[esi+18]
01EBA303 E8 255E0100  call poppages.01ED012D
01EBA308 EB 0A  jmp short poppages.01EBA314
01EBA30A 83F8 01  cmp eax,1
01EBA30D 7E 09  jle short poppages.01EBA318
01EBA30F 83F8 31  cmp eax,31
01EBA312 7D 04  jge short poppages.01EBA318
01EBA314 33C0  xor eax,eax
01EBA316 EB 05  jmp short poppages.01EBA31D
01EBA318 B8 05400080  mov eax,80004005
01EBA31D 5F  pop edi
01EBA31E 5E  pop esi
01EBA31F C9  leave
01EBA320 C2 1400  retn 14
--------------------------------------------------------------------------------
```

### 由 01EBA284 跟进，关键算法。

```
01ED0B3F 55  push ebp
01ED0B40 8B5424 10  mov edx,dword ptr ss:[esp+10] ；把 ID 的 HEX 值“75A80”给 EDX
01ED0B44 8BEC  mov ebp,esp
01ED0B46 56  push esi
01ED0B47 57  push edi
01ED0B48 8345 14 34  add dword ptr ss:[ebp+14],34 ；把 0x69930 加 0x34
01ED0B4C 83FA 01  cmp edx,1
01ED0B4F 7D 05  jge short poppages.01ED0B56
01ED0B51 BA 01000000  mov edx,1
01ED0B56 8B7D 0C  mov edi,dword ptr ss:[ebp+C] ；把 CODE ENTRY 的 HEX 值“106308EB”给 EDI
01ED0B59 8BCF  mov ecx,edi ；再分别赋于 ECX 和 EAX
01ED0B5B 8BC7  mov eax,edi
01ED0B5D 81E1 00E00700  and ecx,7E000 ；ECX 与 0x7E000
01ED0B63 25 001F0000  and eax,1F00 ；EAX 与 0x1F00
01ED0B68 C1E9 0D  shr ecx,0D ；ECX 右移 0x0D
01ED0B6B C1E8 08  shr eax,8 ；EAX 右移 0x8
01ED0B6E 81C1 BC070000  add ecx,7BC ；ECX 加 0x7BC
01ED0B74 6BC9 44  imul ecx,ecx,44 ；ECX 乘 0x44
01ED0B77 69C0 F3000000  imul eax,eax,0F3 ；EAX 乘 0x0F3
01ED0B7D 03C8  add ecx,eax ；ECX 加 EAX 的值放入 ECX 中
01ED0B7F 8BC7  mov eax,edi ；再次把 CODE ENTRY 的 HEX 值“106308EB”给 EAX
01ED0B81 25 00007800  and eax,780000 ；与 0X780000
01ED0B86 C1E8 13  shr eax,13 ；右移 0X13
01ED0B89 69C0 08010000  imul eax,eax,108 ；乘 0X108
01ED0B8F 8D3401  lea esi,dword ptr ds:[ecx+eax] ；把 ECX 和 EAX 的值相加放入 ESI 中
01ED0B92 8BCF  mov ecx,edi ；再次把 CODE ENTRY 的 HEX 值“106308EB”给 ECX
01ED0B94 81E1 0000807F  and ecx,7F800000 ；与 0X7F800000
01ED0B9A 81E7 FF000000  and edi,0FF ；EDI 与 0X0FF
01ED0BA0 C1E9 0F  shr ecx,0F ；ECX 右移 0X0F
01ED0BA3 8B45 14  mov eax,dword ptr ss:[ebp+14] ；把 01ED0B48 语句中的 0X69964 赋予 EAX
01ED0BA6 6BC0 07  imul eax,eax,7 ；乘 0X7
01ED0BA9 03CF  add ecx,edi ；ECX 加 EDI 的值给 ECX
01ED0BAB 6BC9 03  imul ecx,ecx,3 ；ECX 乘 0X3
01ED0BAE 03C8  add ecx,eax ；ECX 加 EAX 的值给 ECX
01ED0BB0 B8 01000000  mov eax,1 ；赋予 EAX1，目的是给注册标志。表示成功
01ED0BB5 8D0C51  lea ecx,dword ptr ds:[ecx+edx*2] ；把 ECX 加上 EDX 乘 2 的值给 ECX
01ED0BB8 8BD6  mov edx,esi
01ED0BBA 6BD2 1F  imul edx,edx,1F ；EDX 乘 0X1F
01ED0BBD 03D1  add edx,ecx ；EDX 加 ECX
01ED0BBF 8D0C75 00000000  lea ecx,dword ptr ds:[esi*2]
01ED0BC6 8BF2  mov esi,edx ；EDX 给 ESI
01ED0BC8 81E6 FFFFFF7F  and esi,7FFFFFFF ；ESI 与 0X7FFFFFFF，换算完毕。
```



# 汇编代码分析

```
01ED0BCE    3B75 08            cmp esi,dword ptr ss:[ebp+8]      ；关键比较，还记得 `EBP+8` 的内容吗？这里是“1E240”，也就是把
                                                              ; 换算的值和我们输入的“123456”的 `HEX` 值比较。`ESI` 内的值为“
                                                              ; 802A35” ，换算为十进制为“8399413”。

01ED0BD1    74 0A              je short poppages.01ED0BDD         ；成功跳至 `01ED0BDD`（这里是真正的爆破点）

01ED0BD3    03D1               add edx,ecx

01ED0BD5    40                 inc eax

01ED0BD6    83F8 32            cmp eax,32

01ED0BD9  ^ 7E EB             jle short poppages.01ED0BC6

01ED0BDB    33C0               xor eax,eax                        ；给 `EAX` 赋予 0，表示注册不成功

01ED0BDD    5F                 pop edi

01ED0BDE    5E                 pop esi

01ED0BDF    5D                 pop ebp

01ED0BE0    C2 1000            retn 10
```

## 破解总结

该控件的难点在于动态加载 `poppages.dll`，有三个关键调用。不能用 `keymaker` 来作内存注册机，因为在代码分析中指出，`poppages.dll` 的加载是动态的，地址是随时可变的，也就是说做出的内存注册机不能通用，只能用一次。因此爆破点应选在 `poppages.dll` 文件中（`w32dasm` 和 `hiew` 中的实际地址是 `10050BD1`）。还需要指出的是在写注册机的时候，以上 `01ED0B5D`、`01ED0B81`、`01ED0B94` 中 `0X7E000`、`0X780000` 和 `0X7F800000` 中的 0 必须写完整，还有必须用 `unsigned long`（无符号，长整数）定义变量，否则不能得出正确结果。另外在 `01ED0BD3` 至 `01ED0BD9` 处的循环可以不用考虑。这就是在【声明】中我提到的修改了的注册机算法。

### 提供一组注册码

* **注 册 项 目：** `3DLINK`
* **CODE ENTRY：** `274925803`
* **COMPUTER ID：** `481920`
* **SN：** `8399413`

## 算法注册机

```c
/* 3Dlink 1.6 sp2 算法注册机 */
/* 在 Turboc 2.0 下调试通过  */
/* 感谢 Shooo 提供算法思路    */

#include "stdio.h"

main()
{ unsigned long code, id;
  unsigned long i, j, k, l, m;
  printf("3DLINK 1.6 SP2 Register by jackily 2004-12-11\n");
  printf("Email:jackily_zhang@msn.com or jackily_zhang@yahoo.com.cn\n");
  printf("please input code entry:");
  scanf("%lu",&code);
  printf("\nplease input computer id:");
  scanf("%lu",&id);
  m=id;
  i=j=k=code;
  i&=0x7e000; i>>=0x0d; i+=0x7bc; i*=0x44;
  j&=0x1f00; j>>=0x08; j*=0xf3;
  i+=j;
  j=code;
  j&=0x780000; j>>=0x13; j*=0x108;
  l=i+j;
  i=code;
  i&=0x7f800000; i>>=0xf;
  k&=0xff;
  i+=k;
  i*=0x3;
  j=0x69964*0x7;
  i+=j;
  m*=2;
  i+=m;
  m=l; m*=0x1f;
  m+=i;
  l=m;
  l&=0x7fffffff;
  printf("\nserial number is %lu", l);
}
```

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！ `2004-12-11`
`4dd466d0e1e0cc123afbe504da2462ca`
