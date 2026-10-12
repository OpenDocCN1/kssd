
```
pRDSU=(PIMAGE_RESOURCE_DIR_STRING_U)((DWORD)pbuf+dwAddr);  

#if (defined _WINDOWS) || (defined _CONSOLE)  

  iLen=WideCharToMultiByte(CP_ACP, 0, pRDSU->NameString, pRDSU->Length,Buf, sizeof(Buf), 0, 0);  

  iLen=iLen/2;  

#else  

  //iLen=Clh_Unicode2Char((UNICODE_STRING \*)pRDSU,Buf,sizeof(Buf));  

  iLen=Clh_WChar2Char(pRDSU->NameString,pRDSU->Length,Buf,sizeof(Buf));  

#endif  

for (j=0;j<iLen-1;j++)  

{  

  if (szCompanyName)  
    if (strcmp("CompanyName",&Buf\[j\])==0)  
      strcpy(szCompanyName,&Buf\[j+strlen("CompanyName")+2\]);  

  if (szFileVersion)  
  {  
    if (strcmp("InternalName",&Buf\[j\])==0)  
      strcpy(szFileVersion,&Buf\[j+strlen("InternalName")+1\]);  
    if (strcmp("FileVersion",&Buf\[j\])==0)  
      strcpy(szTemp1,&Buf\[j+strlen("FileVersion")+2\]);  
  }  
}  

szFileVersion\[strlen(szFileVersion)-4\]='\_';  

szFileVersion\[strlen(szFileVersion)-3\]=0;  

strcat(szFileVersion,szTemp1);  

#if (!defined _WINDOWS) && (!defined _CONSOLE)  

  if (szCompanyName)  
    DbgPrint("szCompanyName=%s\\n",szCompanyName);  
  if (szFileVersion)  
    DbgPrint("szFileVersion=%s\\n",szFileVersion);  

#endif  

}  

}//End of if (pIRDE->DataIsDirectory)  

}//End of for (i=0;i <pIRD->  

return 1;  

}  

int PE_Dump_Resource(char *pbuf,BYTE bRAWFile,  

DWORD dwFirstIRD_Offset,  

char *szCompanyName,  

int iCompanyNameLen,  

char *szFileVersion,  

int iFileVersionLen,  

PVA2RAW_REC  pva2raw_rec)  

{  

DWORD dwDiff,dwAddr;  

int i,iCount,iFixupCount;  

PIMAGE_RESOURCE_DIRECTORY    pIRD;  

PIMAGE_RESOURCE_DIRECTORY_ENTRY  pIRDE;  

PIMAGE_RESOURCE_DIR_STRING_U prdsu;  

pIRD=(PIMAGE_RESOURCE_DIRECTORY)((DWORD)pbuf + dwFirstIRD_Offset);  

pIRDE=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)(pIRD +1);  

//DbgPrint("pIRD=%X  pIRDE=%X\\n",pIRD,pIRDE);  

for (i=0;i <pIRD->NumberOfIdEntries+pIRD->NumberOfNamedEntries;i++,pIRDE++)  

{  

if (pIRDE->Id==RESOURCE_ID_VERSION)  

{  

//printf("RESOURCE\_ID\_VERSION\\n");  

if (pIRDE->DataIsDirectory)  

{  

#if (!defined _WINDOWS) && (!defined _CONSOLE)  

DbgPrint("DataIsDirectory Addr=%X\\n",pIRDE->OffsetToDirectory+(DWORD)pIRD);  

#endif  

PE_Dump_Resource_Dir(pbuf,bRAWFile,  

dwFirstIRD_Offset,(PIMAGE_RESOURCE_DIRECTORY)(pIRDE->OffsetToDirectory+(DWORD)pIRD),1,RESOURCE_ID_VERSION,  

szCompanyName,iCompanyNameLen,szFileVersion,iFileVersionLen,  

pva2raw_rec);  

}  

}  

}  

return 1;  

}  

int PE_Get_VersionString(char *pbuf,BYTE bRAWFile,  

char *szCompanyName,  

int iCompanyNameLen,  

char *szFileVersion,  

int iFileVersionLen)  

{  

int i;  

DWORD dwAddr;  

PIMAGE_DOS_HEADER  pIDH;  

PIMAGE_NT_HEADERS  pINH;  

PIMAGE_FILE_HEADER  pIFH;  

PIMAGE_OPTIONAL_HEADER  pIOH;  

PIMAGE_SECTION_HEADER  pISH;  

VA2RAW_REC  PE_va2raw_rec\[50\];  

#ifdef COMPILE_DEBUG1  

DbgPrint("PE_Get_VersionString(%X)\\n",pbuf);  

#endif  

if (PE_Head_Pointer(pbuf,&pIDH,&pINH,&pIFH,&pIOH,&pISH)<0)  

return -1;  

if (bRAWFile)  

{  

memset(&PE_va2raw_rec,0,sizeof(VA2RAW_REC));  

//1:生成反向查找表  

for (i=0;i<pIFH->NumberOfSections;i++)  

{  

PE_va2raw_rec\[i\].VA    =(pISH+i)->VirtualAddress;  

PE_va2raw_rec\[i\].VAEnd  =(pISH+i)->VirtualAddress  +(pISH+i)->Misc.VirtualSize;  

PE_va2raw_rec\[i\].RAW  =(pISH+i)->PointerToRawData;  

PE_va2raw_rec\[i\].RAWEnd  =(pISH+i)->PointerToRawData  +(pISH+i)->Misc.VirtualSize;  

}  

}  

if (pIOH->DataDirectory\[IMAGE_DIRECTORY_ENTRY_RESOURCE\].Size>0)  

{  

if (bRAWFile)  

dwAddr=VA2RAW(pIOH->DataDirectory\[IMAGE_DIRECTORY_ENTRY_RESOURCE\].VirtualAddress,PE_va2raw_rec);  

else  

dwAddr=pIOH->DataDirectory\[IMAGE_DIRECTORY_ENTRY_RESOURCE\].VirtualAddress;  

if (PE_Dump_Resource(pbuf,bRAWFile,dwAddr,  

szCompanyName,iCompanyNameLen,szFileVersion,iFileVersionLen,  

PE_va2raw_rec)<0 )  

{  

#if (!defined _WINDOWS) && (!defined _CONSOLE)  

DbgPrint("PE_Dump_Resource error\\n");  

#endif  

}  

}  

return 1;  

}  

当然，MSDN 也介绍了 API `GetVersionInfo`，但是笔者认为，既然是 ARK，那么对于并不复杂的 API，能自己实现的，还是自己实现为好。以下是 `GetVersionInfo` 的范例  

void GetInternalName()  

{  

// Get the file version for the notepad.  

FileVersionInfo^ myFileVersionInfo = FileVersionInfo::GetVersionInfo( "%systemroot%\\\\Notepad.exe" );  

// Print the internal name.  

textBox1->Text = String::Concat( "Internal name: ", myFileVersionInfo->InternalName );  

}
