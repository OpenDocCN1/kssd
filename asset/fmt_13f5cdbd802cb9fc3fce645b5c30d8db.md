
```
  tmp := PDWORD(MapViewOfFile(g_hMPM, FILE_MAP_WRITE, 0, phys and $FFFFF000, $1000));  
  if (nil = tmp) then  
  begin  
    Result := false;  
    Exit;  
  end;  
//  tmp[(phys and $FFF) shr 2] := data;  
  PDWORD(DWORD(tmp) + ((phys and $FFF) shr 2) * SizeOf(DWord))^ := data; // Modify by dot.  
  UnmapViewOfFile(tmp);  
  Result := TRUE;  
end;  

//---------------------------------------------------------------------------  
{long __stdcall exeception(struct _EXCEPTION_POINTERS *tmp)  
begin  
  ExitProcess(0);  
  return 1 ;  
end }  
//---------------------------------------------------------------------------  

function YHideProcess: Boolean;  
var  
  thread, process: DWORD;  
  fw, bw: DWORD;  
begin  
//  SetUnhandledExceptionFilter(exeception);  
  if (FALSE = InitNTDLL) then  
  begin  
    Result := FALSE;  
    Exit;  
  end;  
  if (0 = OpenPhysicalMemory) then  
  begin  
    Result := FALSE;  
    Exit;  
  end;  
  thread := GetData(Pointer($FFDFF124)); //kteb  
  process := GetData(Pointer(thread + $44)); //kpeb  
  if (0 = g_osvi.dwMinorVersion) then  
  begin  
    fw := GetData(Pointer(process + $A0));  
    bw := GetData(Pointer(process + $A4));  
    SetData(Pointer(fw + 4), bw);  
    SetData(Pointer(bw), fw);  
    Result := TRUE;  
  end  
  else if (1 = g_osvi.dwMinorVersion) then  
  begin  
    fw := GetData(Pointer(process + $88));  
    bw := GetData(Pointer(process + $8C));  
    SetData(Pointer(fw + 4), bw);  
    SetData(Pointer(bw), fw);  
    Result := TRUE;  
  end  
  else  
  begin  
    Result := False;  
  end;  
  CloseHandle(g_hMPM);  
  CloseNTDLL;  
end;  

function MyHideProcess: Boolean;  
begin  
  if not b_hide then  
  begin  
    b_hide := YHideProcess;  
  end;  
  Result := b_hide;  
end;  

end.
```

`Project1.dpr` 主程序的内容！

```
unit Unit1;  

interface  

uses  
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,  
  Dialogs, StdCtrls;  

type  
  TForm1 = class(TForm)  
    Button1: TButton;  
    Label1: TLabel;  
    Label2: TLabel;  
    procedure Button1Click(Sender: TObject);  
  private  
    { Private declarations }  
  public  
    { Public declarations }  
  end;  

var  
  Form1: TForm1;  

implementation  

uses HideProcess;  

{$R *.dfm}  

procedure TForm1.Button1Click(Sender: TObject);  

begin  
    MyHideProcess;  
end;  

end. 
```

`4dd466d0e1e0cc123afbe504da2462ca`
