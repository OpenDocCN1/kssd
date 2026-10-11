
# RVA 与 Offset 的换算函数

> 原文：[`www.pediy.com/kssd/pediy10/67104.html`](https://www.pediy.com/kssd/pediy10/67104.html)

```
function RVAToFileOffset(FileName:string; RVA: Cardinal): Cardinal;  

var  

  MemPE: TFileStream;  

  PEDosHead: TImageDosHeader;  

  PENtHead: TImageNtHeaders;  

  Section : TImageSectionHeader;  

  i, SectionsCount: Integer;  

begin  

  Result := RVA;  

  MemPE:=TFileStream.Create(FileName,fmOpenReadWrite);  

  try  

    MemPE.Seek(0, soFromBeginning);  

    MemPE.Read(PEDosHead, SizeOf(PEDosHead));  

    MemPE.Seek(PEDosHead.\_lfanew, soFromBeginning);  

    MemPE.Read(PENtHead, SizeOf(PENtHead));  

    SectionsCount := PENtHead.FileHeader.NumberOfSections;  

    if SectionsCount <> 0 then  

      for i := 0 to SectionsCount - 1 do  

      begin  

        MemPE.Read(Section, SizeOf(Section));  

        if (RVA >= Section.VirtualAddress) and (RVA < Section.VirtualAddress + Section.SizeOfRawData) then  

        begin  

          Result := RVA - Section.VirtualAddress + Section.PointerToRawData;  

          Break;  

        end;  

      end;  

  finally  

    FreeAndNil(MemPE);  

  end;  

end;  

function FileOffsetToRVA(FileName:string; Offset: Cardinal): Cardinal;  

var  

  MemPE: TFileStream;  

  PEDosHead: TImageDosHeader;  

  PENtHead: TImageNtHeaders;  

  Section : TImageSectionHeader;  

  i, SectionsCount: Integer;  

begin  

  Result :=Offset;  

  MemPE:=TFileStream.Create(FileName,fmOpenReadWrite);  

  try  

    MemPE.Seek(0, soFromBeginning);  

    MemPE.Read(PEDosHead, SizeOf(PEDosHead));  

    MemPE.Seek(PEDosHead.\_lfanew, soFromBeginning);  

    MemPE.Read(PENtHead, SizeOf(PENtHead));  

    SectionsCount := PENtHead.FileHeader.NumberOfSections;  

    if SectionsCount <> 0 then  

      for i := 0 to SectionsCount - 1 do  

      begin  

        MemPE.Read(Section, SizeOf(Section));  

        if (Offset >= Section.PointerToRawData) and (Offset < Section.PointerToRawData + Section.SizeOfRawData) then  

        begin  

          Result := Offset - Section.PointerToRawData + Section.VirtualAddress;  

          Break;  

        end;  

      end;  

  finally  

    FreeAndNil(MemPE);  

  end;  

end;
```
