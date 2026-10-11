
# 带壳调试《XXX 面试真题 600 道详解》

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-757.htm`](https://www.pediy.com/kssd/pediy07/pediy7-757.htm)

带壳调试《XXX 面试真题 600 道详解》

1. 试运行：

机器码：`CD48047047390`

注册码：`87654321`

提示：密码已存储，在需要的时候调用。

2. 上 `RegMon.exe`：

```
12.68640137   gwyinterview.ex:1448   OpenKey   HKCU\CLSID\{242A8E4F-A261-2211-2211-00C04FB9603F}\05110wsyy7120801   NOT FOUND     
12.68645573   gwyinterview.ex:1448   SetValue  HKCR\CLSID\{242A8E4F-A261-2211-2211-00C04FB9603F}\05110wsyy7120801\yyzcm   SUCCESS  "636877907888216"   
12.68652916   gwyinterview.ex:1448   CloseKey  HKCR\CLSID\{242A8E4F-A261-2211-2211-00C04FB9603F}\05110wsyy7120801   SUCCESS     
```

3. 查壳脱壳：

`ASPack 2.12 -> Alexey Solodnikov [Overlay]`

`OD` 载入，取消继续分析：

```
00574001  g>pushad
00574002    call gwyinter.0057400A
00574007    jmp 45B444F7
...
005743AF    popad
005743B0    jnz short gwyinter.005743BA
005743B2    mov eax,1
005743B7    retn 0C
005743BA    push gwyinter.0051DD7C
005743BF    retn        返回到 0051DD7C (gwyinter.0051DD7C)
```

`dump`，`Borland Delphi 4.0 - 5.0`

脱壳后一运行就闪过一个框，然后退出。

估计有自校验程序。

4. 自校验：`KERNEL32.CreateFileA`

注意堆栈，一直 `F9` 运行，看到主程序出现就停按 `F9`，在：

```
0040696F   call <jmp.&kernel32.CreateFileA>     
```

有两次调用，脱壳与未脱壳进行分别跟踪，第一次调用都是一样处理，第二次调用后 `F8` 往下走，来到一个大循环，次数狂多，未脱壳的循环次数 `ebx`＝`1F4`，脱壳后的循环次数 `ebx`＝`17A`：

```
004E3312    > /8D45 E8           lea eax,dword ptr ss:[ebp-18]
004E3315    . |50              push eax
004E3316    . |8D95 E0EFFFFF    lea edx,dword ptr ss:[ebp-1020]
004E331C    . |B9 00100000      mov ecx,1000
004E3321    . |8D85 48EDFFFF    lea eax,dword ptr ss:[ebp-12B8]
004E3327    . |E8 4832F2FF      call upgwy.00406574
004E332C    . |E8 43F5F1FF      call upgwy.00402874
004E3331    . |8B55 E8          mov edx,dword ptr ss:[ebp-18]
004E3334    . |85D2            test edx,edx
004E3336    . |7E 11           jle short upgwy.004E3349
004E3338    . |8D85 E0EFFFFF    lea eax,dword ptr ss:[ebp-1020]
004E333E    > |33C9           xor ecx,ecx
004E3340    . |8A08          mov cl,byte ptr ds:[eax]
004E3342    . |014D EC       add dword ptr ss:[ebp-14],ecx
004E3345    . |40            inc eax
004E3346    . |4A            dec edx
004E3347    .^|75 F5         jnz short upgwy.004E333E
004E3349    > |4B            dec ebx      循环指针，估计是计算文件大小
004E334A    .^\\75 C6        jnz short upgwy.004E3312
```

退出循环后往下继续走，发现在下面这个地方：

```
004E3366    .  8B45 E4         mov eax,dword ptr ss:[ebp-1C]
004E3369    .  3B45 EC         cmp eax,dword ptr ss:[ebp-14]
  脱壳后：堆栈 ss:[0012FA1C]=094BBC79   eax=0EDC2595
  未脱壳：堆栈 ss:[0012FA1C]=0EDC2595   eax=0EDC2595

