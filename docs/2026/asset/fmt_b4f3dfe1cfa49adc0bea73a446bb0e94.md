
# 冲击波1.02源程序 (9千字)

////////////////////////////

/////// *******.vxd ///////

////////////////////////////

```c
#define DEVICE_MAIN

#include "traceit.h"

Declare_Virtual_Device(TRACEIT)

#undef DEVICE_MAIN

TraceitVM::TraceitVM(VMHANDLE hVM) : VVirtualMachine(hVM) {}

TraceitThread::TraceitThread(THREADHANDLE hThread) : VThread(hThread) {}
```

//-----------------------------------------

```c
#define HookNo  0x30

#define HookCode 0x30cd

VMMFault_THUNK        thunkVMMFault;

PMFault_THUNK        thunkPMFault;

DWORD m_Address[3];

WORD  m_Code[3];

WORD  m_AppCode;

BOOL  m_bAppFlag;

DWORD m_OEP;

#define valueEIP          (pcrs->CRS.Client_EIP)

#define mByteEIP(offset)  (*(BYTE *)(valueEIP-(offset)))

#define mWordEIP(offset)  (*(WORD *)(valueEIP-(offset)))

#define mDwordEIP(offset)  (*(DWORD *)(valueEIP-(offset)))
```
//-----------------------------------------

```c
PVOID __stdcall MyFaultHandler(VMHANDLE hVM, PCLIENT_STRUCT pcrs)
{
    if(!m_bAppFlag)
    {
        valueEIP -= 2;
        m_AppCode = *(WORD *)(*(DWORD *)pcrs->CRS.Client_ESP);
        *(WORD *)(*(DWORD *)pcrs->CRS.Client_ESP) = HookCode;
        *(WORD *)m_Address[0] = m_Code[0];
        *(WORD *)m_Address[1] = m_Code[1];
        *(WORD *)m_Address[2] = m_Code[2];
        m_bAppFlag = true;
    }
    else
    {
        valueEIP -= 2;
        mWordEIP(0) = m_AppCode;
        if((mDwordEIP(0-0x05) & 0x0ffffff) == 0x0E9006A)
        {
            _asm
            {
                pusha
                mov ebp, pcrs
                mov esi, [ebp+0x24]
                xor ecx, ecx
                jmp m000001FC
m000001E2:
                mov eax, dword ptr [esi]
                cmp eax, 0x0A302E0C1
                jne m000001FA
                mov m_OEP, esi
                sub m_OEP, 00000005
                jmp m00000201
m000001FA:
                dec esi
                inc ecx
m000001FC:
                cmp ecx, 0x00000070
                jne m000001E2
m00000201:
                popa
            }
        }
        else if((mDwordEIP(0x0f) & 0x0ffffff) == 0x0E80A6A)
        {
            m_OEP = valueEIP-0x0f;
        }
        else if(mDwordEIP(0x08) == 0x0E8006A50 &&
                (mByteEIP(0x0a) == 0x0C3 || mByteEIP(0x0b) == 0x0C3))
        {
            _asm
            {
                pusha
                mov ebp, pcrs
                mov ebx, dword ptr [ebp+0x30]
                mov ebx, dword ptr [ebx+0x04]
                mov esi, 0x0000000A
                sub ebx, esi
                jmp m_0000029C
m_00000282:
                mov eax, dword ptr [ebx]
                cmp eax, 0x083EC8B55
                je m_00000292
                cmp eax, 0x0B9EC8B55
                jne m_0000029A
m_00000292:
                mov m_OEP, ebx
                jmp m_000002A1
m_0000029A:
                inc esi
                dec ebx
m_0000029C:
                cmp esi, 0x00000030
                jne m_00000282
m_000002A1:
                popa
            }
        }
        else if(mDwordEIP(0x0f) == 0x0E800408D &&
                (mByteEIP(0x11) == 0x0C3 || mByteEIP(0x12) == 0x0C3))
        {
            _asm
            {
                pusha
                mov ebp, pcrs
                mov ebx, dword ptr [ebp+0x30]
                mov ebx, dword ptr [ebx]
                mov esi, 0x0000000A
                sub ebx, esi
                jmp m_000002D9
m_000002C6:
                mov eax, dword ptr [ebx]
                cmp eax, 0x083EC8B55
                jne m_000002D7
                mov m_OEP, ebx
                jmp m_000002DE
m_000002D7:
                inc esi
                dec ebx
m_000002D9:
                cmp esi, 0x00000030
                jne m_000002C6
m_000002DE:
                popa
            }
        }
        else if(mWordEIP(0) == 0x0F08B && mDwordEIP(0x0d) == 0x083EC8B55)
        {
            m_OEP = valueEIP-0x0d;
        }
        else if(mWordEIP(0) == 0x0F08B && mDwordEIP(0x2c) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x2c;
        }
        if(mDwordEIP(0) == 0x0D48AD233 && mDwordEIP(0x2c) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x2c;
        }
        else if(mByteEIP(0) == 0x0A3 && mDwordEIP(0x26) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x26-0x06;
        }
        else if(mWordEIP(0) == 0x0E850 && mDwordEIP(0x150) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x150;
        }
        else if(mWordEIP(0) == 0x0E850 && mDwordEIP(0x12e) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x12e;
        }
        else if(mWordEIP(0) == 0x0E850 && mDwordEIP(0x14b) == 0x6AEC8B55)
        {
            m_OEP = valueEIP-0x14b-0x06;
        }
        else if(mWordEIP(0) == 0x0E850 && mDwordEIP(0x0bf) == 0x83EC8B55)
        {
            m_OEP = valueEIP-0x0bf;
        }
        *(WORD *)m_Address[0] = HookCode;
        *(WORD *)m_Address[1] = HookCode;
        *(WORD *)m_Address[2] = HookCode;
        m_bAppFlag = false;
    }
    return NULL;
}
```

