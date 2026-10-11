
# 天天个人助理(DailyPim)注册算法分析+汇编注册机

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-449.htm`](https://www.pediy.com/kssd/pediy07/pediy7-449.htm)

【破文标题】 天天个人助理(DailyPim)注册算法分析+汇编注册机  
【破文作者】 snake  
【软件名称】 天天个人助理(DailyPim) V3.42  
【下载地址】 `http://www.skycn.com/soft/15570.html`  
【软件简介】 `DailyPim` 是一款个人日常信息管理的软件，具有的功能有日记本、资料管理、文件管理、日程管理、地址簿、网页快抓、收发消息、收发文件、邮箱监视器、查询天气、火车、航班、电话区号、邮政编码、定时关机等。`DailyPim` 是国内功能最强大的个人信息管理软件。  
【调试环境】 `Win2000`、`Ollydbg`  
【作者声明】 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

\----------------------------------------------------------------------------------------------

【破解过程】

### 一、程序脱壳

用 `PEiD`、`FI` 查壳，均为 `Nothing found`，只好手动脱壳了。使用两次 `ESP` 定律即可到达程序 `OEP`。

`Ollydbg` 载入主程序

```
00C70082  d>  60               pushad
00C70083   E8 00000000        call dailypim.00C70088
00C70088   5D               pop ebp
00C70089   81ED 22A54500    sub ebp,dailypim.0045A522
00C7008F   8DBD 1CA54500    lea edi,dword ptr ss:[ebp+45A51C]
00C70095   81EF 82000000    sub edi,82
00C7009B   89BD 84A84500    mov dword ptr ss:[ebp+45A884],edi
00C700A1   8B4F 18          mov ecx,dword ptr ds:[edi+18]
00C700A4   89FE             mov esi,edi
00C700A6   0377 14          add esi,dword ptr ds:[edi+14]
00C700A9   8B47 10          mov eax,dword ptr ds:[edi+10]
......
```

`F8` 一次，在命令栏下命令 `hr esp`，回车，`F9` 运行

```
00C70374  - FFE0             jmp eax                         ; 断在这里
00C70376   42               inc edx
00C70377   4B               dec ebx
......
```

`F9` 再运行一次

```
00C693B0   /75 08           jnz short dailypim.00C693BA       ; 断在这里
00C693B2   |B8 01000000     mov eax,1
00C693B7   |C2 0C00         retn 0C
00C693BA   \68 D85C8200     push dailypim.00825CD8
00C693BF    C3              retn          ; F8 到这里，飞向光明之颠！^\_^
00C693C0    8B85 26040000   mov eax,dword ptr ss:[ebp+426]
00C693C6    8D8D 3B040000   lea ecx,dword ptr ss:[ebp+43B]
00C693CC    51              push ecx
00C693CD    50              push eax
00C693CE    FF95 490F0000   call dword ptr ss:[ebp+F49]
......
```

```
00825CD8    55              push ebp         ; 程序 OEP 处用 LordPE 完全 Dump 这个进程
00825CD9    8BEC            mov ebp,esp
00825CDB    83C4 F0         add esp,-10
00825CDE    53              push ebx
00825CDF    B8 484E8200     mov eax,dailypim.00824E48
00825CE4    E8 AB19BEFF     call dailypim.00407694
00825CE9    8B1D D83F8300   mov ebx,dword ptr ds:[833FD8]        ; dailypim.00835C3C
00825CEF    8B03            mov eax,dword ptr ds:[ebx]
00825CF1    E8 FA57C5FF    call dailypim.0047B4F0
00825CF6    8B03            mov eax,dword ptr ds:[ebx]
......
```

脱壳后程序可正常运行。

\---------------------------------------------------------------------------------------------------

### 二、算法分析

运行程序，输入相关信息

- 机器码：`G732526459`
- 注册名：`snake`
- 注册码：`7878787878`

注册后提示：谢谢您的注册，下次启动生效！

说明程序在启动时要读取注册表或相应文件的内容，经分析查找，发现同目录下 `PIM.ini` 文件里有注册信息。

`Ollydbg` 载入脱壳后的程序

右键，搜所-->所有参考文本串，查找 `PIM.ini` 并 `F2` 下断，`F9` 运行程序

