

# 打造 MyGetProcAddress

> 原文：[`www.pediy.com/kssd/pediy06/pediy6630.htm`](https://www.pediy.com/kssd/pediy06/pediy6630.htm)

function MyGetProcAddress(hModule: HMODULE; lpProcName: LPCSTR): FARPROC; stdcall;  
  
function MyGetProcAddress(hModule: HMODULE; lpProcName: LPCSTR): FARPROC;  
var  
  DataDirectory: TImageDataDirectory;  
  P1: ^Cardinal;  
  P2: ^Word;  
  Base, NumberOfNames, AddressOfFunctions, AddressOfNames, AddressOfNameOrdinals, i, Ordinal: Cardinal;  
  TempStr1, TempStr2: string;  
begin  
  Result := nil;  
  DataDirectory := PImageNtHeaders(Cardinal(hModule) + Cardinal(PImageDosHeader(hModule)^.\_lfanew))^.OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\];  
  P1 := Pointer(hModule + DataDirectory.VirtualAddress + 16);  
  Base := P1^; //输出函数的起始序号。一般为 1。  
  P1 := Pointer(hModule + DataDirectory.VirtualAddress + 24);  
  NumberOfNames := P1^; //输出函数名的指针的数组中的元素个数，也是输出函数名对应的序号的数组中的元素个数。  
  P1 := Pointer(hModule + DataDirectory.VirtualAddress + 28);  
  AddressOfFunctions := P1^; //一个 RVA，指向输出函数入口地址的数组。  
  P1 := Pointer(hModule + DataDirectory.VirtualAddress + 32);  
  AddressOfNames := P1^; //一个 RVA，指向输出函数名的指针的数组。  
  P1 := Pointer(hModule + DataDirectory.VirtualAddress + 36);  
  AddressOfNameOrdinals := P1^; //一个 RVA，指向输出函数名对应的序号的数组。  
  Ordinal := 0;  
  if Cardinal(lpProcName) > $0000FFFF then  
  begin  
    //lpProcName 参数指向函数名  
    TempStr1 := PChar(lpProcName); //要找的函数名  
    for i := 1 to NumberOfNames do  
    begin  
      //按顺序在输出函数名中找  
      P1 := Pointer(hModule + AddressOfNames + (i - 1) \* 4);  
      TempStr2 := PChar(hModule + P1^); //当前输出函数名  
      if TempStr1 = TempStr2 then  
      begin  
        //找到输出函数  
        P2 := Pointer(hModule + AddressOfNameOrdinals + (i - 1) \* 2);  
        //获得序号，不必减 Base  
        Ordinal := P2^;  
        Break;  
      end;  
    end;  
  end  
  else  
    //lpProcName 传过来的是序号，需减 Base  
    Ordinal := Cardinal(lpProcName) - Base;  
  P1 := Pointer(hModule + AddressOfFunctions + Ordinal \* 4); //P1^为函数入口 RVA  
  if (P1^ >= DataDirectory.VirtualAddress) and (P1^ <= DataDirectory.VirtualAddress + DataDirectory.Size) then  
  begin  
    //！！！入口 RVA 在输出表中，指向另一 DLL 的某函数，这一点很容易被忽视，很少有教程提到，也许是没仔细看！！！  
    TempStr1 := PChar(hModule + P1^); //DLL.函数  
    TempStr2 := TempStr1;  
    while Pos('.', TempStr2) > 0 do  
      TempStr2 := Copy(TempStr2, Pos('.', TempStr2) + 1, Length(TempStr2) - Pos('.', TempStr2));  
    TempStr1 := Copy(TempStr1, 1, Length(TempStr1) - Length(TempStr2) - 1); //TempStr1 是 DLL 名，TempStr2 是函数名  
    //递归调用获取新的函数地址  
    Base := GetModuleHandle(PChar(TempStr1));  
    if Base = 0 then  
      Base := LoadLibrary(PChar(TempStr1));  
    if Base > 0 then  
      Result := MyGetProcAddress(Base, PChar(TempStr2));  
  end  
  else  
    //RVA+基址就是函数的真实入口了  
    Result := Pointer(hModule + P1^);  
  //结果 Result := GetProcAddress(hModule, lpProcName);  
