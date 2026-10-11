
## SMC 暴破研究

### 1. 初始汇编代码分析与 SMC 跳转修补

```
00645000    60                pushad  
00645001    E8 00000000       call    00645006  
00645006    5D                pop     ebp  
00645007    81ED EAA84300     sub     ebp, 0043A8EA  
0064500D    B8 E4A84300       mov     eax, 0043A8E4  
00645012    03C5              add     eax, ebp  
00645014    2B85 78AD4300     sub     eax, dword ptr [ebp+43AD78]  
0064501A    8985 84AD4300     mov     dword ptr [ebp+43AD84], eax  
00645020    80BD 6EAD4300 0   cmp     byte ptr [ebp+43AD6E], 0  
00645027    75 15             jnz     short 0064503E  
00645029    FE85 6EAD4300     inc     byte ptr [ebp+43AD6E]  
0064502F    E8 1D000000       call    00645051          ; 我们看这里 00645051 即是 JMP EAX 的下一行代码，当我们 SMC 的时候，我们还需要修补这里的代码。  
00645034    E8 73020000       call    006452AC  
00645039    E8 0A030000       call    00645348  
0064503E    8B85 70AD4300     mov     eax, dword ptr [ebp+43AD70]  
00645044    0385 84AD4300     add     eax, dword ptr [ebp+43AD84]  
0064504A    894424 1C         mov     dword ptr [esp+1C], eax  
0064504E    61                popad  
0064504F    FFE0              jmp     eax               ; 这里的 JMP EAX 让他跳到 SMC 地址 // 修改为 JMP 00646747  
00645051    80BD A2AE4300 0   cmp     byte ptr [ebp+43AEA2], 0 ; 当我们修改 JMP EAX 后以下两行数据将被覆盖  
00645058    74 1D             je      short 00645077  
0064505A    8DB5 A3AE4300     lea     esi, dword ptr [ebp+43AEA3]  
```

### 2. SMC 目标地址数据写入

在 `00646747` 地址写入以下数据以处理软件启动检测、注册、关于部分及功能设置：

```
00646747    66:C705 17844F0   mov     word ptr [4F8417], 45C6      ; 以下三行 // 软件启动时的检测  
00646750    C705 1A844F00 0   mov     dword ptr [4F841A], 8B909001  
0064675A    C605 2A844F00 E   mov     byte ptr [4F842A], 0EB  
00646761    66:C705 DFCD590   mov     word ptr [59CDDF], 45C6      ; 以下三行 // 软件注册部分  
0064676A    C705 E2CD5900 0   mov     dword ptr [59CDE2], 8B909001  
00646774    C605 F2CD5900 E   mov     byte ptr [59CDF2], 0EB  
0064677B    C605 CDFF5800 0   mov     byte ptr [58FFCD], 1         ; 以下两行 // 软件关于部分  
00646782    66:C705 C8FF580   mov     word ptr [58FFC8], 9090  
0064678B    66:C705 76CE590   mov     word ptr [59CE76], 45C6      ; 以下三行 // 功能设置  
00646794    C705 79CE5900 0   mov     dword ptr [59CE79], 8B909001  
0064679E    C605 89CE5900 E   mov     byte ptr [59CE89], 0EB  
006467A5    C605 94EB5800 E   mov     byte ptr [58EB94], 0EB       ; 以下修改解决掉功能限制 // 文件彻底删除助手  
006467AC    C605 5CD75800 E   mov     byte ptr [58D75C], 0EB       ; 文件分割助手  
006467B3    E9 1085F5FF       jmp     0059ECC8          ; 返回到 OEP // 如有功能限制继续添加  
```

对应的二进制代码如下：

```
66 C7 05 17 84 4F 00 C6 45 C7 05 1A 84 4F 00 01 90 90 8B C6 05 2A 84 4F 00 EB 66 C7 05 DF CD 59 
00 C6 45 C7 05 E2 CD 59 00 01 90 90 8B C6 05 F2 CD 59 00 EB C6 05 CD FF 58 00 01 66 C7 05 C8 FF 
58 00 90 90 66 C7 05 76 CE 59 00 C6 45 C7 05 79 CE 59 00 01 90 90 8B C6 05 89 CE 59 00 EB C6 05 
94 EB 58 00 EB C6 05 5C D7 58 00 EB E9 10 85 F5 FF
```

### 3. 修正 CALL 跳转

打上 SMC 补丁数据后，发现程序无法运行，重载发现端倪，`0064502F` 处的 `CALL` 指向 `JMP` 的下一行代码，需要修补：

```
0064502F    E8 8B170000       call    00645051          ; 该 CALL 指向 JMP 的下一行代码，所以我们还需要修补这里的代码。修改为 Call 006467BF  
```

在 `006467BF` 处写入跳转指令以返回原逻辑：

