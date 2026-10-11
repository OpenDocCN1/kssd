
# 直销业绩考核系统

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-294.htm`](https://www.pediy.com/kssd/pediy08/pediy8-294.htm)

**【文章标题】:** 直销业绩考核系统算法分析

**【文章作者】:** caterpilla（惊涛）

**【软件名称】:** 直销业绩考核系统

**【下载地址】:** 自己搜索下载

**【编写语言】:** `POWERBUILDER`

**【使用工具】:** `UE`，`PBKILLER`，`OD`

**【作者声明】:** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

这个软件笨鸟我先飞兄弟已经爆破过了，而且应用 `PBKILLER` 反编译了软件的源码，他把软件和反编译的代码给了我，我只是在阅读源码后，整理了一下算法，没有深入测试。

从源码开始看起了，呵呵，先贴几个关键的。

1、在系统启动时，应用程序调用检测是否注册的用户对象，判断是否注册、过期、调整过爱期、以及是否手工改过注册等。

```
global type dotnew from application

...

integer rt_chk_value
user_reg myreg
event open;ulong l_handle
ulong lu_class
string ls_name

rt_chk_value = myreg.f_check_reg()
if rt_chk_value = -100 then
    messagebox("提示","系统时间有误！")
    halt close
end if
if rt_chk_value = -200 then
    messagebox("提示","注册码有误！")
    halt close
end if
if rt_chk_value = -300 then
    messagebox("提示","试用期到，请注册！电话：13336268237 网址：[www.zcrj.net](http://www.zcrj.net)")
    open(w_wrcode)
    return
end if
if rt_chk_value = -400 then
    messagebox("提示","系统破坏，请联系开发商！")
    return
end if
```

下面的代码就是 `USER_REG` 用户对象，承担检测注册任务。

```
global type user_reg from n_cst_numerical autoinstantiate
end type

global user_reg user_reg

forward prototypes
public function integer f_check_reg ()//检测是否过期、爱期调整、注册，内部调用 f_checkregcode
public function integer f_checkregcode (string reg_code)//关键的检查注册码的部分
public function string f_encode (string source_str)//对注册表信息加密的部分
public function string f_genpuzzle ()
public function string f_genser ()
public function string f_getdisk ()//获取硬盘序列号，与注册码生成有关
public function string f_redreg (string xsxt_key)//读注册表
public function integer f_setreg (string str_num,string str_info)//写注册表
public function string f_swap1 (string str_input)
public function string uf_patchstr (string userstr,integer userlen)
end prototypes
```

这里面其它的函数都是起迷惑作用的，呵呵。。。。。。。。。。。。。

```
public function integer f_check_reg ();string ls_right
string ls_wrong
ulong l_handle
ulong lu_class
string ls_name
string ls_disk
string ls_ser
string ls_flag
string ls_day
string ls_num
string ls_ltime
string ls_regflag
string ls_regcode
date ld_d1
date ld_td
integer tsh
integer tshch
integer check_result

ls_right = f_encode(f_redreg("0"))
ls_wrong = f_encode(f_redreg("1"))
if ((ls_right <> "wright") or (ls_wrong <> "wrong")) then//检测软件是否被破坏，手工改过注册表
    return -400
end if

ls_regflag = f_redreg("3")   //错误标记
if ls_regflag = "err" then
    ls_flag = f_encode("90")
    ls_day = f_encode("30")
    ls_disk = f_encode(f_getdisk())
    f_setreg("2",f_encode(f_genser()))
    f_setreg("3",ls_flag)
    f_setreg("4",f_encode("f7657lo"))
    f_setreg("5",ls_day)
    f_setreg("6",f_encode("fj;\[57lo"))
    f_setreg("7",f_encode(string(today())))
    f_setreg("8",f_encode("fj57lo"))
    return 100
end if

