
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



```
 004D3E9C     8A541A FF              mov dl,byte ptr ds:[edx+ebx-1]           ; 依次取字符  
 004D3EA0     E8 6B0EF3FF            call Unpacked.00404D10                   ; 字符的格式符合要求不？异常  
 004D3EA5     8B45 D4                mov eax,dword ptr ss:[ebp-2C]           ; 字符  
 004D3EA8     E8 4754F3FF            call Unpacked.004092F4                   ; 转 10 进制  
 004D3EAD     03F0                   add esi,eax                              ; 累加到 esi 中  
 004D3EAF     43                     inc ebx                                  ; count++  
 004D3EB0     83FB 11                cmp ebx,11                               ; count==0x11?  
 004D3EB3     ^ 75 E1                jnz short Unpacked.004D3E96  
 004D3EB5     8BC6                   mov eax,esi                              ; eax=esi  
 004D3EB7     03C0                   add eax,eax                              ; eax=eax*2  
 004D3EB9     8D0440                 lea eax,dword ptr ds:[eax+eax*2]         ; eax=eax*2  
 004D3EBC     83C0 12                add eax,12                               ; eax=eax+0x12  
 004D3EBF     8BF0                   mov esi,eax                              ; esi=eax  
 004D3EC1     8D45 D0                lea eax,dword ptr ss:[ebp-30]  
 004D3EC4     50                     push eax  
 004D3EC5     B9 03000000            mov ecx,3  
 004D3ECA     BA 11000000            mov edx,11  
 004D3ECF     8B45 F8                mov eax,dword ptr ss:[ebp-8]  
 004D3ED2     E8 6911F3FF            call Unpacked.00405040                   ; 从第 0x11 位取 3 个字符（刚才调用过）  
 004D3ED7     8B45 D0                mov eax,dword ptr ss:[ebp-30]           ; 返回的 3 字符  
 004D3EDA     E8 1554F3FF            call Unpacked.004092F4                   ; 看作 10 进制数  
 004D3EDF     3BF0                   cmp esi,eax                              ; 是否与刚才计算的 esi 相等  
 004D3EE1     74 0D                  je short Unpacked.004D3EF0               ; 不的就 voer  
```

`//就是取注册码的前 0x10 位为十个十进制数的和在乘 2 乘 3 加 0x12 必须与第 0x11 为到 0x13 位的 3 位十进制数相等，`

`//也就是（1+2+3+4+5+6+7+8+9+0+1+2+3+4+5+6）*2*3+0x12=414 所以第 0x11 位到 0x13 位必须是 414`

`//这是注册码的格式`

