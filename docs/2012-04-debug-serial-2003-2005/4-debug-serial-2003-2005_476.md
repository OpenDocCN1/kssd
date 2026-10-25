
# 神算刘半仙 2003 Build 09.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6248.htm`](https://www.pediy.com/kssd/pediy06/pediy6248.htm)

神算刘半仙 2003 Build 09.01 算法简析

机器码：`92495650(d)`---`5835F22(h)`

注册码：`36b3a175da3637`

试验码：`87654321`

主文件 `ssbx.exe`，无壳，`delphi` 编程。

```
:004C850E 8D45FC              lea eax, dword ptr [ebp-04]
:004C8511 8D957BFFFFFF        lea edx, dword ptr [ebp+FFFFFF7B]
:004C8517 B981000000          mov ecx, 00000081
:004C851C E89BC4F3FF          call 004049BC
:004C8521 8D9574FFFFFF        lea edx, dword ptr [ebp+FFFFFF74]
:004C8527 8B8328030000        mov eax, dword ptr [ebx+00000328]
:004C852D E84272F7FF          call 0043F774
:004C8532 8B8574FFFFFF        mov eax, dword ptr [ebp+FFFFFF74]      //eax=假码
:004C8538 50                  push eax
:004C8539 8D956CFFFFFF        lea edx, dword ptr [ebp+FFFFFF6C]
:004C853F 8B8324030000        mov eax, dword ptr [ebx+00000324]
:004C8545 E82A72F7FF          call 0043F774
:004C854A 8B856CFFFFFF        mov eax, dword ptr [ebp+FFFFFF6C]      //eax=机器码
:004C8550 E85B0CF4FF          call 004091B0
:004C8555 8D9570FFFFFF        lea edx, dword ptr [ebp+FFFFFF70]
:004C855B E804FEFFFF          call 004C8364         //算法 call
:004C8560 8B9570FFFFFF        mov edx, dword ptr [ebp+FFFFFF70]
:004C8566 58                  pop eax
:004C8567 E8E4C5F3FF          call 00404B50         //关键比较，eax 假码，edx 真码
:004C856C 0F858F000000        jne 004C8601         //跳则 over

* Possible StringData Ref from Code Obj ->"注册成功，谢谢你的注册！"
                          |
:004C8572 B874864C00          mov eax, 004C8674
:004C8577 E89405F7FF          call 00438B10
:004C857C A1A4474D00          mov eax, dword ptr [004D47A4]
:004C8581 8B00                mov eax, dword ptr [eax]
:004C8583 8B80F0020000        mov eax, dword ptr [eax+000002F0]
:004C8589 8B8008020000        mov eax, dword ptr [eax+00000208]
:004C858F BA01000000          mov edx, 00000001
:004C8594 E8138AFDFF          call 004A0FAC

* Possible StringData Ref from Code Obj ->"软件已注册"
                          |
:004C8599 BA98864C00          mov edx, 004C8698
:004C859E E86589FDFF          call 004A0F08
:004C85A3 8D8568FFFFFF        lea eax, dword ptr [ebp+FFFFFF68]

* Possible StringData Ref from Code Obj ->"FGHkey1.sys"        //注册码存放的文件
                          |
:004C85A9 B9AC864C00          mov ecx, 004C86AC
:004C85AE 8B55FC              mov edx, dword ptr [ebp-04]
:004C85B1 E8A2C4F3FF          call 00404A58
:004C85B6 8B8D68FFFFFF        mov ecx, dword ptr [ebp+FFFFFF68]
:004C85BC B201                mov dl, 01

* Possible StringData Ref from Code Obj ->"?F"
                          |
:004C85BE A1D4274600          mov eax, dword ptr [004627D4]
:004C85C3 E8BCA2F9FF          call 00462884

* Possible StringData Ref from Code Obj ->"44134co642ls3058372"
                          |
:004C85C8 68C4864C00          push 004C86C4

* Possible StringData Ref from Code Obj ->"sepop"
                          |
:004C85CD B9E0864C00          mov ecx, 004C86E0

* Possible StringData Ref from Code Obj ->"syetup"
                          |
:004C85D2 BAF0864C00          mov edx, 004C86F0
:004C85D7 8B18                mov ebx, dword ptr [eax]
:004C85D9 FF5304              call [ebx+04]
:004C85DC 8D8564FFFFFF        lea eax, dword ptr [ebp+FFFFFF64]

* Possible StringData Ref from Code Obj ->"FGHkey1.sys"
                          |
:004C85E2 B9AC864C00          mov ecx, 004C86AC
:004C85E7 8B55FC              mov edx, dword ptr [ebp-04]
:004C85EA E869C4F3FF          call 00404A58
:004C85EF 8B8564FFFFFF        mov eax, dword ptr [ebp+FFFFFF64]
:004C85F5 BA03000000          mov edx, 00000003
:004C85FA E8F90DF4FF          call 004093F8
:004C85FF EB17                jmp 004C8618

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C856C(C)
|
* Possible StringData Ref from Code Obj ->"注册码错误，请重新输入！"
                          |
:004C8601 B800874C00          mov eax, 004C8700

*********************************************************************

* Referenced by a CALL at Address:
|:004C855B
|
:004C8364 55                  push ebp
:004C8365 8BEC                mov ebp, esp
:004C8367 33C9                xor ecx, ecx
:004C8369 51                  push ecx
:004C836A 51                  push ecx
:004C836B 51                  push ecx
:004C836C 51                  push ecx
:004C836D 51                  push ecx
:004C836E 51                  push ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C82FE(C)
|
:004C836F 53                  push ebx
:004C8370 56                  push esi
:004C8371 8BF2                mov esi, edx
:004C8373 8BD8                mov ebx, eax      //ebx=eax=机器码的 16 进制
:004C8375 33C0                xor eax, eax
:004C8377 55                  push ebp
:004C8378 68B0844C00          push 004C84B0
:004C837D 64FF30              push dword ptr fs:[eax]
:004C8380 648920              mov dword ptr fs:[eax], esp
:004C8383 81F38776FBDD        xor ebx, DDFB7687      //ebx=机器码 xor DDFB7687 =D87829A5
:004C8389 8BC3                mov eax, ebx        //eax=ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C831C(C)
|
:004C838B 33D2                xor edx, edx
:004C838D 52                  push edx
:004C838E 50                  push eax
:004C838F 8D45FC              lea eax, dword ptr [ebp-04]
:004C8392 E8E50DF4FF          call 0040917C
:004C8397 8B45FC              mov eax, dword ptr [ebp-04]        //eax=ebx 的 10 进制值 3631753637
:004C839A 0FB600              movzx eax, byte ptr [eax]        //取 3631753637 的首位字符 hex 值 33 送 eax
:004C839D 8B55FC              mov edx, dword ptr [ebp-04]
:004C83A0 0FB65201            movzx edx, byte ptr [edx+01]        //取 3631753637 的第二位字符 hex 值 36 送 edx
:004C83A4 03C2                add eax, edx        //eax=eax+edx=69
:004C83A6 B905000000          mov ecx, 00000005      //ecx=5
:004C83AB 99                  cdq
:004C83AC F7F9                idiv ecx        //eax / ecx
:004C83AE 80C261              add dl, 61      //dl=余数 dl 加上 61=61
:004C83B1 8855F8              mov byte ptr [ebp-08], dl        //字符"a"(0x61)送[ebp-8]
:004C83B4 8B45FC              mov eax, dword ptr [ebp-04]
:004C83B7 0FB64002            movzx eax, byte ptr [eax+02]        //取 3631753637 的第三位字符 hex 值 33 送 eax
:004C83BB 8B55FC              mov edx, dword ptr [ebp-04]
:004C83BE 0FB65203            movzx edx, byte ptr [edx+03]        //取 3631753637 的第四位字符 hex 值 31 送 edx
:004C83C2 03C2                add eax, edx        //eax=eax+edx=64
:004C83C4 B905000000          mov ecx, 00000005
:004C83C9 99                  cdq
:004C83CA F7F9                idiv ecx
:004C83CC 80C261              add dl, 61      //dl=余数 dl 加上 61=61
:004C83CF 8855F9              mov byte ptr [ebp-07], dl        //字符"a"(0x61)送[ebp-7]
:004C83D2 8B45FC              mov eax, dword ptr [ebp-04]
```



