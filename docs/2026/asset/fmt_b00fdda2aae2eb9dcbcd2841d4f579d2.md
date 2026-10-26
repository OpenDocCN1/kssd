
# [原创]仿天易love读狂牛Key工具源码

这几天，网上下载了些视频教程学习，一运行才发现需要密码，以前也见过类似的EXE视频文件，但大都是加的穿山甲等等神么的壳，穿山甲大家研究的多了，所以相关的帮助教程也较多，无非就是将双进程变为单进程然后将壳脱去就能正确播放了或者是有正确的KEY直接Patch机器码等。但这次遇到的用Peid查壳，发现是Visual C++，再细看一下区段，发现好多不知名的东东，用ExeScope查看下资源，发现了一些信息：狂牛视频加密。以前没碰到过这东东，所以也无从下手，就来论坛里搜索下吧，发现坛子里的天易Love兄研究的较多，于是就下载了天易兄的工具也小试一下老版本的吧，不曾想在我的Win7下及虚拟机XP下运行后均出现乱码，如图：

![](https://bbs.kanxue.com/upload/attach/201211/97866_ip2qbx5znomiyzq.jpg)

再看看天易兄的文章，提到是用的硬编码，百度了一下，发现硬编码这东西貌似很难，无奈了，怎么办？从天易兄的文章中得到一些指导，索性自己写一个吧，仔细拜读了天易兄的文章，再加上他的工具，获得了很多有用的信息：工具的原理：1.获取所有进程；2.选择需要读KEY的狂牛视频进程，遍历内存，找到加密KEY所在的内存地址；3.读取这段内存中的值（当然这里搜索到的可能不止一处，但如果是精确搜索的话，出来的有两处，并且值是一样的，那么取其中一个就行了），然后与`84`（16进制值）进行异或，然后转为Asc字符串就是所需要的KEY了。我大致的理解是这样的，或许天易兄的工具在进行内存定位这一块有更加好的方法，我这方法比较笨。于是，就写了个类似的工具，代码如下：

```
unit frmMain;



interface



uses

  Windows, Messages, SysUtils, Variants, Classes, Graphics,

  Controls, Forms, Dialogs, StdCtrls, ComCtrls,

  uProcessUtils,StrUtils;



type

  TMainFrm = class(TForm)

    GroupBox1: TGroupBox;

    btnRefresh: TButton;

    btnReadKey: TButton;

    Edit1: TEdit;

    ListBox1: TListBox;

    btnEnumProcess: TButton;

    GroupBox2: TGroupBox;

    edtAscStr: TEdit;

    edtHexStr: TEdit;

    btnGetAsc: TButton;

    btnAsc2Hex: TButton;

    edtDestStr: TEdit;

    btnGetKey: TButton;

    procedure FormCreate(Sender: TObject);

    procedure btnRefreshClick(Sender: TObject);

    procedure ListBox1Click(Sender: TObject);

    procedure btnReadKeyClick(Sender: TObject);

    procedure btnEnumProcessClick(Sender: TObject);

    procedure btnGetAscClick(Sender: TObject);

    procedure btnAsc2HexClick(Sender: TObject);

    procedure btnGetKeyClick(Sender: TObject);

  private

    { Private declarations }

  public

    { Public declarations }



  end;



var

  MainFrm: TMainFrm;

implementation



{$R *.dfm}





procedure TMainFrm.btnAsc2HexClick(Sender: TObject);

var

   XorStr,HexStr : AnsiString;

   nLen,i,ordValue : Integer;

   chrArr : array of AnsiChar;

begin

    XorStr := 'wP%';

    nLen := Length(XorStr);

    SetLength(chrArr,nLen);

    for i:=1 to nLen do

    begin

        //chrArr[i] :=

        ordValue := Ord(XorStr[i]);

        HexStr := HexStr+IntToHex(ordValue,2);

    end;

    edtHexStr.Text := HexStr;

end;



procedure TMainFrm.btnEnumProcessClick(Sender: TObject);

begin

    //MemHandle.GetProcList;

    //GetKeyAddress('password=');

end;



procedure TMainFrm.btnGetAscClick(Sender: TObject);

var

   i     : Integer;

   hexStr,ascStr,tempStr,XorStr,s1,s2 : AnsiString;



begin

    hexStr := edtHexStr.Text;

    for i := 0 to (Length(hexStr)-1) div 2 do

    begin

        tempStr := '$'+ Copy(hexStr,i*2,2);

        ascStr := ascStr + Chr(StrToInt(tempStr));

    end;

    edtAscStr.Text := ascStr;

end;



procedure TMainFrm.btnGetKeyClick(Sender: TObject);

var

   XorStr,tempStr,hexStr : string;

   i ,nLen: Integer;

begin

   hexStr := '71F7DFA23E2F522093311B48FDA8A220077DD976ADE13';

   nLen := Length(hexStr);

   for i := 0 to ((nLen-1) div 2) do

   begin

       tempStr := '$'+AnsiMidStr(hexStr,i*2,2);

       //tempStr := '$'+ Copy(hexStr,i*2,2);

       XorStr := XorStr+Chr(StrToInt(tempStr) xor $84);

   end;

   edtDestStr.Text := XorStr;

end;



procedure TMainFrm.btnReadKeyClick(Sender: TObject);

var

   processID : Integer;

   keyStr : string;

   exeName : string;

begin

   if ListBox1.ItemIndex = -1 then

   begin

      Exit;

   end;

   processID := GetProcessID(ListBox1.Items.Strings[ListBox1.ItemIndex]);

   keyStr := ReadKey(processID);

   Edit1.Text := keyStr;

end;



procedure TMainFrm.btnRefreshClick(Sender: TObject);

var

   processList : TStringList;

   i : Integer;

begin

   //processList := TStringList.Create;

   ListBox1.Items.Clear;

   processList := GetProcessList();

   for i:=0 to processList.Count-1 do

   begin

      ListBox1.Items.Add(processList.Strings[i]);

   end;

end;



procedure TMainFrm.FormCreate(Sender: TObject);

var

   processList : TStringList;

   i : Integer;

begin

   //processList := TStringList.Create;

   ListBox1.Items.Clear;

   processList := GetProcessList();

   for i:=0 to processList.Count-1 do

   begin

      ListBox1.Items.Add(processList.Strings[i]);

   end;

end;



procedure TMainFrm.ListBox1Click(Sender: TObject);

begin

    //Edit1.Text := ListBox1.Items.Strings[ListBox1.ItemIndex];



end;



end.
```
