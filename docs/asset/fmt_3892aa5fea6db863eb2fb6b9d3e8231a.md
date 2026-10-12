
```assembly
2000:78E9    push    large [dword ptr ds:314h]
2000:78EE    call    Lzh_Expand               ; Lzh_Expand capable of handling compressed
2000:78EE                                       ; component bigger than 64KB (1 segment)
2000:78F1    pop     dword ptr ds:314h
2000:78F6    pop     word ptr ds:106h
2000:78FA    pop     word ptr ds:104h
2000:78FE LZH_hdr_OK:                        ; CODE XREF: Decompression_Ngine+156
2000:78FE    call    Zero_Init              ; zero init 32KB of scratchpad_seg
2000:7901    pop     ecx
2000:7903    pop     edx
2000:7905    clc
2000:7906 exit:                             ; CODE XREF: Decompression_Ngine+E7
2000:7906                                       ; Decompression_Ngine+F2
2000:7906    pop     es
2000:7907    pop     bx
2000:7908    pop     eax
2000:790A    retn
2000:790A Decompression_Ngine endp
```

这个用红线标记的一行写入“signatures”到内存。比如，`nnoprom.bin` 部分用 `id:4027h` 定义。这个例程中，`nnoprom.bin` 的索引就要被保存。这个索引的计算方法如下（也是上面代码所看到的算法）：

`index = 4*(lo_byte(ID) + 1)`

这个所引用来计算地址来保存信息或者签名。再 `nnoprom.bin` 里，索引是 `A0h`（从`[4*（27h+1）]`）。所以，保存了信息的这个地址开始与 `60A0h`。就像你上面看到的，第一个保存的信息是有压缩的“Extension_BIOS components”这个部分的偏移地址。它被保存在地址 `60A0h`（for `nnoprom.bin`）。然后，临时“压缩/解压段地址”保存在 `60A2h`（for `nnoprom.bin`）。对所有的“扩展 bios 部分”，这个临时的“压缩/解压段地址”经常是 `4000h`，就像你上面看到的代码那样（这个临时的被解压段的值修改成了 `4000h`，在被保存到 RAM 之前）。这个相同的进程就为其他的所有“扩展 bios 部分”执行完成了。

d. 下一步，`original.tmp`（post 执行期间）中的 `POST_8S` 负责准备为解压缩必须的签名，如下面你所看到的：

```assembly
E000:17B8                 POST_8S proc near
.........
E000:183F BE 00 00         mov   si, 0
E000:1842 8E DE              mov   ds, si
E000:1844                   assume ds:nothing
E000:1844 BE 00 70         mov   si, 7000h
E000:1847 8B 44 04         mov   ax, [si+4]         ; ax = FFFFh (0000:7004h filled before by Decompress_System_BIOS
E000:1847                                           ; during bootblock_in_RAM execution)
E000:184A BF 00 00         mov   di, 0
E000:184D 8E C7              mov   es, di
E000:184F                   assume es:nothing
E000:184F BF 00 60         mov   di, 6000h
E000:1852 26 89 45 04        mov   es:[di+4], ax       ; [0000:6004] = FFFFh --> signature to do LZH decompression
E000:1852                                           ; for extension components
E000:1856 3D FF FF           cmp   ax, 0FFFFh
E000:1859 74 10              jz    short signature_ok
E000:185B 8B 04              mov   ax, [si]
E000:185D 26 89 45 04        mov   es:[di+4], ax
E000:1861 8B 44 02         mov   ax, [si+2]
E000:1864 C1 E8 0C           shr   ax, 0Ch
E000:1867 26 89 45 06        mov   es:[di+6], ax
E000:186B                signature_ok:               ; CODE XREF: POST_8S+A1
E000:186B E8 A2 6C           call  _init_Pwr_Mgmt_ctlr
E000:186E F8                  clc
E000:186F C3                  retn
E000:186F                 POST_8S endp
```

e. 下一步，例程 `init_nnoprom_rosupd`（这只是一个例子，其他部分可能会不同）解压缩 `nnoprom.bin` 和 `rosupd.bin`，如下面代码：

