
# 南山软件屋加密程序

> 原文：[`www.pediy.com/kssd/pediy06/pediy6988.htm`](https://www.pediy.com/kssd/pediy06/pediy6988.htm)

密码壳——南山软件屋加密程序的 `UnPackMe` 脱壳

直接下载： `http://tongtian.net/pediybbs/download.php?id=1531`

软件大小： 210.53 KB

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg`

## 脱壳过程

`goodmorning` 说：“保证你没见过的新式壳的 `unpackme`”。其实这是一个密码壳。BTW：加壳的是 `XP` 的计算器。

对于密码壳保护，侧重的是“密码”，但是这个 `DEMO` 版南山软件屋加密软件加密的 `unpackme.exe` 却很难称得上是密码壳保护。为何？看看吧。我只是说的这个 `unpackme`，或许正式版的密码保护是非常坚固的。

### 一、Kill 掉反跟踪

用 `Ollydbg` 载入运行一下却发现被自动关闭了，感谢作者没有使用让机子死悄悄的代码。

重新载入，下断 `BP TerminateProcess`，`F9` 运行，被断下

```
77E416B4     837C24 04 00           cmp dword ptr ss:[esp+4],0//断在这

77E416B9     74 18                  je short kernel32.77E416D3

77E416BB     FF7424 08              push dword ptr ss:[esp+8]

77E416BF     FF7424 08              push dword ptr ss:[esp+8]

77E416C3     FF15 EC13E477          call dword ptr ds:[<&ntdll.NtTerminateProcess>]

77E416C9     85C0                   test eax,eax

77E416CB     7C 0F                  jl short kernel32.77E416DC

77E416CD     33C0                   xor eax,eax

77E416CF     40                     inc eax

77E416D0     C2 0800               retn 8//返回到
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

看看 `BP TerminateProcess` 堆栈：

```
0012F880    00454128   /CALL 到 TerminateProcess 来自 unpackme.00454123

0012F884    0000008C   |hProcess = 0000008C (window)

0012F888    00000000   ExitCode = 0
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`Ctrl+G`：`00454128` 很容易看出这段代码的含义。即：取系统路径+`EXPLORER.EXE`，比较加壳程序的父进程是否是你系统的 `EXPLORER.EXE`。呵呵，很熟悉的代码，记得 `看雪` 老师就曾经写过这个 `Anti` 来对付修改的 `flyODBG`。

再次载入，`BP GetWindowsDirectoryA`

```
00453FD1     E8 A229FBFF            call <jmp.&kernel32.GetWindowsDirectoryA>

00453FD6     8BC8                   mov ecx,eax

00453FD8     8D95 C8FAFFFF          lea edx,dword ptr ss:[ebp-538]

00453FDE     8D45 F4               lea eax,dword ptr ss:[ebp-C]

00453FE1     E8 C204FBFF            call unpackme.004044A8

00453FE6     8D95 C4FAFFFF          lea edx,dword ptr ss:[ebp-53C]

00453FEC     8B45 F4               mov eax,dword ptr ss:[ebp-C]

00453FEF     E8 C842FBFF            call unpackme.004082BC

00453FF4     8B95 C4FAFFFF          mov edx,dword ptr ss:[ebp-53C]

00453FFA     8D45 F4               lea eax,dword ptr ss:[ebp-C]

00453FFD     B9 68414500            mov ecx,unpackme.00454168 ; ASCII "EXPLORER.EXE"

00454002     E8 B506FBFF            call unpackme.004046BC

00454007     33D2                   xor edx,edx

00454009     B8 0F000000            mov eax,0F

0045400E     E8 69FDFFFF            call unpackme.00453D7C

00454013     8BD8                   mov ebx,eax

00454015     8D95 C8FEFFFF          lea edx,dword ptr ss:[ebp-138]

0045401B     8BC3                   mov eax,ebx

0045401D     E8 7AFDFFFF            call unpackme.00453D9C

00454022     83F8 01               cmp eax,1

00454025     1BC0                   sbb eax,eax

00454027     40                     inc eax

00454028     E9 E3000000            jmp unpackme.00454110

0045402D     8D85 C0FAFFFF          lea eax,dword ptr ss:[ebp-540]

00454033     8D95 ECFEFFFF          lea edx,dword ptr ss:[ebp-114]

00454039     B9 04010000            mov ecx,104

0045403E     E8 DD05FBFF            call unpackme.00404620

00454043     8B85 C0FAFFFF          mov eax,dword ptr ss:[ebp-540]

00454049     8D55 F0               lea edx,dword ptr ss:[ebp-10]

0045404C     E8 2F48FBFF            call unpackme.00408880

00454051     8D95 B8FAFFFF          lea edx,dword ptr ss:[ebp-548]

00454057     33C0                   xor eax,eax

00454059     E8 02EAFAFF            call unpackme.00402A60

0045405E     8B85 B8FAFFFF          mov eax,dword ptr ss:[ebp-548]

00454064     8D95 BCFAFFFF          lea edx,dword ptr ss:[ebp-544]

0045406A     E8 1148FBFF            call unpackme.00408880

0045406F     8B95 BCFAFFFF          mov edx,dword ptr ss:[ebp-544]

00454075     8B45 F0               mov eax,dword ptr ss:[ebp-10]

00454078     E8 3707FBFF            call unpackme.004047B4

0045407D     75 1A                  jnz short unpackme.00454099

0045407F     8B85 E0FEFFFF          mov eax,dword ptr ss:[ebp-120]

00454085     8BF0                   mov esi,eax

00454087     50                     push eax

00454088     6A FF                   push -1

0045408A     68 FF0F1F00            push 1F0FFF

0045408F     E8 6429FBFF            call <jmp.&kernel32.OpenProcess>

00454094     8945 FC               mov dword ptr ss:[ebp-4],eax

00454097     EB 64                  jmp short unpackme.004540FD

00454099     8D85 B0FAFFFF          lea eax,dword ptr ss:[ebp-550]

0045409F     8D95 ECFEFFFF          lea edx,dword ptr ss:[ebp-114]

004540A5     B9 04010000            mov ecx,104

004540AA     E8 7105FBFF            call unpackme.00404620

004540AF     8B85 B0FAFFFF          mov eax,dword ptr ss:[ebp-550]

004540B5     8D95 B4FAFFFF          lea edx,dword ptr ss:[ebp-54C]

004540BB     E8 FC41FBFF            call unpackme.004082BC

004540C0     8B85 B4FAFFFF          mov eax,dword ptr ss:[ebp-54C]

004540C6     50                     push eax

004540C7     8D95 A8FAFFFF          lea edx,dword ptr ss:[ebp-558]

004540CD     8B45 F4               mov eax,dword ptr ss:[ebp-C]

004540D0     E8 AB47FBFF            call unpackme.00408880

004540D5     8B85 A8FAFFFF          mov eax,dword ptr ss:[ebp-558]

004540DB     8D95 ACFAFFFF          lea edx,dword ptr ss:[ebp-554]

004540E1     E8 D641FBFF            call unpackme.004082BC

004540E6     8B95 ACFAFFFF          mov edx,dword ptr ss:[ebp-554]

004540EC     58                     pop eax

004540ED     E8 C206FBFF            call unpackme.004047B4

004540F2     75 09                  jnz short unpackme.004540FD

004540F4     8B85 D0FEFFFF          mov eax,dword ptr ss:[ebp-130]

004540FA     8945 F8               mov dword ptr ss:[ebp-8],eax

004540FD     8D95 C8FEFFFF          lea edx,dword ptr ss:[ebp-138]

00454103     8BC3                   mov eax,ebx

00454105     E8 B2FCFFFF            call unpackme.00453DBC

0045410A     83F8 01               cmp eax,1

0045410D     1BC0                   sbb eax,eax

0045410F     40                     inc eax

00454110     84C0                   test al,al

00454112     0F85 15FFFFFF          jnz unpackme.0045402D

00454118     3B75 F8               cmp esi,dword ptr ss:[ebp-8]

0045411B     74 0B                  je short unpackme.00454128//改为 JMP ★

0045411D     6A 00                  push 0

0045411F     8B45 FC               mov eax,dword ptr ss:[ebp-4]

00454122     50                     push eax

00454123     E8 2829FBFF            call <jmp.&kernel32.TerminateProcess>//轻轻关掉你  ![icon_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)

00454128     33C0                   xor eax,eax

0045412A     5A                     pop edx

0045412B     59                     pop ecx

0045412C     59                     pop ecx

0045412D     64:8910               mov dword ptr fs:[eax],edx

00454130     68 5A414500            push unpackme.0045415A

00454135     8D85 A8FAFFFF          lea eax,dword ptr ss:[ebp-558]

0045413B     BA 08000000            mov edx,8

00454140     E8 9702FBFF            call unpackme.004043DC

00454145     8D45 F0               lea eax,dword ptr ss:[ebp-10]

00454148     BA 02000000            mov edx,2

0045414D     E8 8A02FBFF            call unpackme.004043DC

00454152     C3                     retn
```

简单跳过反跟踪：

```
0045411B     74 0B                  je short unpackme.00454128

0045411B     EB 0B                  jmp short unpackme.00454128//改 1 个字节就 OK 啦
```

### 二、得到密码

机器码：`B10A517D`

试炼码：`13572－46890－12345`



`unpackme` 露出了要求注册的框子。密码的跟踪就不细说了，通过“S 大法”+“内存断点”就能找到可疑的地方。

```
004552E8     E8 7F4EFDFF          call 0-unpack.0042A16C //下面是运算注册码，得到 3 组字符
004552ED     FFB5 647EFFFF        push dword ptr ss:[ebp+FFFF7E64]
004552F3     68 48564500         push 0-unpack.00455648
004552F8     8D85 607EFFFF       lea eax,dword ptr ss:[ebp+FFFF7E60]
004552FE     50                  push eax
004552FF     B9 05000000         mov ecx,5
00455304     BA 07000000         mov edx,7
00455309     8B45 EC             mov eax,dword ptr ss:[ebp-14]
0045530C     E8 5B4EFDFF         call 0-unpack.0042A16C
00455311     FFB5 607EFFFF       push dword ptr ss:[ebp+FFFF7E60]
00455317     68 48564500         push 0-unpack.00455648
0045531C     8D85 5C7EFFFF       lea eax,dword ptr ss:[ebp+FFFF7E5C]
00455322     50                  push eax
00455323     B9 05000000         mov ecx,5
00455328     BA 14000000         mov edx,14
0045532D     8B45 EC             mov eax,dword ptr ss:[ebp-14]
00455330     E8 374EFDFF         call 0-unpack.0042A16C
00455335     FFB5 5C7EFFFF       push dword ptr ss:[ebp+FFFF7E5C]
0045533B     8D45 EC             lea eax,dword ptr ss:[ebp-14]
0045533E     BA 05000000         mov edx,5
00455343     E8 E8F3FAFF         call 0-unpack.00404730
00455348     8D95 587EFFFF       lea edx,dword ptr ss:[ebp+FFFF7E58]
0045534E     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00455351     8B80 F4020000       mov eax,dword ptr ds:[eax+2F4]
00455357     E8 9CB0FDFF         call 0-unpack.004303F8
0045535C     FFB5 587EFFFF       push dword ptr ss:[ebp+FFFF7E58]
00455362     68 48564500         push 0-unpack.00455648
00455367     8D95 547EFFFF       lea edx,dword ptr ss:[ebp+FFFF7E54]
0045536D     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00455370     8B80 0C030000       mov eax,dword ptr ds:[eax+30C]
00455376     E8 7DB0FDFF         call 0-unpack.004303F8
0045537B     FFB5 547EFFFF       push dword ptr ss:[ebp+FFFF7E54]
00455381     68 48564500         push 0-unpack.00455648
00455386     8D95 507EFFFF       lea edx,dword ptr ss:[ebp+FFFF7E50]
0045538C     8B45 FC             mov eax,dword ptr ss:[ebp-4]
0045538F     8B80 10030000       mov eax,dword ptr ds:[eax+310]
00455395     E8 5EB0FDFF         call 0-unpack.004303F8
0045539A     FFB5 507EFFFF       push dword ptr ss:[ebp+FFFF7E50]
004553A0     8D45 E8             lea eax,dword ptr ss:[ebp-18]
004553A3     BA 05000000         mov edx,5
004553A8     E8 83F3FAFF         call 0-unpack.00404730
004553AD     8B45 E8             mov eax,dword ptr ss:[ebp-18]
                              //EAX=13572-46890-12345    试炼码
004553B0     8B55 EC             mov edx,dword ptr ss:[ebp-14]
                              //EDX=E396D-6D3A8-A49AA     注册码
004553B3     E8 FCF3FAFF         call 0-unpack.004047B4 //真假码比较   ★
004553B8     0F85 4D010000       jnz 0-unpack.0045550B   //跳则 OVER
```

得到注册码：`E396D-6D3A8-A49AA`

保护软件竟然用了明码比较？前面用了再多的密码学运算又有多大作用？

或者直接修改：

```
004553AD     8B45 E8             mov eax,dword ptr ss:[ebp-18]
004553AD     8B45 EC             mov eax,dword ptr ss:[ebp-14] //呵呵，真的和真的去比较吧
```

### 三、得到解密后的原程序

```
004553BE     8B45 E8             mov eax,dword ptr ss:[ebp-18]
004553C1     E8 DAEFFFF          call unpackme.00453EA0
004553C6     33C9                xor ecx,ecx
004553C8     8B95 E47FFFFF       mov edx,dword ptr ss:[ebp+FFFF7FE4]
004553CE     8B45 F8             mov eax,dword ptr ss:[ebp-8]
004553D1     E8 2633FBFF         call unpackme.004086FC
004553D6     8D85 4C7EFFFF       lea eax,dword ptr ss:[ebp+FFFF7E4C]
004553DC     E8 97EAFFFF         call unpackme.00453E78
004553E1     FFB5 4C7EFFFF       push dword ptr ss:[ebp+FFFF7E4C] //系统临时文件夹路径
004553E7     68 54564500         push unpackme.00455654
004553EC     8D85 487EFFFF       lea eax,dword ptr ss:[ebp+FFFF7E48]
004553F2     8D95 947EFFFF       lea edx,dword ptr ss:[ebp+FFFF7E94] //加壳前程序的真实名称 calc.exe
004553F8     E8 17F2FAFF         call unpackme.00404614
004553FD     FFB5 487EFFFF       push dword ptr ss:[ebp+FFFF7E48]
00455403     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00455406     05 1C030000         add eax,31C
0045540B     BA 03000000         mov edx,3
00455410     E8 1BF3FAFF         call unpackme.00404730
00455415     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00455418     8B80 1C030000       mov eax,dword ptr ds:[eax+31C]
0045541E     E8 5532FBFF         call unpackme.00408678 //CreateFileA 产生程序
00455423     8945 F4             mov dword ptr ss:[ebp-C],eax
00455426     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00455429     8B80 1C030000       mov eax,dword ptr ds:[eax+31C]
0045542F     BA 02000000         mov edx,2
00455434     E8 8733FBFF         call unpackme.004087C0
00455439     33D2                xor edx,edx
0045543B     55                  push ebp
0045543C     68 A5544500         push unpackme.004554A5
00455441     64:FF32             push dword ptr fs:[edx]
00455444     64:8922             mov dword ptr fs:[edx],esp
00455447     8D95 E87FFFFF       lea edx,dword ptr ss:[ebp+FFFF7FE8]
0045544D     B9 00800000         mov ecx,8000
00455452     8B45 F8             mov eax,dword ptr ss:[ebp-8]
00455455     E8 4A32FBFF         call unpackme.004086A4
0045545A     8BD8                mov ebx,eax
0045545C     8D95 E87FFFFF       lea edx,dword ptr ss:[ebp+FFFF7FE8]
00455462     8BCB                mov ecx,ebx
00455464     8B45 F4             mov eax,dword ptr ss:[ebp-C]
00455467     E8 6432FBFF         call unpackme.004086D0
0045546C     85DB                test ebx,ebx
0045546E     74 04               je short unpackme.00455474
00455470     3BD8                cmp ebx,eax
00455472     74 D3               je short unpackme.00455447 //解出程序
00455474     33C0                xor eax,eax
00455476     5A                  pop edx
00455477     59                  pop ecx
00455478     59                  pop ecx
00455479     64:8910             mov dword ptr fs:[eax],edx
0045547C     68 AC544500         push unpackme.004554AC
00455481     B9 02000000         mov ecx,2
00455486     BA A4FEFFFF         mov edx,-15C
0045548B     8B45 F4             mov eax,dword ptr ss:[ebp-C]
0045548E     E8 6932FBFF         call unpackme.004086FC
00455493     8B45 F4             mov eax,dword ptr ss:[ebp-C]
00455496     50                  push eax
00455497     E8 7415FBFF         call <jmp.&kernel32.SetEndOfFile>
0045549C     8B45 F4             mov eax,dword ptr ss:[ebp-C]
0045549F     E8 9C32FBFF         call unpackme.00408740
004554A4     C3                  retn
004554AC     8D45 F0             lea eax,dword ptr ss:[ebp-10]
004554AF     8B55 FC             mov edx,dword ptr ss:[ebp-4]
004554B2     8B92 1C030000       mov edx,dword ptr ds:[edx+31C]
004554B8     E8 93EFFAFF         call unpackme.00404450
004554BD     E8 3ED5FAFF         call unpackme.00402A00
004554C2     8BD8                mov ebx,eax
004554C4     85DB                test ebx,ebx
004554C6     7E 31               jle short unpackme.004554F9
004554C8     BE 01000000         mov esi,1
004554CD     FF75 F0             push dword ptr ss:[ebp-10]
004554D0     68 60564500         push unpackme.00455660
004554D5     8D95 447EFFFF       lea edx,dword ptr ss:[ebp+FFFF7E44]
004554DB     8BC6                mov eax,esi
```



```
004554DD     E8 7ED5FAFF          call unpackme.00402A60   

004554E2     FFB5 447EFFFF        push dword ptr ss:[ebp+FFFF7E44]   

004554E8     8D45 F0             lea eax,dword ptr ss:[ebp-10]   

004554EB     BA 03000000         mov edx,3   

004554F0     E8 3BF2FAFF         call unpackme.00404730   

004554F5     46                  inc esi   

004554F6     4B                  dec ebx   

004554F7     75 D4              jnz short unpackme.004554CD   

004554F9     BA 01000000         mov edx,1   

004554FE     8B45 F0             mov eax,dword ptr ss:[ebp-10]   

00455501     E8 02FBFFFF         call unpackme.00455008   

00455506     E9 83000000         jmp unpackme.0045558E   

0045507E     E8 E517FBFF         call <jmp.&kernel32.CreateProcessA>//程序运行    
```

OK，去你的临时文件夹下会找到 1 个隐藏属性的`\_calc.exe`文件，复制出来，呵呵，`OEP`、输入表都不用修复，运行一下，正常。这个就是加壳前的原程序了。跳过反跟踪和密码验证一共只需要修改 2 个字节。

呵呵，这种方式让我想起以前发的`CEXE`脱壳笔记，只是加了检测父进程、把隐藏文件放到临时文件夹下运行，其它的都很相似了。

—————————————————————————————————

```
         ,     _/  
       /| _.-~/         _   ,        青春都一晌
      ( /~   /          ~-.|
      \`\\   _/           ~ )          忍把浮名
  _-~~~-.)  )__//;;,.        _  //'    
 /'\_,    --~    ~~-  ,;;\_\_( (.-~~~-.      换了破解轻狂
 \`~ _( ,_.-- (      ,;''/    ~--   /._`
  /~~//'   /`~          ) /--.._ ) _ \`~
  "  \`~"  "      \`"      /~'\`    \`\\~~  
                    "     "   "~'  ""
     Cracked By 巢水工作坊——fly [OCN][FCG][NUKE][DCM]
              2004-02-23  01:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
