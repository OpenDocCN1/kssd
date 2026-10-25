
# 楚汉棋缘 v1.43 nag窗口注册按钮的代码逆向

**标题**：楚汉棋缘 v1.43 nag窗口的注册按钮的代码逆向

**作者**：ForEver[CCG][RCT]

**工具**：`ida 4.6`

**加壳**：`aspack2.12`修改版

**算法**：`RSA`

**感谢**：`xIkUg` 帮助脱壳。这个壳我搞不定(~!@#$%^&\*)

【正文】下面算法部分使用到了 `RSA` 算法，用的是 `freelip` 库。大致方法就是把用户名变换成一个`verylong`型数，然后再把注册码的每一组查表变换后形成一个加密数据。然后用`RSA`的公匙解密。得到另一个`verylong`型数，再和前面的比较。如果相同就会做些标记。注册信息保存在`chess.dat`里。

如果你还不太清楚`RSA`算法，就先看看前面的高手发的贴子吧。在看雪的精华里能找到几篇。或者也可以看看密码学的书。(不要仍鸡蛋给我。这个帖子已经够长了。再说老重复发相同的内容也不好吧。^\_^)

下面进入正文，中间的代码是用`VC`表达的。（我还不会`C++ builder`,汗！）

```
.text:0043FCFC _TForm1_SpeedButton1Click proc near     ; DATA XREF: .data:00505A9Fo
.text:0043FCFC
.text:0043FCFC var_1D8         = qword ptr -1D8h
.text:0043FCFC var_1C4         = qword ptr -1C4h
.text:0043FCFC var_1BC         = qword ptr -1BCh
.text:0043FCFC var_1B4         = dword ptr -1B4h
.text:0043FCFC var_1B0         = dword ptr -1B0h
.text:0043FCFC var_1AC         = dword ptr -1ACh
.text:0043FCFC s               = dword ptr -160h
.text:0043FCFC var_14D         = dword ptr -14Dh
.text:0043FCFC buffer          = byte ptr -0E8h
.text:0043FCFC var_D4          = dword ptr -0D4h
.text:0043FCFC var_D0          = dword ptr -0D0h
.text:0043FCFC var_CC          = dword ptr -0CCh
.text:0043FCFC var_C8          = dword ptr -0C8h
.text:0043FCFC var_C4          = dword ptr -0C4h
.text:0043FCFC var_C0          = dword ptr -0C0h
.text:0043FCFC var_B4          = dword ptr -0B4h
.text:0043FCFC var_B0          = dword ptr -0B0h
.text:0043FCFC var_AC          = dword ptr -0ACh
.text:0043FCFC var_A8          = dword ptr -0A8h
.text:0043FCFC var_A4          = dword ptr -0A4h
.text:0043FCFC var_A0          = dword ptr -0A0h
.text:0043FCFC var_9C          = dword ptr -9Ch
.text:0043FCFC var_98          = dword ptr -98h
.text:0043FCFC var_94          = word ptr -94h
.text:0043FCFC var_88          = dword ptr -88h
.text:0043FCFC var_80          = dword ptr -80h
.text:0043FCFC var_7C          = dword ptr -7Ch
.text:0043FCFC var_78          = dword ptr -78h
.text:0043FCFC var_74          = dword ptr -74h
.text:0043FCFC var_70          = dword ptr -70h
.text:0043FCFC var_6C          = dword ptr -6Ch
.text:0043FCFC var_68          = dword ptr -68h
.text:0043FCFC var_64          = dword ptr -64h
.text:0043FCFC var_60          = dword ptr -60h
.text:0043FCFC var_5C          = dword ptr -5Ch
.text:0043FCFC var_58          = dword ptr -58h
.text:0043FCFC var_54          = dword ptr -54h
.text:0043FCFC var_50          = dword ptr -50h
.text:0043FCFC var_4C          = dword ptr -4Ch
.text:0043FCFC var_48          = dword ptr -48h
.text:0043FCFC var_44          = dword ptr -44h
.text:0043FCFC var_40          = dword ptr -40h
.text:0043FCFC var_3C          = dword ptr -3Ch
.text:0043FCFC var_38          = dword ptr -38h
.text:0043FCFC var_34          = dword ptr -34h
.text:0043FCFC var_30          = dword ptr -30h
.text:0043FCFC var_2C          = dword ptr -2Ch
.text:0043FCFC var_28          = dword ptr -28h
.text:0043FCFC var_24          = dword ptr -24h
.text:0043FCFC var_20          = dword ptr -20h
.text:0043FCFC var_1C          = dword ptr -1Ch
.text:0043FCFC var_18          = dword ptr -18h
.text:0043FCFC var_14          = dword ptr -14h
.text:0043FCFC var_10          = dword ptr -10h
.text:0043FCFC var_C           = dword ptr -0Ch
.text:0043FCFC rcode           = dword ptr -8
.text:0043FCFC var_4           = dword ptr -4
.text:0043FCFC
.text:0043FCFC                 push    ebp
.text:0043FCFD                 mov     ebp, esp
.text:0043FCFF                 add     esp, 0FFFFFE3Ch
.text:0043FD05                 push    ebx
.text:0043FD06                 push    esi
.text:0043FD07                 push    edi
.text:0043FD08                 mov     [ebp+var_AC], edx
.text:0043FD0E                 mov     [ebp+var_A8], eax
.text:0043FD14                 xor     eax, eax
.text:0043FD16                 mov     [ebp+var_9C], offset unk_505750
.text:0043FD20                 mov     [ebp+var_98], esp
.text:0043FD26                 mov     [ebp+var_A0], offset unk_4C62F7
.text:0043FD30                 mov     [ebp+var_94], 0
.text:0043FD39                 mov     [ebp+var_88], eax
.text:0043FD3F                 mov     edx, large fs:0
.text:0043FD46                 mov     [ebp+var_A4], edx
.text:0043FD4C                 lea     ecx, [ebp+var_A4]
.text:0043FD52                 mov     large fs:0, ecx
.text:0043FD59                 mov     [ebp+var_94], 14h
.text:0043FD62                 mov     edx, offset a1234567890abcd ; "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabc"...
.text:0043FD67                 lea     eax, [ebp+var_4] ***** var_4是一张表
.text:0043FD6A                 call    tostring
.text:0043FD6F                 inc     [ebp+var_88]
.text:0043FD75                 xor     ecx, ecx
.text:0043FD77                 mov     [ebp+var_94], 8
.text:0043FD80                 mov     [ebp+var_94], 20h
.text:0043FD89                 mov     [ebp+rcode], ecx
.text:0043FD8C                 xor     eax, eax
.text:0043FD8E                 inc     [ebp+var_88]
.text:0043FD94                 xor     edx, edx
.text:0043FD96                 mov     [ebp+var_94], 8
.text:0043FD9F                 mov     [ebp+var_94], 2Ch
.text:0043FDA8                 mov     [ebp+var_C], eax ***** var_c = 0
.text:0043FDAB                 xor     ecx, ecx
.text:0043FDAD                 inc     [ebp+var_88]
.text:0043FDB3                 lea     eax, [ebp+s]
.text:0043FDB9                 mov     [ebp+var_94], 8
.text:0043FDC2                 mov     [ebp+var_94], 38h
.text:0043FDCB                 mov     [ebp+var_10], edx ***** var_10 = 0
.text:0043FDCE                 xor     edi, edi
.text:0043FDD0                 inc     [ebp+var_88]
.text:0043FDD6                 mov     [ebp+var_94], 8
.text:0043FDDF                 mov     [ebp+var_94], 44h
.text:0043FDE8                 mov     [ebp+var_14], ecx ***** var_14 = 0
.text:0043FDEB                 inc     [ebp+var_88]
.text:0043FDF1                 xor     esi, esi
.text:0043FDF3                 mov     [ebp+var_94], 8
.text:0043FDFC                 push    14h             ; n
.text:0043FDFE                 push    0               ; c
.text:0043FE00                 push    eax             ; s
.text:0043FE01                 call    _memset           ***** memset(s,0,20);
.text:0043FE06                 mov     [ebp+var_94], 50h
.text:0043FE0F                 xor     edx, edx
.text:0043FE11                 add     esp, 0Ch
.text:0043FE14                 mov     [ebp+var_2C], edx
.text:0043FE17                 lea     edx, [ebp+var_2C] ****** 用户名缓冲区
.text:0043FE1A                 inc     [ebp+var_88]
.text:0043FE20                 mov     eax, [ebp+var_A8]
.text:0043FE26                 mov     eax, [eax+2E0h]
.text:0043FE2C                 call    @TControl@GetText$qqrv ; TControl::GetText(void) 取用户名
.text:0043FE31                 cmp     [ebp+var_2C], 0
.text:0043FE35                 jz      short loc_43FE3C  ***** 用户名为空则跳
.text:0043FE37                 mov     edx, [ebp+var_2C]
.text:0043FE3A                 jmp     short loc_43FE41
.text:0043FE3C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0043FE3C
.text:0043FE3C loc_43FE3C:                             ; CODE XREF: _TForm1_SpeedButton1Click+139j
.text:0043FE3C                 mov     edx, offset unk_505431
.text:0043FE41
.text:0043FE41 loc_43FE41:                             ; CODE XREF: _TForm1_SpeedButton1Click+13Ej
.text:0043FE41                 push    esi
.text:0043FE42                 push    edi
.text:0043FE43                 xor     eax, eax
.text:0043FE45                 mov     edi, edx
.text:0043FE47                 or      ecx, 0FFFFFFFFh
.text:0043FE4A                 lea     esi, [ebp+s]  ****** 用户名拷贝到 s
```
