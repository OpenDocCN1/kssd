
```
LOCAL   szName[1024] : BYTE  
LOCAL   ea : EXPLICIT\_ACCESS  
LOCAL   pSacl, pOwner, pPrimaryGroup  
LOCAL   dwAclSize, dwSaclSize, dwOwnerSize, dwPrimaryGroup  
LOCAL   bSuccess  
sub     eax, eax  
mov     pSD, eax  
mov     pAbsSD, eax  
mov     dwSDLength, eax  
mov     bDaclPresent, eax  
mov     bDaclDefaulted, eax  
mov     pAcl, eax  
mov     pNewAcl, eax  
mov     pSacl, eax  
mov     pOwner, eax  
mov     pPrimaryGroup, eax  
mov     dwAclSize, eax  
mov     dwSaclSize, eax  
mov     dwOwnerSize, eax  
mov     dwPrimaryGroup, eax  
mov     bSuccess, eax  
invoke  GetKernelObjectSecurity, hToken, DACL\_SECURITY\_INFORMATION, pSD, 0, addr dwSDLength  
invoke  LocalAlloc, LPTR, dwSDLength  
test    eax, eax  
jz      \_exit  
mov     pSD, eax  
invoke  GetKernelObjectSecurity, hToken, DACL\_SECURITY\_INFORMATION, pSD, dwSDLength, addr dwSDLength  
invoke  GetSecurityDescriptorDacl, pSD, addr bDaclPresent, addr pAcl, addr bDaclDefaulted  
mov     eax, sizeof szName  
push    eax  
invoke  GetUserName, addr szName, esp  
pop     eax  
invoke  BuildExplicitAccessWithName, addr ea, addr szName, dwAccess, GRANT\_ACCESS, FALSE  
invoke  SetEntriesInAcl, 1, addr ea, pAcl, addr pNewAcl  
cmp     eax, ERROR\_SUCCESS  
jne     \_exit  
invoke  LocalFree, pAcl  
mov     pAcl, 0  
invoke  MakeAbsoluteSD, pSD, pAbsSD, addr dwSDLength, pAcl, addr dwAclSize, pSacl, addr dwSaclSize, \  
            pOwner, addr dwOwnerSize, pPrimaryGroup, addr dwPrimaryGroup  
invoke  LocalAlloc, LPTR, dwSDLength  
test    eax, eax  
jz      \_exit  
mov     pAbsSD, eax  
invoke  LocalAlloc, LPTR, dwAclSize  
test    eax, eax  
jz      \_exit  
mov     pAcl, eax  
invoke  LocalAlloc, LPTR, dwSaclSize  
test    eax, eax  
jz      \_exit  
mov     pSacl, eax  
invoke  LocalAlloc, LPTR, dwOwnerSize  
test    eax, eax  
jz      \_exit  
mov     pOwner, eax  
invoke  LocalAlloc, LPTR, dwPrimaryGroup  
test    eax, eax  
jz      \_exit  
mov     pPrimaryGroup, eax  
invoke  MakeAbsoluteSD, pSD, pAbsSD, addr dwSDLength, pAcl, addr dwAclSize, pSacl, addr dwSaclSize, \  
            pOwner, addr dwOwnerSize, pPrimaryGroup, addr dwPrimaryGroup  
invoke  SetSecurityDescriptorDacl, pAbsSD, bDaclPresent, pNewAcl, bDaclDefaulted  
invoke  SetKernelObjectSecurity, hToken, DACL\_SECURITY\_INFORMATION, pAbsSD  
mov     bSuccess, 1  
\_exit:  
.if pSD  
    invoke  LocalFree, pSD  
.endif  
.if pAcl  
    invoke  LocalFree, pAcl  
.endif  
.if pNewAcl  
    invoke  LocalFree, pNewAcl  
.endif  
.if pAbsSD  
    invoke  LocalFree, pAbsSD  
.endif  
.if pSacl  
    invoke  LocalFree, pSacl  
.endif  
.if pOwner  
    invoke  LocalFree, pOwner  
.endif  
.if pPrimaryGroup  
    invoke  LocalFree, pPrimaryGroup  
.endif  
mov     eax, bSuccess  
ret  
\_ModifySecurity endp  
```

```
\_EnablePrivilege proc szPriv:DWORD, bFlags:DWORD  
LOCAL   hToken  
LOCAL   tkp : TOKEN\_PRIVILEGES  
invoke  GetCurrentProcess  
mov     edx, eax  
invoke  OpenProcessToken, edx, TOKEN\_ADJUST\_PRIVILEGES or TOKEN\_QUERY, addr hToken  
invoke  LookupPrivilegeValue, NULL, szPriv, addr tkp.Privileges.Luid  
mov     tkp.PrivilegeCount, 1  
xor     eax, eax  
.if bFlags  
    mov     eax, SE\_PRIVILEGE\_ENABLED  
.endif  
mov     tkp.Privileges.Attributes, eax  
invoke  AdjustTokenPrivileges, hToken, FALSE, addr tkp, 0, 0, 0  
push    eax  
invoke  CloseHandle, hToken  
pop     eax  
ret  
\_EnablePrivilege endp  
```

