
#### BitPMC1

```
BitPMC1: array[0..55] of Byte =
  ( 56, 48, 40, 32, 24, 16,  8,
    0, 57, 49, 41, 33, 25, 17,
    9,  1, 58, 50, 42, 34, 26,
   18, 10,  2, 59, 51, 43, 35,
   62, 54, 46, 38, 30, 22, 14,
    6, 61, 53, 45, 37, 29, 21,
   13,  5, 60, 52, 44, 36, 28,
   20, 12,  4, 27, 19, 11,  3 );
```

#### BitPMC2

```
BitPMC2: array[0..47] of Byte =
  ( 13, 16, 10, 23,  0,  4,
    2, 27, 14,  5, 20,  9,
   22, 18, 11,  3, 25,  7,
   15,  6, 26, 19, 12,  1,
   40, 51, 30, 36, 46, 54,
   29, 39, 50, 44, 32, 47,
   43, 48, 38, 55, 33, 52,
   45, 41, 49, 35, 28, 31 );
```

#### 变量声明

```
var
  subKey: array[0..15] of TKeyByte;
```

#### 实现

```
implementation

procedure initPermutation(var inData: array of Byte);
var
  newData: array[0..7] of Byte;
  i: Integer;
begin
  FillChar(newData, 8, 0);
  for i := 0 to 63 do
    if (inData[BitIP[i] shr 3] and (1 shl (7- (BitIP[i] and $07)))) <> 0 then
      newData[i shr 3] := newData[i shr 3] or (1 shl (7-(i and $07)));
  for i := 0 to 7 do inData[i] := newData[i];
end;

procedure conversePermutation(var inData: array of Byte);
var
  newData: array[0..7] of Byte;
  i: Integer;
begin
  FillChar(newData, 8, 0);
  for i := 0 to 63 do
    if (inData[BitCP[i] shr 3] and (1 shl (7-(BitCP[i] and $07)))) <> 0 then
      newData[i shr 3] := newData[i shr 3] or (1 shl (7-(i and $07)));
  for i := 0 to 7 do inData[i] := newData[i];
end;

procedure expand(inData: array of Byte; var outData: array of Byte);
var
  i: Integer;
begin
  FillChar(outData, 6, 0);
  for i := 0 to 47 do
    if (inData[BitExp[i] shr 3] and (1 shl (7-(BitExp[i] and $07)))) <> 0 then
      outData[i shr 3] := outData[i shr 3] or (1 shl (7-(i and $07)));
end;

procedure permutation(var inData: array of Byte);
var
  newData: array[0..3] of Byte;
  i: Integer;
begin
  FillChar(newData, 4, 0);
  for i := 0 to 31 do
    if (inData[BitPM[i] shr 3] and (1 shl (7-(BitPM[i] and $07)))) <> 0 then
      newData[i shr 3] := newData[i shr 3] or (1 shl (7-(i and $07)));
  for i := 0 to 3 do inData[i] := newData[i];
end;

function si(s,inByte: Byte): Byte;
var
  c: Byte;
begin
  c := (inByte and $20) or ((inByte and $1e) shr 1) or
    ((inByte and $01) shl 4);
  Result := (sBox[s][c] and $0f);
end;

procedure permutationChoose1(inData: array of Byte;
  var outData: array of Byte);
var
  i: Integer;
begin
  FillChar(outData, 7, 0);
  for i := 0 to 55 do
    if (inData[BitPMC1[i] shr 3] and (1 shl (7-(BitPMC1[i] and $07)))) <> 0 then
      outData[i shr 3] := outData[i shr 3] or (1 shl (7-(i and $07)));
end;

procedure permutationChoose2(inData: array of Byte;
  var outData: array of Byte);
var
  i: Integer;
begin
  FillChar(outData, 6, 0);
  for i := 0 to 47 do
    if (inData[BitPMC2[i] shr 3] and (1 shl (7-(BitPMC2[i] and $07)))) <> 0 then
      outData[i shr 3] := outData[i shr 3] or (1 shl (7-(i and $07)));
end;

procedure cycleMove(var inData: array of Byte; bitMove: Byte);
var
  i: Integer;
begin
  for i := 0 to bitMove - 1 do
  begin
    inData[0] := (inData[0] shl 1) or (inData[1] shr 7);
    inData[1] := (inData[1] shl 1) or (inData[2] shr 7);
    inData[2] := (inData[2] shl 1) or (inData[3] shr 7);
    inData[3] := (inData[3] shl 1) or ((inData[0] and $10) shr 4);
    inData[0] := (inData[0] and $0f);
  end;
end;

procedure makeKey(inKey: array of Byte; var outKey: array of TKeyByte);
const
  bitDisplace: array[0..15] of Byte =
    ( 1,1,2,2, 2,2,2,2, 1,2,2,2, 2,2,2,1 );

var
  outData56: array[0..6] of Byte;
  key28l: array[0..3] of Byte;
  key28r: array[0..3] of Byte;
  key56o: array[0..6] of Byte;
  i: Integer;

begin
  permutationChoose1(inKey, outData56);
  key28l[0] := outData56[0] shr 4;
  key28l[1] := (outData56[0] shl 4) or (outData56[1] shr 4);
  key28l[2] := (outData56[1] shl 4) or (outData56[2] shr 4);
  key28l[3] := (outData56[2] shl 4) or (outData56[3] shr 4);
  key28r[0] := outData56[3] and $0f;
  key28r[1] := outData56[4];
  key28r[2] := outData56[5];
  key28r[3] := outData56[6];
  for i := 0 to 15 do
  begin
    cycleMove(key28l, bitDisplace[i]);
    cycleMove(key28r, bitDisplace[i]);
    key56o[0] := (key28l[0] shl 4) or (key28l[1] shr 4);
    key56o[1] := (key28l[1] shl 4) or (key28l[2] shr 4);
    key56o[2] := (key28l[2] shl 4) or (key28l[3] shr 4);
    key56o[3] := (key28l[3] shl 4) or (key28r[0]);
    key56o[4] := key28r[1];
    key56o[5] := key28r[2];
    key56o[6] := key28r[3];
    permutationChoose2(key56o, outKey[i]);
  end;
end;

procedure encry(inData, subKey: array of Byte;
   var outData: array of Byte);
var
  outBuf: array[0..5] of Byte;
  buf: array[0..7] of Byte;
  i: Integer;
begin
  expand(inData, outBuf);
  for i := 0 to 5 do outBuf[i] := outBuf[i] xor subKey[i];
                      // outBuf       xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx
  buf[0] := outBuf[0] shr 2;                         //xxxxxx -> 2
  buf[1] := ((outBuf[0] and $03) shl 4) or (outBuf[1] shr 4); // 4 <- xx xxxx -> 4
  buf[2] := ((outBuf[1] and $0f) shl 2) or (outBuf[2] shr 6); //        2 <- xxxx xx -> 6
  buf[3] := outBuf[2] and $3f;                       //                    xxxxxx
  buf[4] := outBuf[3] shr 2;                         //                       xxxxxx
  buf[5] := ((outBuf[3] and $03) shl 4) or (outBuf[4] shr 4); //                       xx xxxx
  buf[6] := ((outBuf[4] and $0f) shl 2) or (outBuf[5] shr 6); //                              xxxx xx
  buf[7] := outBuf[5] and $3f;                       //                                  xxxxxx

  for i := 0 to 7 do buf[i] := si(i, buf[i]);

  for i := 0 to 3 do outBuf[i] := (buf[i*2] shl 4) or buf[i*2+1];

  permutation(outBuf);

  for i := 0 to 3 do outData[i] := outBuf[i];
end;

procedure desData(desMode: TDesMode;
  inData: array of Byte; var outData: array of Byte);
// inData, outData 都为 8Bytes，否则出错
var
  i, j: Integer;
  temp, buf: array[0..3] of Byte;
begin
  for i := 0 to 7 do outData[i] := inData[i];
  initPermutation(outData);
  if desMode = dmEncry then
  begin
    for i := 0 to 15 do
    begin
      for j := 0 to 3 do temp[j] := outData[j];             //temp = Ln
      for j := 0 to 3 do outData[j] := outData[j + 4];       //Ln+1 = Rn
```
