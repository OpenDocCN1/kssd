
# 我的第一篇菜文 文赏图历险之超级女友

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-393.htm`](https://www.pediy.com/kssd/pediy09/pediy09-393.htm)

> 【文章标题】: 我的第一篇菜文 文赏图历险之超级女友  
> 【文章作者】: o 飘 o  
> 【作者邮箱】: `78122163@qq.com`  
> 【作者 QQ 号】: `78122163`  
> 【软件名称】: 赏图历险之超级女友  
> 【软件大小】: `15M` `Version` `2.9(Build` `2.9.8.665)`  
> 【下载地址】: 天空软件站  
> 【加壳方式】: `ASPack` `v2.12`  
> 【保护方式】: 序列号  
> 【编写语言】: `Borland` `Delphi`  
> 【使用工具】: `Ollydbg.exe`, `fi`, `AspackDie`；  
> 【操作平台】: `win2000`  
> 【软件介绍】: 天空软件站  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   
> \[想说的话\] 3 年前在就书滩上看到一本关于加密与解密的书，从那以后迷上你电脑，可惜由于当时没有电脑，我就经常出没于网吧      
>   
> 学习那些基础不能在基础的知识，我是学土墓建筑工程的，我们专业开了一门 `c` 语言，别的在也也没接受到正规培训了，幸运的是  
>   
> 一次偶然的我在我们学校的图书馆里看到了 《加密与解密》（第二版）我按照书中对读者的要求，一步一步的学习，同时也  
>   
> 一步一步的进步着，`汇编`，`api`，`windows` 程序设计 ；我的汇编是一点一点的看书啃出来的。现在想想真的不太容易....  
>   
>   
>   
> 想说的太多了，但没有头绪了，只说 `thanks` 看学论坛， 和在网上无私奉献的人们！    
>   
> 路漫漫其修远，我将 `Go` `on!`  
>   
> 用 `fi` 查壳 `aspack2.12` 用 `AspackDie` 完美脱掉，不知道为什么，我手动脱完壳后有问题，不能成功加载 `DirectX`, `没办法菜鸟就是`  
>   
> 这样的，哈哈！我没有用 `DeDe` 因为我的电脑太次了，`dede` 一分析就死掉，你的机器肯定没问题，因为我的硬盘才４G，哈哈；  
>   
> 你的机器肯定没问题！输入加码后没有任何提示！  
>   
>   
> 这个软件把注册码保存到注册表里，启动时验证！下 `api` `RegOpenKeyExA` 和 `RegQueryValueExA` 断点。  
>   
>   
> 输入注册码：  
>   
>   
> `user`: `pediy`,  
>   
> `code`: `123456789012345678901234567890123456789` ;  
>   
> 我的机器码是：“568958595”  
>   
>   
> 关闭软件，可以在注册表的找到  
>   
>   
> \[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\EPAGame\]  
>   
> "User"="pediy"  
>   
> "Key"="123456789012345678901234567890123456789"  
>   
>   
> 好了，用 `od` 载入，F9 观察堆栈的变化可以看到整个的取间值过程了，到  
>   
>   
> 0012FDD0 0000045C |hKey = 45C  
> 0012FDD4 004D425C |ValueName = "Key"  
> 0012FDD8 00000000 |Reserved = NULL  
> 0012FDDC 0012FDFC |pValueType = 0012FDFC  
> 0012FDE0 00F1C4F8 |Buffer = 00F1C4F8  
> 0012FDE4 0012FE0C \\pBufSize = 0012FE0C  
>   
>   
> 是已取完；逐个退出 call 来到 004E82F9004E82DD          
>   
>   
> 004E82E2 8B00 mov eax,dword ptr ds:\[eax\]  
> 004E82E4 E8 F709F8FF call Unpacked.00468CE0  
> 004E82E9 E8 7AC2F1FF call Unpacked.00404568  
> 004E82EE 8D55 E0 lea edx,dword ptr ss:\[ebp-20\]  
> 004E82F1 8D45 E4 lea eax,dword ptr ss:\[ebp-1C\]  
> 004E82F4 E8 43BEFEFF call Unpacked.004D413C ; 通过注册表取用户名和注册码  
> 004E82F9 8B45 FC mov eax,dword ptr ss:\[ebp-4\]  
> 004E82FC 05 90060000 add eax,690  
> 004E8301 8B55 E4 mov edx,dword ptr ss:\[ebp-1C\]  
> 004E8304 E8 7BC8F1FF call Unpacked.00404B84  
> 004E8309 8B55 E0 mov edx,dword ptr ss:\[ebp-20\] ; 假码  
> 004E830C 8B45 E4 mov eax,dword ptr ss:\[ebp-1C\] ; 用户名  
> 004E830F E8 00BBFEFF call Unpacked.004D3E14 ; 算法和比较 call 跟进  
> 004E8314 8BD0 mov edx,eax ; 相等的话 eax 返回 0xffffffff or 0  
> 004E8316 83FA 01 cmp edx,1  
> 004E8319 1BD2 sbb edx,edx  
> 004E831B F7DA neg edx  
> 004E831D 8B45 FC mov eax,dword ptr ss:\[ebp-4\]  
> 004E8320 8B80 88040000 mov eax,dword ptr ds:\[eax+488\]  
> 004E8326 E8 A106F6FF call Unpacked.004489CC ; 置标记  
> 004E832B B8 89130000 mov eax,1389  
> 004E8330 E8 7BB0F1FF call Unpacked.004033B0  
> 004E8335 8BD0 mov edx,eax  
> 004E8337 8B45 FC mov eax,dword ptr ss:\[ebp-4\]  
> 004E833A 8B98 8C040000 mov ebx,dword ptr ds:\[eax+48C\]  
>   
>   
>   
> //004E830F E8 00BBFEFF call Unpacked.004D3E14 ; 算法和比较 call 跟进  
>   
>   
> 004D3E14 55 push ebp  
> 004D3E15 8BEC mov ebp,esp  
> 004D3E17 B9 08000000 mov ecx,8  
> 004D3E1C 6A 00 push 0  
> 004D3E1E 6A 00 push 0  
> 004D3E20 49 dec ecx  
> 004D3E21 ^ 75 F9 jnz short Unpacked.004D3E1C  
> 004D3E23 53 push ebx  
> 004D3E24 56 push esi  
> 004D3E25 57 push edi  
> 004D3E26 8955 F8 mov dword ptr ss:\[ebp-8\],edx  
> 004D3E29 8945 FC mov dword ptr ss:\[ebp-4\],eax  
> 004D3E2C 8B45 FC mov eax,dword ptr ss:\[ebp-4\] ; 用户名  
> 004D3E2F E8 9C11F3FF call Unpacked.00404FD0  
> 004D3E34 8B45 F8 mov eax,dword ptr ss:\[ebp-8\] ; 假码  
> 004D3E37 E8 9411F3FF call Unpacked.00404FD0  
> 004D3E3C 33C0 xor eax,eax  
> 004D3E3E 55 push ebp  
> 004D3E3F 68 1E404D00 push Unpacked.004D401E  
> 004D3E44 64:FF30 push dword ptr fs:\[eax\]  
> 004D3E47 64:8920 mov dword ptr fs:\[eax\],esp  
> 004D3E4A 33C0 xor eax,eax  
> 004D3E4C 8945 F4 mov dword ptr ss:\[ebp-C\],eax  
> 004D3E4F 837D FC 00 cmp dword ptr ss:\[ebp-4\],0  
> 004D3E53 0F84 90010000 je Unpacked.004D3FE9 ; 用户名==0？  
> 004D3E59 837D F8 00 cmp dword ptr ss:\[ebp-8\],0  
> 004D3E5D 0F84 86010000 je Unpacked.004D3FE9 ; 假码==0？  
> 004D3E63 33C0 xor eax,eax  
> 004D3E65 55 push ebp  
> 004D3E66 68 DF3F4D00 push Unpacked.004D3FDF  
> 004D3E6B 64:FF30 push dword ptr fs:\[eax\]  
> 004D3E6E 64:8920 mov dword ptr fs:\[eax\],esp  
> 004D3E71 8D45 E4 lea eax,dword ptr ss:\[ebp-1C\] ; 保存机器码的变量  
> 004D3E74 E8 23FFFFFF call Unpacked.004D3D9C ; 计算机器码  
> //计算机器码的 call 我也大致跟了一下，用了个 cpuid 指令得到了  
> //0012FBFC 00000672  
> //0012FC00 00000000  
> //0012FC04 00000000  
> //0012FC08 0383F9FF  
> //0012FC0C 00000000  
> //然后加在一起。在经过和用户名差不多的计算过程的到的。我的是“568958595”  
> 004D3E79 8D45 F0 lea eax,dword ptr ss:\[ebp-10\]  
> 004D3E7C 50 push eax ; 反回值  
> 004D3E7D B9 10000000 mov ecx,10 ; 取子字符串的位数  
> 004D3E82 BA 01000000 mov edx,1 ; 从第几位取子字符串  
> 004D3E87 8B45 F8 mov eax,dword ptr ss:\[ebp-8\] ; 假码  
> 004D3E8A E8 B111F3FF call Unpacked.00405040  
> 004D3E8F 33F6 xor esi,esi  
> 004D3E91 BB 01000000 mov ebx,1 ; count  
> 004D3E96 8D45 D4 lea eax,dword ptr ss:\[ebp-2C\] ; 循环入口  
> 004D3E99 8B55 F0 mov edx,dword ptr ss:\[ebp-10\] ; 假码

```
004E82E2       8B00                    mov eax,dword ptr ds:[eax]
004E82E4       E8 F709F8FF             call Unpacked.00468CE0
004E82E9       E8 7AC2F1FF             call Unpacked.00404568
004E82EE       8D55 E0                 lea edx,dword ptr ss:[ebp-20]
004E82F1       8D45 E4                 lea eax,dword ptr ss:[ebp-1C]
004E82F4       E8 43BEFEFF             call Unpacked.004D413C         ; 通过注册表取用户名和注册码
004E82F9       8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004E82FC       05 90060000             add eax,690
004E8301       8B55 E4                 mov edx,dword ptr ss:[ebp-1C]
004E8304       E8 7BC8F1FF             call Unpacked.00404B84
004E8309       8B55 E0                 mov edx,dword ptr ss:[ebp-20]    ; 假码
004E830C       8B45 E4                 mov eax,dword ptr ss:[ebp-1C]    ; 用户名
004E830F       E8 00BBFEFF             call Unpacked.004D3E14         ; 算法和比较 call 跟进
004E8314       8BD0                    mov edx,eax                    ; 相等的话 eax 返回 0xffffffff  or 0
004E8316       83FA 01                 cmp edx,1
004E8319       1BD2                    sbb edx,edx
004E831B       F7DA                    neg edx
004E831D       8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004E8320       8B80 88040000           mov eax,dword ptr ds:[eax+488]
004E8326       E8 A106F6FF             call Unpacked.004489CC         ; 置标记
004E832B       B8 89130000             mov eax,1389
004E8330       E8 7BB0F1FF             call Unpacked.004033B0
004E8335       8BD0                    mov edx,eax
004E8337       8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004E833A       8B98 8C040000           mov ebx,dword ptr ds:[eax+48C]
```

```
//004E830F       E8 00BBFEFF             call Unpacked.004D3E14 ; 算法和比较 call 跟进
```

```
004D3E14       55                    push ebp
004D3E15       8BEC                  mov ebp,esp
004D3E17       B9 08000000           mov ecx,8
004D3E1C       6A 00                 push 0
004D3E1E       6A 00                 push 0
004D3E20       49                    dec ecx
004D3E21      ^ 75 F9               jnz short Unpacked.004D3E1C
004D3E23       53                    push ebx
004D3E24       56                    push esi
004D3E25       57                    push edi
004D3E26       8955 F8               mov dword ptr ss:[ebp-8],edx
004D3E29       8945 FC               mov dword ptr ss:[ebp-4],eax
004D3E2C       8B45 FC               mov eax,dword ptr ss:[ebp-4]           ; 用户名
004D3E2F       E8 9C11F3FF           call Unpacked.00404FD0
004D3E34       8B45 F8               mov eax,dword ptr ss:[ebp-8]           ; 假码
004D3E37       E8 9411F3FF           call Unpacked.00404FD0
004D3E3C       33C0                  xor eax,eax
004D3E3E       55                    push ebp
004D3E3F       68 1E404D00           push Unpacked.004D401E
004D3E44       64:FF30               push dword ptr fs:[eax]
004D3E47       64:8920               mov dword ptr fs:[eax],esp
004D3E4A       33C0                  xor eax,eax
004D3E4C       8945 F4               mov dword ptr ss:[ebp-C],eax
004D3E4F       837D FC 00            cmp dword ptr ss:[ebp-4],0
004D3E53       0F84 90010000         je Unpacked.004D3FE9                 ; 用户名==0？
004D3E59       837D F8 00            cmp dword ptr ss:[ebp-8],0
004D3E5D       0F84 86010000         je Unpacked.004D3FE9                 ; 假码==0？
004D3E63       33C0                  xor eax,eax
004D3E65       55                    push ebp
004D3E66       68 DF3F4D00           push Unpacked.004D3FDF
004D3E6B       64:FF30               push dword ptr fs:[eax]
004D3E6E       64:8920               mov dword ptr fs:[eax],esp
004D3E71       8D45 E4               lea eax,dword ptr ss:[ebp-1C]         ; 保存机器码的变量
004D3E74       E8 23FFFFFF           call Unpacked.004D3D9C               ; 计算机器码
//计算机器码的 call 我也大致跟了一下，用了个 cpuid 指令得到了
//0012FBFC    00000672
//0012FC00    00000000
//0012FC04    00000000
//0012FC08    0383F9FF
//0012FC0C    00000000
//然后加在一起。在经过和用户名差不多的计算过程的到的。我的是“568958595”
004D3E79       8D45 F0               lea eax,dword ptr ss:[ebp-10]
004D3E7C       50                    push eax                              ; 反回值
004D3E7D       B9 10000000           mov ecx,10                            ; 取子字符串的位数
004D3E82       BA 01000000           mov edx,1                             ; 从第几位取子字符串
004D3E87       8B45 F8               mov eax,dword ptr ss:[ebp-8]           ; 假码
004D3E8A       E8 B111F3FF           call Unpacked.00405040
004D3E8F       33F6                  xor esi,esi
004D3E91       BB 01000000           mov ebx,1                             ; count
004D3E96       8D45 D4               lea eax,dword ptr ss:[ebp-2C]         ; 循环入口
004D3E99       8B55 F0               mov edx,dword ptr ss:[ebp-10]         ; 假码
```
