
# 某报价预算软件(PB9程序) Ver 5.1 注册验证分析

日期：2005年9月6日　　破解人：Baby2008

---

- 软件名称：某报价预算软件 成套版 Ver 5.1
- 软件大小：34.5MB
- 下载地址：商业软件，不便提供，看懂得自己搜索
- 软件介绍：
- 保护方式：注册文件+注册码保护
- 破解声明：初学Crack，只是感兴趣，个人感觉PB程序分析破文不多，由我来抛砖引玉吧
- 破解工具：OllyDbg.V1.10，PBKiller（感谢作者:kivens）

## 破解过程

### 一、安装序列号破解

安装时需要软件安装序列号，那先分析一个软件安装序列号；

运行安装程序，到达序列号输入界面，由于安装包是由Setup Factory 7.0制作的，安装程序会在安装过程中会产生一个临时进程来完成安装，

破解安装序列号需要切换到临时进程，启动fllyODBG，附加进程：

```
未命名的窗口，项目 35
进程=00000FC4
名称=irsetup
窗口=产品序列号
路径=C:\DOCUME~1\wxb\LOCALS~1\Temp\irsetup.exe   //安装程序临时进程
```

设置Point H(万能断点)，输入安装序列号1234567890，下一步，OD中断并返回主流程：

```
0042E1DC     FF15 4C144400        call dword ptr ds:[<&USER32.GetWindowTextA>]    ; USER32.GetWindowTextA
0042E1E2     6A FF                push -1                                         ; 返回这里
0042E1E4     8BCF                 mov ecx,edi
0042E1E6     E8 BA2D0000          call irsetup.00430FA5
0042E1EB     EB 13                jmp short irsetup.0042E200
0042E1ED     8BCE                 mov ecx,esi
0042E1EF     E8 CA1E0000          call irsetup.004300BE
0042E1F4     85C0                 test eax,eax
0042E1F6     74 08                je short irsetup.0042E200
0042E1F8     57                   push edi
0042E1F9     8BC8                 mov ecx,eax
0042E1FB     E8 52FFFFFF          call irsetup.0042E152
0042E200     8B07                 mov eax,dword ptr ds:[edi]                      ; 试炼码
0042E202     5F                   pop edi
0042E203     5E                   pop esi
0042E204     8B40 F8              mov eax,dword ptr ds:[eax-8]
0042E207     C2 0800              retn 8
………(省略代码N行)…………………
00415607     E8 53B60100          call irsetup.00430C5F
0041560C     8065 FC 00           and byte ptr ss:[ebp-4],0
00415610     8D4D E8              lea ecx,dword ptr ss:[ebp-18]
00415613     E8 5AB50100          call irsetup.00430B72
00415618     FF75 08              push dword ptr ss:[ebp+8]
0041561B     FF75 F0              push dword ptr ss:[ebp-10]
0041561E     E8 A2CA0000          call irsetup.004220C5
```

堆栈信息：

```
0012F510   00CBA9E8  ASCII "1234567890"
0012F514   00AF20F0  ASCII "X2D"
0012F518   00AF20F0  ASCII "X2D"
0012F51C   0012F50C
0012F520   00CAF648  ASCII "9896-7818-3668-5118"
0012F524   00000000
0012F528   00CAF648  ASCII "9896-7818-3668-5118"     //安装序列号
```

得到安装序列号:9896-7818-3668-5118  顺利完成安装过程。

### 二、注册信息文件分析

程序为PB9编写，OD调试会在“PB解释器”里打转，还是用PB克星PBKiller（再次感谢作者:kivens，DePB公开版本不支持PB9）反编译各事件代码，源代码可读性非常的好，下面对各重要代码稍作解释：

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
//取得数据库内容为:
1,'N',' ',' ','MorrowBudget Ver5.1 '

if li_int_1 < 1 then
        li_int_1 = 1            //重要，指续费次数，后面会用到
end if

if ls_zclm = "N" and (len(ls_zcm) < 1 or isnull(ls_zcm)) then
        ls_zcm = parent.wf_xlh()                                   //软件序列号，相当于软件ID，升级ID
        update update_rq set updateid =' '  using sqlca;           //保存ID
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

ls_gsmc = trim(parent.sle_1.text)                    //读取界面空间内容
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
if len(ls_gsmc) < 1 or isnull(ls_gsmc) then                                //开始判断合法性
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
'a1):K',' ','QcR=opaBoQWaH<t?a2ZvH-64#_cZX%L-p2rEb>?*\aMoG','YcmwL%aZy{+aUsrba%lw)->9@&c+5Rc-eR]AbW}x2a\cf','$2\5>,0.Ypa0aQ5859iHU-D({b8^<\a-@>#J3bSv510g^','w3|wu-1M3G'

select maxuser from c_maxuser  using sqlca;       //最大用户数,结果为空,单机版没用到
/* SQL Parameters List
0-> :ls_maxuser
*/
ls_write_yhzt = ls_yhzt             //为写注册信息文件准备变量
ls_write_dqsj = ls_dqsj
ls_write_zcsj = ls_zcsj
ls_write_dlsj = ls_dlsj
ls_write_bzcs = ls_bzcs
ls_write_dqcs = ls_dqcs
ls_maxuser = parent.wf_jiami(ls_maxuser)     //加密wf_jiami
ls_shengji = string(li_int_1) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))
ls_shengji = parent.wf_numtochar(ls_shengji)   //数字转字符wf_numtochar
ls_shengji = parent.wf_jiami(ls_shengji)
ll_cpu = abs(getcpuid())                    //CPU ID
if ll_cpu > 0 then     //CPU ID 获取失败用随机数代替
        ls_write_cpu = string(ll_cpu)
else
        ls_write_cpu = "77" + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99)) + string(rand(99))
end if
ls_rootpath = "C:"
ls_volumnename = space(256)                //取 C:\ 磁盘序列号
lul_volumenamesize = 256
lul_maximumcomponentlength = 256
ls_filesystemnamebuffer = space(256)
lul_filesystemnamesize = 256
beep(1)
lb_rtn = false
lb_rtn = getvolumeinformationa(ls_rootpath,ls_volumnename,lul_volumenamesize,lul_volumeserialnumber,lul_maximumcomponentlength,lul_filesystemflags,ls_filesystemnamebuffer,lul_filesystemnamesize)
```
