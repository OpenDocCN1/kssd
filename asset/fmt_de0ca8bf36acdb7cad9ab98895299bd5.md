
# 屏幕录像专家 v5.0 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6892.htm`](https://www.pediy.com/kssd/pediy06/pediy6892.htm)

【破文标题】 屏幕录像专家 v5.0 注册码破解分析

【破文作者】 80x86

【作者邮箱】 80x86@sohu.com

【使用工具】 `Ollydbg`, `IDA`

【破解平台】 `XP`

【软件名称】 屏幕录像专家

【下载地址】 `http://download.enet.com.cn/html/040312003020601.html`

【破解内容】

```
00437D20    /.  55                   push ebp
00437D21    |.  8BEC                 mov ebp,esp
00437D23    |.  81C4 CCFEFFFF        add esp,-134
00437D29    |.  53                   push ebx
00437D2A    |.  8995 44FFFFFF        mov dword ptr ss:[ebp-BC],edx
00437D30    |.  8985 48FFFFFF        mov dword ptr ss:[ebp-B8],eax
00437D36    |.  B8 FCBA5000          mov eax,屏录专家.0050BAFC
00437D3B    |.  E8 60440A00          call <屏录专家.@__InitExceptBlockLDTC>
00437D40    |.  66:C785 5CFFFFFF 0800 mov word ptr ss:[ebp-A4],8
00437D49    |.  8D45 FC              lea eax,dword ptr ss:[ebp-4]
00437D4C    |.  E8 BF9BFCFF          call <屏录专家.unknown_libname_37>
00437D51    |.  8BD0                 mov edx,eax
00437D53    |.  FF85 68FFFFFF        inc dword ptr ss:[ebp-98]
00437D59    |.  8B8D 48FFFFFF        mov ecx,dword ptr ss:[ebp-B8]
00437D5F    |.  8B81 E4020000        mov eax,dword ptr ds:[ecx+2E4]
00437D65    |.  E8 1E860500          call <屏录专家.@TControl@GetText$qqrv> ;取注册码
00437D6A    |.  8D55 FC              lea edx,dword ptr ss:[ebp-4]    ；**edx->注册码
00437D6D    |.  FF32                 push dword ptr ds:[edx]
00437D6F    |.  8D45 F8              lea eax,dword ptr ss:[ebp-8]
00437D72    |.  E8 999BFCFF          call <屏录专家.unknown_libname_37>
00437D77    |.  8BD0                 mov edx,eax
00437D79    |.  FF85 68FFFFFF        inc dword ptr ss:[ebp-98]
00437D7F    |.  8B8D 48FFFFFF        mov ecx,dword ptr ss:[ebp-B8]
00437D85    |.  8B81 DC020000        mov eax,dword ptr ds:[ecx+2DC]
00437D8B    |.  E8 F8850500          call <屏录专家.@TControl@GetText$qqrv> ;取注册名
00437D90    |.  8D55 F8              lea edx,dword ptr ss:[ebp-8]    ;**edx->注册名
00437D93    |.  FF32                 push dword ptr ds:[edx]
00437D95    |.  FFB5 48FFFFFF        push dword ptr ss:[ebp-B8]
00437D9B    |.  E8 180C0000          call <屏录专家.old_ver>       ;关于老版本
00437DA0    |.  83C4 0C              add esp,0C
00437DA3    |.  3C 01                cmp al,1
00437DA5    |.  0F94C1               sete cl
00437DA8    |.  83E1 01              and ecx,1
00437DAB    |.  51                   push ecx
00437DAC    |.  FF8D 68FFFFFF        dec dword ptr ss:[ebp-98]
00437DB2    |.  8D45 F8              lea eax,dword ptr ss:[ebp-8]
00437DB5    |.  BA 02000000          mov edx,2
00437DBA    |.  E8 11F70A00          call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00437DBF    |.  FF8D 68FFFFFF        dec dword ptr ss:[ebp-98]
00437DC5    |.  8D45 FC              lea eax,dword ptr ss:[ebp-4]
00437DC8    |.  BA 02000000          mov edx,2
00437DCD    |.  E8 FEF60A00          call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00437DD2    |.  59                   pop ecx
00437DD3    |.  84C9                 test cl,cl
00437DD5    |.  74 48                je short 屏录专家.00437E1F ；不是老版本
00437DD7    |.  66:C785 5CFFFFFF 1400 mov word ptr ss:[ebp-A4],14     ；注册码是老版本
00437DE0    |.  BA DCB75000          mov edx,屏录专家.0050B7DC
00437DE5    |.  8D45 F4              lea eax,dword ptr ss:[ebp-C]
00437DE8    |.  E8 ABF40A00          call <屏录专家.sub_4E7298>
00437DED    |.  FF85 68FFFFFF        inc dword ptr ss:[ebp-98]
00437DF3    |.  8B00                 mov eax,dword ptr ds:[eax]
00437DF5    |.  E8 46310500          call <屏录专家.@Dialogs@ShowMessage$qqrx17System@AnsiString>
00437DFA    |.  FF8D 68FFFFFF        dec dword ptr ss:[ebp-98]
00437E00    |.  8D45 F4              lea eax,dword ptr ss:[ebp-C]
00437E03    |.  BA 02000000          mov edx,2
00437E08    |.  E8 C3F60A00          call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00437E0D    |.  8B8D 4CFFFFFF        mov ecx,dword ptr ss:[ebp-B4]
00437E13    |.  64:890D 00000000     mov dword ptr fs:[0],ecx
00437E1A    |.  E9 7E0B0000          jmp 屏录专家.0043899D    ；返回
```

