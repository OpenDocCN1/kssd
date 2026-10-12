
# 飞马魔法壁纸 V3.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue262.htm`](https://www.pediy.com/kssd/pediy04/kanxue262.htm)

飞马魔法壁纸 V3.0 破解

原版下载地址：`http://www.shareware.net.cn/download.asp?id={1382670D-7C41-4D4C-B470-468F46FF75AD}`

注册机下载：CNCG 主页

破解工具：`trw2000`

先下断点：`bpx hmemcpy`，然后返回到主程序，如下：

```
:004A8564 E82FB9F9FF             call 00443E98
:004A8569 837DF400               cmp dword ptr [ebp-0C], 00000000 <====返回到这里
:004A856D 0F84C1010000           je 004A8734
:004A8573 8D55F0                 lea edx, dword ptr [ebp-10]
:004A8576 8B833C070000           mov eax, dword ptr [ebx+0000073C]
:004A857C E817B9F9FF             call 00443E98
:004A8581 837DF000               cmp dword ptr [ebp-10], 00000000
:004A8585 0F84A9010000           je 004A8734
:004A858B 8D55EC                 lea edx, dword ptr [ebp-14]
:004A858E 8B8340070000           mov eax, dword ptr [ebx+00000740]
:004A8594 E8FFB8F9FF             call 00443E98
:004A8599 837DEC00               cmp dword ptr [ebp-14], 00000000
:004A859D 0F8491010000           je 004A8734
:004A85A3 8D55E8                 lea edx, dword ptr [ebp-18]
:004A85A6 8B833C070000           mov eax, dword ptr [ebx+0000073C]
:004A85AC E8E7B8F9FF             call 00443E98
:004A85B1 8B45E8                 mov eax, dword ptr [ebp-18]
:004A85B4 E89BC5F5FF             call 00404B54
:004A85B9 E8DA0DF6FF             call 00409398
:004A85BE 8BF8                   mov edi, eax
:004A85C0 D1EF                   shr edi, 1
:004A85C2 85FF                   test edi, edi
:004A85C4 7905                   jns 004A85CB
:004A85C6 E8A5B2F5FF             call 00403870
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A85C4(C)
|
:004A85CB 8BC7                   mov eax, edi
:004A85CD 83C002                 add eax, 00000002
:004A85D0 7105                   jno 004A85D7
:004A85D2 E8A1B2F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A85D0(C)
|
:004A85D7 E858A1F5FF             call 00402734
:004A85DC 8BF0                   mov esi, eax
:004A85DE C60600                 mov byte ptr [esi], 00
:004A85E1 8D55E4                 lea edx, dword ptr [ebp-1C]
:004A85E4 8B833C070000           mov eax, dword ptr [ebx+0000073C]
:004A85EA E8A9B8F9FF             call 00443E98
:004A85EF 8B45E4                 mov eax, dword ptr [ebp-1C]
:004A85F2 E85DC5F5FF             call 00404B54
:004A85F7 8BD0                   mov edx, eax
:004A85F9 03D7                   add edx, edi
:004A85FB 8BC6                   mov eax, esi
:004A85FD E89E0EF6FF             call 004094A0
:004A8602 8D55E0                 lea edx, dword ptr [ebp-20]
:004A8605 8B833C070000           mov eax, dword ptr [ebx+0000073C]
:004A860B E888B8F9FF             call 00443E98
:004A8610 8B45E0                 mov eax, dword ptr [ebp-20] <====取申请码
:004A8613 E8FC06F6FF             call 00408D14 <====将申请码转换为相应的十六进制(假设结果为 R1)
:004A8618 8BF8                   mov edi, eax
:004A861A 8D45DC                 lea eax, dword ptr [ebp-24]
:004A861D 8BD6                   mov edx, esi
:004A861F E870C2F5FF             call 00404894
:004A8624 8B45DC                 mov eax, dword ptr [ebp-24]
:004A8627 E8E806F6FF             call 00408D14 <====将申请码的后 5 位转换为相应的十六进制(假设结果为 R2)
:004A862C 03F8                   add edi, eax <====R3=R1+R2
:004A862E 7105                   jno 004A8635 <====无溢出则跳
:004A8630 E843B2F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A862E(C)
|
:004A8635 81C7753D1E00           add edi, 001E3D75 <====R4=R3+1E3D75H
:004A863B 7105                   jno 004A8642
:004A863D E836B2F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A863B(C)
|
:004A8642 8D55D8                 lea edx, dword ptr [ebp-28]
:004A8645 8B8344070000           mov eax, dword ptr [ebx+00000744]
:004A864B E848B8F9FF             call 00443E98
:004A8650 8B45D8                 mov eax, dword ptr [ebp-28]
:004A8653 E804C3F5FF             call 0040495C <====获取用户名长度(L)
:004A8658 03F8                   add edi, eax <====R5=R4+L
:004A865A 7105                   jno 004A8661
:004A865C E817B2F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A865A(C)
|
:004A8661 897DFC                 mov dword ptr [ebp-04], edi
:004A8664 8D55F8                 lea edx, dword ptr [ebp-08]
:004A8667 8B45FC                 mov eax, dword ptr [ebp-04]
:004A866A E80506F6FF             call 00408C74
:004A866F 8B45F8                 mov eax, dword ptr [ebp-08]
:004A8672 E8DDC4F5FF             call 00404B54
:004A8677 E81C0DF6FF             call 00409398
:004A867C 8BF8                   mov edi, eax
:004A867E D1EF                   shr edi, 1
:004A8680 85FF                   test edi, edi
:004A8682 7905                   jns 004A8689
:004A8684 E8E7B1F5FF             call 00403870
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A8682(C)
|
:004A8689 8BC7                   mov eax, edi
:004A868B 83C002                 add eax, 00000002
:004A868E 7105                   jno 004A8695
:004A8690 E8E3B1F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A868E(C)
|
:004A8695 E89AA0F5FF             call 00402734
:004A869A 8BF0                   mov esi, eax
:004A869C C60600                 mov byte ptr [esi], 00
:004A869F 8B45F8                 mov eax, dword ptr [ebp-08]
:004A86A2 E8ADC4F5FF             call 00404B54
:004A86A7 8BD0                   mov edx, eax
:004A86A9 03D7                   add edx, edi
:004A86AB 8BC6                   mov eax, esi
:004A86AD E8EE0DF6FF             call 004094A0
:004A86B2 8D45D4                 lea eax, dword ptr [ebp-2C]
:004A86B5 8BD6                   mov edx, esi
:004A86B7 E8D8C1F5FF             call 00404894
:004A86BC 8B45D4                 mov eax, dword ptr [ebp-2C]
:004A86BF E85006F6FF             call 00408D14 <====将 R5 相应的十进制后 5 位转换成十六进制(R6)
:004A86C4 8BF8                   mov edi, eax
:004A86C6 037DFC                 add edi, dword ptr [ebp-04] <====R7=R6+R5 结果就是注册码（十六进制形式）
:004A86C9 7105                   jno 004A86D0
:004A86CB E8A8B1F5FF             call 00403878
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A86C9(C)
|
:004A86D0 8D55D0                 lea edx, dword ptr [ebp-30]
:004A86D3 8B8340070000           mov eax, dword ptr [ebx+00000740]
:004A86D9 E8BAB7F9FF             call 00443E98
:004A86DE 8B45D0                 mov eax, dword ptr [ebp-30]
:004A86E1 50                     push eax
:004A86E2 8D4DCC                 lea ecx, dword ptr [ebp-34]
:004A86E5 BA01000000             mov edx, 00000001
:004A86EA 8BC7                   mov eax, edi
:004A86EC E8E705F6FF             call 00408CD8
:004A86F1 8B55CC                 mov edx, dword ptr [ebp-34]
:004A86F4 58                     pop eax
:004A86F5 E8A6C3F5FF             call 00404AA0 <====比较注册码
:004A86FA 7531                   jne 004A872D <====不相同就注册失败啦
:004A86FC 6840100000             push 00001040
* Possible StringData Ref from Code Obj ->"提示"
:004A8701 6890874A00             push 004A8790
```



