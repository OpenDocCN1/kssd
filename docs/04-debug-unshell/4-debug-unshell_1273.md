
# 为什么我不能回复帖子，找了半天。英雄王座脱壳代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-507.htm`](https://www.pediy.com/kssd/pediy07/pediy7-507.htm)

既不让我回帖，也不让我贴附件。晕晕ing

那个时候，作的英雄王座的`unpacker`。他的解码主要是第一个节的解码和`IAT`的重建。

代码中也有点错误,不过还没遇到过错误。

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls,Math;

type
  TForm1 = class(TForm)
    Button1: TButton;
    procedure Button1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

type

//   TBSItem = packed record
//
//   end;
//
//   TBSUnpackerINEXE = packed record
//     unknown_000:DWORD;
//     unknown_004:DWORD;
//     unknown_008:DWORD;
//     unknown_00C:DWORD;//<===
//     unknown_010:DWORD;//<===
//     unknown_014:DWORD;//<===
//     unknown_018:DWORD;//<===
//     unknown_01C:DWORD;
//     unknown_020:DWORD;
//     unknown_024:DWORD;
//     unknown_028:DWORD;
//   end;
//
//   TBSFile = packed record
//     classbase:DWORD;//+000;
//     bsFileBuf:PChar;//+004;//
//     bsFileSize:DWORD;//+008;//ReadSize
//     bsFileName:array[0..$100-1] of char;//+0014//FileName
//     LastErrorCode : DWORD;//+214;//LastErrorCode;
//     hbsFile:THandle;//+218;hFile;
//   end;
//
//   TBSPacker = packed record
//     unknown_000:DWORD;//+000
//     unknown_004:DWORD;//+004
//     unknown_008:DWORD;//+008:000
//     unknown_00C:DWORD;//+00C:004=??????=1
//     unknown_010:DWORD;//+010:008=ErrorCode
//     ImageBase:DWORD;//+014:00C=ImageBase=0x00400000<====
//     unknown_018:DWORD;//+018:010=
//     pDosHeader:PImageDosHeader;//+01C:014=pEXE
//     pPEHeader:PImageNtHeaders;//+020:018=pPE
//     pSectionHeader:PImageSectionHeader;//+024:01C=pSection
//     VA_OAT:DWORD;//+028:020=pPE^.78 + ImageBase=OAT
//     VA_IAT:DWORD;//+02C:024=pPE^.80 + ImageBase=IAT<===
//     OriginalImageBase:DWORD;//+030:028=pPE^.34  = OriImageBase
//     Offset_BS:DWORD;//+034:02C=offsetbs
//     unknown_038:DWORD;//+038
//     KEY2 : array[0..$100-1] of char;//+058:
//     KEY2Len : DWORD;//+158
//     //+174 = Size
//     //+170 = buf
//     //+178 = Now
//   end;

   TBSFile = packed record
      bsFileBuf:PChar;//+004;//
      bsFileSize:DWORD;//+008;//ReadSize
   end;

   TBSPacker = packed record
      ImageBase:DWORD;
      pDosHeader:PImageDosHeader;
      pPEHeader:PImageNtHeaders;
      pSectionHeader:PImageSectionHeader;
      NewSectionHeader:PImageSectionHeader;
      exeFileBuf:PChar;
      exeFileSize:DWORD;
   end;

   TBSUnpacker = packed record
      newFileName : String;
      Packer:TBSPacker;
      bsFile:TBSFile;
   end;

procedure Init(var Unpacker:TBSUnpacker);

var
   p:PChar;

begin
   P := Unpacker.Packer.exeFileBuf;
   Unpacker.Packer.pDosHeader := PImageDosHeader(P);
   Unpacker.Packer.pPEHeader := PImageNTHeaders(DWORD(P) + DWORD(Unpacker.Packer.pDosHeader^._lfanew));
   Unpacker.Packer.pSectionHeader := PImageSectionHeader(DWORD(Unpacker.Packer.pPEHeader) + SizeOf(TImageNTHeaders));
   Unpacker.Packer.ImageBase := Unpacker.Packer.pPEHeader^.OptionalHeader.ImageBase;

end;

function RVA_TO_Offset(RVA:DWORD;PE:PImageNTHeaders):DWORD;

