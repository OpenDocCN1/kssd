

# Api32 V2.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50364.htm`](https://www.pediy.com/kssd/pediy05/pediy50364.htm)

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Api32 V2.5 KeyMaker for fun.asm \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;by: ArchFire/ATA @2002-12-14   
;\*   
;\*   
;\* in this case, we will learn RSA algorithm.   
;\*   
;\* as we know that Vitaly Evseenko, the author of Apis32, once been a RCEer. So he shows how to use RAS in his little proggy   
;\* oh, well, it's not so hard, so just find what's happening....   
;\* you'd better have look on PEDiy's BBS Seletion III for more help, thx goes to those who have witten the good tutors :)   
;\*   
;\* IN RSA:   
;\* 1. Find two prime number: p, q. the larger the better   
;\* 2. n=p\*q, f=(p-1)\*(q-1)   
;\* 3. Select a random number e, which is satisfied: GCD(f, e)=1. Often, we choose 7, 13, 65537....   
;\* 4. Find d, where: (d\*e) mod f=1   
;\* 5. Encipher proc: C=(M^e) mod n   
;\*    Decipher proc: M=(C^d) mod n   
;\*   
;\* In Apis32:   
;\* C=((M⁷) mod 8899) mod 0bb=(M⁷) mod 0bb, 'cause 8899 mod 0bb=0   
;\* n=0bb=11\*0b, f=10\*0a=0a0, e=7, d=17   
;\* so, Decipher proc: M=(C¹⁷) mod 0bb   
;\* Learn and enjoy!   
;\*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
.386   
.model flat,stdcall   
option casemap:none   
include hd.h   
DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD   
DeRSA proto :DWORD, :DWORD   
  
  
        .const   
BUFF\_SIZE      equ    32   
  
ID\_MAKE        equ    1002   
ID\_ABOUT        equ    1003   
ID\_CLOSE        equ    1004   
IDC\_NAME    equ    1010   
IDC\_CODE        equ    1011   
OURICON        equ    1020   
DLG\_MAIN        equ    1000   
  
        .data   
szC        db      BUFF\_SIZE dup (0)   
szM        db      BUFF\_SIZE dup (0)   
szTemp          db      BUFF\_SIZE dup (0)   
szFormat    db    "%02X", 0   
MsgTitle        db    "�ATA�KeyGen", 0   
MsgContend      db    "Apis32 2.50 KeyGen for fun", 0dh, 0ah, 0dh, 0ah   
                db      "    by ArchFire/ATA", 0   
  
szInputError    db      "Input 8 chars please...", 0   
  
  
        .data?   
hInstance  HANDLE ?   
  
        .code   
    .RADIX 16   
start:   
    invoke    GetModuleHandle, NULL   
    mov      hInstance,eax   
    invoke    DialogBoxParam,hInstance,DLG\_MAIN,NULL,offset DlgProc,0   
    invoke    ExitProcess,NULL   
  
DlgProc    proc    uses ebx edi esi, \\   
        hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD   
  
        LOCAL Ps :PAINTSTRUCT   
  
        mov    eax,wMsg   
  
    .if    eax == WM\_CLOSE   
        invoke    EndDialog,hWnd,NULL   
  
        .elseif eax==WM\_INITDIALOG   
        invoke LoadIconA, hInstance, OURICON            ;note: use "hInstance" instead of "hWnd"; if "dword ptr OURICON" -> PUSH WORD OURICON, wrong result   
        test eax, eax   
        je initerror   
        push edi   
        mov edi, eax   
        invoke SendMessageA, hWnd, WM\_SETICON, ICON\_BIG, eax   
        invoke SendMessageA, hWnd, WM\_SETICON, ICON\_SMALL, edi   
        pop edi   
        initerror:   
        nop   
  
    .elseif eax == WM\_PAINT   
                invoke BeginPaint,hWnd,ADDR Ps   
                invoke FrameWindow,hWnd,0,1,1   
                invoke FrameWindow,hWnd,1,1,0   
                invoke EndPaint,hWnd,ADDR Ps   
                xor eax, eax   
  
    .elseif    eax == WM\_COMMAND   
          mov eax,wParam   
          .IF lParam!=0   
            .if ax==ID\_MAKE   
                invoke RtlZeroMemory, addr szC, BUFF\_SIZE   
                invoke RtlZeroMemory, addr szM, BUFF\_SIZE   
                invoke GetDlgItemText,hWnd,IDC\_NAME,addr szM, BUFF\_SIZE   
                .if eax < 8   
                    invoke SetDlgItemText,hWnd,IDC\_NAME,addr szInputError   
  
                .else   
                    lea esi, szC   
                    lea edi, szM   
                    invoke DeRSA, edi, esi   
              .endif   
              invoke SetDlgItemText,hWnd,IDC\_CODE,addr szC   
  
            .elseif ax==ID\_CLOSE   
                invoke    EndDialog,hWnd,NULL   
            .elseif ax==ID\_ABOUT   
                invoke MessageBox, hWnd, Addr MsgContend, Addr MsgTitle, MB\_OK   
            .endif   
  
          .ENDIF   
        .else   
        mov eax,FALSE   
        ret   
          .endif   
        mov eax,TRUE   
        ret   
DlgProc    ENDP   
  
  
DeRSA proc szName:DWORD, szCode:DWORD   
      
    mov esi, szName   
    push ebp   
    xor ebp, ebp   
    push 0bb   
    pop ebx   
    ;   
@loop2:   
    mov edi, 16                ;hmmm, when edi=17h, we get a wrong result   
    movzx eax, byte ptr \[esi+ebp\]   
    mov ecx, eax   
@loop1:   
    imul ecx   
    cmp eax, ebx   
    jl @next2   
    idiv ebx   
    mov eax, edx   
@next2:   
    dec edi   
    jnz @loop1   
  
    add ebp, 50   
    xor ax, bp   
    sub ebp, 50   
  
    mov \[esi+ebp\], al   
    inc ebp   
    cmp ebp, 8   
    jl @loop2   
  
    pop ebp   
  
    mov edi, szCode   
    xor ebx, ebx   
p\_loop:   
    xor eax, eax   
    lodsb   
    invoke wsprintf, addr szTemp, addr szFormat, eax   
    cmp bl, 4   
    jnz @f   
    mov al, '-'   
    stosb   
@@:   
    mov ax, word ptr \[szTemp+0\]   
    stosw   
    inc ebx   
    cmp ebx, 08   
    jl p\_loop   
    ret   
      
  
DeRSA endp   
  
end start  4dd466d0e1e0cc123afbe504da2462ca