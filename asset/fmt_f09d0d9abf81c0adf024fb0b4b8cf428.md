
```pascal
PhysicalSize: Integer;

// VirtualSize: Integer;
// Characteristics: Cardinal;
PointerToRawData: Integer;
end;

TNameOrID = (niName, niID);

TPEImport = record
  NameOrID: TNameOrID;
  Name: string;
  ID: Integer;
//  PAddress: PChar; {指向导入表函数用于执行此函数}
end;

TPEImports = record {记录一个 Dll 文件所调用的函数个数}
  DLLName: string;
  Entries: array of TPEImport; {函数数据}
end;

TPEExport = record
  Name: string;
  RelativeID: Integer;
  ID: Integer;
  Address: DWORD;{相对地址}
end;


TPEHandler = class(TThread)
private
  { Private declarations }
  FPE, FTmp: TMemoryStream;
  FSectionFiles: array[0..3] of string;
  FSectionSizes: array[0..3] of integer;
  FImportCount: integer;
  FImportVA: integer;
  FIAT: integer;
  FIATSize: integer;
  procedure HandlerImport;
  function GetEntryPoint: integer;
  procedure GetPEHead;
  Procedure FillSection;
  procedure ChangPEHead;
  procedure ShowInfo(Info: string);
protected
  procedure Execute; override;
end;

implementation

uses uMain;

{ TPEHandler }

procedure TPEHandler.ChangPEHead;
var
  ImageDosHeader:     TImageDosHeader;
  ImageNtHeaders:     TImageNtHeaders;
  ImageSectionHeader: TImageSectionHeader;
  Sections: array of TImageSectionHeader;
  i, RVA, EP: integer;
begin
  FPe.Seek(0, soBeginning);
  FPE.ReadBuffer(ImageDosHeader, sizeof(ImageDosHeader));
  FPE.Position := ImageDosHeader._lfanew;
  FPE.ReadBuffer(ImageNtHeaders,sizeof(TImageNtHeaders));
//////////////////////////////////////////////////////////
//修改如下：

//File Header
  ImageNtHeaders.FileHeader.NumberOfSections := 4;

  //OptionalHeader
  ImageNtHeaders.OptionalHeader.SizeOfCode := FSectionSizes[0];
  ImageNtHeaders.OptionalHeader.SizeOfImage := FPE.Size;
  //ImageNtHeaders.OptionalHeader.SizeOfImage := FPE.Size - $1000;

  EP := GetEntryPoint;
  ImageNtHeaders.OptionalHeader.AddressOfEntryPoint := EP;
  ShowInfo('PE entry Point: 0x' + IntToHex(EP, 8));

  //修改输入表
  HandlerImport;

  for i := 0 to 15 do
  begin
    ImageNtHeaders.OptionalHeader.DataDirectory[i].VirtualAddress := 0;
    ImageNtHeaders.OptionalHeader.DataDirectory[i].Size := 0;
  end;

  //输入表
  ImageNtHeaders.OptionalHeader.DataDirectory[1].VirtualAddress := FImportVA;
  ImageNtHeaders.OptionalHeader.DataDirectory[1].Size := (FImportCount + 1) *
    sizeof(TImageImportDescriptor);

  //输入地址表
  ImageNtHeaders.OptionalHeader.DataDirectory[12].VirtualAddress := FIAT;
  ImageNtHeaders.OptionalHeader.DataDirectory[12].Size := $124;

////////////////////////////////////////////////////////////////////////////

  //写 改好的信息
  FPE.Position := ImageDosHeader._lfanew;
  FPE.WriteBuffer(ImageNtHeaders,sizeof(TImageNtHeaders));

  SetLength(Sections, 4);

  RVA := $1000;
  for i := 0 to 3 do
  begin
    Move(PChar(FSectionFiles[i])^,Sections[i].Name, length(FSectionFiles[i]));
    Sections[i].Misc.VirtualSize := FSectionSizes[i];
    Sections[i].VirtualAddress := RVA;
    RVA := RVA + FSectionSizes[i];
    Sections[i].SizeOfRawData := FSectionSizes[i];
    Sections[i].PointerToRawData := Sections[i].VirtualAddress;
    Sections[i].PointerToRelocations := 0;
    Sections[i].PointerToLinenumbers := 0;
    Sections[i].NumberOfRelocations := 0;
    Sections[i].NumberOfLinenumbers := 0;
    case i of
    0: Sections[i].Characteristics := $C0000060;
    1: Sections[i].Characteristics := $C0000040;
    2: Sections[i].Characteristics := $C00000C0;
    3:
    begin
      Sections[i].Characteristics := $C00000C0;
      //Sections[i].VirtualAddress := $18000;
    end;

    end;
    FPE.WriteBuffer(Sections[i],sizeof(TImageSectionHeader));
  end;

  Fillchar(ImageSectionHeader, sizeof(TImageSectionHeader), 0);
  FPE.WriteBuffer(ImageSectionHeader,sizeof(TImageSectionHeader));

  ShowInfo('change Pe Head');
end;

procedure TPEHandler.Execute;
begin
  FPE := TMemoryStream.Create;
  try

    FSectionFiles[0] := '_text';
    FSectionFiles[1] := '_rdata';
    FSectionFiles[2] := '_data';
    FSectionFiles[3] := '_rsrc';

    FTmp := TMemoryStream.Create;
    try
      GetPEHead;
      FillSection;
      ChangPEHead;
      FPE.SaveToFile('New.exe');
    finally
      FTmp.Free;
    end;

  finally
    FPE.Free;
  end;

end;

procedure TPEHandler.FillSection;
var
  rmod, i, wcount: integer;

begin
  for i := 0 to 3 do
  begin
    FTmp.LoadFromFile(FSectionFiles[i]);
    rMod := FTmp.Size mod $1000;
    if rMod <> 0 then
    begin
      FTmp.Size := FTmp.Size + $1000 - rMod;
    end;
    FSectionSizes[i] := FTmp.Size;

    FPe.Seek(0, soEnd);
    wcount := FPe.Write(PChar(FTmp.Memory)^, FTmp.Size);
    ShowInfo('Fill Section ' + FSectionFiles[i] + ', size: 0x'
      + IntToHex(WCount, 8));
  end;

end;

function TPEHandler.GetEntryPoint: integer;
const
  VCEP: array[0..5] of byte = ($55, $8B, $EC, $6A, $FF, $68);
var
  rByte: byte;
  i: integer;
  bFind: Boolean;
begin
  Result := 0;
  FPe.Position := $1000;
  while FPe.Position < $1000 + FSectionSizes[0] do
  begin
    bFind := true;
    for i := 0 to 5 do
    begin
      FPe.ReadBuffer(rByte, 1);
      if rByte <> VCEP[i] then
      begin
        FPe.Position := FPe.Position - i;
        bFind := false;
        Break;
      end;
    end;
    if bfind then
    begin
      Result := FPe.Position - 6;
      exit;
    end;

  end;
end;

procedure TPEHandler.GetPEHead;
begin
  FTmp.LoadFromFile('1.exe');
  FPe.Size := $1000;
  FPe.Seek(0, soBeginning);
  FTmp.Size := $1000;
  FPe.LoadFromStream(FTmp);
  ShowInfo('Get Pe Head from: ''1.exe''');

end;

procedure TPEHandler.HandlerImport;
const
  LastRVA: integer = $0654;
var
  iRVA, SectionVA, LastVA, DllVA: integer;
  ImageImportDescriptor: TImageImportDescriptor;
  DLLName: array[1..32] of char;
  DllCount: integer;
begin
  SectionVA := FSectionSizes[0]  + $1000;
  LastVA := SectionVA + LastRVA;
  FImportCount := 0;
  FIAT := $0FFFFFFF;
  while True do
  begin
    FPE.Position := LastVA - (FImportCount + 1) * sizeof(TImageImportDescriptor);
    FPe.ReadBuffer(ImageImportDescriptor, sizeof(TImageImportDescriptor));
    DllVA := ImageImportDescriptor.DLLName;
    if (Integer(DllVA) > sectionVA + FSectionSizes[1]) or
    (Integer(DllVA) < sectionVA) then Break;

    if FIAT > ImageImportDescriptor.FirstThunk then
      FIAT := ImageImportDescriptor.FirstThunk;

    ShowInfo('IAT: 0x' + IntToHex(ImageImportDescriptor.FirstThunk, 8));

    FPE.Position := DLLVA;
    FPE.ReadBuffer(DLLName, 32);
    ShowInfo('Import Dll Name:' + DllName);
    Inc(FImportCount);
  end;
  ShowInfo('Import Count:' + IntToStr(FImportCount));
```
