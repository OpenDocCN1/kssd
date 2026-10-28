
# 汇编代码片段

```
Unpacker:00402B31                 push    eax             ; dwType
Unpacker:00402B32                 push    0               ; Reserved
Unpacker:00402B34                 push    ebx             ; lpValueName
Unpacker:00402B35                 mov     eax, [ebp+hKey]
Unpacker:00402B38                 push    eax             ; hKey
Unpacker:00402B39                 call    RegSetValueExA
Unpacker:00402B3E                 mov     ebx, eax
Unpacker:00402B40                 mov     eax, [ebp+hKey]
Unpacker:00402B43                 push    eax             ; hKey
Unpacker:00402B44                 call    RegCloseKey_0
Unpacker:00402B49                 mov     eax, ebx
Unpacker:00402B4B                 pop     ebx
Unpacker:00402B4C                 pop     ecx
Unpacker:00402B4D                 pop     ebp
Unpacker:00402B4E                 retn    0Ch
Unpacker:00402B4E AddRegKey       endp
```

在4个地方分别调用了这个函数创建键值:

`00402A34`创建

`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Game\JX\QY\Url`,内容为00(200h)

`00402C9B`创建`HKEY_CLASSES_ROOT\CLSID\{35FD6584-698F-BCD2-602C-698745210353}\InprocServer32\"`,

`00402CEC`创建`HKEY_CLASSES_ROOT\CLSID\{35FD6584-698F-BCD2-602C-698745210353}\InprocServer32\ThreadingModel`,

