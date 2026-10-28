
# [原创]用HOOK的方法为`keynener_Assistant` v2.12添加支持拖放功能

`keynener_Assistant` v2.12是一个非常实用的工具,尤其是其中的密码学算法分析模块,对于密码学分析来说,有很好的辅助作用.

但程序不支持文件拖放,所以使用上不是很方便.

程序有3处需要打开文件路径,一是密码学分析,二是计算文件HASH,三是数据加密和解密

这3处如果支持拖放则会方便很多.

## 1. 分析功能`CALL`

首先要找到功能`CALL`,然后要找到其参数.

程序用UPX加壳,脱壳后用DelphiDecompiler分析,找到File Hash计算的Open按钮和密码学算法分析模块的Scan按钮响应事件代码:

### (1) 计算文件HASH功能,这里需要输入文件路径

Hashing->Calc Hash->File Hash->Open按钮事件响应代码:

```asm
0052AA28   55                     push    ebp
0052AA29   8BEC                   mov     ebp, esp
0052AA2B   6A00                   push    $00
0052AA2D   53                     push    ebx
0052AA2E   56                     push    esi
0052AA2F   57                     push    edi
0052AA30   8BF2                   mov     esi, edx   //ESI=EDX=0205207C  参数1
0052AA32   8BD8                   mov     ebx, eax   //EBX=EAX=02001074  参数2
0052AA34   33C0                   xor     eax, eax
0052AA36   55                     push    ebp
Possible String Reference to: '樽濏?腽_^[Y]?
|
0052AA37   6818AB5200             push    $0052AB18
0052AA3C   64FF30                 push    dword ptr fs:[eax]
0052AA3F   648920                 mov     fs:[eax], esp
Reference to field TForm1.OFFS_1747
|
0052AA42   80BB7A74010000         cmp     byte ptr [ebx+$1747A], $00
0052AA49   740C                   jz      0052AA57
Reference to field TForm1.OFFS_1747
|
0052AA4B   C6837974010001         mov     byte ptr [ebx+$17479], $01
0052AA52   E9AB000000             jmp     0052AB02
Reference to control TForm1.OpenDialog2 : TOpenDialog
|
0052AA57   8B8350070000           mov     eax, [ebx+$0750]
0052AA5D   8B10                   mov     edx, [eax]
Possible reference to virtual method TOpenDialog.OFFS_3C
|
0052AA5F   FF523C                 call    dword ptr [edx+$3C]   //打开文件对话框
0052AA62   84C0                   test    al, al
0052AA64   0F8498000000           jz      0052AB02
0052AA6A   8D55FC                 lea     edx, [ebp-$04]
Reference to control TForm1.OpenDialog2 : TOpenDialog
|
0052AA6D   8B8350070000           mov     eax, [ebx+$0750]
Reference to: Dialogs.TOpenDialog.GetFileName()
|
0052AA73   E82477F1FF             call    0044219C                   //获取文件路径
0052AA78   8B55FC                 mov     edx, [ebp-$04]             //EDX为路径
0052AA7B   B888505E00             mov     eax, $005E5088
|
0052AA80   E8A7A5EDFF             call    0040502C                    //这个CALL是判断是不是修改了默认选项
0052AA85   803D7B625E0001         cmp     byte ptr [$005E627B], $01
0052AA8C   7517                   jnz     0052AAA5                    //如果没有修改默认选项则直接跳到计算HASH的功能CALL
0052AA8E   6A00                   push    $00
0052AA90   668B0D28AB5200         mov     cx, word ptr [$0052AB28]
0052AA97   B203                   mov     dl, $03
Possible String Reference to: 'Some Hash you use are modified!You want to restore default values?'
0052AA99   B834AB5200             mov     eax, $0052AB34
Reference to: Dialogs.MessageDlg(System.AnsiString; Dialogs.TMsgDlgType; System.[Dialogs.TMsgDlgBtn]; Integer)
|
0052AA9E   E80D8BF1FF             call    004435B0
0052AAA3   8BF8                   mov     edi, eax
0052AAA5   4F                     dec     edi
0052AAA6   7551                   jnz     0052AAF9
0052AAA8   8BD6                   mov     edx, esi
0052AAAA   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn2Click()
|
0052AAAC   E863B4FFFF             call    00525F14    //以下都是选项
0052AAB1   8BD6                   mov     edx, esi
0052AAB3   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn4Click()
|
0052AAB5   E8BAB5FFFF             call    00526074
0052AABA   8BD6                   mov     edx, esi
0052AABC   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn5Click()
|
0052AABE   E811B7FFFF             call    005261D4
0052AAC3   8BD6                   mov     edx, esi
0052AAC5   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn8Click()
|
0052AAC7   E830B8FFFF             call    005262FC
0052AACC   8BD6                   mov     edx, esi
0052AACE   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn9Click()
|
0052AAD0   E87BB9FFFF             call    00526450
0052AAD5   8BD6                   mov     edx, esi
0052AAD7   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn11Click()
|
0052AAD9   E8AABBFFFF             call    00526688
0052AADE   8BD6                   mov     edx, esi
0052AAE0   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn13Click()
|
0052AAE2   E825BDFFFF             call    0052680C
0052AAE7   8BD6                   mov     edx, esi
0052AAE9   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn15Click()
|
0052AAEB   E80CBFFFFF             call    005269FC
0052AAF0   8BD6                   mov     edx, esi
0052AAF2   8BC3                   mov     eax, ebx
Reference to : TForm1.BitBtn17Click()
|
0052AAF4   E8F3C0FFFF             call    00526BEC    //选项结束
0052AAF9   8BD6                   mov     edx, esi
0052AAFB   8BC3                   mov     eax, ebx
Reference to : TForm1.FileHashing()
|
0052AAFD   E8E6FCFFFF             call    0052A7E8    //HASH计算CALL
0052AB02   33C0                   xor     eax, eax
0052AB04   5A                     pop     edx
0052AB05   59                     pop     ecx
0052AB06   59                     pop     ecx
0052AB07   648910                 mov     fs:[eax], edx
** FINALLY
|
Possible String Reference to: '_^[Y]?
|
0052AB0A   681FAB5200             push    $0052AB1F
0052AB0F   8D45FC                 lea     eax, [ebp-$04]
|
0052AB12   E8C1A4EDFF             call    00404FD8
0052AB17   C3                     ret
0052AB18   E9D79DEDFF             jmp     004048F4
0052AB1D   EBF0                   jmp     0052AB0F
** END
|
0052AB1F   5F                     pop     edi
0052AB20   5E                     pop     esi
0052AB21   5B                     pop     ebx
0052AB22   59                     pop     ecx
0052AB23   5D                     pop     ebp
0052AB24   C3                     ret
```

