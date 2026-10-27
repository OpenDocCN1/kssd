
# Decompression_Nngine

```assembly
2000:7857   call  Fetch_Byte          ; al = LZH_hdr_len
2000:785A   movzx ecx, al             ; ecx = LZH_hdr_len
2000:785E   add   bx, 7
2000:7861   call  Fetch_Dword         ; eax = compressed_file_size
2000:7864   sub   bx, 7
2000:7867   add   ecx, eax            ; ecx = LZH_header_len + compressed_file_size
2000:786A   add   ecx, 3              ; ecx = total_compressed_component_size
2000:786E   pop   gs
2000:7870   assume gs:nothing
2000:7870   jmp   exit
2000:7873 ; ---------------------------------------------------------------------------
```

```assembly
2000:7873 not_extension_component:    ; CODE XREF: Decompression_Ngine+90
2000:7873                             ; Decompression_Ngine+A5
2000:7873   pop   gs
2000:7875   call  Make_CRC16_Table
2000:7878   call  Read_Header         ; fetch header component to scratchpad_seg, on error CF=1
2000:787B   jb    exit                ; ret with error code set
2000:787F   mov   ax, ds:108h         ; mov   ax, decomprss_seg_addr
2000:7882   mov   ds:104h, ax         ; mov   nu_decomprss_seg_addr, ax
2000:7885   mov   ax, ds:10Ah         ; mov   ax, decomprss_offst_addr
2000:7888   mov   ds:106h, ax         ; mov   nu_decomprss_offst_addr, ax
2000:788B   mov   ecx, ds:310h        ; ecx = compressed_component_size
2000:7890   xor   eax, eax
2000:7893   mov   al, ds:571Ch        ; al = LZH_hdr_len
2000:7896   add   ecx, eax            ; ecx = compressed_cmpnnt_size + LZH_hdr_len
2000:7899   add   ecx, 3              ; ecx = compressed_cmpnnt_size + LZH_hdr_len +
2000:7899                             ;       sizeof(EOF_byte) + sizeof(LZH_hdr_len_byte) +
2000:7899                             ;       sizeof(LZH_hdr_8bit_chk_sum)
2000:7899                             ; i.e. ecx = overall_component_len
2000:789D   mov   edx, ds:314h        ; mov   edx, original_file_size
2000:78A2   push  edx
2000:78A4   push  ecx
2000:78A6   push  bx
2000:78A7   add   bx, 5               ; point to LZH ID byte
2000:78AA   call  Fetch_Byte
2000:78AD   pop   bx
2000:78AE   cmp   al, '0'             ; is '-lh0-'?
2000:78B0   jnz   short decompress_part
2000:78B2   push  ds
2000:78B3   push  si
2000:78B4   push  bx
2000:78B5   mov   di, ds:10Ah
2000:78B9   movzx ax, byte ptr ds:571Ch
2000:78BE   add   ax, 2
2000:78C1   add   bx, ax
2000:78C3   mov   cx, ds:310h
2000:78C7   mov   ax, ds:108h
2000:78CA   mov   es, ax
2000:78CC   add   cx, 3
2000:78CF   shr   cx, 2
2000:78D2 next_dword:                 ; CODE XREF: Decompression_Ngine+151
2000:78D2   call  Fetch_Dword
2000:78D5   add   bx, 4
2000:78D8   stosd
2000:78DA   loop  next_dword
2000:78DC   pop   bx
2000:78DD   pop   si
2000:78DE   pop   ds
2000:78DF   jmp   short LZH_hdr_OK
2000:78E1 ; ---------------------------------------------------------------------------
```

```assembly
2000:78E1 decompress_part:            ; CODE XREF: Decompression_Ngine+127
2000:78E1   push  word ptr ds:104h    ; save destination seg addr
2000:78E5   push  word ptr ds:106h    ; save destination offset addr
2000:78E9   push  large [dword ptr ds:314h]
2000:78EE   call  Lzh_Expand          ; Lzh_Expand capable of handling compressed
2000:78EE                             ; component bigger than 64KB (1 segment)
2000:78F1   pop   dword ptr ds:314h
2000:78F6   pop   word ptr ds:106h
2000:78FA   pop   word ptr ds:104h
2000:78FE LZH_hdr_OK:                 ; CODE XREF: Decompression_Ngine+156
2000:78FE   call  Zero_Init           ; zero init 32KB of scratchpad_seg
2000:7901   pop   ecx
2000:7903   pop   edx
2000:7905   clc
2000:7906 exit:                       ; CODE XREF: Decompression_Ngine+E7
2000:7906                             ; Decompression_Ngine+F2
2000:7906   pop   es
2000:7907   pop   bx
2000:7908   pop   eax
2000:790A   retn
2000:790A Decompression_Ngine endp
```

