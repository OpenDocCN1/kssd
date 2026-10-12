
```
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls, Menus, jpeg, ExtCtrls, Buttons;

type
  TForm1 = class(TForm)
    img1: TImage;
    grp1: TGroupBox;
    lbl1: TLabel;
    Edit1: TEdit;
    lbl3: TLabel;
    btn1: TBitBtn;
    btn2: TBitBtn;
    lbl2: TLabel;
    Edit2: TEdit;
    procedure Edit1Click(Sender: TObject);
    procedure FormCreate(Sender: TObject);
    procedure btn2Click(Sender: TObject);
    procedure btn1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
const
   xm : array[1..7996] of Byte = (
     $45,$78,$74,$65,$6E,$64,$65,$64,$20,$4D,$6F,$64,$75,$6C,$65,$3A,
......
     $39,$38,$20,$3D,$5D,$02,$00,$00,$28,$00,$00,$00 );

implementation

uses Unit2, Unit3;
{$I XMobj\ufmod.inc *** uFMOD API: WINMM version}
{$R *.dfm}

procedure TForm1.Edit1Click(Sender: TObject);
begin
Edit1.Clear;
end;

procedure TForm1.FormCreate(Sender: TObject);
begin
uFMOD_PlaySong(@xm,Length(xm),XM_MEMORY);
end;

procedure TForm1.btn2Click(Sender: TObject);
begin
form3.show;
end;

procedure TForm1.btn1Click(Sender: TObject);
var
User:string;
Len,RegCode:Integer;
begin
  User:=Form1.Edit1.Text;
  Len:=Length(User);
  asm
      pushad           //保存各寄存器值，免得麻烦
      mov esi,User     //用户名放入 esi
      mov edx,Len      //用户名长度放入 edx
      xor eax,eax
      xor ecx,ecx      //ecx 计数
    @loop:
        movsx   ebx, byte ptr [ecx+esi]
        add     eax, ebx
        inc     ecx
        cmp     ecx, edx
        jl      @loop
      mov RegCode,eax    //计算出用户名各字符之和
      popad
  end;
  Form1.Edit2.Text:=IntToStr($022B8*RegCode+$05480204);
end;

end.
```

经测试，完全支持中文用户名注册！

此网站还有其余十余款类似软件，大家可自行下载练习。

由本文，我们看到了一个简单算法可能给程序带来的致命打击，偷懒害死人哪~~~

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) |
| --- |
| 注册机源码.rar |

**Video Converter**

`iPod Video Converter`

`DVD to iPod Converter`

`Video Converter`

`PSP Video Covnerter`

`FLV Video Converter`

`3GP Video Converter`

`MP4 Video Converter`

**Audio Converter**

`Ease Audio Converter`

`CD to MP3 Maker`

`Ease MP3 Recorder`

`Video Formate`

公司程序列表，我又研究了几个

发现上述所有软件算法基本不变，仅仅改动了一下参与运算的几个数值

汗哪，那样的话，只需半个小时，就能做出此公司全系列注册机了

不知道他们的程序是如何策划设计的:(

我们国产软件应该以此为戒，吸取教训啊
