
# 数据包分析

## 接收数据包 `Recv`

`2   218.22.13.17   10.66.6.71   128   Recv`

```
0000  8F 69 50 6A 6C 71 32 58 76 30 57 4B 71 46 2B 6K    .iPjlq2Xv0WKqF+k
0010  43 7A 58 6B 52 48 67 3D 3D 00 00 00 00 00 00 00    CzXkRHg==.......
0020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030  00 00 00 00 00 00 00 00 00 00 00 36 F8 4E E3 00    ...........6.N..
0040  00 00 00 00 58 02 00 00 20 03 00 00 58 02 00 00    ....X... ...X...
0050  20 03 00 00 00 00 00 00 00 00 00 00 97 A4 C5 77      ..............w
0060  00 00 00 00 20 8F 9C 0A 00 23 2F 0B 00 00 00 00    .... ...#/.....
0070  00 00 00 00 00 00 00 00 90 BE 70 73 60 08 15 00    ..........ps`.
```

## 发送数据包 `Send`

```
0000  17 4F 4B 00 00 00 00 00 00 01 BC B0 CA B1 D3 EA    .OK.............
0010  50 4B B0 E6 BC B4 C8 D5 C6 F0 C3 E2 B7 D1 BF AA    PK..............
0020  B7 C5 2C C3 E2 B7 D1 D5 CA BA C5 A3 BA BC B0 CA    ..,.............
0030  B1 D3 EA A3 AC C3 DC C2 EB A3 BA 36 36 36 00 00    ...........666..
0040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0050  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060  00 00 36 40 C5 13 20 D5 CF 13 4A AC 2E 00 00 00    ..6@.. ...J.....
0070  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
```

## 服务器端解密验证逻辑

以下是服务端用于解密验证相关代码：

```pascal
// 返回的数据包结构
type
  PrecvDat = ^TrecvDat;
  TrecvDat = packed Record
    Tou:Byte;
    Str:string[54];
    Dontkonw_0:DWord;
    Dontkonw_1:DWord;
    Dontkonw_2:DWord;
    Dontkonw_3:DWord;
    Dontkonw_4:DWord;
    Dontkonw_5:DWord;
    Dontkonw_6:DWord;
    Dontkonw_7:DWord;
    Dontkonw_8:DWord;
    Dontkonw_9:DWord;
    Dontkonw_10:DWord;
    z:byte;
    DTlsGetValue:DWord;
    DGetTickCount:DWord;
    DontKonw_11:string[18];
  end;

// 模拟发送的数据包结构
type
  PsendDat = ^TsendDat;
  TsendDat = packed Record
    Tou:Byte;
    Str1:array[0..7]of byte;
    Fd:Byte;
    Str2:string[87];
    Data1:DWord;
    Data2:DWord;
    Data3:Dword;
    DontKonw:array[0..17] of byte;
  end;

// 第 1 个外挂发送来的封包进行解密效验计算
function DeRecv_0:boolean;
begin
result:=decode(recvbuf,3,$2e,$d9);
end;

// 对第 2 个外挂发送来的封包进行解密效验计算
function DeRecv_1:boolean;
begin
result:=decode(recvbuf,$3f,$2e,$d9);
end;

// 第 3 个外挂发送来的封包我们需要取得 2 个有用的数据用来计算发送的封包的数据
procedure DeRecv_2();
var
Temp:trecvDat;
begin
  Move(recvbuf,temp,$80);
  TlGValue:=temp.DTlsGetValue;
  TGTCount:=temp.DGetTickCount;
end;

// 模拟发送的第 1 个数据封包
procedure Ensend_0;
begin
FillChar(sendBuf[0],sizeof(sendbuf[0]),0);

sendBuf[0][0]:=$55;
sendBuf[0][1]:=$4f;
sendBuf[0][2]:=$4b;
sendBuf[0][$7b]:=$25;
sendBuf[0][$7c]:=$c3;

EnCode(sendBuf[0],$80,$2e,$d9);
end;

// 模拟发送的第 2 个数据封包
procedure Ensend_1;
begin
FillChar(sendBuf[1],sizeof(sendbuf[1]),0);

sendBuf[1][1]:=$4f; sendBuf[1][2]:=$4b;sendBuf[1][51]:=$9a;sendBuf[1][52]:=$2;

sendBuf[1][55]:=$77;sendBuf[1][56]:=$d0;sendBuf[1][57]:=$c2;sendBuf[1][58]:=$16;

sendBuf[1][60]:=$60;sendBuf[1][61]:=$f5;sendBuf[1][62]:=$e4;sendBuf[1][63]:=$8f;

sendBuf[1][64]:=$8f;sendBuf[1][65]:=$2b;sendBuf[1][66]:=$37;sendBuf[1][67]:=$56;

sendBuf[1][68]:=$59;sendBuf[1][69]:=$74;sendBuf[1][70]:=$33;sendBuf[1][71]:=$52;

