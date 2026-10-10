

# 又一次写 GetProcAddress 函数

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-521.htm`](https://www.pediy.com/kssd/pediy09/pediy09-521.htm)

> 函数名是想起了以前看过的哪个大牛的文章想到的.这两天把 PeCancer 的 IAT 处理部分代码重新写过了,从代码里提取出了这个函数,实际壳中函数比这个复杂,不过都是用于壳完成功能所以去掉了.  
> 大家共同学习,也希望能帮找找 bug  
> DWORD GetFunctionAddress( HMODULE phModule,char\* pProcName )  
> {  
>   if (!phModule)  
>     return  0;  
>   PIMAGE\_DOS\_HEADER  pimDH  =  (PIMAGE\_DOS\_HEADER)phModule;  
>   PIMAGE\_NT\_HEADERS  pimNH  =  (PIMAGE\_NT\_HEADERS)((char\*)phModule+pimDH->e\_lfanew);  
>   PIMAGE\_EXPORT\_DIRECTORY  pimED  =  (PIMAGE\_EXPORT\_DIRECTORY)((DWORD)phModule+pimNH->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);  
>   DWORD  pExportSize  =  pimNH->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].Size;  
>   DWORD  pResult  =  0;  
>   
>     
>   if ((DWORD)pProcName < 0x10000)  
>   {  
>     if ((DWORD)pProcName  >=  pimED->NumberOfFunctions+pimED->Base || (DWORD)pProcName < pimED->Base)  
>       return  0;  
>     pResult  =  (DWORD)phModule+((DWORD\*)((DWORD)phModule+pimED->AddressOfFunctions))\[(DWORD)pProcName-pimED->Base\];  
>   }else  
>   {  
>     DWORD\*  pAddressOfNames  =  (DWORD\*)((DWORD)phModule+pimED->AddressOfNames);  
>     for (int i=0;i<pimED->NumberOfNames;i++)  
>     {  
>       char\*  pExportName  =  (char\*)(pAddressOfNames\[i\]+(DWORD)phModule);  
>       if (strcmp(pProcName,pExportName) == 0)  
>       {  
>         WORD\*  pAddressOfNameOrdinals  =  (WORD\*)((DWORD)phModule+pimED->AddressOfNameOrdinals);  
>         pResult    =  (DWORD)phModule+((DWORD\*)((DWORD)phModule+pimED->AddressOfFunctions))\[pAddressOfNameOrdinals\[i\]\];  
>         break;  
>       }  
>     }  
>   }  
>   if  (pResult != 0 && pResult  >=  (DWORD)pimED  &&  pResult <  (DWORD)pimED+pExportSize)  
>   {  
>     char\*  pDirectStr  =  (char\*)pResult;  
>     bool  pstrok  =  false;  
>     while (\*pDirectStr)  
>     {  
>       if (\*pDirectStr == '.')  
>       {  
>         pstrok  =  true;  
>         break;  
>       }  
>       pDirectStr++;  
>     }  
>     if (!pstrok)  
>       return  0;  
>     char  pdllname\[MAX\_PATH\];  
>     int    pnamelen  =  pDirectStr-(char\*)pResult;  
>     if (pnamelen <= 0)  
>       return  0;  
>     memcpy(pdllname,(char\*)pResult,pnamelen);  
>     pdllname\[pnamelen\] = 0;  
>     HMODULE  phexmodule  =  GetModuleHandle(pdllname);  
>     pResult  =  GetFunctionAddress(phexmodule,pDirectStr+1);  
>   }  
>   
>   return pResult;  
> } 

 4dd466d0e1e0cc123afbe504da2462ca