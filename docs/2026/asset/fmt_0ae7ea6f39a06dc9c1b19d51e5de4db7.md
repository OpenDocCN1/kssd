
```assembly
                                [edi].Misc.VirtualSize,[edi].PointerToRawData,\
                                [edi].SizeOfRawData,[edi].Characteristics

                        invoke        SendMessage,hListBox,LB_ADDSTRING,0,addr @szBuffer

                        add        edi,SectionHeaderSize

                        dec        @temp

                        jne        _loop

                        popad

                ret

_getSectionView endp
```

;********************************************************************************

;获取第一个节区和最后一个节区偏移地址，分别返回到`baseCode`, `lpLastSection`

;********************************************************************************

```assembly
_getSectionVaddr        proc

                        LOCAL        @temp

                        nop

                        push        esi

                        mov        esi,lpPeHeader

                        assume        esi:ptr IMAGE_NT_HEADERS

                        mov        cx,[esi].FileHeader.NumberOfSections

                        movzx        ecx,cx

                        add        esi,sizeof IMAGE_NT_HEADERS

                        assume        esi:ptr IMAGE_SECTION_HEADER

                        push        [esi].VirtualAddress

                        pop        baseCode

                        mov        eax,sizeof IMAGE_SECTION_HEADER

                        dec        ecx

                        mul        ecx

                        add        esi,eax

                        mov        eax,[esi].VirtualAddress

                        add        eax,[esi].Misc.VirtualSize

                        mov        lpLastSection,eax

                        assume         esi:nothing

                        pop        esi

                ret

_getSectionVaddr endp
```

;********************************************************************************

;添加新的区段

;********************************************************************************

```assembly
_addNewSection        proc        _lpFileName,_lpSectionName

                        LOCAL        @hfile,@filesize,@hfileMap

                        LOCAL        _dwFileAlign,_dwSectionAlign

                        LOCAL         @dwSec,@dwFile

                        pushad

                        invoke        CreateFile,_lpFileName,GENERIC_WRITE+GENERIC_READ,FILE_SHARE_WRITE+FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_ARCHIVE,NULL

                        mov        @hfile,eax

                        invoke        GetFileSize,@hfile,NULL

                        mov        @filesize,eax

                        add        eax,APPEND_SIZE

                        invoke        CreateFileMapping,@hfile,NULL,PAGE_READWRITE,0,eax,0

                        mov        @hfileMap,eax

                        invoke        MapViewOfFile,eax,FILE_MAP_WRITE+FILE_MAP_COPY,0,0,0

                        mov        esi,eax

                        assume        esi:ptr IMAGE_DOS_HEADER

                        add        esi,[esi].e_lfanew

                        assume        esi:ptr IMAGE_NT_HEADERS

                        xor        ecx,ecx

                        mov        cx,word ptr[esi].FileHeader.NumberOfSections        ;区段数目

                        movzx        ecx,cx

                        push        esi

                        inc        word ptr[esi].FileHeader.NumberOfSections

                        push        [esi].OptionalHeader.FileAlignment                ;文件偏移大小

                        pop        _dwFileAlign

                        push        [esi].OptionalHeader.SectionAlignment                ;内存偏移大小

                        pop        _dwSectionAlign

                        add        esi,sizeof IMAGE_NT_HEADERS

                        assume        esi:ptr IMAGE_SECTION_HEADER

                        mov        eax,sizeof IMAGE_SECTION_HEADER

                        dec        ecx

                        mul        ecx

                        add        esi,eax

                        assume        esi:ptr IMAGE_SECTION_HEADER

                        invoke        _Align,_dwSectionAlign,[esi].Misc.VirtualSize        ;虚拟内存大小

                        add        eax,[esi].VirtualAddress                        ;虚拟内存偏移

                        mov        @dwSec,eax

                        mov        edx,[esi].PointerToRawData                        ;物理内存偏移

                        add        edx,[esi].SizeOfRawData                                ;物理内存大小

                        mov        @dwFile,edx

                        add        esi,sizeof IMAGE_SECTION_HEADER

                        assume        esi:ptr IMAGE_SECTION_HEADER

                        push        0E0000020h                                        ;可读可写可执行属性

                        pop        [esi].Characteristics

                        push        @dwSec

                        pop        [esi].VirtualAddress

                        mov        [esi].PointerToRawData,edx

                        invoke        _Align,_dwFileAlign,dwSectionSize                ;按文件偏移大小对齐文件

                        push        eax

                        pop        [esi].SizeOfRawData

                        mov        [esi].Misc.VirtualSize,eax

                        lea        edi,[esi].Name1                                        ;区段名

                        mov        esi,_lpSectionName

                        invoke        lstrlen,esi

                        mov        ecx,eax

                        cld

                        rep        movsb

                        pop        esi

                        assume        esi:ptr IMAGE_NT_HEADERS

                        mov        eax,_dwSectionAlign

                        add        eax,@dwSec

                        mov        [esi].OptionalHeader.SizeOfImage,eax                ;修改镜像大小

                        assume        esi:nothing

                        invoke        CloseHandle,@hfileMap

                        invoke        CloseHandle,@hfile

                        popad

                ret

_addNewSection endp
```

附件：

`[PE_Tools.zip](https://bbs.kanxue.com/attach-download-85445.htm)`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [4Z@CMOHJO75SV2F3C_4JKVI.jpg](javascript:void(0);)

  （306.80kb，6次下载）

- [%J71G`{XB~6R4OGI703}}V9.jpg](javascript:void(0);)

  （288.89kb，7次下载）

- [~MEXLW)C]X7V4HPNECZZ[N4.jpg](javascript:void(0);)

  （71.19kb，6次下载）

- [PE_Tools.zip](javascript:void(0);)

  （344.23kb，98次下载）