```c
BOOL TraceitDevice::OnSysDynamicDeviceInit()
{
    Hook_PM_Fault(HookNo, MyFaultHandler, &thunkPMFault);
    return TRUE;
}

BOOL TraceitDevice::OnSysDynamicDeviceExit()
{
    Unhook_PM_Fault(HookNo, MyFaultHandler, &thunkPMFault);
    return TRUE;
}

DWORD TraceitDevice::OnW32DeviceIoControl(PIOCTLPARAMS pDIOCParams)
{
    switch (pDIOCParams->dioc_IOCtlCode)
    {
        case 1:
            m_Address[0] = *(DWORD *)(pDIOCParams->dioc_InBuf);
            m_Code[0] = *(WORD *)m_Address[0];
            *(WORD *)m_Address[0] = HookCode;
            m_Address[1] = *((DWORD *)(pDIOCParams->dioc_InBuf)+1);
            m_Code[1] = *(WORD *)m_Address[1];
            *(WORD *)m_Address[1] = HookCode;
            m_Address[2] = *((DWORD *)(pDIOCParams->dioc_InBuf)+2);
            m_Code[2] = *(WORD *)m_Address[2];
            *(WORD *)m_Address[2] = HookCode;
            m_bAppFlag = false;
            m_OEP = 0;
            break;
        case 2:
            *(WORD *)m_Address[0] = m_Code[0];
            *(WORD *)m_Address[1] = m_Code[1];
            *(WORD *)m_Address[2] = m_Code[2];
            break;
        case 3:
            *(DWORD *)(pDIOCParams->dioc_OutBuf) = m_OEP;
            pDIOCParams->dioc_cbOutBuf = 4;
            break;
        default:
            break;
    }
    return 0;
}
```

////////////////////////////

/////// *******.exe ///////

////////////////////////////

// Bw2001Dlg.cpp : implementation file
//

```c
#include "stdafx.h"
#include "Bw2001.h"
#include "Bw2001Dlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif
```

```c
/////////////////////////////////////////////////////////////////////////////
// CBw2001Dlg dialog

CBw2001Dlg::CBw2001Dlg(CWnd* pParent /*=NULL*/)
    : CDialog(CBw2001Dlg::IDD, pParent)
{
    //{{AFX_DATA_INIT(CBw2001Dlg)
    //}}AFX_DATA_INIT
    // Note that LoadIcon does not require a subsequent DestroyIcon in Win32
    m_hIcon = AfxGetApp()->LoadIcon(IDR_MAINFRAME);
    hDevice = INVALID_HANDLE_VALUE;
}

void CBw2001Dlg::DoDataExchange(CDataExchange* pDX)
{
    CDialog::DoDataExchange(pDX);
    //{{AFX_DATA_MAP(CBw2001Dlg)
    DDX_Control(pDX, IDC_EDIT1, m_oep);
    DDX_Control(pDX, IDC_BUTTON2, m_stop);
    DDX_Control(pDX, IDC_BUTTON1, m_trace);
    //}}AFX_DATA_MAP
}

BEGIN_MESSAGE_MAP(CBw2001Dlg, CDialog)
    //{{AFX_MSG_MAP(CBw2001Dlg)
    ON_WM_PAINT()
    ON_WM_QUERYDRAGICON()
    ON_BN_CLICKED(IDC_BUTTON1, OnButton1)
    ON_BN_CLICKED(IDC_BUTTON2, OnButton2)
    ON_WM_TIMER()
    //}}AFX_MSG_MAP
```