```
\_GetPidFromProcName proc lpProcName:DWORD  
LOCAL   stProcess : PROCESSENTRY32  
LOCAL   hSnapshot  
LOCAL   dwProcessID  
mov     dwProcessID, 0  
invoke  RtlZeroMemory, addr stProcess, sizeof stProcess  
mov     stProcess.dwSize, sizeof stProcess  
invoke  CreateToolhelp32Snapshot, TH32CS\_SNAPPROCESS, 0  
mov     hSnapshot, eax  
invoke  Process32First, hSnapshot, addr stProcess  
.while eax  
    invoke  lstrcmpi, lpProcName, addr stProcess.szExeFile  
    .if eax==0  
        mov     eax, stProcess.th32ProcessID  
        mov     dwProcessID, eax  
        .break  
    .endif  
    invoke  Process32Next, hSnapshot, addr stProcess  
.endw  
invoke  CloseHandle, hSnapshot  
mov     eax, dwProcessID  
ret  
\_GetPidFromProcName endp  

end start  
```

### :make

```
set path=%path%;c:\\masm32\\bin  
set appname=GetSys2  
ml /nologo /c /coff %appname%.bat  
link /nologo /subsystem:windows %appname%.obj  
del %appname%.obj  
echo.  
pause  
```

`GetSys2`取得`lsass.exe`进程的令牌，缺省情况下操作这个令牌的权限很小，所以需要先取得操作这个令牌的所有权限。这个任务由函数`\_ModifySecurity`来完成。有了权限后，复制一个主令牌，然后在当前线程中扮演`SYSTEM`用户，接着就可以调用`CreateProcessAsUser`函数运行`regedit.exe`程序了。有关安全性编程不清楚的地方可以参考[3]。

### 3. HOOK ZwCreateProcessEx 函数

有关这个[1]里面讲得很清楚了，下面直接给出源代码。

```
;@echo off  
;goto make  
;====================================================================================  
; 一块三毛钱  
; [`zhongts.yeah.net`](http://zhongts.yeah.net)  
; zhongts@163.com  
; 2005.1.15  
;  
; 以 SYSTEM 权限运行程序 - GetSys3  
;  
; 采用 HOOK ZwCreateProcessEx 函数的方法  
;  
;====================================================================================  
.386  
.model flat, stdcall  
option casemap :none  
include c:\\masm32\\include\\windows.inc  
include c:\\masm32\\include\\kernel32.inc  
include c:\\masm32\\include\\advapi32.inc  
include c:\\masm32\\include\\masm32.inc  
includelib c:\\masm32\\lib\\kernel32.lib  
includelib c:\\masm32\\lib\\advapi32.lib  
includelib c:\\masm32\\lib\\masm32.lib  
\_EnablePrivilege proto :DWORD,:DWORD  
\_GetPidFromProcName proto :DWORD  
\_HackedZwCreateProcessEx proto  
CTXT MACRO text  
    local lbl  
    .const  
        lbl db text,0  
    .code  
    exitm <offset lbl>  
ENDM  
ASMJMP struct  
    mov\_eax         BYTE    ?  
    address         DWORD   ?  
    jmp\_eax         WORD    ?  
ASMJMP ends  
.data?  
    g\_hProc dd  ?  
    g\_dwFunc        dd  ?  
.code  
start proc  
    LOCAL   osvi : OSVERSIONINFO  
    LOCAL   lpAsmJmp  
    LOCAL   mbi : MEMORY\_BASIC\_INFORMATION  
    LOCAL   stStartupInfo : STARTUPINFO  
    LOCAL   procinfo : PROCESS\_INFORMATION  
    sub     eax, eax  
    mov     lpAsmJmp, eax  
    invoke  RtlZeroMemory, addr osvi, sizeof osvi  
    invoke  RtlZeroMemory, addr mbi, sizeof mbi  
    invoke  RtlZeroMemory, addr stStartupInfo, sizeof stStartupInfo  
    invoke  RtlZeroMemory, addr procinfo, sizeof procinfo  
    mov     osvi.dwOSVersionInfoSize, sizeof osvi  
    invoke  GetVersionEx, addr osvi  
    cmp     osvi.dwMajorVersion, 5  
    jnz     \_exit  
    .if osvi.dwMinorVersion==1  
        mov     g\_dwFunc, 30h  
    .elseif osvi.dwMinorVersion==2  
        mov     g\_dwFunc, 32h  
    .endif  
```
