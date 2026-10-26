
```pascal
  s3 := '';

  while i <= Length(s2) do

  begin

    s := copy(s2, i, 5);

    w := StrToInt(s);

    s1 := '';

    if w > 0 then

    repeat

      a := w div $25;

      c := w mod $25;

      s1 := buf[c+1] + s1;

      w := a;

    until a = 0;

    s3 := s3 + s1;

    i := i + 5;

  end;

  userLen := Length(s3);

  s1 := '';

  for i := 1 to $c do

  begin

    AIndex := i * userLen div $C;

    s1 := s1 + UpperCase(s3[AIndex]);

  end;

  result := IntToHex(f2(s1), 8);

  result := Format('%s%s%s%s%s%s%s%s', [result[7],

    result[8], result[5], result[6], result[3],

    result[4], result[1], result[2]]);

end;

type

  TKey128 = array [0..15] of Byte;

  TLQCBlock  = array[0..1] of LongInt;     { LockBox Quick Cipher }

var

  Key: TKey128 = ($1D, $0D, $AF, $95, $5E, $36, $6C, $D6, $DA, $57,

    $F4, $46, $D5, $D7, $14, $C2);

  RegCode: string;

procedure EncryptLQC(const Key : TKey128; var Block : TLQCBlock; Encrypt : Boolean);

const

  CKeyBox : array [False..True, 0..3, 0..2] of LongInt =

    (((0, 3, 1), (2, 1, 3), (1, 0, 2), (3, 2, 0)),

     ((3, 2, 0), (1, 0, 2), (2, 1, 3), (0, 3, 1)));

var

  KeyInts : array [0..3] of Longint;                                 {!!.01}

  Blocks  : array [0..1] of Longint;                                 {!!.01}

  Work    : LongInt;

  Right   : LongInt;

  Left    : LongInt;

  R       : LongInt;

  AA, BB  : LongInt;

  CC, DD  : LongInt;

begin

  Move(Key, KeyInts, SizeOf(KeyInts));                               {!!.01}

  Move(Block, Blocks, SizeOf(Blocks));                               {!!.01}

  Right := Blocks[0];

  Left := Blocks[1];

  for R := 0 to 3 do begin

    {transform the right side}

    AA := Right;

    BB := KeyInts[CKeyBox[Encrypt, R, 0]];

    CC := KeyInts[CKeyBox[Encrypt, R, 1]];

    DD := KeyInts[CKeyBox[Encrypt, R, 2]];

    {commented code does not affect results - removed for speed}

    AA := AA + DD; DD := DD + AA; AA := AA xor (AA shr 7);

    BB := BB + AA; AA := AA + BB; BB := BB xor (BB shl 13);

    CC := CC + BB; BB := BB + CC; CC := CC xor (CC shr 17);

    DD := DD + CC; CC := CC + DD; DD := DD xor (DD shl 9);

    AA := AA + DD; DD := DD + AA; AA := AA xor (AA shr 3);

    BB := BB + AA; {AA := AA + BB;}  BB := BB xor (BB shl 7);

    CC := CC + BB; {BB := BB + CC;}  CC := CC xor (DD shr 15);

    DD := DD + CC; {CC := CC + DD;}  DD := DD xor (DD shl 11);

    Work := Left xor DD;

    Left := Right;

    Right := Work;

  end;

  Blocks[0] := Left;

  Blocks[1] := Right;

  Move(Blocks, Block, SizeOf(Block));                                {!!.01}

end;

function BufferToHex(const Buf; BufSize : Cardinal) : string;

var

  I     : LongInt;

begin

  Result := '';

  for I := 0 to BufSize - 1 do

    Result := Result + IntToHex(TByteArray(Buf)[I], 2);              {!!.01}

end;

procedure Generate(hWnd: THandle);

var

  k1, k2: byte;

  Block: TLQCBlock;

begin

  Randomize;

  k1 := Random($FF);

  k2 := Random($FF);

  RegCode := '5B9C' + IntToHex(k1, 2) + IntToHex(k2, 2);

  sn := error;

  SendMessage(hName, WM_GETTEXT, 256, INTEGER(name));

  if Length(name) > 0 then

  begin

    RegCode := RegCode + ComputUsernameValue(name);

    Block[0] := StrToInt('$' + Copy(RegCode, 1, 2));

    Block[0] := Block[0] or StrToInt('$' + Copy(RegCode, 3, 2)) shl 8;

    Block[0] := Block[0] or StrToInt('$' + Copy(RegCode, 5, 2)) shl 16;

    Block[0] := Block[0] or StrToInt('$' + Copy(RegCode, 7, 2)) shl 24;

    Block[1] := StrToInt('$' + Copy(RegCode, 9, 2));

    Block[1] := Block[1] or StrToInt('$' + Copy(RegCode, 11, 2)) shl 8;

    Block[1] := Block[1] or StrToInt('$' + Copy(RegCode, 13, 2)) shl 16;

    Block[1] := Block[1] or StrToInt('$' + Copy(RegCode, 15, 2)) shl 24;

    EncryptLQC(Key, Block, True);

    RegCode := BufferToHex(Block, SizeOf(Block));//Format('%x, %x', [Block[0], Block[1]]);

    sn := PChar(RegCode);

  end;

  SendMessage(hSn, WM_SETTEXT, 0, Integer(sn));

  SendMessage(hSn, EM_SETSEL, 0, -1);

  SendMessage(hSn, WM_COPY, 0, 0);

end;
```

---

## 破解总结

此软件用了 Lockbox Quick Cipher 加密算法, 用户名变换部份较为复杂

我的注册码

```text
+dEMON
AC3F1B9052E59692
```

注册信息放在 `HKEY_CLASSES_ROOT\CLSID\{AF670D53-621C-4647-98CD-E7EDC20524EA}\152877354`

```text
[HKEY_CLASSES_ROOT\CLSID\{AF670D53-621C-4647-98CD-E7EDC20524EA}\152877354]
"0"="74627D6A7B736C657C"
"2"="6E330A1F030A"
"3"="04147C147D0666607D62087472736B77"
```

加了密了，真变态

删除 `HKEY_CLASSES_ROOT\CLSID\{AF670D53-621C-4647-98CD-E7EDC20524EA}` 下的键就可以重新注册了

---

## 感谢

ForEver, MengLong, xIkUg, fly, kvllz, jwh51, bbbsl, winroot, xy2000, kanxue, heXer, sUNbIRD, forgot, cnbragon, hmimys, yyxzz, CrackerABC, newlaos, woyao, linhanshi, DarkNeesOut, LeNgHoSt, shoooo, window, vcasm, lgl

all RCT members, and you!

---

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

本代码的着色效果由 `xTiNt` 自动完成

下载 `xTiNt` `483K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0t1I4x3g2)9J5k6e0V1H3i4K6u0W2y4K6g2Q4x3X3f1^5y4q4)9J5c8Y4N6W2j5W2)9J5c8X3E0S2L8X3q4#2L8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3P5q4c8A6e0Y4c8Q4x3X3g2J5j5i4t1`.[附件:keygensrc.rar](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/4/files/keygensrc.rar)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