ls_ser = f_encode(f_redreg("2"))
ls_flag = f_encode(f_redreg("3")) //注册表中在 HKEY_CURRENT_USER\\Software\\xsxt\\xsxt_3 下存放是否注册的标志
ls_day = f_encode(f_redreg("5"))
ls_ltime = f_encode(f_redreg("7"))
ls_regcode = f_encode(f_redreg("9"))
ld_d1 = date(ls_ltime)
ld_td = today()
tsh = daysafter(ld_d1,ld_td)  //安装爱期与当前爱期比较

if tsh < 0 then
    return -100 落后 //若小，说明改过爱期
end if

if ls_flag = "89" then  //89 这个标记，代表已经注册，继续检测注册码是否合法
    check_result = f_checkregcode(ls_regcode)//调用 f_checkregcode 检查注册码合法性
    if check_result = 0 then
        return 200   //注册
    else
        return -200 //注册码不合法
    end if
else
    tshch = integer(ls_day) - tsh//代表已过期
    if tshch <= 0 then
        f_setreg("5","00")
        f_setreg("7",f_encode(string(today())))
        return -300
    else
        ls_day = f_encode(uf_patchstr(string(tshch),2))
        f_setreg("5",ls_day)
        f_setreg("7",f_encode(string(today())))
        return 300
    end if
end if
end function
```

```
public function integer f_checkregcode (string reg_code);char alpha[36,2]//关键部分
string tser
string ls_disk
string ls_disk1
char tmpchr
char tmpchr1
integer i
integer j

ls_disk = f_getdisk()//获取硬盘序列号
ls_disk1 = ""

//下面的 ALPHA 为明密文转换表，两列相互转化，互为加解密。
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

alpha[2,2] = "Z"
alpha[1,2] = "X"
alpha[4,2] = "C"
alpha[3,2] = "V"
alpha[6,2] = "B"
alpha[5,2] = "9"
alpha[8,2] = "M"
alpha[7,2] = ","
alpha[10,2] = "."
alpha[9,2] = "/"
alpha[12,2] = "7"
alpha[11,2] = ";"
alpha[14,2] = "L"
alpha[13,2] = "8"
alpha[16,2] = "J"
alpha[15,2] = "H"
alpha[18,2] = "6"
alpha[17,2] = "F"
alpha[20,2] = "D"
alpha[19,2] = "S"
alpha[22,2] = "1"
alpha[21,2] = "P"
alpha[24,2] = "O"
alpha[23,2] = "I"
alpha[26,2] = "U"
alpha[25,2] = "Y"
alpha[28,2] = "E"
alpha[27,2] = "Q"
alpha[30,2] = "R"
alpha[29,2] = "T"
alpha[32,2] = "W"
alpha[31,2] = "A"
alpha[34,2] = "G"
alpha[33,2] = "\]"
alpha[36,2] = "K"
alpha[35,2] = "N"

tser = ""

//对从注册表中读出的注册码进行解密
for i = 1 to 16
    tmpchr = mid(reg_code,i,1)
    for j = 1 to 36
        if tmpchr = alpha[j,2] then//查表，变换，第 2 列变为第 1 列，若是想作注册机，需要在此处进行逆运算，1 列变 2 列。
            tmpchr1 = alpha[j,1]
            tser = tser + tmpchr1
        end if
    next
next

ls_disk1 = ""
for i = 1 to 15 step 2//取变换后的注册码的奇数位字符拼接成串
    ls_disk1 = ls_disk1 + mid(tser,i,1)
next

if ls_disk1 <> ls_disk then//与硬盘序列号比较
    return -1
else
    return 0//相等，注册成功
end if
end function
```

```
public function string f_encode (string source_str);ulong lu_ll
char lc_char
string ls_rt_str
integer i
integer li_char

for i = 1 to len(trim(source_str))
    lc_char = mid(source_str,i,1)
    li_char = asc(lc_char)
    lu_ll = of_bitwisexor(li_char,100)
    ls_rt_str = ls_rt_str + char(lu_ll)
next
return ls_rt_str
end function
```

```
public function string f_genpuzzle ();string alpha[36]
string alser
integer i
integer ss