var
   P:PImageSectionHeader;
   Count:DWORD;
   Size,Align:DWORD;

begin
   P := PImageSectionHeader(DWORD(PE) + SizeOf(TImageNTHeaders));
   Count := PE^.FileHeader.NumberOfSections;
   Align := PE^.OptionalHeader.SectionAlignment;
   result := 0;
   while Count > 0 do
   begin
      Size :=  P^.Misc.VirtualSize;
      Size := ((Size + Align - 1 ) div Align) * Align;
      if (Count = 1 ) or ((P^.VirtualAddress + Size) >= RVA) then
      begin
         result := (RVA - P^.VirtualAddress) + P^.PointerToRawData;
         break;
      end;
      inc(P);
      dec(Count);
   end;

end;

function Offset_To_RVA(Offset:DWORD;PE:PImageNTHeaders):DWORD;

var
   P:PImageSectionHeader;
   Count:DWORD;
   Size,Align:DWORD;

begin
   P := PImageSectionHeader(DWORD(PE) + SizeOf(TImageNTHeaders));
   Count := PE^.FileHeader.NumberOfSections;
   Align := PE^.OptionalHeader.FileAlignment;
   result := 0;
   while Count > 0 do
   begin
      Size :=  P^.SizeOfRawData;
      Size := ((Size + Align - 1 ) div Align) * Align;
      if (Count = 1) or ((P^.PointerToRawData + Size) >= Offset) then
      begin
         result := (Offset - P^.PointerToRawData) + P^.VirtualAddress;
         break;
      end;
      inc(P);
      dec(Count);
   end;

end;

function Offset_To_VA(Offset:DWORD;PE:PImageNTHeaders):DWORD;

begin
   result := Offset_To_RVA(Offset,PE) + PE^.OptionalHeader.ImageBase;

end;

const
   mENCODE = 0;
   mDECODE = 1;

type
   TBlock = array[0..3] of DWORD;
   PBlock = ^TBlock;
   TSubkeyBlock = array[0..4-1] of DWORD;
   PSubkeyBlock = ^TSubkeyBlock;
   TSubKeyIndex = array[0..16-1] of PSubkeyBlock;
   TKeyArray = array[0..16-1] of TSubkeyBlock;

procedure InitKey(KEY:PBlock;var UArray:TKeyArray;var KEYBlock:TSubKeyIndex);

var
   Index:Integer;
   K0,K1,K2,K3:DWORD;

begin
   K0 := KEY[0];K1 := KEY[1];K2 := KEY[2];K3 := KEY[3];
   for Index := 0 to 15 do
   begin
      KEYBlock[Index] := @UArray[Index];
      K0 := (K0 + K1) * (K2 + K3);//EAX
      K1 := (K0 + K2) * (K1 + K3);//ECX
      K2 := (K2 + K1) * (K0 + K3);//EDX
      K3 := K3 + ((K0 + K1) + K2);//ESI
      UArray[Index,1]:= K1;
      UArray[Index,0]:= K0;
      UArray[Index,2]:= K2;
      UArray[Index,3]:= K3;
   end;

end;

procedure Coding(var InBlock:TBlock;SubKey:PSubkeyBlock;var OutBlock:TBlock;iMode:DWORD);

var
   K0,K1,K2,K3,I0,I1,I2,I3,U1,U2:DWORD;

begin
   I0 := InBlock[0];I1 := InBlock[1];I2 := InBlock[2];I3 := InBlock[3];
   K0 := SubKey[0];K1 := SubKey[1];K2 := SubKey[2];K3 := SubKey[3];
   U2 := (K0 + K1) * (I2 + I3) * (K2 + K3);//EAX
   U1 := (I2 * I3) + ((K1 + K3) * (K2 + K0));//ECX
   OutBlock[0] := I2;
   OutBlock[1] := I3;
   OutBlock[2] := I0 xor U2;
   OutBlock[3] := I1 xor U1;

end;

procedure Coder(InBlock,KEY,OutBlock:PChar;iMode:DWORD);

var
   KeyArray:TKeyArray;
   SubKeyIndex:TSubKeyIndex;
   TmpInBlock,TmpOutBlock:TBlock;
   Index : Integer;