## Make_CRC16_Table

```assembly
2000:790B                         --- Make_CRC16_Table ---
2000:790B                         1st pass, the base address for DS is 3_0000h
2000:790B                         in: ds = scratch_pad_segment for CRC table
2000:790B                         out: ds:10Ch - ds:11Bh = CRC-16 table
2000:790B                         ; --------------- S U B R O U T I N E ---------------------------------------
2000:790B                         Make_CRC16_Table proc near  ; CODE XREF: Decompression_Ngine+EC
2000:790B 51                        push  cx
2000:790C 53                        push  bx
2000:790D 50                        push  ax
2000:790E 56                        push  si
2000:790F BE 0C 01                  mov   si, 10Ch
2000:7912 B9 00 01                  mov   cx, 100h
2000:7915                         next_byte:                  ; CODE XREF: Make_CRC16_Table+2B
2000:7915 B8 00 01                  mov   ax, 100h
2000:7918 2B C1                     sub   ax, cx
2000:791A 50                        push  ax
2000:791B BB 00 00                  mov   bx, 0
2000:791E                         is_bit:                     ; CODE XREF: Make_CRC16_Table+25
2000:791E A9 01 00                  test  ax, 1
2000:7921 74 07                     jz    short not_bit
2000:7923 D1 E8                     shr   ax, 1
2000:7925 35 01 A0                  xor   ax, 0A001h          ; CRC poly
2000:7928 EB 02                     jmp   short point_to_next_byte
2000:792A                         ; ---------------------------------------------------------------------------
2000:792A                         not_bit:                    ; CODE XREF: Make_CRC16_Table+16
2000:792A D1 E8                     shr   ax, 1
2000:792C                         point_to_next_byte:         ; CODE XREF: Make_CRC16_Table+1D
2000:792C 43                        inc   bx
2000:792D 83 FB 08                  cmp   bx, 8
2000:7930 72 EC                     jb    short is_bit
2000:7932 5B                        pop   bx
2000:7933 89 00                     mov   [bx+si], ax
2000:7935 46                        inc   si
2000:7936 E2 DD                     loop  next_byte
2000:7938 5E                        pop   si
2000:7939 58                        pop   ax
2000:793A 5B                        pop   bx
2000:793B 59                        pop   cx
2000:793C C3                        retn
2000:793C                         Make_CRC16_Table endp
```

## Read_Header

```assembly
2000:79E8                         --- Read_Header ---
2000:79E8                         in: ds = scratchpad_segment
2000:79E8                               ds:102h = LZH_hdr_byte_index
2000:79E8
2000:79E8                         out:  ds:102h = LZH_hdr_byte_index
2000:79E8                               ds:108h = componnt_decomprrsion_seg_addr
2000:79E8                               ds:10Ah = componnt_decomprrsion_offset_addr
2000:79E8                              ds:310h = uncompressed_componnt_size
2000:79E8                              ds:314h =  component_seg:offset_decompression_addr
2000:79E8                              ds:571Ch = LZH_hdr_len
2000:79E8                              ds:571Dh = LZH_hdr_chksum
2000:79E8                              ds:571Eh = LZH crc16 val
2000:79E8                              ds:0 - ds:LZH_hdr_len = copy of current component LZH hdr
2000:79E8                         ; --------------- S U B R O U T I N E ---------------------------------------
2000:79E8                         Read_Header proc near       ; CODE XREF: Decompression_Ngine+EF
2000:79E8 60                        pusha
2000:79E9 06                        push  es
2000:79EA 8B 1E 02 01               mov   bx, ds:102h
2000:79EE E8 DA 00                  call  Fetch_Byte
2000:79F1 FF 06 02 01               inc   word ptr ds:102h
```
