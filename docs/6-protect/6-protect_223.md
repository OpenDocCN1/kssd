

# ECC patcher for the flexlm with ecc

> 原文：[`www.pediy.com/kssd/pediy10/68456.html`](https://www.pediy.com/kssd/pediy10/68456.html)

HANDLE hFile;  
  DWORD  fsize, lr, i, j,phydora;  
  void   \*pMem;  
    char    \*p, q;  
  TCHAR   backup\[MAX\_PATH\];  
  TCHAR   lfile\[MAX\_PATH\];  
    
  IMAGE\_SECTION\_HEADER   \*psec;  
  
  q = 0;  
  
  SetFileAttributes(pafile, FILE\_ATTRIBUTE\_NORMAL);  
  
  hFile = CreateFile(pafile, FILE\_WRITE\_DATA | FILE\_READ\_DATA, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);  
  
  if (hFile == INVALID\_HANDLE\_VALUE)   
    return;  
  
  fsize = GetFileSize(hFile, NULL);  
  
  pMem= VirtualAlloc(NULL, fsize, MEM\_COMMIT | MEM\_TOP\_DOWN, PAGE\_READWRITE);  
  \_try   
  {  
  if(ReadFile(hFile, pMem, fsize, &lr, NULL))  
  {  
      p    =  (char\*)(((char\*)pMem +(\*(DWORD \*)((char\*)pMem + 0x3c)) + 24) + ((IMAGE\_FILE\_HEADER\*)((char\*)pMem +(\*(DWORD \*)((char\*)pMem + 0x3c)) + 4))->SizeOfOptionalHeader);  
      for(int k = 0; k < ((IMAGE\_FILE\_HEADER\*)((char\*)pMem +(\*(DWORD \*)((char\*)pMem + 0x3c)) + 4))->NumberOfSections; k++)  
      if(stricmp((char\*)((IMAGE\_SECTION\_HEADER\*)p)->Name,".data"))  
        p += sizeof(IMAGE\_SECTION\_HEADER);  
  
      psec  = (IMAGE\_SECTION\_HEADER\*)p;  
      p     = (char\*)((char\*)pMem  + psec->PointerToRawData);  
  
      for( i = 0 ; i < (psec->SizeOfRawData - 4); i++, p++)  
        if( 0x006b6f00 == \*((int\*)p))  
          break;  
      phydora = ((IMAGE\_SECTION\_HEADER\*)psec)->PointerToRawData + i + ((IMAGE\_OPTIONAL\_HEADER\*)((char\*)pMem +(\*(DWORD \*)((char\*)pMem + 0x3c)) + 24))->ImageBase + 1;  
  
      for(p = (char\*)pMem, i = 0; i < (fsize - 4); i ++, p++)  
        if(\*((DWORD\*)p) == phydora)  
          break;  
      for( p--, i--, j = 56; i&&j; i--, p--,j--)  
        if((\*p) == (char)0x74)  
          break;  
            q = \*p;  
                                        SetFilePointer(hFile, i, NULL, FILE\_BEGIN);  
            q = 0xEB;  
            if(WriteFile(hFile,&q, 1, &lr, NULL))  
                    FlushFileBuffers(hFile);  
    }  
  }  
  }  
  \_\_except(EXCEPTION\_ACCESS\_VIOLATION ==GetExceptionCode())  
  {  
    VirtualFree(pMem, 0, MEM\_RELEASE);  
      CloseHandle(hFile);  
    return;  
  } 