begin
   ZeroMemory(@KeyArray[0],SizeOf(KeyArray));
   Move(InBlock^,TmpInBlock,4*4);
   InitKey(PBlock(KEY),KeyArray,SubKeyIndex);
   if iMode = mDECODE then
   begin//解密
      for Index := 0 to 15 do
      begin
         Coding(TmpInBlock,SubKeyIndex[Index],TmpOutBlock,mENCODE);
         TmpInBlock := TmpOutBlock;
      end;
   end else
   begin//加密
      for Index := 15 downto 0 do
      begin
         Coding(TmpInBlock,SubKeyIndex[Index],TmpOutBlock,mDECODE);
         TmpInBlock := TmpOutBlock;
      end;
   end;

   //高 8 位和低 8 交换
   TmpOutBlock[0] := TmpInBlock[2];
   TmpOutBlock[1] := TmpInBlock[3];
   TmpOutBlock[2] := TmpInBlock[0];
   TmpOutBlock[3] := TmpInBlock[1];
   Move(TmpOutBlock[0],OutBlock^,4*4);

end;

//根据
    //只计算整数部分
    procedure _Decode(BUF:PChar;BUFSize:DWORD;KEY:PChar;KEYLen:DWORD);
    var
       Count:Integer;
       InBlock,OutBlock:PChar;
    begin
       InBlock := BUF;
       Count := BUFSize shr 4;//(BUFSize + 15) shr 4;//只计算整数部分
       while (Count > 0) do
       begin
          OutBlock := InBlock;
```



```
            Coder(InBlock,KEY,OutBlock,mDECODE);  

            Inc(InBlock,16);  

            Dec(Count);  

         end;  

    end;  

procedure DecodeFirstSection(var Unpacker:TBSUnpacker);  

var  

   P:PChar;  

   Size,Res:DWORD;  

   T:array[0..15] of char;  

begin  

   //UnpackerData.  

   P := PChar(Unpacker.Packer.exeFileBuf + Unpacker.Packer.pSectionHeader^.PointerToRawData);  

   Size := Unpacker.Packer.pSectionHeader^.SizeOfRawData;  

   _Decode(P,Size,'1234567890987654',16);  

   Res := Size and $0F;  

   if res <> 0 then  

   begin  

      FillChar(T,16,0);  

      Move(PChar(DWORD(P) + Size - Res)^,T,Res);  

      _Decode(@T,16,'1234567890987654',16);  

      Move(T,PChar(DWORD(P) + Size - Res)^,Res);  

   end;  

end;  

function GetDWORD(BUF:PCHAR):DWORD;  

begin  

   GetDWORD := PDWORD(BUF)^;  

end;  

//function GetSize(BUF:PCHAR):DWORD;  

//var  

//   Count:DWORD;  

//   p:PDWORD;  

//begin  

//   Result := 0;  

//   Count := PDWORD(BUF)^;  

//   p := PDWORD(DWORD(BUF) + 8);  

//   while Count>0 do  

//   begin  

//      if p^ > Result then  

//         Result := p^;  

//      dec(Count);  

//      p := PDWORD(DWORD(p)+ p^ + 4);  

//   end;  

//end;  

//Updater.bs    UPDATER.EXE  

//BSSerJ.bs     BSSerialJ.exe  

//BSLic.bs      BSLicense.exe  

//BSBookJ.bs    BSBookJ.exe  

//packet.bs     Travia.exe  

function Loadbs(var bsFile:TBSFile;const FileName:String):Boolean;  

var  

   hbsFile : THandle;  

begin  

   Result := False;  

   hbsFile := FileOpen(FileName,  fmOpenRead or fmShareDenyNone);  

   if hbsFile <> $FFFFFFFF then  

   begin  

      bsFile.bsFileSize := GetFileSize(hbsFile,nil);  

      GetMem(bsFile.bsFileBuf,bsFile.bsFileSize);  

      FileRead(hbsFile,bsFile.bsFileBuf^,bsFile.bsFileSize);  

      CloseHandle(hbsFile);  

      result := True;  

   end;  

end;  

function GetIATBlockByIndex(BUF:PChar;Index:DWORD):PChar;  

var  

   P : PDWORD;  

