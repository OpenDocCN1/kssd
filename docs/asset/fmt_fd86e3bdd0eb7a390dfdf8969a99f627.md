
# 7.1.3. `Init_Interrupt_n_PwrMgmt` routine

`Address` `Hex` `Mnemonic`

```
F000:E1A0 BF A6 E1               mov   di, 0E1A6h          ; ret addr below
F000:E1A3 E9 42 99               jmp   Init_Interrupt_n_PwrMgmt
F000:E1A6                        ; ---------------------------------------------------------------------------
F000:E1A6 B0 C1                 mov   al, 0C1h ; '+'
...
F000:7CDD                       delay:                      ; CODE XREF: Init_Interrupt_n_PwrMgmt:delay
F000:7CDD E2 FE                 loop  delay
F000:7CDF FF E7                 jmp   di                  ; jmp to F000:E1A6h
F000:7CDF                       Init_Interrupt_n_PwrMgmt endp
```

## 7.1.4. Call To "Early Silicon Support" Routine

`Address` `Hex` `Mnemonic`

```
F000:E1AD E9 47 02              jmp   Search_BBSS_Signature
F000:E1AD                        ; ---------------------------------------------------------------------------
F000:E1B0 B2 E1                 dw 0E1B2h
F000:E1B2                        ; ---------------------------------------------------------------------------
F000:E1B2 0B F6                 or    si, si
F000:E1B4 74 38                 jz    short _BBSS_not_found
F000:E1B6 8B DE                 mov   bx, si ;checksum start here
F000:E1B8 2E 8B 04              mov   ax, cs:[si]
F000:E1BB 25 00 F0              and   ax, 0F000h
F000:E1BE C1 E8 04              shr   ax, 4
F000:E1C1 05 00 F0              add   ax, 0F000h
F000:E1C4 8E D8                 mov   ds, ax              ; ds=base addr for calculation of BBSS checksum
F000:E1C6                       assume ds:nothing
F000:E1C6 32 E4                 xor   ah, ah
F000:E1C8 33 F6                 xor   si, si
F000:E1CA B9 FF 0F              mov   cx, 0FFFh           ; length of BBSS
F000:E1CD                       next_byte:                  ; CODE XREF: 000FE1D0 j
F000:E1CD AC                    lodsb
F000:E1CE 02 E0                 add   ah, al              ; calc 8-bit chksum for BBSS
F000:E1D0 E2 FB                 loop  next_byte
F000:E1D2 3A 24                 cmp   ah, [si]
F000:E1D4 75 18                 jnz   short _BBSS_not_found
F000:E1D6 2E 8B 07              mov   ax, cs:[bx]         ; chsum ok, jump to (exec) the BBSS engine
F000:E1D9 25 00 F0              and   ax, 0F000h
F000:E1DC 8B F0                 mov   si, ax
F000:E1DE 81 C6 FC 0F           add   si, 0FFCh
F000:E1E2 2E 8B 34              mov   si, cs:[si]
F000:E1E5 BC EC E1              mov   sp, 0E1ECh
F000:E1E8 FF E6                 jmp   si                  ; jmp to F000:60B4h (BBSS Engine), returns at F000:E1F8h
F000:E1E8                        ; the BBSS engine is silicon support routine, used to
F000:E1E8                        ; initialize PwrMgmt GPIO Ctlr, Host Ctlr and DRAM Ctlr
F000:E1E8                        ; ---------------------------------------------------------------------------
F000:E1EA 87                    db  87h ; ?
F000:E1EB DB                    db 0DBh ; -
F000:E1EC F8 E1                 dw 0E1F8h
.........
F000:E1F8                        BBSS_exec_done:
F000:E1F8 B8 00 00              mov   ax, 0
.........
```

这段代码在 `bootblock` 拷贝到 `RAM` 之前执行。万一 `RAM` 有错误，系统就要 `halt` 并且从扬声器输出错误代码。

## 7.1.5. Bootblock Is Copied And Executed In RAM

`Address` `Hex` `Mnemonic`

```
F000:E2AA                        ;----------- Enter 16-bit Protected Mode (Flat) -----------
F000:E2AA                        assume ds:_F000h
F000:E2AA 0F 01 16 F6 E4        lgdt  qword ptr word_F000_E4F6
F000:E2AF 0F 20 C0              mov   eax, cr0
F000:E2B2 0C 01                 or    al, 1
F000:E2B4 0F 22 C0              mov   cr0, eax
```
