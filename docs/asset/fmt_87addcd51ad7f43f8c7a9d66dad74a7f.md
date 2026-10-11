
```
push ecx ; dwBytes  
push 8 ; dwFlags  
push ds:hHeap ; hHeap  
call HeapAlloc  
mov ds:lpMem, eax  

push 0A0000h ; dwBytes  
push 8 ; dwFlags  
push ds:hHeap ; hHeap  
call HeapAlloc  
mov ds:dword_4038A7, eax  

pop ecx  
cmp ecx, 50h  
jbe short loc_401CC1  
mov ds:dword_403786, esi  
mov ds:dword_4038B3, ecx  
push offset loc_40201C  
push ds:dword_4038A7  
push ecx  
push ds:lpMem  
push esi  
call _aP_pack  
mov ds:dword_4037AB, eax  
or eax, eax  
jz short loc_401CC1  

pop edi  
pop ecx  
mov esi, ds:lpMem  
rep movsb  
jmp short @Clean  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

loc_401CC1:        ; CODE XREF: CompressIt+31j  
       ; CompressIt+5Ej  
pop edi  
pop ecx  
mov ds:dword_4037AB, eax  

@Clean:       ; CODE XREF: CompressIt+6Aj  
push ds:lpMem ; lpMem  
push 0 ; dwFlags  
push ds:hHeap ; hHeap  
call HeapFree  
push ds:dword_4038A7 ; lpMem  
push 0 ; dwFlags  
push ds:hHeap ; hHeap  
call HeapFree  
mov eax, ds:dword_4037AB  
retn  

CompressIt endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401CF4 proc near ; CODE XREF: CompressPE+2B5p  
       ; sub_402050+Ep  
push esi  
cmp dword ptr [eax+14h], 0  
jz short @SkipThisSection  
cmp dword ptr [eax+10h], 0  
jz short @SkipThisSection  
cmp dword ptr [eax+8], 50h  
jbe short @SkipThisSection  
call sub_401D24  
jb short @SkipThisSection  
mov esi, offset szSkipedSectionName ; ".tls.relBSS"  

loc_401D13:        ; CODE XREF: sub_401CF4+28j  
xchg eax, ebx  
lodsd  
xchg eax, ebx  
cmp ebx, [eax]  
jz short @SkipThisSection  
or ebx, ebx  
jnz short loc_401D13  

pop esi  
clc  
retn  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

@SkipThisSection:   ; CODE XREF: sub_401CF4+5j  
       ; sub_401CF4+Bj ...  
pop esi  
stc  
retn  

sub_401CF4 endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401D24 proc near ; CODE XREF: sub_401CF4+13p  
pusha  
mov edx, ds:dword_4038C4  
cmp [eax+0Ch], edx  
jz short loc_401D51  
mov edx, ds:dword_40379F  
or edx, edx  
jz short loc_401D4E  
mov ebx, [eax+0Ch]  
cmp ebx, edx  
jg short loc_401D4E  
jz short loc_401D51  
mov ebx, [eax+34h]  
or ebx, ebx  
jz short @SkipThisSection  
cmp ebx, edx  
jg short loc_401D51  

loc_401D4E:        ; CODE XREF: sub_401D24+14j  
       ; sub_401D24+1Bj  
popa  
clc  
retn  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

loc_401D51:        ; CODE XREF: sub_401D24+Aj  
       ; sub_401D24+1Dj ...  
popa  
stc  
retn  

sub_401D24 endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

GetPEHeader proc near ; CODE XREF: CompressPE+74p  
       ; CompressPE+D1p ...  
mov esi, [eax+3Ch]  
add esi, eax  
mov ds:lpPEHeader, esi  
retn  

GetPEHeader endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401D60 proc near ; CODE XREF: CompressPE+5Ep  
       ; CompressPE+C6p ...  
push ecx  
push offset FileSizeHigh ; lpFileSizeHigh  
push ds:hFile ; hFile  
call GetFileSize  
call sub_401DBE  
jnb short loc_401D7B  

pop ecx  
jmp short locret_401DBD  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

loc_401D7B:        ; CODE XREF: sub_401D60+16j  
mov ds:dword_40375E, eax  
pop ecx  
add eax, ecx  
push 0 ; lpName  
push eax ; dwMaximumSizeLow  
push 0 ; dwMaximumSizeHigh  
push 4 ; flProtect  
push 0 ; lpFileMappingAttributes  
push ds:hFile ; hFile  
call CreateFileMappingA  
call sub_401DBE  
jb short locret_401DBD  

mov ds:hObject, eax  
push 0 ; dwNumberOfBytesToMap  
push 0 ; dwFileOffsetLow  
push 0 ; dwFileOffsetHigh  
push 2 ; dwDesiredAccess  
push eax ; hFileMappingObject  
call MapViewOfFile  
call sub_401DBE  
jb short locret_401DBD  

mov ds:lpBaseAddress, eax  

locret_401DBD:        ; CODE XREF: sub_401D60+19j  
       ; sub_401D60+3Cj ...  
retn  

sub_401D60 endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401DBE proc near ; CODE XREF: CompressPE+2Ep  
       ; sub_401D60+11p ...  
cmp eax, -1  
jz short loc_401DC9  

or eax, eax  
jz short loc_401DC9  

clc  
retn  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

loc_401DC9:        ; CODE XREF: sub_401DBE+3j  
       ; sub_401DBE+7j  
stc  
retn  

sub_401DBE endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

FreeFile proc near ; CODE XREF: CompressPE+BCp  
       ; CompressPE+545p ...  
push ds:lpBaseAddress ; lpBaseAddress  
call UnmapViewOfFile  
push ds:hObject ; hObject  
call CloseHandle  
retn  

FreeFile endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

GetMaxPath proc near ; CODE XREF: CompressPE+365p  
       ; CompressPE+381p ...  
mov ecx, 200h  

GetMaxPath endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401DE7 proc near ; CODE XREF: CompressPE+47Fp  
       ; sub_401E09+78p ...  
xor edx, edx  
div ecx  
or edx, edx  
jz short loc_401DF0  

inc eax  

loc_401DF0:        ; CODE XREF: sub_401DE7+6j  
mul ecx  
retn  

sub_401DE7 endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

; HFILE __stdcall MyOpenFile(LPCSTR lpFileName,LPOFSTRUCT lpReOpenBuff,UINT uStyle)  
MyOpenFile proc near ; CODE XREF: CompressPE+29p  
push 0 ; hTemplateFile  
push 20h ; dwFlagsAndAttributes  
push 3 ; dwCreationDisposition  
push 0 ; lpSecurityAttributes  
push 0 ; dwShareMode  
push 0C0000000h ; dwDesiredAccess  
push esi ; lpFileName  
call CreateFileA  
retn  

MyOpenFile endp  

; 〓〓〓〓〓〓〓〓 S U B R O U T I N E 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓  

sub_401E09 proc near ; CODE XREF: CompressPE+41p  
       ; CompressPE+590p  
xor ecx, ecx  
call sub_401D60  
jb @InvalidPE  

mov ecx, ds:dword_40375E  
or ecx, ecx  
jz loc_401F4E  

mov ds:dword_40377E, ecx  
call GetPEHeader  
mov eax, [esi+3Ch]  
mov ds:dword_403762, eax  
xor eax, eax  
mov ax, [esi+14h]  
add eax, 18h  
add eax, esi  
push esi  
push eax  
xchg eax, edi  
mov ds:dword_4038A3, edi  
movzx ecx, word ptr [esi+6]  
xor ebx, ebx  

loc_401E51:        ; CODE XREF: sub_401E09+52j  
call sub_401F54  
inc ebx  
cmp ebx, ecx  
jz short loc_401E5D  
jmp short loc_401E51  

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  

loc_401E5D:        ; CODE XREF: sub_401E09+50j  
pop eax  
pop esi  
pusha  
push eax  
push esi  
call sub_4015F3  
popa  
push eax  
xor edx, edx  
mov ecx, 28h  
xor eax, eax  
mov ax, [esi+6]  
inc eax  
mul ecx  
xchg eax, ebx  
pop eax  
push eax  
add eax, ebx  
mov ecx, [esi+3Ch]  
call sub_401DE7  
xchg eax, ebx  
pop eax  
mov ecx, ebx  
sub ecx, ds:lpBaseAddress  
mov [esi+54h], ecx  
mov ecx, 200h  
mov [esi+3Ch], ecx  
movzx ecx, word ptr [esi+6]  
mov edi, ebx  
```
