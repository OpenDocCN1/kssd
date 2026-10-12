
# LFS S2 0.5P ALPHA 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-325.htm`](https://www.pediy.com/kssd/pediy07/pediy7-325.htm)

## 作者
noword

## 简介
LIVE FOR SPEED (下面简称 LFS) 严格的说是一款模拟赛车类游戏：没有街机模式，没有转向辅助--你所做的是接近于真实的驾驶操控。官方网址：http://www.liveforspeed.net

`S1` 指的是 LFS 的第一版，注册费 12 英镑，`S2` 是第二版，需要 24 英镑。`S2 0.5P ALPHA` 于 2005 年 6 月 25 日发布，虽说是个测试版，但是已相当完善，完全可以作为正式版推出。但是，制作方本着严谨的作风，希望有一段时间进行车辆和赛道的微调，尽可能的做出完美的正式版。

## 保护手法
游戏在用户输入用户名和密码后，会连到官方的服务器上进行验证，验证后根据用户的注册程度，把游戏激活成 `S1` 或 `S2`。在没有激活前，为 `DEMO` 状态，只能选 3 辆车，一组赛道；激活成为 `S1` 的用户能多选 5 辆车，外加两组赛道；成为 `S2` 的用户能选全部的车和赛道。

在 `S2` 出来以前，有一个针对 `S1` 的注册器，可能是伪装了服务器的网络通讯，该注册器也能在 `P` 版以前的 LFS S2（流传出来的 L 版和 F 版）上使用，激活成为 `S1` 用户。`P` 版出来后，虽然也能用这个激活成 `S1` 用户，而且游戏中 `S1` 的车辆和赛道能够被选择，但是已经不能打开(屏幕上空空如也)。虽说如此，这个注册器对于寻找 `P` 版的突破口还是有帮助的。

用 File Monitor 发现，激活时会在游戏的 `misc\data` 目录下生成 3 个文件 `f1.xxx` `f2.xxx` `f3.xxx`。同时还发现，如果用户选择了 `S1` 的车辆或赛道后，游戏会读取 `f1.xxx` 和 `f2.xxx`，而如果选择的是 `demo` 中的 3 辆车，则没有读取的动作。可见该游戏采用的是 key file 保护。前期的侦查工作到此完毕。

## 分析
可能作者对自己的算法很有信心，文件没加壳，省了不少事。

先用 `w32dsm` 和 `IDA` 反汇编了一遍，方便在跟踪时随时看看上下文，特别是 `IDA` 可以看到许多标准 C 函数。

用 `ODBG` 载入，"f1.xxx"的字符串在 `ds:[561d58]`，选择一辆 `S1` 的车后，很容易断到这里：

```
0042059A  |.  8B15 581D5600           mov edx,dword ptr ds:[561D58]                      ;  f1.xxx
```

向上翻几页，找到整个子程序的源头：

