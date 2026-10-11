
# 代码实现与逻辑分析

以下代码片段展示了 `SetStrInput` 函数及资源处理函数 `ChageRes` 的实现。

```cpp
__declspec(naked) void SetStrInput()
{
  __asm
  {
    pushad
    pushfd
    mov pwstr,ecx
  }
  WritePrivateProfileStringW(_T("KEY"),_T("register"),pwstr,allEnvir.wscIniFileName);
  __asm
  {
    popfd
    popad
    mov eax,0x40753e
    push eax
    mov eax,0x407900
    jmp eax
  }
}

void add()
{
  return;
}

void ChageRes()
{
   HANDLE hFile = CreateFileA(allEnvir.szResFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
   SetFilePointer(hFile,0xd4000,NULL,FILE_BEGIN);
   DWORD dwRead = 0;
   DWORD dwReadLen = 0x10;
   DWORD dwWriteAddr = 0x4d4000;
   char szBuffer[0x11];
   for(int i = 0;i<0x4cff0;i=i+0x10)
   {
     memset(szBuffer,0,0x11);
     ReadFile(hFile,szBuffer,dwReadLen,&dwRead,NULL);    
     GetLastError();
     memcpy((void*)dwWriteAddr,szBuffer,0x10);
     dwWriteAddr=dwWriteAddr +0x10;
   }
}
```

## 注册挂接分析

代码写得比较乱，但核心逻辑如下。在注册部分，直接挂接程序的 `0x00407539` 地址：

```asm
00407530  |.  894C24 18     mov     dword ptr [esp+18], ecx
00407534  |.  E8 11470600   call    0046BC4A
00407539  |.  E8 C2030000   call    00407900                       ;  getstring
0040753E  |.  8BDF          mov     ebx, edi
00407540  |.  E8 3B030000   call    00407880
```

将其替换为上述 `SetStrInput` 函数。这个函数的本意是复用原有的注册框，将用户输入的注册码提取出来，然后写入到一个配置文件中。

## 验证逻辑

验证部分的代码如下：

```cpp
DWORD* addr1 =(DWORD*) 0x4d2718;
DWORD* addr2 = (DWORD*)(*(addr1)+0x24);
//MessageBoxW(NULL,(LPCTSTR)(*(addr2)),(LPCTSTR)(*(addr2)),MB_OK);
WCHAR *wszDiskData = (WCHAR*)(*(addr2));
USES_CONVERSION;
char szoutmd5[33];
memset(szoutmd5,0,33);
char* pstring =  W2A(wszDiskData);
pstring[0] = 'U';
pstring[1] = 'E';
pstring[3] = pstring[3]+1;
pstring[4] = pstring[4]+3;
pstring[5] = pstring[5]+2;
char* pstrmd5 = MD5String(pstring,szoutmd5);
char szBuffer[MAX_PATH];
GetPrivateProfileStringA("KEY","register","yeallright",szBuffer,MAX_PATH,allEnvir.szIniFileName);
if (stricmp(szBuffer,pstrmd5)!=0)
{
  MessageBoxA(NULL,"注册码错误，请从新联系 QQ：516674080 注册使用","注册码错误，请从新联系 QQ：516674080 注册使用",MB_OK);
  //ExitProcess(0);
  return 0;
}
```

这段代码是验证核心。首先通过分析程序验证逻辑中取硬件特征码的固定偏移量：

```cpp
DWORD* addr1 =(DWORD*) 0x4d2718;
DWORD* addr2 = (DWORD*)(*(addr1)+0x24);
```

此处获取的地址即为机器码。经过处理后，与配置文件中取出的注册码进行 `stricmp` 比较即可。（是否有更简单的方法？能否不写代码就生成注册过程？还是觉得这样太麻烦了。）

## 资源替换策略

接下来是处理资源。由于程序加了壳且未完全脱壳，不能在静态阶段替换资源（因为静态时资源尚未解密）。

因此，策略是“能猥琐就猥琐”，直接在程序动态解密完成后，替换内存中的资源。将其替换为使用 `eXeScope` 修改后的资源。代码如下：

```cpp
void ChageRes()
{
   HANDLE hFile = CreateFileA(allEnvir.szResFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
   SetFilePointer(hFile,0xd4000,NULL,FILE_BEGIN);
   DWORD dwRead = 0;
   DWORD dwReadLen = 0x10;
   DWORD dwWriteAddr = 0x4d4000;
   char szBuffer[0x11];
   for(int i = 0;i<0x4cff0;i=i+0x10)
   {
     memset(szBuffer,0,0x11);
     ReadFile(hFile,szBuffer,dwReadLen,&dwRead,NULL);    
     GetLastError();
     memcpy((void*)dwWriteAddr,szBuffer,0x10);
     dwWriteAddr=dwWriteAddr +0x10;
   }
}
```

其中 `0x4d4000` 是资源节，文件的 `0xd4000` 也是资源节。基本上资源部分的替换使用 `eXeScope` 即可完成。

另外，将该 `dll` 感染到目标 `pe` 中即可。直接使用 `LoadPE` 就完成了此功能。

--------------------------------------------------------------------------------
## 经验总结

字就码完了。感觉没啥可说的。近来全走的是猥琐路子，没沉淀。最近在做虚拟机还原的一些积累，还做得不好，都还在学习阶段。完了一起分享起。

--------------------------------------------------------------------------------
【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2011 年 02 月 01 日 1:50:33

[`bbs.pediy.com/attachment.php?...1&d=1296497066`](http://bbs.pediy.com/attachment.php?attachmentid=53895&stc=1&d=1296497066)

[`bbs.pediy.com/attachment.php?...1&d=1296497153`](http://bbs.pediy.com/attachment.php?attachmentid=53896&stc=1&d=1296497153)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `OhYe.rar`
|
