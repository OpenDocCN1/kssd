
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