这个函数修改的是\_l\_ckout\_string\_key，这可是大名鼎鼎的，laoqian 的寻找明码就是这个函数，还有一个看雪兄弟（抱歉记不住名字）提到的 flexwrap 方法也是在这个函数里，  
此函数 4k 长，没想到这次去 ecc 也在这儿，玩法很多啊，不知道有效否。  
记得以前提到的去 ecc 的方法修改的是\_l\_pubkey\_verify 函数，让其返回 1 即可。  
很多朋友问找\_l\_pubkey\_verify 函数的方法，我在坛子里提到三种方法，  
比较简单的方法是查找 292a，通用。当然有更好的方法请不吝赐教。  
回到正题，一处修改的是：  
.text:00425410 50                                            push    eax  
.text:00425411 FF 95 64 FE FF FF                             call    \[ebp+var\_19C\]          
.text:00425417 83 C4 24                                      add     esp, 24h  
.text:0042541A 89 85 4C FE FF FF                             mov     \[ebp+var\_1B4\], eax  
.text:00425420 8B 8D 50 FE FF FF                             mov     ecx, \[ebp+var\_1B0\]  
.text:00425426 51                                            push    ecx             ; void \*  
.text:00425427 E8 1F 15 00 00                                call    \_l\_free  
.text:0042542C 83 C4 04                                      add     esp, 4  
.text:0042542F 83 BD 4C FE FF FF 00                          cmp     \[ebp+var\_1B4\], 0  
.text:00425436 74 0C                                         jz      short loc\_425444    ；将 74 修改成 eb，也就是永远跳到 ok。  
.text:00425438 C7 85 84 FE FF FF 00 00 00 00                 mov     \[ebp+var\_17C\], 0  
.text:00425442 EB 0A                                         jmp     short loc\_42544E  
.text:00425444                               ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:00425444  
.text:00425444                               loc\_425444:                             ; CODE XREF: \_l\_ckout\_string\_key+35Fj  
.text:00425444 C7 85 84 FE FF FF E0 8A 4B 00                 mov     \[ebp+var\_17C\], offset aOk ; "ok"  
.text:0042544E  
.text:0042544E                               loc\_42544E:                             ; CODE XREF: \_l\_ckout\_string\_key+36Bj  
.text:0042544E E9 D2 0C 00 00                                jmp     loc\_426125  
  
好了，让我们再看看 C 源代码，l\_ckout\_string\_key 函数被定义成 l\_string\_key 函数，属于 l\_strkey.c 文件。  
  
  
#ifdef LM\_CKOUT  
    LM\_DS\_AUTH app\_filter = (LM\_DS\_AUTH)kf->app\_filter;  
    unsigned char \*sig;  
    int keylen = strlen(license\_key);  
    int status;  
    unsigned int ii, b, jj, kk;  
    char s\[3\];  
  
    sig = (unsigned char \*)l\_malloc(job, (keylen/2) + 1);  
    s\[2\] = 0;  
    for (kk = 0, jj = 0, ii = 0; license\_key\[ii\]; ii++)  
    {  
      if (license\_key\[ii\] == ' ')  
                continue;  
      s\[jj++\] = license\_key\[ii\];  
      if (jj == 2)  
      {  
        jj = 0;  
        sscanf(s, "%02X", &b);  /\* overrun checked \*/  
        sig\[kk++\] = b;  
      }  
    }  
  
    status = (\*app\_filter)((char \*)job, input, inputlen, sig, kk,  
      &kf->pubkeysize\[0\], kf->pubkey, kf->pubkeyinfo1, kf->sign\_level);  ；就是这儿有 pubkeyinfo  
          
    free(sig);  
    if (status)                                   ！！在这儿修改了。  
      ret = 0; /\* failed \*/  
        
    else  
      ret = (unsigned char \*)"ok";  
    goto exit\_l\_string\_key;  
  
那么这个 l\_ckout\_string\_key 函数与\_l\_pubkey\_verify 函数有什么共同的地方呢？  
如果我们查找 pubkeyinfo 这个变量的话，就发现只有这两个函数才有了。当然可能其他的共同点，我的 C 代码功底比较差。 

pubkey 的 (G, order, ploy)在一起，带 checksum 的。整个数据结构是一常量。  
都是 EC(2^M)有 113， 163， 239 三个 dgree.  
一般 patch G 和 checksum 即可。  
  
P 加密过了的，数据量大些。