```
006467BF    80BD A2AE4300 0   cmp     byte ptr [ebp+43AEA2], 0 ; 我们在这里写入数据,需要写两行。     
006467C6    0F84 ABE8FFFF     je      00645077  
006467CC    E9 89E8FFFF       jmp     0064505A          ; 返回  
```

再次保存，软件就 OK 了。

### 4. 功能使用条件判断与修复

我们再翻回头来看一下软件对功能使用时的条件判断。

当程序返回软件算法 `CALL 004F72B0` 时候，出现以下汇编语句：

```
call    004F72B0  
cmp     byte ptr [ebp-5], 0   // 都在比较 [ebp-5] 的数值是否为 0  
je                     // 如果是 0 则挂掉  
```

遗憾的是未在算法 `CALL` 中找到对 `[ebp-5]` 赋值的语句，因此进行了如下修改，在调用后将 `[ebp-5]` 设为 1：

```
0059CDDA    E8 D1A4F5FF       call    004F72B0          ; 注册  
0059CDDF    C645 FB 01        mov     byte ptr [ebp-5], 1  

0059CE71    E8 3AA4F5FF       call    004F72B0          ; 功能设置  
0059CE76    C645 FB 01        mov     byte ptr [ebp-5], 1  

0058D753    E8 589BF6FF       call    004F72B0  
0058D758    C645 FB 00        mov     byte ptr [ebp-5], 1         ; 文件分割助手  

0058EB8B    E8 2087F6FF       call    004F72B0  
0058EB90    C645 FB 00        mov     byte ptr [ebp-5], 1         ; 文件彻底删除助手  
```

设置特征码进行全局搜索：

```
call    004F72B0  
cmp     byte ptr [ebp-5], 0  
```

通过 `Ctrl+S` 搜索该特征码，找到一些数据，猜测这些是针对一些功能使用上的判断，修改方法和上文相同。

例如在 `0056B567` 处：

```
0056B567    E8 44BDF8FF       call    004F72B0  
0056B56C    807D FB 00        cmp     byte ptr [ebp-5], 0  
0056B570    74 0F             je      short 0056B581  
0056B572    8B45 FC           mov     eax, dword ptr [ebp-4]  
0056B575    BA ACB65600       mov     edx, 0056B6AC              ; ASCII "YesIKnow"  
0056B57A    E8 ED92E9FF       call    0040486C  
0056B57F    74 12             je      short 0056B593  
0056B581    8D83 50030000     lea     eax, dword ptr [ebx+350]  
0056B587    BA C0B65600       mov     edx, 0056B6C0              ; ASCII " Top 5 "  
0056B58C    E8 238FE9FF       call    004044B4  
0056B591    EB 0B             jmp     short 0056B59E  
```

在 `0056D394` 处：

```
0056D394    E8 179FF8FF       call    004F72B0  
0056D399    807D FB 00        cmp     byte ptr [ebp-5], 0  
0056D39D    74 0F             je      short 0056D3AE  
0056D39F    8B45 FC           mov     eax, dword ptr [ebp-4]  
0056D3A2    BA E0D45600       mov     edx, 0056D4E0              ; ASCII "YesIKnow"  
0056D3A7    E8 C074E9FF       call    0040486C  
0056D3AC    74 12             je      short 0056D3C0  
0056D3AE    8D83 34030000     lea     eax, dword ptr [ebx+334]  
0056D3B4    BA F4D45600       mov     edx, 0056D4F4              ; ASCII " Top 5 "  
0056D3B9    E8 F670E9FF       call    004044B4  
0056D3BE    EB 0B             jmp     short 0056D3CB  
```

在 `00570DA7` 处：

```
00570DA7    E8 0465F8FF       call    004F72B0  
00570DAC    807D FB 00        cmp     byte ptr [ebp-5], 0  
00570DB0    74 0F             je      short 00570DC1  
00570DB2    8B45 FC           mov     eax, dword ptr [ebp-4]  
00570DB5    BA 7C0E5700       mov     edx, 00570E7C              ; ASCII "YesIKnow"  
00570DBA    E8 AD3AE9FF       call    0040486C  
00570DBF    74 12             je      short 00570DD3  
00570DC1    8D83 20030000     lea     eax, dword ptr [ebx+320]  
00570DC7    BA 900E5700       mov     edx, 00570E90              ; ASCII " Top 1 "  
00570DCC    E8 E336E9FF       call    004044B4  
00570DD1    EB 0B             jmp     short 00570DDE  
```

在 `00570F9D` 处：

```
00570F9D    E8 0E63F8FF       call    004F72B0  
00570FA2    807D FB 00        cmp     byte ptr [ebp-5], 0  
00570FA6    74 0F             je      short 00570FB7  
00570FA8    8B45 FC           mov     eax, dword ptr [ebp-4]  
00570FAB    BA 34105700       mov     edx, 00571034              ; ASCII "YesIKnow"  
00570FB0    E8 B738E9FF       call    0040486C  
00570FB5    74 19             je      short 00570FD0  
```
