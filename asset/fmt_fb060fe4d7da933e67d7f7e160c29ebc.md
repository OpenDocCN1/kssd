
# 获取硬盘 ID/网卡 MAC 的代码

> 原文：[`www.pediy.com/kssd/pediy10/71859.html`](https://www.pediy.com/kssd/pediy10/71859.html)

闲来无事，整理了获取硬盘 ID/网卡 MAC 的源代码。

```c
// =============================================================================

// xID.h - 获取机器 ID - Azithromycin.13 - 2008.05.12

// =============================================================================

#pragma once

#ifndef _X_ID_

#define _X_ID_

typedef struct _IDSECTOR {

  USHORT  wGenConfig;

  USHORT  wNumCyls;

  USHORT  wReserved;

  USHORT  wNumHeads;

  USHORT  wBytesPerTrack;

  USHORT  wBytesPerSector;

  USHORT  wSectorsPerTrack;

  USHORT  wVendorUnique[3];

  CHAR    sSerialNumber[20];

  USHORT  wBufferType;

  USHORT  wBufferSize;

  USHORT  wECCSize;

  CHAR    sFirmwareRev[8];

  CHAR    sModelNumber[40];

  USHORT  wMoreVendorUnique;

  USHORT  wDoubleWordIO;

  USHORT  wCapabilities;

  USHORT  wReserved1;

  USHORT  wPIOTiming;

  USHORT  wDMATiming;

  USHORT  wBS;

  USHORT  wNumCurrentCyls;

  USHORT  wNumCurrentHeads;

  USHORT  wNumCurrentSectorsPerTrack;

  ULONG   ulCurrentSectorCapacity;

  USHORT  wMultSectorStuff;

  ULONG   ulTotalAddressableSectors;

  USHORT  wSingleWordDMA;

  USHORT  wMultiWordDMA;

  BYTE    bReserved[128];

} IDSECTOR, *PIDSECTOR;

extern char xID[ 64 ];

void xGetHardDiskID();

void xGetNetCardID();

#endif

// =============================================================================

// xID.cpp - 获取机器 ID - Azithromycin.13 - 2008.05.12

// =============================================================================

#include "stdafx.h"

#include "xID.h"

#include <winioctl.h>

char xID[ 64 ] = { 0 };

void xGetHardDiskID()

{

  HANDLE hDevice;

  BOOL bResult;

  DWORD dwRet;

  memset( ( void* )xID,0,64 );

  hDevice = CreateFile( "\\\\.\\PhysicalDrive0",GENERIC_READ | GENERIC_WRITE,FILE_SHARE_READ | FILE_SHARE_WRITE,NULL,OPEN_EXISTING,0,NULL  );

  if( hDevice == INVALID_HANDLE_VALUE ) return;

  GETVERSIONINPARAMS vip;

  bResult = DeviceIoControl( hDevice,SMART_GET_VERSION,NULL,0,&vip,sizeof( GETVERSIONINPARAMS ),&dwRet,NULL );

  if( !bResult ) {

    CloseHandle( hDevice );

    return;

  }

  if( vip.bIDEDeviceMap == 0 ) {

    CloseHandle( hDevice );

    return;

  }

  SENDCMDINPARAMS cmdin;

  memset( ( void* )&cmdin,0,sizeof( cmdin ) );

  cmdin.irDriveRegs.bSectorCountReg = 0x01;

  cmdin.irDriveRegs.bSectorNumberReg = 0x01;

  cmdin.irDriveRegs.bFeaturesReg = 0x00;

  cmdin.bDriveNumber = 0x00;

  cmdin.irDriveRegs.bCylLowReg = 0x00;

  cmdin.irDriveRegs.bCylHighReg = 0x00;

  cmdin.irDriveRegs.bDriveHeadReg = 0xa0;

  cmdin.irDriveRegs.bCommandReg = 0xec;

  cmdin.cBufferSize = 0x200;

  BYTE cmdout[ sizeof( SENDCMDOUTPARAMS ) + 512 -1 ];

  memset( ( void* )cmdout,0,sizeof( SENDCMDOUTPARAMS ) + 512 -1 );

  SENDCMDOUTPARAMS* pcmdout = ( SENDCMDOUTPARAMS* )&cmdout;

  pcmdout->cBufferSize = 0x200;

  bResult = DeviceIoControl(  hDevice,SMART_RCV_DRIVE_DATA,( LPVOID )&cmdin,sizeof( SENDCMDINPARAMS ),cmdout,sizeof( cmdout ),&dwRet,NULL );

  if( !bResult ) {

    CloseHandle( hDevice );

    return;

  }

  IDSECTOR* pisd = ( IDSECTOR* )&cmdout[ sizeof( SENDCMDOUTPARAMS ) - 1 ];

  char tmpch1,tmpch2;

  int j = 0;

  for( int i = 0; i < sizeof( pisd->sSerialNumber ); i += 2 ) {

    tmpch1 = pisd->sSerialNumber[ i + 1 ];

    tmpch2 = pisd->sSerialNumber[ i  ];

    if( isdigit( tmpch1 ) || isalpha( tmpch1 ) ) {

      xID[ j++ ] = tmpch1;

    }

    if( isdigit( tmpch2 ) || isalpha( tmpch2 ) ) {

      xID[ j++ ] = tmpch2;

    }

  }

  CloseHandle( hDevice );

}

#include <Iphlpapi.h>

#pragma comment( lib,"Iphlpapi.lib" )

void xGetNetCardID()

{

  PIP_ADAPTER_INFO pinfo;

  unsigned char buf[ 4096 ];

  unsigned long len = 4096;

  unsigned long nError = 0;

  ZeroMemory( buf,4096 );

  pinfo = ( PIP_ADAPTER_INFO )buf;

  nError = GetAdaptersInfo( pinfo,&len );

  if( nError == ERROR_SUCCESS ) {

    sprintf( xID,"%02X%02X%02X%02X%02X%02X",pinfo->Address[0],pinfo->Address[1],pinfo->Address[2],pinfo->Address[3],pinfo->Address[4],pinfo->Address[5] );

  }

}
```

===〉调用相应的函数，`ID` 保存在 `xID` 数组中。

===〉一般地，要从 `PhysicalDrive0`/`PhysicalDrive1`/`PhysicalDrive2`/`PhysicalDrive3` 循环获取。

===〉以下为 `VBScript` 代码

```vbscript
strComputer = "."  'Dot (.) equals local computer in WMI

Set objWMIService = GetObject("winmgmts:\\" & strComputer)

Set colServices = objWMIService.InstancesOf("Win32_PhysicalMedia")

For Each objService In colServices

    WScript.Echo "Win32_PhysicalMedia--Tag         :" & objService.Tag & vbCrLf & _

                 "                   --SerialNumber:" & Trim(objService.SerialNumber) & vbCrLf

Next

Set colServices = objWMIService.InstancesOf("Win32_Processor")

For Each objService In colServices

    WScript.Echo "Win32_Processor--ProcessorId:" & objService.ProcessorId & vbCrLf & _

                 "               --UniqueId   :" & Trim(objService.UniqueId) & vbCrLf

Next

Set colServices = objWMIService.InstancesOf("Win32_NetworkAdapter")

For Each objService In colServices

    WScript.Echo "Win32_NetworkAdapter--MACAddress:" & objService.MACAddress & vbCrLf

Next

Set colServices = objWMIService.InstancesOf("Win32_BIOS")

For Each objService In colServices

    WScript.Echo "Win32_BIOS--SerialNumber :" & objService.SerialNumber & vbCrLf

Next

Set colServices = objWMIService.InstancesOf("Win32_BaseBoard")

For Each objService In colServices

    WScript.Echo "Win32_BaseBoard--Tag         :" & objService.Tag & vbCrLf & _

                 "               --SerialNumber :" & objService.SerialNumber & vbCrLf

Next
```

// 以下代码可以取得系统特征码

//（网卡 `MAC`、硬盘序列号、`CPU` `ID`、`BIOS` 编号）

```c
#include <afx.h>

#include <stdio.h>

#include <iphlpapi.h>

#define  FILE_DEVICE_SCSI                0x0000001b

#define  IOCTL_SCSI_MINIPORT_IDENTIFY   ( ( FILE_DEVICE_SCSI << 16 ) + 0x0501 )

#define  IOCTL_SCSI_MINIPORT 0x0004D008   //  see NTDDSCSI.H for definition

#define  IDENTIFY_BUFFER_SIZE  512

#define  SENDIDLENGTH  ( sizeof( SENDCMDOUTPARAMS ) + IDENTIFY_BUFFER_SIZE )

#define  IDE_ATAPI_IDENTIFY  0xA1  //  Returns ID sector for ATAPI.

#define  IDE_ATA_IDENTIFY    0xEC  //  Returns ID sector for ATA.

#define  DFP_RECEIVE_DRIVE_DATA   0x0007c088

typedef struct _IDSECTOR

{

    USHORT  wGenConfig;

    USHORT  wNumCyls;

    USHORT  wReserved;

    USHORT  wNumHeads;

    USHORT  wBytesPerTrack;

    USHORT  wBytesPerSector;

    USHORT  wSectorsPerTrack;

    USHORT  wVendorUnique[3];

    CHAR    sSerialNumber[20];

    USHORT  wBufferType;

    USHORT  wBufferSize;

    USHORT  wECCSize;

    CHAR    sFirmwareRev[8];

    CHAR    sModelNumber[40];

    USHORT  wMoreVendorUnique;

    USHORT  wDoubleWordIO;

    USHORT  wCapabilities;

    USHORT  wReserved1;

    USHORT  wPIOTiming;

    USHORT  wDMATiming;

    USHORT  wBS;

    USHORT  wNumCurrentCyls;

    USHORT  wNumCurrentHeads;

    USHORT  wNumCurrentSectorsPerTrack;

    ULONG   ulCurrentSectorCapacity;

    USHORT  wMultSectorStuff;

    ULONG   ulTotalAddressableSectors;

    USHORT  wSingleWordDMA;

    USHORT  wMultiWordDMA;

    BYTE    bReserved[128];

} IDSECTOR, *PIDSECTOR;

typedef struct _DRIVERSTATUS

{

    BYTE  bDriverError;  //  Error code from driver, or 0 if no error.

    BYTE  bIDEStatus;    //  Contents of IDE Error register.

    //  Only valid when bDriverError is SMART_IDE_ERROR.

    BYTE  bReserved[2];  //  Reserved for future expansion.

    DWORD  dwReserved[2];  //  Reserved for future expansion.
```
