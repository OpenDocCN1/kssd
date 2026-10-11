
```
    >       cmp al,00h  
    >       jz endofbuffer  
    >       cmp al,60h  
    >       jc notinlowerfield  
    >       cmp al,7bh  
    >       jnc notinlowerfield  
    >       sub al,20h  
    > notinlowerfield:  
    >       stos byte ptr [edi]//EDI  
    >   loop __makeupperloop  
    > endofbuffer:  
    >   pop edi  
    >   pop esi  
    >   pop eax  
    >   pop ecx  
    >   pop ebp  
    >   retn 08h  
    > //--------------------------------------------------------  
    > //========================================================  
    > //------------- load necessary api functions -------------  
    > //以下都是重定位之后的地址  
    > _api_load:  
    >   //取得_p_szKernel32 字符串地址  
    >   lea edi,[ebp+_p_szKernel32]  
    >   //取得_p_GetModuleHandle 的地址  
    >   lea ebx,[ebp+_p_GetModuleHandle]  
    >   //取得_jmp_GetModuleHandle 地址  
    >   lea ecx,[ebp+_jmp_GetModuleHandle]  
    >   //取得_jmp_GetModuleFileName 的地址？  
    >   add ecx,02h  
    > _api_get_lib_address_loop:  
    >       //保存 ECX  
    >       push ecx  
    >       push edi  
    >       mov eax,offset _p_LoadLibrary  
    >       //加载 kernel32.dll  
    >       call [ebp+eax]// LoadLibrary(lpLibFileName);  
    >       //-------------------  
    >       pop ecx  
    >       //kernel32.dll 模块地址  
    >       mov esi,eax    // esi -> hModule  
    >       //字符串长度  
    >       push edi  
    >       call __strlen  
    >       //平衡堆栈,弹出参数  
    >       add esp,04h  
    >       //指向下一个字符串  
    >       add edi,eax  
    > _api_get_proc_address_loop:  
    >           push ecx  
    >           //edi=getmodulhanda  
    >           push edi  
    >           //esi=hmodule  
    >           push esi  
    >           mov eax,offset _p_GetProcAddress  
    >           call [ebp+eax]// GetModuleHandle=GetProcAddress(hModule, lpProcName);  
    >           //--------------------  
    >           pop ecx  
    >           //存入 GetModuleHandle 的实际地址  
    >           mov [ebx],eax  
    >           //注意看这里！！  
    >           mov [ecx],ebx  
    >           //下一个  
    >           add ebx,04h  
    >           //下一个  
    >           add ecx,06h  
    >           //下一个  
    >           push edi  
    >           call __strlen  
    >           add esp,04h  
    >           add edi,eax  
    >           mov al,byte ptr [edi]  
    >       test al,al  
    >       //不为 0 继续循环  
    >       jnz _api_get_proc_address_loop  
    >       //edi 地址加一  
    >       inc edi  
    >       mov al,byte ptr [edi]  
    >   test al,al  
    >   //不为 0 则载入 user32.dll  
    >   jnz _api_get_lib_address_loop  
    >   ret  
    > //----------------------------------------------------------  
    > //--------------------------------------------------------  
    > //========================================================  
    > // -------- exception handler expression filter ----------  
    > _except_handler1_OEP_Jump:  
    >   push ebp  
    >   mov ebp,esp  
    >   mov eax,[ebp+010h]    // PCONTEXT: pContext <- eax  
    >   //---------------  
    >   push edi  
    >   // restore original SEH  
    >   mov edi,[eax+0C4h]    // pContext.Esp  
    >   push dword ptr ds:[edi]  
    >   pop dword ptr fs:[0]  
    >   add dword ptr [eax+0C4h],8    // pContext.Esp  
    >   
    >   // set the Eip to the OEP  
    >   mov edi,[eax+0A4h] // edi <- pContext.Ebx  
    >   mov [eax+0B8h],edi // pContext.Eip <- edi  
    >   //  
    >   pop edi  
    >   //---------------  
    >   mov eax, EXCEPTION_CONTINUE_SEARCH  
    >   leave  
    >   ret  
    > //--------------------------------------------------------  
    > //========================================================  
    >   dword_type(DYN_LOADER_START_DATA1)  
    > //0xcccccccc 自填充----------------------------------  
    > _p_dwFileType:                    dword_type(0xCCCCCCCC)  
    > _p_dwImageBase:                    dword_type(0xCCCCCCCC)  
    > _p_dwOrgEntryPoint:                dword_type(0xCCCCCCCC)  
    > _p_dwImportVirtualAddress:        dword_type(0xCCCCCCCC)  
    > _p_dwRelocationVirtualAddress:    dword_type(0xCCCCCCCC)  
    > _p_dwRelocationSize:            dword_type(0xCCCCCCCC)  
    > //----------------------------------  
    > _tls_dwStartAddressOfRawData:    dword_type(0xCCCCCCCC)  
    > _tls_dwEndAddressOfRawData:        dword_type(0xCCCCCCCC)  
    > _tls_dwAddressOfIndex:            dword_type(0xCCCCCCCC)  
    > _tls_dwAddressOfCallBacks:        dword_type(0xCCCCCCCC)  
    > _tls_dwSizeOfZeroFill:            dword_type(0xCCCCCCCC)  
    > _tls_dwCharacteristics:            dword_type(0xCCCCCCCC)  
    > //----------------------------------  
    > _p_szKernel32:                //db "Kernel32.dll",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szGetModuleHandle:        //db "GetModuleHandleA",0,17  
    >       db db db db db db db db db db db db db db db db db  
    > _p_szVirtualProtect:        //db "VirtualProtect",0,15  
    >       db db db db db db db db db db db db db db db  
    > _p_szGetModuleFileName:    //db "GetModuleFileNameA",0,19  
    >       db db db db    db db db db    db db db db db db db db db db  
    > _p_szCreateFile:            //db "CreateFileA",0,12  
    >       db db db db db db db db db db db db  
    > _p_szGlobalAlloc:            //db "GlobalAlloc",0,12  
    >       db db db db db db db db db db db db  
    > _p_szVirtualAlloc:            //db "VirtualAlloc",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szLoadLibrary:            //db "LoadLibraryA",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szGetProcAddress:        //db "GetProcAddress",0,15  
    >       db db db db db db db db db db db db db db db  
    >       byte_type(0)  
    >   
    > _p_szUser32:                //db "User32.dll",0,11  
    >       db db db db db db db db db db db  
    > _p_szMessageBox:            //db "MessageBoxA",0,12  
    >       db db db db db db db db db db db db  
    >       byte_type(0)  
    >       byte_type(0)  
    > //----------------------------------  
    > _p_LoadLibrary:                    dword_type(0xCCCCCCCC)  
    > _p_GetProcAddress:                dword_type(0xCCCCCCCC)  
    > _p_GetModuleHandle:  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >   
    > _jmp_GetModuleHandle:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_VirtualProtect:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_GetModuleFileName:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_CreateFile:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_GlobalAlloc:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_VirtualAlloc:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_LoadLibrary:                __jmp_api    dword_type(0xCCCCCCCC)  
```