```
 004D3EE3     33C0                   xor eax,eax  
 004D3EE5     5A                     pop edx  
 004D3EE6     59                     pop ecx  
 004D3EE7     59                     pop ecx  
 004D3EE8     64:8910                mov dword ptr fs:[eax],edx  
 004D3EEB     E9 F9000000            jmp Unpacked.004D3FE9  
 004D3EF0     8D55 CC                lea edx,dword ptr ss:[ebp-34]  
 004D3EF3     8B45 FC                mov eax,dword ptr ss:[ebp-4]             ; 用户名  
 004D3EF6     E8 B9FBFFFF            call Unpacked.004D3AB4                   ; 计算用户名，下面跟进  
 004D3EFB     8B45 CC                mov eax,dword ptr ss:[ebp-34]           ; 通过用户名返回的 10 进制的字符串  
 004D3EFE     E8 6554F3FF            call Unpacked.00409368                   ; 10 进制的字符串转换成数字  
 004D3F03     52                     push edx  
 004D3F04     50                     push eax  
 004D3F05     8B45 E4                mov eax,dword ptr ss:[ebp-1C]            ; 机器码字符串  
 004D3F08     E8 5B54F3FF            call Unpacked.00409368                   ; 转成数字  
 004D3F0D     030424                 add eax,dword ptr ss:[esp]               ; （机器码）0x21E99E83 和 0x481F39C4 相加  
 004D3F10     135424 04              adc edx,dword ptr ss:[esp+4]             ; 高位  
 004D3F14     83C4 08                add esp,8  
 004D3F17     05 7617D775            add eax,75D71776                         ; eax=+0x75d71776  
 004D3F1C     83D2 00                adc edx,0  
 004D3F1F     8945 D8                mov dword ptr ss:[ebp-28],eax            ; 保存起来  
 004D3F22     8955 DC                mov dword ptr ss:[ebp-24],edx  
 004D3F25     8D45 EC                lea eax,dword ptr ss:[ebp-14]  
 004D3F28     50                     push eax  
 004D3F29     8B45 F8                mov eax,dword ptr ss:[ebp-8]             ; 假码  
 004D3F2C     E8 B70EF3FF            call Unpacked.00404DE8                   ; 假码的位数  
 004D3F31     8BC8                   mov ecx,eax  
 004D3F33     83E9 13                sub ecx,13                               ; 取假码没处理字符的位数  
 004D3F36     BA 14000000            mov edx,14                               ; 从第 0x14 位开始取  
 004D3F3B     8B45 F8                mov eax,dword ptr ss:[ebp-8]             ; 假码  
 004D3F3E     E8 FD10F3FF            call Unpacked.00405040                   ; 取子串  
 004D3F43     837D EC 00             cmp dword ptr ss:[ebp-14],0              ; 子串为空吗？  
 004D3F47     75 0D                  jnz short Unpacked.004D3F56  
 004D3F49     33C0                   xor eax,eax  
 004D3F4B     5A                     pop edx  
 004D3F4C     59                     pop ecx  
 004D3F4D     59                     pop ecx  
 004D3F4E     64:8910                mov dword ptr fs:[eax],edx  
 004D3F51     E9 93000000            jmp Unpacked.004D3FE9  
 004D3F56     BB 01000000            mov ebx,1                                ; 下面的有一点不太好  
 004D3F5B     8D45 E8                lea eax,dword ptr ss:[ebp-18]            ; 理解，我也说不太明白！  
 004D3F5E     E8 CD0BF3FF            call Unpacked.00404B30  
 004D3F63     8D45 C8                lea eax,dword ptr ss:[ebp-38]            ; 循环入口  
 004D3F66     8B55 EC                mov edx,dword ptr ss:[ebp-14]            ; 从 0x14 位开始的假码的子串  
 004D3F69     8A541A FF              mov dl,byte ptr ds:[edx+ebx-1]  
 004D3F6D     E8 9E0DF3FF            call Unpacked.00404D10                   ; 字符的格式符合要求不？异常  
 004D3F72     8B45 C8                mov eax,dword ptr ss:[ebp-38]            ; 每个字符  
 004D3F75     E8 7A53F3FF            call Unpacked.004092F4  
 004D3F7A     8BF0                   mov esi,eax                              ; 字符转成数字  
 004D3F7C     8D741E 01              lea esi,dword ptr ds:[esi+ebx+1]         ; 把这个数字+ebx+1 当作字符指针  
 004D3F80     8BDE                   mov ebx,esi                              ; 准备取“真正”要判断的字符  
 004D3F82     8D45 C4                lea eax,dword ptr ss:[ebp-3C]  
 004D3F85     8B55 EC                mov edx,dword ptr ss:[ebp-14]  
 004D3F88     8A541A FF              mov dl,byte ptr ds:[edx+ebx-1]           ; 取“真正”要判断的字符了  
 004D3F8C     E8 7F0DF3FF            call Unpacked.00404D10                   ; 字符的格式符合要求不？异常  
 004D3F91     8B55 C4                mov edx,dword ptr ss:[ebp-3C]  
 004D3F94     8D45 E8                lea eax,dword ptr ss:[ebp-18]            ; 地址  
 004D3F97     E8 540EF3FF            call Unpacked.00404DF0                   ; 保存取的字符的  
 004D3F9C     43                     inc ebx  
 004D3F9D     8B45 EC                mov eax,dword ptr ss:[ebp-14]  
```



004D3FA0 E8 430EF3FF call Unpacked.00404DE8 ; 位数

004D3FA5 3BD8 cmp ebx,eax

004D3FA7 ^ 7C BA jl short Unpacked.004D3F63

//这段我说不明白，根据取得的一个字符（这个字符只起定位作用）的值来取真正要取的字符。自己琢磨吧！

//如果取得的字符都为 0(究竟取什么值由写注册机的人决定 ) 那么取的字符就是 X（即 0X0X0X0X0X0X0X0X0X0X）

004D3FA9 FF75 DC push dword ptr ss:\[ebp-24\]

004D3FAC FF75 D8 push dword ptr ss:\[ebp-28\] ; 0xDFDFEFBD

