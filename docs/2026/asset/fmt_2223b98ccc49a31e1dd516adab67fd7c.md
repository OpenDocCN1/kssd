
# kongfoo 兄弟的 CoolCryptor V0.9 脱壳

kongfoo 兄弟的 CoolCryptor V0.9 脱壳

直接下载：[点击下载（或鼠标右键另存为）](https://www.kanxue.com/pediy6462/COOLcryptor.rar)

软件大小： 38 KB

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`flyODBG 1.10B`修改版、`PEiD`、`LordPE`、`ImportREC`

―――――――――――――――――――――――――――――――――

## 【脱壳过程】：

“几多天真的思想，几多找到是颓丧，沉默去迎失望，几多心中创伤”

――呵呵，经过 kongfoo 兄弟几多辛苦的努力，`COOLcryptor`升级至`V0.9`了。花指令繁多；其反跟踪方面已经比一般的保护壳更强了；其双进程方式则是除`Armadillo`之外所仅见的，虽然`COOLcryptor`的双进程还不具备更多的功能，但是kongfoo擅于溶和、吸收其它壳优点的能力已经让人佩服了！

再提几点建议吧：

1、 `COOLcryptor`自身的兼容性。如今`COOLcryptor`只是For XP，以发展的眼光来看，一款优秀的壳最好能够顺利的跨平台运行，至少能够在目前流行的各WinDows平台上都能够运行。现在的`COOLcryptor V0.9`在`Win XP`的中文目录下也无法运行起来。这是`COOLcryptor`所必须解决的问题，否则随着`COOLcryptor`的不断升级，想纠正就会花费更多的精力了。

2、 `COOLcryptor`加壳后被保护程序的兼容性。`COOLcryptor`加壳后的一些程序在`XP`上也无法运行了，如果这样下去，`COOLcryptor`只是成为一个自身的`UnPackMe`，而无法成为一款优秀的壳。

3、 `IAT`处理，代码变形等方面还要继续变态，呵呵，对于壳来说，越变态越好啦。 8)

原来这个笔记应由kitty兄弟来写的，既然兄弟客气，我就来简单写一下吧，希望大家多多写作脱壳笔记，呵呵

OK，说了一堆废话，下面就以今天刚发布的`COOLcryptor0.9.exe`来脱壳演示一下。

―――――――――――――――――――――――――――――――――

设置`Ollydbg`忽略所有其它选项。用`IsDebug 1.4`插件去掉调试器标志。

### 一、反 反跟踪

1、去掉 `lock cmpxchg`

```
00408060     E8 03000000         call COOLcryp.00408068//进入OD后停在这
00408065     E9 EB07FF04         jmp 053F8855
```
`Ctrl+B`搜索2进制字符：`F00FC7C8` 在`00408181`处，下断

```
00408181     F0:0FC7C8           lock cmpxchg8b eax; 非法使用寄存器
00408185     83AD D5214000 04    sub dword ptr ss:[ebp+4021D5],4//[ebp+4021D5]=324C
0040818C     79 F3               jns short COOLcryp.00408181
0040818E     64:8F05 00000000    pop dword ptr fs:[0]
00408195     83C4 04             add esp,4//此处下断，F9 断在这！
00408198     EB 51               jmp short COOLcryp.004081EB
```
`00408181`处利用`EXECryptor`的`lock cmpxchg8b`异常解码。用修改的`Ollydbg 1.10B`版调试，增加忽略`C000001E`异常，在`00408195`处下断，`F9`运行，异常解码完毕后就停在`00408195`了。

当然，可以不管`lock cmpxchg8b`这个异常解码，直接开始第2步。

―――――――――――――――――――――――――――――――――

2、进程比较  `BP GetModuleFileNameA+3`