```
:004C83D5 0FB64004                movzx eax, byte ptr [eax+04]       //取 3631753637 的第五位字符 hex 值 37 送 eax    
:004C83D9 8B55FC                  mov edx, dword ptr [ebp-04]   
:004C83DC 0FB65205                movzx edx, byte ptr [edx+05]       //取 3631753637 的第六位字符 hex 值 35 送 edx    
:004C83E0 03C2                    add eax, edx       //eax=eax+edx=6C    
:004C83E2 B905000000              mov ecx, 00000005   
:004C83E7 99                      cdq   
:004C83E8 F7F9                    idiv ecx   
:004C83EA 80C261                  add dl, 61      //dl=余数 dl 加上 61=3+1=64    
:004C83ED 8855FA                  mov byte ptr [ebp-06], dl       //字符"d"(0x64)送[ebp-6]    
:004C83F0 8B45FC                  mov eax, dword ptr [ebp-04]   
:004C83F3 0FB64006                movzx eax, byte ptr [eax+06]       //取 3631753637 的第七位字符 hex 值 33 送 eax    
:004C83F7 8B55FC                  mov edx, dword ptr [ebp-04]   
:004C83FA 0FB65207                movzx edx, byte ptr [edx+07]       //取 3631753637 的第八位字符 hex 值 36 送 edx    
:004C83FE 03C2                    add eax, edx       //eax=eax+edx=69    
:004C8400 8B55FC                  mov edx, dword ptr [ebp-04]   
:004C8403 0FB65208                movzx edx, byte ptr [edx+08]       //取 3631753637 的第九位字符 hex 值 33 送 edx    
:004C8407 03C2                    add eax, edx       //eax=eax+edx=69+33=9C    
:004C8409 B905000000              mov ecx, 00000005   
:004C840E 99                      cdq   
:004C840F F7F9                    idiv ecx   
:004C8411 80C261                  add dl, 61      //dl=余数 dl 加上 61=1+61=62    
:004C8414 8855FB                  mov byte ptr [ebp-05], dl       //字符"b"(0x62)送[ebp-6]    
:004C8417 8D45F4                  lea eax, dword ptr [ebp-0C]   
:004C841A 8A55F8                  mov dl, byte ptr [ebp-08]      //dl=字符"a"    
:004C841D E812C5F3FF              call 00404934   
:004C8422 8B45F4                  mov eax, dword ptr [ebp-0C]   
:004C8425 8D55FC                  lea edx, dword ptr [ebp-04]   
:004C8428 B907000000              mov ecx, 00000007          //ecx=7    
:004C842D E8BAC8F3FF              call 00404CEC      //这个 call 把字符"a"插入到 3631753637 的第 7 位    
:004C8432 8D45F0                  lea eax, dword ptr [ebp-10]      //edx=a3637    
:004C8435 8A55FB                  mov dl, byte ptr [ebp-05]      //dl=字符"b"    
:004C8438 E8F7C4F3FF              call 00404934   
:004C843D 8B45F0                  mov eax, dword ptr [ebp-10]   
:004C8440 8D55FC                  lea edx, dword ptr [ebp-04]   
:004C8443 B903000000              mov ecx, 00000003         //ecx=3    
:004C8448 E89FC8F3FF              call 00404CEC      //这个 call 把字符"b"插入到 363175a3637 的第 3 位    
:004C844D 8D45EC                  lea eax, dword ptr [ebp-14]     //edx=b3175a3637    
:004C8450 8A55F9                  mov dl, byte ptr [ebp-07]      //dl=字符"a"    
:004C8453 E8DCC4F3FF              call 00404934   
:004C8458 8B45EC                  mov eax, dword ptr [ebp-14]   
:004C845B 8D55FC                  lea edx, dword ptr [ebp-04]   
:004C845E B905000000              mov ecx, 00000005         //ecx=5    
:004C8463 E884C8F3FF              call 00404CEC      //这个 call 把字符"a"插入到 36b3175a3637 的第 5 位    
:004C8468 8D45E8                  lea eax, dword ptr [ebp-18]      //edx=a175a3637    
:004C846B 8A55FA                  mov dl, byte ptr [ebp-06]      //dl=字符"d"    
:004C846E E8C1C4F3FF              call 00404934   
:004C8473 8B45E8                  mov eax, dword ptr [ebp-18]   
:004C8476 8D55FC                  lea edx, dword ptr [ebp-04]   
:004C8479 B909000000              mov ecx, 00000009        //ecx=9    
:004C847E E869C8F3FF              call 00404CEC      //这个 call 把字符"d"插入到 36b3a175a3637 的第 9 位    
:004C8483 8BC6                    mov eax, esi       //edx=da3637    
:004C8485 8B55FC                  mov edx, dword ptr [ebp-04]       //edx=36b3a175da3637，真码啦。    
:004C8488 E81BC3F3FF              call 004047A8   
:004C848D 33C0                    xor eax, eax   
:004C848F 5A                      pop edx   
:004C8490 59                      pop ecx   
:004C8491 59                      pop ecx   
:004C8492 648910                  mov dword ptr fs:[eax], edx   
:004C8495 68B7844C00              push 004C84B7   

* Referenced by a (U)nconditional or (C)onditional Jump at Address: 
:004C84B5(U)   
:004C849A 8D45E8                  lea eax, dword ptr [ebp-18]   
:004C849D BA04000000              mov edx, 00000004   
:004C84A2 E8D1C2F3FF              call 00404778   
:004C84A7 8D45FC                  lea eax, dword ptr [ebp-04]   
:004C84AA E8A5C2F3FF              call 00404754   
:004C84AF C3                      ret   
```

