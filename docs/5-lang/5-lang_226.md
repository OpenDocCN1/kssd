
# 某报价预算软件(PB9 程序) Ver 5.1 注册验证分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-377.htm`](https://www.pediy.com/kssd/pediy07/pediy7-377.htm)

某报价预算软件(PB9 程序) Ver 5.1 注册验证分析(一)

日期：2005 年 9 月 6 日　　破解人：Baby2008

『软件名称』：某报价预算软件 成套版 Ver 5.1
『软件大小』：34.5MB
『下载地址』：商业软件,不便提供,看懂得自己搜索
『软件介绍』：
『保护方式』：注册文件+注册码保护
『破解声明』：初学 Crack，只是感兴趣，个人感觉 PB 程序分析破文不多，由我来抛砖引玉吧
『破解工具』：`OllyDbg.V1.10`，`PBKiller` (感谢作者:kivens)
『破解过程』：

## 一、安装序列号破解

安装时需要软件安装序列号，那先分析一个软件安装序列号；

运行安装程序，到达序列号输入界面，由于安装包是由 `Setup Factory 7.0` 制作的，安装程序会在安装过程中会产生一个临时进程来完成安装，

破解安装序列号需要切换到临时进程，启动 `fllyODBG`，附加进程：

```
未命名的窗口，项目 35

进程=00000FC4
名称=irsetup
窗口=产品序列号
路径=C:\DOCUME~1\wxb\LOCALS~1\Temp\irsetup.exe   //安装程序临时进程
```

设置 `Point H(万能断点)`，输入安装序列号 `1234567890`，下一步，`OD` 中断并返回主流程:

:

```
0042E1DC     FF15 4C144400        call dword ptr ds:[<&USER32.GetWindowTextA>]    ; USER32.GetWindowTextA
0042E1E2     6A FF               push -1                                                 ; 返回这里
0042E1E4     8BCF               mov ecx,edi
0042E1E6     E8 BA2D0000        call irsetup.00430FA5
0042E1EB     EB 13              jmp short irsetup.0042E200
0042E1ED     8BCE               mov ecx,esi
0042E1EF     E8 CA1E0000        call irsetup.004300BE
0042E1F4     85C0               test eax,eax
0042E1F6     74 08              je short irsetup.0042E200
0042E1F8     57                 push edi
0042E1F9     8BC8               mov ecx,eax
0042E1FB     E8 52FFFFFF        call irsetup.0042E152
0042E200     8B07               mov eax,dword ptr ds:[edi]                               ; 试炼码
0042E202     5F                 pop edi
0042E203     5E                 pop esi
0042E204     8B40 F8            mov eax,dword ptr ds:[eax-8]
0042E207     C2 0800            retn 8
………(省略代码 N 行)…………………
00415607     E8 53B60100        call irsetup.00430C5F
0041560C     8065 FC 00         and byte ptr ss:[ebp-4],0
00415610     8D4D E8            lea ecx,dword ptr ss:[ebp-18]
00415613     E8 5AB50100        call irsetup.00430B72
00415618     FF75 08            push dword ptr ss:[ebp+8]
0041561B     FF75 F0            push dword ptr ss:[ebp-10]
0041561E     E8 A2CA0000        call irsetup.004220C5
```

堆栈信息:

```
0012F510   00CBA9E8  ASCII "1234567890"
0012F514   00AF20F0  ASCII "X2D"
0012F518   00AF20F0  ASCII "X2D"
0012F51C   0012F50C
0012F520   00CAF648  ASCII "9896-7818-3668-5118"
0012F524   00000000
0012F528   00CAF648  ASCII "9896-7818-3668-5118"     //安装序列号
```

得到安装序列号:`9896-7818-3668-5118`  顺利完成安装过程。

## 二、注册信息文件分析

程序为 `PB9` 编写，`OD` 调试会在“PB 解释器”里打转，还是用 PB 克星 `PBKiller` (再次感谢作者:kivens,`DePB` 公开版本不支持 `PB9`)反编译各事件代

码，源代码可读性非常的好，下面对各重要代码稍作解释：

注册界面生成注册信息按钮事件：

```
string ls_gsmc
string ls_gsdz
string ls_tel
string ls_fax
string ls_yb
string ls_lxr
string ls_zcm
string ls_zclm
string ls_ver
string ls_email
integer li_int_1
date ldt_zcrq
string ls_write_cpu
string ls_write_hard
string ls_write_macf
string ls_write_regpass
string ls_write_gsmc
string ls_jq
string ls_zcmc
string ls_yhzt
string ls_dqcs
string ls_bzcs
string ls_dqsj
string ls_zcsj
string ls_dlsj
date ldt_xtsj
long ll_zcsj
long ll_dqsj
long ll_dlsj
long ll_bzcs
long ll_dqcs
string ls_write_dqcs
string ls_write_bzcs
string ls_write_dlsj
string ls_write_zcsj
string ls_write_yhzt
string ls_write_dqsj
string ls_maxuser
string ls_shengji
long ll_cpu
string ls_rootpath
string ls_volumnename
ulong lul_volumenamesize
ulong lul_volumeserialnumber
ulong lul_maximumcomponentlength
ulong lul_filesystemflags
string ls_filesystemnamebuffer
ulong lul_filesystemnamesize
boolean lb_rtn = false
string ls_regpass
integer li_i
string li_j
string ls_docname
string ls_named
string ls_typename
integer li_value
integer ll_file
select int_1 , zclm , updateid , dwmc , ver from update_rq  using sqlca;

/* SQL Parameters List
0-> :li_int_1
1-> :ls_zclm
2-> :ls_zcm
3-> :ls_gsmc
4-> :ls_ver
*/
```

//取得数据库内容为:

```
1,'N',' ',' ','MorrowBudget Ver5.1 '
```