```
0080D9BB   |.  BA BCE48000    mov edx,x.0080E4BC                  ;  ASCII "PIM.ini"  断的此处
0080D9C0   |.  E8 6376BFFF    call x.00405028
0080D9C5   |.  8B4D E0        mov ecx,[local.8]
0080D9C8   |.  B2 01          mov dl,1
0080D9CA   |.  A1 D4024800    mov eax,dword ptr ds:[4802D4]
0080D9CF   |.  E8 B029C7FF    call x.00480384
0080D9D4   |.  8BD8           mov ebx,eax
0080D9D6   |.  6A 00          push 0
0080D9D8   |.  8D45 D8        lea eax,[local.10]
0080D9DB   |.  50             push eax
0080D9DC   |.  B9 CCE48000    mov ecx,x.0080E4CC                  ;  ASCII "SKINFILE"
0080D9E1   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
0080D9E6   |.  8BC3           mov eax,ebx
0080D9E8   |.  8B38           mov edi,dword ptr ds:[eax]
0080D9EA   |.  FF17           call dword ptr ds:[edi]
0080D9EC   |.  8B55 D8        mov edx,[local.10]
0080D9EF   |.  A1 E8408300    mov eax,dword ptr ds:[8340E8]
0080D9F4   |.  E8 B373BFFF    call x.00404DAC
0080D9F9   |.  68 ECE48000    push x.0080E4EC
0080D9FE   |.  8D45 D4        lea eax,[local.11]
0080DA01   |.  50             push eax
0080DA02   |.  B9 FCE48000    mov ecx,x.0080E4FC                  ;  ASCII "WEATHER"
0080DA07   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
0080DA0C   |.  8BC3           mov eax,ebx
0080DA0E   |.  8B38           mov edi,dword ptr ds:[eax]
0080DA10   |.  FF17           call dword ptr ds:[edi]
0080DA12   |.  8B55 D4        mov edx,[local.11]
0080DA15   |.  A1 D4378300    mov eax,dword ptr ds:[8337D4]
0080DA1A   |.  E8 8D73BFFF    call x.00404DAC
0080DA1F   |.  6A 00          push 0
0080DA21   |.  8D45 D0        lea eax,[local.12]
0080DA24   |.  50             push eax
0080DA25   |.  B9 0CE58000    mov ecx,x.0080E50C                  ;  ASCII "REGUSER"
0080DA2A   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
0080DA2F   |.  8BC3           mov eax,ebx
0080DA31   |.  8B38           mov edi,dword ptr ds:[eax]
0080DA33   |.  FF17           call dword ptr ds:[edi]
0080DA35   |.  8B55 D0        mov edx,[local.12]                  ;  取注册名 snake
0080DA38   |.  A1 B83B8300    mov eax,dword ptr ds:[833BB8]
0080DA3D   |.  E8 6A73BFFF    call x.00404DAC
0080DA42   |.  6A 00          push 0
0080DA44   |.  8D45 FC        lea eax,[local.1]
0080DA47   |.  50             push eax
0080DA48   |.  8B0D 70438300  mov ecx,dword ptr ds:[834370]        ;  x.00838930
0080DA4E   |.  8B09           mov ecx,dword ptr ds:[ecx]           ;  取机器码 G732526459
0080DA50   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
0080DA55   |.  8BC3           mov eax,ebx
0080DA57   |.  8B38           mov edi,dword ptr ds:[eax]
0080DA59   |.  FF17           call dword ptr ds:[edi]              ;  取假注册码 7878787878
0080DA5B   |.  837D FC 00     cmp [local.1],0                      ;  是否为空
0080DA5F   |.  75 16          jnz short x.0080DA77
0080DA61   |.  6A 00          push 0
0080DA63   |.  8D45 FC        lea eax,[local.1]
0080DA66   |.  50             push eax
0080DA67   |.  B9 1CE58000    mov ecx,x.0080E51C                  ;  ASCII "REGSN"
0080DA6C   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
0080DA71   |.  8BC3           mov eax,ebx
0080DA73   |.  8B38           mov edi,dword ptr ds:[eax]
0080DA75   |.  FF17           call dword ptr ds:[edi]
0080DA77   |>   A1 EC3D8300    mov eax,dword ptr ds:[833DEC]
0080DA7C   |.  8B55 FC        mov edx,[local.1]
0080DA7F   |.  E8 2873BFFF    call x.00404DAC                      ;  转存假码 7878787878
0080DA84   |.  6A 01          push 1
0080DA86   |.  B9 2CE58000    mov ecx,x.0080E52C                  ;  ASCII "SKIN"
0080DA8B   |.  BA E0E48000    mov edx,x.0080E4E0                  ;  ASCII "PIM"
......（省略部分）
```

```
0080E4AC     .  5F              pop edi
0080E4AD     .  5E              pop esi
0080E4AE     .  5B              pop ebx
0080E4AF     .  8BE5            mov esp,ebp
0080E4B1     .  5D              pop ebp
0080E4B2     .  C3              retn                     ; 返回程序
```

\--------------------------------------------------------------------------------------------------

```
007AF2A2     .  8B03            mov eax,dword ptr ds:[ebx]
007AF2A4     .  E8 D7E60500     call x.0080D980
007AF2A9     .  8B03            mov eax,dword ptr ds:[ebx]           ;  返回此处
007AF2AB     .  E8 00060600     call x.0080F8B0                     ;  F7 跟进
007AF2B0     .  A1 D83F8300     mov eax,dword ptr ds:[833FD8]
......
```
