
# 楚汉棋缘 v1.42

> 原文：[`www.pediy.com/kssd/pediy06/pediy6298.htm`](https://www.pediy.com/kssd/pediy06/pediy6298.htm)

软件名称: `楚汉棋缘`   

最新版本: `v1.42`   

适用平台: `Win9x`, `WinME`, `Win2000`, `WinXP`   

作者主页: `http://www.xqpartner.com`   

软件简介:   

主要功能：中国象棋人机对战！网络对战！同机对战！棋谱研究！   

人机对战：很强的智力！有入门、初、中、高级和自由级选择，可进行段位赛，人机对战时可随时交换红黑方!   

同机对战：双人同机交替走棋，是研究棋谱的好地方，人机对战和同机对战可随时交换！   

网络对战：可以在局域网和 `Internet` 上双人对战，具有旁观、聊天等功能，是和棋友切磋的好工具！   

棋谱研究：还有系统自带的近 2000 多个精典象棋棋谱，包括：《橘中秘》、《梅花泉》，《现代布局精要》、《适情雅趣》和《烂柯神机》等 13 本秘籍；并且实现真正的"自己走试试"，对棋谱当中你不理解的地方，可随时和计算机演练，其定会对你棋艺的提高大有裨益！   

【软件限制】：功能限制   

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！   

【破解工具】：`olldydbg`、`IDA Pro`、`PeId 0.9`、`Aspack Die1.41`   

===============================================================================   

【过    程】：   

为了练习 `RSA`，找到了这个软件练习一下。

## 1、壳与自校验：   

用 `peid` 探测得壳为 `aspack2.12`，用 `aspack die1.41` 很容易脱去，好像是 `BCB` 程序。   

脱壳后注册对话框中输入注册码的编辑框隐藏了，显然有自校验存在。对付自校验一般就下断点 `CreateFileA`   

顺利的找到了这里

在 `FormCreate` 中

```
.text:00416276                 call    unknown_libname_677 ; CreateFileA
.text:0041627B                 mov     edi, eax
.text:0041627D                 dec     dword ptr [esi+1Ch]
.text:00416280                 lea     eax, [ebp-20h]
.text:00416283                 mov     edx, 2
.text:00416288                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)
.text:0041628D                 mov     ecx, 2
.text:00416292                 xor     edx, edx
.text:00416294                 mov     eax, edi        ; 将文件指针移到文件尾，返回值即为文件大小
.text:00416296                 call    @Sysutils@FileSeek$qqriii ; Sysutils::FileSeek(int,int,int)
.text:0041629B                 mov     [ebp-70h], eax  ; [ebp-70h]保存文件大小
.text:004162F6                 mov     dword_50CD78, eax
.text:004162FB                 xor     ecx, ecx
.text:004162FD                 mov     edx, 43Bh
.text:00416302                 mov     eax, edi        ; 移到文件可执行文件首偏移 43Bh 处
.text:00416304                 call    @Sysutils@FileSeek$qqriii ; Sysutils::FileSeek(int,int,int)
.text:00416309                 mov     edx, [ebx+334h]
.text:0041630F                 mov     eax, [edx+128h]
.text:00416315                 call    unknown_libname_223
.text:0041631A                 mov     edx, eax
.text:0041631C                 mov     eax, dword_50CD78
.text:00416321                 mov     ecx, [eax]
.text:00416323                 call    dword ptr [ecx+8]
.text:00416326                 lea     edx, [ebp-71h]
.text:00416329                 mov     ecx, 1
.text:0041632E                 mov     eax, edi
.text:00416330                 call    sub_4B9B14      ; 保存在[ebp-71h]处
.text:00416330                                   ; 读取文件偏移 43B 位置的一字节
... 往下走了几步，到该字节校验处
.text:00416390                 cmp     byte ptr [ebp-71h], 32h      //读取文件 1 字节校验
.text:00416394                 jz      short loc_4163CD
... 不放心，再来一次
.text:0041655D                 cmp     byte ptr [ebp-71h], 32h
.text:00416561                 jz      short loc_41656B
... //还没完，继续
.text:00416575                 xor     ecx, ecx
.text:00416577                 mov     edx, 600h       ; 文件指针移向文件偏移 600h 处
.text:0041657C                 mov     eax, edi
.text:0041657E                 call    @Sysutils@FileSeek$qqriii ; Sysutils::FileSeek(int,int,int)
.text:00416583                 lea     edx, [ebp-71h]
.text:00416586                 mov     ecx, 1
.text:0041658B                 mov     eax, edi
.text:0041658D                 call    sub_4B9B14      ; 文件偏移 600h 处读 1 字节
.text:00416592                 mov     eax, edi
.text:00416594                 call    sub_4B9B78
.text:00416599                 cmp     byte ptr [ebp-71h], 20h ; //读取文件 1 字节校验
.text:0041659D                 jz      short loc_4165B4
...
.text:00416689                 cmp     dword ptr [ebp-70h], 0A1220h     //文件大小校验，不能太大
.text:00416690                 jnb     short loc_4166A2
.text:00416692                 cmp     dword ptr [ebp-70h], 9C400h      //文件大小校验，也不能太小
，不知道脱壳后有没有文件体积变小的
.text:00416699                 jbe     short loc_4166A2
.text:004166A2                 push    0Ah
.text:004166A4                 push    0
.text:004166A6                 push    0
.text:004166A8                 lea     ecx, [ebp-7Ch]
.text:004166AB                 push    0
.text:004166AD                 push    ecx
.text:004166AE                 push    0Ch
.text:004166B0                 push    0
.text:004166B2                 push    offset unk_4E0367
.text:004166B7                 call    GetVolumeInformationA   //调用这个干吗，好像是把序列号也保存在文件里了，虽然注册码输入时不要，但 chess.dat 拷贝到别的机器上也许也不能用？没试过。
.text:004166BC                 mov     eax, [ebx+2164h]
.text:004166C2                 cmp     eax, [ebp-7Ch]
.text:004166C5                 jz      short loc_416736
```

然后通过查找 `call @Sysutils@FileSeek$qqriii` 的 cross reference 或立即数 `0A1220h` 可以找到其余还有一处的文件大小校验之处

```
.text:0041D2B3                 call    @Sysutils@FileSeek$qqriii ; Sysutils::FileSeek(int,int,int)
.text:0041D2B8                 mov     esi, eax
.text:0041D2BA                 mov     [ebp+var_2C], 8
.text:0041D2C0                 mov     eax, ebx        ; hObject
.text:0041D2C2                 call    sub_4B9B78
.text:0041D2C7                 cmp     esi, 0A1220h        // here
.text:0041D2CD                 jle     short loc_41D32F
```

========================================================================

## 2. 用户名及注册码变换：   

注册按钮按下响应事件

`_TAboutBox_SpeedButton1Click`

```
.text:0043298C                 push    ebp
.text:0043298D                 mov     ebp, esp
.text:0043298F                 add     esp, 0FFFFFEC8h
.text:00432995                 push    ebx
.text:00432996                 push    esi
.text:00432997                 push    edi
.text:00432998                 mov     [ebp+var_AC], edx
.text:0043299E                 mov     [ebp+var_A8], eax
.text:004329A4                 xor     eax, eax
.text:004329A6                 mov     [ebp+var_9C], offset unk_4FFDA4
.text:004329B0                 mov     [ebp+var_98], esp
.text:004329B6                 mov     [ebp+var_A0], offset __ExceptionHandler
.text:004329C0                 mov     [ebp+var_94], 0
.text:004329C9                 mov     [ebp+var_88], eax
.text:004329CF                 mov     edx, large fs:0
.text:004329D6                 mov     [ebp+var_A4], edx
.text:004329DC                 lea     ecx, [ebp+var_A4]
.text:004329E2                 mov     large fs:0, ecx
.text:004329E9                 mov     [ebp+var_94], 14h
.text:004329F2                 mov     edx, offset a1234567890abcd ; "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabc"...
.text:004329F7                 lea     eax, [ebp+var_4]
.text:004329FA                 call    sub_4D3198
.text:004329FF                 inc     [ebp+var_88]
.text:00432A05                 xor     ecx, ecx
.text:00432A07                 mov     [ebp+var_94], 8
.text:00432A10                 mov     [ebp+var_94], 20h
.text:00432A19                 mov     [ebp+var_8], ecx
.text:00432A1C                 xor     eax, eax
.text:00432A1E                 inc     [ebp+var_88]
.text:00432A24                 xor     edx, edx
.text:00432A26                 mov     [ebp+var_94], 8
.text:00432A2F                 mov     [ebp+var_94], 2Ch
.text:00432A38                 mov     [ebp+var_C], eax
.text:00432A3B                 xor     ecx, ecx
.text:00432A3D                 inc     [ebp+var_88]
```



```asm
.text:00432A43                     mov     [ebp+var_94], 8   

.text:00432A4C                     mov     [ebp+var_94], 38h   

.text:00432A55                     mov     [ebp+var_10], edx   

.text:00432A58                     inc     [ebp+var_88]   

.text:00432A5E                     mov     [ebp+var_94], 8   

.text:00432A67                     mov     [ebp+var_94], 44h   

.text:00432A70                     mov     [ebp+var_14], ecx   

.text:00432A73                     inc     [ebp+var_88]   

.text:00432A79                     xor     edi, edi   

.text:00432A7B                     mov     [ebp+var_94], 8   

.text:00432A84                     push    14h             ; n   

.text:00432A86                     push    0               ; c   

.text:00432A88                     push    offset unk_53B440 ; s   

.text:00432A8D                     call    _memset   

.text:00432A92                     mov     [ebp+var_94], 50h   

.text:00432A9B                     xor     eax, eax   

.text:00432A9D                     add     esp, 0Ch   

.text:00432AA0                     mov     [ebp+var_2C], eax   

.text:00432AA3                     lea     edx, [ebp+var_2C]   

.text:00432AA6                     inc     [ebp+var_88]   

.text:00432AAC                     mov     ecx, [ebp+var_A8]   

.text:00432AB2                     mov     eax, [ecx+308h] ; 获取用户名   

.text:00432AB8                     call    @TControl@GetText$qqrv ; TControl::GetText(void)   

.text:00432ABD                     cmp     [ebp+var_2C], 0   

.text:00432AC1                     jz      short loc_432AC8   

.text:00432AC3                     mov     edx, [ebp+var_2C]   

.text:00432AC6                     jmp     short loc_432ACD   

.text:00432AC8 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

.text:00432AC8   

.text:00432AC8 loc_432AC8:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+135j   

.text:00432AC8                     mov     edx, offset unk_4FF7D7   

.text:00432ACD   

.text:00432ACD loc_432ACD:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+13Aj   

.text:00432ACD                     push    edi   

.text:00432ACE                     xor     eax, eax   

.text:00432AD0                     mov     edi, edx   

.text:00432AD2                     or      ecx, 0FFFFFFFFh   

.text:00432AD5                     repne scasb   

.text:00432AD7                     not     ecx   

.text:00432AD9                     sub     edi, ecx   

.text:00432ADB                     mov     esi, offset unk_53B440   

.text:00432AE0                     xchg    esi, edi   

.text:00432AE2                     mov     edx, ecx   

.text:00432AE4                     mov     eax, edi   

.text:00432AE6                     shr     ecx, 2   

.text:00432AE9                     lea     eax, [ebp+var_2C]   

.text:00432AEC                     repe  movsd   

.text:00432AEE                     mov     ecx, edx   

.text:00432AF0                     mov     edx, 2   

.text:00432AF5                     and     ecx, 3   

.text:00432AF8                     repe  movsb   

.text:00432AFA                     pop     edi   

.text:00432AFB                     dec     [ebp+var_88]   

.text:00432B01                     call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432B06                     xor     esi, esi   

.text:00432B08                     mov     ebx, offset unk_53B440   

.text:00432B0D   

.text:00432B0D loc_432B0D:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+199j   

.text:00432B0D                     xor     eax, eax   

.text:00432B0F                     lea     ecx, [esi+1]   

.text:00432B12                     mov     al, [ebx]   

.text:00432B14                     inc     ebx   

.text:00432B15                     mov     edx, eax   

.text:00432B17                     inc     esi   

.text:00432B18                     imul    edx, eax   

.text:00432B1B                     add     edx, eax   

.text:00432B1D                     imul    edx, ecx   

.text:00432B20                     add     edi, edx   

.text:00432B22                     cmp     esi, 14h        ; 只取用户名前 20 位，不足用 0 补足   

.text:00432B25                     jl      short loc_432B0D ; 相当于   

.text:00432B25                               ; for(i=0; i<sizeof(szName); i++)   

.text:00432B25                               ; {   

.text:00432B25                               ;     dwEndPoint += (szName[i] * szName[i] +   

szName[i])*(i+1);   

.text:00432B25                               ; }   

.text:00432B27                     lea     eax, [edi+5BA0h] ; 加一固定值   

.text:00432B27                               ; dwEndPoint  +＝ 0x5BA0   

.text:00432B2D                     xor     edx, edx   

.text:00432B2F                     mov     [ebp+var_B4], eax   

.text:00432B35                     mov     [ebp+var_B0], edx   

.text:00432B3B                     cmp     [ebp+var_B0], 0   

.text:00432B42                     jnz     short loc_432B5E   

.text:00432B44                     cmp     [ebp+var_B4], 5C25Ch   

.text:00432B4E                     jnz     short loc_432B5E   

.text:00432B50                     add     [ebp+var_B4], 78h   

.text:00432B57                     adc     [ebp+var_B0], 0   

.text:00432B5E   

.text:00432B5E loc_432B5E:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+1B6j   

.text:00432B5E                               ; _TAboutBox_SpeedButton1Click+1C2j   

.text:00432B5E                     push    [ebp+var_B0]   

.text:00432B64                     push    [ebp+var_B4]   

.text:00432B6A                     push    offset aLd_0    ; "%ld"   

.text:00432B6F                     lea     eax, [ebp+var_10]   

.text:00432B72                     push    eax   

.text:00432B73                     call    @System@WideString@printf$qpxbe ; System::WideString::printf   

(wchar_t *,...)   

.text:00432B78                     add     esp, 10h   

.text:00432B7B                     push    14h             ; n   

.text:00432B7D                     push    0               ; c   

.text:00432B7F                     push    offset buffer   ; s   

.text:00432B84                     call    _memset   

.text:00432B89                     add     esp, 0Ch   

.text:00432B8C                     push    64h             ; n   

.text:00432B8E                     push    0               ; c   

.text:00432B90                     push    offset unk_53B3DC ; s   

.text:00432B95                     call    _memset   

.text:00432B9A                     add     esp, 0Ch   

.text:00432B9D                     cmp     [ebp+var_10], 0   

.text:00432BA1                     jz      short loc_432BA8   

.text:00432BA3                     mov     edx, [ebp+var_10]   

.text:00432BA6                     jmp     short loc_432BAD   

.text:00432BA8 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

.text:00432BA8   

.text:00432BA8 loc_432BA8:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+215j   

.text:00432BA8                     mov     edx, offset unk_4FF7DC   

.text:00432BAD   

.text:00432BAD loc_432BAD:                      ; CODE XREF:   

_TAboutBox_SpeedButton1Click+21Aj   

.text:00432BAD                     xor     eax, eax   

.text:00432BAF                     mov     edi, edx   

.text:00432BB1                     or      ecx, 0FFFFFFFFh   

.text:00432BB4                     mov     esi, offset buffer   

.text:00432BB9                     repne scasb   

.text:00432BBB                     not     ecx   

.text:00432BBD                     sub     edi, ecx   

.text:00432BBF                     mov     edx, ecx   

.text:00432BC1                     xchg    esi, edi   

.text:00432BC3                     shr     ecx, 2   

.text:00432BC6                     mov     eax, edi   

.text:00432BC8                     repe  movsd   

.text:00432BCA                     mov     ecx, edx   

.text:00432BCC                     lea     edx, [ebp+var_30]   

.text:00432BCF                     and     ecx, 3   

.text:00432BD2                     repe  movsb
```



```assembly
.text:00432BD4             mov     [ebp+var_94], 5Ch   

.text:00432BDD             xor     eax, eax   

.text:00432BDF             mov     [ebp+var_30], eax   

.text:00432BE2             inc     [ebp+var_88]   

.text:00432BE8             mov     ecx, [ebp+var_A8]   

.text:00432BEE             mov     eax, [ecx+30Ch] ; 获取试炼注册码   

.text:00432BF4             call    @TControl@GetText$qqrv ; TControl::GetText(void)   

.text:00432BF9             lea     edx, [ebp+var_30]   

.text:00432BFC             lea     eax, [ebp+var_8]   

.text:00432BFF             call    @System@AnsiString@$basg$qqrrx17System@AnsiString ;   

System::AnsiString::operator=(System::AnsiString &)   

.text:00432C04             dec     [ebp+var_88]   

.text:00432C0A             lea     eax, [ebp+var_30]   

.text:00432C0D             mov     edx, 2   

.text:00432C12             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432C17             lea     edi, [ebp+var_124]   

.text:00432C1D             mov     esi, offset unk_4FF6FC   

.text:00432C22             mov     ecx, 14h   

.text:00432C27             mov     edx, offset unk_4FF7DD   

.text:00432C2C             repe    movsd   

.text:00432C2E             mov     [ebp+var_94], 8   

.text:00432C37             mov     [ebp+var_94], 68h   

.text:00432C40             lea     eax, [ebp+var_34]   

.text:00432C43             call    sub_4D3198   

.text:00432C48             inc     [ebp+var_88]   

.text:00432C4E             mov     eax, [eax]   

.text:00432C50             xor     edx, edx   

.text:00432C52             mov     [ebp+var_38], edx   

.text:00432C55             lea     edx, [ebp+var_38]   

.text:00432C58             inc     [ebp+var_88]   

.text:00432C5E             call    sub_424E8C   

.text:00432C63             lea     eax, [ebp+var_38]   

.text:00432C66             mov     eax, [eax]     ; 显示提示重启程序对话框   

.text:00432C68             call    @Dialogs@ShowMessage$qqrx17System@AnsiString ;   

Dialogs::ShowMessage(System::AnsiString)   

.text:00432C6D             dec     [ebp+var_88]   

.text:00432C73             lea     eax, [ebp+var_38]   

.text:00432C76             mov     edx, 2   

.text:00432C7B             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432C80             dec     [ebp+var_88]   

.text:00432C86             lea     eax, [ebp+var_34]   

.text:00432C89             mov     edx, 2   

.text:00432C8E             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432C93             mov     [ebp+var_94], 74h   

.text:00432C9C             mov     edx, offset unk_4FF802   

.text:00432CA1             lea     eax, [ebp+var_18]   

.text:00432CA4             call    sub_4D3198   

.text:00432CA9             inc     [ebp+var_88]   

.text:00432CAF             mov     edx, offset unk_4FF803   

.text:00432CB4             mov     [ebp+var_94], 8   

.text:00432CBD             mov     [ebp+var_94], 80h   

.text:00432CC6             lea     eax, [ebp+var_3C]   

.text:00432CC9             call    sub_4D3198   

.text:00432CCE             inc     [ebp+var_88]   

.text:00432CD4             lea     edx, [ebp+var_3C] ; "-"   

.text:00432CD7             lea     eax, [ebp+var_8] ; 用户输入注册码串   

.text:00432CDA             call    @System@AnsiString@Pos$xqqrrx17System@AnsiString ;   

System::AnsiString::Pos(System::AnsiString &)   

.text:00432CDF             mov     [ebp+var_C0], eax ; 注册码中第一个"-"位置   

.text:00432CE5             dec     [ebp+var_88]   

.text:00432CEB             lea     eax, [ebp+var_3C]   

.text:00432CEE             mov     edx, 2   

.text:00432CF3             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432CF8             mov     [ebp+var_94], 8   

.text:00432D01             mov     [ebp+var_94], 8Ch   

.text:00432D0A             xor     ecx, ecx   

.text:00432D0C             lea     eax, [ebp+var_40]   

.text:00432D0F             mov     [ebp+var_40], ecx   

.text:00432D12             push    eax   

.text:00432D13             inc     [ebp+var_88]   

.text:00432D19             lea     eax, [ebp+var_8]   

.text:00432D1C             mov     ecx, [ebp+var_C0]   

.text:00432D22             mov     edx, 1   

.text:00432D27             dec     ecx           ; 取第一个"-"前注册码部分   

.text:00432D28             call    @System@AnsiString@SubString$xqqrii ;   

System::AnsiString::SubString(int,int)   

.text:00432D2D             lea     edx, [ebp+var_40]   

.text:00432D30             lea     eax, [ebp+var_18]   

.text:00432D33             call    @System@AnsiString@$basg$qqrrx17System@AnsiString ;   

System::AnsiString::operator=(System::AnsiString &)   

.text:00432D38             dec     [ebp+var_88]   

.text:00432D3E             lea     eax, [ebp+var_40]   

.text:00432D41             mov     edx, 2   

.text:00432D46             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432D4B             lea     eax, [ebp+var_8]   

.text:00432D4E             mov     ecx, [ebp+var_C0]   

.text:00432D54             mov     edx, 1         ; 总注册码串中删除 第一个"-"注册码部分   

.text:00432D59             call    @System@AnsiString@Delete$qqrii ; System::AnsiString::Delete   

(int,int)   

.text:00432D5E             mov     esi, 1   

.text:00432D63             lea     ebx, [ebp+var_124]   

.text:00432D69             cmp     esi, [ebp+var_C0]   

.text:00432D6F             jge     short loc_432DD9   

.text:00432D71   

.text:00432D71 loc_432D71:                     ; CODE XREF:   

_TAboutBox_SpeedButton1Click+44Bj   

.text:00432D71             mov     [ebp+var_94], 98h   

.text:00432D7A             xor     eax, eax   

.text:00432D7C             lea     edx, [ebp+var_44]   

.text:00432D7F             mov     [ebp+var_44], eax   

.text:00432D82             push    edx   

.text:00432D83             inc     [ebp+var_88]   

.text:00432D89             mov     edx, esi   

.text:00432D8B             lea     eax, [ebp+var_18]   

.text:00432D8E             mov     ecx, 1         ; 第一个"-"前注册码串部分：   

.text:00432D8E                               ; 取 1 字节   

.text:00432D93             call    @System@AnsiString@SubString$xqqrii ;   

System::AnsiString::SubString(int,int)   

.text:00432D98             lea     edx, [ebp+var_44]   

.text:00432D9B             lea     eax, [ebp+var_10]   

.text:00432D9E             call    @System@AnsiString@$basg$qqrrx17System@AnsiString ;   

System::AnsiString::operator=(System::AnsiString &)   

.text:00432DA3             dec     [ebp+var_88]   

.text:00432DA9             lea     eax, [ebp+var_44]   

.text:00432DAC             mov     edx, 2   

.text:00432DB1             call    @System@AnsiString@$bdtr$qqrv ;   

System::AnsiString::~AnsiString(void)   

.text:00432DB6             lea     edx, [ebp+var_10]   

.text:00432DB9             lea     eax, [ebp+var_4] ; 该字节在   

表'1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz'中的位置   

.text:00432DB9                               ; 表长 60   

.text:00432DBC             call    @System@AnsiString@Pos$xqqrrx17System@AnsiString ;   

System::AnsiString::Pos(System::AnsiString &)
```



## 注册码第一部分变换（汇编分析）

```asm
.text:00432DC1             mov     [ebp+var_94], 8
.text:00432DCA             dec     eax
.text:00432DCB             mov     [ebx], eax       ; 查表所得值-1，即从 0 开始计
.text:00432DCD             inc     esi
.text:00432DCE             add     ebx, 4
.text:00432DD1             cmp     esi, [ebp+var_C0]
.text:00432DD7             jl      short loc_432D71 ; 循环处理第一部分注册码所有字节
.text:00432DD9
.text:00432DD9 loc_432DD9:                          ; CODE XREF:
_TAboutBox_SpeedButton1Click+3E3j
.text:00432DD9             xor     edi, edi
.text:00432DDB             mov     esi, [ebp+var_C0]
.text:00432DE1             add     esi, 0FFFFFFFEh
.text:00432DE4             lea     ebx, [ebp+esi*4+var_124]
.text:00432DEB             test    esi, esi
.text:00432DED             jl      short loc_432E4D ; 此变换第一部分部分注册码所得值
.text:00432DEF
.text:00432DEF loc_432DEF:                          ; CODE XREF:
_TAboutBox_SpeedButton1Click+4BFj
.text:00432DEF             mov     [ebp+var_128], esi
.text:00432DF5             fild    [ebp+var_128]    ; 各位所在位，从 0 开始计算
.text:00432DFB             add     esp, 0FFFFFFF8h  ; y
.text:00432DFE             fstp    [esp+14Ch+var_14C]
.text:00432E01             push    404D8000h        ; 双精度数 59
.text:00432E06             push    0                ; x
.text:00432E08             call    _pow
.text:00432E0D             mov     eax, [ebx]
.text:00432E0F             xor     edx, edx
.text:00432E11             mov     dword ptr [ebp+var_130], eax
.text:00432E17             mov     dword ptr [ebp+var_130+4], edx
.text:00432E1D             fild    [ebp+var_130]
.text:00432E23             add     esp, 10h
.text:00432E26             xor     ecx, ecx
.text:00432E28             mov     dword ptr [ebp+var_138], edi
.text:00432E2E             mov     dword ptr [ebp+var_138+4], ecx
.text:00432E34             fmulp   st(1), st
.text:00432E36             fild    [ebp+var_138]
.text:00432E3C             faddp   st(1), st         ; 此次值=(59^所在位数)*Y + 上次值
.text:00432E3C                              ; Y 为注册码字节查表所得值，从 0 开始计
.text:00432E3C                              ; 处理顺序从右至左
.text:00432E3C                              ; 如
.text:00432E3C                              ; "cbqX7"
.text:00432E3C                              ; 从左取一位'7' 查表得到 6，再-1 得 5
.text:00432E3C                              ; 此次值=(59⁴ ) *6
.text:00432E3C                              ; 再取一位'X' 查表得到 34，再-1 得 33
.text:00432E3C                              ; 此次值=(59³ ) *33
.text:00432E3C                              ; ...
.text:00432E3C                              ;
.text:00432E3E             call    @_ftol$qv         ; _ftol(void)
.text:00432E43             mov     edi, eax          ; 所得值化为长整数
.text:00432E45             dec     esi
.text:00432E46             add     ebx, 0FFFFFFFCh
.text:00432E49             test    esi, esi
.text:00432E4B             jge     short loc_432DEF
.text:00432E4D
.text:00432E4D loc_432E4D:                          ; CODE XREF:
_TAboutBox_SpeedButton1Click+461j
.text:00432E4D             push    edi              ; 此变换第一部分部分注册码所得值
.text:00432E4E             push    offset unk_4FF805 ; "%ld"
.text:00432E53             lea     edx, [ebp+var_10] ; 长整数 ----> 字符串
.text:00432E56             push    edx
.text:00432E57             call    @System@WideString@printf$qpxbe ; System::WideString::printf
(wchar_t *,...)
.text:00432E5C             add     esp, 0Ch
.text:00432E5F             lea     edx, [ebp+var_10]
.text:00432E62             lea     eax, [ebp+var_C] ; [ebp+var_C]存放 各部分注册码变换所得字符串
.text:00432E65             call    @System@AnsiString@$brplu$qqrrx17System@AnsiString ;
System::AnsiString::operator+=(System::AnsiString &)
```

这是注册码第一部分的变换过程，注册码形如 `XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`，每一部分都做类似的变换。

最后连接起来字符串形如 `"2383652768488373095639834072960844867409644"`。

## 3、RSA 加密验证

程序中使用了 `freelib` 大数库来进行 `RSA256` 变换。

就在注册码变换下面，有这么一段，对注册码的变换结果做了 `RSA` 加密处理，不过这一段是作者陪你玩的障眼法。

真正的注册码变换在注册对话框的定时器处理事件中。

```asm
...
.text:004337C1             mov     [ebp+var_D0], eax
.text:004337C7             mov     [ebp+var_D4], edx
.text:004337CD             lea     ecx, [ebp+var_CC]
.text:004337D3             push    ecx
.text:004337D4             push    offset unk_53B3DC
.text:004337D9             call    sub_43FAEC       ; zsread
.text:004337DE             add     esp, 8
.text:004337E1             lea     eax, [ebp+var_C8]
.text:004337E7             push    eax              ; int
.text:004337E8             push    offset a10001    ; buffer  "10001"
.text:004337ED             call    sub_43F8E8       ; zhsread
.text:004337F2             add     esp, 8
.text:004337F5             lea     edx, [ebp+var_C4]
.text:004337FB             push    edx              ; int
.text:004337FC             push    offset a504337c07ebd_0 ; buffer "504337C07EBD946C1CB"
.text:00433801             call    sub_43F8E8       ; zhsread
.text:00433806             add     esp, 8
.text:00433809             lea     ecx, [ebp+var_D4]
.text:0043380F             push    ecx
.text:00433810             mov     eax, [ebp+var_C4]
.text:00433816             push    eax
.text:00433817             mov     edx, [ebp+var_C8]
.text:0043381D             push    edx
.text:0043381E             mov     ecx, [ebp+var_CC]
.text:00433824             push    ecx
.text:00433825             call    sub_43EA60       ; zexpmod
.text:0043382A             add     esp, 10h
.text:0043382D             lea     eax, [ebp+var_D0]
.text:00433833             push    eax              ; int
.text:00433834             push    offset buffer    ; buffer
.text:00433839             call    sub_43F8E8       ; zhsread
.text:0043383E             add     esp, 8
.text:00433841             mov     edx, [ebp+var_D4]
.text:00433847             push    edx
.text:00433848             mov     ecx, [ebp+var_D0]
.text:0043384E             push    ecx
.text:0043384F             call    sub_43CBC0       ; zcompare
.text:00433854             add     esp, 8
.text:00433857             test    eax, eax
.text:00433859             jnz     loc_43391F
```

### 真正的验证注册码过程

```asm
.text:00433FA8 _TAboutBox_Timer1Timer proc near        ; DATA XREF: .data:005005F6o
...
.text:0043403F             lea     eax, [esp+14h+var_10]
.text:00434043             mov     [esp+14h+var_4], ecx
.text:00434047             push    eax              ; int
.text:00434048             push    offset a504337c07ebd_1 ; buffer
.text:0043404D             call    sub_43F8E8       ; zhsread  e = 504337C07EBD946C1CB
.text:00434052             add     esp, 8
.text:00434055             lea     edx, [esp+14h+var_C]
.text:00434059             push    edx
.text:0043405A             push    offset unk_53B3DC
.text:0043405F             call    sub_43FAEC       ; zsread
.text:00434064             add     esp, 8
.text:00434067             push    esp              ; int
```



### 4. 算法总结

```
.text:00434068                 push    offset a1f3662faa8e2_0 ; buffer   
.text:0043406D                 call    sub_43F8E8      ; zhsread  n =   
1F3662FAA8E266F962E0F02439186AC00561   
.text:00434072                 add     esp, 8   
.text:00434075                 lea     ecx, [esp+14h+var_4]   ; c   
.text:00434079                 push    ecx   
.text:0043407A                 mov     eax, [esp+18h+var_14]  ; n   
.text:0043407E                 push    eax   
.text:0043407F                 mov     edx, [esp+1Ch+var_10]  ; e   
.text:00434083                 push    edx   
.text:00434084                 mov     ecx, [esp+20h+var_C]   ; m   
.text:00434088                 push    ecx   
.text:00434089                 call    sub_43EA60      ; zexpmod   
.text:0043408E                 add     esp, 10h   
.text:00434091                 lea     eax, [esp+14h+var_8]   
.text:00434095                 push    eax             ; int   
.text:00434096                 push    offset buffer   ; buffer    
.text:0043409B                 call    sub_43F8E8      ; zhsread  读取对用户名处理的结果字符串，转为   
freelip 中的 verylong 格式   
.text:004340A0                 add     esp, 8   
.text:004340A3                 mov     edx, [esp+14h+var_4]  ; 用户名最终变换结果   
.text:004340A7                 push    edx   
.text:004340A8                 mov     ecx, [esp+18h+var_8]  ; 注册码变换结果   
.text:004340AC                 push    ecx   
.text:004340AD                 call    sub_43CBC0      ; zcompare  核心比较   
.text:004340B2                 add     esp, 8   
.text:004340B5                 test    eax, eax   
.text:004340B7                 jnz     loc_43416F      ; 注册码不正确则跳   
... 
```

一个很明显的 `RSA` 算法，在分析这个 `Call` 之前让我们以最快的速度来复习一下关于 `RSA` 的概念和公式。

`_______` 下面一小段引自 `dr0` 于 2000 年 08 月 22 日发表在看雪论坛的文章－《Windows 优化大师 v2.9+的注册码加密算法 》`_____` 

`RSA` 算法简述 

1、取两个素数 `p` 和 `q` 

2、计算 `n=pq,f=(p-1)(q-1)` 

3、随机选取整数 `e`,满足条件 `gcd(e, f)=1`,其中 `gcd` 为最大公约数 

4、计算 `d`,使得乘积 `de` 对 `f` 求余的结果为 `1`,即 `de` 和 `1` 对 `f` 同余 

上述只有 `e` 和 `n` 对外公开,用于加密 

加密过程(符号 `^` 表示乘幂,`mod` 表示求余) 设 `e` 为加密密钥,明文为 `m`,密文为 `c` 则加密公式如下 `c = (m ^ e) mod n` 

解密过程(设 `e` 为加密密钥,明文为 `m`,密文为 `c`,`d` 为解密密钥) 则解密公式如下 `m= (c ^ d) mod n` 

`____________________________________` 引用结束 感谢 `dr0` `____________________________________` 

现在我们来看一下我们的程序 

现在我们已知 `N=1F3662FAA8E266F962E0F02439186AC00561` `E=504337C07EBD946C1CB` 如上所说我们的首要任务就是求出 `D` 。运行 `tE!` 的 `RSATool` 求得 `D＝1A29B946F3C5D4721A0D685EE51A967AFD77` 

`\============================================================================` 

#### 4、算法总结： 

对用户名的处理较为简单 

相当于 

```
for(i=0; i<sizeof(szName); i++) 
{ 
    用户名变换结果 += (szName[i] * szName[i] + szName[i])*(i+1); 
} 
```

首先注册码形如 `XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`，共 5 部分，每一部分的位数不一定。 

对每一部分处理如下： 

```
for(int i=0; i<此部分总位数; i++) 
{ 
    F(Xn) += (59^i)*y; 
} 
```

其中 `i` 为此字节的所在位置从 0 开始计算 

    `y` 为使用此字节值在表 `'1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz'` 中的位置，查表所得处理所有注册码然后将各部分所得值化为字符串，再连接起来，所得就是我们用来 `RSA` 变换的待加密数。 

用 `RSA(N=1F3662FAA8E266F962E0F02439186AC00561` `E=504337C07EBD946C1CB)` 对该数进行变换后,所得结果再与用户名变换结果进行比较，如果相等则为注册成功。 

用户名和注册码进行简单 `xor` 后，保存在 `chess.dat` 中。 

写注册机过程中，`RSA` 变换公式固定，用起来也比较容易，这里让我头痛的是如何将 `RSA` 解密结果进行恰当分组。分组后再进行 `F(Xn) += (59^i)*y` 的反求注册码。 

因为注册码一共 5 组，而 32 位整数所能表示最大整数为 294967295，10 位整数。分组不当会造成溢出丢失数据。而且所以各组数开始有可能为 `'0'`,反求时也会丢失数据，从而无法生成正确结果。 

所以分组有两点要求 

(1)各分组大小 `<=9` 

(2)不能以 `'0'` 开头 

费了半天劲未果，最后一怒之下，使用五重循环进行穷举有效的分组。发挥它的特长--穷举, `^\_^` 

`\=========================================` 

给一组可用注册码 

`u: yesky1 [IPB]` 

`s: w0j0-nROHA-EeU7h-ygppt-jxwg32` 

`\=========================================` 

不足之处，请多多指教。 

`by yesky1 [BCG][IPB]` 

10.31

`4dd466d0e1e0cc123afbe504da2462ca`
