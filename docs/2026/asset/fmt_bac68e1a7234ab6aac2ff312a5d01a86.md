
# [原创]分享个bios虚拟机检测

```c
typedef UINT (__stdcall *EnumSystemFirmwareTables)(DWORD, PVOID, DWORD); 

typedef UINT (__stdcall *GetSystemFirmwareTable)(DWORD, DWORD, PVOID, DWORD);

HMODULE g_hKernel32 = GetModuleHandle("kernel32.dll");

EnumSystemFirmwareTables pFunEnumSystemFirmwareTables = (EnumSystemFirmwareTables)GetProcAddress(g_hKernel32, "EnumSystemFirmwareTables");

GetSystemFirmwareTable pGetSystemFirmwareTable = (GetSystemFirmwareTable)GetProcAddress(g_hKernel32, "GetSystemFirmwareTable"); 

char Virtualbox[] = "Virtualbox";

char VMware[] = "VMware";

char Parallels[] = "Parallels";

int main(int argc, char* argv[])

{

    bool isVM = false;

    int nCount = pFunEnumSystemFirmwareTables('FIRM', 0, 0);

    int *pTables = (int*)new char[nCount];

    pFunEnumSystemFirmwareTables('FIRM', pTables, nCount);

    for (size_t i = 0; i < nCount / sizeof(int); i++)

    {

        int nSize = pGetSystemFirmwareTable('FIRM', pTables[i], 0, 0);

        char *pBuf = new char[nSize];

        pGetSystemFirmwareTable('FIRM', pTables[i], pBuf, nSize);

        for (size_t i = 0, iv = 0, iw = 0, ip = 0; i < nSize; i++)

        {

            if (iv == strlen(Virtualbox) || iw == strlen(VMware) || ip == strlen(Parallels))

            {

                MessageBox(0, "found vm", 0, 0);

                break;

            }

            if (pBuf[i] == Virtualbox[iv])

                iv++;

            else

                iv = 0;

            if (pBuf[i] == VMware[iw])

                iw++;

            else

                iw = 0;

            if (pBuf[i] == Parallels[ip])

                ip++;

            else

                ip = 0;

        }

        delete pBuf; 

    }

    delete pTables;

	return 0;

}
```

## Requirements

|  |  |
| --- | --- |
| Minimum supported client | Windows Vista, Windows XP Professional x64 Edition [desktop apps only] |
| Minimum supported server | Windows Server 2008, Windows Server 2003 with SP1 [desktop apps only] |

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
