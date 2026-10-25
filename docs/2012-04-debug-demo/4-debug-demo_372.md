
# 《韦氏可视化词典》之 `HTMDIY`

> 原文：[`www.pediy.com/kssd/pediy10/103887.html`](https://www.pediy.com/kssd/pediy10/103887.html)

《韦氏可视化词典》之 `HTMDIY` By 天易 love 2009-12-27

## 时代背景
2009 年即将过去，为了给以后留下点可以回味的文字，特此撰文一篇，希望对正在学习和即将学习英语的朋友提供一条捷径。《韦氏可视化词典》(`Merriam-webster Visual Dictionary`) 是一本不可多得的好词典。前两天偶尔得到且试用了一下，发觉学习效果非比一般。但是有一点细节方面的问题需要修改，否则对学习效率有所影响。

### 一、存在问题
对类似下面插图中的文字不能鼠标即指即读，影响学习效果。

`House.Jpg`

![](img/8fd2f2d8e01665d44de8da9e01ec72c1.jpg)

`Fruit.jpg`

![](img/7bc02fa95a043ecf95c7343dde42504c.jpg)

### 二、分析原因
通过分析 `htm` 文件内容，发现插图中的每个文字标题都对应类似下面的一行代码：

```html
<area shape="rect" coords="428,307,474,322" href="#worker10692" alt="worker" />
```

原本鼠标单击插图中的每个文字标题都会跳转到 `href="#worker10692"` 所指定的地方，但使用中发现，这样的效果很差，基本上不知道一下子跳到了哪里。我个人认为在插图上还是鼠标即指即读效果更好。其实很简单，只要把 `href="#worker10692"` 替换为

```html
onmouseover="DHTMLSound('../../sounds/astronomy/astronomical-observation/05971.aif')"
```

即可解决问题。

只是这个声音文件的路径要利用 `worker10692` 这个关键字来搜索网页内容提取。例如：

先定位网页中该处，然后即可提取出对应的声音文件详细路径。

```html
<div class="descript" id="worker10692">
<h4>aperture door <span id="dummyspan"><embed src='../../sounds/astronomy/astronomical-observation/05971.aif' hidden=true autostart=false loop=false height=1 width=1></span>
<img src="../../images/big-speaker.jpg" onmouseup="DHTMLSound('../../sounds/astronomy/astronomical-observation/05971.aif')"></h4></div>
```

修改成这样：
```html
<area shape="rect" coords="428,307,474,322" onmouseover="DHTMLSound('../../sounds/astronomy/astronomical-observation/05971.aif')" />
```

### 三、解决问题
由于整个词典接近 `1.8G`，仅仅声音文件就接近 `2` 万个，需要修改的地方多如牛毛，只能编个程序（懒人被逼急了使出了杀手锏）。工具还是用 `Delphi` 快，花了我半天时间（时间都花在语法上了），算法代码极其丑陋，反正也只用一次，处理完就可以扔掉。通过实际测试，`2009` 年购买的 `4500` 左右的 `thinkpad` 需要 `60` 秒，其他价格品牌依次类推，再打个更形象的比方就是尿个尿回来就搞定了。随便打开一个网页即指即读，一个字爽！

![](img/04622cb2dd9244f260a7e4011ec544ef.jpg)

贴一下代码：

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Controls, Forms,
  Dialogs, StdCtrls, FileCtrl, Classes, Grids, Outline, DirOutln;

type
  TForm1 = class(TForm)
    Memo1: TMemo;
    Button1: TButton;
    Memo2: TMemo;
    Button2: TButton;
    DirectoryOutline1: TDirectoryOutline;
    DriveComboBox1: TDriveComboBox;
    Label1: TLabel;
    Memo3: TMemo;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure DriveComboBox1Change(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  number: integer;

implementation

{$R *.dfm}

function newline(tt: string): string;
var
  i, j, lines: integer;
  ss: string;
begin
  for lines := Form1.memo2.lines.count - 1 downto 0 do
    begin
      if (pos(tt, Form1.memo2.lines[lines]) <> 0) then
        begin
          i := pos('DHTMLSound(', Form1.memo2.lines[lines + 1]);
          j := pos(')">', Form1.memo2.lines[lines + 1]);
          ss := copy(Form1.memo2.lines[lines + 1], i + 11, j - i - 11);
          break;
        end;
    end;
  result := 'onmouseover="DHTMLSound(' + ss + ')"  />';
end;

procedure TForm1.Button1Click(Sender: TObject);
var
  i, j: byte;
  F: TextFile;
  path, singleLine, content, tt, tt1, tt2: string;
  lines: integer;
begin
  for lines := 0 to memo3.lines.count - 1 do
    begin
      path := memo3.lines[lines];
      memo2.Lines.LoadFromFile(path);
      content := memo2.Text;
      AssignFile(F, path);
      Reset(F);
      while (not eof(F)) do
        begin
          Readln(F, singleLine);
          i := pos('" href="#', singleLine);
          if i <> 0 then
            begin
              j := pos('" alt="', singleLine);
              tt := copy(singleLine, i + 9, j - i - 9);
              tt1 := copy(singleLine, 1, i + 1);
              tt2 := tt1 + newline(tt);
              Memo1.Lines.Append(tt2);
            end
            else
              Memo1.Lines.Append(singleLine);
        end;
      CloseFile(F);
      memo1.Lines.SaveToFile(path);
      memo1.Clear;
      memo2.Clear;
    end;
  Label1.Caption := Label1.Caption + '  处理完: ' + inttostr(memo3.lines.count) + ' 个';
end;

function searchHtm(dir: string): boolean;
var
  sr: tsearchrec;
  sdir: string;
begin
  sdir := dir + '\*.*';
  if findfirst(sdir, faanyfile, sr) = 0 then
    begin
      repeat
        begin
          if (sr.Name = '.') or (sr.Name = '..') then continue;
          if sr.Attr and fadirectory = 0 then
            begin
              if ExtractFileExt(sr.Name) = '.htm' then
                begin
                  form1.Memo3.Lines.Append(dir + sr.Name);
                  number := number + 1;
                end;
            end
            else
              begin
                searchHtm(dir + sr.Name + '\');
              end;
        end
      until findnext(sr) <> 0;
      findclose(sr);
    end;
  result := true;
end;

procedure TForm1.Button2Click(Sender: TObject);
var
  dir: string;
begin
  number := 0;
  Label1.Caption := '网页文件:   个';
  dir := DirectoryOutline1.Directory + '\';
  if searchHtm(dir) then
    Label1.Caption := '网页文件: ' + inttostr(number) + ' 个';
  number := 0;
end;

procedure TForm1.DriveComboBox1Change(Sender: TObject);
var
  str: string;
  cc: char;
begin
  str := DriveComboBox1.Text;
  cc := str[1];
  DirectoryOutline1.Drive := cc;
end;

end.
```