注册信息加密保存在 `C:WINDOWSSYSTEM` 的 `FGHkey1.sys` 中，属性隐藏。

李逍遥
2003.09.09

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Controls, Forms,
  Dialogs, StdCtrls, jpeg, ExtCtrls;

type
  TForm1 = class(TForm)
    Label1: TLabel;
    Label2: TLabel;
    Edit1: TEdit;
    Edit2: TEdit;
    Button1: TButton;
    Button2: TButton;
    Label3: TLabel;
    Image1: TImage;
    procedure Button2Click(Sender: TObject);
    procedure Button1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

procedure TForm1.Button2Click(Sender: TObject);
begin
  Close;
end;

procedure TForm1.Button1Click(Sender: TObject);
var
  H: string[15]; //保存机器码
  S: string[4]; //保存计算过程中间值
  T: Cardinal;   //临时存放机器码
  i: integer;    //计数器
  n: integer;    //计数器
  HH: Cardinal;  //存放注册码
  {H2: Cardinal; //临时
  H1: Cardinal; //临时}
label Exit, Error;
begin
  if edit1.GetTextLen = 0 then
  begin
    showmessage('请确认您的机器码！');
    goto Exit;
  end
  else begin
    H := edit1.Text;
    for i := 1 to length(H) do
      if (H[i] < '0') or (H[i] > '9') then
      begin
        H := '';
        goto Error;
      end;
  end;

  //以下将输入的机器码进行计算
  T := strtoint(H);
  HH := T xor 3724244615;
  H := inttostr(HH);

  //以下对结果再运算
  n := (integer(H[1]) + integer(H[2])) mod 5;
  S[1] := char(n + 97);

  n := (integer(H[3]) + integer(H[4])) mod 5;
  S[2] := char(n + 97);

  n := (integer(H[5]) + integer(H[6])) mod 5;
  S[3] := char(n + 97);

  n := (integer(H[7]) + integer(H[8]) + integer(H[9])) mod 5;
  S[4] := char(n + 97);

  //下面调整生成的注册码，插入 7,3,5,9 位(9，3，5，10 四位)
  for i := 11 downto 4 do
    H[i] := H[i-1];
  H[3] := S[4];

  for i := 12 downto 6 do
    H[i] := H[i-1];
  H[5] := S[2];

  for i := 13 downto 10 do
    H[i] := H[i-1];
  H[9] := S[3];

  for i := 14 downto 11 do
    H[i] := H[i-1];
  H[10] := S[1];

  edit2.Text := H + H[11] + H[12] + H[13] + H[14];

  goto Exit;

Error:
  showmessage('请确认您输入的机器码！');

Exit:
end;

end.
```

以上在 `WIn2000P3` + `Delphi7` 下通过

多说一句，这个注册机可以注册老夫子的一系列软件……

`4dd466d0e1e0cc123afbe504da2462ca`