alser = ""
alpha[1] = "A"
```



## PowerBuilder 注册机核心逻辑

以下代码展示了基于硬盘序列号的注册码生成逻辑。

```powerbuilder
alpha[2] = "B"
alpha[3] = "C"
alpha[4] = "D"
alpha[5] = "E"
alpha[6] = "F"
alpha[7] = "G"
alpha[8] = "H"
alpha[9] = "I"
alpha[10] = "J"
alpha[11] = "K"
alpha[12] = "L"
alpha[13] = "M"
alpha[14] = "N"
alpha[15] = "O"
alpha[16] = "P"
alpha[17] = "Q"
alpha[18] = "R"
alpha[19] = "S"
alpha[20] = "T"
alpha[21] = "U"
alpha[22] = "V"
alpha[23] = "W"
alpha[24] = "X"
alpha[25] = "Y"
alpha[26] = "Z"
alpha[27] = "0"
alpha[28] = "1"
alpha[29] = "2"
alpha[30] = "3"
alpha[31] = "4"
alpha[32] = "5"
alpha[33] = "6"
alpha[34] = "7"
alpha[35] = "8"
alpha[36] = "9"

randomize(0)

for i = 1 to 8
    ss = rand(36)
    alser = alser + alpha[ss]
next

return alser
end function
```

### 生成序列号函数 `f_genser`

```powerbuilder
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
tser = left(ls_mhser, 4) + left(alser, 4) + right(ls_mhser, 4) + right(alser, 4)
return tser
end function
```

### 获取硬盘序列号函数 `f_getdisk`

```powerbuilder
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

sdrv = "C:\\"
str = space(256)
str2 = space(256)

// 调用 API，获取硬盘序列号
ll_retval = getvolumeinformationa(sdrv, str, 256, retval, a, b, str2, 256)

// 转换为 16 进制串
ls_retval = ln_1.of_hex(retval)

if len(ls_retval) > 4 then
    li_len = len(ls_retval) - 4
    // 若不足 8 位，在串前补 0
    ls_retval = upper(left(ls_format, 4 - li_len) + left(ls_retval, li_len) + right(ls_retval, 4))
end if

return ls_retval
end function
```

### 注册表读取与写入

```powerbuilder
public function string f_redreg (string xsxt_key);
string ls_value
string ls_path
integer li_flag

ls_path = "HKEY_CURRENT_USER\Software\xsxt\xsxt_" + xsxt_key
li_flag = registryget(ls_path, xsxt_key, regstring!, ls_value)

if li_flag = 1 then
    return ls_value
else
    return "err"
end if
end function

public function integer f_setreg (string str_num, string str_info);
string ls_path
integer li_flag

ls_path = "HKEY_CURRENT_USER\Software\xsxt\xsxt_" + str_num
li_flag = registryset(ls_path, str_num, regstring!, str_info)

if li_flag = 1 then
    return 0
else
    return -1
end if
end function
```

### 字符替换函数 `f_swap1`

```powerbuilder
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
    tmpchr = mid(alser, i, 1)
    for j = 1 to 36
        if tmpchr = alpha[j,1] then
            tmpchr1 = alpha[j,2]
            tser = tser + tmpchr1
        end if
    next
next

return tser
end function
```

### 字符串补零函数 `uf_patchstr`

```powerbuilder
public function string uf_patchstr (string userstr, integer userlen);
integer i
integer strlen

userstr = trim(userstr)

if len(userstr) > userlen then
    messagebox("错误", "长度错误,无法转换")
    return "error"
end if

strlen = userlen - len(userstr)

for i = 1 to strlen
    userstr = "0" + userstr
next

return userstr
end function
```

### 对象事件处理

```powerbuilder
on user_reg.create
    call super::create;
end on

on user_reg.destroy
    call super::destroy;
