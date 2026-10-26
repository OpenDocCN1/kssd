
以下是获取硬盘序列号并生成注册码的 PowerBuilder 代码片段，以及对应的注册机 Pascal 源码。

```powerbuilder
alpha[36] = "9"

randomize(0)

for i = 1 to 8
    ss = rand(36)
    alser = alser + alpha[ss]
next

return alser
end function

public function string f_genser ();
string alser
string tser
char tmpchr
char tmpchr1
integer i
integer j
string ls_mhser

alser = f_getdisk()
alser = f_swap1(alser)
ls_mhser = f_genpuzzle()
ls_mhser = f_swap1(ls_mhser)
tser = left(ls_mhser,4) + left(alser,4) + right(ls_mhser,4) + right(alser,4)

return tser
end function

public function string f_getdisk ();
long retval
string sdrv
string str
string str2
string ls_retval
long a
long b
ulong ll_retval
n_cst_numerical ln_1
string ls_format = "0000"
integer li_len

sdrv = "C:\"
str = space(256)
str2 = space(256)
ll_retval = getvolumeinformationa(sdrv,str,256,retval,a,b,str2,256) //调用API，获取硬盘序列号
ls_retval = ln_1.of_hex(retval) //转换为16进制串

if len(ls_retval) > 4 then
    li_len = len(ls_retval) - 4
    ls_retval = upper(left(ls_format,4 - li_len) + left(ls_retval,li_len) + right(ls_retval,4)) //若不足8位，在串前补0
end if

return ls_retval
end function

public function string f_redreg (string xsxt_key);
string ls_value
string ls_path
integer li_flag

ls_path = "HKEY_CURRENT_USER\Software\xsxt\xsxt_" + xsxt_key
li_flag = registryget(ls_path,xsxt_key,regstring!,ls_value)

if li_flag = 1 then
    return ls_value
else
    return "err"
end if
end function

public function integer f_setreg (string str_num,string str_info);
string ls_path
integer li_flag

ls_path = "HKEY_CURRENT_USER\Software\xsxt\xsxt_" + str_num
li_flag = registryset(ls_path,str_num,regstring!,str_info)

if li_flag = 1 then
    return 0
else
    return -1
end if
end function

public function string f_swap1 (string str_input);
char alpha[36,2]
string tser
string alser
char tmpchr
char tmpchr1
integer i
integer j
integer strlen

alpha[1,1] = "A"
alpha[2,1] = "B"
alpha[3,1] = "C"
alpha[4,1] = "D"
alpha[5,1] = "E"
alpha[6,1] = "F"
alpha[7,1] = "G"
alpha[8,1] = "H"
alpha[9,1] = "I"
alpha[10,1] = "J"
alpha[11,1] = "K"
alpha[12,1] = "L"
alpha[13,1] = "M"
alpha[14,1] = "N"
alpha[15,1] = "O"
alpha[16,1] = "P"
alpha[17,1] = "Q"
alpha[18,1] = "R"
alpha[19,1] = "S"
alpha[20,1] = "T"
alpha[21,1] = "U"
alpha[22,1] = "V"
alpha[23,1] = "W"
alpha[24,1] = "X"
alpha[25,1] = "Y"
alpha[26,1] = "Z"
alpha[27,1] = "0"
alpha[28,1] = "1"
alpha[29,1] = "2"
alpha[30,1] = "3"
alpha[31,1] = "4"
alpha[32,1] = "5"
alpha[33,1] = "6"
alpha[34,1] = "7"
alpha[35,1] = "8"
alpha[36,1] = "9"

alpha[1,2] = "Z"
alpha[2,2] = "X"
alpha[3,2] = "C"
alpha[4,2] = "V"
alpha[5,2] = "B"
alpha[6,2] = "9"
alpha[7,2] = "M"
alpha[8,2] = ","
alpha[9,2] = "."
alpha[10,2] = "/"
alpha[11,2] = "7"
alpha[12,2] = ";"
alpha[13,2] = "L"
alpha[14,2] = "8"
alpha[15,2] = "J"
alpha[16,2] = "H"
alpha[17,2] = "6"
alpha[18,2] = "F"
alpha[19,2] = "D"
alpha[20,2] = "S"
alpha[21,2] = "1"
alpha[22,2] = "P"
alpha[23,2] = "O"
alpha[24,2] = "I"
alpha[25,2] = "U"
alpha[26,2] = "Y"
alpha[27,2] = "E"
alpha[28,2] = "Q"
alpha[29,2] = "R"
alpha[30,2] = "T"
alpha[31,2] = "W"
alpha[32,2] = "A"
alpha[33,2] = "G"
alpha[34,2] = "]"
alpha[35,2] = "K"
alpha[36,2] = "N"

tser = ""
alser = trim(str_input)
strlen = len(alser)

for i = 1 to strlen
    tmpchr = mid(alser,i,1)
    for j = 1 to 36
        if tmpchr = alpha[j,1] then
            tmpchr1 = alpha[j,2]
            tser = tser + tmpchr1
        end if
    next
next

return tser
end function

public function string uf_patchstr (string userstr,integer userlen);
integer i
integer strlen

userstr = trim(userstr)

if len(userstr) > userlen then
    messagebox("错误","长度错误,无法转换")
    return "error"
end if

strlen = userlen - len(userstr)
for i = 1 to strlen
    userstr = "0" + userstr
next

return userstr
end function

on user_reg.create
call super::create;
end on

on user_reg.destroy
call super::destroy;
end on
```