sendBuf[1][72]:=$38;sendBuf[1][73]:=$6f;sendBuf[1][74]:=$44;sendBuf[1][75]:=$66;

sendBuf[1][76]:=$76;sendBuf[1][77]:=$4d;sendBuf[1][78]:=$44;sendBuf[1][79]:=$32;

sendBuf[1][80]:=$4f;sendBuf[1][81]:=$77;sendBuf[1][82]:=$58;sendBuf[1][83]:=$4e;

sendBuf[1][84]:=$77;sendBuf[1][85]:=$75;sendBuf[1][86]:=$41;sendBuf[1][87]:=$3d;

sendBuf[1][88]:=$3d;sendBuf[1][123]:=$d4;sendBuf[1][124]:=$a9;sendBuf[1][125]:=$76;

sendBuf[1][126]:=$ec;

EnCode(sendBuf[1],$80,$2e,$d9);
end;

// 模拟发送的第 3 个数据封包
procedure Ensend_2();
var
senddata:TsendDat;
data_1:dword;
  a:word; s:string;
begin
FillChar(sendBuf[2],sizeof(sendbuf[2]),0);

data_1:=$11111111;//这个是我自己设定的数据,可以设为任意值

senddata.Tou:=$17;

FillChar(senddata.Str1,8,0);

senddata.Str1[0]:=$4f;

senddata.Str1[1]:=$4b;//填充数据包头

senddata.Fd:=01;

FillChar(senddata.Str2,88,0);

senddata.Str2:='我是 PowerBoy 我破解的好用吗?呵呵!';

senddata.Data1:=Data_1;                   //设定的 DATA_1 进行填充

senddata.Data2:=data_1 xor TlGValue xor $999;//由 DATA_1 算出 DATA_2 填充

senddata.Data3:=TGTCount xor $258369;      //DATA_3 也是上面数据经过计算之后进行填充

FillChar(senddata.DontKonw ,18,0);          //封包其余部分都为 00

Move(senddata,sendBuf[2],$80);
end;
```

## 加解密汇编实现

```pascal
procedure EnCode(Indata: Pointer; len: integer; Key1 :Byte; Key2 :Byte);stdcall;
asm
          PUSH    EBP

          PUSH    EBX

          PUSH    ESI

          MOV     EBP,ESP

          MOV     EDX,LEN

          XOR     ECX,ECX

          XOR     EAX,EAX

          DEC     EDX

          MOV     ESI,indata

@@loc_00401DE0: ADD     AL,[ESI+ECX]

          INC     ECX

          CMP     ECX,EDX

          JNZ     @@loc_00401DE0

          XOR     AL,key1

          MOV     DL,key2

          IMUL    DL

          MOV     [ESI+ECX],AL

          MOV     EDX,4BCh

          XOR     ECX,ECX

          XOR     EAX,EAX

          MOV     ESI,indata

@@loc_00401DFF: MOV     AL,[ESI+ECX]

          MOV     BL,DH

          XOR     BL,AL

          MOVZX   EAX,BL

          ADD     EAX,EDX

          MOV     [ESI+ECX],BL

          LEA     EDX,[EAX+EAX*2]

          SHL     EDX,3

          SUB     EDX,EAX

          SHL     EDX,3

          SUB     EDX,EAX

          LEA     EDX,[EAX+EDX*4]

          SHL     EDX,5

          SUB     EDX,EAX

          SUB     EDX,3596h

          INC     ECX

          CMP     ECX,len

          JNZ     @@loc_00401DFF

          POP     ESI

          POP     EBX

          POP     EBP
end;

function DeCode(var indata:Tbuff;len:integer;Key1:Byte;Key2:Byte):boolean;
var
t:Pointer;
begin
t:=@indata;

asm
          PUSH    EBP

          MOV     EBP,ESP

          PUSH    ECX

          PUSH    EBX

          PUSH    ESI

          MOV     EDX,4BCh

          XOR     ECX,ECX

          XOR     EAX,EAX

          MOV     ESI,t

@@loc_00401D62: MOV     AL,[ESI+ECX]

          MOV     BL,DH

          XOR     BL,AL

          MOVZX   AX,AL

          ADD     EAX,EDX

          MOV     [ESI+ECX],BL

          LEA     EDX,[EAX+EAX*2]

          SHL     EDX,3

          SUB     EDX,EAX

          SHL     EDX,3

          SUB     EDX,EAX

          LEA     EDX,[EAX+EDX*4]

          SHL     EDX,5

          SUB     EDX,EAX

          SUB     EDX,3596h

          INC     ECX

          CMP     ECX,len

          JNZ     @@loc_00401D62

          MOV     EDX,len

          XOR     ECX,ECX

          XOR     EAX,EAX

          DEC     EDX

          MOV     ESI,t

@@loc_00401D9E: ADD     AL,[ESI+ECX]

          INC     ECX

          CMP     ECX,EDX

          JNZ     @@loc_00401D9E

          XOR     AL,key1

          MOV     DL,key2

          IMUL    DL

          CMP     [ESI+ECX],AL
```