```
004204F0  /$  8B4424 08             mov eax,dword ptr ss:[esp+8]
004204F4  |.  81EC D8000000         sub esp,0D8
004204FA  |.  53                   push ebx
004204FB  |.  55                   push ebp
004204FC  |.  56                   push esi
004204FD  |.  57                   push edi
004204FE  |.  8BE9                 mov ebp,ecx
00420500  |.  50                   push eax
00420501  |.  E8 9A660400          call LFS.00466BA0                            ; 得到车的序号 (demo 0-2, s1 3-7，s2 >8 )
00420506  |.  8BD8                 mov ebx,eax
00420508  |.  83C4 04             add esp,4
0042050B  |.  83FB FF             cmp ebx,-1
0042050E  |.  75 0F               jnz short LFS.0042051F
00420510  |.  5F                 pop edi
00420511  |.  5E                 pop esi
00420512  |.  5D                 pop ebp
00420513  |.  33C0               xor eax,eax
00420515  |.  5B                 pop ebx
00420516  |.  81C4 D8000000       add esp,0D8
0042051C  |.  C2 3400            retn 34
0042051F  |> \8D8C24 84000000     lea ecx,dword ptr ss:[esp+84]
00420526  |.  8D5424 2C          lea edx,dword ptr ss:[esp+2C]
0042052A  |.  51                 push ecx
0042052B  |.  52                 push edx
0042052C  |.  53                 push ebx                             ; 车号
0042052D  |.  E8 9E680400        call LFS.00466DD0                       ; 根据车号，得到需要解密的数据地址
00420532  |.  83C4 0C           add esp,0C
00420535  |.  83FB 03           cmp ebx,3
00420538  |.  0F8C 8D030000     jl LFS.004208CB
0042053E  |.  833D EC738400 01  cmp dword ptr ds:[8473EC],1
00420545  |.  7D 0F             jge short LFS.00420556
00420547  |.  5F                 pop edi
00420548  |.  5E                 pop esi
00420549  |.  5D                 pop ebp
0042054A  |.  33C0               xor eax,eax
0042054C  |.  5B                 pop ebx
0042054D  |.  81C4 D8000000       add esp,0D8
00420553  |.  C2 3400            retn 34
00420556  |>  8D8424 A8000000    lea eax,dword ptr ss:[esp+A8]
0042055D  |.  6A 40             push 40
0042055F  |.  50                 push eax
00420560  |.  E8 5B630400        call LFS.004668C0                        ; 从注册表"Software\Microsoft\Windows\CurrentVersion\ProductID"中得到 Windows 的产品号，就是显示在“控制面板”->“系统”
00420565  |.  83C4 08           add esp,8
00420568  |.  85C0               test eax,eax
0042056A  |.  75 12             jnz short LFS.0042057E
0042056C  |.  5F                 pop edi
0042056D  |.  5E                 pop esi
0042056E  |.  5D                 pop ebp
0042056F  |.  A3 EC738400        mov dword ptr ds:[8473EC],eax
00420574  |.  5B                 pop ebx
00420575  |.  81C4 D8000000       add esp,0D8
0042057B  |.  C2 3400            retn 34
0042057E  |>  B9 06000000        mov ecx,6
00420583  |.  8DB424 A8000000    lea esi,dword ptr ss:[esp+A8]              ;  windows 的序列号
0042058A  |.  8D7C24 54         lea edi,dword ptr ss:[esp+54]
0042058E  |.  F3:A5            rep movs dword ptr es:[edi],dword ptr ds:>
00420590  |.  8D4C24 44         lea ecx,dword ptr ss:[esp+44]
00420594  |.  51                 push ecx
00420595  |.  E8 A6620400        call LFS.00466840                          ; 得到 CPU 信息
0042059A  |.  8B15 581D5600       mov edx,dword ptr ds:[561D58]                  ;  f1.xxx
004205A0  |.  68 54875300        push LFS.00538754                       ;  ASCII "rb"
004205A5  |.  52                 push edx
004205A6  |.  E8 4D940C00        call LFS.004E99F8                       ;  fopen
004205AB  |.  8BF0               mov esi,eax
004205AD  |.  33FF               xor edi,edi
004205AF  |.  83C4 0C           add esp,0C
004205B2  |.  3BF7               cmp esi,edi
004205B4  |.  75 15             jnz short LFS.004205CB
004205B6  |.  893D EC738400      mov dword ptr ds:[8473EC],edi
004205BC  |.  5F                 pop edi
004205BD  |.  5E                 pop esi
004205BE  |.  5D                 pop ebp
004205BF  |.  33C0               xor eax,eax
004205C1  |.  5B                 pop ebx
004205C2  |.  81C4 D8000000       add esp,0D8
004205C8  |.  C2 3400            retn 34
004205CB  |>  56                 push esi
004205CC  |.  6A 18             push 18
004205CE  |.  8D4424 74         lea eax,dword ptr ss:[esp+74]
004205D2  |.  6A 01             push 1
004205D4  |.  50                 push eax                        ;   
004205D5  |.  E8 85940C00        call LFS.004E9A5F                       ;  ReadFile
004205DA  |.  56                 push esi
004205DB  |.  E8 39930C00        call LFS.004E9919                       ;  fclose
```

现在已经得到了 3 样东西：`CPUID`, `Windows ProductID`，文件 `f1.xxx` 内容，依次保存在堆栈 `ss:[esp+58]`到 `ss:[esp+94]`的 16 个字中，下面会变什么戏法呢？

