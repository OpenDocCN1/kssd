
```
F000:F685                  ; ---------------------------------------------------------------------------  
F000:F685                  reinit_SCI_interrupt:       ; CODE XREF: Chipset_Reg_Early_Init+92j  
F000:F685 B9 42 3B           mov   cx, 3B42h           ; B#0 D#7 F#3: SCI Interrupt Configuration  
F000:F688 BC 8E F6           mov   sp, 0F68Eh  
F000:F68B EB 7B              jmp   short Read_PCI_Byte  
F000:F68B                  ; ---------------------------------------------------------------------------  
F000:F68D 90                 db  90h ; ?  
F000:F68E 90 F6              dw 0F690h  
F000:F690                  ; ---------------------------------------------------------------------------  
F000:F690 A8 40              test  al, 40h             ; check SUSC State  
F000:F692 74 F1              jz    reinit_SCI_interrupt  
F000:F694 EB 27              jmp   short exit  
F000:F696                  ; ---------------------------------------------------------------------------  
F000:F696                  silicon_rev_lt_10h:         ; CODE XREF: Chipset_Reg_Early_Init+41j  
F000:F696 B9 80 3B           mov   cx, 3B80h  
F000:F699 B0 00              mov   al, 0  
F000:F69B BC A1 F6           mov   sp, 0F6A1h  
F000:F69E E9 81 00           jmp   Write_PCI_Byte  
F000:F69E                  ; ---------------------------------------------------------------------------  
F000:F6A1 A3 F6              dw 0F6A3h  
F000:F6A3                  ; ---------------------------------------------------------------------------  
F000:F6A3 B9 81 3B           mov   cx, 3B81h  
F000:F6A6 B0 50              mov   al, 50h ; 'P'  
F000:F6A8 BC AE F6           mov   sp, 0F6AEh  
F000:F6AB EB 75              jmp   short Write_PCI_Byte  
F000:F6AB                  ; ---------------------------------------------------------------------------  
F000:F6AD 90                 db  90h ; ?  
F000:F6AE B0 F6              dw 0F6B0h  
F000:F6B0                  ; ---------------------------------------------------------------------------  
F000:F6B0 B9 84 3B           mov   cx, 3B84h  
F000:F6B3 B0 07              mov   al, 7  
F000:F6B5 BC BB F6           mov   sp, 0F6BBh  
F000:F6B8 EB 68              jmp   short Write_PCI_Byte  
F000:F6B8                  ; ---------------------------------------------------------------------------  
F000:F6BA 90                 db  90h ; ?  
F000:F6BB BD F6              dw 0F6BDh  
F000:F6BD                  ; ---------------------------------------------------------------------------  
F000:F6BD                  exit:                   ; CODE XREF: Chipset_Reg_Early_Init+94j  
F000:F6BD 66 C1 EC 10        shr   esp, 10h  
F000:F6C1 C3                 retn  
F000:F6C1                  Chipset_Reg_Early_Init endp  
.........  
F000:F6C4                  Begin_Chipset_Reg_Val  
F000:F6C4 47 38              dw 3847h                ; bus#0 dev#7 func#0: PCI2ISA Bridge reg 47h  
F000:F6C6 BF                 db 0BFh                ; disable PCI delay transaction  
F000:F6C7 A0                 db 0A0h                ; - Use INIT as CPU Reset  
F000:F6C7                    ; - Enable (ports 4D0-1 per EISA specification)  
F000:F6C7                    ;  
F000:F6C8 50 00              dw 50h                  ; Host Bridge reg 50h Request Phase Control  
F000:F6CA FF                 db 0FFh                ; and mask  
F000:F6CB 10                 db 10h                  ; enable Defer Retry When HLOCK Active  
F000:F6CB                    ;  
F000:F6CC 51 00              dw 51h                  ; Host Bridge reg 51h Response Phase Control  
F000:F6CE DD                 db 0DDh                ; reset reserved bits  
F000:F6CF 0D                 db 0Dh                  ; enable:  
F000:F6CF                    ; - Non-Posted IOW  
F000:F6CF                    ; - Concurrent PCI Master / Host Operation  
F000:F6CF                    ;  
F000:F6D0 52 00              dw 52h                  ; Host Bridge reg 52h Dynamic Defer Timer  
F000:F6D2 E0                 db 0E0h                ; disable dynamic defer  
F000:F6D3 08                 db 8                    ; snoop stall count = 8  
F000:F6D3                    ;  
F000:F6D4 6B 00              dw 6Bh                  ; Host Bridge reg 6Bh DRAM Arbitration Control  
F000:F6D6 00                 db 0                    ; - Park at last bus owner  
F000:F6D6                    ; - Disable Fast Read to Write turn-around  
F000:F6D6                    ; - Memory Module Configuration - Normal Operation  
F000:F6D6                    ; - MD Bus 2nd Level Strength Control = Normal Slew rate  
F000:F6D6                    ; - CAS Bus 2nd Level Strength Control = Normal Slew rate  
F000:F6D6                    ; - VC-DRAM disable  
F000:F6D6                    ; - Multi-Page Open disable  
F000:F6D7 01                 db 1                    ; Enable Multi-Page Open  
F000:F6D7                    ;  
F000:F6D8 71 00              dw 71h                  ; Host Bridge reg 71h CPU to PCI Flow Control 1  
F000:F6DA 00                 db 0                    ; Disable:  
F000:F6DA                    ; - Dynamic Burst  
F000:F6DA                    ; - Byte Merge  
F000:F6DA                    ; - PCI I/O Cycle Post Write  
F000:F6DA                    ; - PCI Burst  
F000:F6DA                    ; - PCI Fast Back-to-Back Write  
F000:F6DA                    ; - Quick Frame Generation  
F000:F6DA                    ; - 1 Wait State PCI Cycles  
F000:F6DB 08                 db 8                    ; Enable PCI Burst  
F000:F6DB                    ;  
F000:F6DC 75 00              dw 75h                  ; Host Bridge reg 75h PCI Arbitration 1  
F000:F6DE 00                 db 0                    ; - PCI has priority  
F000:F6DE                    ; - REQ-based (arbitrate at end of REQ#)  
F000:F6DE                    ; - Disable PCI Master Bus Time-Out  
F000:F6DF 80                 db 80h                  ; Fair arbitration between PCI and CPU  
F000:F6DF                    ;  
F000:F6E0 76 00              dw 76h                  ; Host Bridge reg 76h PCI Arbitration 2  
F000:F6E2 CF                 db 0CFh                ; and mask  
F000:F6E3 80                 db 80h                  ; or mask  
F000:F6E3                    ;  
```