```
if li_int_1 < 1 then
  li_int_1 = 1           //重要，指续费次数，后面会用到
end if

if ls_zclm = "N" and (len(ls_zcm) < 1 or isnull(ls_zcm)) then
  ls_zcm = parent.wf_xlh()                             //软件序列号，相当于软件 ID，升级 ID
  update update_rq set updateid =' '  using sqlca;       //保存 ID
  /* SQL Parameters List
  0-> :ls_zcm
  */
  commit using sqlca;
  parent.cb_2.enabled = false
else
  if (ls_zclm = "" or len(ls_zclm) < 1) and (len(ls_zcm) < 1 or isnull(ls_zcm)) then
    ls_zcm = parent.wf_xlh()
    update update_rq set updateid =' '  using sqlca;
    /* SQL Parameters List
    0-> :ls_zcm
    */
    commit using sqlca;
    parent.cb_2.enabled = false
  else
    if ls_zclm = "Y" then
      messagebox("注册提示","本软件已经注册,注册公司: " + ls_gsmc)
      return
    end if
  end if
end if

ls_gsmc = trim(parent.sle_1.text)                     //读取界面空间内容
parent.is_gsmc = f_get_spellcn(ls_gsmc)
ls_gsdz = trim(parent.sle_2.text)
ls_tel = trim(parent.sle_4.text)
ls_fax = trim(parent.sle_5.text)
ls_yb = trim(parent.sle_6.text)
ls_lxr = trim(parent.sle_3.text)
ls_email = trim(parent.sle_7.text)

update update_rq set gsdz =' ' , lxr =' ' , tel =' '  using sqlca;     //保存
/* SQL Parameters List
0-> :ls_gsdz
1-> :ls_lxr
2-> :ls_tel
*/

if len(ls_gsmc) < 1 or isnull(ls_gsmc) then                              //开始判断合法性
  messagebox("提示","公司名称必须为法定名称，重新命名!")
  return
end if
if len(ls_gsdz) < 1 or isnull(ls_gsdz) then
  messagebox("提示","公司地址必须为法定地址，重新命名!")
  return
end if
if len(ls_tel) < 6 or isnull(ls_tel) then
  messagebox("提示","电话号码必须真实，重新命名!")
  return
end if
if len(ls_fax) < 6 or isnull(ls_fax) then
  messagebox("提示","传真号码必须真实，重新命名!")
  return
end if
if len(ls_yb) <> 6 then
  messagebox("提示","邮政编码必须真实，重新命名!")
  return
end if
if len(ls_lxr) < 0 or isnull(ls_lxr) then
  messagebox("提示","联系人必须真实，重新命名!")
  return
end if

ldt_xtsj = date(today())     //系统时间

select dqcs , yhzt , dqsj , zcsj , dlsj , bzcs from lcs  using sqlca;      //取数据
/* SQL Parameters List
0-> :ls_dqcs
1-> :ls_yhzt
2-> :ls_dqsj
3-> :ls_zcsj
4-> :ls_dlsj
5-> :ls_bzcs
*/

//取得的数据为,(经过加密)
dqcs,yhzt,dqsj,zcsj,dlsj,bzcs
'a1):K',' ','QcR=opaBoQWaH<t?a2ZvH-64#_cZX%L-p2rEb>?*\\aMoG','YcmwL%aZy{+aUsrba%lw)->9@&c+5Rc-eR]AbW}x2a\\cf','$2
\\5>,0.Ypa0aQ5859iHU-D({b8^<\\a-@>#J3bSv510g^','w3|wu-1M3G'

select maxuser from c_maxuser  using sqlca;      //最大用户数,结果为空,单机版没用到
/* SQL Parameters List
0-> :ls_maxuser
*/

ls_write_yhzt = ls_yhzt             //为写注册信息文件准备变量
ls_write_dqsj = ls_dqsj
ls_write_zcsj = ls_zcsj
ls_write_dlsj = ls_dlsj
ls_write_bzcs = ls_bzcs
ls_write_dqcs = ls_dqcs

ls_maxuser = parent.wf_jiami(ls_maxuser)     //加密 wf_jiami
ls_shengji = string(li_int_1) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))
ls_shengji = parent.wf_numtochar(ls_shengji)   //数字转字符 wf_numtochar
ls_shengji = parent.wf_jiami(ls_shengji)

ll_cpu = abs(getcpuid())                 //CPU ID
if ll_cpu > 0 then     //CPU ID 获取失败用随机数代替
  ls_write_cpu = string(ll_cpu)
else
  ls_write_cpu = "77" + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))
end if

ls_rootpath = "C:"
```



```
ls_volumnename = space(256)                //取 C:\ 磁盘序列号  

lul_volumenamesize = 256  

lul_maximumcomponentlength = 256  

ls_filesystemnamebuffer = space(256)  

lul_filesystemnamesize = 256  

beep(1)  

lb_rtn = false  

lb_rtn = getvolumeinformationa  
(ls_rootpath, ls_volumnename, lul_volumenamesize, lul_volumeserialnumber, lul_maximumcomponentlength, lul_filesystemflags, ls_filesystemnamebuffer, lul_filesystemnamesize)  

if lb_rtn = true then    //取 C:\ 磁盘序列号用随机数代替  
  ls_write_hard = trim(string(lul_volumeserialnumber))  
else  
  ls_write_hard = "68" + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))  
end if  

ls_write_macf = f_mac()        //网卡序列号  

if ls_write_macf = "" or isnull(ls_write_macf) then   //失败用随机数代替  
  ls_write_macf = "37" + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))  
end if  

select regpass from c_gsxx  using sqlca;  

/* SQL Parameters List  
0-> :ls_regpass  
*/  

if len(ls_regpass) < 1 or isnull(ls_regpass) then      //变量用途未知，估计是升级口令之类的  
  ls_write_regpass = parent.wf_round() + parent.wf_round() + parent.wf_round()  
end if  

ls_write_cpu = parent.wf_numtochar(ls_write_cpu)      //替换  
ls_write_hard = parent.wf_numtochar(ls_write_hard)  
ls_write_macf = parent.wf_numtochar(ls_write_macf)  
ls_write_regpass = parent.wf_numtochar(ls_write_regpass)  

ls_write_cpu = parent.wf_jiami(ls_write_cpu)           //加密  
ls_write_hard = parent.wf_jiami(ls_write_hard)  
ls_write_macf = parent.wf_jiami(ls_write_macf)  
ls_write_regpass = parent.wf_jiami(ls_write_regpass)  

ls_jq = parent.wf_toserial() + "#" + string(year(today())) + string(month(today())) + string(day(today()))  

ls_zcmc = "zhuce" + ls_jq + "good"  

update c_gsxx set zcmc = ' '  using sqlca;     //重要，保存信息用于后面判断注册文件的合法性 wf_toserial() 取得 Windows 安装时的用户名称和公司名称  

/* SQL Parameters List  
0-> :ls_zcmc  
*/  

ls_jq = parent.wf_jiami(ls_jq)  
ls_zcmc = ls_jq  

for li_i = 1 to 50  
  li_j = parent.wf_round()  
next  

ls_named = ls_ver + ls_gsmc    //注册信息文件名  
ls_docname = ls_named  

li_value = getfilesavename("保存文件", ls_docname, ls_named, "sps", "sps Files (\*.sps),\*.sps")  

if li_value = 1 then  
  if fileexists(ls_docname) then  
    if messagebox("操作提示", ls_docname + "文件已经存在,是否覆盖它", question!, yesno!) = 2 then  
      return  
    end if  
  end if  
  ll_file = fileopen(ls_docname, linemode!, write!, lockreadwrite!, replace!)  
  if ll_file = -1 then  
    messagebox("操作提示", "文件打开操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_ver) = -1 then                        //软件版本  
    messagebox("操作提示", "文件写操作失败")                      
    return  
  end if  
  if filewrite(ll_file, ls_gsmc) = -1 then                        //公司名称  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_gsdz) = -1 then                        //公司地址  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_tel) = -1 then                         //电话  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_fax) = -1 then                         //传真  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_yb) = -1 then                          //邮编  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_lxr) = -1 then                         //联系人  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_email) = -1 then                       //邮件  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_zcm) = -1 then                         //软件注册码=Update ID  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_shengji) = -1 then                     //1 +随机数  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_cpu) = -1 then                   //CPU ID  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_hard) = -1 then                  //C:\ SerialNo  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_macf) = -1 then                  //网卡 ID  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_regpass) = -1 then               //随机数  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_zcmc) = -1 then                        // wf_jiami(ls_regowner + "**" + ls_regcompany #日期)  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_yhzt) = -1 then                  //用户状态  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_dqsj) = -1 then  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_zcsj) = -1 then  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_dlsj) = -1 then  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_bzcs) = -1 then  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_write_dqcs) = -1 then  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
  if filewrite(ll_file, ls_maxuser) = -1 then                     //最大用户数  
    messagebox("操作提示", "文件写操作失败")  
    return  
  end if  
else  
  messagebox("操作提示", "文件命名不合法，重新命名！")  
  return  
end if  

fileclose(ll_file)  

messagebox("系统写文件成功", "报价单数据包保存在~r~n" + ls_docname + "~r~n 请将以上文件发送到指定的邮箱中")  

parent.cb_2.enabled = false  

close(parent)  

return
```

涉及重要函数:`wf_numtochar`,`wf_jiami`，分析一下:

```
--------------------------------------------------------------------------
Function wf_jiami     (加密)
--------------------------------------------------------------------------
string ls_source
string ls_parm
string ls_arry[91]
string ls_mima[]
string ls_hunxiao[3]
integer li_i
integer li_j
integer li_long

li_long = len(ls_parmstring)
for li_i = 1 to li_long
  ls_mima[li_i] = mid(ls_parmstring, li_i, 1)
next

ls_source = "0132456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{},./?[]\\|+=_-)(\*&^%$#@!\`:<>"
for li_i = 1 to 91
  ls_arry[li_i] = right(left(ls_source, li_i), li_i - (li_i - 1))
next

for li_i = 1 to li_long
  ls_parm = ls_parm + ls_arry[rand(91)] + ls_mima[li_i] + ls_arry[rand(91)] + ls_arry[rand(91)] + ls_arry[rand(91)] + ls_arry[rand(91)]
next

return ls_parm
```

本人不会 PB，只好将代码翻译为 Delphi 7.0 的代码，为做注册机做准备(下同):

Delphi 7.0 实现如下：

```
Function wf_jiami(Parm: String): String;

Const
  ls_source: Array[0..90] Of Char = '0132456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{},./?[]\\|+=_-)(\*&^%$#@!\`:<>';

Var
  i: Integer;

Begin
  For i := 1 To Length(Parm) Do
    Result := Result + ls_source[Random(90)] + Parm[i] + ls_source[Random(90)] + ls_source[Random(90)] + ls_source[Random(90)];
End;
```

对应反函数:

```
--------------------------------------------------------------------------
Function wf_jiemi  (解密)
--------------------------------------------------------------------------
string ls_source
string ls_parm
string ls_mima[]
string ls_hunxiao[3]
integer li_i
integer li_j
integer li_long

li_long = len(ls_parmstring)
for li_i = 1 to li_long
  ls_mima[li_i] = mid(ls_parmstring, li_i, 1)
next

for li_i = 2 to li_long step 5
  ls_parm = ls_parm + ls_mima[li_i]
next

return ls_parm
```

Delphi 7.0 实现如下：

```
Function wf_jiemi(Parm: String): String;

Var
```



# 三、注册授权文件分析

## 字符替换函数分析

### 代码实现：数值转字符

```
i: Integer;  

Begin  

  i := 2;  

  While i <= Length(Parm) Do  

  Begin  

    Result := Result + Parm[i];  

    i := i + 5;  

  End;  

End;  
```

另有字符替换函数：`wf_numtochar`

```
--------------------------------------------------------------------------

Function wf_numtochar  

--------------------------------------------------------------------------  

string ls_mima[]  

string ls_parming  

integer li_i  

integer li_len  

li_len = len(ls_parm)  

for li_i = 1 to li_len  

  ls_mima[li_i] = mid(ls_parm,li_i,1)  

  if ls_mima[li_i] = "0" then  

    ls_mima[li_i] = "a"  

  end if  

  if ls_mima[li_i] = "1" then  

    ls_mima[li_i] = "b"  

  end if  

  if ls_mima[li_i] = "2" then  

    ls_mima[li_i] = "c"  

  end if  

  if ls_mima[li_i] = "3" then  

    ls_mima[li_i] = "d"  

  end if  

  if ls_mima[li_i] = "4" then  

    ls_mima[li_i] = "e"  

  end if  

  if ls_mima[li_i] = "5" then  

    ls_mima[li_i] = "f"  

  end if  

  if ls_mima[li_i] = "6" then  

    ls_mima[li_i] = "g"  

  end if  

  if ls_mima[li_i] = "7" then  

    ls_mima[li_i] = "h"  

  end if  

  if ls_mima[li_i] = "8" then  

    ls_mima[li_i] = "i"  

  end if  

  if ls_mima[li_i] = "9" then  

    ls_mima[li_i] = "j"  

  end if  

next  

for li_i = 1 to li_len  

  ls_parming = ls_parming + ls_mima[li_i]  

next  

return ls_parming  
```

`Delphi 7.0` 实现如下：`Function wf_numtochar(Parm: String): String;`

```
Function wf_numtochar(Parm: String): String;  

Var  

  i: Integer;  

Begin  

  Result := Parm;  

  For i := 1 To Length(Parm) Do  

    If Parm[i] In ['0'..'9'] Then Result[i] := Char(Ord(Parm[i]) + $31);  

End;  
```

### 代码实现：字符转数值

`wf_chartonum`

```
--------------------------------------------------------------------------

Function wf_chartonum  

--------------------------------------------------------------------------  

string ls_mima[]  

string ls_parming  

integer li_i  

integer li_len  

li_len = len(ls_parm)  

for li_i = 1 to li_len  

  ls_mima[li_i] = mid(ls_parm,li_i,1)  

  if ls_mima[li_i] = "a" then  

    ls_mima[li_i] = "0"  

  end if  

  if ls_mima[li_i] = "b" then  

    ls_mima[li_i] = "1"  

  end if  

  if ls_mima[li_i] = "c" then  

    ls_mima[li_i] = "2"  

  end if  

  if ls_mima[li_i] = "d" then  

    ls_mima[li_i] = "3"  

  end if  

  if ls_mima[li_i] = "e" then  

    ls_mima[li_i] = "4"  

  end if  

  if ls_mima[li_i] = "f" then  

    ls_mima[li_i] = "5"  

  end if  

  if ls_mima[li_i] = "g" then  

    ls_mima[li_i] = "6"  

  end if  

  if ls_mima[li_i] = "h" then  

    ls_mima[li_i] = "7"  

  end if  

  if ls_mima[li_i] = "i" then  

    ls_mima[li_i] = "8"  

  end if  

  if ls_mima[li_i] = "j" then  

    ls_mima[li_i] = "9"  

  end if  

next  

for li_i = 1 to li_len  

  ls_parming = ls_parming + ls_mima[li_i]  

next  

return ls_parming  
```

`Delphi 7.0` 实现如下：`Function wf_chartonum(Parm: String): String;`

```
Function wf_chartonum(Parm: String): String;  

Var  

  i: Integer;  

Begin  

  Result := Parm;  

  For i := 1 To Length(Parm) Do  

    If Parm[i] In ['a'..'j'] Then Result[i] := Char(Ord(Parm[i]) - $31);  

End;  
```

至此，注册信息产生过程已经搞清楚了,产生的文件信息后面还可以再利用，分析一下很有必要。

## 注册按钮事件分析

软件作者根据注册信息文件返回注册授权文件，看一下“注册”按钮对注册授权文件有什么要求:

```
string ls_file  

string ls_docname  

string ls_named  

string ls_typename  

string ls_ver  

integer li_value  

string ls_gsmc  

string ls_hard  

string ls_cpu  

string ls_maxuser  

string ls_macf  

string ls_regpass  

long ll_file  

string ls_regcompany  

string ls_regowner  

string ls_jq  

string ls_jqmatch  

string ls_zcm  

string ls_yhzt  

string ls_dqcs  

string ls_bzcs  

string ls_dqsj  

string ls_zcsj  

string ls_dlsj  

string ls_zclm  

string ls_dlsjold  

string ls_dlsjnew  

string ls_zcm1  

date ldt_dlsjold  

date ldt_dlsjnew  

long ll_match  

string ls_docname1  

string ls_path  

integer li_ren  

boolean rtn = false  

string ls_yhzt1  

integer li_max  

li_value = getfileopenname("保存文件",ls_docname,ls_named,"PAS","Pas Files (*.PAS),*.pas")  

if li_value = 1 then  

  ls_file = ls_docname  

end if  

if not fileexists(ls_file) then  

  messagebox("操作提示","该文件不存在或者您并没有输入文件的位置")  

  return  

end if  

ll_file = fileopen(ls_file,linemode!,read!,lockreadwrite!,replace!)               //开始读取注册文件  

if ll_file = -1 then  

  messagebox("操作提示","文件打开操作失败，确定文件的格式为.PAS")  

  return  

end if  

if fileread(ll_file,ls_ver) = -1 then  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(1))   //软件版本  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_zcm) = -1 then  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(1))   //软件注册码 Update ID  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_gsmc) = -1 then                                         //公司名称  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(1))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_hard) = -1 then         //硬盘序列号  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(2))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_cpu) = -1 then         //CPU ID  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(3))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_macf) = -1 then        //网卡 ID  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(4))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_regpass) = -1 then               //随机数?  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(5))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_jqmatch) = -1 then        //重要,判断是否是非法注册文件  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(7))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_maxuser) = -1 then  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_yhzt) = -1 then             //用户状态  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_dqcs) = -1 then           
  
  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_bzcs) = -1 then  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_dqsj) = -1 then            //到期时间  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_zcsj) = -1 then            //注册时间  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

if fileread(ll_file,ls_dlsj) = -1 then            //最后登入时间  

  messagebox("操作提示","文件读操作失败，确定文件的格式为.PAS" + "/" + string(6))  

  fileclose(ll_file)  

  return  

end if  

parent.is_gsmc = f_get_spellcn(ls_gsmc)      //公司名称－>转汉字拼音首字母  

if len(ls_maxuser) > 10 then             //最大用户数，单机版不用  

  ls_maxuser = mid(ls_maxuser,9,len(ls_maxuser) - 16)     
  
else  

  ls_maxuser = ""  

end if  

select updateid , zclm from update_rq  using sqlca;  

/* SQL Parameters List  

0-> :ls_zcm1  

1-> :ls_zclm  

*/  

ls_dlsjnew = ls_dlsj                   //登入时间  

ls_dlsjnew = parent.wf_jiemi(ls_dlsjnew)  

ls_dlsjnew = parent.wf_chartonum(ls_dlsjnew)   //字母替换  

select dlsj from lcs  using sqlca;  

/* SQL Parameters List  

0-> :ls_dlsjold  

*/  

ls_dlsjold = parent.wf_jiemi(ls_dlsjold)  

ls_dlsjold = parent.wf_chartonum(ls_dlsjold)  

ldt_dlsjold = date(ls_dlsjold)  
```



```
ldt_dlsjnew = date(ls_dlsjnew)  

ll_match = daysafter(ldt_dlsjold, ldt_dlsjnew)     

if ls_zcm <> ls_zcm1 then  

  messagebox("操作提示", "注册文件过期，请与慕龙公司联系")  

  halt  

end if  

if ls_zclm = "Y" then    //是否已经注册标志  

  messagebox("操作提示", "本软件已经注册过了, 不能重复注册")  

  halt  

end if  

select zcmc from c_gsxx using sqlca;      //注册名称  

/* SQL Parameters List  

0-> :ls_jq  

*/  

ls_jqmatch = parent.wf_jiemi(ls_jqmatch)  

ls_jqmatch = "zhuce" + ls_jqmatch + "good"             //重要标志，在产生文件时强调过  

if not ls_jqmatch = ls_jq then  

  messagebox("注册错误！", "非法注册！")  

  halt  

end if  

ls_hard = parent.wf_jiemi(ls_hard)    //解密 C: 盘序列号  

ls_cpu = parent.wf_jiemi(ls_cpu)      //CPU ID  

li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion", "PathName", regstring!, ls_path)  

if li_ren = -1 then  

  li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows me\\CurrentVersion", "PathName", regstring!, ls_path)  

end if  

if li_ren = -1 then  

  li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion", "PathName", regstring!, ls_path)  

end if  

if li_ren = -1 then  

  li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows 98\\CurrentVersion", "PathName", regstring!, ls_path)  

end if  

if li_ren = -1 then  

  ls_path = "c:"  

end if  

rtn = fileexists(ls_path + "\\mrbdt.ini")      //内容为注册名称和序列号  

if not rtn then  

  ll_file = filecopy(parent.is_file + "\\mlpath.txt", ls_path + "\\mrbdt.ini")  

  fileclose(ll_file)  

  ls_docname1 = ls_path + "\\mrbdt.ini"  

  ll_file = fileopen(ls_docname1, linemode!, write!, lockwrite!, replace!)  

  if ll_file = -1 then  

    messagebox("操作提示", "注册失败，异常号：001")  

    fileclose(ll_file)  

    close(parent)  

    return  

  end if  

  if filewrite(ll_file, ls_gsmc) = -1 then  

    messagebox("操作提示", "注册失败，异常号：002")  

    fileclose(ll_file)  

    close(parent)  

    return  

  end if  

  if filewrite(ll_file, ls_zcm) = -1 then  

    messagebox("操作提示", "注册失败，异常号：003")  

    fileclose(ll_file)  

    close(parent)  

    return  

  end if  

  fileclose(ll_file)  

end if  

//分系统版本取 windows 安装时公司名称  

li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows  

NT\\CurrentVersion\\Winlogon", "DefaultDomainName", regstring!, ls_regcompany)  

if li_ren = -1 then  

  registryget  

("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Winlogon", "DefaultDomainName", regstring!, ls_regcompany)  

end if  

if li_ren = -1 then  

  registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows  

ME\\CurrentVersion\\Winlogon", "DefaultDomainName", regstring!, ls_regcompany)  

end if  

if li_ren = -1 then  

  registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows 98  

\\CurrentVersion\\Winlogon", "DefaultDomainName", regstring!, ls_regcompany)  

end if  

//分系统版本取机器名称  

li_ren = registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows  

NT\\CurrentVersion", "RegisteredOwner", regstring!, ls_regowner)  

if li_ren = -1 then  

  registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion", "RegisteredOwner", regstring!, ls_regowner)  

end if  

if li_ren = -1 then  

  registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows  

ME\\CurrentVersion", "RegisteredOwner", regstring!, ls_regowner)  

end if  

if li_ren = -1 then  

  registryget("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows 98  

\\CurrentVersion", "RegisteredOwner", regstring!, ls_regowner)  

end if  

if isnull(ls_regowner) then  

  ls_regowner = parent.is_gsmc   //假如空，用软件注册时的公司名称代替  

end if  

if isnull(ls_regcompany) then                //同上  

  ls_regcompany = parent.is_gsmc  

end if  

ls_regcompany = ls_regcompany + "^^^" + ls_regowner     //更新标志  

ls_yhzt1 = parent.wf_jiemi(ls_yhzt)  

ls_yhzt1 = parent.wf_charntonum(ls_yhzt1)  

if ls_yhzt1 = "2" then        //系统状态，2 为试用版 9 为正式版  

  ls_zclm = "N"  

else  

  ls_zclm = "Y"  

end if  

update c_gsxx set gsmc =' ', zcmc =' '  using sqlca;   //下面更新一下数据库信息  

/* SQL Parameters List  

0-> :ls_gsmc  

1-> :ls_regcompany  

*/  

commit using sqlca;  

update update_rq set dwmc =' ', zclm =' ', updateid =' ', zcrq = {d '2005-08-31' } , updatepass ='8888' , ver =' '  using  

sqlca;  

/* SQL Parameters List  

0-> :ls_gsmc  

1-> :ls_zclm  

2-> :ls_zcm  

3-> :ldt_dlsjnew  

4-> :ls_ver  

*/  

commit using sqlca;  

update lcs set qhkg ='n' , yhzt =' ' , dqcs =' ' , bzcs =' ' , dqsj =' ' , zcsj =' ' , dlsj =' '  using sqlca;  

/* SQL Parameters List  

0-> :ls_yhzt  

1-> :ls_dqcs  

2-> :ls_bzcs  

3-> :ls_dqsj  

4-> :ls_zcsj  

5-> :ls_dlsj  

*/  

commit using sqlca;  

select count ( * ) from c_maxuser using sqlca;  

/* SQL Parameters List  

0-> :li_max  

*/  

if not ls_maxuser = "A" then  

  if li_max > 1 then  

    delete from c_maxuser using sqlca;  

    insert into c_maxuser ( maxuser ) values ( ' ' )  using sqlca;  

/* SQL Parameters List  

0-> :ls_maxuser  

*/  

  else  

    update c_maxuser set maxuser =' '  using sqlca;  

/* SQL Parameters List  

0-> :ls_maxuser  

*/  

  end if  

end if  

update c_gsxx SET gsmc =' ', cpu =' ', hard =' ', macf =' ', regpass =' '  using sqlca;  

/* SQL Parameters List  

0-> :ls_gsmc  

1-> :ls_cpu  

2-> :ls_hard  

3-> :ls_macf  

4-> :ls_regpass  

*/  

if sqlca.sqlcode <> 0 then  

  rollback using sqlca;  

  messagebox("操作提示", "机器注册出错 1")  

  return  

else  

  commit using sqlca;  

  messagebox("祝贺！", "机器注册成功，请登陆并使用！")  

  parent.cb_2.enabled = false  

  openwithparm(w_xufeichenggong, ls_zcm)  

end if  

return  

--------------------------------------------------------------------------  
```

