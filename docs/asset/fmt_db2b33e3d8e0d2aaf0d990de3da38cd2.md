
Shift: TShiftState; X, Y: Integer);  

begin  

if Button = mbLeft then  

   begin  

    ReleaseCapture();  

    SendMessage(Handle, WM_NCLBUTTONDOWN, HTCAPTION, 0);  

   end;  

end;  

end.

### 补丁模块源码

引用: //Loader by wynney  

```pascal
program loader;  

uses  

  ShellApi, Windows, Inifiles, SysUtils, Messages, Variants, Classes, Graphics,  

  Controls, ExtCtrls, StdCtrls, Dialogs, Forms, Kill in 'Kill.pas';  

{$R Loader.RES}  

Var  

  WindowName : integer;  

  ProcessId : integer;  

  ThreadId : integer;  

  Buf : PChar;  

  HandleWindow : integer;  

  Handle : integer;  

  Write : Cardinal;  

  Buf1 : integer;  

  Appz: Pchar;  

  WindowTitle : Pchar;  

  Address : Cardinal;  

  PokeValue : Cardinal;  

  NumberOfBytes : Cardinal;  

  Inifile: Tinifile;//INI  

  Files: string;//配置文件  

  Des: TFileStream;//资源  

  size: integer;//资源大小  

  FileMemo: TMemorystream;//文件内存流对象  

  s: TStrings;  

  i, j: Cardinal;  

  Info: String;  

Var  

  Str: String;  

  Len: Byte absolute Str;  

begin  

  Files := ExtractFilePath(Application.ExeName) + '\\Config.ini'; //释放配置文件  

  FileMemo := TMemoryStream.Create;  

  Des := TFileStream.Create(Application.ExeName, fmShareDenyNone);  

  try  

    des.Seek(-sizeof(size), soFromEnd);  

    des.ReadBuffer(size, sizeof(size));  

    des.Seek(-Size, soFromEnd);  

    FileMemo.CopyFrom(Des, size - sizeof(size));  

    if FileExists(Files) = False then //若配置文件已存，则取消释放  

      begin  

        FileMemo.SaveToFile(Files);  

      end;  

  finally  

    FileMemo.Free;  

    des.Free;  

  end;  

  Inifile := Tinifile.Create(Files);  

  s := TStringlist.Create;  

  IniFile.ReadSections(s);  

  SetFileAttributes(Pchar(Files), FILE_ATTRIBUTE_HIDDEN + FILE_ATTRIBUTE_SYSTEM);  

  Info := Inifile.ReadString('Info', 'MyInfo', '');  

  WindowTitle := PChar(Inifile.ReadString('Appz', 'TitleName', ''));  

  Appz := PChar(Inifile.ReadString('Appz', 'AppzName', ''));  

  j := StrToInt(Inifile.ReadString('Nume', 'Number', ''));  

  if Info <> '' then  

    begin  

      if Application.MessageBox(PChar(Info), '友好提示', MB_YESNO + MB_IconQuestion) = mrNO then Exit;  

    end;  

  ShellExecute(Handle, PChar('Open'), Appz, nil, Appz, SW_SHOWNORMAL); //要打开的目标程序  

  //开始 Loader  

  for Buf1 := 0 to 999999 do //多次查找  

    begin  

      WindowName := FindWindow(nil, WindowTitle);  

      if WindowName = 0 then  

        begin  

          WindowName := FindWindow(nil, WindowTitle);  

          if Buf1 = 999999 then  

            begin  

              ShowMessage('程序异常，请重启！');  

              Exit; //退出补丁  

              KillTask(Appz); //杀死目标进程  

            end;  

        end  

        else  

        begin  

          ThreadId := GetWindowThreadProcessId(WindowName, @ProcessID);  

          HandleWindow := OpenProcess(Process_All_Access, False, ProcessId);  

          for i := 0 to j do  

            begin  

              Address := StrToInt(Inifile.ReadString('Data', 'Address' + IntToStr(i), '')); //读取数据  

              PokeValue := StrToInt(Inifile.ReadString('Data', 'NewData' + IntToStr(i), '')); //读取数据  

              NumberOfBytes := 1;  

              GetMem(Buf, 1);  

              Buf^ := Chr(PokeValue);  

              WriteProcessMemory(HandleWindow, Ptr(Address), Buf, NumberOfBytes, Write);  

            end;  

          DeleteFile(Files); //用完之后马上删除  

          Exit; //自己随后退出  

        end;  

    end;  

end.
```