end;  
  
相关：  
  \_IMAGE\_DATA\_DIRECTORY = record  
    VirtualAddress: DWORD;  
    Size: DWORD;  
  end;  
  {$EXTERNALSYM \_IMAGE\_DATA\_DIRECTORY}  
  TImageDataDirectory = \_IMAGE\_DATA\_DIRECTORY;  
  
  PImageExportDirectory = ^TImageExportDirectory;  
  \_IMAGE\_EXPORT\_DIRECTORY = packed record  
      Characteristics: DWord;  
      TimeDateStamp: DWord;  
      MajorVersion: Word;  
      MinorVersion: Word;  
      Name: DWord;  
      Base: DWord;  
      NumberOfFunctions: DWord;  
      NumberOfNames: DWord;  
      AddressOfFunctions: ^PDWORD;  
      AddressOfNames: ^PDWORD;  
      AddressOfNameOrdinals: ^PWord;  
  end;  
  {$EXTERNALSYM \_IMAGE\_EXPORT\_DIRECTORY}  
  TImageExportDirectory = \_IMAGE\_EXPORT\_DIRECTORY;  
  
这是我用来在 EncryptPE 新版壳中处理 IAT 的源代码，98、2003 下测试通过。如果你发现问题请告诉我！  
假如你是 Cracker，看到这段代码，你准备如何设断，避开 IAT 加密呢？  
  
EncryptPE 作者老王(wfs)  
2004.5.6

同一函数，我用在 VB\_Shell 里面的。  
Function GetProcAddressDirectly(ByVal lpImageDosHeader As IMAGE\_DOS\_HEADER Ptr, FuncName As Asciiz) As Dword  
    Dim lpImageNtHeaders          As Local IMAGE\_NT\_HEADERS Ptr  
    Dim lpImageExportDirectory    As Local IMAGE\_EXPORT\_DIRECTORY Ptr  
    Dim lpNameOrdinals            As Local Word Ptr  
    Dim lpFunctions               As Local Dword Ptr  
    Dim lpName                    As Local Dword Ptr  
    Dim lpExpFuncName             As Local Asciiz Ptr  
    Dim i                         As Local Dword  
    Dim j                         As Local Dword  
    Dim lpFuncName                As Asciiz Ptr  
  
  
    If @lpImageDosHeader.e\_magic <> %IMAGE\_DOS\_SIGNATURE Then ' invalid DOS signature  
        Function = 1  
        MsgBox "Invalid DOS signature",%MB\_ICONWARNING,"Error in GetProcAddressDirectly()"  
        Exit Function  
    End If  
    lpImageNtHeaders = lpImageDosHeader + @lpImageDosHeader.e\_lfanew  
   '================================  
  
    If @lpImageNtHeaders.Signature <> %IMAGE\_NT\_SIGNATURE Then ' Invalid NT signature  
        Function = 1  
        MsgBox "Invalid NT signature",%MB\_ICONWARNING,"Error in GetProcAddressDirectly()"  
        Exit Function  
    End If  
   '================================  
  
    If @lpImageNtHeaders.FileHeader.SizeOfOptionalHeader <> SizeOf(@lpImageNtHeaders.OptionalHeader) Or \_  
        @lpImageNtHeaders.OptionalHeader.Magic <> %IMAGE\_NT\_OPTIONAL\_HDR32\_MAGIC Then  
        Function = 0  
        MsgBox "SizeOfOptionalHeader or OptionalHeader.Magic",%MB\_ICONWARNING,"Error in GetProcAddressDirectly()"  
        Exit Function  
    End If  
    lpImageExportDirectory = @lpImageNtHeaders.OptionalHeader.DataDirectory(%IMAGE\_DIRECTORY\_ENTRY\_EXPORT).VirtualAddress  
   '================================  
  
    If lpImageExportDirectory = 0 Then  
        Function = 0  
        MsgBox "lpImageExportDirectory",%MB\_ICONWARNING,"Error in GetProcAddressDirectly()"  
        Exit Function  
    End If  
    lpImageExportDirectory = lpImageExportDirectory + lpImageDosHeader  
   '================================  
  
    lpNameOrdinals = @lpImageExportDirectory.AddressOfNameOrdinals + lpImageDosHeader  
    lpName         = @lpImageExportDirectory.AddressOfNames        + lpImageDosHeader  
    lpFunctions    = @lpImageExportDirectory.AddressOfFunctions    + lpImageDosHeader  
   '================================  
  
    lpFuncName = VarPtr(FuncName)  
   '================================  
  
    If HiWrd(lpFuncName) Then ' Name  
       For i = 0 To @lpImageExportDirectory.NumberOfFunctions - 1  
          If @lpFunctions\[i\] Then  
             For j = 0 To @lpImageExportDirectory.NumberOfNames - 1  
                If @lpNameOrdinals\[j\] = i Then  
                   lpExpFuncName = @lpName\[j\] + lpImageDosHeader  
                   If @lpExpFuncName = FuncName Then Function = @lpFunctions\[i\] + lpImageDosHeader : Exit Function  
                End If  
             Next  
          End If  
       Next  
  
    Else  
       For i = 0 To @lpImageExportDirectory.NumberOfFunctions - 1  
          If lpFuncName = @lpImageExportDirectory.nBase + i Then  
             If  @lpFunctions\[i\] Then Function = @lpFunctions\[i\] + lpImageDosHeader  
             Exit Function  
          End If  
       Next  
    End If  
   '================================  
  
