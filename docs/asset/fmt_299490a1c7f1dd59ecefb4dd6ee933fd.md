
### 2197E14D `XLUser` 模块分析

```
  2197E14D   /$  B8 66FF9821    MOV EAX,XLUser.2198FF66  

  2197E152   |.  E8 79050100    CALL XLUser.2198E6D0  

  2197E157   |.  51             PUSH ECX  

  2197E158   |.  53             PUSH EBX  

  2197E159   |.  56             PUSH ESI  

  2197E15A   |.  8BF1           MOV ESI,ECX  

  2197E15C   |.  57             PUSH EDI  

  2197E15D   |.  8D86 98100000 LEA EAX,DWORD PTR DS:[ESI+1098]  

  2197E163   |.  50             PUSH EAX                                       ; /pCriticalSection  

  2197E164   |.  FF15 24209921 CALL DWORD PTR DS:[<&KERNEL32.EnterCriticalSection>]    ; \EnterCriticalSection  

  2197E16A   |.  8D86 7C100000 LEA EAX,DWORD PTR DS:[ESI+107C]  

  2197E170   |.  8945 F0        MOV DWORD PTR SS:[EBP-10],EAX  

  2197E173   |.  8D9E 6C100000 LEA EBX,DWORD PTR DS:[ESI+106C]  

  2197E179   |.  33FF           XOR EDI,EDI  

  2197E17B   |.  8BCB           MOV ECX,EBX  

  2197E17D   |.  897D FC        MOV DWORD PTR SS:[EBP-4],EDI  

  2197E180   |.  E8 49D8FFFF    CALL XLUser.2197B9CE  

  2197E185   |.  85C0           TEST EAX,EAX  

  2197E187   |.  76 2F          JBE SHORT XLUser.2197E1B8  

  2197E189   |>  FFB6 2C100000 /PUSH DWORD PTR DS:[ESI+102C]  

  2197E18F   |.  8B86 70100000 |MOV EAX,DWORD PTR DS:[ESI+1070]  

  2197E195   |.  FFB6 28100000 |PUSH DWORD PTR DS:[ESI+1028]  

  2197E19B   |.  8D04B8         |LEA EAX,DWORD PTR DS:[EAX+EDI*4]  

  2197E19E   |.  8B00           |MOV EAX,DWORD PTR DS:[EAX]  

  2197E1A0   |.  FFB6 24100000 |PUSH DWORD PTR DS:[ESI+1024]  

  2197E1A6   |.  8B08           |MOV ECX,DWORD PTR DS:[EAX]  

  2197E1A8   |.  50             |PUSH EAX  

  2197E1A9   |.  FF51 0C        |CALL DWORD PTR DS:[ECX+C]                     ;  F7 跟进去  

  2197E1AC   |.  8BCB           |MOV ECX,EBX  

  2197E1AE   |.  47             |INC EDI  

  2197E1AF   |.  E8 1AD8FFFF    |CALL XLUser.2197B9CE  

  2197E1B4   |.  3BF8           |CMP EDI,EAX  

  2197E1B6   |.^ 72 D1          \JB SHORT XLUser.2197E189  

  2197E1B8   |>  8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]  

  2197E1BB   |.  83C0 1C        ADD EAX,1C  

  2197E1BE   |.  50             PUSH EAX                                       ; /pCriticalSection  

  2197E1BF   |.  FF15 28209921 CALL DWORD PTR DS:[<&KERNEL32.LeaveCriticalSection>]    ; \LeaveCriticalSection  

  2197E1C5   |.  8B4D F4        MOV ECX,DWORD PTR SS:[EBP-C]  

  2197E1C8   |.  5F             POP EDI  

  2197E1C9   |.  5E             POP ESI  

  2197E1CA   |.  5B             POP EBX  

  2197E1CB   |.  64:890D 00000>MOV DWORD PTR FS:[0],ECX  

  2197E1D2   |.  C9             LEAVE  

  2197E1D3   \\.  C3             RETN  
```

`"2197E1A9"` `F7` 跟进去后，来到下面，进入 `BaseCommunity.dll` 领空

