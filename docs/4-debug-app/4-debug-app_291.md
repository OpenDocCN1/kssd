

# 扫雷作弊的 delphi 代码

> 原文：[`www.pediy.com/kssd/pediy06/pediy6047.htm`](https://www.pediy.com/kssd/pediy06/pediy6047.htm)

unit ForWinmineU;   
  
interface   
  
uses   
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,   
  Dialogs, StdCtrls, ExtCtrls, Grids, ComCtrls;   
  
type   
  TForm1 = class(TForm)   
    btnStart: TButton;   
    RadioGroup1: TRadioGroup;   
    Button4: TButton;   
    StringGrid1: TStringGrid;   
    btnContinue: TButton;   
    CheckBox1: TCheckBox;   
    Label1: TLabel;   
    UpDown1: TUpDown;   
    Edit1: TEdit;   
    CheckBox2: TCheckBox;   
    btnCheat: TButton;   
    procedure btnStartClick(Sender: TObject);   
    procedure RadioGroup1Click(Sender: TObject);   
    procedure FormClose(Sender: TObject; var Action: TCloseAction);   
    procedure Button4Click(Sender: TObject);   
    procedure btnContinueClick(Sender: TObject);   
    procedure FormCreate(Sender: TObject);   
    procedure CheckBox1Click(Sender: TObject);   
    procedure btnCheatClick(Sender: TObject);   
  private   
    { Private declarations }   
    procedure OnHotKey(var msg: TMessage); message WM\_HOTKEY;   
  public   
    { Public declarations }   
  end;   
  
type   
  TCell = record   
    Value: char;   
    bNeed: boolean;   
  end;   
const   
  dx: array\[1..8\] of short = (-1, 0, 1, 1, 1, 0, -1, -1);   
  dy: array\[1..8\] of short = (-1, -1, -1, 0, 1, 1, 1, 0);   
var   
  Form1: TForm1;   
  hSweep: THandle;   
  mSweep: HMenu;   
  dcSweep: HDC;   
  atomSweep: ATOM;   
  ColNum, RowNum: short;   
  Cells: array\[0..31, 0..17\] of TCell;   
  
implementation   
  
{$R \*.dfm}   
  
//0..8 表示数字，9 表示没有打开，\*表示雷，F 表示标记   
function GetCellInfo(x, y: short): char;   
var   
  cl0, cl1: DWORD;   
begin   
{0c..colnum\*10+0b}{1..colnum}   
{37..rownum\*10+36}{1..rownum}   
  if (x<1)or(y<1)or(x>ColNum)or(y>RowNum) then   
  begin   
    Result := '0';   
    Exit;   
  end;   
  cl0 := GetPixel(dcSweep, (x-1)\*16+$C+1, (y-1)\*16+$37+1);   
  if cl0=$00FFFFFF then   
  begin   
    cl1 := GetPixel(dcSweep, (x-1)\*16+$C+8, (y-1)\*16+$37+8);   
    case cl1 of   
    $00C6C3C6:  Result := '9';   
    $00000000:  Result := 'F';   
    end;   
    Exit;   
  end;   
  cl1 := GetPixel(dcSweep, (x-1)\*16+$C+8, (y-1)\*16+$37+8);   
//  SetPixel(dcSweep, (x-1)\*16+$C, (y-1)\*16+$37, clRed);   
//  Ellipse(dcSweep, (x-1)\*16+$C-2, (y-1)\*16+$37-2, (x-1)\*16+$C+2, (y-1)\*16+$37+2);   
  case cl1 of   
  $00C6C3C6:   
    begin   
      cl0 := GetPixel(dcSweep, (x-1)\*16+$C+10, (y-1)\*16+$37+10);   
      if cl0=0 then   
        Result := '7'   
      else   
        Result := '0';   
    end;   
  $00FF0000:  Result := '1';   
  $00008200:  Result := '2';   
  $000000FF:  Result := '3';   
  $00840000:  Result := '4';   
  $00000084:  Result := '5';   
  $00848200:  Result := '6';   
  $00848284:  Result := '8';   
  $00000000:  Result := '\*';   
  else   
    Result := '7';   
  end;   
end;   
  
//模拟左键点击   
procedure ClickIt(x, y: short);   
begin   
  if hSweep<=0 then   
    Exit;   
  SendMessage(hSweep, WM\_LBUTTONDOWN, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
  SendMessage(hSweep, WM\_LBUTTONUP, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
end;   
  
//模拟右键标旗子   
procedure FlagIt(x, y: short);   
begin   
  if hSweep<=0 then   
    Exit;   
  SendMessage(hSweep, WM\_RBUTTONDOWN, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
end;   
  
//模拟左右键双击   
procedure DbClickIt(x, y: short);   
begin   
  if hSweep<=0 then   
    Exit;   
  SendMessage(hSweep, WM\_LBUTTONDOWN, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
  SendMessage(hSweep, WM\_RBUTTONDOWN, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
  SendMessage(hSweep, WM\_LBUTTONUP, 0, MAKELONG((x-1)\*16+$C, (y-1)\*16+$37));   
end;      
  
procedure TForm1.OnHotKey(var msg: TMessage);   
begin   
  if (msg.LParamHi=$58)and(msg.LParamLo=MOD\_CONTROL or MOD\_ALT) then   
  begin   
    if hSweep<=0 then   
      Exit;   
    if not btnContinue.Enabled then   
      Exit;   
    btnContinueClick(nil);   
  end;   
end;   
  
procedure TForm1.FormCreate(Sender: TObject);   
begin   
  Randomize;   
  CheckBox1Click(Sender);   
end;   
  
procedure TForm1.btnStartClick(Sender: TObject);   
var   
  pmii: MENUITEMINFO;   
  buf: array\[0..255\] of char;   
  s: string;   
begin   
  hSweep := FindWindow(nil, '扫雷');   
  if hSweep<=0 then   
  begin   
    FillChar(buf, SizeOf(buf), #0);   
    GetSystemDirectory(buf, 255);   
    s := buf;   
    WinExec(PChar(s+'\\winmine.exe'), SW\_SHOW);   
    hSweep := FindWindow(nil, '扫雷');   
  end;   
//  atomSweep := GlobalAddAtom('ForWinMine');   
//  RegisterHotKey(Handle, atomSweep, MOD\_CONTROL or MOD\_ALT or MOD\_SHIFT, $41);   
//  showmessage(inttostr(getlasterror));   
  BringWindowToTop(hSweep);   
  mSweep := GetMenu(hSweep);   
  mSweep := GetSubMenu(mSweep, 0);   
  RadioGroup1Click(Sender);   
end;   
  
procedure TForm1.RadioGroup1Click(Sender: TObject);   
var   
  mID: DWORD;   
begin   
  if (mSweep<=0)or(hSweep<=0) then   
    Exit;   
  case RadioGroup1.ItemIndex of   
  0:   
    begin   
      mID := GetMenuItemID(mSweep, 2);   
      ColNum := 9;   
      RowNum := 9;   
      UpDown1.Max := 3;   
    end;   
  1:   
    begin   
      mID := GetMenuItemID(mSweep, 3);   
      ColNum := 16;   
      RowNum := 16;   
      UpDown1.Max := 5;   
    end;   
  2:   
    begin   
      mID := GetMenuItemID(mSweep, 4);   
      ColNum := 30;   
      RowNum := 16;    
      UpDown1.Max := 10;   
    end;   
  else   
    Exit;   
  end;   
  SendMessage(hSweep, WM\_COMMAND, mID, 0);   
  btnContinue.Enabled := true;   
  if not CheckBox1.Checked then   
    Exit;   
  for mID:=1 to UpDown1.Position do   
    ClickIt(Round(Random(ColNum)+1), Round(Random(RowNum)+1));   
  btnContinueClick(sender);   
end;   
  
procedure TForm1.FormClose(Sender: TObject; var Action: TCloseAction);   
begin   
  if hSweep>0 then   
    SendMessage(hSweep, WM\_CLOSE, 0, 0);   
//  UnRegisterHotKey(Application.Handle, atomSweep);   
//  GlobalDeleteAtom(atomSweep);   
end;   
  
procedure TForm1.Button4Click(Sender: TObject);   
var   
  r, c: short;   
begin   
  if hSweep<=0 then   
    Exit;   
  BringWindowToTop(hSweep);   
  dcSweep := GetDC(hSweep);   
  with StringGrid1 do   
  begin   
    ColCount := ColNum + 2;   
    RowCount := RowNum + 2;   
    for c:=0 to ColNum+1 do   
    begin   
      Cells\[c, 0\] := ' ';   
      Cells\[c, RowNum+1\] := ' ';   
    end;   
    for r:=0 to RowNum+1 do   
    begin   
      Cells\[0, r\] := ' ';   
      Cells\[ColNum+1, r\] := ' ';   
    end;   
    for r:=1 to RowNum do   
      for c:=1 to ColNum do   
      begin   
        Cells\[c, r\] := GetCellInfo(c, r);   
        if (Cells\[c, r\]='9')or(Cells\[c, r\]='0') then   
          Cells\[c, r\] := ' ';   
      end;   
  end;   
  ReleaseDC(hSweep, dcSweep);   
end;   
  
procedure TForm1.btnContinueClick(Sender: TObject);   
var   
  row, col, sum, k: short;   
  fnum, mnum: short;   
  bChange: boolean;   
  
  procedure neighbour;   
  var   
    k: short;   
  begin   
    fnum := 0; mnum := 0;   
    for k:=1 to 8 do   
    begin   
      Cells\[col+dx\[k\], row+dy\[k\]\].Value := GetCellInfo(col+dx\[k\], row+dy\[k\]);   
      if Cells\[col+dx\[k\], row+dy\[k\]\].Value='9' then   
        inc(mnum);   
      if Cells\[col+dx\[k\], row+dy\[k\]\].Value='F' then   
        inc(fnum);   
    end;   
  end;   
  
  procedure FlagAll;   
  var   
    k: short;   
  begin   
    for k:=1 to 8 do   
    begin   
      if Cells\[col+dx\[k\], row+dy\[k\]\].Value='9' then   
        FlagIt(col+dx\[k\], row+dy\[k\]);   
    end;   
  end;   
  
begin   
  if hSweep<=0 then   
    Exit;   
  BringWindowToTop(hSweep);   
  dcSweep := GetDC(hSweep);   
//初始化 Map   
  for row:=0 to RowNum+1 do   
  for col:=0 to ColNum+1 do   
  begin   
//    Cells\[col, row\].Value := GetCellInfo(col, row);   
    Cells\[col, row\].bNeed := true;   
  end;   
  repeat   
    bChange := false;   
    for row:=1 to RowNum do   
    for col:=1 to ColNum do   
    begin   
      if not Cells\[col, row\].bNeed then   
        continue;   
      Cells\[col, row\].Value := GetCellInfo(col, row);   
//点爆了   
      if Cells\[col, row\].Value='\*' then   
      begin                          
        btnContinue.Enabled := false;   
        ReleaseDC(hSweep, dcSweep);   
        Exit;   
      end;   
      if (Cells\[col, row\].Value='0')or(Cells\[col, row\].Value='F') then   
        Cells\[col, row\].bNeed := false;   
      if not(Cells\[col, row\].Value in \['1'..'8'\]) then   
        continue;   
      neighbour;   
      if fnum=Ord(Cells\[col, row\].Value)-$30 then   
      begin   
        DbClickIt(col, row);   
        Cells\[col, row\].bNeed := false;   
        bChange := true;   
      end;   
      if (mnum>0)and(fnum+mnum=Ord(Cells\[col, row\].Value)-$30) then   
      begin   
        FlagAll;   
        bChange := true;   
      end;   
    end;   
    //如果上面的找完了，暂时随机点击   
    if CheckBox2.Checked then   
      if not bChange then   
      begin   
        for row:=1 to RowNum do   
          if not bChange then   
            for col:=1 to ColNum do   
            begin   
              if not Cells\[col, row\].bNeed then   
                continue;   
              if GetCellInfo(col, row)='9' then   
              begin   
                ClickIt(col, row);   
                bChange := true;   
                break;   
              end;   
            end;   
      end;   
  until not bChange;   
  ReleaseDC(hSweep, dcSweep);   
end;   
  
procedure TForm1.CheckBox1Click(Sender: TObject);   
begin   
  Label1.Enabled := CheckBox1.Checked;   
  Edit1.Enabled := CheckBox1.Checked;   
  UpDown1.Enabled := CheckBox1.Checked;   
end;   
  
procedure TForm1.btnCheatClick(Sender: TObject);   
var   
  pID, pHandle, pAddr: DWORD;   
  rNum, cNum, mNum, tmpNum: DWORD;   
  r, c: short;   
  buf: array\[0..$400\] of byte;   
  s: string;   
begin   
  if hSweep<=0 then   
    Exit;   
  GetWindowThreadProcessId(hSweep, pID);   
  pHandle := OpenProcess(PROCESS\_VM\_READ, false, pID);   
  if pHandle<=0 then   
  begin   
    ShowMessage('很遗憾，不能作弊了！');   
    Exit;   
  end;   
  pAddr := $010052C4;   
  ReadProcessMemory(pHandle, Pointer(pAddr), Pointer(@mNum), 4, tmpNum);   
  pAddr := $010052C8;   
  ReadProcessMemory(pHandle, Pointer(pAddr), Pointer(@rNum), 4, tmpNum);   
  pAddr := $010052CC;   
  ReadProcessMemory(pHandle, Pointer(pAddr), Pointer(@cNum), 4, tmpNum);   
  s := Format('行%d，列%d，雷数%d，确定作弊么？', \[rNum, cNum, mNum\]);   
  if MessageDlg(s, mtConfirmation, \[mbYes, mbNo\], 0)=mrYes then   
  begin   
    pAddr := $01005700;   
    ReadProcessMemory(pHandle, Pointer(pAddr), Pointer(@buf), $3FF, tmpNum);   
    for r:=1 to rNum do   
    for c:=1 to cNum do   
    begin   
      {if buf\[r shl 5 + c\] and $80 = $80 then   
        FlagIt(c, r);}   
      if (r=rNum)and(c=cNum) then   
        break;   
      if buf\[r shl 5 + c\] and $80 <> $80 then   
        ClickIt(c, r);   
    end;   
  end;   
  CloseHandle(pHandle);   
end;   
  
end.  4dd466d0e1e0cc123afbe504da2462ca