end on
```

## 注册算法逻辑总结

在上面的代码中，关键部分作为注释，现总结如下：

对注册起作用的是 `f_checkregcode` 和 `f_getdisk`。

注册码算法为：

1. 注册码为 `16` 位长度的串，为大写。
2. 取出硬盘序列号后，若为 `8` 位 `16` 进制串，则不变，否则在前补 `0`，拼成 `8` 位串。
3. 对所得的硬盘序列号串，按 `f_checkregcode` 中的明密文转换表，进行变换，变换的结果，分别记入注册表串的奇数位，即 `1`, `3`, `5`, `7`, `9`, `11`, `13`, `15` 的位置。
4. 注册码偶数位的字符任意，只要是明密文表中能找到可以了。我在注册机中指定为‘`A`’。

## 注册机 Delphi 源码

```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls, StrUtils;

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
  GetVolumeInformation(PChar('c:'), nil, MAX_PATH + 1, @serialnumber, comlen, flag, nil, MAX_PATH + 1);

  str := IntToHex(serialnumber, 4);
  len := Length(str);

  if (len > 4) then
  begin
    len := len - 4;
    // ls_retval = upper(left(ls_format,4 - li_len) + left(ls_retval,li_len) + right(ls_retval,4))
    str := LeftStr('0000', 4 - len) + LeftStr(str, len) + RightStr(str, 4);
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
end;
end.
```



# 变量初始化

`alpha[22][1] := 'V';`  
`alpha[23][1] := 'W';`  
`alpha[24][1] := 'X';`  
`alpha[25][1] := 'Y';`  
`alpha[26][1] := 'Z';`  
`alpha[27][1] := '0';`  
`alpha[28][1] := '1';`  
`alpha[29][1] := '2';`  
`alpha[30][1] := '3';`  
`alpha[31][1] := '4';`  
`alpha[32][1] := '5';`  
`alpha[33][1] := '6';`  
`alpha[34][1] := '7';`  
`alpha[35][1] := '8';`  
`alpha[36][1] := '9';`  
`alpha[2][2] := 'Z';`  
`alpha[1][2] := 'X';`  
`alpha[4][2] := 'C';`  
`alpha[3][2] := 'V';`  
`alpha[6][2] := 'B';`  
`alpha[5][2] := '9';`  
`alpha[8][2] := 'M';`  
`alpha[7][2] := ',';`  
`alpha[10][2] := '.';`  
`alpha[9][2] := '/';`  
`alpha[12][2] := '7';`  
`alpha[11][2] := ';';`  
`alpha[14][2] := 'L';`  
`alpha[13][2] := '8';`  
`alpha[16][2] := 'J';`  
`alpha[15][2] := 'H';`  
`alpha[18][2] := '6';`  
`alpha[17][2] := 'F';`  
`alpha[20][2] := 'D';`  
`alpha[19][2] := 'S';`  
`alpha[22][2] := '1';`  
`alpha[21][2] := 'P';`  
`alpha[24][2] := 'O';`  
`alpha[23][2] := 'I';`  
`alpha[26][2] := 'U';`  
`alpha[25][2] := 'Y';`  
`alpha[28][2] := 'E';`  
`alpha[27][2] := 'Q';`  
`alpha[30][2] := 'R';`  
`alpha[29][2] := 'T';`  
`alpha[32][2] := 'W';`  
`alpha[31][2] := 'A';`  
`alpha[34][2] := 'G';`  
`alpha[33][2] := ']';`  
`alpha[35][2] := 'N';`  
`alpha[36][2] := 'K';`

# 解码逻辑

```
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
        break;
      end;
    end;
    inc(j);
  end
  else
  begin
    enstr[i] := 'A';
  end;
end;
edit1.Text:=enstr;
end;
end.
```

--------------------------------------------------------------------------------

## 经验总结

这次破解主要在笨鸟我先飞兄弟的基础上做的，他把 `PBKILLER` 反编译后的代码给了我，我只是阅读了 `PB` 的源码，`PB` 脚本与 `BASIC` 语法类似，兄弟们如果破 `PB`，可以参照 `BASIC` 语法来阅读源码。

`PBKILLER` 真的很强，反编译出来的代码可读性很好，呵呵。

--------------------------------------------------------------------------------

## 版权声明

请保持文章完整及作者信息。

2006 年 09 月 04 8:30:32

`4dd466d0e1e0cc123afbe504da2462ca`