begin  

   P := PDWORD(DWORD(BUF) + 8);  

   while Index > 0 do  

   begin  

      dec(Index);  

      p := PDWORD(DWORD(p)+ p^ + 4);  

   end;  

   GetIATBlockByIndex := Pointer(DWORD(P)+4);  

end;  

function GetIATBlockSizeByIndex(BUF:PChar;Index:DWORD):DWORD;  

var  

   P : PDWORD;  

begin  

   P := PDWORD(DWORD(BUF) + 8);  

   while Index > 0 do  

   begin  

      dec(Index);  

      p := PDWORD(DWORD(p)+ p^ + 4);  

   end;  

   GetIATBlockSizeByIndex := P^;  

end;  

type  

   TImport = packed record  

       ImportFlag:DWORD;  

       TimeStamp:DWORD;  

       MajorVersion,MinorVersion:WORD;  

       RVA_DLLNAME:DWORD;  

       RVA_ImportLookupTable:DWORD;  

//       RVA_ImportAddress:DWORD;  

   end;  

   PImport = ^TImport;  

//10001F9C   

function MakeIAT(FileBase:DWORD;PE:PImageNTHeaders;BUF:PChar;P3:PImport;var P1:PDWORD;var P2:PCHAR):Boolean;  

var  

   nCountOfIATBlock : DWORD;  

   Index : DWORD;  

   P,PAPI,PAPI2,PFIX:PChar;  

//   Size:DWORD;  

   Index2,Count2:DWORD;  

   Index3,Count3:DWORD;  

   prefix : PDWORD;  

   fixrva : PDWORD;  

   PD : PDWORD;  

begin  

   Result := True;  

   nCountOfIATBlock := GetDWORD(BUF);//项目个数,0=DLL,1..API//=Count  

   P := GetIATBlockByIndex(BUF,0);//P=DLLNAME//10002006  

   P2 := PCHAR(DWORD(P1) + nCountOfIATBlock * 4);  

   P3^.RVA_ImportLookupTable := Offset_To_RVA(DWORD(P1) - FileBase,PE);  

   P3^.RVA_DLLNAME := Offset_To_RVA(DWORD(P2) - FileBase,PE);  

//   P3^.RVA_ImportAddress :=  

   Move(P^,P2^,strlen(P));//DLLName  

   Inc(P2,strlen(P));  

   P2^ := #0;  

   Inc(P2);  

   P2^ := #0;  

   if (DWORD(P2) and 1) <> 0 then Inc(P2);  

   for Index := 1 to nCountOfIATBlock - 1 do//API 个数  

   begin  

      PAPI := GetIATBlockByIndex(BUF,Index);//100020A5=??Mem1  

      Count2 := GetDWORD(PAPI);       

          PAPI2 := GetIATBlockByIndex(PAPI,0);//100020E9=API=Memx  

          if PAPI2^ <> #0 then  

          begin//By Name,+5  

             Inc(PAPI2,5);  

             P1^ := Offset_To_RVA(DWORD(P2) - FileBase,PE);  

             P2^ := #0;Inc(P2);P2^ := #0;Inc(P2);  

             Move(PAPI2^,P2^,strlen(PAPI2));  

             Inc(P2,strlen(PAPI2));  

             P2^ := #0;  

             Inc(P2);  

             P2^ := #0;  

             if (DWORD(P2) and 1) <> 0 then Inc(P2);  

          end else  

          begin//By Order,+5  

             Inc(PAPI2,5);  

             PD := PDWORD(PAPI2);  

             P1^ := PD^ or $80000000;  

          end;  

          //Fix//=3  

          for Index2 := 1 to Count2 - 1 do//几种模式  

          begin  

             PFIX := GetIATBlockByIndex(PAPI,Index2);  

             Count3 := GetIATBlockSizeByIndex(PAPI,Index2);  

             Count3 := (Count3 - 8) shr 2;//  

             fixrva := PDWORD(DWORD(PFIX) + 8);  

             prefix := PDWORD(PFIX);  

             for Index3 := 1 to Count3 do//几个地方  

             begin  

                PD := PDWORD(RVA_to_Offset(fixrva^,PE) + FileBase);  

                if PD = nil then  

                    MessageBeep(0);  

                PD^ := prefix^;  

                if PD^ > $FF then  

                asm  

                   push eax  

                   push edx  

                   mov edx,PD  

                   mov eax,[edx]  

                   bswap eax  

                   shr eax,16  

                   mov [edx],eax  

                   pop edx  

                   pop eax  

                end;  

                if prefix^ = $A1 then  

                begin//+1  

                   PD := PDWORD(DWORD(PD) + 1);  

                end else  

                begin//+2  

                   PD := PDWORD(DWORD(PD) + 2);  

                end;  

                PD^ := DWORD(Offset_To_VA(DWORD(P1) - FileBase,PE));//?  

                inc(fixrva);  

             end;  

          end;  

          //Size := GetIATBlockSizeByIndex(PAPI,0);  

          inc(P1);  

   end;  

   P1^ := 0;  

   Inc(P1);  

