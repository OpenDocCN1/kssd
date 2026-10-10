

# ANTI-DEDE

> 原文：[`www.pediy.com/kssd/pediy06/pediy6849.htm`](https://www.pediy.com/kssd/pediy06/pediy6849.htm)

DEDE 对于 DELPHI 程序的威胁是很大的，怎样 ANTI-DEDE 呢？  
  
方法一：在 FORM 上使用一个 FRAME，防止通过 DEDE 的 DFM EDITOR 直接跳到函数或过程里。  
   
  目前 DEDE 并不能很好地支持 FRAME，所以加上一个 FRAME，他就无法生成 FORM 的图形界面，现在可以利用 DEDE 的这个缺陷。  
  
  类似的缺陷，如果感兴趣可以自己找，比如：  
  在 FORM 里的某个控件里使用中文字，好像有的 DEDE 就无法正常显示这个 FORM，诸如此类。  
  
方法二：隐藏你的过程或函数：  
  有些人在分析 DELPHI 写的程序时，有时会发现在反编译后 FORM 的事件列表里竟然是空的，这是怎么实现的呢？  
  
  DEDE 使用 DELPHI 的 RTTL 原理，所以可以轻易地列出所有 PUBLISHED 下面的函数及过程名--注意是过程名，所以通过 DEDE 可以轻松地找到函数或过程的入口。  
  
  如果你真的想阻止 DEDE 简单准确地跳到函数或过程里，可以尝试下面的方法：  
  不要用 DELPHI 自动生成事件，或者是最后自己手工删除在属性框里对这些事件引用，改为动态生成事件。  
     但 注意：这比较累，要写些动态代码。是不是要真的这样做，你自己考虑好。  
  把 PRIVATE 上面的函数或过程移动到 PRIVATE 里面定义。  
  
  这样作的好处是：  
   DEDE 无法直接列出实名的过程或函数；但某些版本仍可以列出全部随机命名的事件，但这时 DEDE 可以提供的帮助就有限了；  
   由于事件是动态生成，所以想通过查找 FORM 里的类似 OnClick 事件就找到入口地址的努力化为泡影，或者说增加了难度。   
   如果你的一个 FORM 都是利用上面的方法生成的--比如连 FORMCREATE 你都自定义，那 DEDE 在显示事件栏里会显示空。  
  
   缺点：  
     手工多；必须删除属性框里生成的事件代码，比较累，偶尔玩玩还可以。  
  
  
简单的例子：  
//=========================================================  
unit Unit1;    
interface     
uses  
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,  
  Dialogs, StdCtrls;  
  
type  
  TForm1 = class(TForm)  
    Button1: TButton;  
  private  
    { Private declarations }  
    //！！！  
    procedure yourFormCreate;  
    procedure yourFormClose(Sender: TObject; var Action: TCloseAction);  
    procedure yourFormDestroy(Sender: TObject);  
  
    procedure yourButton1Click(Sender: TObject);  
  
  public  
    { Public declarations }  
  
    constructor Create(AOwner: TComponent); override;  
  end;  
  
var  
  Form1: TForm1;  
  
implementation  
  
{$R \*.dfm}  
  
constructor TForm1.Create(AOwner: TComponent);  
begin   
  inherited Create(AOwner);  
  
  yourFormCreate;  
  
end;  
  
procedure TForm1.yourFormCreate;  
begin         
//!!!!!!!  
    OnClose                         := yourFormClose;  
    OnDestroy                       := yourFormDestroy;  
    Button1.OnClick                 := yourButton1Click;  
  
end;  
  
procedure TForm1.yourFormClose(Sender: TObject; var Action: TCloseAction);  
begin  
  showmessage('yourFormClose');  
  close;  
end;  
  
procedure TForm1.yourFormDestroy(Sender: TObject);  
begin  
  showmessage('yourFormDestroy');  
end;  
procedure TForm1.yourButton1Click(Sender: TObject);  
begin  
  showmessage('yourButton1Click');  
end;  
  
end.  
//=========================================================  
  
 以上是我试出来的，欢迎大家抨击不当之处，谢谢！  
   
 新建工程,只添加一个 Button1，大家把以上代码复制到 UNIT1.pas 里，接着就编译，然后用 DEDE 载入，看看可以看到什么？？呵呵，在 DEDE 里查看 FORM1 的事件列表是“空”的！  
  
  注意：与方法一结合起来，会更有效地阻止他人取得事件入口点。方法一自己去试吧，就不贴代码了。 

  
 4dd466d0e1e0cc123afbe504da2462ca