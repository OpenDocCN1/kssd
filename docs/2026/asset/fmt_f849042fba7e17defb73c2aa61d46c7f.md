
```
_In_   PVOID InputBuffer,
_In_   ULONG InputBufferLength,
_Out_  PVOID OutputBuffer,
_In_   ULONG OutputBufferLength
)
{
    end = clock();

    if (end - Begin > 100)
    {
        //OutputDebugStringA("in xxx");
        Begin = end;

        if (isFree == FALSE)
        {
            HMODULE aa = GetModuleHandleA("爱冰的沫.DLL");
            ::FreeLibrary(GetModuleHandleA("爱冰的沫.DLL"));
            isFree = TRUE;
        }

        if (is_hook_user == FALSE)
        {
            if (Game_dx.IsCur() == TRUE)
            {
                Hook_dx();
                HookPrsent();
                //HookDrawPrimitive();
                is_hook_user = TRUE;
            }
        }

        SetGame();
    }

    OldNtDeviceIoControlFile = (OrgNtDeviceIoControlFile)oldNtDeviceIoControlFileAdr;

    return OldNtDeviceIoControlFile(
        FileHandle,
        Event,
        ApcRoutine,
        ApcContext,
        IoStatusBlock,
        IoControlCode,
        InputBuffer,
        InputBufferLength,
        OutputBuffer,
        OutputBufferLength);
}

void Hook32()
{
    OutputDebugStringA("hook Kifastcall 32");

    kihook32.Initialization();
    kihook32.InstallHook("NtOpenFile", (DWORD)MyZwOpenFile, &OldNtOpenFile);
    kihook32.InstallHook("NtCreateFile", (DWORD)MyZwCreateFile, &OldZwCreateFile);
    kihook32.InstallHook("NtDeviceIoControlFile", (DWORD)MyNtDeviceIoControlFile, &oldNtDeviceIoControlFileAdr);
}

//////////////////////////////////////////////////////////////////////////

DWORD OldCoCreateGuid_add5 = 0;

typedef HRESULT(__stdcall* OrgCoCreateGuid)(
    GUID* pguid
);

__declspec(naked) HRESULT __stdcall TmpCoCreateGuid(
    GUID* pguid
)
{
    __asm {
        nop
        nop
        nop
        nop
        nop
        nop
        nop
        jmp OldCoCreateGuid_add5
    }
}

unsigned long Data1 = 1;
unsigned short Data2 = 1;
unsigned short Data3 = 1;
unsigned char Data4[8];

HRESULT __stdcall MyCoCreateGuid(
    GUID* pguid
)
{
    OutputDebugStringA("old32");

    GUID GI;
    GI.Data1 = Data1;
    GI.Data2 = Data2;
    GI.Data3 = Data3;
    memcpy(GI.Data4, Data4, 8);

    Data1++;
    Data2++;
    Data3++;
    (BYTE)Data4[0]++;

    return S_OK;

    HRESULT R1 = TmpCoCreateGuid(pguid);
    OutputDebugStringA((char*)pguid);
    return R1;
}

void HookOle32()
{
    BYTE ReSet[5] = {0x8b, 0xff, 0x55, 0x8b, 0xec};
    DWORD OldPro = 0;
    HMODULE m1 = GetModuleHandleA("ole32.dll");

    if (m1 == NULL)
    {
        return;
    }

    DWORD Pro1 = (DWORD)::GetProcAddress(m1, "CoCreateGuid");

    VirtualProtect((BYTE*)Pro1, 5, PAGE_EXECUTE_READWRITE, &OldPro);
    VirtualProtect(TmpCoCreateGuid, 5, PAGE_EXECUTE_READWRITE, &OldPro);

    memcpy((BYTE*)Pro1, ReSet, 5);
    memcpy(TmpCoCreateGuid, (BYTE*)Pro1, 5);

    OldCoCreateGuid_add5 = Pro1 + 5;

    DWORD jmpcode = (DWORD)MyCoCreateGuid - Pro1 - 5;

    *((BYTE*)Pro1) = 0xe9;
    *((DWORD*)(Pro1 + 1)) = jmpcode;
}

//////////////////////////////////////////////////////////////////////////

DWORD OldGetLastInputInfo_add5 = 0;

typedef BOOL(__stdcall* OrgGetLastInputInfo)(
    __out PLASTINPUTINFO plii
);

__declspec(naked) BOOL __stdcall TmpGetLastInputInfo(
    __out PLASTINPUTINFO plii
)
{
    __asm {
        nop
        nop
        nop
        nop
        nop
        nop
        nop
        jmp OldGetLastInputInfo_add5
    }
}

BOOL __stdcall MyGetLastInputInfo(
    __out PLASTINPUTINFO plii
)
{
    plii->dwTime = (GetTickCount() - 5000);
    OutputDebugStringA("lastinput");
    return TRUE;
}

void HookInputTime()
{
    BYTE ReSet[5] = {0x8b, 0xff, 0x55, 0x8b, 0xec};
    DWORD OldPro = 0;
    HMODULE m1 = GetModuleHandleA("user32.dll");

    DWORD Pro1 = (DWORD)::GetProcAddress(m1, "GetLastInputInfo");

    VirtualProtect((BYTE*)Pro1, 5, PAGE_EXECUTE_READWRITE, &OldPro);
    VirtualProtect(TmpGetLastInputInfo, 5, PAGE_EXECUTE_READWRITE, &OldPro);

    memcpy((BYTE*)Pro1, ReSet, 5);
    memcpy(TmpGetLastInputInfo, (BYTE*)Pro1, 5);

    OldGetLastInputInfo_add5 = Pro1 + 5;

    DWORD jmpcode = (DWORD)MyGetLastInputInfo - Pro1 - 5;

    *((BYTE*)Pro1) = 0xe9;
    *((DWORD*)(Pro1 + 1)) = jmpcode;
}

HKL EnumIme(char* InputName, char* FileName)
{
    HKL* HKLList;
    HKL Imehandle;
    int StrCount = 0, FileCount = 0;
    HKLList = NULL;

    UINT uCount = GetKeyboardLayoutList(0, NULL);
    if (0 != uCount)
    {
        HKLList = new HKL[uCount];
        GetKeyboardLayoutList(uCount, HKLList);
        //TRACE("GetKeyboardLayoutList OK!!\n");
    }
    else
    {
        int nErr = GetLastError();
        //        TRACE("Error is %d\n", nErr);
    }

    char strLayoutText[256];
    char strFileText[256];
    char InputString[256];
    char InputFile[256];

    for (UINT i = 0; i < uCount; i++)
    {
        //取得输入法名
        StrCount = ImmGetDescriptionA(HKLList[i], strLayoutText, 256);
        //InputString = &strLayoutText[StrCount];
        strcpy(InputString, &strFileText[StrCount]);

        if (InputString == InputName)
        {
            FileCount = ImmGetIMEFileName(HKLList[i], strFileText, 256);
            strcpy(InputFile, &strFileText[FileCount]);

            if (InputFile == FileName)
            {
                Imehandle = HKLList[i];
                break;
            }
        }
    }

    delete[] HKLList;
    return Imehandle;
}

//获得类虚拟成员函数指针
char Des[1024] = "C:\\";
BOOL IsCur_Bool = FALSE;

void DX_FUNC::Get_SendData()
{
    //OutputDebugStringA("Get_SendData");

    DWORD PID = GetCurrentProcessId();

    char* pid_str;
```
