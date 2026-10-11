
# 终截者入侵阻止 v5.26

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-514.htm`](https://www.pediy.com/kssd/pediy08/pediy8-514.htm)

## 基本信息

*   **目标**: 终截者入侵阻止 v5.26
*   **工具**: `W32Dasm` 中文版(GOLD)、`OllyICE` v1.10 修改版、`EXESCOPE` v6.30、`LordPE` v1.4，`PEiD` 0.94
*   **操作平台**: `WindowsXP` SP1
*   **作者**: `edisonH`
*   **软件说明**：《终截者入侵阻止》 -- 100%拦截已知与未知病毒，可以有效拦截最新变种病毒及别有用心之人制作的“免杀病毒”；保护你所有的帐号与密码,是你的网上银行交易、游戏、聊天的保护专家！
*   **声明**: 本文为原创作品，转载请注明出处并保持文章的完整性, 谢谢!。

## 缘起

看中终截者的驱动级注册表 hook，找来一个最新版，安装后发现需要未注册版本只有 30 天试用期，且有以下不满之处，于是决定破解了并改造不满之处。

## 破解试用 30 天

软件安装后，主程序启动正常，调节时间到 30 天后，使试用期过期，开启程序，出现过期对话框。

### 破解开始

按例，用 `PEiD` 先查壳，无壳。

1.  使用 `eXeScope` 打开主程序 `SecMain.exe`，查找过期对话框，发现对话框 ID 为 `0x95`
2.  使用 `W32Dasm` 打开主程序 `SecMain.exe`，查找对话框 ID 为 `0x95` 的，在地址 `0x405740` 处发现

```
 0040573D  .|  8BF1          mov     esi, ecx
 0040573F  .|  50          push    eax
 00405740  .|  68 95000000 push    95
 00405745  .|  897424 14   mov     [esp+14], esi
 00405749  .|  FF15 34144100 call  [<&SkinWinLib.CSecBaseDlg::CSecBaseDlg>]  ;  对话框创建
