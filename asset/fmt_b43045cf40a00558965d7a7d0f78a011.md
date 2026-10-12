
# PBackhomev 1.02

> 原文：[`www.pediy.com/kssd/pediy03/forum1044.htm`](https://www.pediy.com/kssd/pediy03/forum1044.htm)

`//回家（PBackhome)v 1.02 版注册机`

`//delphi5.0 编译`

`//注册码计算：`

`//取计算机用户名（转为大写字母）作为机器码，机器码字符长度不足 15 位,`

`//则在机器码后依次加上 PolarBearSoftware，凑足 15 位字符`

`//分别取每个字符的 ASCII 码，转为 10 进制数值，加 1，累积求和 sum1`

`//若机器码字符长度<15 位，sum1\*64\*机器码字符长度＝注册码`

`//否则 sum1\*64\*15=注册码`

```
unit Unit3;

interface

uses

 Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
 StdCtrls;

type

 TForm1 = class(TForm)
   Edit1: TEdit;
   Edit2: TEdit;
   Button1: TButton;
   Label1: TLabel;
   Label2: TLabel;
   Button2: TButton;
   Label3: TLabel;
   procedure Button1Click(Sender: TObject);
   procedure Button2Click(Sender: TObject);
 private
   { Private declarations }
 public
   { Public declarations }
 end;

var

 Form1: TForm1;
 a,b,d,e,h,i,len1,len2:longint;
 c:String;

implementation

{$R \*.DFM}

procedure TForm1.Button1Click(Sender: TObject);

begin
   len1 := length(edit1.text);    //len=机器码长度
   name := edit1.text;            //name=机器码
   b := 0;
   e := 0;
   for i := 1 to len1 do
   begin
       a := ord(name\[i\])  + 1; //取机器码每个字符 ascii 码，转为 10 进制数
       b := a + b;               //累计求和
   end;
   len2 := 15 - len1;

   if len2 <= 0 then edit2.text := inttostr(b\*64\*15);
   if len2 > 0 then
       begin
           c := 'PolarBearSoftware';
           for h := 1 to len2 do
               begin
               d := ord(c\[h\]) + 1;
               e := d + e;
               end;
           edit2.text := inttostr((b+e)\*64\*len1);
       end;
end;

procedure TForm1.Button2Click(Sender: TObject);

begin
close;
end;

end.
```
`4dd466d0e1e0cc123afbe504da2462ca`
