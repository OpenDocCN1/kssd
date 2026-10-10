

# 无责任硬件写断点的 loader 例子

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-451.htm`](https://www.pediy.com/kssd/pediy07/pediy7-451.htm)

program Loader;  
  
(\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*)  
(\*        无 - 责 - 任          \*)  
(\*  应用硬件写断点的 loader 例子  \*)  
(\*          by tt.t             \*)  
(\*                              \*)  
(\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*)  
  
uses  
  JwaWinType,  
  JwaWinNt,  
  JwaWinBase,  
  JwaWinUser;  
  
{$R \*.res}  
  
const  
  VictimName = 'test.exe';  
  (\*  
    test.exe 为测试程序，会写$44fbc0(0x44fbc0)这个地址，我们的目标是当它改写$44fbc0 后，  
    重新将$44fbc0 的值设为 0。  
    上次用设页面属性配合 debug api 实现了这个目标（参见：http://bbs.pediy.com/showthread.php?s=&threadid=17939），  
    这次尝试用硬件断点实现同样功能，可以看出用硬件断点可以少打很多字，我喜欢。  
    硬件断点的介绍请参阅 hume 的相关文章和 intel 手册，我没看过 intel 手册，下面代码或许  
    有问题，功力实在有限，而且懒得看书，测试通过就拿出来了，所以不能保证下面代码的  
    规范稳定。如果出现数据丢失、死机、硬盘损坏、房屋倒塌、地球爆炸等严重后果本人概不负责。  
  \*)  
  MemPos: DWORD = $44fbc0;  
  RegFlag: DWORD = 0;  
  
var  
  si: STARTUPINFO;  
  pi: PROCESS\_INFORMATION;  
  
function SimpleExtractFilePath(f: String): String;  
var  
  i: integer;  
begin  
  for i := Length(f) downto 1 do  
    if f\[i\] = '' then Break;  
  result := copy(f, 0, i);  
end;  
  
procedure CreateVictimProcess(Path: String);  
var  
  DbgEvent: TDebugEvent;  
  DbgParam: DWORD;  
  NewPrt: DWORD;  
  DbgContext: TContext;  
begin      
  ZeroMemory(@si, SizeOf(STARTUPINFO));  
  si.cb := SizeOf(STARTUPINFO);  
  if not CreateProcess(PChar(Path), nil, nil, nil, False, CREATE\_DEFAULT\_ERROR\_MODE, nil,  
                       PChar(SimpleExtractFilePath(Path)), si, pi) then  
  begin  
    MessageBox(0, 'CreateProcess failed! ', 'Error!', 0);  
    exit;  
  end;  
  if WaitForInputIdle(pi.hProcess, INFINITE) <> 0 then  
  begin  
    MessageBox(0, 'WaitForInputIdle failed! ', 'Error!', 0);  
    exit;  
  end;  
  if not DebugActiveProcess(pi.dwProcessId) then  
  //这里用 DebugActiveProcess 来附加到目标进程，如果用 CreateProcess 时加上 DEBUG\_ONLY\_THIS\_PROCESS  
  //方式调试，设置硬件断点方法参阅 hume 的文章。  
  begin  
    MessageBox(0, 'DebugActiveProcess failed! ', 'Error!', 0);  
    exit;  
  end;               
  DbgContext.ContextFlags := CONTEXT\_ALL;  
  while WaitForDebugEvent(DbgEvent, INFINITE) do  
  begin  
    DbgParam := DBG\_CONTINUE;  
    case DbgEvent.dwDebugEventCode of  
      CREATE\_PROCESS\_DEBUG\_EVENT:  
      (\*  
        对 DebugActiveProcess 附加 debugee 的情况，必须在 CREATE\_PROCESS\_DEBUG\_EVENT 中，  
        不可在第一个 EXCEPTION\_BREAKPOINT 处设置硬件断点。这是试验出的结论，不知道为什么。  
        没有试验 deguber 挂起 debugee 后设硬件断点情况。  
       \*)  
      begin  
        GetThreadContext(pi.hThread, DbgContext);  
        DbgContext.Dr0 := MemPos;  
        DbgContext.Dr7 := $D0501;    //Dr7 flag：对 dr0 指向地址设写中断  
        SetThreadContext(pi.hThread, DbgContext);  
      end;  
      EXCEPTION\_DEBUG\_EVENT:  
      begin  
        case DbgEvent.Exception.ExceptionRecord.ExceptionCode of  
          EXCEPTION\_ACCESS\_VIOLATION:  
          begin  
            DbgParam := DBG\_EXCEPTION\_NOT\_HANDLED;  
          end;  
          EXCEPTION\_SINGLE\_STEP:  
          begin  
            begin  
              GetThreadContext(pi.hThread, DbgContext);  
              if DbgContext.Dr6 and 1 = 1 then             
              begin  //触发了 dr0 断点，这时 dr0 指向的地址已经被改写，这里将其恢复为 0  
                VirtualProtectEx(pi.hProcess, Pointer(MemPos), 4, PAGE\_READWRITE, @NewPrt);  
                WriteProcessMemory(pi.hProcess, Pointer(MemPos), @RegFlag, 4, nil);  
                VirtualProtectEx(pi.hProcess, Pointer(MemPos), 4, NewPrt, @NewPrt);  
              end  
              else  
                DbgParam := DBG\_EXCEPTION\_NOT\_HANDLED;  
            end;  
          end;  
        end;  
      end;  
      EXIT\_PROCESS\_DEBUG\_EVENT:  
      begin  //收工  
        ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);  
        Break;  
      end;  
    end;  
    ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);  
  end;  
end;  
  
var  
  Victim: string;  
begin  
  Victim := SimpleExtractFilePath(ParamStr(0)) + VictimName;  
  CreateVictimProcess(Victim);  
  halt;  
end. 

 4dd466d0e1e0cc123afbe504da2462ca