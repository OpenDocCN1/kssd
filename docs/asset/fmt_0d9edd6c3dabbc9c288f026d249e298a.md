
```delphi
T_SLM_KEYDATA = packed record
    ...;
    acFeature:array[0..3] of char;
    acbyUmknown:array[0..28] of char;
    acServerCodeSum:array[0..3] of char;
    ...
end;

PT_SLM_KEYDATA = ^T_SLM_KEYDATA;
PT_SLM_INSTANCE = ^DWORD;
T_slm_startapi = function(var pSlm_instance:PT_SLM_INSTANCE):integer; stdcall;
T_slm_endapi = function(pSlm_instance:PT_SLM_INSTANCE):integer; stdcall;
T_slm_key = function(pSlm_instance:PT_SLM_INSTANCE;
way:integer;key:pchar;keydata:PT_SLM_KEYDATA;options:integer):integer; stdcall;
T_DoEnCodeOrCode = function(pSlm_instance:PT_SLM_INSTANCE;arg_out:pchar;
    arg_way:integer;arg_in:pchar):integer;stdcall; 
T_DoGetSumInAsc4 = function(pcIn,pcOut:pchar):integer;stdcall; 


function CalcServerCodeSum(sServerCode:string):string;
var
  acDeCode:array[0..1024] of char;
  acSum:array[0..1024] of char;
begin
  fillchar(acDeCode,sizeof(acDeCode),#0);
  fillchar(acSum,sizeof(acDeCode),#0);
  slm_DoEnCodeOrCode(p_slm_instance,@acDeCode,1,pchar(sServerCode));
  slm_DoGetSumInAsc4(@acDeCode,@acSum);
  result := result + acSum[0];
  result := result + acSum[1];
  result := result + acSum[2];
  result := result + acSum[3];
end;


function InitElanlmApi:integer;
begin
 result :=-1;
 if LibHandle = 0 then
  begin
    LibHandle := LoadLibrary('elanlm.dll');
    if LibHandle = 0 then
    begin
      ShowMessage('load Elanlm error!');
      result := -1;
      exit;
    end;
    @slm_startapi := GetProcAddress(LibHandle,'slm_startapi');
    @slm_endapi := GetProcAddress(LibHandle,'slm_endapi');
    @slm_key := GetProcAddress(LibHandle,'slm_key');
//下面 XXXX 和 ZZZZ 分别是 slm_DoEnCodeOrCode 和 slm_DoGetSumInAsc4
//相对于 slm_key 的偏移。
    DWORD(@slm_DoEnCodeOrCode):= DWORD(@slm_key)+XXXX;
    DWORD(@slm_DoGetSumInAsc4) := DWORD(@slm_key)+ ZZZZ;

    if ((@slm_startapi = nil) or (@slm_endapi = nil) or (@slm_key = nil))  then
    begin
      FreeLibrary(LibHandle);
      ShowMessage('Get Elanlm Api error!');
      result := -1;
      exit;
    end;
    p_slm_instance := nil;
    result := slm_startapi(p_slm_instance);
  end;
end;

// 调用这个函数前必须先调 InitElanlmApi
function elamlmCreateKey(sFeature:string;sServerCode:string):string;
var
  pKeydata:PT_SLM_KEYDATA;
  iResult,i:integer;
  s,sSum:string;
  acMyKey:array[0..100] of char;
  dw:DWORD;
begin
  GetMem(pKeydata,sizeof(T_SLM_KEYDATA));
  fillchar(pKeydata^,sizeof(T_SLM_KEYDATA),#0);
  ...
  pKeydata^.acFeature[0] := sFeature[1];
  pKeydata^.acFeature[1] := sFeature[2];;
  pKeydata^.acFeature[2] := sFeature[3];;
  pKeydata^.acFeature[3] := sFeature[4];;

  sSum := CalcServerCodeSum(sServerCode);
  pKeydata^.acServerCodeSum[0] := sSum[1];
  pKeydata^.acServerCodeSum[1] := sSum[2]';
  pKeydata^.acServerCodeSum[2] := sSum[3];
  pKeydata^.acServerCodeSum[3] := sSum[4];
  ...
 try
    iResult := slm_key(p_slm_instance,0,pchar(@acMyKey),pKeydata,2);
    if (iResult = 0) then
     begin
      s:='';
      for i := 0 to sizeof(acMyKey) - 1 do
        begin
          if acMyKey[i] = #0 then
              break;
          s := s + acMyKey[i];
        end;
      result := result + format('%s',[s]);
      end
    else
     begin
       result := format('slm_key return error:%x,%d',[iResult,iResult]);
      end;
  finally
    FreeMem(pKeydata);
  end;
end;
```

写这么点东西的时间超过了编写上面的 Delphi 代码花的时间，希望能对需要的人有点帮助。

对这个编辑器用得不熟，感觉排版很不方便，另附上 PDF 版。

> hfade 2011 04 22

上传的附件

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || elanlmCrk.pdf  
|

