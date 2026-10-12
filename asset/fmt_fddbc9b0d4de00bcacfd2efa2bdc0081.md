
```
EncryptResSec proc uses ebx ecx edx edi esi pEntry : LPVOID, dwLevel : DWORD
;; level = 0,1,2

    mov esi, pEntry
    assume esi : ptr IMAGE\_RESOURCE\_DIRECTORY
    ;; 计算有几个资源在当前目录下
    mov cx, word ptr [esi].NumberOfNamedEntries
    add cx, word ptr [esi].NumberOfIdEntries
    movzx ecx, cx
    ;; 这里存在着 ecx 个 IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    add esi, sizeof IMAGE\_RESOURCE\_DIRECTORY
    mov edi, esi
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
ResDirEntryLoop:
    ;; 加密资源层
    ;; 清除加密标志,如果是 0 级别则清除
    cmp dwLevel, 0
    jnz SkipClearEncryptFlag
    push 0
    pop g\_dwEncrypt
    SkipClearEncryptFlag:
    invoke JudegeResourceType, edi, dwLevel
    mov eax, dword ptr [edi].OffsetToData
    test eax, FIRSTBIT
    jz FoundTheDataEntry
    and eax, 0000FFFFh
    add eax, g\_pResSection
    mov edx, dwLevel
    inc edx
    invoke EncryptResSec, eax, edx
    jmp ContinueResDirEntryLoop
    FoundTheDataEntry:
    ;; 加密资源文件
    mov ebx, g\_dwEncrypt
    test ebx, ebx
    jz ContinueResDirEntryLoop
    
    add eax, g\_pResSection
    assume eax : ptr IMAGE\_RESOURCE\_DATA\_ENTRY
    mov edx, dword ptr [eax].Size1
    mov eax, dword ptr [eax].OffsetToData
    ;; 转化 RVA 到偏移
    invoke RVA2Offset, g\_pMem, eax
    add eax, g\_pMem
    
    ;; 加密资源
    EncryptResourceData:
    push edi
    push esi
    push ecx
    mov edi, eax
    mov esi, edi
    mov ecx, edx     
    cld
    EnResourceLoop:
    lodsb
    xor al, CRYPT\_KEY
    stosb
    loop EnResourceLoop     
    pop ecx
    pop esi
    pop edi
    ContinueResDirEntryLoop:
    add edi, sizeof IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    dec ecx
    test ecx, ecx
    jnz ResDirEntryLoop
    
    
    assume esi : nothing
    assume edi : nothing
    ret     
EncryptResSec endp

JudegeResourceType proc uses eax edi pResEntry : LPVOID, dwLevel : DWORD
    
    ;; 不加密 ICON,ICON\_GROUP,还有版本信息
    ;; 在 PE 加载器加载时要读取这些资源
    ;; 否则程序会崩溃     
    
    mov edi, pResEntry
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    mov eax, dwLevel       
    ;; 不是类型层,直接退出 
    cmp eax, 0
    jnz ExitJudgeLevel
    ;; 显示类型
    mov eax, dword ptr [edi].Name1
    cmp eax, RT\_ICON
    jnz CmpICON\_GROUP
    jmp ExitJudgeLevel
CmpICON\_GROUP:
    cmp eax, RT\_GROUP\_ICON
    jnz CmpVERSION 
    jmp ExitJudgeLevel
CmpVERSION:
    cmp eax, RT\_VERSION
    jnz EncryptData
         
ExitJudgeLevel:
    assume eax : nothing
    assume edi : nothing
    ret

EncryptData:
    push 1
    pop g\_dwEncrypt
    jmp ExitJudgeLevel

JudegeResourceType endp
```

解密代码如下:

代码:
