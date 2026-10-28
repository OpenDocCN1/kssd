
# [旧帖][原创]保密卡程序 附效果图+源码+程序0.00雪花

![](https://bbs.kanxue.com/upload/attach/201201/378330_z3mr5dos2bfpgeb.gif)

一直用巨盾自带的保密卡程序，每次都要启动巨盾，然后输密码查保密卡。感觉不是很好，一是巨盾占内存资源大，二是程序界面不很友好，不太喜欢。故根据自己的喜好和设想，设计编写一个保密卡程序。

## 实现的功能：

一、输入正确的密码后显示保密卡界面，防止别人偷偷使用。

二、卡上坐标显示，有形象的十字指示标志。

三、点下相应坐标后显示密码，点击其他坐标后原坐标位置隐藏。

四、支持多个卡操作。

五、显示卡时防截屏。

六、程序加密。

第一个功能：实现很简单，用一个分组框覆盖总程序界面，用一个编辑框，加一个按钮，通过判断输入密码和设定密码是否一致来取消分组框的显示。

第二个功能：程序的难点，我的想法是通过建立组件数组来动态创建`BitBtn`组件，然后通过鼠标的进入离开事件来实现十字指示标志。

第三个功能：通过建立一个临时`BitBtn`组件记录上次按下的组件位置，通过`visble`属性来实现。

第四个功能：简单，`image`组件读入不同的卡就可以。

第五个功能：一般，用时钟组件，反复清空剪辑板可实现。

第六个功能：简单，现成的`exe`加壳工具如`aspack`，`ups`等。

## 下面是窗体代码：

```pascal
unit USafeCard;

{$R 'Card.res' 'Card.rc'}

interface

uses

  Windows, SysUtils, Classes, Forms,Dialogs,Graphics,Clipbrd,

  StdCtrls, ExtCtrls, Buttons, Controls;

type

  TForm8 = class(TForm)

    GroupBox1: TGroupBox;

    Image1: TImage;

    BitBtn1: TBitBtn;

    Button1: TButton;

    Button2: TButton;

    Shape1: TShape;

    Label1: TLabel;

    Label2: TLabel;

    Panel1: TPanel;

    Timer1: TTimer;

    GroupBox2: TGroupBox;

    Edit1: TEdit;

    Button3: TButton;

    procedure BitBtn1Click(Sender: TObject);

    procedure BitBtn1MouseEnter(Sender: TObject);

    procedure BitBtn1MouseLeave(Sender: TObject);

    procedure FormCreate(Sender: TObject);

    procedure Button1Click(Sender: TObject);

    procedure Button2Click(Sender: TObject);

    procedure Shape1MouseEnter(Sender: TObject);

    procedure Shape1MouseLeave(Sender: TObject);

    procedure Timer1Timer(Sender: TObject);

    procedure Button3Click(Sender: TObject);

  private

    { Private declarations }

  public

    { Public declarations }

  end;

var

  Form8: TForm8;

  BitBtn:array[1..8,1..15] of TBitBtn;//二维数组组件的定义

  BitBtnTemp:TBitBtn;//记录上次点击按钮的临时组件

implementation

{$R *.dfm}

procedure TForm8.BitBtn1Click(Sender: TObject);

begin

  (Sender as TBitBtn).Visible:=False;

  if BitBtnTemp=nil then

  begin

    BitBtnTemp:= (Sender as TBitBtn);

    Exit;

  end;

  if not(BitBtnTemp=nil) then

  begin

    BitBtnTemp.Visible:=True;

    BitBtnTemp:=(Sender as TBitBtn);

    Exit;

  end;

end;

procedure TForm8.BitBtn1MouseEnter(Sender: TObject);

var

BBTemp:TBitBtn;

PosX,PosY:Integer;

i,j:Integer;

begin

   BBTemp:=(Sender as TBitBtn);

   PosY:=BBTemp.Tag div 10;//10-150

   PosX:=BBTemp.Tag mod 10;//1-8

   Label2.Caption:=Char(PosX+64)+IntToStr(PosY div 10);//Char(65):A

   for i := 1 to 8 do

   begin

      if i=PosX then

      begin

        for j := 1 to 15 do

           begin

             BitBtn[i,j].Kind:=bkRetry;

             BitBtn[i,j].Caption:='&';

           end;

      end;

   end;

   for j := 1 to 15 do

   begin

      if j=(PosY div 10) then

      begin

        for i := 1 to 8 do

           begin

             BitBtn[i,j].Kind:=bkRetry;

             BitBtn[i,j].Caption:='&';

           end;

      end;

   end;

end;

procedure TForm8.BitBtn1MouseLeave(Sender: TObject);

var

i,j:Integer;

begin

  Image1.Visible:=False;

  for i := 1 to 8 do

  begin

    for j := 1 to 15 do

      begin

        if BitBtn[i,j].Kind=bkRetry then

        begin

          BitBtn[i,j].Kind:=bkCustom;

          BitBtn[i,j].Glyph:=nil;

          BitBtn[i,j].Caption:='';

        end;

      end;

  end;

  Image1.Visible:=True;//有时刷新会暴露按钮后的图片，不知何原因，故控制图片的显示时间

end;

procedure TForm8.Button1Click(Sender: TObject);

begin

    Image1.Picture.Bitmap.LoadFromResourceName(HInstance,'ABNNEK'); //Load card1 模糊名称

    if not(BitBtnTemp=nil) then

    begin

      BitBtnTemp.Visible:=True;

      BitBtnTemp:=nil;

    end;

end;

procedure TForm8.Button2Click(Sender: TObject);

begin

    Image1.Picture.Bitmap.LoadFromResourceName(HInstance,'BBTKR'); //Load card2

    if not(BitBtnTemp=nil) then

    begin

      BitBtnTemp.Visible:=True;

      BitBtnTemp:=nil;

    end;

end;

procedure TForm8.Button3Click(Sender: TObject);

begin

  if Edit1.Text='mmm   ' then

  GroupBox2.Visible:=False;

end;

procedure TForm8.FormCreate(Sender: TObject);

var

i,j:Integer;

begin

  SetWindowPos(Handle,HWND_TOPMOST,0,0,Self.Width,Self.Height,SWP_SHOWWINDOW);//令窗体在最前

  for i := 1 to 8 do

  begin

    for j := 1 to 15 do

      begin

        BitBtn[i, j]:=TBitBtn.Create(Self);//动态创建组件，create参数多用self，也就是窗体本身

        BitBtn[i, j].Parent:=Panel1;//动态创建组件，要正常显示需指定父容器

        BitBtn[i, j].Width:=30;

        BitBtn[i, j].Height:=28; //大小

        BitBtn[i, j].Left:=29*(i-1);

        BitBtn[i, j].Top:=26*(j-1);  //位置

        BitBtn[i, j].OnClick:=BitBtn1Click;  //批量响应事件

        BitBtn[i, j].OnMouseEnter:=BitBtn1MouseEnter;

        BitBtn[i, j].OnMouseLeave:=BitBtn1MouseLeave;

        BitBtn[i, j].TabStop:=False;

        BitBtn[i, j].Tag:=100*j+i;//通过tag来识别组件

      end;

  end;

  Image1.Picture.Bitmap.LoadFromResourceName(HInstance,'ABNNEK'); //Load card1 模糊名称

  Edit1.SetFocus;

end;

procedure TForm8.Shape1MouseEnter(Sender: TObject);

begin

  Shape1.Pen.Mode:=pmNop;//显示卡序号

end;

procedure TForm8.Shape1MouseLeave(Sender: TObject);

begin

  Shape1.Pen.Mode:=pmWhite;//覆盖卡序号

end;

procedure TForm8.Timer1Timer(Sender: TObject);

begin

  Clipboard.Clear;//引用Clipbrd单元，则可以直接调用Clipboard全局变量。

end;

end.
```

## 下面是资源代码：

```pascal
ABNNEK BITMAP "1.bmp"  
BBTKR BITMAP "2.bmp" 
```

## 几点心得体会：

一、数组的使用。看明白和会使用是两回事，如何定义，如何使用要通过实践来加深记忆。

二、剪辑板的使用，`Clipbrd`组件`use`后`Clipboard`就代表系统剪辑板。

三、`div`和`mod`对定位坐标的使用。

四、组件`tag`属性的用途，在这个程序里我吧组件信息存在里面了。

存在问题：有时候组件刷新会在很短的时间内显示部分密保卡图片，不知道如何解决。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [SafeCard.rar](javascript:void(0);) （104.09kb，16次下载）
- [PSafeCard.rar](javascript:void(0);) （391.39kb，21次下载）
- [2010103113184528.gif](javascript:void(0);) （224.91kb，14次下载）