```c
/*-------------------------------------------------------------*/

/*  修改号   日期         修改人     修改描述                 */

/*-------------------------------------------------------------*/

/*  初始版本 2004-09-04   HZH        初始版本                 */

/*-------------------------------------------------------------*/

#ifndef _ELANLM_H

#define _ELANLM_H

/*-------------------------------------------------------------*/

/*  include                                                     */

/*-------------------------------------------------------------*/

#include "def.h"

#include "bn.h"

#include "des.h"

/*-------------------------------------------------------------*/

/*  define                                                      */

/*-------------------------------------------------------------*/

#define ELAN_MAX_LEN 196

/*-------------------------------------------------------------*/

/*  slm_checksum32                                              */

/*-------------------------------------------------------------*/

static T_u_int32 DS_checksum[] = {
    0x0000000D, 0x00000011, 0x0000018D, 0x00000013,
    0x00000017, 0x00000191, 0x0000001D, 0x0000001F,
    0x00000199, 0x00000025, 0x00000029, 0x000001A3,
    0x0000002B, 0x0000002F, 0x000001A5, 0x00000035,
    0x0000003B, 0x000001AF, 0x0000003D, 0x00000043,
    0x000001B1, 0x00000047, 0x00000049, 0x000001B7,
    0x0000004F, 0x00000053, 0x000001BB, 0x00000059,
    0x00000061, 0x000001C1, 0x00000065, 0x00000067,
    0x000001C9, 0x0000006B, 0x0000006D, 0x000001CD,
    0x00000071, 0x0000007F, 0x000001CF, 0x00000083,
    0x00000089, 0x000001D3, 0x0000008B, 0x00000095,
    0x000001DF, 0x00000097, 0x0000009D, 0x000001E7,
    0x000000A3, 0x000000A7, 0x000001EB, 0x000000AD,
    0x000000B3, 0x000001F3, 0x000000B5, 0x000000BF,
    0x000001F7, 0x000000C1, 0x000000C5, 0x000001FD,
    0x000000C7, 0x000000D3, 0x00000209, 0x000000DF,
    0x000000E3, 0x0000020B, 0x000000E5, 0x000000E9,
    0x0000021D, 0x000000EF, 0x000000F1, 0x00000223,
    0x000000FB, 0x00000101, 0x0000022D, 0x00000107,
    0x0000010D, 0x00000233, 0x0000010F, 0x00000115,
    0x00000239, 0x00000119, 0x0000011B, 0x0000023B,
    0x00000125, 0x00000133, 0x00000241, 0x00000137,
    0x00000139, 0x0000024B, 0x0000013D, 0x0000014B,
    0x00000251, 0x00000151, 0x0000015B, 0x00000257,
    0x0000015D, 0x00000161, 0x00000259, 0x00000167,
    0x0000016F, 0x0000025F, 0x00000175, 0x0000017B,
    0x00000265, 0x0000017F, 0x00000185 };

static T_u_int32 slm_checksum32(char *in, T_u_byte base)
{
    T_u_int32 rtn = 0;
    T_u_int32 cnt = 0;
    if (in[cnt] == '\0')
    {
        return (rtn);
    }
    while (in[cnt])
    {
        rtn += (in[cnt] - base + 1) * (DS_checksum[cnt % 0x6C] + (cnt / 0x6C));
        cnt++;
    }
    return (rtn);
}

/*-------------------------------------------------------------*/

/*  slm_strconvert                                              */

/*-------------------------------------------------------------*/

static T_u_byte slm_strconvert(char *in, T_u_byte len, char * out)
{
    T_u_byte cnt1, cnt2, sig;
    cnt2 = 0;
    sig  = 0;
    for (cnt1 = 0; cnt1 < len; cnt1 += 3)
    {
        out[cnt2] = in[cnt1 + sig];
        cnt2++;
    }
    sig  = 1;
    for (cnt1 = 0; cnt1 < len; cnt1 += 3)
    {
        out[cnt2] = in[cnt1 + sig];
        cnt2++;
    }
    sig  = 2;
    for (cnt1 = 0; cnt1 < len; cnt1 += 3)
    {
        out[cnt2] = in[cnt1 + sig];
        cnt2++;
    }
    return (len);
}

/*-------------------------------------------------------------*/

/*  DigitsRequired                                              */

/*-------------------------------------------------------------*/

static T_u_byte digits[] = {
    0x00, 0x01, 0x01, 0x01, 0x02, 0x02, 0x03, 0x03,
    0x03, 0x04, 0x04, 0x05, 0x05, 0x05, 0x06, 0x06,
    0x07, 0x07, 0x08, 0x08, 0x08, 0x09, 0x09, 0x0A,
    0x0A, 0x0A, 0x0B, 0x0B, 0x0C, 0x0C, 0x0D, 0x0D,
    0x0D, 0x0E, 0x0E, 0x0F, 0x0F, 0x0F, 0x10, 0x10,
    0x11, 0x11, 0x12, 0x12, 0x12, 0x13, 0x13, 0x14,
    0x14, 0x14, 0x15, 0x15, 0x16, 0x16, 0x17, 0x17,
```
