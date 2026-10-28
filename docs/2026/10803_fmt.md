
# [旧帖][原创]用户态的 API Hook 0.00 雪花

以前的账号丢了，写两行代码看看能不能要个邀请码。

老生长谈了。。。。。。。。。。。。。![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)

## 陷阱式

```pascal
program Project2;

{$APPTYPE CONSOLE}

uses

  SysUtils, Windows;

type

  TMessageBox = function(hwnd: HWND; lpText: PChar; lpCaption: PChar; uType:

    Cardinal):

    integer; stdcall;

  TMessageBoxA = function(hwnd: HWND; lpText: PAnsiChar; lpCaption: PAnsiChar;

    uType: Cardinal):

    integer; stdcall;

  TMessageBoxW = function(hwnd: HWND; lpText: PWideChar; lpCaption: PWideChar;

    uType: Cardinal):

    integer; stdcall;

var

  baseAddress: pointer;

  OldEntry, NewEntry: array[0..7] of Byte;

  pA, pW: Pointer;

function MyBox(hwnd: HWND; lpText: PChar; lpCaption: PChar; uType:

  Cardinal):

  integer; stdcall;

var

  dwSize: DWORD;

begin

  WriteProcessMemory(GetCurrentProcess, pA, @OldEntry, 8, dwSize);

  Result := MessageBoxA(hwnd, lpText, 'Hook Api A', uType);

  WriteProcessMemory(GetCurrentProcess, pA, @NewEntry, 8, dwSize);

end;

function MyBoxA(hwnd: HWND; lpText: PAnsiChar; lpCaption: PAnsiChar; uType:

  Cardinal):

  integer; stdcall;

var

  dwSize: DWORD;

begin

  WriteProcessMemory(GetCurrentProcess, pA, @OldEntry, 8, dwSize);

  Result := MessageBoxA(hwnd, lpText, 'Hook Api A', uType);

  WriteProcessMemory(GetCurrentProcess, pA, @NewEntry, 8, dwSize);

end;

function MyBoxW(hwnd: HWND; lpText: PWideChar; lpCaption: PWideChar; uType:

  Cardinal):

  integer; stdcall;

var

  dwSize: DWORD;

begin

  WriteProcessMemory(GetCurrentProcess, pW, @OldEntry, 8, dwSize);

  Result := MessageBoxW(hwnd, lpText, 'Hook Api W', uType);

  WriteProcessMemory(GetCurrentProcess, pW, @NewEntry, 8, dwSize);

end;

procedure HookApi(Module: PChar; FunName: Pchar; NewFun: Pointer; var

  baseAddress:

  Pointer);

var

  MyFun, dwReadSize: DWORD;

  Addr: array[0..3] of BYTE;

begin

  baseAddress := GetProcAddress(GetModulehandle(Module), FunName);

  if baseAddress = nil then

    exit;

  MyFun := DWORD(NewFun);

  Move(MyFun, Addr, 4);

  NewEntry[0] := $B8;

  NewEntry[1] := Addr[0];

  NewEntry[2] := Addr[1];

  NewEntry[3] := Addr[2];

  NewEntry[4] := Addr[3]; //mov ax,addr

  NewEntry[5] := $FF;

  NewEntry[6] := $E0; //jmp ax

  NewEntry[7] := 0;

  //读取原始头部

  if ReadProcessMemory(GetCurrentProcess(), baseAddress, @OldEntry, 8,

    dwReadSize) then

  begin

    VirtualProtectEx(GetCurrentProcess(), baseAddress, 0, PAGE\_READWRITE,

      nil);

    //可写

    dwReadSize := 0;

    WriteProcessMemory(GEtCurrentProcess(), baseAddress, @NewEntry, 8,

      dwReadSize);

  end;

end;

begin

  HookApi(PChar('user32.dll'), PChar('MessageBoxA'), @MyBoxA, pA);

  HookApi(PChar('user32.dll'), PChar('MessageBoxW'), @MyBoxW, pW);

  MessageBox(0, 'hello', 'hello', 0);

  MessageBoxW(0, 'hello', 'hello', 0);

  readln;

end.
```

## 修改 IAT 式

