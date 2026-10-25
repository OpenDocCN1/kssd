
# Delphi 使用 WindowsAPI 枚举递归枚举注册表

> 原文：`www.pediy.com/kssd/pediy10/80441.html`

首先声明:本人是 DELPHI 初学者。

最近需要一个枚举注册表的函数，而且还是需要 DELPHI 来写。在网络搜索很久，都没找到。

也许是我的搜索关键字不对。

这下我只能利用 VC 的代码转换为 DELPHI。

测试已经通过，很稳定。

使用方式:如果要枚举 `HKEY_CURRENT_USER`

测试代码:

```
Reg_RecursionEnum(HKEY_CURRENT_USER, '', '', 
                 mpr_Form_RegFindInvalidFilePath.ST_ScanRegPath);
```

代码:

```
{**********************************************************************
/*函数名:  Reg_RecursionEnum(hkey_param_Root:HKEY;
                             const str_param_SubKey:string;
                             const str_param_FullKeyPath:string;
                             var SText_param_ScanRegPath:TStaticText):Boolean;

/*功能:    递归注册表

/*参数:   1> 注册表根项
          2> 子键项名
          3> 递归时所有子键项名的总和[全注册表路径]
          4> 在界面上显示的路径[跟 GUI 交互]

/*作者:   ZTS

/*返回值:  True  递归成功
          False 递归失败

**********************************************************************}
function Reg_RecursionEnum(hkey_param_Root:HKEY;
                           const str_param_SubKey:string;
                           const str_param_FullKeyPath:string;
                           var SText_param_ScanRegPath:TStaticText):Boolean;
var
  int_numSubKey:Integer       ;
  int_numSubKeyVal:Integer    ;
  dword_sizeSubKey:DWORD      ;
  dword_sizeSubKeyVal:DWORD   ;
  int_SubKeyIndex:Integer     ;
  int_SubKeyValIndex:Integer  ;

  str_QueryVal:string         ;
  dword_QueryValSize:DWORD    ;
  dworl_QueryValType:DWORD    ;



  char_SubKey:array[0..G_INT_MAX_KEY_LENGTH-1] of Char ;
  char_SubKeyVal:array[0..G_INT_MAX_KEY_LENGTH-1] of Char ;

begin
  Result := True ;

  int_numSubKey := 0 ;
  int_numSubKeyVal := 0 ;

  if RegOpenKeyEx(hkey_param_Root, PChar(str_param_SubKey), 0,
                  KEY_ALL_ACCESS,
                  hkey_param_Root) = ERROR_SUCCESS then
  begin

    // 打印: 子键项名的总和[全注册表路径]
    // SText_param_ScanRegPath.Caption := str_param_FullKeyPath ;

    //*首先查得当前键下的子键项数以及子键项值数目*//
    RegQueryInfoKey(hkey_param_Root, nil, nil, nil,
                    @int_numSubKey, nil, nil, @int_numSubKeyVal, nil,
                    nil, nil, nil) ;
    if int_numSubKey <> 0 then
    begin
      for int_SubKeyIndex := 0 to int_numSubKey-1 do
      begin
        {*子键项值枚举查询*}
        if int_numSubKeyVal <> 0 then
        begin
          for int_SubKeyValIndex := 0 to int_numSubKeyVal-1 do
          begin
            ZeroMemory(@char_SubKeyVal, G_INT_MAX_KEY_LENGTH);
            dword_sizeSubKey := G_INT_MAX_KEY_LENGTH ;

            RegEnumValue(hkey_param_Root, int_SubKeyValIndex, char_SubKeyVal, dword_sizeSubKey,
                         nil, nil, nil, nil) ;

           // 打印: 子键项值名称
           // SText_param_ScanRegPath.Caption := string(char_SubKeyVal) ;

           // 查询类型[REG_SZ, REG_EXPAND_SZ]
           dword_QueryValSize := 0 ;
           dworl_QueryValType := 0 ;
           if RegQueryValueEx(hkey_param_Root, char_SubKeyVal, nil,
                              @dworl_QueryValType, nil, @dword_QueryValSize) = ERROR_SUCCESS then
           begin
             if dworl_QueryValType in [REG_SZ, REG_EXPAND_SZ] then
             begin
                 SetLength(str_QueryVal, dword_QueryValSize) ;
                 if RegQueryValueEx(hkey_param_Root, char_SubKeyVal, nil,
                                    @dworl_QueryValType, PByte(str_QueryVal), @dword_QueryValSize) = ERROR_SUCCESS then
                 begin
                    SetLength(str_QueryVal, StrLen(PChar(str_QueryVal))) ;

                    // 打印: 子键项值内容
                    // SText_param_ScanRegPath.Caption := str_QueryVal ;

                 end;

             end;

           end;

          end;

        end;

        {*子键项枚举查询*}
        ZeroMemory(@char_SubKey, G_INT_MAX_KEY_LENGTH);
        dword_sizeSubKey := G_INT_MAX_KEY_LENGTH ;

        // 开始枚举子键项,枚举出一个子键项则递归调用本函数体
        RegEnumKeyEx(hkey_param_Root, int_SubKeyIndex, char_SubKey, dword_sizeSubKey,
                     nil, nil, nil, nil) ;

        // 进入下一次递归[子键项]
        Reg_RecursionEnum(hkey_param_Root, string(char_SubKey), str_param_FullKeyPath+string(char_SubKey)+'\',
                          SText_param_ScanRegPath) ;

      end;

    end;

    RegCloseKey(hkey_param_Root) ;

  end;

end; // End Reg_RecursionEnum()
```