```
  1000B9D0    .  6A FF         PUSH -1  

  1000B9D2    .  68 F7030510   PUSH BaseComm.100503F7                       ;  SE 处理程序安装  

  1000B9D7    .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]  

  1000B9DD    .  50             PUSH EAX  

  1000B9DE    .  64:8925 00000>MOV DWORD PTR FS:[0],ESP  

  1000B9E5    .  81EC 9C000000 SUB ESP,9C  

  1000B9EB    .  A1 C42D0810   MOV EAX,DWORD PTR DS:[10082DC4]  

  1000B9F0    .  33C4          XOR EAX,ESP  

  1000B9F2    .  55             PUSH EBP  

  1000B9F3    .  8BAC24 B80000>MOV EBP,DWORD PTR SS:[ESP+B8]  

  1000B9FA    .  57             PUSH EDI  

  1000B9FB    .  8BBC24 B80000>MOV EDI,DWORD PTR SS:[ESP+B8]  

  1000BA02    .  3BFD           CMP EDI,EBP  

  1000BA04    .  898424 A00000>MOV DWORD PTR SS:[ESP+A0],EAX  

  1000BA0B    .  0F84 1A030000 JE BaseComm.1000BD2B  

  1000BA11    .  83FF 02        CMP EDI,2  

  1000BA14    .  53             PUSH EBX  

  1000BA15    .  56             PUSH ESI  

  1000BA16    .  0F85 6C020000 JNZ BaseComm.1000BC88  

  1000BA1C    .  E8 AFA9FFFF    CALL BaseComm.100063D0  

  1000BA21    .  8BC8           MOV ECX,EAX  

  1000BA23    .  E8 9865FFFF    CALL BaseComm.10001FC0                       ;  F7 跟进去  

  1000BA28    .  85C0           TEST EAX,EAX  

  1000BA2A    .  8B35 54710610 MOV ESI,DWORD PTR DS:[<&KERNEL32.GetTickCount>]         ;  kernel32.GetTickCount  
```

------------------------------------------

`"1000BA23"` `F7` 跟进去后，来到下面

```
  10001FC0   /$  51             PUSH ECX                                       ;  (Initial CPU selection)  

  10001FC1   |.  8B41 1C        MOV EAX,DWORD PTR DS:[ECX+1C]  

  10001FC4   |.  85C0           TEST EAX,EAX  

  10001FC6   |.  74 17          JE SHORT BaseComm.10001FDF  

  10001FC8   |.  8D1424         LEA EDX,DWORD PTR SS:[ESP]  

  10001FCB   |.  52             PUSH EDX  

  10001FCC   |.  C74424 04 000>MOV DWORD PTR SS:[ESP+4],0  

  10001FD4   |.  8B08           MOV ECX,DWORD PTR DS:[EAX]  

  10001FD6   |.  50             PUSH EAX  

  10001FD7   |.  FF51 0C        CALL DWORD PTR DS:[ECX+C]                     ;  F7 跟进去  

  10001FDA   |.  8B0424         MOV EAX,DWORD PTR SS:[ESP]  

  10001FDD   |.  59             POP ECX  

  10001FDE   |.  C3             RETN  

  10001FDF   |>  33C0           XOR EAX,EAX  

  10001FE1   |.  59             POP ECX  

  10001FE2   \\.  C3             RETN  
```

------------------------------------------

`"10001FD7"` `F7` 跟进去后，来到下面，进入 `XLUser.dll` 领空

```
  2197A95B    .  8B4424 04      MOV EAX,DWORD PTR SS:[ESP+4]  

  2197A95F    .  83B8 10100000>CMP DWORD PTR DS:[EAX+1010],2  

  2197A966    .  74 07          JE SHORT XLUser.2197A96F  

  2197A968    .  B8 04400080   MOV EAX,80004004  

  2197A96D    .  EB 0F          JMP SHORT XLUser.2197A97E  

  2197A96F    >   0FB680 E80500>MOVZX EAX,BYTE PTR DS:[EAX+5E8]             ;  修改为 mov eax,1  

  2197A976    .  8B4C24 08      MOV ECX,DWORD PTR SS:[ESP+8]  

  2197A97A    .  8901           MOV DWORD PTR DS:[ECX],EAX  

  2197A97C    .  33C0          XOR EAX,EAX  

  2197A97E    >   C2 0800       RETN 8  
```

在 `2197A96F` 打补丁，

`MOVZX EAX,BYTE PTR DS:[EAX+5E8]` 修改为 `mov eax,1`

------------------------------------------

修改后，保存，即可，这样就是 VIP 用户了。

呵呵，赶快替换原有的`XLUser.dll`文件吧，体验一下不花一分钱的 VIP 用户的功能。
![扮鬼脸](img/f43c1e514327696dccbb0da997496b2a.jpg)

--------------------------------------------------------------------------------

### 经验总结

知识是一点一点的积累的，我们要勤于动手。本文的破解方法对迅雷通用（迅雷公司不针对此文章做相关修改），希望下次在更新的时候，你能自己破解 VIP![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

--------------------------------------------------------------------------------

### 版权声明

文章发自看雪论坛，转载请注明作者并保持文章的完整, 谢谢!

2009 年 10 月 18 日 4:23:37

本代码的着色效果由 `xTiNt` 自动完成

下载 `xTiNt`

测试过了，除了会员皮肤和去广告功能其他的会员功能都不能用 其他的是要迅雷服务器验证的吧