```
00409524     FF95 3D5B4000       call dword ptr ss:[ebp+405B3D]; kernel32.GetModuleFileNameA
0040952A     8BC7                mov eax,edi//中断后 Alt+F9 返回这里
0040952C     8038 60             cmp byte ptr ds:[eax],60
0040952F     72 03               jb short COOLcryp.00409534
00409531     8028 20             sub byte ptr ds:[eax],20
00409534     40                  inc eax
00409535     8038 00             cmp byte ptr ds:[eax],0
00409538     75 F2               jnz short COOLcryp.0040952C
0040953A     8D85 9B574000       lea eax,dword ptr ss:[ebp+40579B]
00409540     8D9D B1594000       lea ebx,dword ptr ss:[ebp+4059B1]
00409546     33C9                xor ecx,ecx
00409548     8A1401              mov dl,byte ptr ds:[ecx+eax]
                               //修改：mov dl,byte ptr ds:[ecx+ebx] ★
0040954B     381419              cmp byte ptr ds:[ecx+ebx],dl
0040954E     75 0E               jnz short COOLcryp.0040955E//跳就OVER了
00409550     41                  inc ecx
00409551     81F9 03010000       cmp ecx,103
00409557     75 EF               jnz short COOLcryp.00409548
00409559     E9 73070000         jmp COOLcryp.00409CD1
```

――――――――――――――――――――――――

3、大小比较  `BP GetFileSize`

```
00409F81     FF95 415B4000       call dword ptr ss:[ebp+405B41]; kernel32.CreateFileA
00409F87     8BD8                mov ebx,eax
00409F89     6A 00               push 0
00409F8B     50                  push eax
00409F8C     FF95 515B4000       call dword ptr ss:[ebp+405B51]; kernel32.GetFileSize
00409F92     8985 2F564000       mov dword ptr ss:[ebp+40562F],eax//中断后 Alt+F9 返回这里
00409F98     53                  push ebx
00409F99     FF95 555B4000       call dword ptr ss:[ebp+405B55]; kernel32.CloseHandle
00409F9F     8D85 B1594000       lea eax,dword ptr ss:[ebp+4059B1]
00409FA5     6A 00               push 0
00409FA7     68 80000000         push 80
00409FAC     6A 03               push 3
00409FAE     6A 00               push 0
00409FB0     6A 01               push 1
00409FB2     68 00000080         push 80000000
00409FB7     50                  push eax
00409FB8     FF95 415B4000       call dword ptr ss:[ebp+405B41]; kernel32.CreateFileA
00409FBE     8BD8                mov ebx,eax
00409FC0     6A 00               push 0
00409FC2     50                  push eax
00409FC3     FF95 515B4000       call dword ptr ss:[ebp+405B51]; kernel32.GetFileSize
00409FC9     8BF8                mov edi,eax
00409FCB     53                  push ebx
00409FCC     FF95 555B4000       call dword ptr ss:[ebp+405B55]; kernel32.CloseHandle
00409FD2     33BD 2F564000       xor edi,dword ptr ss:[ebp+40562F]//修改：xor edi,edi ★
00409FD8     75 03               jnz short COOLcryp.00409FDD//跳就ExitProcess了
00409FDA     61                  popad
00409FDB     EB 20               jmp short COOLcryp.00409FFD
```

――――――――――――――――――――――――

4、还有一处 进程比较

`Ctr+F` 在当前位置搜索命令：`cmp byte ptr ds:[ecx+ebx],dl` 在`0040A102`处

```
0040A0F1     8D85 9B574000       lea eax,dword ptr ss:[ebp+40579B]
0040A0F7     8D9D B1594000       lea ebx,dword ptr ss:[ebp+4059B1]
0040A0FD     33C9                xor ecx,ecx
0040A0FF     8A1401              mov dl,byte ptr ds:[ecx+eax]
                               //修改：mov dl,byte ptr ds:[ecx+ebx] ★
0040A102     381419              cmp byte ptr ds:[ecx+ebx],dl
0040A105     75 0B               jnz short COOLcryp.0040A112//跳就OVER了
0040A107     41                  inc ecx
0040A108     81F9 04010000       cmp ecx,104
0040A10E     75 EF               jnz short COOLcryp.0040A0FF
0040A110     EB 20               jmp short COOLcryp.0040A132
```
修改之后`COOLcryptor`的双进程就`OVER`啦  :-)

―――――――――――――――――――――――

5、MeltICE表 `BP CreateFileA`

