
# smc 之双进程保护

> 原文：[`www.pediy.com/kssd/pediy11/124875.html`](https://www.pediy.com/kssd/pediy11/124875.html)

文章的思路很简单，/一般的 smc 技术都是程序在运行的时候对自己的代码进行解密，也就是说解密算法可以在原程序找到。然后我就想不想把解密算法放到一个程序里面。

简单的说，有三个程序 A, B, C，C 是待保护的程序

A 对 C 的关键代码进行加密

B 对 C 坐内存补丁，然后运行 C

下面简单的说下代码实现吧

代码：

```
//C 部分
//这里设置一些标志,方便 A 对 C 进行加密
  _asm
  {
//    int 3
        mov eax, 0xAAAAAAAA
    lea eax, EncryptAddrStart
    lea eax, EncryptAddrEnd
  }
EncryptAddrStart:
  
  MessageBox(GetActiveWindow(), TEXT("hello smc"), TEXT("It's just a test"), MB_OK);

EncryptAddrEnd::

//A 部分
//寻找加密标志, 返回偏移
encryptoffset = SearchEncryptFlags(lpBaseImage, encryptFlags, imagesize);
//找到以后就将关键数据保存
encryptoffset += 6;
memcpy(&EncryptAddrStart, (void *)encryptoffset, sizeof(DWORD));
encryptoffset += 6;
memcpy(&EncryptAddrEnd, (void *)encryptoffset, sizeof(DWORD));
//加密算法
//这里只做简单的演示,所以采用简单的异或算法
VOID EncryptCode(DWORD EncryptAddr, DWORD EncryptSize, DWORD key)
{

  for (int i = 0; i < (int)EncryptSize; i++)
  {
    _asm
    {
      mov eax, EncryptAddr       
      mov al, BYTE PTR ds : [eax]
      mov ebx, key
      xor al, bl

      mov esi, EncryptAddr
      mov BYTE ptr ds : [esi], al
    }
    EncryptAddr += 1;
  }
}
//这里将一些关键数据写回,方便 B 对 C 进行解密
  SetFilePointer(hfile, 0, 0, FILE_END);
  EncryptAddrStart = EncryptAddrStart + (DWORD)imagebase - (DWORD)lpBaseImage;
//  EncryptAddrEnd = EncryptAddrEnd + (DWORD)imagebase - (DWORD)lpBaseImage;
  WriteFile(hfile, &EncryptAddrStart, sizeof(DWORD), &bRtn, NULL);
  WriteFile(hfile, &EncryptAddrEnd, sizeof(DWORD), &bRtn, NULL);

//下面是 B 对 C 进行解密部分
//首先获取一些关键信息
hfile = CreateFile(szExeFileName, GENERIC_READ|GENERIC_WRITE, FILE_SHARE_READ|FILE_SHARE_WRITE, NULL,\
                   OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);

  if (INVALID_HANDLE_VALUE == hfile)
  {
    ShowErrorInformation();
    return FALSE;
  }
  SetFilePointer(hfile, -8, NULL, FILE_END);
  ReadFile(hfile, &DecryptAddrStart, sizeof(DWORD), &bRtn, NULL);
  ReadFile(hfile, &DecryptAddrEnd, sizeof(DWORD), &bRtn, NULL);

//然后创建进程
CreateProcess(szExeFileName, NULL, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
//对其进行解密
DeCryptCodeFun(pi.hProcess, (void *)DecryptAddrStart, DecryptSize, key);
  ResumeThread(pi.hThread);
//下面是解密函数
resultcheck = ReadProcessMemory(decryptprocess, addrstart, lpDecryptBuffer, decryptsize, &bRtn);
  if ((FALSE == resultcheck) || (bRtn != decryptsize))
  {
    ShowErrorInformation();
    VirtualFree(lpDecryptBuffer, decryptsize, MEM_DECOMMIT);
    lpDecryptBuffer = NULL;
    return FALSE;
  }
    lpOffset = (DWORD) lpDecryptBuffer;
  for (int i = 0; i < (int)decryptsize; i=i+1)
  {
    _asm
    {
      mov eax, lpOffset       
      mov al, BYTE PTR ds : [eax]
      mov ebx, key
      xor al, bl
       
      mov esi, lpOffset
      mov BYTE ptr ds : [esi], al
    }
    lpOffset = lpOffset + 1;
  }
  resultcheck = WriteProcessMemory(decryptprocess, addrstart, lpDecryptBuffer, decryptsize, &bRtn);
  if ((NULL == resultcheck) || (bRtn != decryptsize))
  {
    ShowErrorInformation();
    VirtualFree(lpDecryptBuffer, decryptsize,MEM_DECOMMIT);
    lpDecryptBuffer = NULL;
    return FALSE;
  }
```

ok... 基本完成... 源码见附件

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `A.rar`
|
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `B.rar`
|
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `C.rar`
|