004D3FAF 8D45 C0 lea eax,dword ptr ss:\[ebp-40\]

004D3FB2 E8 CD52F3FF call Unpacked.00409284 ; 转 10 进制字符串

004D3FB7 8B55 C0 mov edx,dword ptr ss:\[ebp-40\] ; (ASCII "3755995069")正确的字符串

004D3FBA 8B45 E8 mov eax,dword ptr ss:\[ebp-18\] ; 4d3f63 循环处取的假串

004D3FBD E8 6A0FF3FF call Unpacked.00404F2C ; 真串和假串比较

//如果取得的字符都为 0 "3755995069" 的实际应存为"03070505090905000609"

004D3FC2 74 0A je short Unpacked.004D3FCE ; 不相等就 over 了

004D3FC4 33C0 xor eax,eax

004D3FC6 5A pop edx

004D3FC7 59 pop ecx

004D3FC8 59 pop ecx

004D3FC9 64:8910 mov dword ptr fs:\[eax\],edx

004D3FCC EB 1B jmp short Unpacked.004D3FE9

004D3FCE C745 F4 FFFFFFFFFFFF mov dword ptr ss:\[ebp-C\],-1

004D3FD5 33C0 xor eax,eax

004D3FD7 5A pop edx

004D3FD8 59 pop ecx

//004D3EF6 E8 B9FBFFFF call Unpacked.004D3AB4 ; 计算用户名

004D3ABA 53 push ebx

004D3ABB 56 push esi

004D3ABC 57 push edi

004D3ABD 33C9 xor ecx,ecx

004D3ABF 894D EC mov dword ptr ss:\[ebp-14\],ecx

004D3AC2 8BFA mov edi,edx

004D3AC4 8945 FC mov dword ptr ss:\[ebp-4\],eax

004D3AC7 8B45 FC mov eax,dword ptr ss:\[ebp-4\]

004D3ACA E8 0115F3FF call Unpacked.00404FD0

004D3ACF 33C0 xor eax,eax

004D3AD1 55 push ebp

004D3AD2 68 813B4D00 push Unpacked.004D3B81

004D3AD7 64:FF30 push dword ptr fs:\[eax\]

004D3ADA 64:8920 mov dword ptr fs:\[eax\],esp

004D3ADD C745 F0 00000000 mov dword ptr ss:\[ebp-10\],0

004D3AE4 C745 F4 00000000 mov dword ptr ss:\[ebp-C\],0

004D3AEB 8B45 FC mov eax,dword ptr ss:\[ebp-4\]

004D3AEE E8 F512F3FF call Unpacked.00404DE8 ; 用户名

004D3AF3 8BF0 mov esi,eax ; 用户名位数

004D3AF5 85F6 test esi,esi

004D3AF7 7E 23 jle short Unpacked.004D3B1C

004D3AF9 BB 01000000 mov ebx,1 ; count

004D3AFE 8BCB mov ecx,ebx

004D3B00 8B45 FC mov eax,dword ptr ss:\[ebp-4\]

004D3B03 0FB64418 FF movzx eax,byte ptr ds:\[eax+ebx-1\] ; 取用户名的每个字符扩展到 eax 中

004D3B08 D3E0 shl eax,cl ; 左移 count 位， 即字符在用户名的第几位就左移几位

004D3B0A 33D2 xor edx,edx

004D3B0C 0345 F0 add eax,dword ptr ss:\[ebp-10\] ; 累加的低位

004D3B0F 1355 F4 adc edx,dword ptr ss:\[ebp-C\] ; 累加的高位

004D3B12 8945 F0 mov dword ptr ss:\[ebp-10\],eax ; 保存以便下次循环累加

004D3B15 8955 F4 mov dword ptr ss:\[ebp-C\],edx

004D3B18 43 inc ebx ; count++

004D3B19 4E dec esi ; 循环控制

004D3B1A ^ 75 E2 jnz short Unpacked.004D3AFE

004D3B1C 8B45 FC mov eax,dword ptr ss:\[ebp-4\] ; 用户名

004D3B1F E8 C412F3FF call Unpacked.00404DE8 ; eax 返回用户名的位数

004D3B24 8B55 FC mov edx,dword ptr ss:\[ebp-4\] ; 用户名

004D3B27 0FB64402 FF movzx eax,byte ptr ds:\[edx+eax-1\] ; 取用户名的最后的一个字符

004D3B2C 69C0 80969800 imul eax,eax,989680 ; eax=eax*0x989680

