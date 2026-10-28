
```asm
;invoke WriteApi,WProcess,Papi1, addr ApiBak1 ,8 ;先还原API

;invoke rtcMidBstr

;invoke WriteApi,WProcess,Papi1, addr hacker ,sizeof HOOKAPI ;调用完后再改回来

MyAPI1 endp

;*******************************************************************************************
MyAPI2 proc

    pop eax

    CMP AH,0
    JE @F

    CMP DWORD PTR DS:[EAX+4h],320037h
    JNZ @F
    CMP DWORD PTR DS:[EAX+8h],300034h

    push ebx
    mov ebx,[My1]
    mov DWORD PTR DS:[EAX],ebx
    mov ebx,[My2]
    mov DWORD PTR DS:[EAX+4h],ebx
    mov ebx,[My3]
    mov DWORD PTR DS:[EAX+8h],ebx
    pop ebx

    @@:

    PUSH ESI
    MOV ESI,DWORD PTR SS:[ESP+0Ch]
    TEST ESI,ESI
    Jl @F
    jmp DWORD PTR DS:[Papi30]
    @@:
    jmp DWORD PTR DS:[Papi20]

MyAPI2 endp

;***************************************************************************写注册表模块*****************************************************************************
_SetIDMRGE Proc _dwFlag
    .if _dwFlag
        invoke _RegSetValue,addr szKeyIDM,addr szValueFName,addr szValueFNamesj,REG_SZ,100
        invoke _RegSetValue,addr szKeyIDM,addr szValueLName,addr szValueLNamesj,REG_SZ,100
        invoke _RegSetValue,addr szKeyIDM1,addr szValueEmail,addr szValueEmailsj,REG_SZ,100
        invoke _RegSetValue,addr szKeyIDM1,addr szValueSerial,addr szValueSerialsj,REG_SZ,100
    .else
        invoke _RegDelValue,addr szKeyIDM,addr szValueSerial
    .endif
    ret
_SetIDMRGE endp
;*******************************************************************************************************************

End DllEntry
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
