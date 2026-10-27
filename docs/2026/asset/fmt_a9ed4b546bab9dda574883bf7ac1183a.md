
```asm
        mov        edx,esi
        add        edx,sizeof IMAGE\_NT\_HEADERS
        mov        cx,[esi].FileHeader.NumberOfSections
        movzx        ecx,cx
        assume        edx:ptr IMAGE\_SECTION\_HEADER
        .while        ecx > 0
                .if        edi >= [edx].PointerToRawData
                        mov        eax,[edx].PointerToRawData
                        add        eax,[edx].SizeOfRawData
                        .if        edi < eax
                                mov        eax,[edx].PointerToRawData
                                sub        edi,eax
                                mov        eax,[edx].VirtualAddress
                                add        eax,edi
                                ret
                        .endif
                .endif
                add edx,sizeof IMAGE\_SECTION\_HEADER
                dec ecx
        .endw
        assume        edx:nothing
        assume        esi:nothing
        mov        eax,FALSE
        ret
Offset2RVA        endp

GetImportsInfo        proc        uses ebx esi edi @lpImportsInfo:dword
        local        @dwIATMin:dword
        local        @dwIATMax:dword
        mov        esi,lpPeHeader
        assume        esi:ptr IMAGE\_NT\_HEADERS
        lea        eax,[esi].OptionalHeader.DataDirectory\
                        [IMAGE\_DIRECTORY\_ENTRY\_IMPORT\*sizeof IMAGE\_DATA\_DIRECTORY]
        mov        esi,eax
        assume        esi:ptr IMAGE\_DATA\_DIRECTORY
        mov        edi,@lpImportsInfo
        assume        edi:ptr IMPORTS\_INFO
        mov        eax,[esi].VirtualAddress
        mov        [edi].dwImportDirRVA,eax
        invoke        RVA2Offset,eax
        add        eax,lpFileHeader
        mov        [edi].lpImportDir,eax
        push        [esi].isize
        pop        [edi].dwImportDirSize
        xor        eax,eax
        mov        @dwIATMax,eax
        or        eax,7FFFFFFFh
        mov        @dwIATMin,eax
        mov        eax,[esi].VirtualAddress
        invoke        RVA2Offset,eax
        add        eax,lpFileHeader
        mov        esi,eax
        assume        esi:ptr IMAGE\_IMPORT\_DESCRIPTOR
        .while        [esi].OriginalFirstThunk || [esi].TimeDateStamp || \
                        [esi].ForwarderChain || [esi].Name1 || [esi].FirstThunk
                .if        [esi].OriginalFirstThunk
                        mov        ebx,[esi].OriginalFirstThunk
                .else
                        mov        ebx,[esi].FirstThunk
                .endif
                invoke        RVA2Offset,ebx
                mov        ebx,eax
                add        ebx,lpFileHeader
                mov        eax,[esi].FirstThunk
                .while        dword ptr [ebx]
                        .if        eax < @dwIATMin
                                mov        @dwIATMin,eax
                        .endif
                        .if        eax > @dwIATMax
                                mov        @dwIATMax,eax
                        .endif
                        add        eax,4
                        add        ebx,4
                .endw
                add        esi,sizeof IMAGE\_IMPORT\_DESCRIPTOR
        .endw
        push        @dwIATMin
        pop        [edi].dwIATDirRVA
        mov        eax,@dwIATMax
        sub        eax,@dwIATMin
        add        eax,4
        mov        [edi].dwIATDirSize,eax
        invoke        RVA2Offset,@dwIATMin
        add        eax,lpFileHeader
        mov        [edi].lpIATDir,eax
        assume        esi:nothing
        assume        edi:nothing
        ret
GetImportsInfo        endp

GetCodeScnInfo        proc        uses ebx ecx edx esi edi @lpCodeScnInfo:dword
        mov        esi,lpPeHeader
        assume        esi:ptr IMAGE\_NT\_HEADERS
        mov        ebx,[esi].OptionalHeader.AddressOfEntryPoint
        mov        edx,esi
        add        edx,sizeof IMAGE\_NT\_HEADERS
        mov        cx,[esi].FileHeader.NumberOfSections
        movzx        ecx,cx
        assume        edx:ptr IMAGE\_SECTION\_HEADER
        .while        ecx > 0
                .if        ebx >= [edx].VirtualAddress
                        mov        eax,[edx].VirtualAddress
                        add        eax,[edx].SizeOfRawData
                        .if        ebx < eax
                                mov        edi,@lpCodeScnInfo
                                assume        edi:ptr SCN\_INFO
                                push        [edx].VirtualAddress
                                pop        [edi].dwVirtualAddress
                                push        [edx].Misc.VirtualSize
                                pop        [edi].dwVirtualSize
                                push        [edx].SizeOfRawData
                                pop        [edi].dwRawSize
                                push        [edx].Characteristics
                                pop        [edi].dwCharacteristics
                                mov        eax,[edx].PointerToRawData
                                mov        [edi].dwRawAddress,eax
                                add        eax,lpFileHeader
                                mov        [edi].lpScn,eax
                                mov        esi,edx
                                mov        ecx,8
                                rep        movsb
                                mov        eax,TRUE
                                ret
                        .endif
                .endif
                add        edx,sizeof IMAGE\_SECTION\_HEADER
                dec        ecx
        .endw
        assume        esi:nothing
        assume        edi:nothing
        assume        edx:nothing
        mov        eax,FALSE
        ret
GetCodeScnInfo        endp

; 偷取原始入口点的部分指令，经变形后存储在指定区域，清除原来的指令，返回值为结束偷取后第一条指令的RVA
ObfuscateOEP        proc        uses ecx esi edi @lpSubstitution:dword
        mov        esi,lpPeHeader
        mov        edi,@lpSubstitution
        assume        esi:ptr IMAGE\_NT\_HEADERS
        mov        eax,[esi].OptionalHeader.AddressOfEntryPoint
        assume        esi:nothing
        invoke        RVA2Offset,eax
        add        eax,lpFileHeader
        mov        esi,eax
        mov        eax,[esi]
        .while        al != 0E8h && ax != 15FFh && ax != 25FFh &&\
                al != 74h && al != 75h && al != 76h && al != 77h && al != 0E9h && al != 0EBh
                .if        al == 55h                ;push ebp
                        mov        eax,4EC8303h        ;sub esp,4        //3 bytes
                        stosd
                        mov        eax,242C8903h        ;mov [esp],ebp        //3 bytes
                        stosd
                        xor        al,al
                        mov        [esi],al
                        inc        esi
                .elseif        ax == 0EC8Bh                ;mov ebp,esp
                        mov        ax,5401h        ;push esp        //1 byte
                        stosw
                        mov        ax,5D01h        ;pop ebp        //1 byte
                        stosw
                        xor        ax,ax
                        mov        [esi],ax
                        add        esi,2
                .elseif        al == 6Ah                ;push byte        //2 bytes
                        mov        byte ptr [edi],2
                        inc        edi
                        movsw
                        xor        ax,ax
                        mov        [esi-2],ax
                .elseif        al == 68h                ;push dword
                        mov        ax,5001h        ;push eax        //1 byte
                        stosw
                        mov        eax,2404C707h        ;mov dword ptr [esp],XXXXXXXXX        //7 bytes
```