```
0040B59E     FF95 415B4000       call dword ptr ss:[ebp+405B41]; kernel32.CreateFileA
0040B5A4     40                  inc eax//中断后 Alt+F9 返回这里
0040B5A5     75 0D               jnz short COOLcryp.0040B5B4
0040B5A7     803E 00             cmp byte ptr ds:[esi],0
0040B5AA     74 DE               je short COOLcryp.0040B58A
0040B5AC     46                  inc esi
0040B5AD     803E FF             cmp byte ptr ds:[esi],0FF
0040B5B0     75 F5               jnz short COOLcryp.0040B5A7
0040B5B2     EB 02               jmp short COOLcryp.0040B5B6
0040B5B4     61                  popad
0040B5B5     C3                  retn
```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

MeltICE表

```
0040B466  5C 5C 2E 5C 53 49 43 45 00 5C 5C 2E 5C 4E 54 49   \.SICE.\.NTI
0040B476  43 45 00 5C 5C 2E 5C 4E 54 49 43 45 37 38 37 31   CE.\.NTICE7871
0040B486  00 5C 5C 2E 5C 4E 54 49 43 45 44 30 35 32 00 5C   .\.NTICED052.\
0040B496  5C 2E 5C 54 52 57 44 45 42 55 47 00 5C 5C 2E 5C   .TRWDEBUG.\.
0040B4A6  54 52 57 00 5C 5C 2E 5C 54 52 57 32 30 30 30 00   TRW.\.TRW2000.\
0040B4B6  5C 5C 2E 5C 53 55 50 45 52 42 50 4D 00 5C 5C 2E   \.SUPERBPM.\.
0040B4C6  5C 49 43 45 44 55 4D 50 00 5C 5C 2E 5C 52 45 47   ICEDUMP.\.REG
0040B4D6  4D 4F 4E 00 5C 5C 2E 5C 46 49 4C 45 4D 4F 4E 00   MON.\.FILEMON.\
0040B4E6  5C 5C 2E 5C 52 45 47 56 58 44 00 5C 5C 2E 5C 46   \.REGVXD.\.F
0040B4F6  49 4C 45 56 58 44 00 5C 5C 2E 5C 56 4B 45 59 50   ILEVXD.\.VKEYP
0040B506  52 4F 44 00 5C 5C 2E 5C 42 57 32 4B 00 5C 5C 2E   ROD.\.BW2K.\.
0040B516  5C 53 49 57 44 45 42 55 47 00 5C 5C 2E 5C 53 49   SIWDEBUG.\.SI
0040B526  57 56 49 44 00 5C 5C 2E 5C 47 6C 6F 62 61 6C 5C   WVID.\.Global
0040B536  46 49 4C 45 4D 4F 4E 00 5C 5C 2E 5C 47 6C 6F 62   FILEMON.\.Glob
0040B546  61 6C 5C 52 45 47 4D 4F 4E 00 5C 5C 2E 5C 46 49   alREGMON.\.FI
0040B556  4C 45 56 58 44 2E 56 58 44 00 5C 5C 2E 5C 52 45   LEVXD.VXD.\.RE
0040B566  47 56 58 44 2E 56 58 44 00 5C 5C 2E 5C 46 52 4F   GVXD.VXD.\.FRO
0040B576  47 53 49 43 45 00 5C 5C 2E 5C 49 63 65 45 78 74   GSICE.\.IceExt
```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

我用`Ollydbg`修改版调试，这里就没有威胁啦。 :-)

――――――――――――――――――――――――

6、时间监测  `BP GetTickCount+5`

`COOLcryptor V0.9`加了时间监测，要下`BP GetTickCount+5`，呵呵。`Shift+F9`走

```
0040A35F     FF95 F15A4000       call dword ptr ss:[ebp+405AF1]; kernel32.77E5751F
0040A365     8985 1F564000       mov dword ptr ss:[ebp+40561F],eax//第1次返回
0040A5B7     FF95 F15A4000       call dword ptr ss:[ebp+405AF1]; kernel32.77E5751F
0040A5BD     33D2                xor edx,edx//第2次返回
```
