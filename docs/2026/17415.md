
# [原创]快速高效的写shellcode

自己写shellcode的一点小经验技巧...见笑了~

```
	format PE GUI 4.0

	include '%include%\win32ax.inc'

c fix stdcall

macro MakeHash [apiname]
{ 
	local ch,length,hash
	virtual at 0 
		db `apiname
		length = $
		hash = 0
		repeat length
			load ch byte from %-1
			hash = ( hash + ch - 1 )
			hash = (((hash shl 7) and 0xFFFFFFFF) or (hash shr (32-7))) 	;hash = hash rol 7
		end repeat
	end virtual
	jmp2#apiname db 0xE9
	dd hash
}

macro i proc,[arg]
 { common
    if ~ arg eq
   reverse
     pushd arg
   common
    end if
    if defined jmp2#proc
    call jmp2#proc
    else if defined jmp2#proc#A
    call jmp2#proc#A
    else
   call [proc]
   end if
}

entry $
;得到Kernel基址
	xor ecx,ecx
	mov esi,[fs:ecx]
	lodsd
	lodsd
@@:
	dec eax
	xor ax,ax
	cmp word[eax],'MZ'
	jne @B
;得到LoadLibraryA的地址
	c GetProcAddr,eax,0xDECD7FBF
	xchg eax,ebx
;得到各个Api的地址 并构造IAT 代码中需要使用API 将可以直接call,而不要再去重定位.
;我说的"快速高效"也就是指的这一段的使用.![](https://bbs.kanxue.com/view/img/face/12.gif) 
	call @F
	db 'user32.dll',0
	MakeHash MessageBoxA
	db 'kernel32.dll',0
	MakeHash ExitProcess,CreateThread,ExitThread
@@:	
	pop esi
play:
	cmp byte[esi],0x5E     ;0x5E   pop esi
	je over
	cmp byte[esi],0xE9     ;这个地方不明白为什么是0xE9可以看上面的MakeHash宏
	je getproc
	c ebx,esi
	xchg eax,edi
@@:
	lodsb
	cmp al,0
	jne @B
	jmp play

getproc:
	inc esi
	c GetProcAddr,edi,dword[esi]
	sub eax,esi
	sub eax,4
	mov dword[esi],eax
	lodsd                     ;add esi,4
	jmp play
over:
	xor edi,edi
	push edi edi edi
	call @F                   ;
	jmp MsgBox           ;这个地方有点好玩.小技巧可以多用~
	@@:                      ;
	push edi edi
	i CreateThread
	i Sleep,1000*5
	jmp jmp2ExitProcess

MsgBox:
	i MessageBox,NULL,'Hello World !','About',MB_OK
	jmp jmp2ExitThread
	
;下面是普通的Hash查找Getproceaddress.就不说了~
proc GetProcAddr; hDll:DWORD,dwApi:DWORD
	;ecx计数，ebx基址,edi指针
	pushad
	mov ebx,[esp+4*(1+8)]
	mov edi,[ebx+0x3C]			;edi->PE addr
	mov edi,[edi+ebx+0x78]	;edi->addr IMAGE_EXPORT_DIRECTORY
	add edi,ebx
	mov ecx,[edi+IMAGE_EXPORT_DIRECTORY.NumberOfNames]
	mov edx,[edi+IMAGE_EXPORT_DIRECTORY.AddressOfNames]
	add edx,ebx
.play:
	dec ecx
	push edx
	mov esi,[edx+ecx*4]
	add esi,ebx
	
	;CalcHash
	xor edx,edx
@@:
	lodsb
	test al,al
	je @F
	movzx eax,al
	add edx,eax
	dec edx
	rol edx,7
	jmp @B
@@:
	xchg eax,edx

	cmp eax,[esp+4*(2+8+1)]
	pop edx
	jne .play
	mov eax,[edi+0x24]
	add eax,ebx
	mov ecx,[eax+ecx*2]
	movzx ecx,cx
	
	mov eax,[edi+0x1C]
	add eax,ebx
	mov eax,[eax+ecx*4]
	
	add eax,ebx
	mov dword[esp+4*7],eax   ;这个地方可以看一下~
	popad
	retn 4*2
endp
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
