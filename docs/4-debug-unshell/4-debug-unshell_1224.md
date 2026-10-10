

# dbpe2.10 的 asm SDK

> 原文：[`www.pediy.com/kssd/pediy05/pediy50341.htm`](https://www.pediy.com/kssd/pediy05/pediy50341.htm)

;标准版的 dbpe2.10 可以改造成支持 SDK 的专业版   
;需要改造的是其 dbpe.dll 和 dbpe2.dll   
;这里不公布改造方法,只展示一下 SDK   
;有能力的自行改造吧,很有意思 PEDIY   
  
  
;   
;dbpe2.10 SDK for asm   
;   
GetUserName\_dbpe    PROTO    :DWORD   
GetHardCode\_dbpe    PROTO    :DWORD   
Md5Hash\_dbpe        PROTO    :DWORD,:DWORD   
;以下 6 个 proc 用法十分简单,与原 API 用法完全一样   
;建议调试程序时用原始 API,最后换成\_dbpe 的,再用 dbpe2.10 专业版加壳就 OK 了   
;   
CreateFile\_dbpe        PROTO    :DWORD,:DWORD,:DWORD,:DWORD,:DWORD,:DWORD,:DWORD   
ReadFile\_dbpe        PROTO    :DWORD,:DWORD,:DWORD,:DWORD,:DWORD   
WriteFile\_dbpe        PROTO    :DWORD,:DWORD,:DWORD,:DWORD,:DWORD   
SetFilePointer\_dbpe    PROTO    :DWORD,:DWORD,:DWORD,:DWORD   
CloseHandle\_dbpe    PROTO    :DWORD   
DeleteFile\_dbpe        PROTO    :DWORD   
  
  
.code   
GetUserName\_dbpe    proc    pUserName:DWORD   
  
            invoke    GetClassName,0FEFE2002h,pUserName,4   
            ret   
GetUserName\_dbpe    endp   
  
  
GetHardCode\_dbpe    proc    pHardCode:DWORD   
  
            invoke    GetClassName,0FEFE2003h,pHardCode,4   
            ret   
GetHardCode\_dbpe    endp   
  
  
Md5Hash\_dbpe        proc    pSrcData:DWORD,   
                pResMd5:DWORD   
  
        LOCAL    ClassName\[2\]:DWORD   
  
            mov    eax,pSrcData   
            mov    ClassName,eax   
            mov    eax,pResMd5   
            mov    ClassName\[4\],eax   
            invoke    GetClassName,0FEFE2004h,ADDR ClassName,8   
            ret   
Md5Hash\_dbpe        endp   
  
  
CreateFile\_dbpe        proc    pFileName:DWORD,   
                dwDesiredAccess:DWORD,   
                dwShareMode:DWORD,   
                pSecurityAttributes:DWORD,   
                dwCreationDistribution:DWORD,   
                dwFlagsAndAttributes:DWORD,   
                hTemplateFile:DWORD   
  
        LOCAL    ClassName\[7\]:DWORD   
  
            mov    eax,pFileName   
            mov    ClassName,eax   
            mov    eax,dwDesiredAccess   
            mov    ClassName\[4\],eax   
            mov    eax,dwShareMode   
            mov    ClassName\[8\],eax   
            mov    eax,pSecurityAttributes   
            mov    ClassName\[12\],eax   
            mov    eax,dwCreationDistribution   
            mov    ClassName\[16\],eax   
            mov    eax,dwFlagsAndAttributes   
            mov    ClassName\[20\],eax   
            mov    eax,hTemplateFile   
            mov    ClassName\[24\],eax   
            invoke    GetClassName,0FEFE2011h,ADDR ClassName,1Ch   
            ret   
CreateFile\_dbpe        endp   
  
  
ReadFile\_dbpe        proc    hFile:DWORD,   
                pBuffer:DWORD,   
                nNumberOfBytesToRead:DWORD,   
                pNumberOfBytesRead:DWORD,   
                pOverlapped:DWORD   
  
        LOCAL    ClassName\[5\]:DWORD   
  
            mov    eax,hFile   
            mov    ClassName,eax   
            mov    eax,pBuffer   
            mov    ClassName\[4\],eax   
            mov    eax,nNumberOfBytesToRead   
            mov    ClassName\[8\],eax   
            mov    eax,pNumberOfBytesRead   
            mov    ClassName\[12\],eax   
            mov    eax,pOverlapped   
            mov    ClassName\[16\],eax   
            invoke    GetClassName,0FEFE2012h,ADDR ClassName,14h   
            ret   
ReadFile\_dbpe        endp   
  
  
WriteFile\_dbpe        proc    hFile:DWORD,   
                pBuffer:DWORD,   
                nNumberOfBytesToWrite:DWORD,   
                pNumberOfBytesWritten:DWORD,   
                pOverlapped:DWORD   
  
        LOCAL    ClassName\[5\]:DWORD   
  
            mov    eax,hFile   
            mov    ClassName,eax   
            mov    eax,pBuffer   
            mov    ClassName\[4\],eax   
            mov    eax,nNumberOfBytesToWrite   
            mov    ClassName\[8\],eax   
            mov    eax,pNumberOfBytesWritten   
            mov    ClassName\[12\],eax   
            mov    eax,pOverlapped   
            mov    ClassName\[16\],eax   
            invoke    GetClassName,0FEFE2013h,ADDR ClassName,14h   
            ret   
WriteFile\_dbpe        endp   
  
  
SetFilePointer\_dbpe    proc    hFile:DWORD,   
                lDistanceToMove:DWORD,   
                lDistanceToMoveHigh:DWORD,   
                dwMoveMethod:DWORD   
  
        LOCAL    ClassName\[4\]:DWORD   
  
            mov    eax,hFile   
            mov    ClassName,eax   
            mov    eax,lDistanceToMove   
            mov    ClassName\[4\],eax   
            mov    eax,lDistanceToMoveHigh   
            mov    ClassName\[8\],eax   
            mov    eax,dwMoveMethod   
            mov    ClassName\[12\],eax   
            invoke    GetClassName,0FEFE2014h,ADDR ClassName,10h   
            ret   
SetFilePointer\_dbpe    endp   
  
  
CloseHandle\_dbpe    proc    hFile:DWORD,   
  
        LOCAL    ClassName:DWORD   
  
            mov    eax,hFile   
            mov    ClassName,eax   
            invoke    GetClassName,0FEFE2015h,ADDR ClassName,4   
            ret   
CloseHandle\_dbpe    endp   
  
  
DeleteFile\_dbpe        proc    pFileName:DWORD,   
  
        LOCAL    ClassName:DWORD   
  
            mov    eax,pFileName   
            mov    ClassName,eax   
            invoke    GetClassName,0FEFE2016h,ADDR ClassName,4   
            ret   
DeleteFile\_dbpe        endp  4dd466d0e1e0cc123afbe504da2462ca