根据注册授权文件要求，结合注册信息文件的内容，用 `Delphi 7` 实现下注册授权文件如下：

```
Procedure TForm1.btn1Click(Sender: TObject); //读取注册信息  

Var  

  Reg: TMemo;  

Begin  

  If dlgOpenRegFile.Execute Then  

  Begin  

    Reg := TMemo.Create(Self);  

    Reg.Parent := Self;  

    Reg.WordWrap := False;  

    Reg.Visible := False;  

    Reg.Lines.LoadFromFile(dlgOpenRegFile.FileName);  

    If Reg.Lines.Count <> 22 Then  

    Begin  

      Application.MessageBox('选择的不是注册文件！', '信息提示', MB_OK + MB_ICONWARNING);  

      Exit;  

    End;  

    With Reg Do  

    Begin  

      edtgsmc.Text := Lines[1];  

      edtgsdz.Text := Lines[2];  

      edtlxr.Text := Lines[6];  

      edtyb.Text := Lines[5];  

      edtTel.Text := Lines[3];  

      edtFax.Text := Lines[4];  

      edtEmail.Text := Lines[7];  

      ls_ver := Lines[0];  

      ls_gsmc := Lines[1];  

      ls_gsdz := Lines[2];  

      ls_tel := Lines[3];  

      ls_fax := Lines[4];  

      ls_yb := Lines[5];  

      ls_lxr := Lines[6];  

      ls_email := Lines[7];  

      ls_zcm := Lines[8];  

      ls_shengji := Lines[9];  

      ls_write_cpu := Lines[10];  

      ls_write_hard := Lines[11];  

      ls_write_macf := Lines[12];  

      ls_write_regpass := Lines[13]; //wf_numtochar(18 个字符)  

      ls_zcmc := Lines[14]; //zhuceLenovo User**Legend (Beijing) Limited#200594good  

      ls_write_yhzt := Lines[15]; //用户状态    9:正常  2:试用  

      ls_write_dqsj := Lines[16]; // 到期时间  

      ls_write_zcsj := Lines[17]; // 注册时间  

      ls_write_dlsj := Lines[18]; //最后登入时间  

      ls_write_bzcs := Lines[19];  

      ls_write_dqcs := Lines[20]; //到期次数  

      ls_maxuser := Lines[21]; //最大用户数  

    End;  

    Reg.Free;  

  End;  

End;  

Procedure TForm1.btn2Click(Sender: TObject);  

Var  

  Reg: TMemo;  

Begin  

  If (edtgsmc.Text = '') Or (edtgsdz.Text = '') Or (edtlxr.Text = '') Or (edtTel.Text = '') Then  

  Begin  

    Application.MessageBox('注册信息不全, 请重新读取注册文件！', '信息提示', MB_OK + MB_ICONWARNING);  

    Exit;  

  End;  

  Reg := TMemo.Create(Self);  
```



### Reg.Parent := Self;

```pascal
Reg.Parent := Self;  
Reg.WordWrap := False;  
Reg.Visible := False;  

With Reg Do //生成注册授权文件  
Begin  
    Lines.Add(ls_ver); //软件版本  
    Lines.Add(ls_zcm); //注册码 Update ID  
    Lines.Add(ls_gsmc); //公司名称  
    Lines.Add(ls_write_hard); //硬盘序列号 C:\  
    Lines.Add(ls_write_cpu); //CPU ID  
    Lines.Add(ls_write_macf); //MAC ID  
    Lines.Add(ls_write_regpass);  
    Lines.Add(ls_zcmc); //重要,判断注册文件是否有效  
    Lines.Add(ls_maxuser); //最大用户数  
    Lines.Add(wf_jiami('9')); //用户状态 9:正常 2:试用版  
    Lines.Add(ls_write_dqcs);  
    Lines.Add(ls_write_bzcs);  
    // Lines.Add(ls_write_dqsj); //到期时间  
    Lines.Add(wf_jiami(FormatDateTime('yyyy-m-d', Date - 330)));  
    // Lines.Add(ls_write_zcsj); //注册时间  
    Lines.Add(wf_jiami(FormatDateTime('yyyy-m-d', Date)));  
    Lines.Add(ls_write_dlsj); //最后登入时间  
End;  

dlgSaveRegFile.FileName := ls_ver + ' ' + ls_gsmc + ' 注册文件';  
If dlgSaveRegFile.Execute Then  
Begin  
    If UpperCase(Copy(dlgSaveRegFile.FileName, Length(dlgSaveRegFile.FileName) - 2, 3)) <> UpperCase('pas') Then  
        dlgSaveRegFile.FileName := dlgSaveRegFile.FileName + '.pas';  
    Try  
        Reg.Lines.SaveToFile(dlgSaveRegFile.FileName);  
    Except  
        Application.MessageBox('生成注册文件失败,请检查目标文件是否可读写。', '信息提示', MB_OK + MB_ICONSTOP);  
    End;  
End;  

Reg.Free;  

End;
```

待续……

某报价预算软件(PB9 程序) `Ver 5.1` 注册验证分析(二)

### 四、软件启动时验证注册授权文件信息分析

软件启动事件代码如下:

```powerbuilder
string ls_cbdm
string ls_cbnr
string ls_cpdm
string ls_parm
string ls_regcompany
string ls_regowner
string ls_regcompany1
string ls_regcompanymatch
string ls_xtsj
integer net
environment le_env
long ll_rtn
string ls_cs
string ls_str
integer answer
integer answer1
integer answer2
integer answer3
integer answer4
integer answer5
integer answer6
integer answer7
string ls_driver
string ls_start
string ls_location
string ls_value
string db_path
string s_dsn
string db_name
string ls_yhzt
string ls_dqcs
string ls_bzcs
string ls_ver
string ls_ver1
string ls_dwmc
date ldt_dqsj
date ldt_xtsj
date ldt_zcsj
date ldt_dlsj
long ll_zcsj
long ll_dqsj
long ll_dlsj
long ll_bzcs
long ll_dqcs
string ls_dlsj
string ls_dqsj
string ls_zcsj
string ls_zclm
string ls_parming

if getenvironment(le_env) <> 1 then
    messagebox("Application: Open","Unable to get environment information.~nHalting ...")
    halt
end if

if le_env.screenheight = 768 then
    gs_screen = "1"
else
    gs_screen = "1"
end if

gu_exp = create nuo_exp_function
g_api = create lht_nvo_api
as_dir = getcurrentdirectory()
gs_servertype = lower(profilestring("mldbase.ini","database","servertype",""))

if gs_servertype = "mssql" then
    sqlca.dbms = "MSS Microsoft SQL Server"
    sqlca.database = "mldbase"
    sqlca.servername = profilestring("mldbase.ini","Database","host","")
    sqlca.logid = "sa"
    sqlca.logpass = "gf"
    sqlca.autocommit = false
else
    if gs_servertype = "asa" then
        s_dsn = "mldbase"
        db_name = "mldbase"
        ls_driver = as_dir + "\\dbodbc8.dll"
        ls_start = as_dir + "\\dbeng8.exe"
        if not fileexists(ls_driver) and fileexists(ls_start) then
            messagebox("错误",as_dir + "目录中没有安装 dbeng8.exe 和 dbodbc8.dll 文件!",stopsign!)
            return
        else
            registryset("HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\App Paths\dbeng8.exe","path",regstring!,as_dir)
            registryset("HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\App Paths\dbeng8.exe","",regstring!,as_dir)
        end if
        answer1 = registryset("HKEY_LOCAL_MACHINE\Software\ODBC\ODBCINST.INI\ODBC DRIVERS","Adaptive Server Anywhere 8.0",regstring!,"Installed")
        answer2 = registryset("HKEY_LOCAL_MACHINE\Software\ODBC\ODBCINST.INI\Adaptive Server Anywhere 8.0","Driver",regstring!,ls_driver)
        answer3 = registryset("HKEY_LOCAL_MACHINE\Software\ODBC\ODBCINST.INI\Adaptive Server Anywhere 8.0","Setup",regstring!,ls_driver)
        if answer1 = -1 or answer2 = -1 or answer3 = -1 then
            messagebox("错误","应用程序无法设置 ODBC DRIVERS，运行不能继续!",stopsign!)
            return
        end if
        db_path = as_dir + "\\" + db_name + ".db"
        answer = registryset("HKEY_LOCAL_MACHINE\Software\ODBC\ODBC.INI\ODBC Data Sources",s_dsn,regstring!,"Adaptive Server Anywhere 8.0")
        answer1 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"driver",regstring!,ls_driver)
        answer2 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"start",regstring!,ls_start + " -d -Q -c1024")
        answer3 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"autostop",regstring!,"yes")
        answer4 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"DataBaseFile",regstring!,db_path)
        answer5 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"DataBaseName",regstring!,db_name)
        answer6 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"uid",regstring!,"dba")
        answer7 = registryset("HKEY_LOCAL_MACHINE\software\odbc\odbc.ini\" + s_dsn,"pwd",regstring!,"sql")
        if answer = -1 or answer1 = -1 or answer2 = -1 or answer3 = -1 or answer4 = -1 or answer5 = -1 or answer6 = -1 or answer7 = -1 then
            messagebox("错误","应用程序无法设置 ODBC SYSTEM DATA SOURCE，运行不能继续!",stopsign!)
            return
        end if
        sqlca.dbms = "ODBC"
        sqlca.autocommit = false
        sqlca.dbparm = "ConnectString='DSN=mldbase;UID=mulong;PWD=68483059'"
        sqlca.autocommit = false
    else
        halt close
    end if
end if

connect using sqlca;
if sqlca.sqlcode <> 0 then
    messagebox("操作提示","数据库连接失败")
    return
end if
//上面是检查运行环境等，不重要

gs_yhdm = "sa"
ls_parm = wf_login()
ldt_xtsj = date(today()) //系统时间

select zclm , ver , dwmc from update_rq using sqlca;
/* SQL Parameters List
0-> :ls_zclm
1-> :ls_ver
2-> :ls_dwmc
*/

数据库内容如下：
`zclm,ver,dwmc`
`'Y','MorrowBudget Ver5.1 ','看雪技术论坛'`

select dqcs , yhzt , dqsj , zcsj , dlsj , bzcs from lcs using sqlca; //几个时间非常重要
/* SQL Parameters List
0-> :ls_dqcs
1-> :ls_yhzt
2-> :ls_dqsj
3-> :ls_zcsj
4-> :ls_dlsj
5-> :ls_bzcs
*/

`dqcs,yhzt,dqsj,zcsj,dlsj,bzcs`
`'a1):K','093(io','QcR=opaBoQWaH<t?a2ZvH-64#cZX%L-p2rEb>?\*\\aMoG','YcmwL%aZy{+aUsrba%lw)->9@&c+5Rc-eR]AbW}x2a\\cf','$2 \\5>,0.Ypa0aQ5859iHU-D({b8^<\\a-@>#J3bSv510g^','w3|wu-1M3G'`

`ls_ver1 = ml2003.toolbarpopmenutext`
`gs_ver = ls_ver1`
`gs_gsmc = ls_dwmc`

`if ls_ver1 <> ls_ver or isnull(ls_ver) then //数据库中的软件版本`
`messagebox("警告","数据库与软件版本不匹配!")`
`halt`
`end if`

`ls_yhzt = wf_jiemi(ls_yhzt) //用户状态`
`ls_dqcs = wf_jiemi(ls_dqcs)`
`ls_bzcs = wf_jiemi(ls_bzcs)`
`ls_dqsj = wf_jiemi(ls_dqsj)`
`ls_zcsj = wf_jiemi(ls_zcsj)`
`ls_dlsj = wf_jiemi(ls_dlsj)`
`ls_dqsj = wf_chartonum(ls_dqsj)`
`ls_dlsj = wf_chartonum(ls_dlsj)`
`ls_zcsj = wf_chartonum(ls_zcsj)`
`ldt_dqsj = date(ls_dqsj) //2000-2-10`
`ldt_zcsj = date(ls_zcsj) //2000-2-10`
`ldt_dlsj = date(ls_dlsj) //2005-8-31`
`ll_bzcs = long(ls_bzcs)`
`ll_dqcs = long(ls_dqcs)`
`ll_zcsj = daysafter(ldt_zcsj,ldt_xtsj) //注册时间,和当前时间`
`ll_dqsj = daysafter(ldt_xtsj,ldt_dqsj) //到期时间`
`ll_dlsj = daysafter(ldt_dlsj,ldt_xtsj) //最后登入时间,当前时间`

`if ll_dlsj < 0 then`
`messagebox("提示","系统时间已被修改!软件不能正常使用")`
`halt`
`end if`



## 软件注册与到期处理逻辑

### 核心判断逻辑