end;  

function Decodebs(var Unpacker:TBSUnpacker;KEY:PChar;KEYLen:DWORD):Boolean;  

var  

   nCountOfIATBlock : DWORD;  

   Index : DWORD;  

   P:PChar;  

   Size:DWORD;  

   P1:PDWORD;  

   P2:PCHAR;  

   P3:PImport;  

   IATSize : DWORD;  

   FileBase:DWORD;  

   hf,NewFileSize : DWORD;  

   Align:DWORD;  

begin  

   Result := True;  

   nCountOfIATBlock := GetDWORD(Unpacker.bsFile.bsFileBuf);  

   //留一个空  

   P3 := PImport(DWORD(Unpacker.Packer.exeFileBuf) + Unpacker.Packer.NewSectionHeader^.PointerToRawData);  

   Unpacker.Packer.pPEHeader^.OptionalHeader.DataDirectory[1].VirtualAddress := Unpacker.Packer.NewSectionHeader^.VirtualAddress;  

   IATSize := (nCountOfIATBlock + 1) * SizeOf(TImport);  

   FillChar(P3^,IATSize,0);  

   Unpacker.Packer.pPEHeader^.OptionalHeader.DataDirectory[1].Size := IATSize;  

   P1 := PDWORD(DWORD(P3) + IATSize);  

   //修改 IAT 指针  

   FileBase := DWORD(Unpacker.Packer.exeFileBuf);  

   for Index := 0 to nCountOfIATBlock - 1 do  

   begin  

       P := GetIATBlockByIndex(Unpacker.bsFile.bsFileBuf,Index);  

       Size := GetIATBlockSizeByIndex(Unpacker.bsFile.bsFileBuf,Index);  

       Inc(P,4);  

       Dec(Size,4);  

       _Decode(P,Size,KEY,KEYLen);//直接整数部分//余数部分不用处理  

       //P1=起始地址  

       MakeIAT(FileBase,Unpacker.Packer.pPEHeader,P,P3,P1,P2);  

       P1 := PDWORD(P2);  

       inc(P3);  

   end;  

   NewFileSize := (DWORD(P1) - FileBase);  

   Align := Unpacker.Packer.pPEHeader^.OptionalHeader.FileAlignment;  

   Size := NewFileSize - Unpacker.Packer.NewSectionHeader^.PointerToRawData;  

   Size := ((Size + Align - 1) div Align) * Align;  

   Unpacker.Packer.NewSectionHeader^.SizeOfRawData := Size;  

   Align := Unpacker.Packer.pPEHeader^.OptionalHeader.SectionAlignment;  

   Size := NewFileSize - Unpacker.Packer.NewSectionHeader^.PointerToRawData;  

   Size := ((Size + Align - 1) div Align) * Align;
```



```pascal
Unpacker.Packer.NewSectionHeader^.Misc.VirtualSize := Size;

//Unpacker.Packer.pPEHeader^.OptionalHeader.DataDirectory[1].Size := Size;

Unpacker.Packer.pPEHeader^.OptionalHeader.SizeOfImage :=Unpacker.Packer.NewSectionHeader^.VirtualAddress + Size;

NewFileSize := Unpacker.Packer.NewSectionHeader^.PointerToRawData + Unpacker.Packer.NewSectionHeader^.SizeOfRawData;

hf := FileCreate(Unpacker.newFileName);

