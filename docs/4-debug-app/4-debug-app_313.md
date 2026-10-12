
# 分区序列号

> 原文：[`www.pediy.com/kssd/pediy04/kanxue416.htm`](https://www.pediy.com/kssd/pediy04/kanxue416.htm)

```asm
CSEG  SEGMENT   
    ASSUME CS:CSEG,DS:CSEG,ES:CSEG   
    ORG 100H   

START:   
    mov ah,9   
    mov dx,offset MsgInputDisk   
    int 21h   
    mov ah,1   
    int 21h   
    or al,20h   
    cmp al,'a'   
    jb ErrorInput   
    cmp al,'z'   
    ja ErrorInput   
    jmp GetDiskOrder   

GetDiskOrder:   
    sub al,'a'   
    inc al   
    mov bl,al   
    mov bh,0   
    push bx          ;Push Disk Number   
    mov dx,offset DiskInfo   
    mov ax,6900h   
    int 21h   
    jnc GetDiskInfoOK   
    jmp AccessError   

ErrorInput:   
    mov dx,offset MsgErrorInput   
    jmp Exit   

GetDiskInfoOK:   
    mov bx,4   
    mov cl,4   
    mov si,offset LongDiskSN-1   
    mov di,offset lpSerialNum   

NextByte:   
    mov al,[si+bx]   
    mov ah,al   
    shr al,cl   

Bin2Asc:   
    and al,0fh   
    add al,'0'   
    cmp al,'9'   
    jna StoreAsc   
    add al,7   

StoreAsc:   
    stosb   
    mov al,ah   
    add bh,80h   
    jnz Bin2Asc   
    dec bl   
    jnz NextByte   
    mov ah,9   
    mov dx,offset MsgDiskSN   
    int 21h   
    mov ah,9   
    mov dx,offset MsgInputDisk   
    int 21h   
    mov ah,0ah   
    mov dx,offset InputBuffer   
    int 21h   
    cmp byte ptr InputBuffer[1],8   
    jne ErrorInput   
    mov si,offset InputedDiskSN   
    mov cx,8   
    mov di,offset LongDiskSN+3   
    mov ah,0   

Asc2Bin:   
    lodsb   
    cmp al,'0'   
    jb ErrorInput   
    cmp al,'9'   
    ja A2F   
    sub al,'0'   
    jmp StoreBin1   

A2F:   
    or al,20h   
    cmp al,'a'   
    jb ErrorInput   
    cmp al,'f'   
    ja ErrorInput   
    sub al,'0'+7   

StoreBin1:   
    add ah,80h   
    je StoreBin2   
    mov bl,al   
    shl bl,1   
    shl bl,1   
    shl bl,1   
    shl bl,1   
    loop Asc2Bin   

StoreBin2:   
    or al,bl   
    stosb   
    dec di   
    dec di   
    loop Asc2Bin   
    pop bx   
    mov ax,6901h   
    mov dx,offset DiskInfo   
    int 21h   
    jnc OK   

AccessError:   
    mov dx,offset MsgAccessError   
    jmp Exit   

OK:   
    mov dx,offset MsgOK   

Exit:   
    mov ah,9   
    int 21h   
    MOV Ah,4CH   
    INT 21H   

MsgInputDisk  db 0dh,0ah,"Please input your driver letter:$"   
MsgInputSN    db 0dh,0ah,"Please input your new Disk Serial Number(as XXXXXXXX):"   
MsgAccessError db 0dh,0ah,"Get(Or Set) Disk SN Error!$"   
MsgErrorInput  db 0dh,0ah,"Input Error, please input a char within 'a' and 'z'! and must be 8 bytes HEX chars!$"   
MsgOK          db 0dh,0ah,"Your Disk SN Successfully changed!$"   
MsgDiskSN      db 0dh,0ah,"Your OLD disk serial number is:"   

lpSerialNum    db 8 dup (0),0dh,0ah,24h   
InputBuffer    db 80,8   

InputedDiskSN  label byte   
org offset InputBuffer+90h   

DiskInfo       label byte   
org offset DiskInfo+2   

LongDiskSN     label byte   
CSEG ends   
    END START  4dd466d0e1e0cc123afbe504da2462ca
```
