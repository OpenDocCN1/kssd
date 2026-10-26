
# [原创]简单字符串加解密函数提取

简单字符串加解密函数提取

kongfoo/2008.4.14

网上下载的中图法分类号查询普及版，delphi程序，数据库用`mdb`，数据加密了。目的是要使用该数据库。窗体有个`TreeView`，显示类目，点击就显示详细资料。DeDe看`TreeView1Change`找到SQL语句，生成加密后的关键字在数据库中搜索，计算函数在`4bb3c8`：

## 计算函数

```
004D071A    E8 A9ACFEFF     CALL ztf_ztc.004BB3C8
```

### 函数代码

```
***** TRY
|
004BB3EB   64FF30                 push    dword ptr fs:[eax]
004BB3EE   648920                 mov     fs:[eax], esp
* Reference to: System.Randomize;
|
004BB3F1   E89677F4FF             call    00402B8C
004BB3F6   C745F802000000         mov     dword ptr [ebp-$08], $00000002
004BB3FD   8B45FC                 mov     eax, [ebp-$04]
* Reference to: System.@LStrLen(String):Integer;
|
004BB400   E8C795F4FF             call    004049CC
004BB405   8BD0                   mov     edx, eax
004BB407   03D2                   add     edx, edx
004BB409   42                     inc     edx
004BB40A   8BC7                   mov     eax, edi
* Reference to: System.@LStrSetLength;
|
004BB40C   E83F99F4FF             call    00404D50
004BB411   8B45FC                 mov     eax, [ebp-$04]
* Reference to: System.@LStrLen(String):Integer;
|
004BB414   E8B395F4FF             call    004049CC
004BB419   48                     dec     eax
004BB41A   85C0                   test    eax, eax
004BB41C   7C63                   jl      004BB481
004BB41E   40                     inc     eax
004BB41F   8945F4                 mov     [ebp-$0C], eax
004BB422   33DB                   xor     ebx, ebx
004BB424   8B45FC                 mov     eax, [ebp-$04]
004BB427   0FB63418               movzx   esi, byte ptr [eax+ebx]  ==取出字符
004BB42B   8BC7                   mov     eax, edi
* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB42D   E8EA97F4FF             call    00404C1C
004BB432   8BD3                   mov     edx, ebx
004BB434   03D2                   add     edx, edx
004BB436   03C2                   add     eax, edx
004BB438   50                     push    eax
004BB439   8BC6                   mov     eax, esi        ==esi:字符
004BB43B   83E00F                 and     eax, +$0F       ==保留低位
004BB43E   8B55F8                 mov     edx, [ebp-$08]  ==strLength
004BB441   03D2                   add     edx, edx
004BB443   8D14D524464D00         lea     edx, [$4D4624+edx*8]  ==查表

004D4624  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4634  30 31 32 33 34 35 36 37 38 39 61 62 63 64 65 66  0123456789abcdef
004D4644  61 7A 68 6A 6C 3B 2A 38 30 2E 23 28 59 42 4E 4B  azhjl;*80.#(YBNK
004D4654  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4664  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4674  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4684  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D4694  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D46A4  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq
004D46B4  61 62 63 64 65 66 68 69 6A 6B 6C 6D 6E 6F 70 71  abcdefhijklmnopq

004BB44A   8A0402                 mov     al, byte ptr [edx+eax]  ==从表中取出值
004BB44D   5A                     pop     edx
004BB44E   8802                   mov     [edx], al ==保存该值
004BB450   8BC7                   mov     eax, edi
* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB452   E8C597F4FF             call    00404C1C
004BB457   8BD3                   mov     edx, ebx
004BB459   03D2                   add     edx, edx
004BB45B   8D441001               lea     eax, [eax+edx+$01]
004BB45F   50                     push    eax
004BB460   81E6F0000000           and     esi, $000000F0  ==字符值，保留高位
004BB466   C1EE04                 shr     esi, $04        ==放到低位，重复做一下上面的查表操作
004BB469   8B45F8                 mov     eax, [ebp-$08]
004BB46C   03C0                   add     eax, eax
004BB46E   8D04C524464D00         lea     eax, [$4D4624+eax*8]
004BB475   8A0430                 mov     al, byte ptr [eax+esi]
004BB478   5A                     pop     edx
004BB479   8802                   mov     [edx], al
004BB47B   43                     inc     ebx
004BB47C   FF4DF4                 dec     dword ptr [ebp-$0C]
004BB47F   75A3                   jnz     004BB424
004BB481   8D55F0                 lea     edx, [ebp-$10]
004BB484   8B45F8                 mov     eax, [ebp-$08]
* Reference to: SysUtils.IntToStr(Integer):AnsiString;overload;
|
004BB487   E8F8DBF4FF             call    00409084
004BB48C   8B45F0                 mov     eax, [ebp-$10]
004BB48F   8A18                   mov     bl, byte ptr [eax]
004BB491   8B45FC                 mov     eax, [ebp-$04]
* Reference to: System.@LStrLen(String):Integer;
|           or: System.@DynArrayLength;
|           or: System.DynArraySize(Pointer):Integer;
|           or: Variants.DynArraySize(Pointer):Integer;
|
004BB494   E83395F4FF             call    004049CC
004BB499   8BF0                   mov     esi, eax
004BB49B   03F6                   add     esi, esi
004BB49D   8BC7                   mov     eax, edi
* Reference to: crtl.__pure_error_;
|           or: crtl.__matherrl;
|           or: crtl._gcvt;
|           or: System.FPower10;
|           or: System.UniqueString(String;String);overload;
|           or: System.@UniqueStringA(String;String);
|
004BB49F   E87897F4FF             call    00404C1C
004BB4A4   881C30                 mov     [eax+esi], bl
004BB4A7   33C0                   xor     eax, eax
004BB4A9   5A                     pop     edx
004BB4AA   59                     pop     ecx
004BB4AB   59                     pop     ecx
004BB4AC   648910                 mov     fs:[eax], edx

****** FINALLY
```

### 先将上面的代码逆向成delphi代码

```
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

上面的代码是点击`TreeView`事件中用到的计算代码，用于根据分类号进行查询，函数内用到的表长度为160字节，查表过程中字符串长度会乘以16，即如果字串长度大于10将导致查表越界。所以这里是假设分类号不超10个字符。但数据库内其他字段的数据很多是很长的，哪么应该还有其它函数进行解密并在软件界面中显示的操作。

`FormCreate`中看看：

## 解密函数

```
004D0B0A   E8C5A9FEFF             call    004BB4D4
```

### 跟踪到解密字串过程