```

3.  `OllyICE` 载入主程序 `SecMain.exe`，在地址 `0x405740` 处下断，运行，中断在 `0x405740` 处，回朔，发现过期检查处

```
00407D10  /$  64:A1 0000000> mov     eax, fs:[0]
00407D16  |.  6A FF         push    -1
00407D18  |.  68 EB064100   push    004106EB
00407D1D  |.  50          push    eax
00407D1E  |.  A1 5C7E4100   mov     eax, [417E5C]   ; 逝去的时间，只有软件刚打开时为 0
00407D23  |.  64:8925 00000> mov     fs:[0], esp
00407D2A  |.  81EC 10030000 sub     esp, 310
00407D30  |.  85C0         test    eax, eax    ; 是否刚打开
00407D32  |.  56          push    esi
00407D33  |.  8B35 44104100 mov     esi, [<&KERNEL32.GetTickCount>]     ;  kernel32.GetTickCount
00407D39  |.  57          push    edi
00407D3A     74 1D       je      short 00407D59   ; 刚打开 则跳去 过期检查
00407D3C  |.  FFD6       call    esi                   ; [GetTickCount
00407D3E  |.  2B05 5C7E4100 sub     eax, [417E5C]   ; 上次检查到现在的时间差
00407D44  |.  8BF8       mov     edi, eax
00407D46  |.  FFD6       call    esi                   ; [GetTickCount
00407D48  |.  81FF 00974901 cmp     edi, 1499700   ; 比较时间差是否大于 6 小时
00407D4E  |.  A3 5C7E4100 mov     [417E5C], eax   ; 保存这次检查的时间
00407D53     0F82 5F010000 jb      00407EB8    ; 若距上次检查在 6 小时以内 则跳过 过期检查
00407D59  |>  FFD6       call    esi                   ; [GetTickCount
00407D5B  |.  A3 5C7E4100 mov     [417E5C], eax
00407D60  |.  E8 49510000 call    <JMP.&MFC42.#1168_AFXGETMODULESTATE>
00407D65  |.  8B40 04     mov     eax, [eax+4]
00407D68  |.  8B88 F0020000 mov     ecx, [eax+2F0]   ; 从安装到现在的时间差
00407D6E  |.  83F9 1E     cmp     ecx, 1E    ; 比较是否大于 30 天
00407D71  |.  7D 10       jge     short 00407D83   ; 大于等于 30 天则跳
00407D73  |.  B8 1E000000 mov     eax, 1E
00407D78  |.  2BC1       sub     eax, ecx
00407D7A  |.  83F8 05     cmp     eax, 5    ; 比较距过期是否大于 5 天
00407D7D  |.  0F8D 35010000 jge     00407EB8    ; 大于等于 5 天则跳
00407D83  |>  8B35 907E4100 mov     esi, [417E90]
......
00407DA9  |.  E8 48730000 call    <JMP.&MAINDLG.GETMAINDLG>
00407DAE  |.  50          push    eax
00407DAF  |.  8D4C24 14   lea     ecx, [esp+14]
00407DB3  |.  E8 68D9FFFF call    00405720        ; 回朔到这里,往上看，发现过期检查处
{
 00405720  /$  6A FF         push    -1
 00405722  |.  68 D0FF4000   push    0040FFD0          ;  SE 处理程序安装
 00405727  |.  64:A1 0000000> mov     eax, fs:[0]
 ...
 0040573D  |.  8BF1        mov     esi, ecx
 0040573F  |.  50          push    eax
 00405740  |.  68 95000000 push    95        ; 过期对话框 ID0x95，这里下断，回朔回上一级调用
 00405745  |.  897424 14   mov     [esp+14], esi
 00405749  |.  FF15 34144100 call    [<&SkinWinLib.CSecBaseDlg::CSecBaseDlg>]  ;  创建对话框
 ...
 004057F9  |.  64:890D 00000> mov     fs:[0], ecx
 00405800  |.  83C4 10     add     esp, 10
 00405803  \.  C2 0400     ret     4
}
00407DB8  |.  C68424 240300> mov     byte ptr [esp+324], 1
...
00407EC1  |.  64:890D 00000> mov     fs:[0], ecx
00407EC8  |.  81C4 1C030000 add     esp, 31C
00407ECE  \.  C3          ret
```

4. 可以看到，跳往 `00407EB8` 即可解除过期限制，这里有两处:

```
00407D53     0F82 5F010000 jb      00407EB8
```

和

```
00407D7D  |.  0F8D 35010000 jge     00407EB8
```

所以这里的爆破方案有两种：

1.  
```
00407D3A     74 1D       je      short 00407D59 ; 改为 nop
00407D53     0F82 5F010000 jb      00407EB8  ; 改为 jmp 00407EB8
```

2.  
```
00407D71  |.  7D 10       jge     short 00407D83 ; 改为 nop
00407D7D  |.  0F8D 35010000 jge     00407EB8  ; 改为 jmp 00407EB8
```

另外，从上面的分析中可以看到，试用并没有真正的 30 天，而是 25 天。

## DIY 改造

### 不满之处

终截者有主程序 `SecMain.exe` 及服务程序 `SecBkSrv.exe`

1.  每次主程序 `SecMain.exe` 启动后，启动服务进程 `SecBkSrv.exe`，并设置服务 `SecBkSrv` 为自动启动，下次系统重启时服务自动启动
2.  当关闭主程序后，服务没有关闭，继续占用资源
3.  服务进程 `SecBkSrv.exe` 启动后，在注册表 `Run` 项中添加 主程序 `SecMain.exe` 自启动项，下次重启后主程序自启动

### 计划

1.  服务改为始终为手动方式，不会随重启自启动
2.  主程序退出时，关闭服务，节省资源
3.  主程序改为非强制自启动

### 详细步骤

全部完成后，将修改 `SecMain.exe` 和 `SecBkSrv.exe`，事先备份。

1.  服务改为始终为手动方式

    `OllyICE` 载入 `SecMain.exe`，下断服务参数设置 api 函数 `ChangeServiceConfigA`，回朔

    ```
    00409265  |.  6A 03       push    2    ; |StartType 参数 (2 为自动,3 为手动,改为 push 3）
    00409267  |.  68 A06A4100 push    00416AA0                ;  ASCII "SecBkSrv"
    0040926C  |.  E8 4F010000 call    004093C0   ; 回朔到这里
    {
     004093C0  /$  55          push    ebp
     004093C1  |.  57          push    edi
     ...
     004093EB  |.  53          push    ebx
     004093EC  |.  56          push    esi
     ...
     00409406  |.  8B4C24 18   mov     ecx, [esp+18]  ; |StartType 启动类型，[esp+18]即为 00409265 处压入的参数 push    3
     0040940A  |.  6A 00       push    0               ; /DisplayName = NULL
     0040940C  |.  6A 00       push    0               ; |Password = NULL
     0040940E  |.  6A 00       push    0               ; |ServiceStartName = NULL
     00409410  |.  6A 00       push    0               ; |pDependencies = NULL
     00409412  |.  6A 00       push    0               ; |pTagId = NULL
     00409414  |.  6A 00       push    0               ; |LoadOrderGroup = NULL
     00409416  |.  6A 00       push    0               ; |BinaryPathName = NULL
     00409418  |.  6A FF       push    -1              ; |ErrorControl = SERVICE_NO_CHANGE
     0040941A  |.  51          push    ecx             ; |StartType
     0040941B  |.  68 10010000 push    110             ; |ServiceType = SERVICE_WIN32_OWN_PROCESS|SERVICE_INTERACTIVE_PROCESS
     00409420  |.  56          push    esi             ; |hService
     00409421  |.  FF15 2C104100 call    [<&ADVAPI32.ChangeServiceConfigA>; 下断，回朔
     ...
     0040949E  \.  C3          ret
    }
    ```

2.  改造原程序，使其拥有关闭服务功能

    先来看看，主程序怎么启动服务的

    ```
    00408F00  /$  6A FF         push    -1
    00408F02  |.  68 C8074100   push    004107C8   ;  SE 处理程序安装
    ...
    00408F18  |.  68 3F000F00   push    0F003F
    00408F1D  |.  6A 00       push    0
    00408F1F  |.  6A 00       push    0
    00408F21  |.  FF15 1C104100 call  [<&ADVAPI32.OpenSCManagerA>];  第一步，打开服务控制管理器
    00408F27  |.  8BD8       mov     ebx, eax   ;  得到服务控制管理器的句柄 hSCManager，存于 ebx 中
    ...
    00408F3E  |>  8B4424 1C   mov     eax, [esp+1C]
    00408F42  |.  6A 30       push    10
    00408F44  |.  50          push    eax
    00408F45  |.  53          push    ebx
    00408F46  |.  FF15 18104100 call    [<&ADVAPI32.OpenServiceA>] ;  第二步，打开自己的服务 SecBkSrv
    00408F4C  |.  8BF0       mov     esi, eax   ;  得到句柄 hSecBkSrv，存于 esi 中
    ...
    00408F6E  |.  6A 00       push    0
    00408F70  |.  6A 00       push    0
    00408F72  |.  56          push    esi    ;  hSecBkSrv
    00408F73  |.  FF15 00104100 call  [<&ADVAPI32.StartServiceA>] ;  第三步，启动服务进程 SecBkSrv.exe
    ```



......

```
00408FF0  \.  C3            retn
```

再来看看，如果要关闭服务，要怎么做

```
call OpenSCManagerA  打开服务控制管理器
call OpenServiceA  打开服务
call ControlService 传递 SERVICE_STOPED 参数来关闭服务
```

看来，关闭服务和打开服务的第一第二步是一样的，为了减少增加的代码，改造一下上面主程序的打开服务的代码，增加关闭的功能。

(1) 先要修改打开服务时，设置的权限，改服务启动权限为 `SERVICE_START | SERVICE_STOP`，保证有结束权限

```
00408F42  |.  6A 10         push    10    ;SERVICE_START=0X10,SERVICE_STOP=0X20,改为 push 30
00408F44  |.  50            push    eax
00408F45  |.  53            push    ebx
00408F46  |.  FF15 18104100 call    [<&ADVAPI32.OpenServiceA>]
```

(2) 用 `LordPE.EXE` 为程序添加了 `ADVAPI32.ControlService` 输入函数

(3) 先看看 `ControlService` 函数

```
BOOL ControlService(
    SC_HANDLE hService,   // handle to service
    DWORD dwControl,   // control code
    LPSERVICE_STATUS lpServiceStatus  // pointer to service status structure
);
```

`SERVICE_STATUS` 结构

```
typedef struct _SERVICE_STATUS { // 按注释赋值即可
    DWORD dwServiceType;  // 0
    DWORD dwCurrentState;  // 1 ,SERVICE_STOPED
    DWORD dwControlsAccepted;   // 0
    DWORD dwWin32ExitCode;   // -1
    DWORD dwServiceSpecificExitCode;  // 0
    DWORD dwCheckPoint;   // 0
    DWORD dwWaitHint;    // 0
} SERVICE_STATUS, *LPSERVICE_STATUS;
```

(4) 准备的差不多了，找个空的地方，打补丁增加关闭服务

补丁代码：

`0x408f6e:`

```
jmp 410e15
;修改为我的关闭服务函数
```

`0x410e15:`

```
mov eax,[esp+28]
cmp eax,12121212   ; 检查是否执行关闭服务的标记
je 410e2B
push 0
push 0
push esi
jmp 408f73        ;  不是关闭服务，跳回原函数
pushad
push 0            ;  在堆栈中压入 SERVICE_STATUS 结构
push 0
push 0
push -1
push 0
push 1
push 0
push esp          ;  &SERVICE_STATUS
push 1            ;  SERVICE_STOPED
push esi          ;  hSecBkSrv 服务句柄
call [42001e]  ;  ADVAPI32.ControlService
add esp,1c
popad
pop edi
pop ebp
pop esi
pop ebx
mov ecx,[esp+4]
mov fs:[0],ecx
add esp,10
ret
```

(5) 那么在哪里调用关闭服务的函数呢？

在主程序退出前，这里。

```
0040D46A  |.  8945 98       mov     [local.26], eax
0040D46D  |.  50            push    eax                     ; /status
0040D46E      FF15 A4134100 call    [<&MSVCRT.exit>]         ;  MSVCRT.exit
```

`0x40d46e:`

```
jmp 410e5b
nop
```

`0x410e5b:`

```
;调用关闭服务
pushad
mov eax,12121212  ;  自定义调用的标记
push eax
push 416aa0    ;  "SecBkSrv"
call 408F00      ;  增加了关闭服务的函数入口
pop eax
pop eax
popad
call   [4113A4]     ; MSVCRT.exit
jmp 40d474
```

### 3. 改服务程序 `SecBkSrv.exe`，使主程序改为非强制自启动

简单起见，只要改了 `SecBkSrv.exe` 中的字符串 `"SecMain.exe Hide"` 为一个目录中不存在的文件即可，这里我改为 `no.exe`

用 `W32Dasm` 在 `SecBkSrv.exe` 查找 `"SecMain.exe Hide"`，这里

```
00404776  |.  BF 74AB4000   mov     edi, 0040AB74                ;  "SecMain.exe Hide"
0040AB74  53 65 63 4D 61 69 6E 2E 65 78 65 20 48 69 64 65  SecMain.exe Hide
```

改为

```
0040AB74  6E 6F 2E 65 78 65 00 00 00 00 00 00 00 00 00 00  no.exe..........
```

即可

修改好后，保存 `SecMain.exe` 和 `SecBkSrv.exe`。

运行主程序 `SecMain.exe`，退出

一切按照要求工作了，成功！

`4dd466d0e1e0cc123afbe504da2462ca`
