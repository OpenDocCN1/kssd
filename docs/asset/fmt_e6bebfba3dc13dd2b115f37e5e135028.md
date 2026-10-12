
```
2000:779C  mov   word ptr ds:100h, es ; ds:100h = compressed_seg_addr>>0xC

2000:779C                          ; 1st pass ds:100h = 1

2000:77A0  mov   ds:102h, bx       ; ds:102h = compressed_offset_addr

2000:77A0                          ; 1st pass bx = 0

2000:77A4  xor   ecx, ecx

2000:77A7  mov   ds:57A8h, ecx

2000:77AC  mov   ds:57ACh, ecx

2000:77B1  lea   cx, ds:57A8h

2000:77B5  ror   ecx, 4

2000:77B9  mov   ax, ds

2000:77BB  add   cx, ax

2000:77BD  rol   ecx, 4

2000:77C1  mov   word ptr ds:57A2h, 18h

2000:77C7  mov   ds:57A4h, ecx

2000:77CC  mov   dword ptr ds:57B0h, 0FFFFh

2000:77D5  mov   ax, es

2000:77D7  movzx ecx, ah           ; es = hi_word addr of desc_base

2000:77DB  ror   ecx, 8            ; ecx = base_24_31 << 24

2000:77DF  mov   cl, al

2000:77E1  or    ecx, 8F9300h

2000:77E8  mov   ds:57B4h, ecx

2000:77ED  mov   dword ptr ds:57B8h, 0FFFFh

2000:77F6  mov   dword ptr ds:57BCh, 8F9300h

2000:77FF  push  gs

2000:7801  mov   di, 0

2000:7804  mov   gs, di

2000:7806  assume gs:nothing

2000:7806  mov   di, 6000h

2000:7809  mov   word ptr gs:[di], 7789h

2000:780E

2000:780E ;check LZH header

2000:780E  add   bx, 12h           ; LZH-header decomp_seg_addr_hi_byte index

2000:7811  call  Fetch_Byte

2000:7814  sub   bx, 12h           ; restore bx to point to first byte

2000:7817  cmp   al, 40h ; '@'     ; is extension component?

2000:7817                          ; at 1st: al equ 50h (original.tmp)

2000:7817                          ; at 2nd: al equ 41h (awardext.rom)

2000:7819  jnz   short not_extension_component ; 1st-pass jmp taken

2000:781B  add   bx, 11h

2000:781E  call  Fetch_Byte        ; fetch "dest_seg_addr" lo_byte

2000:7821  sub   bx, 11h           ; restore bx to point to first byte

2000:7824  or    al, al            ; if extension component, jmp taken

2000:7826  jnz   short extension_component

2000:7828  cmp   dword ptr gs:[di+4], 0

2000:782E  jnz   short not_extension_component

2000:7830 extension_component:      ; CODE XREF: Decompression_Ngine+9D

2000:7830  movzx dx, al            ; dl = "dest_seg_addr" lo_byte

2000:7833  inc   bx                ; bx = LZH_hdr_chksum byte index

2000:7834  call  Fetch_Byte

2000:7837  sub   al, dl            ; LZH_hdr_chksum = LZH_hdr_chksum - "dest_seg_addr"_lo_byte

2000:7839  call  Patch_Byte        ; store new checksum

2000:783C  dec   bx                ; restore bx

2000:783D  xor   al, al            ; al = 00h

2000:783F  add   bx, 11h           ; bx = "dest_seg_addr"_lo_byte index

2000:7842  call  Patch_Byte        ; patch "dest_seg_addr"_lo_byte to 00h

2000:7845  sub   bx, 11h

2000:7848  inc   dx                ; dx = "dest_seg_addr"_lo_byte + 1

2000:7849  shl   dx, 2             ; dx = 4*("dest_seg_addr"_lo_byte + 1)

2000:784C  add   di, dx            ; di = 6000h + dx  -- look above!

2000:784E  mov   gs:[di], bx       ; 0000:[di] = compressed_offset_addr

2000:7851  mov   cx, es

2000:7853  mov   gs:[di+2], cx     ; 0000:[di+2] = compressed_seg_addr>>0xC (hi_word of src phy addr)

2000:7857  call  Fetch_Byte        ; al = LZH_hdr_len

2000:785A  movzx ecx, al           ; ecx = LZH_hdr_len

2000:785E  add   bx, 7

2000:7861  call  Fetch_Dword       ; eax = compressed_file_size

2000:7864  sub   bx, 7

2000:7867  add   ecx, eax          ; ecx = LZH_header_len + compressed_file_size

2000:786A  add   ecx, 3            ; ecx = total_compressed_component_size

2000:786E  pop   gs

2000:7870  assume gs:nothing

2000:7870  jmp   exit

2000:7873 ; ---------------------------------------------------------------------------

2000:7873 not_extension_component: ; CODE XREF: Decompression_Ngine+90

2000:7873                          ; Decompression_Ngine+A5

2000:7873  pop   gs

2000:7875  call  Make_CRC16_Table

2000:7878  call  Read_Header       ; fetch header component to scratchpad_seg, on error CF=1

2000:787B  jb    exit              ; ret with error code set

2000:787F  mov   ax, ds:108h       ; mov   ax, decomprss_seg_addr

2000:7882  mov   ds:104h, ax       ; mov   nu_decomprss_seg_addr, ax

2000:7885  mov   ax, ds:10Ah       ; mov   ax, decomprss_offst_addr

2000:7888  mov   ds:106h, ax       ; mov   nu_decomprss_offst_addr, ax

2000:788B  mov   ecx, ds:310h      ; ecx = compressed_component_size

2000:7890  xor   eax, eax

2000:7893  mov   al, ds:571Ch      ; al = LZH_hdr_len

2000:7896  add   ecx, eax          ; ecx = compressed_cmpnnt_size + LZH_hdr_len

2000:7899  add   ecx, 3            ; ecx = compressed_cmpnnt_size + LZH_hdr_len +

2000:7899                          ;       sizeof(EOF_byte) + sizeof(LZH_hdr_len_byte) +

2000:7899                          ;       sizeof(LZH_hdr_8bit_chk_sum)

2000:7899                          ; i.e. ecx = overall_component_len

2000:789D  mov   edx, ds:314h      ; mov   edx, original_file_size

2000:78A2  push  edx

2000:78A4  push  ecx

2000:78A6  push  bx

2000:78A7  add   bx, 5             ; point to LZH ID byte

2000:78AA  call  Fetch_Byte

2000:78AD  pop   bx

2000:78AE  cmp   al, '0'           ; is '-lh0-'?

2000:78B0  jnz   short decompress_part

2000:78B2  push  ds

2000:78B3  push  si

2000:78B4  push  bx

2000:78B5  mov   di, ds:10Ah

2000:78B9  movzx ax, byte ptr ds:571Ch

2000:78BE  add   ax, 2

2000:78C1  add   bx, ax

2000:78C3  mov   cx, ds:310h

2000:78C7  mov   ax, ds:108h

2000:78CA  mov   es, ax

2000:78CC  add   cx, 3

2000:78CF  shr   cx, 2

2000:78D2 next_dword:              ; CODE XREF: Decompression_Ngine+151

2000:78D2  call  Fetch_Dword

2000:78D5  add   bx, 4

2000:78D8  stosd

2000:78DA  loop  next_dword

2000:78DC  pop   bx

2000:78DD  pop   si

2000:78DE  pop   ds

2000:78DF  jmp   short LZH_hdr_OK

2000:78E1 ; ---------------------------------------------------------------------------

2000:78E1 decompress_part:        ; CODE XREF: Decompression_Ngine+127

2000:78E1  push  word ptr ds:104h  ; save destination seg addr

2000:78E5  push  word ptr ds:106h  ; save destination offset addr

2000:78E9  push  large [dword ptr ds:314h]

2000:78EE  call  Lzh_Expand        ; Lzh_Expand capable of handling compressed

2000:78EE                          ; component bigger than 64KB (1 segment)

2000:78F1  pop   dword ptr ds:314h

2000:78F6  pop   word ptr ds:106h

2000:78FA  pop   word ptr ds:104h

2000:78FE LZH_hdr_OK:             ; CODE XREF: Decompression_Ngine+156

2000:78FE  call  Zero_Init         ; zero init 32KB of scratchpad_seg

2000:7901  pop   ecx

2000:7903  pop   edx

2000:7905  clc

2000:7906 exit:                   ; CODE XREF: Decompression_Ngine+E7

2000:7906                          ; Decompression_Ngine+F2

2000:7906  pop   es

2000:7907  pop   bx

2000:7908  pop   eax

2000:790A  retn

2000:790A Decompression_Ngine endp

_______________________


2000:790B             --- Make_CRC16_Table ---

2000:790B             1st pass, the base address for DS is 3_0000h
```