* Possible StringData Ref from Code Obj -> `"感谢您的注册！为保存注册信息，请重新运行魔法壁"`
* Possible StringData Ref from Code Obj -> `"纸。"`

```text
:004A8706 6898874A00    push 004A8798
:004A870B 8BC3          mov eax, ebx
:004A870D E8C61EFAFF    call 0044A5D8
:004A8712 50            push eax
```

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, Buttons, StdCtrls, ExtCtrls;

type
  TForm1 = class(TForm)
    GroupBox1: TGroupBox;
    Edit1: TEdit;
    GroupBox2: TGroupBox;
    Edit2: TEdit;
    GroupBox3: TGroupBox;
    Edit3: TEdit;
    Panel1: TPanel;
    SpeedButton1: TSpeedButton;
    Panel2: TPanel;
    SpeedButton2: TSpeedButton;
    SpeedButton3: TSpeedButton;
    procedure SpeedButton3Click(Sender: TObject);
    procedure SpeedButton2Click(Sender: TObject);
    procedure SpeedButton1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

uses Unit2;

{$R *.dfm}

procedure TForm1.SpeedButton3Click(Sender: TObject);
begin
  form1.Close;
end;

procedure TForm1.SpeedButton2Click(Sender: TObject);
begin
  form2.show;
end;

procedure TForm1.SpeedButton1Click(Sender: TObject);
var
  name: string;
  namelen: byte;
  code: string;
  codehex: int64;
  coder5: integer;
  r1, r2, r3, r4: int64;
  regcode: string;
  position: pchar;
  i: byte;
begin
  name := edit1.Text;
  namelen := strlen(pchar(name));
  if namelen = 0 then
  begin
    messagedlg('请输入你的用户名！', mterror, [mbok], 0);
    exit;
  end;
  code := edit2.Text;
  if strlen(pchar(edit2.Text)) = 0 then
  begin
    messagedlg('请输入你的注册申请码！', mterror, [mbok], 0);
    exit;
  end;
  codehex := strtoint(code);
  coder5 := strtoint(copy(code, 6, 5));
  r1 := codehex + coder5;
  r2 := r1 + 1981813;
  r3 := r2 + namelen;
  coder5 := strtoint(copy(inttostr(r3), 6, 5));
  r4 := r3 + coder5;
  regcode := inttohex(r4, 16);
  i := 1;
  while (regcode[1] = '0') and (i < 17) do
  begin
    delete(regcode, 1, 1);
    i := i + 1;
  end;
  edit3.Text := regcode;
end;
end.
```

`4dd466d0e1e0cc123afbe504da2462ca`
