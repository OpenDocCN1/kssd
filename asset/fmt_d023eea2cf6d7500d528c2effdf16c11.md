
# 揭秘迅雷 VIP 用户的破解方法

> 原文：[`www.pediy.com/kssd/pediy10/99624.html`](https://www.pediy.com/kssd/pediy10/99624.html)

**【文章标题】**: 揭秘迅雷 VIP 用户的破解方法  
**【文章作者】**: 我是土匪  
**【作者主页】**: `http://lwzy-crack.blog.163.com`  
**【软件名称】**: 迅雷(Thunder) 5.9.9.1118 正式版  
**【下载地址】**: 自己搜索下载  
**【使用工具】**: `OLLYDBG`  
**【操作平台】**: `XP SP2`  
**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

**【详细过程】**

今晚睡不着觉，发现迅雷又有新版本更新，于是就来自己 DIY 一下迅雷"VIP"，![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

我的迅雷安装到 C 盘了。我就用这个路径了，大家可以自己去找自己的安装路径。

`C:\Program Files\Thunder Network\Thunder\Program\XLUser.dll`

先用 `OD` 载入这个文件，查找字符串“`isvip`”找到后，在它上面的“`register.ini`”语句上面的 `call` 语句下断点

```
  21987044   .  E8 C882FFFF   CALL XLUser.2197F311                       ;  在此处下断点 ，F7 跟入  
  21987049   .  68 B43D9921   PUSH XLUser.21993DB4                       ;  register.ini  
  2198704E   .  8D86 14050000 LEA EAX,DWORD PTR DS:[ESI+514]  
  21987054   .  50             PUSH EAX  
  21987055   .  8D45 9C        LEA EAX,DWORD PTR SS:[EBP-64]  
  21987058   .  50             PUSH EAX  
  21987059   .  E8 CE4EFFFF   CALL XLUser.2197BF2C  
  2198705E   .  83C4 0C        ADD ESP,0C  
  21987061   .  80BE FC050000>CMP BYTE PTR DS:[ESI+5FC],0  
  21987068   .  C645 FC 12     MOV BYTE PTR SS:[EBP-4],12  
  2198706C   .  8D4D 9C        LEA ECX,DWORD PTR SS:[EBP-64]  
  2198706F   .  74 0E          JE SHORT XLUser.2198707F  
  21987071   .  FF15 DC209921 CALL DWORD PTR DS:[<&MSVCP71.std::basic\_string<cha>;  
  msvcp71.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::data  
  21987077   .  50             PUSH EAX  
  21987078   .  68 C8259921   PUSH XLUser.219925C8                       ;  1  
  2198707D   .  EB 0C          JMP SHORT XLUser.2198708B  
  2198707F   >  FF15 DC209921 CALL DWORD PTR DS:[<&MSVCP71.std::basic\_string<cha>;  
  msvcp71.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::data  
  21987085   .  50             PUSH EAX  
  21987086   .  68 C4259921   PUSH XLUser.219925C4                       ;  0  
  2198708B   >  68 003A9921   PUSH XLUser.21993A00                       ; |isvip  
  21987090   .  68 F8399921   PUSH XLUser.219939F8                       ; |login  
  21987095   .  FF15 34209921 CALL DWORD PTR DS:[<&KERNEL32.WritePrivateProfileS>; \WritePrivateProfileStringA  
```

此时就用 `OD` 载入“`Thunder.exe`”，`F9` 运行迅雷。在这里中断后，`F7` 跟入。

```
  21987044   .  E8 C882FFFF   CALL XLUser.2197F311                       ;  在此处下断点 ，F7 跟入  
```

进入 `call` 子程序后，我们找子程序的末尾处，找到后，找最后一个跳转语句，在找跳转语句上面的 `call` 语句，在 `call` 语句上面下断点，然后 `F9`，待到程序中断后，`F7` 跟进去。

```
  2197F311  /$  55             PUSH EBP                                   ;  子程序开始处  
  2197F312  |.  8BEC           MOV EBP,ESP  
  2197F314  |.  51             PUSH ECX  
  2197F315  |.  53             PUSH EBX  
  2197F316  |.  56             PUSH ESI  
  2197F317  |.  8BF1           MOV ESI,ECX  
  2197F319  |.  57             PUSH EDI  
  2197F31A  |.  8D9E 4C100000 LEA EBX,DWORD PTR DS:[ESI+104C]  
  2197F320  |.  53             PUSH EBX                                   ; /pCriticalSection  
  2197F321  |.  895D FC        MOV DWORD PTR SS:[EBP-4],EBX               ; |  
  2197F324  |.  FF15 24209921 CALL DWORD PTR DS:[<&KERNEL32.EnterCriticalSection>]    ; \EnterCriticalSection  
  2197F32A  |.  8B7D 08        MOV EDI,DWORD PTR SS:[EBP+8]  
  2197F32D  |.  39BE 24100000 CMP DWORD PTR DS:[ESI+1024],EDI  
  2197F333  |.  75 06          JNZ SHORT XLUser.2197F33B  
  2197F335  |.  53             PUSH EBX  
  2197F336  |.  E9 ED000000    JMP XLUser.2197F428  
  2197F33B  |>  33DB           XOR EBX,EBX  
  2197F33D  |.  399E 64100000 CMP DWORD PTR DS:[ESI+1064],EBX  
  2197F343  |.  74 2F          JE SHORT XLUser.2197F374  
  2197F345  |.  83FF 02        CMP EDI,2  
  2197F348  |.  75 0F          JNZ SHORT XLUser.2197F359  
  2197F34A  |.  399E A4000000 CMP DWORD PTR DS:[ESI+A4],EBX  
  2197F350  |.  75 07          JNZ SHORT XLUser.2197F359  
  2197F352  |.  8BCE           MOV ECX,ESI  
  2197F354  |.  E8 AED9FFFF    CALL XLUser.2197CD07  
  2197F359  |>  399E 64100000 CMP DWORD PTR DS:[ESI+1064],EBX  
  2197F35F  |.  74 13          JE SHORT XLUser.2197F374  
  2197F361  |.  3BFB           CMP EDI,EBX  
  2197F363  |.  75 0F          JNZ SHORT XLUser.2197F374  
  2197F365  |.  399E A4000000 CMP DWORD PTR DS:[ESI+A4],EBX  
  2197F36B  |.  75 07          JNZ SHORT XLUser.2197F374  
  2197F36D  |.  8BCE           MOV ECX,ESI  
  2197F36F  |.  E8 C8D9FFFF    CALL XLUser.2197CD3C  
  2197F374  |>  3BFB           CMP EDI,EBX  
  2197F376  |.  8B86 24100000 MOV EAX,DWORD PTR DS:[ESI+1024]  
  2197F37C  |.  8986 28100000 MOV DWORD PTR DS:[ESI+1028],EAX  
  2197F382  |.  8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]  
  2197F385  |.  89BE 24100000 MOV DWORD PTR DS:[ESI+1024],EDI  
  2197F38B  |.  8986 2C100000 MOV DWORD PTR DS:[ESI+102C],EAX  
  2197F391  |.  75 69          JNZ SHORT XLUser.2197F3FC  
  2197F393  |.  68 F0229921   PUSH XLUser.219922F0  
  2197F398  |.  8D8E F0060000 LEA ECX,DWORD PTR DS:[ESI+6F0]  
  2197F39E  |.  889E FC050000 MOV BYTE PTR DS:[ESI+5FC],BL  
  2197F3A4  |.  FF15 E8209921 CALL DWORD PTR DS:[<&MSVCP71.std::basic\_string<char,std>;  
  msvcp71.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::operator=  
  2197F3AA  |.  68 484C9921   PUSH XLUser.21994C48                         ;  19000000  
  2197F3AF  |.  8D8E 3C060000 LEA ECX,DWORD PTR DS:[ESI+63C]  
  2197F3B5  |.  899E 34060000 MOV DWORD PTR DS:[ESI+634],EBX  
  2197F3BB  |.  899E 38060000 MOV DWORD PTR DS:[ESI+638],EBX  
  2197F3C1  |.  FF15 E8209921 CALL DWORD PTR DS:[<&MSVCP71.std::basic\_string<char,std>;  
  msvcp71.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::operator=  
  2197F3C7  |.  8D8E D4060000 LEA ECX,DWORD PTR DS:[ESI+6D4]  
  2197F3CD  |.  899E 10060000 MOV DWORD PTR DS:[ESI+610],EBX  
  2197F3D3  |.  899E 14060000 MOV DWORD PTR DS:[ESI+614],EBX  
  2197F3D9  |.  899E 60060000 MOV DWORD PTR DS:[ESI+660],EBX  
  2197F3DF  |.  899E 1C060000 MOV DWORD PTR DS:[ESI+61C],EBX  
  2197F3E5  |.  899E 18060000 MOV DWORD PTR DS:[ESI+618],EBX  
  2197F3EB  |.  899E 20060000 MOV DWORD PTR DS:[ESI+620],EBX  
  2197F3F1  |.  899E D0060000 MOV DWORD PTR DS:[ESI+6D0],EBX  
  2197F3F7  |.  E8 08F2FFFF    CALL XLUser.2197E604  
  2197F3FC  |>  83BE 64100000>CMP DWORD PTR DS:[ESI+1064],2  
  2197F403  |.  75 07          JNZ SHORT XLUser.2197F40C  
  2197F405  |.  8BCE           MOV ECX,ESI  
  2197F407  |.  E8 41EDFFFF    CALL XLUser.2197E14D                         ;  这里是跳转语句上面的第一个 call，我们 F7  
  跟进去。  
  2197F40C  |>  83FF 02        CMP EDI,2  
  2197F40F  |.  75 14          JNZ SHORT XLUser.2197F425                       ;  子程序的最后一个跳转语句  
  2197F411  |.  53             PUSH EBX  
  2197F412  |.  53             PUSH EBX  
  2197F413  |.  8D46 10        LEA EAX,DWORD PTR DS:[ESI+10]  
  2197F416  |.  8B08           MOV ECX,DWORD PTR DS:[EAX]  
  2197F418  |.  6A 0F          PUSH 0F  
  2197F41A  |.  50             PUSH EAX  
  2197F41B  |.  FF51 2C        CALL DWORD PTR DS:[ECX+2C]  
  2197F41E  |.  8BCE           MOV ECX,ESI  
  2197F420  |.  E8 65EAFFFF    CALL XLUser.2197DE8A  
  2197F425  |>  FF75 FC        PUSH DWORD PTR SS:[EBP-4]                       ; /pCriticalSection  
  2197F428  |>  FF15 28209921 CALL DWORD PTR DS:[<&KERNEL32.LeaveCriticalSection>]    ; \LeaveCriticalSection  
  2197F42E  |.  5F             POP EDI  
  2197F42F  |.  5E             POP ESI  
  2197F430  |.  5B             POP EBX  
  2197F431  |.  C9             LEAVE  
  2197F432  \.  C2 0800        RETN 8                                     ;  子程序的结尾处  
```

`2197F407` `F7` 跟进去后，来到下面
