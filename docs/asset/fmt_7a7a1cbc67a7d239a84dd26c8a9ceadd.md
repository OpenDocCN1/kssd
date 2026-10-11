
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
