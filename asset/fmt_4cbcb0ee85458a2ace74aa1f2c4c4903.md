
# 易通文件夹锁 2006 3.0.2.928

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-361.htm`](https://www.pediy.com/kssd/pediy08/pediy8-361.htm)

【文章作者】: `tzl`

【作者邮箱】: `tigerisme@126.com`

【软件名称】: 易通文件夹锁 2006 3.0.2.928

【软件大小】: 3099KB

【下载地址】: `http://www.newhua.com/soft/43125.htm`

【加壳方式】: `aspack`

【保护方式】: 注册码

【编写语言】: `Borland Delphi 6.0 - 7.0`

【使用工具】: `OD`, `PEID`

【操作平台】: `XP SP2`

【软件介绍】: 四大保护功能，三种保护方式。集文件夹锁、文件锁、磁盘锁、增强锁于一体。提供锁定、隐藏、移动三种保护方式。防删除,复制等。是您文件资料的安全屏障。

## 分析过程

脱壳略去，调试主程序 `CDirLock.exe`，这个程序是调用了 `ETDirLock.dll` 进行注册加密，我们下 `bp MessageBoxA` 命令，运行并填写试练码后断下，返回反汇编窗口，在这里下断进行分析。注册名：`tigerisme` 试练码：`11111-22222-33333-44444`

