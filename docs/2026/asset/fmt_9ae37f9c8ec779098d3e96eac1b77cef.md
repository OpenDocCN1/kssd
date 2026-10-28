
```
.text:00401070 sub_401070      proc near               ; CODE XREF: WinMain(x,x,x,x)+7Fp

.text:00401070

.text:00401070 pe              = PROCESSENTRY32 ptr -128h

.text:00401070

.text:00401070                 sub     esp, 128h

.text:00401076                 push    ebx

.text:00401077                 push    ebp

.text:00401078                 push    esi

.text:00401079                 mov     ebx, ecx

.text:0040107B                 push    edi

.text:0040107C                 xor     eax, eax

.text:0040107E                 mov     ecx, 49h

.text:00401083                 lea     edi, [esp+138h+pe.cntUsage]

.text:00401087                 mov     [esp+138h+pe.dwSize], 128h

.text:0040108F                 push    eax             ; th32ProcessID

.text:00401090                 rep stosd

.text:00401092                 push    2               ; dwFlags

.text:00401094                 call    CreateToolhelp32Snapshot

.text:00401099                 mov     edi, eax

.text:0040109B                 cmp     edi, 0FFFFFFFFh

.text:0040109E                 jz      short loc_40110A

.text:004010A0                 lea     eax, [esp+138h+pe]

.text:004010A4                 push    eax             ; lppe

.text:004010A5                 push    edi             ; hSnapshot

.text:004010A6                 call    Process32First

.text:004010AB                 test    eax, eax

.text:004010AD                 jz      short loc_401103

.text:004010AF                 lea     ebp, [ebx+8]

.text:004010B2

.text:004010B2 loc_4010B2:                             ; CODE XREF: sub_401070+91j

.text:004010B2                 push    ebp             ; String

.text:004010B3                 call    __strlwr

.text:004010B8                 lea     ecx, [esp+13Ch+pe.szExeFile]

.text:004010BC                 mov     esi, eax

.text:004010BE                 push    ecx             ; String

.text:004010BF                 call    __strlwr

.text:004010C4                 add     esp, 8

.text:004010C7

.text:004010C7 loc_4010C7:                             ; CODE XREF: sub_401070+75j

.text:004010C7                 mov     dl, [eax]

.text:004010C9                 mov     cl, dl

.text:004010CB                 cmp     dl, [esi]

.text:004010CD                 jnz     short loc_4010EB

.text:004010CF                 test    cl, cl

.text:004010D1                 jz      short loc_4010E7

.text:004010D3                 mov     dl, [eax+1]

.text:004010D6                 mov     cl, dl

.text:004010D8                 cmp     dl, [esi+1]

.text:004010DB                 jnz     short loc_4010EB

.text:004010DD                 add     eax, 2

.text:004010E0                 add     esi, 2

.text:004010E3                 test    cl, cl

.text:004010E5                 jnz     short loc_4010C7

.text:004010E7

.text:004010E7 loc_4010E7:                             ; CODE XREF: sub_401070+61j

.text:004010E7                 xor     eax, eax

.text:004010E9                 jmp     short loc_4010F0

.text:004010EB ; ---------------------------------------------------------------------------

.text:004010EB

.text:004010EB loc_4010EB:                             ; CODE XREF: sub_401070+5Dj

.text:004010EB                                         ; sub_401070+6Bj

.text:004010EB                 sbb     eax, eax

.text:004010ED                 sbb     eax, 0FFFFFFFFh

.text:004010F0

.text:004010F0 loc_4010F0:                             ; CODE XREF: sub_401070+79j

.text:004010F0                 test    eax, eax

.text:004010F2                 jz      short loc_401117

.text:004010F4                 lea     eax, [esp+138h+pe]

.text:004010F8                 push    eax             ; lppe

.text:004010F9                 push    edi             ; hSnapshot

.text:004010FA                 call    Process32Next

.text:004010FF                 test    eax, eax

.text:00401101                 jnz     short loc_4010B2

.text:00401103

.text:00401103 loc_401103:                             ; CODE XREF: sub_401070+3Dj

.text:00401103                 push    edi             ; hObject

.text:00401104                 call    ds:CloseHandle

.text:0040110A

.text:0040110A loc_40110A:                             ; CODE XREF: sub_401070+2Ej

.text:0040110A                 pop     edi

.text:0040110B                 pop     esi

.text:0040110C                 pop     ebp

.text:0040110D                 xor     eax, eax

.text:0040110F                 pop     ebx

.text:00401110                 add     esp, 128h

.text:00401116                 retn

.text:00401117 ; ---------------------------------------------------------------------------

.text:00401117

.text:00401117 loc_401117:                             ; CODE XREF: sub_401070+82j

.text:00401117                 mov     eax, [esp+138h+pe.th32ProcessID]

.text:0040111B                 pop     edi

.text:0040111C                 pop     esi

.text:0040111D                 mov     [ebx+4], eax

.text:00401120                 pop     ebp

.text:00401121                 pop     ebx

.text:00401122                 add     esp, 128h

.text:00401128                 retn

.text:00401128 sub_401070      endp
```

发现这里遍历进程查找`explorer`进程，估计后面不用讲也知道这个程序就是一个shell，它将`dll`释放到c盘下，然后提升自己的权限，找到`explorer`的进程，将`dll`注入其中运行以达到隐藏进程的目的..很简单..

### 解决方案

反过来就是了..提升自身权限，查找到`explorer`进程注入代码将`dll`卸载掉..等等，这样不行，还有线程没结束，会导致`explorer`崩溃的，所以在做`dll`卸载前要做的一件事就是将与之有关的线程结束掉。所以我们还要枚举线程，将得到`TBI`表，以便得到线程起始地址，然后通过`GetMappedFileName`确定是不是在`GUI32.dll`当中。

### 完整代码
