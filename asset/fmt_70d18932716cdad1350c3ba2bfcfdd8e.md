
```
E000:6F08 EB 26 jmp   short decompress  
E000:6F0A ; ---------------------------------------------------------------------------  
E000:6F0A above_1MB: ; CODE XREF: decompress_BIOS_component+68  
E000:6F0A 66 81 C3 00 00 0E 00 add   ebx, 0E0000h  
E000:6F11 26 67 8B 4B 11 mov   cx, es:[ebx+11h] ; cx = target segment addr (the component will be  
E000:6F11 ; decompressed into this segmnt)  
E000:6F16 51 push  cx  
E000:6F17 26 67 FF 33 push  word ptr es:[ebx]  
E000:6F1B F6 C4 80 test  ah, 80h  
E000:6F1E 74 10 jz    short decompress  
E000:6F20 26 67 89 53 11 mov   es:[ebx+11h], dx  
E000:6F25 02 CD add   cl, ch  
E000:6F27 02 D6 add   dl, dh  
E000:6F29 2A CE sub   cl, dh  
E000:6F2B 26 67 28 4B 01 sub   es:[ebx+1], cl  
E000:6F30 decompress: ; CODE XREF: decompress_BIOS_component+AF  
E000:6F30 ; decompress_BIOS_component+BF ...  
E000:6F30 66 C1 CB 10 ror   ebx, 10h  
E000:6F34 8E C3 mov   es, bx ; es = phy_addr>>0xC (hi_word of phy addr)  
E000:6F36 66 C1 CB 10 ror   ebx, 10h ; bx = compressed component offset  
E000:6F3A 0E push  cs  
E000:6F3B 68 49 6F push  6F49h ; ret addr below  
E000:6F3E 68 FF DF push  0DFFFh  
E000:6F41 BA 00 30 mov   dx, 3000h ; scratch_pad_seg  
E000:6F44 68 00 20 push  2000h  
E000:6F47 57 push  di ; call Decompression_Ngine  
E000:6F48 CB retf          
E000:6F49 ; ---------------------------------------------------------------------------  
E000:6F49 68 00 E0 push  0E000h  
E000:6F4C 68 5A 6F push  6F5Ah  
E000:6F4F 68 31 EC push  0EC31h  
E000:6F52 68 D4 E3 push  0E3D4h ; disable_A20...  
E000:6F55 EA 30 EC 00 F0 jmp   far ptr F000_Vector  
E000:6F5A ; ---------------------------------------------------------------------------  
E000:6F5A E8 3E 7C call  ds_ss_Enter_Voodoo_Mode  
E000:6F5D 8C D8 mov   ax, ds  
E000:6F5F 8E C0 mov   es, ax ; es --> voodoo mode, base at 0000_0000h  
E000:6F61 E8 54 7C call  ss_Leave_Voodoo_mode  
E000:6F64 67 66 A1 00 00 08 00 mov   eax, ds:80000h  
E000:6F6B 67 66 3B 05 00 00 16 00 cmp   eax, ds:160000h  
E000:6F73 75 18 jnz   short restore_header  
E000:6F75 66 C1 CB 10 ror   ebx, 10h  
E000:6F79 8E C3 mov   es, bx  
E000:6F7B 66 C1 CB 10 ror   ebx, 10h  
E000:6F7F 26 8F 07 pop   word ptr es:[bx]  
E000:6F82 26 8F 47 11 pop   word ptr es:[bx+11h]  
E000:6F86 26 66 8B 5F 0B mov   ebx, es:[bx+0Bh]  
E000:6F8B EB 0F jmp   short disable_A20  
E000:6F8D ; ---------------------------------------------------------------------------  
E000:6F8D restore_header: ; CODE XREF: decompress_BIOS_component+12A  
E000:6F8D 26 67 8F 03 pop   word ptr es:[ebx]  
E000:6F91 26 67 8F 43 11 pop   word ptr es:[ebx+11h]  
E000:6F96 26 67 66 8B 5B 0B mov   ebx, es:[ebx+0Bh]  
E000:6F9C disable_A20: ; CODE XREF: decompress_BIOS_component+142  
E000:6F9C 68 00 E0 push  0E000h  
E000:6F9F 68 AD 6F push  6FADh  
E000:6FA2 68 31 EC push  0EC31h  
E000:6FA5 68 24 E4 push  0E424h ; disable gate A20  
E000:6FA8 EA 30 EC 00 F0 jmp   far ptr F000_Vector  
E000:6FAD ; ---------------------------------------------------------------------------  
E000:6FAD F8 clc  
E000:6FAE EB 01 jmp   short decomprss_BIOS_componnt_ret  
E000:6FB0 ; ---------------------------------------------------------------------------  
E000:6FB0 no_cmprssd_component: ; CODE XREF: decompress_BIOS_component+35  
E000:6FB0 ; decompress_BIOS_component+3C  
E000:6FB0 F9 stc  
E000:6FB1 decomprss_BIOS_componnt_ret:  
E000:6FB1 ; CODE XREF: decompress_BIOS_component+46  
E000:6FB1 ; decompress_BIOS_component+165  
E000:6FB1 9C pushf  
E000:6FB2 66 53 push  ebx  
E000:6FB4 68 00 E0 push  0E000h  
E000:6FB7 68 C5 6F push  6FC5h  
E000:6FBA 68 31 EC push  0EC31h  
E000:6FBD 68 D4 E3 push  0E3D4h ; enable gate A20  
E000:6FC0 EA 30 EC 00 F0 jmp   far ptr F000_Vector  
E000:6FC5 ; ---------------------------------------------------------------------------  
E000:6FC5 E8 D3 7B call  ds_ss_Enter_Voodoo_Mode  
E000:6FC8 8C D8 mov   ax, ds  
E000:6FCA 8E C0 mov   es, ax  
E000:6FCC E8 E9 7B call  ss_Leave_Voodoo_mode  
E000:6FCF 67 66 A1 00 00 08 00 mov   eax, ds:80000h  
E000:6FD6 67 66 3B 05 00 00 16 00 cmp   eax, ds:160000h  
E000:6FDE 75 2B jnz   short exit  
E000:6FE0 66 BF 00 00 08 00 mov   edi, 80000h  
E000:6FE6 66 B9 00 40 00 00 mov   ecx, 4000h  
E000:6FEC 66 33 C0 xor   eax, eax  
E000:6FEF FC cld  
E000:6FF0 67 F3 66 AB rep stos dword ptr es:[edi]  
E000:6FF4 66 BE 00 00 14 00 mov   esi, 140000h  
E000:6FFA 66 BF 00 00 09 00 mov   edi, 90000h  
E000:7000 66 B9 00 40 00 00 mov   ecx, 4000h  
E000:7006 FC cld  
E000:7007 67 F3 66 A5 rep movs dword ptr es:[edi], dword ptr [esi]  
E000:700B exit: ; CODE XREF: decompress_BIOS_component+195  
E000:700B 68 00 E0 push  0E000h  
E000:700E 68 1C 70 push  701Ch  
E000:7011 68 31 EC push  0EC31h  
E000:7014 68 24 E4 push  0E424h ; turn off Gate A20  
E000:7017 EA 30 EC 00 F0 jmp   far ptr F000_Vector  
E000:701C ; ---------------------------------------------------------------------------  
E000:701C 66 5B pop   ebx  
E000:701E B0 00 mov   al, 0 ; disable CPU cache  
E000:7020 E8 49 FC call  F0_init_cpu_cache  
E000:7023 9D popf  
E000:7024 5D pop   bp  
E000:7025 07 pop   es  
E000:7026 1F pop   ds  
E000:7027 C3 retn  
E000:7027 decompress_BIOS_component endp ; sp = -18h
```
