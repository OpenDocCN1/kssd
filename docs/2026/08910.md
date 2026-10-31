
# [原创]ECC patcher for the flexlm with ecc

```c
HANDLE hFile;
DWORD  fsize, lr, i, j, phydora;
void   *pMem;
char   *p, q;
TCHAR  backup[MAX_PATH];
TCHAR  lfile[MAX_PATH];
IMAGE_SECTION_HEADER *psec;

q = 0;
SetFileAttributes(pafile, FILE_ATTRIBUTE_NORMAL);
hFile = CreateFile(pafile, FILE_WRITE_DATA | FILE_READ_DATA, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
if (hFile == INVALID_HANDLE_VALUE)
    return;

fsize = GetFileSize(hFile, NULL);
pMem  = VirtualAlloc(NULL, fsize, MEM_COMMIT | MEM_TOP_DOWN, PAGE_READWRITE);

_try
{
    if (ReadFile(hFile, pMem, fsize, &lr, NULL))
    {
        p = (char*)(((char*)pMem + (*(DWORD *)((char*)pMem + 0x3c)) + 24) + ((IMAGE_FILE_HEADER*)((char*)pMem + (*(DWORD *)((char*)pMem + 0x3c)) + 4))->SizeOfOptionalHeader);

        for (int k = 0; k < ((IMAGE_FILE_HEADER*)((char*)pMem + (*(DWORD *)((char*)pMem + 0x3c)) + 4))->NumberOfSections; k++)
            if (stricmp((char*)((IMAGE_SECTION_HEADER*)p)->Name, ".data"))
                p += sizeof(IMAGE_SECTION_HEADER);

        psec = (IMAGE_SECTION_HEADER*)p;
        p    = (char*)((char*)pMem + psec->PointerToRawData);

        for (i = 0; i < (psec->SizeOfRawData - 4); i++, p++)
            if (0x006b6f00 == *((int*)p))
                break;

        phydora = ((IMAGE_SECTION_HEADER*)psec)->PointerToRawData + i + ((IMAGE_OPTIONAL_HEADER*)((char*)pMem + (*(DWORD *)((char*)pMem + 0x3c)) + 24))->ImageBase + 1;

        for (p = (char*)pMem, i = 0; i < (fsize - 4); i++, p++)
            if (*((DWORD*)p) == phydora)
                break;

        for (p--, i--, j = 56; i && j; i--, p--, j--)
            if ((*p) == (char)0x74)
                break;

        q = *p;
        SetFilePointer(hFile, i, NULL, FILE_BEGIN);
        q = 0xEB;
        if (WriteFile(hFile, &q, 1, &lr, NULL))
            FlushFileBuffers(hFile);
    }
}
__except(EXCEPTION_ACCESS_VIOLATION == GetExceptionCode())
{
    VirtualFree(pMem, 0, MEM_RELEASE);
    CloseHandle(hFile);
    return;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)