```
if len(ls_yhzt) < 1 or isnull(ls_yhzt) or ls_yhzt = "0" then    // 0 或空表示未注册
   open(w_ruanjianzhuce)      // 注册窗口
   return
end if

if ls_parm = "w_xtsz" and ls_zclm = "Y" then
   open(w_xtsz)
   return
end if

if ls_parm = "w_xtsz" and ls_zclm = "N" then
   open(w_ruanjianzhuce)      // 注册窗口
end if

if ls_parm = "setupagain" then
   messagebox("提示","请重新安装版并注册后才能使用~r~n" + "联系电话: 010-62102392  62102393")
   halt
end if

if ls_yhzt = "9" and ll_zcsj <= 330 and ll_dqsj > 0 and ls_zclm = "Y" and ls_parm = "w_p_main" then
   ls_dqcs = string(ll_dqcs + 1)    // 到期时间
   ls_xtsj = string(ldt_xtsj)       // 系统时间
   ls_xtsj = wf_numtochar(ls_xtsj)   // 系统时间加密
   ls_xtsj = wf_jiami(ls_xtsj)
   ls_dqcs = wf_jiami(ls_dqcs)      // 到期时间
   update lcs set dlsj =' ' , dqcs =' '  using sqlca;
   /* SQL Parameters List
   0-> :ls_xtsj
   1-> :ls_dqcs
   */
   open(w_login)
   return
end if

if ls_yhzt = "9" and ll_zcsj > 330 and ll_zcsj < 365 and ll_dqsj > 0 and ls_zclm = "Y" and ls_parm = "w_p_main" then
   ls_dqcs = string(ll_dqcs + 1)
   ls_xtsj = string(ldt_xtsj)
   ls_xtsj = wf_numtochar(ls_xtsj)
   ls_xtsj = wf_jiami(ls_xtsj)
   ls_dqcs = wf_jiami(ls_dqcs)
   update lcs set dlsj =' ' , dqcs =' '  using sqlca;
   /* SQL Parameters List
   0-> :ls_xtsj
   1-> :ls_dqcs
   */
   commit using sqlca;
   open(w_login)
   ls_parming = string(365 - ll_zcsj)
   openwithparm(wf_xufei,ls_parming)             // 续费窗口
   return
end if

if ls_yhzt = "9" and ll_dqsj < 0 then    // 续费
   open(wf_xufei)
   return
end if

//// ls_yhzt = "2" 试用版
if ls_yhzt = "2" and ll_dqsj > ll_dlsj and ll_bzcs > ll_dqcs and ls_zclm = "N" and ls_parm = "shiyong" then
   net = messagebox("涉及软件名称,怕给论坛带来麻烦,编辑")
   ls_dqcs = string(ll_dqcs + 1)
   ls_xtsj = string(ldt_xtsj)
   ls_xtsj = wf_numtochar(ls_xtsj)
   ls_xtsj = wf_jiami(ls_xtsj)
   ls_dqcs = wf_jiami(ls_dqcs)
   update lcs set dlsj =' ' , dqcs =' '  using sqlca;
   /* SQL Parameters List
   0-> :ls_xtsj
   1-> :ls_dqcs
   */
   open(w_login)
   return
else
   net = messagebox("涉及软件名称,怕给论坛带来麻烦,编辑")
   open(w_ruanjianzhuce)
   return
end if
```

由上面代码可以得知，注册日期与到期日期相差 330 天以内，用户状态为 `9` 即可。根据要求重新修改注册授权文件生成代码：

```
Lines.Add(wf_jiami('9')); // 用户状态 9:正常 2:试用版
Lines.Add(ls_write_dqcs);
Lines.Add(ls_write_bzcs);
// Lines.Add(ls_write_dqsj); // 到期时间
Lines.Add(wf_jiami(FormatDateTime('yyyy-m-d', Date - 330)));
// Lines.Add(ls_write_zcsj); // 注册时间
Lines.Add(wf_jiami(FormatDateTime('yyyy-m-d', Date)));
```

### 五、软件到期注册序列号分析

首次注册成功软件仅可以使用最多 330 天，到期需要续费注册；

续费注册事件代码：

```
string ls_zcm1
string ls_zcm
string ls_ver
string ls_dqsj
integer li_int_1
date ldt_dlsjnew
date ldt_dqsj

ls_zcm1 = trim(parent.sle_14.text)
ls_zcm = parent.wf_zhuanhuan()                // 序列号，重要

if ls_zcm1 = ls_zcm then
   select int_1 from update_rq  using sqlca;    // 产生的序列号与输入的序列号比较
   /* SQL Parameters List
   0-> :li_int_1
   */
   li_int_1 = li_int_1 + 1           // 续费次数+1
   parent.cb_3.enabled = false
   ldt_dlsjnew = date(today())        // 最后登入时间
   select dqsj from lcs  using sqlca;
   /* SQL Parameters List
   0-> :ls_dqsj
   */
   ls_dqsj = parent.w_jiemi(ls_dqsj)              // 到期时间
   ls_dqsj = parent.wf_charatonum(ls_dqsj)
   ldt_dqsj = relativedate(date(ls_dqsj),365)    // 到期时间延长 365 天
   ls_dqsj = string(ldt_dqsj)
   ls_dqsj = parent.wf_numtochar(ls_dqsj)
   ls_dqsj = parent.w_jiami(ls_dqsj)
   update update_rq set zclm ='Y' , zcrq ={d '2005-08-31' } , ver =' ' , int_1 =0  using sqlca;  // 写数据库
   /* SQL Parameters List
   0-> :ldt_dlsjnew
   1-> :gs_ver
   2-> :li_int_1
   */
   commit using sqlca;
   update lcs set dqsj =' '  using sqlca;
   /* SQL Parameters List
   0-> :ls_dqsj
   */
   commit using sqlca;
   openwithparm(w_xufeichenggong,ls_zcm)    // 续费成功!
else                                          // 输入序列号错误
   messagebox("提示","您输入的注册码错误，重核对!")
end if

return
```

#### 涉及产生序列号重要函数

`parent.wf_zhuanhuan`

```
string ls_arry[]
string ls_zcm
string ls_zcm1
string ls_gsmc
string ls_pinyinma
string ls_gsdz
string ls_lxr
string ls_tel
string ls_z1
string ls_z2
string ls_z3
string ls_z4
string ls_z5
string ls_z6
integer li_zishu
integer li_i
integer li_asc
integer li_int_1

select int_1 , dwmc , gsdz , lxr , tel , updateid from update_rq  using sqlca;
/* SQL Parameters List
0-> :li_int_1
1-> :ls_gsmc
2-> :ls_gsdz
3-> :ls_lxr
4-> :ls_tel
5-> :ls_zcm
*/

ls_pinyinma = f_get_spellcn(ls_gsmc)           // 公司名称转拼音
ls_gsdz = f_get_spellcn(ls_gsdz)             // 公司地址
ls_lxr = f_get_spellcn(ls_lxr)               // 联系人
ls_tel = f_get_spellcn(ls_tel)               // 电话
ls_zcm = left(ls_zcm,4) + mid(ls_zcm,6,4) + mid(ls_zcm,11,4) + mid(ls_zcm,16,4) + mid(ls_zcm,26,4)  // 注册码去分割字符
ls_pinyinma = ls_pinyinma + ls_gsdz + ls_zcm + ls_lxr + ls_tel  // 连接
li_zishu = len(ls_pinyinma)  // 长度

for li_i = 1 to li_zishu
   ls_arry[li_i] = right(left(ls_pinyinma,li_i),li_i - (li_i - 1))  // 取一个字符
   li_asc = asc(ls_arry[li_i])  // ascii
   if ((li_asc >= 97 and li_asc <= 122) or (li_asc >= 65 and li_asc <= 90)) or (li_asc >= 48 and li_asc <= 57) then
      ls_zcm1 = ls_zcm1 + ls_arry[li_i]     // 过滤一些字符
   end if
next

ls_zcm1 = ls_zcm1 + ls_zcm1  // 剩余字符+剩余字符
li_zishu = len(ls_zcm1)      // 长度

for li_i = 1 to li_zishu
   ls_arry[li_i] = right(left(ls_pinyinma,li_i),li_i - (li_i - 1))    // 转到数组
next

if li_int_1 < 1 then
   li_int_1 = 10      // 这个数重要啊,续费次数
end if

// 取位 连接成注册码
ls_z1 = ls_arry[li_int_1] + ls_arry[li_int_1 + 5] + ls_arry[li_int_1 + 9] + ls_arry[li_int_1 + 13]
ls_z2 = ls_arry[li_int_1 + 1] + ls_arry[li_int_1 + 6] + ls_arry[li_int_1 + 10] + ls_arry[li_int_1 + 14]
ls_z3 = ls_arry[li_int_1 + 2] + ls_arry[li_int_1 + 7] + ls_arry[li_int_1 + 11] + ls_arry[li_int_1 + 15]
ls_z4 = ls_arry[li_int_1 + 3] + ls_arry[li_int_1 + 8] + ls_arry[li_int_1 + 12] + ls_arry[li_int_1 + 16]
ls_z5 = ls_arry[li_int_1 + 4] + ls_arry[li_int_1 + 9] + ls_arry[li_int_1 + 13] + ls_arry[li_int_1 + 17]
ls_z6 = ls_arry[li_int_1 + 5] + ls_arry[li_int_1 + 10] + ls_arry[li_int_1 + 14] + ls_arry[li_int_1 + 18]
ls_zcm = ls_z1 + "-" + ls_z2 + "-" + ls_z3 + "-" + ls_z4 + "-" + ls_z5 + "-" + ls_z6

return ls_zcm
```

