

# 打造自己的 MyGetProcAddress 和 MyLoadLibrary

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-609.htm`](https://www.pediy.com/kssd/pediy07/pediy7-609.htm)

刚看到一篇打造自己的 MyGetProcAddress 没有看见～MyLoadLibrary  
这次索性都贴出来好了～  
MyGetProcAddress 是利用函数名称的 Crc32 数值搜索出来的  
既然 MyGetProcAddress 好打造了那 MyLoadLibrary 也很好作大了～  
利用用 PEB 获取地址 kernel32.dll 基址然后再用 GetProcAddress 搜出函数指针  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> UNIT APISE;
> {$WARNINGS OFF}
> INTERFACE
> 
> type
> DWORD=LongWord;
>   
> PROCEDURE BuildCRC32Table;
> FUNCTION  CalculateCRC32(VAR Buffer;CONST Size:DWORD) : DWORD;
> FUNCTION MyGetProcAddress(Module:Cardinal;ProcessCRC:DWORD) : Pointer;
> FUNCTION MyLoadLibraryA:Pointer;
> 
> IMPLEMENTATION
> 
> CONST
>   IMAGE\_DIRECTORY\_ENTRY\_EXPORT     = 0;
>   SIZE\_OF\_80387\_REGISTERS          = 80;
>   IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES = 16;
>   MAX\_API\_STRING\_LENGTH            = 150;
>   IMAGE\_DOS\_SIGNATURE              = $5A4D;
>   IMAGE\_NT\_SIGNATURE               = $00004550;
>   MIN\_KERNEL\_SEARCH\_BASE           = $70000000;
> 
> TYPE
>   PImageDosHeader               =^TImageDosHeader;
>   TImageDosHeader               = PACKED RECORD
>     e\_magic                     : WORD;
>     e\_cblp                      : WORD;
>     e\_cp                        : WORD;
>     e\_crlc                      : WORD;
>     e\_cparhdr                   : WORD;
>     e\_minalloc                  : WORD;
>     e\_maxalloc                  : WORD;
>     e\_ss                        : WORD;
>     e\_sp                        : WORD;
>     e\_csum                      : WORD;
>     e\_ip                        : WORD;
>     e\_cs                        : WORD;
>     e\_lfarlc                    : WORD;
>     e\_ovno                      : WORD;
>     e\_res                       : ARRAY \[0..3\] OF WORD;
>     e\_oemid                     : WORD;
>     e\_oeminfo                   : WORD;
>     e\_res2                      : ARRAY \[0..9\] OF WORD;
>     \_lfanew                     : LongInt;
>   END;
> 
>   PImageFileHeader              =^TImageFileHeader;
>   TImageFileHeader              = PACKED RECORD
>     Machine                     : WORD;
>     NumberOfSections            : WORD;
>     TimeDateStamp               : LongWord;
>     PointerToSymbolTable        : LongWord;
>     NumberOfSymbols             : LongWord;
>     SizeOfOptionalHeader        : WORD;
>     Characteristics             : WORD;
>   END;
> 
>   PImageDataDirectory           =^TImageDataDirectory;
>   TImageDataDirectory           = RECORD
>     VirtualAddress              : LongWord;
>     Size                        : LongWord;
>   END;
> 
>   PImageOptionalHeader          =^TImageOptionalHeader;
>   TImageOptionalHeader          = PACKED RECORD
>     Magic                       : WORD;
>     MajorLinkerVersion          : Byte;
>     MinorLinkerVersion          : Byte;
>     SizeOfCode                  : LongWord;
>     SizeOfInitializedData       : LongWord;
>     SizeOfUninitializedData     : LongWord;
>     AddressOfEntryPoint         : LongWord;
>     BaseOfCode                  : LongWord;
>     BaseOfData                  : LongWord;
>     ImageBase                   : LongWord;
>     SectionAlignment            : LongWord;
>     FileAlignment               : LongWord;
>     MajorOperatingSystemVersion : WORD;
>     MinorOperatingSystemVersion : WORD;
>     MajorImageVersion           : WORD;
>     MinorImageVersion           : WORD;
>     MajorSubsystemVersion       : WORD;
>     MinorSubsystemVersion       : WORD;
>     Win32VersionValue           : LongWord;
>     SizeOfImage                 : LongWord;
>     SizeOfHeaders               : LongWord;
>     CheckSum                    : LongWord;
>     Subsystem                   : WORD;
>     DllCharacteristics          : WORD;
>     SizeOfStackReserve          : LongWord;
>     SizeOfStackCommit           : LongWord;
>     SizeOfHeapReserve           : LongWord;
>     SizeOfHeapCommit            : LongWord;
>     LoaderFlags                 : LongWord;
>     NumberOfRvaAndSizes         : LongWord;
>     DataDirectory               : PACKED ARRAY\[0..IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES-1\] OF TImageDataDirectory;
>   END;
> 
>   PImageNtHeaders               =^TImageNtHeaders;
>   TImageNtHeaders               = PACKED RECORD
>     Signature                   : LongWord;
>     FileHeader                  : TImageFileHeader;
>     OptionalHeader              : TImageOptionalHeader;
>   END;
> 
>   PImageExportDirectory         =^TImageExportDirectory;
>   TImageExportDirectory         = PACKED RECORD
>     Characteristics             : LongWord;
>     TimeDateStamp               : LongWord;
>     MajorVersion                : WORD;
>     MinorVersion                : WORD;
>     Name                        : LongWord;
>     Base                        : LongWord;
>     NumberOfFunctions           : LongWord;
>     NumberOfNames               : LongWord;
>     AddressOfFunctions          :^PLongWord;
>     AddressOfNames              :^PLongWord;
>     AddressOfNameOrdinals       :^PWord;
>   END;
> 
> VAR
>   CRC32TAB : ARRAY\[0..255\] OF DWORD;
> 
> PROCEDURE BuildCRC32Table; ASSEMBLER;
> ASM
>     mov  ebx, 0EDB88320h
>     lea  edi, crc32tab
>     xor  ecx, ecx
>   @loc1:
>     mov  eax, ecx
>     mov  edx, 8
>   @loc2:
>     test eax, 1
>     jz  @loc3
>     shr  eax, 1
>     xor  eax, ebx
>     jmp  @loc4
>   @loc3:
>     shr  eax, 1
>   @loc4:
>     dec  edx
>     jnz  @loc2
>     stosd
>     inc  ecx
>     cmp  ecx, 256
>     jb  @loc1
> END;
> 
> FUNCTION CalculateCRC32(VAR Buffer;CONST Size:DWORD) : DWORD; ASSEMBLER;
> ASM
>     push esi
>     push edi
>     push ebx
>     mov edi,edx
>     mov esi,eax
>     xor ebx,ebx
>     mov eax,$ffffffff
>     mov ecx,edi
>     shr ecx,2
>     jecxz @Rest
>   @Loop:
>     mov edx,\[esi\]
>     mov bl,al
>     xor bl,dl
>     shr eax,8
>     xor eax,dword ptr \[CRC32tab+ebx\*4\]
>     mov bl,al
>     xor bl,dh
>     shr eax,8
>     xor eax,dword ptr \[CRC32tab+ebx\*4\]
>     shr edx,16
>     mov bl,al
>     xor bl,dl
>     shr eax,8
>     xor eax,dword ptr \[CRC32tab+ebx\*4\]
>     mov bl,al
>     xor bl,dh
>     shr eax,8
>     xor eax,dword ptr \[CRC32tab+ebx\*4\]
>     add esi,4
>     loop @Loop
>   @Rest:
>     mov ecx,edi
>     and ecx,3
>     jecxz @End
>   @Loop\_Rest:
>     mov bl,al
>     xor bl,\[esi\]
>     shr eax,8
>     inc esi
>     xor eax,dword ptr \[CRC32tab+ebx\*4\]
>     loop @Loop\_Rest
> @End:
>     xor eax,$ffffffff
>     pop ebx
>     pop edi
>     pop esi
> END;
> 
> //------------------------------------------------------------------------------
> FUNCTION MyGetProcAddress(Module:Cardinal;ProcessCRC:DWORD) : Pointer;
> VAR
>   ExportName           : pChar;
>   Address              : Cardinal;
>   J                    : Cardinal;
>   ImageDosHeader       : PImageDosHeader;
>   ImageNTHeaders       : PImageNTHeaders;
>   ImageExportDirectory : PImageExportDirectory;
> BEGIN
>   ImageDosHeader:=Pointer(Module);
>   ImageNTHeaders:=Pointer(Module+ImageDosHeader.\_lfanew);
>   ImageExportDirectory:=Pointer(ImageNtHeaders.OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress+Module);
>   J:=0;
>   Address:=0;
>   REPEAT
>     ExportName:=Pointer(Cardinal(Pointer(Cardinal(ImageExportDirectory.AddressOfNames)+Module+J\*4)^)+Module);
>     IF CalculateCRC32(ExportName^,Length(ExportName))=ProcessCRC THEN
>       Address:=Cardinal(Pointer(Word(Pointer(J SHL 1+Cardinal(
>                ImageExportDirectory.AddressOfNameOrdinals)+Module)^) AND
>                $0000FFFF SHL 2+Cardinal(ImageExportDirectory.AddressOfFunctions)
>                +Module)^)+Module;
>     Inc(J);
>   UNTIL (Address<>0)OR(J=ImageExportDirectory.NumberOfNames);
>   Result:=Pointer(Address);
> END;
> 
> FUNCTION MyLoadLibraryA:Pointer;
> const
>   MyLoadLibraryA=$3FC1BD8D;//LoadLibraryA 字符串的 Crc32 数值
> asm
>   mov eax,fs:$30
>   mov eax,\[eax + $0c\]
>   mov esi,\[eax + $1c\]
>   lodsd
>   mov eax,\[eax+$08\]
>   mov edx,MyLoadLibraryA
>   call GetProcAddress
> end;
> 
> END.
> 
> * * *
> 
> 
> ```

  
  
使用的时候先声明一下 LoadLibrary 过程  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> PROGRAM Project;
> 
> USES
>   APISE;
> 
> CONST
>   MessageBoxA = $572D5D8E; //MessageBoxA 字符串的 Crc32 数值
> 
> VAR
>   MessageBox : FUNCTION(hWnd:DWORD;lpText,lpCaption:PChar;uType:DWORD) : Integer; STDCALL;
>   LoadLibrary:function(lpLibFileName: PChar): HMODULE; stdcall;
> 
> function IntToStr(Value: Integer): String;
> begin
>   Str(Value, Result);
> end;
> 
> BEGIN
>   BuildCRC32Table;
>   @LoadLibrary:=MyLoadLibraryA;
>   MessageBox:=MyGetProcAddress(LoadLibrary('user32.dll'),MessageBoxA);
>   Messagebox(0,Pchar('LoadLibraryA 函数地址为:'+IntToStr(integer(MyLoadLibraryA))),'Message',0);
> END.
> 
> * * *
> 
> 
> ```

  
这个代码是翻译 C 的通用 ShellCode 的代码～～SEH 搜索法。又打雷关电脑 

 4dd466d0e1e0cc123afbe504da2462ca