在上面的代码中，关键部分作为注释，现总结一下：

对注册起作用的是`f_checkregcode`和`f_getdisk`。

注册码算法为：

1. 注册码为 16 位长度的串，为大写。
2. 取出硬盘序列号后，若为 8 位 16 进制串，则不变，否则在前补 0，拼成 8 位串。
3. 对所得的硬盘序列号串，按`f_checkregcode`中的明密文转换表，进行变换，变换的结果，分别记入注册表串的奇数位，即 1，3，5，7，9，11，13，15 的位置。
4. 注册码偶数位的字符任意，只要是明密文表中能找到可以了。我在注册机中指定为'A'。

注册机源码：

```pascal
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls, Strutils;

type
  TForm1 = class(TForm)
    Edit1: TEdit;
    Label1: TLabel;
    procedure FormCreate(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

procedure TForm1.FormCreate(Sender: TObject);
var
  serialnumber: integer;
  comlen: DWORD;
  flag: DWORD;
  str: AnsiString;
  alpha: array[1..36, 1..2] of char;
  i, j: integer;
  len: integer;
  enstr: array[1..16] of char;
begin
  GetVolumeInformation(Pchar('c:\'), nil, MAX_PATH + 1, @serialnumber, comlen, flag, nil, MAX_PATH + 1);
  str := inttohex(serialnumber, 4);
  len := length(str);
  if (len > 4) then
  begin
    len := len - 4;
    // ls_retval = upper(left(ls_format,4 - li_len) + left(ls_retval,li_len) + right(ls_retval,4))
    str := leftstr('0000', 4 - len) + leftstr(str, len) + rightstr(str, 4);
  end;
  str := UpperCase(str);
  Label1.Caption := str;

  alpha[1][1] := 'A';
  alpha[2][1] := 'B';
  alpha[3][1] := 'C';
  alpha[4][1] := 'D';
  alpha[5][1] := 'E';
  alpha[6][1] := 'F';
  alpha[7][1] := 'G';
  alpha[8][1] := 'H';
  alpha[9][1] := 'I';
  alpha[10][1] := 'J';
  alpha[11][1] := 'K';
  alpha[12][1] := 'L';
  alpha[13][1] := 'M';
  alpha[14][1] := 'N';
  alpha[15][1] := 'O';
  alpha[16][1] := 'P';
  alpha[17][1] := 'Q';
  alpha[18][1] := 'R';
  alpha[19][1] := 'S';
  alpha[20][1] := 'T';
  alpha[21][1] := 'U';
  alpha[22][1] := 'V';
  alpha[23][1] := 'W';
  alpha[24][1] := 'X';
  alpha[25][1] := 'Y';
  alpha[26][1] := 'Z';
  alpha[27][1] := '0';
  alpha[28][1] := '1';
  alpha[29][1] := '2';
  alpha[30][1] := '3';
  alpha[31][1] := '4';
  alpha[32][1] := '5';
  alpha[33][1] := '6';
  alpha[34][1] := '7';
  alpha[35][1] := '8';
  alpha[36][1] := '9';

  alpha[2][2] := 'Z';
  alpha[1][2] := 'X';
  alpha[4][2] := 'C';
  alpha[3][2] := 'V';
  alpha[6][2] := 'B';
  alpha[5][2] := '9';
  alpha[8][2] := 'M';
  alpha[7][2] := ',';
  alpha[10][2] := '.';
  alpha[9][2] := '/';
  alpha[12][2] := '7';
  alpha[11][2] := ';';
  alpha[14][2] := 'L';
  alpha[13][2] := '8';
  alpha[16][2] := 'J';
  alpha[15][2] := 'H';
  alpha[18][2] := '6';
  alpha[17][2] := 'F';
  alpha[20][2] := 'D';
  alpha[19][2] := 'S';
  alpha[22][2] := '1';
  alpha[21][2] := 'P';
  alpha[24][2] := 'O';
  alpha[23][2] := 'I';
  alpha[26][2] := 'U';
  alpha[25][2] := 'Y';
  alpha[28][2] := 'E';
  alpha[27][2] := 'Q';
  alpha[30][2] := 'R';
  alpha[29][2] := 'T';
  alpha[32][2] := 'W';
  alpha[31][2] := 'A';
  alpha[34][2] := 'G';
  alpha[33][2] := ']';
  alpha[35][2] := 'N';
  alpha[36][2] := 'K';

  j := 1;
  for i := 1 to 16 do
  begin
    if i mod 2 <> 0 then
    begin
      for len := 1 to 36 do
      begin
        if alpha[len][1] = str[j] then
        begin
          enstr[i] := alpha[len][2];
```