#### 涉及函数：转拼音函数 `f_get_spellcn`

```
long i
string ls_ch
string ls_returnstr

for i = 1 to 2 * lenw(as_inputstring) step 2
   ls_ch = mid(as_inputstring,i,2)
   if asc(ls_ch) < 128 then
      ls_returnstr = ls_returnstr + ls_ch
   else
      choose case ls_ch
         case IS >= "匝"
            ls_returnstr = ls_returnstr + "Z"
            continue
         case IS >= "丫"
            ls_returnstr = ls_returnstr + "Y"
            continue
         case IS >= "夕"
            ls_returnstr = ls_returnstr + "X"
            continue
         case IS >= "哇"
            ls_returnstr = ls_returnstr + "W"
            continue
         case IS >= "他"
            ls_returnstr = ls_returnstr + "T"
            continue



```pascal
case IS >= "撒"  
  ls_returnstr = ls_returnstr + "S"  
  continue  
case IS >= "然"  
  ls_returnstr = ls_returnstr + "R"  
  continue  
case IS >= "七"  
  ls_returnstr = ls_returnstr + "Q"  
  continue  
case IS >= "趴"  
  ls_returnstr = ls_returnstr + "P"  
  continue  
case IS >= "哦"  
  ls_returnstr = ls_returnstr + "O"  
  continue  
case IS >= "拿"  
  ls_returnstr = ls_returnstr + "N"  
  continue  
case IS >= "妈"  
  ls_returnstr = ls_returnstr + "M"  
  continue  
case IS >= "廓"  
  ls_returnstr = ls_returnstr + "L"  
  continue  
case IS >= "咖"  
  ls_returnstr = ls_returnstr + "K"  
  continue  
case IS >= "讥"  
  ls_returnstr = ls_returnstr + "J"  
  continue  
case IS >= "哈"  
  ls_returnstr = ls_returnstr + "H"  
  continue  
case IS >= "嘎"  
  ls_returnstr = ls_returnstr + "G"  
  continue  
case IS >= "发"  
  ls_returnstr = ls_returnstr + "F"  
  continue  
case IS >= "讹"  
  ls_returnstr = ls_returnstr + "E"  
  continue  
case IS >= "搭"  
  ls_returnstr = ls_returnstr + "D"  
  continue  
case IS >= "擦"  
  ls_returnstr = ls_returnstr + "C"  
  continue  
case IS >= "八"  
  ls_returnstr = ls_returnstr + "B"  
  continue  
case IS >= "阿"  
  ls_returnstr = ls_returnstr + "A"  
end choose  
end if  
next  
return ls_returnstr
```

翻译成 Delphi 7.0 代码:

```pascal
Function wf_zhuanhuan(li_int: Integer): String;  
Var  
  r_ls_pinyinma, r_ls_gsdz, r_ls_lxr, r_ls_tel, r_ls_zcm: String;  
  li_i: Integer;  
  ls_zcm1, ls_arry: String;  
  li_asc: Integer;  
  ls_z1, ls_z2, ls_z3, ls_z4, ls_z5, ls_z6: String;  
  li_int_1: Integer;  
Begin  
  r_ls_pinyinma := f_get_spellcn(ls_gsmc); //公司名称转拼音  
  r_ls_gsdz := f_get_spellcn(ls_gsdz); //公司地址  
  r_ls_lxr := f_get_spellcn(ls_lxr); //联系人  
  r_ls_tel := f_get_spellcn(ls_tel); //电话  
  r_ls_zcm := LeftStr(ls_zcm, 4) + MidStr(ls_zcm, 6, 4) + MidStr(ls_zcm, 11, 4) + MidStr(ls_zcm, 16, 4) + MidStr(ls_zcm, 26, 4); //注册码去分割字符  
  r_ls_pinyinma := r_ls_pinyinma + r_ls_gsdz + r_ls_zcm + r_ls_lxr + r_ls_tel; //连接  
  For li_i := 1 To Length(r_ls_pinyinma) Do  
  Begin  
    li_asc := Ord(r_ls_pinyinma[li_i]); //ascii  
    If ((li_asc >= 97) And (li_asc <= 122)) Or ((li_asc >= 65) And (li_asc <= 90)) Or ((li_asc >= 48) And (li_asc <= 57)) Then  
      ls_zcm1 := ls_zcm1 + r_ls_pinyinma[li_i]; //过滤一些字符  
  End;  
  ls_zcm1 := ls_zcm1 + ls_zcm1; //剩余字符+剩余字符  
  If li_int < 1 Then li_int_1 := 10 Else li_int_1 := li_int;  

//取位 连接成注册码  
  ls_arry := ls_zcm1;  
  ls_z1 := ls_arry[li_int_1] + ls_arry[li_int_1 + 5] + ls_arry[li_int_1 + 9] + ls_arry[li_int_1 + 13];  
  ls_z2 := ls_arry[li_int_1 + 1] + ls_arry[li_int_1 + 6] + ls_arry[li_int_1 + 10] + ls_arry[li_int_1 + 14];  
  ls_z3 := ls_arry[li_int_1 + 2] + ls_arry[li_int_1 + 7] + ls_arry[li_int_1 + 11] + ls_arry[li_int_1 + 15];  
  ls_z4 := ls_arry[li_int_1 + 3] + ls_arry[li_int_1 + 8] + ls_arry[li_int_1 + 12] + ls_arry[li_int_1 + 16];  
  ls_z5 := ls_arry[li_int_1 + 4] + ls_arry[li_int_1 + 9] + ls_arry[li_int_1 + 13] + ls_arry[li_int_1 + 17];  
  ls_z6 := ls_arry[li_int_1 + 5] + ls_arry[li_int_1 + 10] + ls_arry[li_int_1 + 14] + ls_arry[li_int_1 + 18];  
  Result := ls_z1 + '-' + ls_z2 + '-' + ls_z3 + '-' + ls_z4 + '-' + ls_z5 + '-' + ls_z6;  
End;
```

取汉字首字符代码太长了，网上有现成的，此处省略。

『破解总结』:

`PB` 的程序有反编译器，基本上只要读懂 `PB` 代码，了解几个变量用途即可，有耐性的基本上都能完成软件调试。

上面分析结果通过验证，已经能成功进入软件主界面,有没有其它功能限制不清楚，因为这个软件本人不用，仅仅凭兴趣调试一下，不妥之处敬请谅解。

`BTW`:代码量可能比较多,排版效果不好，能看到这里说明你很有耐性，这个软件你肯定也能搞定。

----------完--------------
`4dd466d0e1e0cc123afbe504da2462ca`
