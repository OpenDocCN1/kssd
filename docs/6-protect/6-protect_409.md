

# 检测 softice（delphi 版）

> 原文：[`www.pediy.com/kssd/pediy04/kanxue129.htm`](https://www.pediy.com/kssd/pediy04/kanxue129.htm)

检查是否加载了 Softice   
  
This tip show, how to check if the debugging program Softice is loaded.   
  
Dieser Tip zeigt, wie 黚 erpr 黤 t, wird ob das debugging Programm Softice geladen ist.   
  
//SoftIce in W9x OS   
function IsSoftIce95Loaded: boolean;   
var   
  hFile: Thandle;   
begin   
result := false;   
hFile := CreateFileA('\\.SICE', GENERIC\_READ or GENERIC\_WRITE,   
  FILE\_SHARE\_READ or FILE\_SHARE\_WRITE, nil, OPEN\_EXISTING,   
  FILE\_ATTRIBUTE\_NORMAL, 0);   
if( hFile <> INVALID\_HANDLE\_VALUE ) then begin   
  CloseHandle(hFile);   
  result := TRUE;   
end;   
end;   
  
// SoftIce in NT/2000 OS   
function IsSoftIceNTLoaded: boolean;   
var   
  hFile: Thandle;   
begin   
result := false;   
hFile := CreateFileA('\\.NTICE', GENERIC\_READ or GENERIC\_WRITE,   
  FILE\_SHARE\_READ or FILE\_SHARE\_WRITE, nil, OPEN\_EXISTING,   
  FILE\_ATTRIBUTE\_NORMAL, 0);   
if( hFile <> INVALID\_HANDLE\_VALUE ) then begin   
  CloseHandle(hFile);   
  result := TRUE;   
end;   
end;   
  
procedure TForm1.Button1Click(Sender: TObject);   
begin   
  if IsSoftIce95Loaded or IsSoftIceNTLoaded then   
  begin   
    //Do something if Softice is loaded   
  end;   
end;  4dd466d0e1e0cc123afbe504da2462ca