004D3B32 33D2 xor edx,edx

004D3B34 3345 F0 xor eax,dword ptr ss:\[ebp-10\] ; eax=eax xor 0x1b44

004D3B37 3355 F4 xor edx,dword ptr ss:\[ebp-C\]

004D3B3A 8945 F0 mov dword ptr ss:\[ebp-10\],eax ; 0x481f39c4

004D3B3D 8955 F4 mov dword ptr ss:\[ebp-C\],edx

004D3B40 FF75 F4 push dword ptr ss:\[ebp-C\] ; 0x00000000

004D3B43 FF75 F0 push dword ptr ss:\[ebp-10\] ; 0x481f39c4

004D3B46 8D45 EC lea eax,dword ptr ss:\[ebp-14\]

004D3B49 E8 3657F3FF call Unpacked.00409284 ; 转成 10 的 ascII 码字符串

004D3B4E 8B45 EC mov eax,dword ptr ss:\[ebp-14\] ; (ASCII "1210005956")

004D3B51 8D55 FC lea edx,dword ptr ss:\[ebp-4\]

004D3B54 E8 DF54F3FF call Unpacked.00409038

004D3B59 8BC7 mov eax,edi

004D3B5B 8B55 FC mov edx,dword ptr ss:\[ebp-4\]

004D3B5E E8 2110F3FF call Unpacked.00404B84

004D3B63 33C0 xor eax,eax

004D3B65 5A pop edx

004D3B66 59 pop ecx

004D3B67 59 pop ecx

004D3B68 64:8910 mov dword ptr fs:\[eax\],edx

004D3B6B 68 883B4D00 push Unpacked.004D3B88

004D3B70 8D45 EC lea eax,dword ptr ss:\[ebp-14\]

004D3B73 E8 B80FF3FF call Unpacked.00404B30

004D3B78 8D45 FC lea eax,dword ptr ss:\[ebp-4\]

004D3B7B E8 B00FF3FF call Unpacked.00404B30

004D3B80 C3 retn

//注册机 c 语言源码 在 c- free 中编译通过，注册机没有经过测试，应该有很多 bug ,我和菜，高手见笑

```c
#include "stdio.h"

main()
{
    char username\[50\],s\[100\];
    unsigned long machinecode;
    unsigned long sum=0,temp=0,i,j;
    printf("%d",sizeof(char));
    printf("please input machinecode\n");
    scanf("%u",&machinecode);
    printf("please input username\n");
    scanf("%s",&username);
       for(i=0;i<strlen(username);i++)
}
```



```c
       {
          temp=(unsigned long)username[i];
          temp<<=(i+1);
          sum+=temp;
       }

       /*  machinecode=568958595;   //我的是这个*/
       sum^=username[strlen(username)-1]*0x989680;
       sum+=machinecode;
       sum+=0x75d71776;
       sprintf(s,"%u",sum);
       printf("code: 1234567890123456414");
       i=0;
       while(s[i]!=0)
          { printf("0");
            putchar(s[i++]);
          }
       printf("\n");
       getch();
   }
```

--------------------------------------------------

**【经验总结】**

取注册码的前 `0x10` 位为十个十进制数的和在乘 2 乘 3 加 `0x12` 必须与第 `0x11` 为到 `0x13` 位的 3 位十进制数相等，也就是说（1+2+3+4+5+6+7+8+9+0+1+2+3+4+5+6）*2*3+0x12=414 所以第 `0x11` 位到 `0x13` 位必须是 `414`，前 `0x10` 为可以是任意 10 进制值

这就是注册码有 N 多了！

通过用户名和机器码计算出一串数字存放在的 `0x14` 位以后，注意它不是连续存放的，而是初始化一片内存区域，然后读取某些值来确定存放的位值，注册码比较时先验证 1 至 `0x14` 位，看看是否符合要求，再读取后面的有效值。进行比较

我把那片内存初始化为 `0000000000000000000000000000000000000000000` 那么有效值就存放在 `0X0X0X0X0X0X0X0X0X0X0X0X` 的 X 位置,用户名和机器码的具体计算过程看注册机和分析破解过程吧！好累，

**//我的注册信息**
`Username`: `pediy`
`machine`: `568958595`
`code`: `123456789012345641403070505090905000609`

--------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 31 日 2:49:56
`4dd466d0e1e0cc123afbe504da2462ca`