End Function  
  
ＶＣ：  
DWORD GetProcAddressDirectly(PIMAGE\_DOS\_HEADER lpImageDosHeader, char \* FuncName)  
{  
    PIMAGE\_NT\_HEADERS lpImageNtHeaders;  
    PIMAGE\_EXPORT\_DIRECTORY pExportDir;  
    PWORD lpNameOrdinals;  
    unsigned char \* lpFunctions;  
    DWORD \* lpName;  
    char \* lpExpFuncName;  
    DWORD i;  
    DWORD j;  
    char \* lpFuncName;  
  
    if(lpImageDosHeader->e\_magic != IMAGE\_DOS\_SIGNATURE)  
  {  
        MessageBox(NULL,"Invalid DOS signature",0,0);  
        return 1;  
  }  
      
    lpImageNtHeaders = (PIMAGE\_NT\_HEADERS)(lpImageDosHeader + lpImageDosHeader->e\_lfanew);  
    
    if (lpImageNtHeaders->Signature != IMAGE\_NT\_SIGNATURE)  
  {  
    MessageBox(NULL,"Invalid NT signature",0,0);  
        return 1;  
    }  
    
    if ((lpImageNtHeaders->FileHeader.SizeOfOptionalHeader != sizeof(lpImageNtHeaders->OptionalHeader)) ||  
        (lpImageNtHeaders->OptionalHeader.Magic != IMAGE\_NT\_OPTIONAL\_HDR32\_MAGIC))  
  {  
    MessageBox(NULL,"SizeOfOptionalHeader or OptionalHeader->Magic",0,0);  
        return 0;  
    }  
  
  DWORD exportsStartRVA, exportsEndRVA;  
    PIMAGE\_NT\_HEADERS pNTHeader;  
  //PIMAGE\_NT\_HEADERS64 pNTHeader64;  
    PBYTE pImageBase = (PBYTE)lpImageDosHeader;  
      
  // Make pointers to 32 and 64 bit versions of the header.  
    pNTHeader = MakePtr( PIMAGE\_NT\_HEADERS, lpImageDosHeader,  
                                lpImageDosHeader->e\_lfanew );  
  
  
    exportsStartRVA = GetImgDirEntryRVA(pNTHeader,IMAGE\_DIRECTORY\_ENTRY\_EXPORT);  
    exportsEndRVA = exportsStartRVA +  
             GetImgDirEntrySize(pNTHeader, IMAGE\_DIRECTORY\_ENTRY\_EXPORT);  
  
    // Get the IMAGE\_SECTION\_HEADER that contains the exports.  This is  
    // usually the .edata section, but doesn't have to be.  
  PIMAGE\_SECTION\_HEADER header;  
    header = GetEnclosingSectionHeader( exportsStartRVA, pNTHeader );  
    if ( !header ) return 0;  
  
  INT delta;  
    delta = (INT)(header->VirtualAddress - header->PointerToRawData);  
    pExportDir = (PIMAGE\_EXPORT\_DIRECTORY)GetPtrFromRVA(exportsStartRVA, pNTHeader, pImageBase);  
  
    
    pExportDir =(PIMAGE\_EXPORT\_DIRECTORY) (lpImageNtHeaders->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);  
    
    if (pExportDir == 0)  
  {  
    MessageBox(NULL,"Error in GetProcAddressDirectly()",0,0);  
        return 0;  
    }  
    
    pExportDir =(PIMAGE\_EXPORT\_DIRECTORY) (DWORD)pExportDir + (DWORD)lpImageDosHeader;  
    
    lpNameOrdinals =(unsigned short \*)(pExportDir->AddressOfNameOrdinals + lpImageDosHeader);  
    //lpNameOrdinals =  (PWORD)  GetPtrFromRVA( pExportDir->AddressOfNameOrdinals, pNTHeader, pImageBase );  
    lpName         =(DWORD \*) (pExportDir->AddressOfNames        + lpImageDosHeader);  
    lpFunctions    =(unsigned char \*) (pExportDir->AddressOfFunctions    + lpImageDosHeader);  
  
  //PDWORD pdwFunctions;  
  
    lpFuncName = FuncName;  
    
    if(HIWORD(lpFuncName)!=0 )  
  {  
    for( i = 0;i<=pExportDir->NumberOfFunctions - 1;i++)  
    {  
      DWORD entryPointRVA = \*lpFunctions;  
        
      if ( entryPointRVA == 0 ) continue;  // Skip over gaps in exported function  
                                         // ordinals (the entrypoint is 0 for  
        
      for( j = 0;j<=pExportDir->NumberOfNames - 1;j++)  
      {  
        if( lpNameOrdinals\[j\] == i)  
        {  
          lpExpFuncName = (char \*) (lpName\[j\] + lpImageDosHeader);  
          if(lpExpFuncName = FuncName)  return (unsigned long) (lpFunctions\[i\] + lpImageDosHeader);  
        }  
      }  
    }  
  }  
    else  
  {  
    for (i = 0 ;i<=pExportDir->NumberOfFunctions - 1;i++)  
    {  
      if (lpFuncName == (char \*)(pExportDir->Base + i))  
      {  
        if (lpFunctions\[i\]) return (unsigned long) (lpFunctions\[i\] + lpImageDosHeader);  
      }  
    }  
    }  
  return 0;  
}

function MyGetProcAddress(hModule: HMODULE; lpProcName: LPCSTR): FARPROC; stdcall  
将导致调用的时候是  
push lpProcName  
push hModule  
call MyGetProcAddress  
形式  
一般我会断 LoadLibrary 函数（此函数我想一般人不会自己去实现它），然后  
观察对 kenerl32.dll 等重要函数的 loalibray.得到的返回值，下对这个值得  
内存访问断点.很容易找到你实现的 MyGetProcAddress 函数，一开始可能看不出来但是看几次堆栈后 ret 就知道你这个玩意就是模仿 getprocaddress 了，

 4dd466d0e1e0cc123afbe504da2462ca