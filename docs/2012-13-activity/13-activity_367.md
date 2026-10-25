
# 第一阶段 第三题 keygen 代码(Delphi 版)

> 原文：[`www.pediy.com/kssd/pediy11/123539.html`](https://www.pediy.com/kssd/pediy11/123539.html)

代码:

```
unit Unit1;
{
默认情况必须使用 D2009 之前的编译, 否则需要更改设置或者需要修改

程序使用 Delphi 5 编译!

SHA1        -  written by Dave Barton (davebarton@bigfoot.com) 
BASE32    -  没找到现成代码-\_-!, 我偷懒写了一个能算 key 的
}

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls, sha1;

type
  TForm1 = class(TForm)
    eSerial: TEdit;
    eName: TEdit;
    eCode: TEdit;
    Label1: TLabel;
    Label2: TLabel;
    Label3: TLabel;
    cbModify: TCheckBox;
    cbAuto: TCheckBox;
    procedure FormCreate(Sender: TObject);
    procedure eNameChange(Sender: TObject);
    procedure FormActivate(Sender: TObject);
    procedure cbModifyClick(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.DFM}

procedure TForm1.FormCreate(Sender: TObject);
var
  Serial, vv: DWORD;
begin
  GetVolumeInformationA('C:\\',  nil, 0, @Serial, vv, vv, nil, 0);
  eSerial.Text := IntToHex(Serial, 8);
end;

const
  TAB32: array[0..32 - 1] of Char = 'ABCDEFGHJKMNPQRSTVWXYZ1234567890';
  BinTAB: array[0..1] of Char = '01';

function ByteToBin(AByte: Byte): String;
var
  I: Integer;
begin
  Result := '00000000';
  for I := 8 downto 1 do
  begin
    Result[I] := BinTAB[AByte and 1];
    AByte := AByte shr 1;
  end;
end;

function BinToByte(Str: String): Byte;
var
  I, Len: Integer;
begin
  Result := 0;
  Len := Length(Str);
  for I := Len downto 1 do
  begin
    Result := (Result shl 1) or (Ord(Str[Len - I + 1]) - $30);   //0,1
  end;
end;

function MakeCode(Name: String; cSerial: DWORD): String;
var
  Context: TSHA1Context;
  Digest: TSHA1Digest;
  Str: String;
  I: Integer;
begin
  Str := Name;
  Str := Name + Char(cSerial and $FF)
              + Char((cSerial shr 8) and $FF)
              + Char((cSerial shr 16) and $FF)
              + Char((cSerial shr 24) and $FF)
              + 'Tencent';
  SHA1Init(Context);

  Context.Hash[0]:= $B1CAB1CA;
  Context.Hash[1]:= $CCBFCCBF;
  Context.Hash[2]:= $BFB2D6BE;
  Context.Hash[3]:= $F8C7D8B5;
  Context.Hash[4]:= $EEC7BCCD;

  SHA1Update(Context, PAnsiChar(Str), Length(Str));

  SHA1Final(Context, Digest);

  //ABCDEFGHJKMNPQRSTVWXYZ1234567890

  //160bits/5=32
  Str := '';
  for i := 0 to 20 - 1 do
  begin
    Str := Str + ByteToBin(Digest[i]);
  end;

  //
  Result := '';
  for i := 0 to 32 - 1 do
  begin
    Result := Result + Tab32[BinToByte(Copy(Str, i * 5 + 1, 5))];
    if (((I + 1) mod 8) = 0) and (I <> 31)  then
    begin
      Result := Result + '-';
    end;
  end;
end;

procedure TForm1.eNameChange(Sender: TObject);
var
  Name: String;
begin
  Name := eName.Text;
  if (Length(Name) <= 0) or (Length(Name) > 32) then
  begin
    eCode.Text := 'Name must 1-32 char(s) at least!';
  end
  else
  begin
    eCode.Text := MakeCode(Name, StrToIntDef('$' + Trim(eSerial.Text), 0));
    if cbAuto.Checked then
    begin
      eCode.SelectAll;
      eCode.CopyToClipboard;
    end;
  end;
end;

procedure TForm1.FormActivate(Sender: TObject);
begin
  eNameChange(eName);
end;

procedure TForm1.cbModifyClick(Sender: TObject);
begin
  eSerial.Enabled := cbModify.Checked;
  eSerial.ReadOnly := not cbModify.Checked;
  if not cbModify.Checked then
  begin
    FormCreate(Self);
  end;
end;

end.
```

## Unit1.dfm 文件

代码:

```
object Form1: TForm1
  Left = 232
  Top = 175
  BorderIcons = [biSystemMenu, biMinimize]
  BorderStyle = bsSingle
  Caption = 'Keygen'
  ClientHeight = 112
  ClientWidth = 344
  Color = clBtnFace
  Font.Charset = DEFAULT_CHARSET
  Font.Color = clBlack
  Font.Height = -11
  Font.Name = 'MS Sans Serif'
  Font.Style = []
  OldCreateOrder = False
  OnActivate = FormActivate
  OnCreate = FormCreate
  PixelsPerInch = 96
  TextHeight = 13
  object Label1: TLabel
    Left = 8
    Top = 17
    Width = 29
    Height = 13
    Caption = 'Serial:'
  end
  object Label2: TLabel
    Left = 8
    Top = 49
    Width = 31
    Height = 13
    Caption = 'Name:'
  end
  object Label3: TLabel
    Left = 8
    Top = 81
    Width = 28
    Height = 13
    Caption = 'Code:'
  end
  object eSerial: TEdit
    Left = 40
    Top = 13
    Width = 105
    Height = 21
    Enabled = False
    ReadOnly = True
    TabOrder = 0
    OnChange = eNameChange
  end
  object eName: TEdit
    Left = 40
    Top = 45
    Width = 289
    Height = 21
    MaxLength = 32
    TabOrder = 1
    Text = 'PeDiY&TeNcEnT'
    OnChange = eNameChange
  end
  object eCode: TEdit
    Left = 40
    Top = 77
    Width = 289
    Height = 21
    ReadOnly = True
    TabOrder = 2
  end
  object cbModify: TCheckBox
    Left = 152
    Top = 17
    Width = 57
    Height = 17
    Caption = 'Modify'
    TabOrder = 3
    OnClick = cbModifyClick
  end
  object cbAuto: TCheckBox
    Left = 208
    Top = 17
    Width = 129
    Height = 17
    Caption = 'Auto copy to clipboard'
    Checked = True
    State = cbChecked
    TabOrder = 4
    OnClick = cbModifyClick
  end
end
```

跟帖一起晒 src，

## keygen.c

代码:

```
#include <windows.h>
#include <intrin.h>
#include <stdio.h>
#include "sha1.h"

const char pKeyMap[] = "ABCDEFGHJKMNPQRSTVWXYZ1234567890=";

void gethwid(BYTE *pHwid, char *pUserName, DWORD hwidkey);

typedef struct {
  const unsigned char *source;
  unsigned int tag;
  unsigned int bitcount;
} DEPACKDATA;

static int getbit(DEPACKDATA *ud)
{
  unsigned int bit;

  if (!ud->bitcount--)
  {
    ud->tag = *ud->source++;
    ud->bitcount = 7;
  }

  bit = (ud->tag >> 7) & 0x01;
  ud->tag <<= 1;

  return bit;
}

BYTE  get5bit(DEPACKDATA *ud)
{
  BYTE  res;
  res = (getbit(ud) << 4) 
    | (getbit(ud) << 3) 
    | (getbit(ud) << 2) 
    | (getbit(ud) << 1) 
    | (getbit(ud) << 0);
  return res;
}

//进 0x14，出 0x20
void depack(BYTE *source, char *destination)
{
  DEPACKDATA ud = {0};
  int insize = 0x14;
  int outsize = 0x20;

  int i;

  ud.source = source;
  ud.bitcount = 0;

  i = 0;
  do
  {
    destination[i ++] = pKeyMap[get5bit(&ud)];
  }while(i < outsize);
  return;
}

DWORD _bswap(DWORD a)
{
  DWORD res = a;
  __asm
  {
    mov eax, res;
    bswap eax;
    mov res, eax;
  }
  return res;
}

int main(int argc, char *argv[])
{
  char tmp[33] = {0};
  char key[37] = {0};
  BYTE hwid[0x14];
  DWORD  hwidkey;
  int i, j;
  if(argc < 2)
  {
    printf("usage: keygen <username> \[Volume Number of the Driver C:\]\\nexample: keygen 海风月影 A09DDB25\\n");
    return 1;
  }
  else if(argc == 2)
  {
    hwidkey = 0;
  }
  else if(argc >= 3)
  {
    sscanf(argv[2], "%08X", &hwidkey);
  }

  gethwid(hwid, argv[1], hwidkey);
  depack(hwid, tmp);
  for(i = 0, j = 0; i < 32;)
  {
    key[j ++] = tmp[i ++];
    if(i % 8 == 0)
    {
      key[j ++] = '-';
    }
  }
  key[--j] = '\x0';
  printf("%s\n", key);
  return 0;
}

//////////////////////////////////////////////////////////////////////////
const char holyshit[] = "时时刻刻局部地区图穷";
void gethwid(BYTE *pHwid, char *pUserName, DWORD hwidkey)  //sha1,0x14
{
  char    buf[36 + 4 + 7] = {0};
  SHA1Context sha1;
  DWORD    key;
  int      len = strlen(pUserName);
  int      i;
  SHA1Reset(&sha1);
  memcpy(sha1.Message_Digest, holyshit, 0x14);

  if(hwidkey != 0)
  {
    key = hwidkey;
  }
  else
  {
    GetVolumeInformationA("c:\\\\", NULL, NULL, &key, NULL, NULL, NULL, NULL);
  }

  strcpy(buf, pUserName);
  *(DWORD*)&buf[len] = key;
  strcpy(&buf[len + 4], "Tencent");

  len = len + 4 + 7;

  SHA1Input(&sha1, buf, len);

  SHA1Result(&sha1);

  for(i = 0; i < 5; i ++)
  {
    *(DWORD*)&pHwid[i * 4] = _bswap(sha1.Message_Digest[i]);
  }
  return ;
}
```

跟随 膜拜楼上

代码:



```cpp
#include "stdafx.h"

#include "sha1.h"
#include "base32.h "
#include <windows.h>

const char *szTX = "Tencent";
const char *szVolume="C:\\\\";


void SHA1Init_m(SHA1_CTX* context)//修改初始化
{
    /* SHA1 initialization constants 时时刻刻局部地区图穷*/
    context->state[0] = 0xB1CAB1CA;
    context->state[1] = 0xCCBFCCBF;
    context->state[2] = 0xBFB2D6BE;
    context->state[3] = 0xF8C7D8B5;
    context->state[4] = 0xEEC7BCCD;
    context->count[0] = context->count[1] = 0;
}

int base32_encode_m(const uint8_t *data, int length, uint8_t *result, int bufSize){     
  if (length < 0 || length > (1 << 28)) {
    return -1;     
  }     
  int count = 0;     
  if (length > 0) {
    int buffer = data[0];
    int next = 1;       
    int bitsLeft = 8;       
    while (count < bufSize && (bitsLeft > 0 || next < length)) {
      if (bitsLeft < 5) {           
        if (next < length) {             
          buffer <<= 8;             
          buffer |= data[next++] & 0xFF;             
          bitsLeft += 8;           
        } else {
          int pad = 5 - bitsLeft;             
          buffer <<= pad;             
          bitsLeft += pad;           
        }         
      }         
      int index = 0x1F & (buffer >> (bitsLeft - 5));         
      bitsLeft -= 5;         
      result[count++] = "ABCDEFGHJKMNPQRSTVWXYZ1234567890"[index];  //修改 XXOO     
    }     
  }     
  if (count < bufSize) {       
    result[count] = '\000';     
  }     
  return count;   
}   

int _tmain(int argc, _TCHAR* argv[])
{
  char *username=new char[512];

  unsigned char digest[20] = {0};

  unsigned char outkey[255] = {0};

  DWORD dwVolSerial;

  printf("Made by GuluYZ 2010.10.22\n\n");

  GetVolumeInformationA(szVolume, NULL, 0, &dwVolSerial, NULL, NULL, NULL, 0);

  printf("VolumeSerialNumber:%08X\n\n", dwVolSerial);

  printf("Please enter your user name (Max length 35):");

  scanf("%s", username);

  int nuserlen = strlen(username);

  if (nuserlen < 0 ||  nuserlen > 35)
  {
    printf("User name length error\n");

    system("pause");

    return 0;
  }
  
  memcpy((byte *)(username + nuserlen), &dwVolSerial, 4);

  memcpy((byte*)(username+nuserlen + 4), szTX, 7);

  //UserName+VolumeSerialNumber+"Tencent" -> SHA_M -> BASE32_M

  SHA1_CTX * newCTX=new SHA1_CTX;

  SHA1Init_m (newCTX);//初始化 SHA 时时刻刻局部地区图穷

  SHA1Update (newCTX,  (unsigned char *)username,  nuserlen + 0xB );//UPDATE SHA

  SHA1Final (digest,  newCTX);//Get SHA 

  base32_encode_m(digest, 0x20, outkey, 0x20);// SHA TO BASE32

  printf("Key:%8.8s-%8.8s-%8.8s-%8.8s\n\n",(byte *)outkey,  (byte *)(outkey+8), (byte *)(outkey+16),  (byte *)(outkey + 24));

  system("pause");

  delete username;

  return 0;
}
```

我也添一段:

```cpp
int m_nLen[5]={2,2,1,2,1};

BOOL Exhaustive(char *lpLienceCode,int nDeep,int nIndex)

{  

  BOOL bResult=FALSE;  

  if (nDeep>=0x20)  

  {  

    return TRUE;  

  }  

  if (m_nLen[nIndex%5]==2)  

  {  

    for (int i=0;i<0x400;i++)  

    {  

      lpLienceCode[nDeep+0]=m_sz2Char[i][0];  

      lpLienceCode[nDeep+1]=m_sz2Char[i][1];  

      if (CheckLienceCode(lpLienceCode,nDeep+2,m_szBuffer))  

      {  

        if (Exhaustive(lpLienceCode,nDeep+2,nIndex+1))  

        {  

          return TRUE;  

        }  

      }  

    }  

  }  

  else if (m_nLen[nIndex%5]==1)  

  {  

    for (int i=0;i<0x20;i++)  

    {  

      lpLienceCode[nDeep+0]=m_sz1Char[i];  

      if (CheckLienceCode(lpLienceCode,nDeep+1,m_szBuffer))  

      {  

        if (Exhaustive(lpLienceCode,nDeep+1,nIndex+1))  

        {  

          return TRUE;  

        }        

      }  

    }  

  }  

  return bResult;  

}
```