```
004205E0  |.  8B4C24 64           mov ecx,dword ptr ss:[esp+64]
004205E4  |.  8B7424 70           mov esi,dword ptr ss:[esp+70]
004205E8  |.  8B9424 90000000     mov edx,dword ptr ss:[esp+90]
004205EF  |.  8B8424 80000000     mov eax,dword ptr ss:[esp+80]
004205F6  |.  33CE                 xor ecx,esi
004205F8  |.  8B7424 74           mov esi,dword ptr ss:[esp+74]
004205FC  |.  33CA                 xor ecx,edx
004205FE  |.  8B5424 60           mov edx,dword ptr ss:[esp+60]
00420602  |.  33C8                 xor ecx,eax
00420604  |.  8B8424 84000000     mov eax,dword ptr ss:[esp+84]
0042060B  |.  894C24 48           mov dword ptr ss:[esp+48],ecx             ;  key[0] = buf[3] ^ buf[6] ^ buf[14] ^ buf[10]
0042060F  |.  8B8C24 94000000     mov ecx,dword ptr ss:[esp+94]
00420616  |.  33D6                 xor edx,esi
00420618  |.  8B7424 78           mov esi,dword ptr ss:[esp+78]
0042061C  |.  33D1                 xor edx,ecx
0042061E  |.  8B8C24 88000000     mov ecx,dword ptr ss:[esp+88]
00420625  |.  33D0                 xor edx,eax
00420627  |.  8B4424 5C           mov eax,dword ptr ss:[esp+5C]
0042062B  |.  895424 4C           mov dword ptr ss:[esp+4C],edx             ;  key[1] = buf[7] ^ buf[2] ^ buf[11] ^ buf[15]
0042062F  |.  8B5424 68           mov edx,dword ptr ss:[esp+68]
```



```
00420633   |.  33C6               xor eax,esi
00420635   |.  8B7424 6C          mov esi,dword ptr ss:[esp+6C]
00420639   |.  33C2               xor eax,edx
0042063B   |.  8B9424 8C000000    mov edx,dword ptr ss:[esp+8C]
00420642   |.  33C1               xor eax,ecx
00420644   |.  8B4C24 58          mov ecx,dword ptr ss:[esp+58]
00420648   |.  894424 50          mov dword ptr ss:[esp+50],eax            ;  key[2] = buf[8] ^ buf[12] ^ buf[1] ^ buf[4]
0042064C   |.  8B4424 7C          mov eax,dword ptr ss:[esp+7C]
00420650   |.  33C8               xor ecx,eax
00420652   |.  68 54875300        push LFS.00538754                        ;  ASCII "rb"
00420657   |.  33CE               xor ecx,esi
00420659   |.  33CA               xor ecx,edx
0042065B   |.  8B15 5C1D5600      mov edx,dword ptr ds:[561D5C]           ;  f2.xxx
00420661   |.  52                 push edx
00420662   |.  894C24 5C          mov dword ptr ss:[esp+5C],ecx           ;  key[3] = buf[5] ^ buf[13] ^ buf[0] ^ buf[9]
00420666   |.  E8 8D930C00        call LFS.004E99F8                       ;  fopen
```

这 16 个字经过一系列交错的异或运算得到了 4 个字（128 位）

```
0042066B   |.  8BF0               mov esi,eax
0042066D   |.  83C4 1C            add esp,1C
00420670   |.  3BF7               cmp esi,edi
00420672   |.  75 15              jnz short LFS.00420689
00420674   |.  893D EC738400      mov dword ptr ds:[8473EC],edi
0042067A   |.  5F                 pop edi
0042067B   |.  5E                 pop esi
0042067C   |.  5D                 pop ebp
0042067D   |.  33C0               xor eax,eax
0042067F   |.  5B                 pop ebx
00420680   |.  81C4 D8000000      add esp,0D8
00420686   |.  C2 3400            retn 34
00420689   |>  83FB 07            cmp ebx,7                                ;  ebx -> 车号
0042068C   |.  7D 2B              jge short LFS.004206B9                   ;  s1 的车
0042068E   |.  81C3 FDFFFF0F      add ebx,0FFFFFFD
00420694   |.  8D4424 10          lea eax,dword ptr ss:[esp+10]
00420698   |.  C1E3 04            shl ebx,4
0042069B   |.  50                 push eax                                   ;  fpos_t
0042069C   |.  56                 push esi
0042069D   |.  895C24 18          mov dword ptr ss:[esp+18],ebx
004206A1   |.  897C24 1C          mov dword ptr ss:[esp+1C],edi
004206A5   |.  E8 9D930C00        call LFS.004E9A47                          ;  fsetpos
004206AA   |.  56                 push esi
004206AB   |.  6A 04              push 4
004206AD   |.  8D4C24 28          lea ecx,dword ptr ss:[esp+28]
004206B1   |.  6A 04              push 4
004206B3   |.  51                 push ecx
004206B4   |.  E9 9E010000        jmp LFS.00420857
004206B9   |>  83FB 08            cmp ebx,8
004206BC   |.  7D 50              jge short LFS.0042070E                   ;  s2 的车
……
```

代码太长，简单介绍一下功能——根据车号，读出 `f2.xxx` 文件中的某 4 个字（128 位数）

……