`00402D4C`创建

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\explorer\ShellExecuteHooks\{35FD6584-698F-BCD2-602C-698745210353}`,内容为"`rijxckin.dll`"

## 注入 explorer 进程

把`C:\WINDOWS\system32\rijxzkin.dll`文件注入到explorer进程

一共分三步完成

### 第一步: 获取explorer.exe的句柄

```
Unpacker:00402510 GetExplorerHandle proc near             ; CODE XREF:
sub_402DD8+8 p
Unpacker:00402510
Unpacker:00402510 var_138         = dword ptr -138h
Unpacker:00402510 var_114         = byte ptr -114h
Unpacker:00402510
Unpacker:00402510                 push    ebx
Unpacker:00402511                 push    esi
Unpacker:00402512                 push    edi
Unpacker:00402513                 push    ebp
Unpacker:00402514                 add     esp, 0FFFFFED8h
Unpacker:0040251A                 mov     ebx, edx
Unpacker:0040251C                 mov     esi, eax
Unpacker:0040251E                 xor     edi, edi
Unpacker:00402520                 xor     edx, edx
Unpacker:00402522                 mov     eax, 2
Unpacker:00402527                 call    CreateModuleSnapshot ; 创建快照
Unpacker:0040252C                 mov     ebp, eax
Unpacker:0040252E                 mov     [esp+138h+var_138], 128h
Unpacker:00402535                 mov     edx, esp
Unpacker:00402537                 mov     eax, ebp
Unpacker:00402539                 call    FindProcess     ; BDS 2005-2006 and
Delphi6-7 Visual Component Library
Unpacker:0040253E                 jmp     short loc_402560
Unpacker:00402540 ; -----------------------------------------------------------------
----------
Unpacker:00402540
Unpacker:00402540 loc_402540:                             ; CODE XREF:
GetExplorerHandle+58 j
Unpacker:00402540                 lea     eax, [esp+138h+var_114]
Unpacker:00402544
Unpacker:00402544 loc_402544:                             ; CODE XREF:
Unpacker:loc_40BA44 j
Unpacker:00402544                                         ; DATA XREF:
Unpacker:0040BA3F o
Unpacker:00402544                 push    eax
Unpacker:00402545                 push    esi
Unpacker:00402546                 call    lstrcmpi       ;比较线程名字是
否"explorer",不是的话通过循环继续获取比较
Unpacker:0040254B                 test    eax, eax
Unpacker:0040254D                 jnz     short loc_402557
Unpacker:0040254F                 mov     edi, [esp+140h+var_138]
Unpacker:00402553                 test    bl, bl
Unpacker:00402555                 jz      short loc_40256A
Unpacker:00402557
Unpacker:00402557 loc_402557:                             ; CODE XREF: GetExplorerHandle+3D j
Unpacker:00402557                 mov     edx, esp
Unpacker:00402559                 mov     eax, ebp
Unpacker:0040255B                 call    FindProcessNext ; 获取快照中的下一个
进程名字
Unpacker:00402560
Unpacker:00402560 loc_402560:                             ; CODE XREF:
GetExplorerHandle+2E j
Unpacker:00402560                 cmp     eax, 1
Unpacker:00402563                 sbb     eax, eax
Unpacker:00402565                 inc     eax
Unpacker:00402566                 cmp     al, 1
Unpacker:00402568                 jz      short loc_402540
Unpacker:0040256A
Unpacker:0040256A loc_40256A:                             ; CODE XREF:
GetExplorerHandle+45 j
Unpacker:0040256A                 push    ebp             ; hObject
Unpacker:0040256B                 call    CloseHandle
Unpacker:00402570                 mov     eax, edi
Unpacker:00402572                 add     esp, 128h
Unpacker:00402578                 pop     ebp
Unpacker:00402579                 pop     edi
Unpacker:0040257A                 pop     esi
Unpacker:0040257B                 pop     ebx
Unpacker:0040257B GetExplorerHandle endp ;
```

### 第二步: 检查explorer进程中是否已有rijxzkin.dll文件

```
Unpacker:00402580 FindDllModule   proc near               ; CODE XREF:
sub_402DD8+18 p
Unpacker:00402580
Unpacker:00402580 var_234         = dword ptr -234h
Unpacker:00402580 var_220         = dword ptr -220h
Unpacker:00402580 var_214         = byte ptr -214h
Unpacker:00402580 var_114         = byte ptr -114h
Unpacker:00402580
Unpacker:00402580                 push    ebx
Unpacker:00402581                 push    esi
Unpacker:00402582                 push    edi
Unpacker:00402583                 push    ebp
Unpacker:00402584                 add     esp, 0FFFFFDDCh
Unpacker:0040258A                 mov     edi, ecx
Unpacker:0040258C                 mov     ebx, edx
Unpacker:0040258E                 mov     esi, eax
Unpacker:00402590                 xor     ebp, ebp
Unpacker:00402592                 test    edi, edi
Unpacker:00402594                 jz      short loc_4025A2
Unpacker:00402596                 mov     edx, 104h
Unpacker:0040259B                 mov     eax, edi
Unpacker:0040259D                 call    @Windows@ZeroMemory$qqrpvui ;
Windows::ZeroMemory(void *,uint)
Unpacker:004025A2
Unpacker:004025A2 loc_4025A2:                             ; CODE XREF:
FindDllModule+14 j
Unpacker:004025A2                 mov     edx, esi
Unpacker:004025A4                 mov     eax, 8
Unpacker:004025A9                 call    CreateModuleSnapshot ; 创建快照
Unpacker:004025AE                 mov     esi, eax
Unpacker:004025B0                 mov     [esp+234h+var_234], 224h
Unpacker:004025B0                                         ; CODE XREF:
Unpacker:loc_40BAB4 j
Unpacker:004025B7                 mov     edx, esp
Unpacker:004025B9                 mov     eax, esi
Unpacker:004025BB                 call    FindModuleFirst ; 查找explorer进程中是
否已有"rijxzkin.dll"
Unpacker:004025C0                 cmp     eax, 1
Unpacker:004025C3                 sbb     eax, eax
Unpacker:004025C5                 inc     eax
Unpacker:004025C6                 cmp     al, 1
Unpacker:004025C8                 jnz     short loc_402608
Unpacker:004025CA
Unpacker:004025CA loc_4025CA:                             ; CODE XREF:
FindDllModule+86 j
Unpacker:004025CA                 test    ebx, ebx
Unpacker:004025CC                 jz      short loc_4025DD
Unpacker:004025CE                 lea     eax, [esp+234h+var_214]
Unpacker:004025D2                 push    eax
Unpacker:004025D3                 push    ebx
Unpacker:004025D4                 call    lstrcmpi
Unpacker:004025D9                 test    eax, eax
Unpacker:004025DB                 jnz     short loc_4025F5
Unpacker:004025DD
Unpacker:004025DD loc_4025DD:                             ; CODE XREF:
FindDllModule+4C j
Unpacker:004025DD                 test    edi, edi
Unpacker:004025DF                 jz      short loc_4025EF
Unpacker:004025E1                 lea     eax, [esp+234h+var_114]
Unpacker:004025E8                 push    eax
Unpacker:004025E9                 push    edi
```