FileWrite(hf,Unpacker.Packer.exeFileBuf^,NewFileSize);

FileClose(hf);

end;

function LoadExe(var Unpacker:TBSUnpacker;const FileName:String):Boolean;

var

   hexeFile : THandle;

begin

   Result := False;

   hexeFile := FileOpen(FileName,  fmOpenRead or fmShareDenyNone);

   if hexeFile <> $FFFFFFFF then

   begin

      Unpacker.Packer.exeFileSize := GetFileSize(hexeFile,nil);

      GetMem(Unpacker.Packer.exeFileBuf,Unpacker.Packer.exeFileSize + Unpacker.bsFile.bsFileSize);

      FileRead(hexeFile,Unpacker.Packer.exeFileBuf^,Unpacker.Packer.exeFileSize);

      CloseHandle(hexeFile);

      Unpacker.Packer.exeFileSize := Unpacker.Packer.exeFileSize + Unpacker.bsFile.bsFileSize;

      result := True;

   end;

end;

//            ('UPDATER.EXE',   'Updater.bs'),

//            ('BSSerialJ.exe', 'BSSerJ.bs'),

//            ('BSLicense.exe', 'BSLic.bs'),

//            ('BSBookJ.exe',   'BSBookJ.bs'),

//            ('Travia.exe',    'packet.bs'));

procedure UnpackIt(exeFileName,bsFileName,toFileName:String);

var

   Unpacker :TBSUnpacker;

   Offset,Align:DWORD;

begin     

   Unpacker.newFileName := toFileName;

   if LoadBs(Unpacker.bsFile,bsFileName) then//Load bs file;

   begin

      if LoadExe(Unpacker,exeFileName) then

      begin

         Init(Unpacker);

         //解开第一个节

         DecodeFirstSection(Unpacker);

         //处理其他,新节的数据

         Inc(Unpacker.Packer.pSectionHeader,Unpacker.Packer.pPEHeader.FileHeader.NumberOfSections);

         Inc(Unpacker.Packer.pPEHeader.FileHeader.NumberOfSections);

         Unpacker.Packer.NewSectionHeader := Unpacker.Packer.pSectionHeader;

         Dec(Unpacker.Packer.pSectionHeader);

         FillChar(Unpacker.Packer.NewSectionHeader^,SizeOf(TImageSectionHeader),0);

         Move('.idata',Unpacker.Packer.NewSectionHeader^.Name,6);

         Offset := Unpacker.Packer.pSectionHeader^.Misc.VirtualSize;

         Align :=  Unpacker.Packer.pPEHeader^.OptionalHeader.SectionAlignment;

         Offset := ((Offset + Align - 1) div Align) * Align;

         Unpacker.Packer.NewSectionHeader^.VirtualAddress := Unpacker.Packer.pSectionHeader^.VirtualAddress + Offset;

         Offset := Unpacker.Packer.pSectionHeader^.SizeOfRawData;

         Align :=  Unpacker.Packer.pPEHeader^.OptionalHeader.FileAlignment;

         Offset := ((Offset + Align - 1) div Align) * Align;           

         Unpacker.Packer.NewSectionHeader^.PointerToRawData := Unpacker.Packer.pSectionHeader^.PointerToRawData  + Offset;

         Unpacker.Packer.NewSectionHeader^.Characteristics := $E0000020;

         //解开重建 IAT

         Decodebs(Unpacker,'1234567890987654',16);

         //Unpacker.Packer.NewSectionHeader^.SizeOfRawData := 0;

         //Unpacker.Packer.NewSectionHeader^.Misc.VirtualSize := 0;

         //

         FreeMem(Unpacker.Packer.exeFileBuf,Unpacker.Packer.exeFileSize);

      end;

      FreeMem(Unpacker.bsFile.bsFileBuf,Unpacker.bsFile.bsFileSize);

   end;

end;

procedure TForm1.Button1Click(Sender: TObject);

begin

   UnpackIt('Travia.exe','packet.bs','Travia~.exe');

   UnpackIt('Updater.exe','Updater.bs','Updater~.exe');

   ShowMessage('Finished.');

end;

end.
```

4dd466d0e1e0cc123afbe504da2462ca