```pascal
program Project1;

{$APPTYPE CONSOLE}

uses

  SysUtils, Windows, Classes;

type

  PIMAGE\_IMPORT\_ENTRY = ^TIMAGE\_IMPORT\_ENTRY;

  TIMAGE\_IMPORT\_ENTRY = packed record

    Characteristics: DWORD;

    TimeDateStamp: DWORD;

    MajorVersion: WORD;

    MinorVersion: WORD;

    Name: DWORD;

    LookupTable: DWORD;

  end;

  PIMORT\_CODE = ^TIMPORT\_CODE;

  TIMPORT\_CODE = packed record

    JMPPtr: Word;

    PtrAdd: ^Pointer

  end;

  TMyBox = function(hwnd: HWND; lpText: PChar; lpCaption: PChar; uType:

    Cardinal): Integer; stdcall;

  TMyBoxA = function(hwnd: HWND; lpText: PAnsiChar; lpCaption: PAnsiChar; uType:

    Cardinal): Integer; stdcall;

  TMyBoxW = function(hwnd: HWND; lpText: PWideChar; lpCaption: PWideChar; uType:

    Cardinal): Integer; stdcall;

var

  OldBox: TMyBox;

  OldBoxA: TMyBoxA;

  OldBoxW: TMyBoxW;

function MyBox(hwnd: HWND; lpText: PChar; lpCaption: PChar; uType:

  Cardinal): Integer; stdcall;

begin

  Result := OldBox(hwnd, lpText, PChar('Hook API'), uType);

end;

function MyBoxA(hwnd: HWND; lpText: PAnsiChar; lpCaption: PAnsiChar; uType:

  Cardinal): Integer; stdcall;

begin

  Result := OldBoxA(hwnd, lpText, PAnsiChar('Hook API'), uType);

end;

function MyBoxW(hwnd: HWND; lpText: PWideChar; lpCaption: PWideChar; uType:

  Cardinal): Integer; stdcall;

begin

  Result := OldBoxW(hwnd, lpText, PWideChar(PChar('Hook API')), uType);

end;

function GetAPIAddress(APIPtr: Pointer): Pointer;

begin

  Result := APIPtr;

  if APIPtr = nil then

    exit;

  try

    if (PIMORT\_CODE(APIPtr).JMPPtr = $25FF) then

      Result := PIMORT\_CODE(APIPtr).PtrAdd^;

  except

    Result := nil;

  end;

end;

function SwapPtr(OldPtr, NewPtr: Pointer): integer;

var

  IsDosHead: TList;

  function hkSwapPtr(h: Cardinal; OldPtr, NewPtr: Pointer): integer;

  var

    DosHeader: PIMageDosHeader;

    NTHeader: PImageNTHeaders;

    impEty: PIMAGE\_IMPORT\_ENTRY;

    VAddr: DWORD;

    Func: ^Pointer;

    DLLName: string;

    fOld: Pointer;

    wBytes: DWORD;

  begin

    Result := 0;

    DosHeader := Pointer(h);

    if IsDosHead.IndexOf(DosHeader) >= 0 then

      exit;

    IsDosHead.Add(DosHeader);

    OldPtr := GetAPIAddress(OldPtr); //原函数地址

    if IsBadReadPtr(DosHeader, SizeOf(TIMageDosHeader)) then

      exit;

    if DosHeader.e\_magic <> IMAGE\_DOS\_SIGNATURE then

      exit;

    NTHeader := Pointer(Integer(DosHeader) + DosHeader.\_lfanew);

    VAddr :=

      NTHeader^.OptionalHeader.DataDirectory[IMAGE\_DIRECTORY\_ENTRY\_IMPORT].VirtualAddress;

    if VAddr = 0 then

      exit;

    impEty := Pointer(Integer(DosHeader) + VAddr);

    while ImpEty^.Name <> 0 do

    begin

      DllName := PChar(Integer(DosHeader) + ImpEty^.Name);

      hkSwapPtr(GetModuleHandle(PChar(dllName)), OldPtr, NewPtr);

      Func := Pointer(Integer(DosHeader) + ImpEty^.LookupTable);

      while Func^ <> nil do

      begin

        fOld := GetAPIAddress(Func^);

        if fOld = OldPtr then

        begin

          WriteProcessMemory(GetCurrentProcess, Func, @NewPTr, 4, wBytes);

          if WBytes > 0 then

            inc(Result);

        end;

        inc(Func);

      end;

      Inc(ImpEty);

    end;

  end;

begin

  IsDosHead := TList.Create;

  try

    Result := hkSwapPtr(GetModuleHandle(nil), OldPtr, NewPtr);

  except

  end;

  IsDosHead.Free;

end;

procedure DoHook;

begin

  if @OldBox = nil then

    @OldBox := GetApiAddress(@MessageBox);

  if @OldBoxA = nil then

    @OldBoxA := GetApiAddress(@MessageBoxA);

  if @OldBoxW = nil then

    @OldBoxW := GetApiAddress(@MessageBoxW);

  SwapPtr(@OldBox, @MyBox);

  SwapPtr(@OldBoxA, @MyBoxA);

  SwapPtr(@OldBoxW, @MyBoxW);

end;

begin

  DoHook;

  MessageBox(0, 'hello', 'hello', 0);

  readln;

end.
```

![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/15.gif)![](https://bbs.kanxue.com/view/img/face/16.gif)![](https://bbs.kanxue.com/view/img/face/16.gif)![](https://bbs.kanxue.com/view/img/face/16.gif)![](https://bbs.kanxue.com/view/img/face/16.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