004E336C       75 13           jnz short upgwy.004E3381
```

修改 `004E336C` 处的跳转，然后 `F9` 运行，`ok`，软件加载成功。

5. 脱壳爆破后运行：

提示未注册，需要 `key` 文件。

但是未脱壳的确没有这个提示，决定带壳调试。

6. 如 5 所示，那么注册表里写入的东西没用？

6.1 `OD` 加载未脱壳的程序，先运行到 `oep`，然后下断 `bpx ADVAPI32.RegQueryValueExA`，`F9` 运行，程序都加载完毕后还是没有在堆栈里见到注册码，于是在程序里点击需要注册才能看的项目，断下，堆栈出现 `yyzcm`，`F9` 再一次注册码也出现了`"636877907888216"`。靠，看来是关键时刻才判断。

小插曲：程序断下的时候，画面鼠标挪不动，要 `ctrl＋alt＋delete` 后再取消就可以了。

6.2 从断点返回调用的 `call`，接着 `F8` 步进，期间出现狂多的注册码，不过没怎么计算，注意代码下的信息窗口，一直到出现机器码（软件一般根据机器码计算，所以跟着它没错，至于注册码，可能会有很多假动作）：

```
004CBBE6   mov eax,dword ptr ds:[eax+1003B8]  ds:[014A03BC]=00E4AD30, (ASCII "CD48047047390")
004CBBEC   call gwyinter.00409390     判断是否非字母数字的字符串
004CBBF1   mov ecx,dword ptr ss:[ebp-198]  ds:[014A03BC]=00E4AD30, (ASCII "CD48047047390")
004CBBF7   mov eax,dword ptr ss:[ebp-4]
004CBBFA   mov edx,dword ptr ds:[eax+100398]  ds:[014A039C]=00E4C484, (ASCII "SHou0597!@")
004CBC00   mov eax,dword ptr ss:[ebp-4]
004CBC03   call gwyinter.004E37F0
```

这个 `call` 太像是关键计算，两个字符串 `ecx`，`edx`，要跟。

6.3 进该 `call` 看看：

```
004CBC03     E8 E87B0100      call gwyinter.004E37F0
004E37F0   push ebp
004E37F1   mov ebp,esp
004E37F3   add esp,-24
004E37F6   push ebx
004E37F7   push esi
004E37F8   push edi
004E37F9   xor ebx,ebx
004E37FB   mov dword ptr ss:[ebp-24],ebx
004E37FE   mov dword ptr ss:[ebp-1C],ebx
004E3801   mov dword ptr ss:[ebp-C],ecx
004E3804   mov dword ptr ss:[ebp-8],edx
004E3807   mov dword ptr ss:[ebp-4],eax
004E380A   mov eax,dword ptr ss:[ebp-8]
004E380D   call gwyinter.00404100
004E3812   mov eax,dword ptr ss:[ebp-C]
004E3815   call gwyinter.00404100
004E381A   xor eax,eax
004E381C   push ebp
004E381D   push gwyinter.004E39E8
004E3822   push dword ptr fs:[eax]
004E3825   mov dword ptr fs:[eax],esp
004E3828   mov eax,dword ptr ss:[ebp-4]
004E382B   call gwyinter.004E058C
004E3830   mov dword ptr ss:[ebp-18],42
004E3837   xor eax,eax
004E3839   mov dword ptr ss:[ebp-10],eax
004E383C   lea eax,dword ptr ss:[ebp-1C]
004E383F   call gwyinter.00403CCC
004E3844   xor eax,eax
004E3846   mov dword ptr ss:[ebp-20],eax
004E3849   lea eax,dword ptr ss:[ebp-C]
004E384C   push eax
004E384D   mov eax,dword ptr ss:[ebp-C]    机器码＝(ASCII "CD48047047390")
004E3850   call gwyinter.00403F4C    机器码长度
004E3855   mov ecx,eax
004E3857   mov edx,5
004E385C   mov eax,dword ptr ss:[ebp-C]
004E385F   call gwyinter.00404154    从机器码第 5 位开始截取后面的值＝047047390
004E3864   mov eax,dword ptr ss:[ebp-C]    00E15968  30 34 37 30 34 37 33 39 30 047047390
004E3867   call gwyinter.00403F4C    得到 047047390 的长度＝9
004E386C   mov esi,eax
004E386E   test esi,esi
004E3870   jle short gwyinter.004E3886
004E3872   mov ebx,1
004E3877   mov eax,dword ptr ss:[ebp-C]
004E387A   movzx eax,byte ptr ds:[eax+ebx-1]
004E387F   add dword ptr ss:[ebp-10],eax  求和 047047390＝1D2＝ss:[ebp-10]
004E3882   inc ebx
004E3883   dec esi
004E3884   jnz short gwyinter.004E3877
004E3886   xor eax,eax
004E3888   mov dword ptr ss:[ebp-14],eax
004E388B   mov eax,dword ptr ss:[ebp-8]    字符串＝(ASCII "SHou0597!@")
004E388E   call gwyinter.00403F4C    字符串长度
004E3893   mov esi,eax
004E3895   test esi,esi
004E3897   jle short gwyinter.004E38AD
004E3899   mov ebx,1
004E389E   mov eax,dword ptr ss:[ebp-8]
004E38A1   movzx eax,byte ptr ds:[eax+ebx-1]
004E38A6   add dword ptr ss:[ebp-14],eax  求和 "SHou0597!@"＝2B5＝ss:[ebp-14]
004E38A9   inc ebx
004E38AA   dec esi
004E38AB   jnz short gwyinter.004E389E
004E38AD   mov eax,dword ptr ss:[ebp-C]    截取的机器码 047047390
004E38B0   call gwyinter.00403F4C    取长度
004E38B5   mov esi,eax
004E38B7   test esi,esi
004E38B9   jle gwyinter.004E39A3
004E38BF   mov ebx,1
004E38C4   mov eax,dword ptr ss:[ebp-8]     
004E38C7   call gwyinter.00403F4C     
004E38CC   inc eax
004E38CD   cmp ebx,eax
004E38CF   jge short gwyinter.004E391E
004E38D1   lea eax,dword ptr ds:[ebx+5]    eax=1＋5＝6
004E38D4   mov edx,dword ptr ss:[ebp-8]
004E38D7   movzx edx,byte ptr ds:[edx+ebx-1]
004E38DC   imul edx
004E38DE   add eax,ebx
004E38E0   mov edx,dword ptr ss:[ebp-C]
004E38E3   movzx edx,byte ptr ds:[edx+ebx-1]
004E38E8   imul edx
004E38EA   mov ecx,ebx
004E38EC   add ecx,ecx
004E38EE   mov edx,ecx
004E38F0   imul edx,dword ptr ss:[ebp-18]
004E38F4   add eax,edx
004E38F6   imul ecx,ebx
004E38F9   add ecx,0D
004E38FC   imul ecx,dword ptr ss:[ebp-10]
004E3900   add eax,ecx
004E3902   lea edx,dword ptr ds:[ebx+ebx*2]
004E3905   add edx,0C
004E3908   imul edx,dword ptr ss:[ebp-14]
004E390C   add eax,edx
004E390E   mov edx,0E6
004E3913   sub edx,ebx
004E3915   mov ecx,edx
004E3917   cdq
004E3918   idiv ecx
004E391A   mov edi,edx
004E391C   jmp short gwyinter.004E395E
...
004E395E   cmp dword ptr ss:[ebp-20],3
004E3962   jle short gwyinter.004E3972
004E3964   mov eax,edi
004E3966   mov ecx,9
004E396B   cdq
004E396C   idiv ecx
004E396E   mov edi,edx
004E3970   jmp short gwyinter.004E397E
004E3972   mov eax,edi
004E3974   mov ecx,63
004E3979   cdq
```



# 汇编代码分析

## 关键循环逻辑

```asm
004E397A  idiv ecx
004E397C  mov edi,edx
004E397E  cmp edi,9
004E3981  jle short gwyinter.004E3986
004E3983  inc dword ptr ss:[ebp-20]
004E3986  lea edx,dword ptr ss:[ebp-24]
004E3989  mov eax,edi
004E398B  call gwyinter.00409578    ; 每次循环计算得到注册码的 2 位数
004E3990  mov edx,dword ptr ss:[ebp-24]
004E3993  lea eax,dword ptr ss:[ebp-1C]
004E3996  call gwyinter.00403F54    ; 将 004E398B 计算值一个个连接起来,注意堆栈的值
004E399B  inc ebx
004E399C  dec esi                 ; 循环指针
004E399D  jnz gwyinter.004E38C4
004E39A3  mov eax,dword ptr ss:[ebp+8]
004E39A6  mov edx,dword ptr ss:[ebp-1C]
004E39A9  call gwyinter.00403D20
004E39AE  xor ecx,ecx
004E39B0  mov edx,5
004E39B5  mov eax,dword ptr ss:[ebp-4]
004E39B8  call gwyinter.004E05CC
004E39BD  xor eax,eax
004E39BF  pop edx
004E39C0  pop ecx
004E39C1  pop ecx
004E39C2  mov dword ptr fs:[eax],edx
004E39C5  push gwyinter.004E39EF
004E39CA  lea eax,dword ptr ss:[ebp-24]
004E39CD  call gwyinter.00403CCC
004E39D2  lea eax,dword ptr ss:[ebp-1C]
004E39D5  call gwyinter.00403CCC
004E39DA  lea eax,dword ptr ss:[ebp-C]
004E39DD  mov edx,2
004E39E2  call gwyinter.00403CF0
004E39E7  retn
004E39E8  jmp gwyinter.004036E0
004E39ED  jmp short gwyinter.004E39CA
004E39EF  pop edi                 ; 0012FC6C
004E39F0  pop esi
004E39F1  pop ebx
004E39F2  mov esp,ebp
004E39F4  pop ebp
004E39F5  retn 4
```

## 小结

该循环对两个值进行计算：一个是机器码：`"CD48047047390"`；一个是字符串：`"SHou0597!@"`。

堆栈得到一个值：
`0012FA54` `00E62E84` `ASCII "2044854858838"`

## 6.4 退出 6.3 的 call 后 F8 继续

```asm
004CBC1C  mov eax,dword ptr ss:[ebp-10]
```
堆栈 `ss:[0012FC70]=014A2150`, `(ASCII "636877907888216")`

```asm
004CBC1F  mov edx,dword ptr ss:[ebp-14]
```
堆栈 `ss:[0012FC6C]=00E62E84`, `(ASCII "2044854858838")`

```asm
004CBC22  call gwyinter.0040405C    ; 估计是注册码比较的地方
004CBC27  jnz short gwyinter.004CBC75    ; 不等就完了吧？
```

这个注册码的判断太像自校验的判断。

## 7. 试验

将计算得到的注册码（？）：`2044854858838` 替换掉注册表里的`"636877907888216"`，运行程序，之前不能点的都可以点了。

另外脱壳爆破后的提示画面好像有麻痹作用。

## 8. 算法总结

- 程序先将输入的注册码转换成数字存在注册表里。
- 运行的时候没有对注册表进行读取注册码比较，一直到点击需注册才能看到的项目时，它才进行关键比较。
- 注册码的计算涉及到两个值，一个机器码，一个内置的字符串 `"SHou0597!@"`。
- 计算过程复杂，不过最后还是明码比较，将计算得到的注册码与注册表里的比较，相等就可以了。

```
4dd466d0e1e0cc123afbe504da2462ca
```