```assembly
E000:71C1                 init_nnoprom_rosupd proc near ; CODE XREF: POST_11S
.........
E000:71CF BF A0 00           mov   di, 0A0h ; 'a'       ; nnoprom.bin index
E000:71CF                                           ; nnoprom.bin-->4027h; A0h = 4h*(lo_byte(4027h)+1h)
E000:71D2 E8 74 FC           call  near ptr decompress_BIOS_component ; decompress NNOPROM.BIN
E000:71D5 0F 82 ED 00        jb    decompression_error
E000:71D9 68 00 40           push  4000h
E000:71DC 1F                 pop   ds                  ; ds = 4000h; decompression result seg
E000:71DD                   assume ds:nothing
E000:71DD 33 F6               xor   si, si
E000:71DF 68 00 70           push  7000h
E000:71E2 07                 pop   es                  ; es = 7000h
E000:71E3                   assume es:nothing
E000:71E3 33 FF               xor   di, di
E000:71E5 B9 00 40           mov   cx, 4000h
E000:71E8 FC                 cld
E000:71E9 F3 66 A5            rep movsd                 ; copy nnoprom decompression result from
E000:71E9                                           ; seg 4000h to seg 7000h
E000:71EC BF 03 00           mov   di, 3
E000:71EF 66 26 81 3D 24 4E 4E 4O   cmp   dword ptr es:[di], 'ONN$' ; match (decompressed) nnoprom.bin signature
E000:71F7 0F 85 CB 00        jnz   decompression_error
E000:71FB 68 F8 9F           push  9FF8h
E000:71FE 07                 pop   es                  ; es = 9FF8h
E000:71FF                   assume es:nothing
E000:71FF 33 FF               xor   di, di
E000:7201 B9 68 00           mov   cx, 68h ; 'h'
E000:7204 32 C0               xor   al, al
E000:7206 F3 AA               rep stosb
E000:7208 BF A4 00           mov   di, 0A4h ; 'a'       ; ROSUPD.bin index
E000:720B E8 3B FC           call  near ptr decompress_BIOS_component ; decompress ROSUPD.bin
E000:720E 0F 82 B4 00        jb    decompression_error
E000:7212 1E                 push  ds
E000:7213 06                 push  es
E000:7214 0F A0                push  fs
E000:7216 0F A8                push  gs
E000:7218 9A DD 5F 00 F0       call  ds_es_fs_gs_flat_4GB
E000:721D 66 33 F6           xor   esi, esi
E000:7220 8E DE               mov   ds, si
E000:7222                   assume ds:nothing
E000:7222 8E C6               mov   es, si
E000:7224                   assume es:nothing
E000:7224 68 00 40           push  4000h
E000:7227 5E                 pop   si
E000:7228 66 C1 E6 04        shl   esi, 4              ; esi = 4_0000h (decompressed ROSUPD.bin)
E000:722C 66 BF 00 00 10 00   mov   edi, 100000h
E000:7232 66 8B CB           mov   ecx, ebx
E000:7235 66 C1 E9 02        shr   ecx, 2
E000:7239 FC                 cld
E000:723A F3 67 66 26 A5     rep movs dword ptr es:[edi], dword ptr es:[esi] ; copy decmprssd ROSUPD.BIN to 1MB
E000:723F 0F A9               pop   gs
E000:7241 0F A1               pop   fs
E000:7243 07                 pop   es
E000:7244                   assume es:nothing
E000:7244 1F                 pop   ds
E000:7245                   assume ds:nothing
E000:7245 68 F8 9F           push  9FF8h
E000:7248 07                 pop   es
E000:7249                   assume es:nothing
E000:7249 66 26 C7 06 00 00 00 00+  mov   dword ptr es:0, 100000h
E000:7253 66 26 C7 06 04 00 00 00+  mov   dword ptr es:4, 40000h
E000:725D 66 33 C0           xor   eax, eax
E000:7260 B8 00 E0           mov   ax, 0E000h
E000:7263 66 C1 E0 04        shl   eax, 4
E000:7267 66 05 56 71 00 00   add   eax, 7156h
E000:726D 66 26 A3 08 00     mov   es:8, eax
E000:7272 B8 07 00           mov   ax, 7
E000:7275 26 A3 0C 00        mov   es:0Ch, ax
```