```
0057C290    55              push ebp
0057C291    8BEC            mov ebp,esp
0057C293    B9 0B000000     mov ecx,0B
0057C298    6A 00           push 0
0057C29A    6A 00           push 0
0057C29C    49              dec ecx
0057C29D  ^ 75 F9           jnz short ETDirLoc.0057C298
0057C29F    53              push ebx
0057C2A0    8BD8            mov ebx,eax
0057C2A2    33C0            xor eax,eax
0057C2A4    55              push ebp
0057C2A5    68 06C55700     push ETDirLoc.0057C506
0057C2AA    64:FF30         push dword ptr fs:[eax]
0057C2AD    64:8920         mov dword ptr fs:[eax],esp
0057C2B0    8D55 F4         lea edx,dword ptr ss:[ebp-C]
0057C2B3    8B83 20030000   mov eax,dword ptr ds:[ebx+320]
0057C2B9    E8 6A6BFDFF     call ETDirLoc.00552E28
0057C2BE    8B45 F4         mov eax,dword ptr ss:[ebp-C]                ; 注册名=ebp-C 送 eax
0057C2C1    8D55 F8         lea edx,dword ptr ss:[ebp-8]
0057C2C4    E8 53CCF8FF     call ETDirLoc.00508F1C
0057C2C9    837D F8 00      cmp dword ptr ss:[ebp-8],0                  ; 注册名与 0 比较,不为 0 则正常跳转
0057C2CD    0F84 80000000   je ETDirLoc.0057C353
0057C2D3    8D55 EC         lea edx,dword ptr ss:[ebp-14]
0057C2D6    8B83 24030000   mov eax,dword ptr ds:[ebx+324]
0057C2DC    E8 476BFDFF     call ETDirLoc.00552E28
0057C2E1    8B45 EC         mov eax,dword ptr ss:[ebp-14]               ; 第一组“11111”送 eax
0057C2E4    8D55 F0         lea edx,dword ptr ss:[ebp-10]
0057C2E7    E8 30CCF8FF     call ETDirLoc.00508F1C
0057C2EC    837D F0 00      cmp dword ptr ss:[ebp-10],0                 ; 第一组“11111”与 0 比较,不为 0 则正常跳转
0057C2F0    0F85 9A000000   jnz ETDirLoc.0057C390
0057C2F6    8D55 E4         lea edx,dword ptr ss:[ebp-1C]
0057C2F9    8B83 2C030000   mov eax,dword ptr ds:[ebx+32C]
0057C2FF    E8 246BFDFF     call ETDirLoc.00552E28
0057C304    8B45 E4         mov eax,dword ptr ss:[ebp-1C]
0057C307    8D55 E8         lea edx,dword ptr ss:[ebp-18]
0057C30A    E8 0DCCF8FF     call ETDirLoc.00508F1C
0057C30F    837D E8 00      cmp dword ptr ss:[ebp-18],0
0057C313    75 7B           jnz short ETDirLoc.0057C390
0057C315    8D55 DC         lea edx,dword ptr ss:[ebp-24]
0057C318    8B83 30030000   mov eax,dword ptr ds:[ebx+330]
0057C31E    E8 056BFDFF     call ETDirLoc.00552E28
0057C323    8B45 DC         mov eax,dword ptr ss:[ebp-24]
0057C326    8D55 E0         lea edx,dword ptr ss:[ebp-20]
0057C329    E8 EECBF8FF     call ETDirLoc.00508F1C
0057C32E    837D E0 00      cmp dword ptr ss:[ebp-20],0
0057C332    75 5C           jnz short ETDirLoc.0057C390
0057C334    8D55 D4         lea edx,dword ptr ss:[ebp-2C]
0057C337    8B83 34030000   mov eax,dword ptr ds:[ebx+334]
0057C33D    E8 E66AFDFF     call ETDirLoc.00552E28
0057C342    8B45 D4         mov eax,dword ptr ss:[ebp-2C]
0057C345    8D55 D8         lea edx,dword ptr ss:[ebp-28]
0057C348    E8 CFCBF8FF     call ETDirLoc.00508F1C
0057C34D    837D D8 00      cmp dword ptr ss:[ebp-28],0
0057C351    75 3D           jnz short ETDirLoc.0057C390
0057C353    68 30000400     push 40030
0057C358    8D55 D0         lea edx,dword ptr ss:[ebp-30]
0057C35B    B8 20C55700     mov eax,ETDirLoc.0057C520
0057C360    E8 E793FFFF     call ETDirLoc.0057574C
0057C365    8B45 D0         mov eax,dword ptr ss:[ebp-30]
0057C368    E8 B78AF8FF     call ETDirLoc.00504E24
0057C36D    50              push eax
0057C36E    8D55 CC         lea edx,dword ptr ss:[ebp-34]
0057C371    B8 38C55700     mov eax,ETDirLoc.0057C538
0057C376    E8 D193FFFF     call ETDirLoc.0057574C
0057C37B    8B45 CC         mov eax,dword ptr ss:[ebp-34]
0057C37E    E8 A18AF8FF     call ETDirLoc.00504E24
0057C383    50              push eax
0057C384    6A 00           push 0
0057C386    E8 B9B5F8FF     call ETDirLoc.00507944              ; jmp 到 user32.MessageBoxA
0057C38B    E9 EC000000     jmp ETDirLoc.0057C47C
0057C390    8D55 C8         lea edx,dword ptr ss:[ebp-38]
0057C393    8B83 24030000   mov eax,dword ptr ds:[ebx+324]
0057C399    E8 8A6AFDFF     call ETDirLoc.00552E28
0057C39E    FF75 C8         push dword ptr ss:[ebp-38]          ; ebp-38=11111
0057C3A1    8D55 C4         lea edx,dword ptr ss:[ebp-3C]
0057C3A4    8B83 2C030000   mov eax,dword ptr ds:[ebx+32C]      ; eax=5
0057C3AA    E8 796AFDFF     call ETDirLoc.00552E28
0057C3AF    FF75 C4         push dword ptr ss:[ebp-3C]          ; ebp-3C=22222
0057C3B2    8D55 C0         lea edx,dword ptr ss:[ebp-40]
0057C3B5    8B83 30030000   mov eax,dword ptr ds:[ebx+330]      ; eax=5
0057C3BB    E8 686AFDFF     call ETDirLoc.00552E28
0057C3C0    FF75 C0         push dword ptr ss:[ebp-40]          ; ebp-40=33333
0057C3C3    8D55 BC         lea edx,dword ptr ss:[ebp-44]
0057C3C6    8B83 34030000   mov eax,dword ptr ds:[ebx+334]      ; eax=5
0057C3CC    E8 576AFDFF     call ETDirLoc.00552E28
0057C3D1    FF75 BC         push dword ptr ss:[ebp-44]          ; ebp-44=44444
0057C3D4    8D45 FC         lea eax,dword ptr ss:[ebp-4]
0057C3D7    BA 04000000     mov edx,4                           ; 4 送 edx
0057C3DC    E8 0B89F8FF     call ETDirLoc.00504CEC
0057C3E1    8D55 B8         lea edx,dword ptr ss:[ebp-48]
0057C3E4    8B83 20030000   mov eax,dword ptr ds:[ebx+320]
0057C3EA    E8 396AFDFF     call ETDirLoc.00552E28
0057C3EF    8B55 B8         mov edx,dword ptr ss:[ebp-48]       ; 注册名送 edx
0057C3F2    8B45 FC         mov eax,dword ptr ss:[ebp-4]        ; 试练码送 eax
0057C3F5    E8 5A97FFFF     call ETDirLoc.00575B54              ; 算法 call(1)
0057C3FA    84C0            test al,al
0057C3FC    74 46           je short ETDirLoc.0057C444          标志位判断，爆破点**********
0057C3FE    68 40000400     push 40040
0057C403    8D55 B4         lea edx,dword ptr ss:[ebp-4C]
0057C406    B8 20C55700     mov eax,ETDirLoc.0057C520
0057C40B    E8 3C93FFFF     call ETDirLoc.0057574C
0057C410    8B45 B4         mov eax,dword ptr ss:[ebp-4C]
0057C413    E8 0C8AF8FF     call ETDirLoc.00504E24
0057C418    50              push eax
0057C419    8D55 B0         lea edx,dword ptr ss:[ebp-50]
0057C41C    B8 68C55700     mov eax,ETDirLoc.0057C568
0057C421    E8 2693FFFF     call ETDirLoc.0057574C
0057C426    8B45 B0         mov eax,dword ptr ss:[ebp-50]
0057C429    E8 F689F8FF     call ETDirLoc.00504E24
0057C42E    50              push eax
0057C42F    6A 00           push 0
0057C431    E8 0EB5F8FF     call ETDirLoc.00507944              ; 注册成功提示************
0057C436    A1 2CD15900     mov eax,dword ptr ds:[59D12C]
0057C43B    8B00            mov eax,dword ptr ds:[eax]
0057C43D    E8 AE88FCFF     call ETDirLoc.00544CF0
0057C442    EB 38           jmp short ETDirLoc.0057C47C
********************************************************************************************

算法 call(1)

00575B6A    55              push ebp
00575B6B    68 AA5C5700     push ETDirLoc.00575CAA
```
