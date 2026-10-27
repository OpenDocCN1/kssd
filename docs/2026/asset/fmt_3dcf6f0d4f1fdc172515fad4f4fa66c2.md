
```pascal
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

   for Index := 1 to nCountOfIATBlock - 1 do//API个数

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

   //修改IAT指针

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
```
