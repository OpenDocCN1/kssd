
# 屏幕录像专家 V5.5 Build20050609 注册算法逆向

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-276.htm`](https://www.pediy.com/kssd/pediy07/pediy7-276.htm)

```asm
//注：这个是注册按钮对应的函数的逆向  

.text:00438E8C sub_438E8C      proc near          

.text:00438E8C  

.text:00438E8C var_154         = qword ptr -154h  

.text:00438E8C var_148         = dword ptr -148h  

.text:00438E8C Buffer          = byte ptr -144h  

.text:00438E8C var_131         = byte ptr -131h  

.text:00438E8C dest            = byte ptr -12Ch  

.text:00438E8C s               = dword ptr -114h  

.text:00438E8C buffer          = byte ptr -0FCh  

.text:00438E8C var_E4          = dword ptr -0E4h  

.text:00438E8C var_E0          = dword ptr -0E0h  

.text:00438E8C var_DC          = dword ptr -0DCh  

.text:00438E8C var_D8          = dword ptr -0D8h  

.text:00438E8C var_D4          = dword ptr -0D4h  

.text:00438E8C var_D0          = dword ptr -0D0h  

.text:00438E8C var_CC          = dword ptr -0CCh  

.text:00438E8C var_C8          = dword ptr -0C8h  

.text:00438E8C var_C4          = dword ptr -0C4h  

.text:00438E8C var_C0          = dword ptr -0C0h  

.text:00438E8C var_B0          = word ptr -0B0h  

.text:00438E8C var_A4          = dword ptr -0A4h  

.text:00438E8C var_9C          = dword ptr -9Ch  

.text:00438E8C var_98          = dword ptr -98h  

.text:00438E8C var_94          = dword ptr -94h  

.text:00438E8C var_90          = dword ptr -90h  

.text:00438E8C var_8C          = dword ptr -8Ch  

.text:00438E8C var_88          = dword ptr -88h  

.text:00438E8C var_84          = dword ptr -84h  

.text:00438E8C var_80          = dword ptr -80h  

.text:00438E8C var_7C          = dword ptr -7Ch  

.text:00438E8C var_78          = dword ptr -78h  

.text:00438E8C var_74          = dword ptr -74h  

.text:00438E8C var_70          = dword ptr -70h  

.text:00438E8C var_6C          = dword ptr -6Ch  

.text:00438E8C var_68          = dword ptr -68h  

.text:00438E8C var_64          = dword ptr -64h  

.text:00438E8C var_60          = dword ptr -60h  

.text:00438E8C var_5C          = dword ptr -5Ch  

.text:00438E8C var_58          = dword ptr -58h  

.text:00438E8C var_54          = dword ptr -54h  

.text:00438E8C var_50          = dword ptr -50h  

.text:00438E8C var_4C          = dword ptr -4Ch  

.text:00438E8C var_48          = dword ptr -48h  

.text:00438E8C var_44          = dword ptr -44h  

.text:00438E8C var_40          = dword ptr -40h  

.text:00438E8C var_3C          = dword ptr -3Ch  

.text:00438E8C var_38          = dword ptr -38h  

.text:00438E8C var_34          = dword ptr -34h  

.text:00438E8C var_30          = dword ptr -30h  

.text:00438E8C var_2C          = dword ptr -2Ch  

.text:00438E8C var_28          = dword ptr -28h  

.text:00438E8C var_24          = dword ptr -24h  

.text:00438E8C var_20          = dword ptr -20h  

.text:00438E8C var_1C          = dword ptr -1Ch  

.text:00438E8C var_18          = dword ptr -18h  

.text:00438E8C var_14          = dword ptr -14h  

.text:00438E8C var_10          = dword ptr -10h  

.text:00438E8C var_C           = dword ptr -0Ch  

.text:00438E8C var_8           = dword ptr -8  

.text:00438E8C var_4           = dword ptr -4  

.text:00438E8C  

.text:00438E8C                 push    ebp  

.text:00438E8D                 mov     ebp, esp  

.text:00438E8F                 add     esp, 0FFFFFEB8h  

.text:00438E95                 push    ebx  

.text:00438E96                 mov     [ebp+var_C8], edx  

.text:00438E9C                 mov     [ebp+var_C4], eax  

