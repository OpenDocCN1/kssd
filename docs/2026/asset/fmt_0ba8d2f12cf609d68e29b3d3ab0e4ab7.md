
# 反汇编片段

```
:0040EBE3 EB0F
jmp 0040EBF4
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EB40(C)
|
:0040EBE5 FF45A0
inc [ebp-60]
:0040EBE8 8B4DA0
mov ecx, dword ptr [ebp-60]
:0040EBEB 83F906
cmp ecx, 00000006
:0040EBEE 0F8C58FEFFFF
jl 0040EA4C
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040EB35(U), :0040EBE3(U)
|
:0040EBF4 FF45A4
inc [ebp-5C]
:0040EBF7 8B45A4
mov eax, dword ptr [ebp-5C]
:0040EBFA 83F814
cmp eax, 00000014  <====注册码共20位
:0040EBFD 0F8C02FDFFFF
jl 0040E905
<====循环计算注册码
:0040EC03 66C745B88000
mov [ebp-48], 0080
:0040EC09 8D55F8
lea edx, dword ptr [ebp-08]
:0040EC0C 8B45CC
mov eax, dword ptr [ebp-34]
:0040EC0F E8640D2B00
call 006BF978
:0040EC14 8B45CC
mov eax, dword ptr [ebp-34]
:0040EC17 66C745B88C00
mov [ebp-48], 008C
:0040EC1D 50
push eax
:0040EC1E FF4DC4
dec [ebp-3C]
:0040EC21 8D45F4
lea eax, dword ptr [ebp-0C]
:0040EC24 BA02000000
mov edx, 00000002
:0040EC29 E81A0D2B00
call 006BF948
:0040EC2E FF4DC4
dec [ebp-3C]
:0040EC31 8D45F8
lea eax, dword ptr [ebp-08]
:0040EC34 BA02000000
mov edx, 00000002
:0040EC39 E80A0D2B00
call 006BF948
:0040EC3E FF4DC4
dec [ebp-3C]
:0040EC41 8D45FC
lea eax, dword ptr [ebp-04]
:0040EC44 BA02000000
mov edx, 00000002
:0040EC49 E8FA0C2B00
call 006BF948
:0040EC4E 58
pop eax
:0040EC4F 66C745B88000
mov [ebp-48], 0080
:0040EC55 FF45C4
inc [ebp-3C]
:0040EC58 8B55A8
mov edx, dword ptr [ebp-58]
:0040EC5B 64891500000000
mov dword ptr fs:[00000000], edx
:0040EC62 5B
pop ebx
:0040EC63 8BE5
mov esp, ebp
:0040EC65 5D
pop ebp
:0040EC66 C3
ret
```

至此,整个运算过程结束了,下面用DELPHI写出注册机:

## Delphi 代码

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes,
Graphics, Controls, Forms,
  Dialogs, XP_Form, XP_Button, jpeg, ExtCtrls,
XP_GroupBox, StdCtrls,registry;

type
  TForm1 = class(TForm)
    frmain: TXP_Form;
    reg: TXP_Button;
    about: TXP_Button;
    exitprogram: TXP_Button;
    XP_GroupBox1: TXP_GroupBox;
    Image1: TImage;
    procedure aboutClick(Sender: TObject);
    procedure exitprogramClick(Sender: TObject);
    procedure regClick(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

function GetHardDiskSerieNumber(): integer;

var
  Form1: TForm1;

implementation

uses Unit2;

{$R *.dfm}

function GetHardDiskSerieNumber: integer;
var
  sysinfo: tsysteminfo;
  lpRootPathName          : PChar;
  lpVolumeNameBuffer      : PChar;
  nVolumeNameSize         : DWORD;
  lpVolumeSerialNumber    : DWORD;
  lpMaximumComponentLength : DWORD;
  lpFileSystemFlags        : DWORD;
  lpFileSystemNameBuffer  : PChar;
  nFileSystemNameSize     : DWORD;
begin
  lpRootPathName:=pchar('c:\');
  windows.GetSystemInfo(sysinfo);
  GetMem( lpVolumeNameBuffer, MAX_PATH + 1 );
  GetMem( lpFileSystemNameBuffer, MAX_PATH + 1 );
  nVolumeNameSize    := MAX_PATH + 1;
  nFileSystemNameSize:= MAX_PATH + 1;
  Windows.GetVolumeInformation(  lpRootPathName,
          lpVolumeNameBuffer,
          nVolumeNameSize,
                  @lpVolumeSerialNumber,
          lpMaximumComponentLength,
          lpFileSystemFlags,
                    lpFileSystemNameBuffer,
          nFileSystemNameSize );
  Result := lpVolumeSerialNumber;
end;

procedure TForm1.aboutClick(Sender: TObject);
begin
  form2.show;
end;

procedure TForm1.exitprogramClick(Sender: TObject);
begin
  form1.Close;
end;

procedure TForm1.regClick(Sender: TObject);
label
  isregcode,quitloop;
var
  v1,v2,v3,v4,v5,v6,v7,v8:integer;
  hdid:longword;
  hdln:integer;
  idstr:string;
  i,j,k:integer;
  regcode:array[1..20] of char ;
  cmod:integer;
  cdiv:integer;
  lidbit,idbit:integer;
  idbitstr:string;
  newidbit:integer;
  regcodebit:integer;
  reg:TRegistry;
begin
  hdid:=GetHardDiskSerieNumber;
  hdln:=length(inttostr(hdid) );
  idstr:=inttostr(hdid);
  v1:=7;
  v2:=1;
  v3:=3;
  v4:=0;
  for i:=1 to 20 do
  begin
    v3:=((v3+1) mod ((6 mod hdln)+1))+1;
    v4:=v4+v1;
    v1:=(v1+$18b) mod $0A;
    idbitstr:='';
    for j:=1 to v3 do
    begin
      v4:=((v4+v2) mod hdln)+1;
      v2:=(v2+1) mod 6;
      idbitstr:=idbitstr+idstr[v4];
    end;
    newidbit:=strtoint(idbitstr);
    v5:=$85;
    v6:=$6f;
    v7:=$42;
    v8:=$a6;
    for k:=1 to 6 do
    begin
      regcodebit:=((newidbit mod (v5+1))+(newidbit mod (v6+1))+(newidbit mod (v7+1))) mod (v8+1);
      v5:=v5+2;
      v6:=v6-7;
      v7:=v7+5;
      v8:=v8-3;
      if (regcodebit>=48) and (regcodebit<=57) then goto isregcode;
      if (regcodebit>=65) and (regcodebit<=90) then goto isregcode;
      if (regcodebit>=97) and (regcodebit<=122) then goto isregcode;
    end;
    if regcodebit<61 then
    begin
      regcode[i]:=chr((regcodebit mod 10)+48);
      goto quitloop;
    end;
    if regcodebit>93 then
    begin
      regcode[i]:=chr((regcodebit mod 26)+97);
      goto quitloop;
    end;
    if (regcodebit<=93) then
    begin
      regcode[i]:=chr((regcodebit mod 26)+65);
      goto quitloop;
    end;
isregcode:
    regcode[i]:=chr(regcodebit);
quitloop:
  end;
  Reg := TRegistry.Create;
  try
    Reg.RootKey := HKEY_LOCAL_MACHINE;
    if Reg.
```
