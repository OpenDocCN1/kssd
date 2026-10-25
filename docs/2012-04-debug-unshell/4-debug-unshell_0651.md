
# EXECryptor 部分代码分析

抛砖引玉,望巨人给予修复被抽取代码的指点

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-659_u.htm`](https://www.pediy.com/kssd/pediy08/pediy8-659_u.htm)

工具 `OD`,以下是分析代码时的部分笔记,写的有点乱,有误之处还请指点

壳用到的 `SEH` 异常有 `1,int3`
```
2,int1
3,push pushfd
   or dowrd ptr [esp],100
   popfd //设置单步异常
```
`F7`,`F8` 碰到第二,三种时,最好在 `SEH` 回调涵数中设断,再 `F9` 运行

设置 `OD` 中断在系统事件

参考文章:`simonzh2000` 大侠的<<`ExeCryptor` 2.2.X 的 `Anti` `Ollydbg` 小结>>

### 1, `OEP` 处的 `CC` 检测

在 `TLS` `callback` `008Ac981` //`TLS` 回调涵数 `F2` 调断

中断后的第一处 `SEH`

`seh` 回调涵数中修改 `eip` 的值
```
008A9499 C701 FCA68A00 mov dword ptr ds:[ecx],zhengtu.008AA6FC
```

得到预先保存的校验值
```
008AA6E7 8B00 mov eax,dword ptr ds:[eax]
```

进行比较,相等跳
```
008A5056 29D0 sub eax,edx
008A5058 0F84 D9340000 je zhengtu.008A8537
```

### 2, 反 `dump`

得到 `peb`
```
008A4EFB 64:8B05 >mov eax,dword ptr fs:[30]
```

得到 `peb.c`
```
008A5DF6 81C0 30B5AB18 add eax,18ABB530
008A5DFC 81C0 DC4A54E7 add eax,E7544ADC
008A5E02 8B00 mov eax,dword ptr ds:[eax]
```

`seh` 修改 `eip` `8AC2B0`

得到模块信息
```
008AC526 8B00 mov eax,dword ptr ds:[eax]
```

修改镜像大小
```
008A5AB1 C702 001>mov dword ptr ds:[edx],1000
```

### 3,

修改的 `eip` `008A86CC`

`seh` 中修改 `eip`
```
008A73D5 C701 81A>mov dword ptr ds:[ecx],zhengtu.008AA781
```

`seh` 中修改 `eip`
```
008A5B0A C701 C07>mov dword ptr ds:[ecx],zhengtu.008A70C0
```

`seh` 中修改 `eip`
```
008AB71F C701 227>mov dword ptr ds:[ecx],zhengtu.008A7E22
```

得到 `PEB`
```
008A4A50 64:8B05 >mov eax,dword ptr fs:[30]
008AC621 03C7 add eax,edi//eax=eax+2
```

得到 `PEB.2`
```
008A7AD3 8B00 mov eax,dword ptr ds:[eax]
```

比较 `PEB.2` 的值
```
008A4C3F 08C0 or al,al
008A4C41 0F84 E06>je zhengtu.008AB827
```

### 4,

得到 `PEB`
```
008AA7E6 64:8B05 >mov eax,dword ptr fs:[30]
```

`seh` 中修改 `eip`
```
008A8F7A C701 AF6>mov dword ptr ds:[ecx],zhengtu.008A69AF
```

得到 `PEB.68`
```
008A69B9 81C0 1D0>add eax,FC450C1D
008A69BF 81C0 4BF>add eax,3BAF44B//eax=PEB+68
008A5E51 8B00 mov eax,dword ptr ds:[eax]//eax=PEB.68=70 OD 调试时
```

`seh` 中修改 `eip`
```
008AA23E C701 9BB>mov dword ptr ds:[ecx],zhengtu.008ABF9B
```

`seh` 中修改 `eip`
```
008A8BEC C701 34908A00 mov dword ptr ds:[ecx],zhengtu.008A9034
```

`seh` 中修改 `eip`
```
008A83EF C701 A37F8A00 mov dword ptr ds:[ecx],zhengtu.008A7FA3
```

比较测试值是否为 `70`
```
008A7631 68 5C49F>push A3F0495C
008A7636 5F pop edi
008A7637 81C7 19E>add edi,C226E919
008A763D 81CF E8F>or edi,6578F7E8
008A7643 81F7 8DF>xor edi,677FF78D//edi=0x70
008A7649 9D popfd
008A764A 85C7 test edi,eax//
```

修改的 `eip` `8A9034`

`seh` 中修改 `eip`
```
008A83EF C701 A37>mov dword ptr ds:[ecx],zhengtu.008A7FA3
```

测试比较的结果
```
008A53C8 /0F84 4E4>je zhengtu.008A9F1C
```

`Test` `PEB.68`,2
```
008AA760 81F7 FC7>xor edi,BFFB77FC
008AA766 85C7 test edi,eax//和 2 做 test 运算
008AA768 873424 xchg dword ptr ss:[esp],esi
008AA76B 8BFE mov edi,esi
008AA76D 5E pop esi
008AA76E 0F84 631>je zhengtu.008AC3D7
```

最开始的反调试就到这结束,接着就是解压代码,获得几个外壳需要的 `API` 地址

再接着就是重定位 `call` 和 `jmp` 处的地址

//还原 `e8` 和 `e9` 后的地址值 也就是 `call` 和 `jmp` `bswap`
```
0075455D FC cld
0075455E AC lods byte ptr ds:[esi]
0075455F D0E8 shr al,1
00754561 80F8 74 cmp al,74
00754564 75 0E jnz short zhengtu.00754574
00754566 8B06 mov eax,dword ptr ds:[esi]
00754568 0FC8 bswap eax
0075456A 01C8 add eax,ecx
0075456C 8906 mov dword ptr ds:[esi],eax
0075456E 83C6 04 add esi,4
00754571 83E9 04 sub ecx,4
00754574 49 dec ecx
00754575 ^ 7F E7 jg short zhengtu.0075455E
00754577 59 pop ecx
00754578 5E pop esi
00754579 C3 retn
```

`SEH` 反跟踪修改 `eip` 的指令
```
008A5CDB C701 B45C8A00 mov dword ptr ds:[ecx],zhengtu.008A5CB4
```

//获得部分外壳需要的 `API`
```
008AC77F /0F84 E0000000 je zhengtu.008AC865
008AC785 |F7C3 00000080 test ebx,80000000
008AC78B |0F85 04000000 jnz zhengtu.008AC795
008AC791 |8D5C13 02 lea ebx,dword ptr ds:[ebx+edx+2]
008AC795 |81E3 FFFFFFFF and ebx,7FFFFFFF
008AC79B |53 push ebx
008AC79C |FF75 F8 push dword ptr ss:[ebp-8]
008AC79F |FF15 BC407500 call dword ptr ds:[<&kernel32.GetProcA>; kernel32.GetProcAddress
008AC7A5 |09C0 or eax,eax
008AC7A7 |0F84 0B000000 je zhengtu.008AC7B8
008AC7AD |8907 mov dword ptr ds:[edi],eax
008AC7AF |8345 F4 04 add dword ptr ss:[ebp-C],4
008AC7B3 ^|E9 A6FFFFFF jmp zhengtu.008AC75E
008AC7B8 |53 push ebx
```

`seh` 修改 `eip`
```
008A94C4 C701 B9C08A00 mov dword ptr ds:[ecx],zhengtu.008AC0B9
```

`seh` 修改 `eip`
```
008A4CDB C701 19A48A00 mov dword ptr ds:[ecx],zhengtu.008AA419
```

.....

`CheckRemoteDebuggerPresent` 检测是否被调试
```
007230CF 50 push eax//当前进程 ID
007230D0 FF55 F4 call dword ptr ss:[ebp-C]//CheckRemoteDebuggerPresent
007230D3 85C0 test eax,eax//测试返回值
```

......

之后就创建的 `16` 个子线程,很惭愧,由于功力不够,没有把子线程分析出来

......

在代码段 查找 `FF` `15` 就能知道,输入表的地址是 `61d000` 段

以下是壳解密 `API` 地址的代码部分分析
```
0072A862 8955 F8 mov dword ptr ss:[ebp-8],edx ; ebp.8=edx=API 加密值
0072A865 8945 FC mov dword ptr ss:[ebp-4],eax ; ebp.4=eax=hMoudle
0072A868 33C0 xor eax,eax
0072A86A 8945 F4 mov dword ptr ss:[ebp-C],eax ; ebp.c=0=计数用...
0072A86D 837D FC 00 cmp dword ptr ss:[ebp-4],0
0072A871 ^ 0F84 9694FEFF je zhengtu.00713D0D ; if hMoudle == NULL 跳
00752B0A 871424 xchg dword ptr ss:[esp],edx
00752B0D 5A pop edx ; add esp,4
00752B0E 8945 F4 mov dword ptr ss:[ebp-C],eax ; ebp.c=eax=0
00752B11 837D FC 00 cmp dword ptr ss:[ebp-4],0
00752B15 ^ 0F84 F211FCFF je zhengtu.00713D0D ; 比较模块句柄是否为 0
00752B1B 8B45 FC mov eax,dword ptr ss:[ebp-4]
00752B1E 83C0 3C add eax,3C ; 得到模块的 pe 的 RVA
00752B21 ^ E9 AF5CFDFF jmp zhengtu.007287D5
007287D5 8B00 mov eax,dword ptr ds:[eax] ; eax=RVA PE 结构
007287D7 0345 FC add eax,dword ptr ss:[ebp-4] ; RVA+hModule=PE 结构 VA
007287DA 83C0 18 add eax,18 ; +18=可选头
007287DD ^ E9 E6CAFEFF jmp zhengtu.007152C8
00731165 8945 F0 mov dword ptr ss:[ebp-10],eax ; ebp.10=PE.可选头
00731168 ^ E9 97B5FFFF jmp zhengtu.0072C704
0073421C 8B45 F0 mov eax,dword ptr ss:[ebp-10]
0073421F 8B40 60 mov eax,dword ptr ds:[eax+60] ; eax=数据目录表
00734222 0345 FC add eax,dword ptr ss:[ebp-4] ; +基址
00734225 8945 EC mov dword ptr ss:[ebp-14],eax ; ebp.14=eax=IET 输出表
00734228 8B45 EC mov eax,dword ptr ss:[ebp-14]
0073422B 8B40 18 mov eax,dword ptr ds:[eax+18] ; eax=IET.NumberOfFunctions 涵数个数
```



## 导出函数遍历与校验逻辑

```
0073422E      48               dec eax                        ; 个数-1
0073422F      E9 F6570000      jmp zhengtu.00739A2A
00739A2A      85C0             test eax,eax                   ; 如果输出函数个数小于=0,跳
00739A2C    ^ 0F8C DBA2FDFF    jl zhengtu.00713D0D
00739A32      40               inc eax                        ; 项++
00739A33      8945 DC          mov dword ptr ss:[ebp-24],eax    ; ebp.24=导出函数个数
00739A36      C745 E8 00000000 mov dword ptr ss:[ebp-18],0
00739A3D      8B45 EC          mov eax,dword ptr ss:[ebp-14]    ; eax=ebp.14=IET
00739A40      8B40 20          mov eax,dword ptr ds:[eax+20]    ; eax=IET.Base=基址
00739A43    ^ E9 805EFDFF      jmp zhengtu.0070F8C8
0070F8C8      0345 FC          add eax,dword ptr ss:[ebp-4]     ; eax+hModule=导出函数数组
0070F8CB      33D2             xor edx,edx
0070F8CD      52               push edx                       ; push 0
0070F8CE      50               push eax                       ; push 导出函数数组
0070F8CF      E9 D6C50000      jmp zhengtu.0071BEAA
0071BEAA      8B45 E8          mov eax,dword ptr ss:[ebp-18]    ; eax 已处理函数计数
00712610      C1E0 02          shl eax,2                      ; 序号*4
00712613      99               cdq
00752682      8B00             mov eax,dword ptr ds:[eax]       ; 取导出函数项的 RVA
0071B87F      0345 FC          add eax,dword ptr ss:[ebp-4]     ; RVA+hModule
00730F50      FF45 F4          inc dword ptr ss:[ebp-C]        ; 函数名数组++
007311AD     /0F85 1E7F0100    jnz zhengtu.007490D1             ; != NULL 跳
007311B3      |8B45 FC          mov eax,dword ptr ss:[ebp-4]     ; 得出函数名长度
007311B6      |8945 F4          mov dword ptr ss:[ebp-C],eax
007311B9      |C165 F8 08       shl dword ptr ss:[ebp-8],8       ; 函数名长度左移 8 位???
00708209      0FB600           movzx eax,byte ptr ds:[eax]      ; eax=取函数名的字节值
0070820C      0145 F8          add dword ptr ss:[ebp-8],eax     ; 长度左移 8 位后+函数名字节
0070820F      8B45 F8          mov eax,dword ptr ss:[ebp-8]
00708212      C1E0 0A          shl eax,0A                       ; 再左移 A 位

即出循环

00712CE2      8B45 F8          mov eax,dword ptr ss:[ebp-8]     ; 取函数名运算后的值
0074505D      C1E0 03          shl eax,3                        ; 运算后的值左移 3 位
007236AA      C1E8 0B          shr eax,0B                       ; 左移 B 位
007236AD      3145 F8          xor dword ptr ss:[ebp-8],eax     ; 异域
007236B0      8B45 F8          mov eax,dword ptr ss:[ebp-8]
007236B3      C1E0 0F          shl eax,0F                       ; 左移 F 位
007236B6    ^ E9 2520FFFF      jmp zhengtu.007156E0
007156E0      0145 F8          add dword ptr ss:[ebp-8],eax
007156E3      68 B6947100      push zhengtu.007194B6
007194B6      8B45 F8          mov eax,dword ptr ss:[ebp-8]     ; eax=函数名的运算值 ----函数返回
007194B9      8BE5             mov esp,ebp
007194BB      5D               pop ebp
007194BC      C3               retn
0071B882      E8 D4E80000      call <zhengtu.Get 函数校验值>//以上为该函数的运算
0071B887      3B45 F8          cmp eax,dword ptr ss:[ebp-8]     ; 比较还回的值校验值,是否是需要寻找的 API
0071B88A    ^ 0F85 9499FFFF    jnz zhengtu.00715224             ; 不相等继续
0071B890      E9 9AE30100      jmp zhengtu.00739C2F             ; 相等,跳-----------------------------------在此外下断,就能得到 API 的真实地址
```

## 修复 IAT 代码

随便找一块空白区域写上以下代码：

```
//修复输入表写上去的代码

00753F26      55               push ebp
00753F27      BE 68D06100      mov esi,zhengtu.0061D000        ; //未处理 IAT 首地址
00753F2C      AD               lods dword ptr ds:[esi]
00753F2D      3D 00000010      cmp eax,10000000
00753F32     ^ 77 F8           ja short zhengtu.00753F2C
00753F34      85C0             test eax,eax
00753F36     ^ 74 F4           je short zhengtu.00753F2C
00753F38      8935 40387500    mov dword ptr ds:[753840],esi   ; //保存 IAT 地址
00753F3E      BB 4B3F7500      mov ebx,zhengtu.00753F4B        ; //保存还回的地址
00753F43      891D 44387500    mov dword ptr ds:[753844],ebx
00753F49      FFD0             call eax
00753F4B      8B35 40387500    mov esi,dword ptr ds:[753840]   ; //还原 IAT 地址
00753F51      8946 FC          mov dword ptr ds:[esi-4],eax    ; //保存值
00753F54      81FE 20D36100    cmp esi,zhengtu.0061D320        ; //比较输入表是否被处理完毕
00753F5A     - 77 FE           ja short zhengtu.00753F5A       ; //大于进入死循环
00753F5C     ^ EB CE          jmp short zhengtu.00753F2C      ; //跳回接着处理
```

然后在壳解密出 API 真实地址处写上以下代码：

```
0070A27E      8B1D 44387500    mov ebx,dword ptr ds:[753844]   ; 返回自已写的代码处,继续处理下一个 API
0070A284      53               push ebx
0070A285      C3               retn
```

到此，代码已解开，IAT 也修复了，但 OEP 被抽取的代码，盼望大侠指点

`4dd466d0e1e0cc123afbe504da2462ca`