### Kill（杀进程）模块源码

引用:  
```pascal
unit Kill;  

interface  

uses  

  Windows, SysUtils, Variants, Classes, Graphics, StdCtrls, Tlhelp32;  

function KillTask(ExeFileName: string): integer;  

implementation  

function KillTask(ExeFileName: string): integer; //杀死进程的函数，来对付防火墙进程。  

const  

  PROCESS_TERMINATE = $0001; //进程的 PROCESS_TERMINATE 访问权限  

var  

  ContinueLoop: BOOL;  

  FSnapshotHandle: THandle;  

  FProcessEntry32: TProcessEntry32;  

begin  

  result := 0;  

  FSnapshotHandle := CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);  

  //获取系统所有进程快照  

  FProcessEntry32.dwSize := Sizeof(FProcessEntry32);  

  //调用 Process32First 前用 Sizeof(FProcessEntry32)填充 FProcessEntry32.dwSize  

  ContinueLoop := Process32First(FSnapshotHandle, FProcessEntry32);  

  //获取快照中第一个进程信息并保存到 FProcessEntry32 结构体中  

  while integer(ContinueLoop) <> 0 do  

  //循环枚举快照中所有进程信息  

    begin  

      if ((UpperCase(ExtractFileName(FProcessEntry32.szExeFile)) = UpperCase(ExeFileName))  

        or (UpperCase(FProcessEntry32.szExeFile) = UpperCase(ExeFileName))) then  

      //找到要中止的进程名  

        Result := Integer(TerminateProcess(OpenProcess(PROCESS_TERMINATE, BOOL(0),  

          FProcessEntry32.th32ProcessID), 0));  

      //中止进程  

        ContinueLoop := Process32Next(FSnapshotHandle, FProcessEntry32);  

      //查找下一个符合条件进程  

    end;  

end;  

end.
```

最后附上 `Dup2`、`KeyMake` 补丁模块范例, 经本人调试此源码对本文目标程序补丁无效`[因为壳的缘故]`，大家可做参考  

引用:  

```pascal
program Loader;  

uses  

  Windows, TlHelp32;  

const  

  OldData : array[0..1] of byte = ($8B, $C3);  

  NewData : array[0..1] of byte = ($B0, $01);  

var  

  StartInfo: TStartupInfo;  

  ProcInfo: TProcessInformation;  

  BytesRead : DWORD;  

  TmpData : array[0..1] of Byte;  

begin  

  ZeroMemory(@StartInfo, SizeOf(TStartupInfo));  

  StartInfo.cb := SizeOf(TStartupInfo);  

  if not CreateProcess(nil, 'Tail4win.exe', nil, nil, False, Create_Suspended, nil, nil, StartInfo, ProcInfo) then Exit;  

  ReadProcessMemory(ProcInfo.hProcess, Pointer($004D2A7A), @TmpData, 2, BytesRead);  

  if (TmpData[0] = OldData[0]) and (TmpData[1] = OldData[1]) then  

    begin  

      if WriteProcessMemory(ProcInfo.hProcess, Pointer($004D2A7A), @NewData, 2, BytesRead) then  

        MessageBox(0, '恭喜补丁成功', nil, 0) else MessageBox(0, '补丁失败', nil, 0);  

    end else  

    begin  

      MessageBox(0, '补丁失败', nil, 0)  

    end;  

  ResumeThread(ProcInfo.hThread);  

  CloseHandle(ProcInfo.hProcess);  

  CloseHandle(ProcInfo.hThread);  

end.
```

--------------------------------------------------------------------------------  

**【经验总结】**  

1. 如果把补丁和原目标程序整合为一个应用程序，那么就更加完美了，可惜现期还没时间去写，可以利用捆绑的思想，有兴趣的朋友可以继续：）  

2. 个人觉得带壳调试已经是不可阻挡的趋势了，大家可以多花点工夫在上面：）  

3. 谢绝用心不良的人转载源码  

4. 文章有点长，感谢你耐心看完：）  

--------------------------------------------------------------------------------  

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢! 

`4dd466d0e1e0cc123afbe504da2462ca`