不是老版本：

```
00437E1F    |>  6A 14                push 14
00437E21    |.  6A 00                push 0
00437E23    |.  8D85 00FFFFFF        lea eax,dword ptr ss:[ebp-100]
00437E29    |.  50                   push eax
00437E2A    |.  E8 A13F0A00          call <屏录专家._memset>
00437E2F    |.  83C4 0C              add esp,0C
00437E32    |.  33D2                 xor edx,edx
00437E34    |.  8995 40FFFFFF        mov dword ptr ss:[ebp-C0],edx
00437E3A    |.  6A 14                push 14
00437E3C    |.  6A 00                push 0
00437E3E    |.  8D8D 18FFFFFF        lea ecx,dword ptr ss:[ebp-E8]
00437E44    |.  51                   push ecx
00437E45    |.  E8 863F0A00          call <屏录专家._memset>
00437E4A    |.  83C4 0C              add esp,0C
00437E4D    |.  6A 14                push 14
00437E4F    |.  6A 00                push 0
00437E51    |.  8D85 E8FEFFFF        lea eax,dword ptr ss:[ebp-118]
00437E57    |.  50                   push eax
00437E58    |.  E8 733F0A00          call <屏录专家._memset>
00437E5D    |.  83C4 0C              add esp,0C
00437E60    |.  66:C785 5CFFFFFF 2000 mov word ptr ss:[ebp-A4],20
00437E69    |.  8D45 F0              lea eax,dword ptr ss:[ebp-10]
00437E6C    |.  E8 9F9AFCFF          call <屏录专家.unknown_libname_37>
00437E71    |.  8BD0                 mov edx,eax
00437E73    |.  FF85 68FFFFFF        inc dword ptr ss:[ebp-98]
00437E79    |.  8B8D 48FFFFFF        mov ecx,dword ptr ss:[ebp-B8]
00437E7F    |.  8B81 DC020000        mov eax,dword ptr ds:[ecx+2DC]
00437E85    |.  E8 FE840500          call <屏录专家.@TControl@GetText$qqrv> ；取注册名
00437E8A    |.  8D45 F0              lea eax,dword ptr ss:[ebp-10]
             **eax->取注册名
00437E8D    |.  E8 46DEFCFF          call <屏录专家.@System@AnsiString@c_str$xqqrv>
00437E92    |.  50                   push eax
00437E93    |.  8D95 E8FEFFFF        lea edx,dword ptr ss:[ebp-118]
00437E99    |.  52                   push edx
00437E9A    |.  E8 5D400A00          call <屏录专家._strcpy>          ;copy 注册名
00437E9F    |.  83C4 08              add esp,8
00437EA2    |.  FF8D 68FFFFFF        dec dword ptr ss:[ebp-98]
00437EA8    |.  8D45 F0              lea eax,dword ptr ss:[ebp-10]
00437EAB    |.  BA 02000000          mov edx,2
00437EB0    |.  E8 1BF60A00          call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00437EB5    |.  66:C785 5CFFFFFF 2C00 mov word ptr ss:[ebp-A4],2C
00437EBE    |.  8D45 EC              lea eax,dword ptr ss:[ebp-14]
00437EC1    |.  E8 4A9AFCFF          call <屏录专家.unknown_libname_37>
00437EC6    |.  8BD0                 mov edx,eax
```