.text:00438EA2                 mov     eax, offset stru_50DFFC  

.text:00438EA7                 call    @__InitExceptBlockLDTC  

.text:00438EAC                 mov     [ebp+var_B0], 8  

.text:00438EB5                 lea     eax, [ebp+var_4]  

.text:00438EB8                 call    nop_38  

.text:00438EBD                 mov     edx, eax  

.text:00438EBF                 inc     [ebp+var_A4]  

.text:00438EC5                 mov     ecx, [ebp+var_C4]  

.text:00438ECB                 mov     eax, [ecx+2E4h]  

.text:00438ED1                 call    @TControl@GetText$qqrv ; TControl::GetText(void)  

.text:00438ED6                 lea     edx, [ebp+var_4]       //取假的注册码  

.text:00438ED9                 push    dword ptr [edx]  

.text:00438EDB                 lea     eax, [ebp+var_8]  

.text:00438EDE                 call    nop_38  

.text:00438EE3                 mov     edx, eax  

.text:00438EE5                 inc     [ebp+var_A4]  

.text:00438EEB                 mov     ecx, [ebp+var_C4]  

.text:00438EF1                 mov     eax, [ecx+2DCh]  

.text:00438EF7                 call    @TControl@GetText$qqrv ; TControl::GetText(void)  

.text:00438EFC                 lea     edx, [ebp+var_8]       //取注册名  

.text:00438EFF                 push    dword ptr [edx]  

.text:00438F01                 push    [ebp+var_C4]  

.text:00438F07                 call    sub_439CD0             //验证旧的注册码  

.text:00438F0C                 add     esp, 0Ch  

.text:00438F0F                 cmp     al, 1  

.text:00438F11                 setz    cl  

.text:00438F14                 and     ecx, 1  

.text:00438F17                 push    ecx  

.text:00438F18                 dec     [ebp+var_A4]  

.text:00438F1E                 lea     eax, [ebp+var_8]  

.text:00438F21                 mov     edx, 2  

.text:00438F26                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:00438F2B                 dec     [ebp+var_A4]  

.text:00438F31                 lea     eax, [ebp+var_4]  

.text:00438F34                 mov     edx, 2  

.text:00438F39                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:00438F3E                 pop     ecx  

.text:00438F3F                 test    cl, cl  

.text:00438F41                 jz      short loc_438F8B       //旧注册码验证失败则跳  

.text:00438F43                 mov     [ebp+var_B0], 14h  

.text:00438F4C                 mov     edx, offset aUFICVSIIVSIEVT ; "你输入的是老版本的注册码,请把注册时的定"...  

.text:00438F51                 lea     eax, [ebp+var_C]  

.text:00438F54                 call    ToStr_4E84C8  

.text:00438F59                 inc     [ebp+var_A4]  

.text:00438F5F                 mov     eax, [eax]  

.text:00438F61                 call    @Dialogs@ShowMessage$qqrx17System@AnsiString ; Dialogs::ShowMessage(System::AnsiString)  

.text:00438F66                 dec     [ebp+var_A4]  

.text:00438F6C                 lea     eax, [ebp+var_C]  

.text:00438F6F                 mov     edx, 2  

.text:00438F74                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:00438F79                 mov     ecx, [ebp+var_C0]  

.text:00438F7F                 mov     large fs:0, ecx  

.text:00438F86                 jmp     loc_439CB5  

.text:00438F8B ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

.text:00438F8B  

.text:00438F8B loc_438F8B:                    

.text:00438F8B                 push    14h             ; n  

.text:00438F8D                 push    0               ; c  

.text:00438F8F                 lea     eax, [ebp+s]  

.text:00438F95                 push    eax             ; s  

.text:00438F96                 call    _memset           //s 20 个字节清零  

.text:00438F9B                 add     esp, 0Ch  

.text:00438F9E                 xor     edx, edx  

.text:00438FA0                 mov     [ebp+var_CC], edx    //清零  

.text:00438FA6                 push    14h             ; n  

.text:00438FA8                 push    0               ; c  

.text:00438FAA                 lea     ecx, [ebp+buffer]  

.text:00438FB0                 push    ecx             ; s  

.text:00438FB1                 call    _memset           //buffer 20 个字节清零
```
