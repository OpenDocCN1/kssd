
# `_strlwr` 汇编实现

```text
.text:00418148                 jmp     loc_4181E6
.text:0041814D ; ---------------------------------------------------------------------------
.text:0041814D
.text:0041814D loc_41814D:                             ; CODE XREF: _strlwr+14j
.text:0041814D                 push    1               ; int
.text:0041814F                 push    0               ; CodePage
.text:00418151                 push    0               ; cchDest
.text:00418153                 push    0               ; lpDestStr
.text:00418155                 push    0FFFFFFFFh      ; cbMultiByte
.text:00418157                 mov     edx, [ebp+lpMultiByteStr]
.text:0041815A                 push    edx             ; lpMultiByteStr
.text:0041815B                 push    100h            ; dwMapFlags
.text:00418160                 mov     eax, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h
.text:00418165                 push    eax             ; Locale
.text:00418166                 call    ___crtLCMapStringA
.text:0041816B                 add     esp, 20h
.text:0041816E                 mov     [ebp+cchDest], eax
.text:00418171                 cmp     [ebp+cchDest], 0
.text:00418175                 jnz     short loc_418179
.text:00418177                 jmp     short loc_4181D5
.text:00418179 ; ---------------------------------------------------------------------------
.text:00418179
.text:00418179 loc_418179:                             ; CODE XREF: _strlwr+85j
.text:00418179                 push    64h
.text:0041817B                 push    offset aStrlwr_c ; "strlwr.c"
.text:00418180                 push    2
.text:00418182                 mov     ecx, [ebp+cchDest]
.text:00418185                 push    ecx
.text:00418186                 call    __malloc_dbg
.text:0041818B                 add     esp, 10h
.text:0041818E                 mov     [ebp+lpDestStr], eax
.text:00418191                 cmp     [ebp+lpDestStr], 0
.text:00418195                 jnz     short loc_418199
.text:00418197                 jmp     short loc_4181D5
.text:00418199 ; ---------------------------------------------------------------------------
.text:00418199
.text:00418199 loc_418199:                             ; CODE XREF: _strlwr+A5j
.text:00418199                 push    1               ; int
.text:0041819B                 push    0               ; CodePage
.text:0041819D                 mov     edx, [ebp+cchDest]
.text:004181A0                 push    edx             ; cchDest
.text:004181A1                 mov     eax, [ebp+lpDestStr]
.text:004181A4                 push    eax             ; lpDestStr
.text:004181A5                 push    0FFFFFFFFh      ; cbMultiByte
.text:004181A7                 mov     ecx, [ebp+lpMultiByteStr]
.text:004181AA                 push    ecx             ; lpMultiByteStr
.text:004181AB                 push    100h            ; dwMapFlags
.text:004181B0                 mov     edx, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h
.text:004181B6                 push    edx             ; Locale
.text:004181B7                 call    ___crtLCMapStringA
.text:004181BC                 add     esp, 20h
.text:004181BF                 test    eax, eax
.text:004181C1                 jnz     short loc_4181C5
.text:004181C3                 jmp     short loc_4181D5
.text:004181C5 ; ---------------------------------------------------------------------------
.text:004181C5
.text:004181C5 loc_4181C5:                             ; CODE XREF: _strlwr+D1j
.text:004181C5                 mov     eax, [ebp+lpDestStr]
.text:004181C8                 push    eax             ; Source
.text:004181C9                 mov     ecx, [ebp+lpMultiByteStr]
.text:004181CC                 push    ecx             ; Dest
.text:004181CD                 call    _strcpy
.text:004181D2                 add     esp, 8
.text:004181D5
.text:004181D5 loc_4181D5:                             ; CODE XREF: _strlwr+87j
.text:004181D5                                         ; _strlwr+A7j ...
.text:004181D5                 push    2
.text:004181D7                 mov     edx, [ebp+lpDestStr]
.text:004181DA                 push    edx
.text:004181DB                 call    __free_dbg
.text:004181E0                 add     esp, 8
.text:004181E3                 mov     eax, [ebp+lpMultiByteStr]
.text:004181E6
.text:004181E6 loc_4181E6:                             ; CODE XREF: _strlwr+58j
.text:004181E6                 mov     esp, ebp
.text:004181E8                 pop     ebp
.text:004181E9                 retn
.text:004181E9 _strlwr         endp
```

