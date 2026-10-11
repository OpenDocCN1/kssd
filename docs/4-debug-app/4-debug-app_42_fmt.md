
# 简单字符串加解密函数提取

> 原文：[`www.pediy.com/kssd/pediy10/63129.html`](https://www.pediy.com/kssd/pediy10/63129.html)

`kongfoo/2008.4.14`

网上下载的中图法分类号查询普及版，`delphi` 程序，数据库用 `mdb`，数据加密了。目的是要使用该数据库。窗体有个 `TreeView`，显示类目，点击就显示详细资料。`DeDe` 看 `TreeView1Change` 找到 `SQL` 语句，生成加密后的关键字在数据库中搜索，计算函数在 `4bb3c8`：

## 计算函数

`004D071A` `E8 A9ACFEFF` `CALL ztf_ztc.004BB3C8`

## 函数代码

```assembly
***** TRY
|
004BB3EB   64FF30           push    dword ptr fs:[eax]
004BB3EE   648920           mov     fs:[eax], esp

* Reference to: System.Randomize;
|
004BB3F1   E89677F4FF       call    00402B8C
004BB3F6   C745F802000000  mov     dword ptr [ebp-$08], $00000002
004BB3FD   8B45FC           mov     eax, [ebp-$04]

* Reference to: System.@LStrLen(String):Integer;
|
004BB400   E8C795F4FF       call    004049CC
004BB405   8BD0             mov     edx, eax
004BB407   03D2             add     edx, edx
004BB409   42               inc     edx
004BB40A   8BC7             mov     eax, edi

* Reference to: System.@LStrSetLength;
|
004BB40C   E83F99F4FF       call    00404D50
004BB411   8B45FC           mov     eax, [ebp-$04]

* Reference to: System.@LStrLen(String):Integer;
|
004BB414   E8B395F4FF       call    004049CC
004BB419   48               dec     eax
004BB41A   85C0             test    eax, eax
004BB41C   7C63             jl      004BB481
004BB41E   40               inc     eax
004BB41F   8945F4           mov     [ebp-$0C], eax
004BB422   33DB             xor     ebx, ebx
004BB424   8B45FC           mov     eax, [ebp-$04]
004BB427   0FB63418         movzx   esi, byte ptr [eax+ebx]  ==取出字符
004BB42B   8BC7             mov     eax, edi

* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB42D   E8EA97F4FF       call    00404C1C
004BB432   8BD3             mov     edx, ebx
004BB434   03D2             add     edx, edx
004BB436   03C2             add     eax, edx
004BB438   50               push    eax
004BB439   8BC6             mov     eax, esi        ==esi:字符
004BB43B   83E00F          and     eax, +$0F       ==保留低位
004BB43E   8B55F8           mov     edx, [ebp-$08]  ==strLength
004BB441   03D2             add     edx, edx
004BB443   8D14D524464D00  lea     edx, [$4D4624+edx*8]  ==查表

004D4624  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4634  30 31 32 33 34 35 36 37 38 39 61 62 63 64 65 66  0123456789abcdef
004D4644  61 7A 68 6A 6C 3B 2A 38 30 2E 23 28 59 42 4E 4B  azhjl;*.80.#(YBNK
004D4654  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4664  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4674  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4684  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4694  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D46A4  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D46B4  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq


004BB44A   8A0402           mov     al, byte ptr [edx+eax]  ==从表中取出值
004BB44D   5A               pop     edx
004BB44E   8802             mov     [edx], al ==保存该值
004BB450   8BC7             mov     eax, edi

* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB452   E8C597F4FF       call    00404C1C
004BB457   8BD3             mov     edx, ebx
004BB459   03D2             add     edx, edx
004BB45B   8D441001         lea     eax, [eax+edx+$01]
004BB45F   50               push    eax
004BB460   81E6F0000000     and     esi, $000000F0  ==字符值，保留高位
004BB466   C1EE04           shr     esi, $04        ==放到低位，重复做一下上面的查表操作
004BB469   8B45F8           mov     eax, [ebp-$08]
004BB46C   03C0             add     eax, eax
004BB46E   8D04C524464D00  lea     eax, [$4D4624+eax*8]
004BB475   8A0430           mov     al, byte ptr [eax+esi]
004BB478   5A               pop     edx
004BB479   8802             mov     [edx], al
004BB47B   43               inc     ebx
004BB47C   FF4DF4           dec     dword ptr [ebp-$0C]
004BB47F   75A3             jnz     004BB424
004BB481   8D55F0           lea     edx, [ebp-$10]
004BB484   8B45F8           mov     eax, [ebp-$08]

* Reference to: SysUtils.IntToStr(Integer):AnsiString;overload;
|
004BB487   E8F8DBF4FF       call    00409084
004BB48C   8B45F0           mov     eax, [ebp-$10]
004BB48F   8A18             mov     bl, byte ptr [eax]
004BB491   8B45FC           mov     eax, [ebp-$04]

* Reference to: System.@LStrLen(String):Integer;
|           or: System.@DynArrayLength;
|           or: System.DynArraySize(Pointer):Integer;
|           or: Variants.DynArraySize(Pointer):Integer;
|
004BB494   E83395F4FF       call    004049CC
004BB499   8BF0             mov     esi, eax
004BB49B   03F6             add     esi, esi
004BB49D   8BC7             mov     eax, edi

* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB49F   E87897F4FF       call    00404C1C
004BB4A4   881C30           mov     [eax+esi], bl
004BB4A7   33C0             xor     eax, eax
004BB4A9   5A               pop     edx
004BB4AA   59               pop     ecx
004BB4AB   59               pop     ecx
004BB4AC   648910           mov     fs:[eax], edx

***** FINALLY
```

### 逆向后的 Delphi 代码

```pascal
procedure TForm1.Button1Click(Sender: TObject);
var s,OutPutStr:String;
    strLen,i,Index:Integer;
    aChar:Char;
begin
  s:=Edit1.Text;
  strLen:=Length(s);
  OutPutStr:='';
  for i:=1 to strLen do
  begin
    aChar:=s[i];
    Index:=Ord(aChar) and $F;
    OutPutStr:=OutPutStr+keyArray[strLen*2*8+Index];
    Index:=Ord(aChar) and $F0 shr 4;
    OutPutStr:=OutPutStr+keyArray[strLen*2*8+Index];
  end;
  Edit2.Text:=OutPutStr;
end;
```

上面的代码是点击 `TreeView` 事件中用到的计算代码，用于根据分类号进行查询，函数内用到的表长度为 `160` 字节，查表过程中字符串长度会乘以 `16`，即如果字串长度大于 `10` 将导致查表越界。所以这里是假设分类号不超 `10` 个字符。但数据库内其他字段的数据很多是很长的，哪么应该还有其它函数进行解密并在软件界面中显示的操作。

## 解密函数

在 `FormCreate` 中看看：

`004D0B0A` `E8C5A9FEFF` `call` `004BB4D4`

跟踪到解密字串过程：



```
004BB51A    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  ==字串
004BB51D    E8 AA94F4FF     CALL ztf_ztc.004049CC         ==Length
004BB522    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]  ==字串
004BB525    8A5402 FF       MOV DL,BYTE PTR DS:[EDX+EAX-1]  ==字串尾字符
004BB529    8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
004BB52C    E8 C393F4FF     CALL ztf_ztc.004048F4         ==StrFromChar
004BB531    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
004BB534    E8 AFDBF4FF     CALL ztf_ztc.004090E8         ==StrToInt，最后一位是数字，通常是 2
004BB539    8BF8            MOV EDI,EAX
004BB53B    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  ==字串
004BB53E    E8 8994F4FF     CALL ztf_ztc.004049CC         ==Length
004BB543    8BD0            MOV EDX,EAX
004BB545    D1FA            SAR EDX,1
004BB547    79 03           JNS SHORT ztf_ztc.004BB54C

004BB54C    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004BB54F    E8 FC97F4FF     CALL ztf_ztc.00404D50         ==SetLength
004BB554    BE 01000000     MOV ESI,1
004BB559    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  ==字串
004BB55C    E8 6B94F4FF     CALL ztf_ztc.004049CC         ==Length
004BB561    3D FA000000     CMP EAX,0FA
004BB566    7E 50           JLE SHORT ztf_ztc.004BB5B8

004BB572    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  ==字串
004BB575    8A5430 FF       MOV DL,BYTE PTR DS:[EAX+ESI-1] ==从头开始取字符
004BB579    8BC7            MOV EAX,EDI                    ==edi 就是字串末位的数字，2
004BB57B    E8 24FEFFFF     CALL ztf_ztc.004BB3A4

004BB3A4    53              PUSH EBX
004BB3A5    56              PUSH ESI
004BB3A6    BE 0F000000     MOV ESI,0F
004BB3AB    B9 33464D00     MOV ECX,ztf_ztc.004D4633                   ; ASCII 71,"0123456789abcdefazhjl;\\*80.#(YBNKabcdefhijklmnopqabcdefhijklmnopqabcdefhijklmnopqabcdefhijklmnopqabcdefhijklmnopqa"
004BB3B0    8BD8            MOV EBX,EAX  ==4d4633 就是上面的表开头 4d4624+0F，这里 eax 就是字串末位数值
004BB3B2    03DB            ADD EBX,EBX
004BB3B4    3A14D9          CMP DL,BYTE PTR DS:[ECX+EBX*8]  ==DL:要解密的字符
004BB3B7    74 07           JE SHORT ztf_ztc.004BB3C0  ==在表中向表头方向查找，找到即返回下标值
004BB3B9    4E              DEC ESI
004BB3BA    49              DEC ECX
004BB3BB    83FE FF         CMP ESI,-1
004BB3BE   75 F0           JNZ SHORT ztf_ztc.004BB3B0
004BB3C0    8BC6            MOV EAX,ESI
004BB3C2    5E              POP ESI
004BB3C3    5B              POP EBX
004BB3C4    C3              RETN

004BB580    33DB            XOR EBX,EBX
004BB582    83F8 FF         CMP EAX,-1
004BB585    74 02           JE SHORT ztf_ztc.004BB589
004BB587    8BD8            MOV EBX,EAX
004BB589    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  ==字串
004BB58C    8A1430          MOV DL,BYTE PTR DS:[EAX+ESI]  ==第 2 个字符
004BB58F    8BC7            MOV EAX,EDI
004BB591    E8 0EFEFFFF     CALL ztf_ztc.004BB3A4  ==解密
004BB596    83F8 FF         CMP EAX,-1
004BB599    74 05           JE SHORT ztf_ztc.004BB5A0
004BB59B    C1E0 04         SHL EAX,4
004BB59E    02D8            ADD BL,AL
004BB5A0    83C6 02         ADD ESI,2        ==两值组合
004BB5A3    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004BB5A6    E8 7196F4FF     CALL ztf_ztc.00404C1C
004BB5AB    8BD6            MOV EDX,ESI
004BB5AD    D1FA            SAR EDX,1
004BB5AF    79 03           JNS SHORT ztf_ztc.004BB5B4
004BB5B1    83D2 00         ADC EDX,0
004BB5B4    885C10 FF       MOV BYTE PTR DS:[EAX+EDX-1],BL  ==结果写入字串
004BB5B8    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004BB5BB    E8 0C94F4FF     CALL ztf_ztc.004049CC
004BB5C0    3BF0            CMP ESI,EAX
004BB5C2   7C AE           JL SHORT ztf_ztc.004BB572
```

将上面解密字串函数转成 Delphi:

代码:

```delphi
function DecryptChar(aChar:Char;magicValue:Integer):Integer;
var i,Index,offset:Integer;
begin
  Index:=$F;
  offset:=0;
  for i:=0 to $F do
  begin
    if keyArray[magicValue*2*8+$F+offset]=aChar then
    begin
      result:=Index;
      break;
    end;
    Dec(Index);
    Dec(offset);
  end;
end;

procedure TForm1.Button2Click(Sender: TObject);
var s,OutPutStr:String;
    magicValue,strLen,i,result1,result2:Integer;
begin
  s:=Edit4.Text;
  strLen:=Length(s);
  if strLen>$FA then Exit;
  magicValue:=StrToInt(Copy(s,strLen,1));
  i:=1;
  OutPutStr:='';
  while i<strLen-1 do
  begin
    result1:=DecryptChar(s[i],magicValue);
    result2:=DecryptChar(s[i+1],magicValue);
    OutPutStr:=OutPutStr+Chr(result2 shl 4 + result1);
    Inc(i,2);
  end;
  Edit3.Text:=OutPutStr;
end;
```

解密代码成功将

`;(#BhB((l(NY#YaYB(8Nl(jK;BB(8Ya(#(jKj#0#zj0j*j8jBhzj.jzj8jlY#Nj#.#2`

解密成

第一次世界大战前后（1867-1917 年）

有了解密代码的研究，再回头看看之前生成密文的代码，修改一下就变成加密代码了：

代码:

```delphi
procedure TForm1.Button1Click(Sender: TObject);
var s,OutPutStr:String;
    strLen,i,Index,magicValue:Integer;
    aChar:Char;
begin
  s:=Edit1.Text;
  strLen:=Length(s);
  magicValue:=2;
  OutPutStr:='';
  for i:=1 to strLen do
  begin
    aChar:=s[i];
    Index:=Ord(aChar) and $F;
    OutPutStr:=OutPutStr+keyArray[magicValue*2*8+Index];
    Index:=Ord(aChar) and $F0 shr 4;
    OutPutStr:=OutPutStr+keyArray[magicValue*2*8+Index];
  end;
  OutPutStr:=OutPutStr+IntToStr(magicValue);
  Edit2.Text:=OutPutStr;
end;
```

根据解密函数对`keyArray`的使用方法，代码中修改了`keyArray`下标`strLen*2*8+Index`->`magicValue*2*8+Index`。

这样就得到了一套字串加解密函数了。为了保留对汇编代码的直观性，代码没有进行优化。接下来是解密字串写回数据库，这部分略。