### (2) 密码学分析功能`CALL`

Scanning->San按钮事件响应代码:

```asm
00521734   55                     push    ebp
00521735   8BEC                   mov     ebp, esp
00521737   6A00                   push    $00
00521739   53                     push    ebx
0052173A   8BD8                   mov     ebx, eax
0052173C   33C0                   xor     eax, eax
0052173E   55                     push    ebp
0052173F   688B175200             push    $0052178B
00521744   64FF30                 push    dword ptr fs:[eax]
00521747   648920                 mov     fs:[eax], esp
Reference to control TForm1.OpenDialog1 : TOpenDialog
|
0052174A   8B8330030000           mov     eax, [ebx+$0330]
00521750   8B10                   mov     edx, [eax]
Possible reference to virtual method TOpenDialog.OFFS_3C
|
00521752   FF523C                 call    dword ptr [edx+$3C]
00521755   84C0                   test    al, al
00521757   741C                   jz      00521775
00521759   8D55FC                 lea     edx, [ebp-$04]
Reference to control TForm1.OpenDialog1 : TOpenDialog
|
0052175C   8B8330030000           mov     eax, [ebx+$0330]
Reference to: Dialogs.TOpenDialog.GetFileName()
|
00521762   E8350AF2FF             call    0044219C
00521767   8B55FC                 mov     edx, [ebp-$04]  //EDX为文件路径 参数1
Reference to control TForm1.Traget : TEdit
|
0052176A   8B834C030000           mov     eax, [ebx+$034C]//EAX为参数2,$034C为偏移,PATCH代码会用到
Reference to: Controls.TControl.SetText(System.AnsiString)
|
00521770   E8B394F2FF             call    0044AC28        //调用密码学分析CALL
00521775   33C0                   xor     eax, eax
00521777   5A                     pop     edx
00521778   59                     pop     ecx
00521779   59                     pop     ecx
```