`strlwr()`与`strupr()`的处理方式类似，用两种方式进行字符串转换。一种通过`LCMapString`，它传递的标志为`0x100`而`strupr()`传递的为`0x200h`。

## 11、`strnx` 系列函数

`strnx`系列函数类似只是将条件转换成计数器。

## 12、`strpbrk`

```text
.text:00408D90                 push    ebp             ; 保存前栈帧栈基
.text:00408D91                 mov     ebp, esp        ; 开辟新栈帧
.text:00408D93                 push    esi             ; 保存esi
.text:00408D94                 xor     eax, eax        ; eax清0
.text:00408D96                 push    eax
.text:00408D97                 push    eax
.text:00408D98                 push    eax
.text:00408D99                 push    eax
.text:00408D9A                 push    eax
.text:00408D9B                 push    eax
.text:00408D9C                 push    eax
.text:00408D9D                 push    eax             ; 初始化栈空间，用于bt指令
.text:00408D9E                 mov     edx, [ebp+control] ; 要查询的字符串地址保存到edx中
.text:00408DA1                 lea     ecx, [ecx+0]    ; [ecx+0] ->Ecx未知
.text:00408DA4
.text:00408DA4 listnext:                               ; CODE XREF: _strpbrk+1Fj
.text:00408DA4                 mov     al, [edx]       ; 取一个字符
.text:00408DA6                 or      al, al          ; 如果为0字符
.text:00408DA8                 jz      short listdone
.text:00408DAA                 inc     edx             ; 地址递增 CF不变
.text:00408DAB                 bts     [esp+24h+var_24], eax ; eax为序号，var_24为顺序表起始地埴，将eax所指的bit位设置为CF
.text:00408DAB                                         ; 同时将该位设置为1
.text:00408DAF                 jmp     short listnext  ; 继续下一个字符
.text:00408DB1 ; ---------------------------------------------------------------------------
.text:00408DB1
.text:00408DB1 listdone:                               ; CODE XREF: _strpbrk+18j
.text:00408DB1                 mov     esi, [ebp+string] ; esi=string
.text:00408DB4
.text:00408DB4 dstnext:                                ; CODE XREF: _strpbrk+2Fj
.text:00408DB4                 mov     al, [esi]       ; 取第一个字符
.text:00408DB6                 or      al, al          ; 判断是否为0字符
.text:00408DB8                 jz      short dstdone   ; 是0则跳到结束位置
.text:00408DBA                 inc     esi             ; 递增地址，CF不变
.text:00408DBB                 bt      [esp+24h+var_24], eax ; var_24为线性表起始位置，eax为inx序号将该序号的值保存到cf中
.text:00408DBF                 jnb     short dstnext   ; 小于则表示cf=0继续循环
.text:00408DC1                 lea     eax, [esi-1]    ; 因为上面inc esi了，所以如果CF=1即表示找到，还需要-1定位其地址
.text:00408DC4
.text:00408DC4 dstdone:                                ; CODE XREF: _strpbrk+28j
.text:00408DC4                 add     esp, 20h        ; 回收栈帧
.text:00408DC7                 pop     esi
.text:00408DC8                 leave
.text:00408DC9                 retn
.text:00408DC9 _strpbrk        endp
```

类似于上面的 `strcspn`函数。

## 13、`strrev`

```text
.text:00418FA0                 push    ebp             ; 保存前栈帧栈基
.text:00418FA1                 mov     ebp, esp        ; 开辟新栈帧
.text:00418FA3                 push    edi             ; 保存esi edi
.text:00418FA4                 push    esi
.text:00418FA5                 mov     edi, [ebp+string] ; edi =string地址
.text:00418FA8                 mov     edx, edi        ; edx=string地址
.text:00418FAA                 mov     esi, edi        ; esi=string地址
.text:00418FAC                 xor     eax, eax        ; eax清0
```