```
00420860   |.  E8 B4900C00        call LFS.004E9919                          ;  fclose
00420865   |.  8D4424 38          lea eax,dword ptr ss:[esp+38]
00420869   |.  8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
0042086D   |.  50                 push eax                                   ;  key 的地址
0042086E   |.  8D5424 20          lea edx,dword ptr ss:[esp+20]
00420872   |.  51                 push ecx                                   ;  f2.xxx 的部分内容（前两个字）
00420873   |.  52                 push edx
00420874   |.  E8 67FC0700        call LFS.004A04E0                          ;  解码
00420879   |.  8D4424 44          lea eax,dword ptr ss:[esp+44]
0042087D   |.  8D4C24 30          lea ecx,dword ptr ss:[esp+30]
00420881   |.  50                 push eax                                   ;  key 的地址
00420882   |.  8D5424 34          lea edx,dword ptr ss:[esp+34]
00420886   |.  51                 push ecx                                   ;  f2.xxx 的部分内容（后两个字）
00420887   |.  52                 push edx
00420888   |.  E8 53FC0700        call LFS.004A04E0                          ;  解码
……
```

后面的代码是来到另一个深不可测的 call，把解出来的 128 位数作为 key，去解 `0042052D` 中得到的一大陀加密数据

……

`:004A04E0` 是关键，杀进去看看：

```
004A04E0  /$  83EC 08             sub esp,8                                  ;  decode
004A04E3  |.  53                 push ebx
004A04E4  |.  55                 push ebp
004A04E5  |.  56                 push esi
004A04E6  |.  8B7424 20          mov esi,dword ptr ss:[esp+20]              ;  key
004A04EA  |.  8B4424 18          mov eax,dword ptr ss:[esp+18]              ;  密文
004A04EE  |.  57                 push edi
004A04EF  |.  8B3E               mov edi,dword ptr ds:[esi]                ;  key
004A04F1  |.  BA 2037EFC6        mov edx,C6EF3720                           ;  !!!!!!!!
……
004A0555  |.  81C2 4786C861      add edx,61C88647                           ;  !!!!!!!!
……
```

我根本没仔细看这个算法，找到两个立即数，就上 google 搜索一下（小技巧：搜索时，在 16 进制数前面加上 `0x`。例如光搜索“`C6EF3720`”只有 3 个结果，而“`0xC6EF3720`”就出来 231 个结果，还是 C 源码的）。

这儿用了 `TEA`(Tiny Encryption Algorithm)，`TEA` 是一种对称算法，加密时，进去 2 个字的明文，加上一组 128 位的密匙，出来 2 个字的密文；解密时，进去 2 个字的密文，加上同一组 128 位的密匙，出来 2 个字的明文。

### 结论

`S2 0.5P` 的验证手段是这样的：当选择的是 `S1` 或 `S2` 的车（赛道）时，根据本地的 `CPUID`、`Windows ProductID` 和 `f1.xxx` 的内容，算出一个 128 位的数，把这个数作为密匙，运用 `TEA` 算法，去解 `f2.xxx` 中的某个 128 位的数（解两次），然后把解出来的 128 位数作为密匙，去解真正要解的加密数据(车辆和赛道的关键信息)。

所以，破解的关键是得到解密后的 `f2.xxx`。

顺便说一下 `f3.xxx` 中存放的是用户名和密码信息，用于链接官方服务器，进行 online 游戏时验证之用，也用了 `TEA` 算法。

### 破解方法

在 1997 年，J. Kelsey, B. Schneier, D. Wagner 写了篇论文 Related-Key Cryptanalysis of 3-WAY, Biham-DES, CAST, DES-X, NewDES, RC2, and TEA，说是可以用 2 的 32 次方攻破 `TEA`。我 P 颠 P 颠的找来一看，原来是选择明文攻击，没用 @_@

但是，这种验证方法是有弱点的——正式注册用户能够算出正确的密匙的，然后根据这个密匙，就能够写出注册器。所以，`www.chinaspeed.org` 的 3 位车迷——`SHEEPY`, `JAROD`, `WARMGUN`，合资买了一份 `S2`，然后把用于激活的电脑上的 `CPUID`，`Windows ProductID` 和 `f1.xxx`、`f2.xxx` 文件传了给我，然后 `keygen` 就出来了。 ^_^

当然，用注册器注册后的游戏，只能进行单机游戏，无法通过 `LFS` 官方服务器的验证。不过，只要不上官网就没问题，`www.chinaspeed.org` 已经开了 3 组 host。

关于 `TEA` 算法，在 `http://www-users.cs.york.ac.uk/~matthew/TEA/` 有非常详细的介绍，包括各个演进版本和算法弱点。有兴趣的可以看一下。

`4dd466d0e1e0cc123afbe504da2462ca`
