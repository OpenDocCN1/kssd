
# 应用内存断点的 Loader

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-437.htm`](https://www.pediy.com/kssd/pediy07/pediy7-437.htm)

```
program Loader;  

(*****************************************************************)  

(*                          *)  

(* Memory Access Breakpoint Loader *)  

(*      by tt.t             *)  

(*                          *)  

(*                          *)  

(*****************************************************************)  

{  
目标：拦截对指定地址的写操作，nop 掉写地址的指令。  
难点：在要补丁地址设 Memory Write Breakpoint，查了半天没找到现成的东西，只好自己写。  
思路：将要补丁的地址设为不可写，当写时会发生 AV 错误，然后进行 Patch。但 VirtualProtectEx 会将整  
个 Page 设为不可写属性，所有写 Page 的操作都会产生 Access violation，无法直接找到要 Patch 的代码。  
解决方法：要保存地址所在 Page 的地址范围，发生 AV 错误时判断是否因为 VirtualProtect 产生的 Exceptio  
n，如不是将 Page 的属性设为可写，同时设置单步标志，写操作完成后恢复 Page 为不可写属性，继续执  
行，直至找到需 Patch 代码。  
不知道有没有更好的方法。  
}  

uses  
  JwaWinType,  
  JwaWinNt,  
  JwaWinBase,  
  JwaWinUser;  
{$R *.res}  

var  
  si: STARTUPINFO;  
  pi: PROCESS_INFORMATION;  

function MyExtractFilePath(f: String): String;  

var  
  i, l: integer;  

begin  
  l := Length(f);  
  for i := l downto 1 do  
    if f[i] = '' then Break;  
  result := copy(f, 0, i);  
end;  

procedure CreateVictimProcess(Path: String);  

const  
  Nop: PChar = Chr($90) + Chr($90) + Chr($90);  

var  
  DbgEvent: TDebugEvent;  
  DbgParam: DWORD;  
  OldPrt, NewPrt: DWORD;  
  pPatch: PByte;  
  PgMin, PgMax: DWORD;  
  MemInfo: TMemoryBasicInformation;  
  WExpAddr: DWORD;  
  DbgContext: TContext;  
  rm: Boolean;  
  hThread: DWORD;  

begin     
  ZeroMemory(@si, sizeof(STARTUPINFO));  
  si.cb := sizeof(STARTUPINFO);  
  if not CreateProcess(PChar(Path), nil, nil, nil, False, CREATE_SUSPENDED or   
CREATE_DEFAULT_ERROR_MODE, nil,  
                      PChar(MyExtractFilePath(Path)), si, pi) then  
  begin  
    MessageBox(0, 'CreateProcess failed! ', 'Error!', 0);  
    exit;  
  end;  
  ResumeThread(pi.hThread);  
  if WaitForInputIdle(pi.hProcess, INFINITE) <> 0 then  
  begin  
    MessageBox(0, 'WaitForInputIdle failed! ', 'Error!', 0);  
    exit;  
  end;  
  if not DebugActiveProcess(pi.dwProcessId) then  
  begin  
    MessageBox(0, 'DebugActiveProcess failed! ', 'Error!', 0);  
    exit;  
  end;   
  if VirtualQueryEx(pi.hProcess, Pointer($9c66BC), MemInfo, SizeOf(MemInfo)) = 0 then  
  begin  
    MessageBox(0, 'VirtualQueryEx failed! ', 'Error!', 0);  
    exit;  
  end;  
  PgMin := DWORD(MemInfo.BaseAddress);  
  pgMax := PgMin + MemInfo.RegionSize;  
  {VirtualProtect 会将整个 Page 设为不可写属性，所有写 Page 的操作都会产生 Access violation，要保存  
整个 Page 的地址范围，  
   后面可以判断是否因为 VirtualProtect 产生的 Exception}       
  if not VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, PAGE_EXECUTE_READ, @OldPrt) then  
  begin  
    MessageBox(0, 'VirtualProtectEx failed! ', 'Error!', 0);  
    exit;  
  end;  
  {改写 Page 属性为不可写}  
  rm := false;  
  while WaitForDebugEvent(DbgEvent, INFINITE) do  
  begin  
    DbgParam := DBG_CONTINUE;  
    case DbgEvent.dwDebugEventCode of  
      EXCEPTION_DEBUG_EVENT:  
      begin  
        if DbgEvent.Exception.ExceptionRecord.ExceptionCode <> EXCEPTION_BREAKPOINT then  
          case DbgEvent.Exception.ExceptionRecord.ExceptionCode of  
            EXCEPTION_SINGLE_STEP: {单步中断}  
            begin  
              if rm then {由于 EXCEPTION_ACCESS_VIOLATION 产生的单步中断，恢复 Page 为不可写属性}  
              begin  
                rm := false;  
                VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, PAGE_EXECUTE_READ, @NewPrt);  
              end;  
            end;  
            EXCEPTION_ACCESS_VIOLATION: {AV 中断}  
            begin  
              DbgParam := DBG_EXCEPTION_NOT_HANDLED;  
              if DbgEvent.Exception.ExceptionRecord.ExceptionInformation[0] = 1 then {写操作}  
              begin  
                WExpAddr := DbgEvent.Exception.ExceptionRecord.ExceptionInformation[1]; {写操作的目标地  
址}  
                if (WExpAddr >= PgMin) and (WExpAddr <= PgMax) then {目标地址在 Page 范围}  
                begin  
                  DbgParam := DBG_CONTINUE;  
                  if(WExpAddr <> $9c66BC) then {不是写指定地址}  
                  begin  
                    VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, OldPrt, @NewPrt);  
                    DbgContext.ContextFlags := CONTEXT_CONTROL;  
                    hThread := OpenThread(THREAD_ALL_ACCESS, false, DbgEvent.dwThreadId);  
                    GetThreadContext(hThread, DbgContext);  
                    DbgContext.EFlags := DbgContext.EFlags or $100;  
{设单步标志，会触发 EXCEPTION_SINGLE_STEP}  
                    SetThreadContext(hThread, DbgContext);  
                    rm := true; {标志，表明是 EXCEPTION_ACCESS_VIOLATION 产生的单步中断}  
                  end  
                  else  
                  begin {Patch}  
                    pPatch := DbgEvent.Exception.ExceptionRecord.ExceptionAddress;  
                    VirtualProtectEx(pi.hProcess, pPatch, 3, PAGE_READWRITE, @NewPrt);  
                    WriteProcessMemory(pi.hProcess, pPatch, Nop, 3, nil);  
                    VirtualProtectEx(pi.hProcess, pPatch, 3, NewPrt, @NewPrt);  
                  end;  
                end;  
              end;  
            end;  
            else  
              DbgParam := DBG_EXCEPTION_NOT_HANDLED;  
          end;  
      end;  
      EXIT_PROCESS_DEBUG_EVENT:  
      begin  
        ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);  
        Break;  
      end;  
    end;  
    ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);  
  end;  
end;  

var  
  Victim: string;  

begin  
  Victim := MyExtractFilePath(ParamStr(0)) + 'Wise\*.exe'; //受害程序  
  CreateVictimProcess(Victim);  
  halt;  
end. 
```

ft，既然看不懂那就那就详细注释下。  

记得要大体了解下 debug api 的用法和使用流程先.  

```
procedure CreateVictimProcess(Path: String);  

const  
  Nop: PChar = Chr($90) + Chr($90) + Chr($90);  

var  
  DbgEvent: TDebugEvent;  
  DbgParam: DWORD;  //ContinueDebugEvent 用,标志如何处理调试消息  
  OldPrt, NewPrt: DWORD;  
  pPatch: PByte;  
  PgMin, PgMax: DWORD;  
  MemInfo: TMemoryBasicInformation;  
  WExpAddr: DWORD;  
  DbgContext: TContext;  
  rm: Boolean;  
  hThread: DWORD;  

begin     
  ZeroMemory(@si, sizeof(STARTUPINFO));  
  si.cb := sizeof(STARTUPINFO);  
  if not CreateProcess(PChar(Path), nil, nil, nil, False, CREATE_SUSPENDED or CREATE_DEFAULT_ERROR_MODE, nil,  
                      PChar(MyExtractFilePath(Path)), si, pi) then  
  begin  
    MessageBox(0, 'CreateProcess failed! ', 'Error!', 0);  
    exit;  
  end;  
  //建立目标进程  
  ResumeThread(pi.hThread);  
  //恢复进程执行.其实 CreateProcess 时不加 CREATE_SUSPENDED 标志就可以省掉这句  
  if WaitForInputIdle(pi.hProcess, INFINITE) <> 0 then  
  begin  
    MessageBox(0, 'WaitForInputIdle failed! ', 'Error!', 0);  
    exit;  
  end;  
  //等待目标进程完全运行至其开始等待用户输入.  
  //因为目标程序是加过壳的,壳的部分会检查调试器,所以等到壳运行结束在去 debug 它.  
  if not DebugActiveProcess(pi.dwProcessId) then  
  begin  
    MessageBox(0, 'DebugActiveProcess failed! ', 'Error!', 0);  
    exit;  
  end;   
  //挂上目标程序  
  if VirtualQueryEx(pi.hProcess, Pointer($9c66BC), MemInfo, SizeOf(MemInfo)) = 0 then  
  begin  
    MessageBox(0, 'VirtualQueryEx failed! ', 'Error!', 0);  
    exit;  
  end;  
  //查询需要设内存断点的地址($9c66BC,也就是 0x9c66BC,后面简称 addr)所在 page 的情况  
  PgMin := DWORD(MemInfo.BaseAddress);  
  pgMax := PgMin + MemInfo.RegionSize;  
  //得到 addr 所在的 page 的始末地址.  
  {因为 VirtualProtect 会将 addr 所在整个 Page 设为不可写属性(见 VirtualProtect 的 API 说明)，所有写 Page 的操作,即使不是写我们感兴趣(但位于那个 page 上)的地址的操作,都会因为将 addr 设为不可写属性而产生 Access violation.所以这里要保存整个 Page 的地址范围，以便后面判断 AV 是不是因写 addr 而产生的}   
  if not VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, PAGE_EXECUTE_READ, @OldPrt) then  
  begin  
```



`MessageBox(0, 'VirtualProtectEx failed! ', 'Error!', 0);`

`exit;`

`end;`

`{改写 addr 属性为不可写.这里会将 addr 所在的整个 Page 设为不可写属性}`

`rm := false; {标志是不是因处理 AV 设置的单步中断产生的中断}`

`while WaitForDebugEvent(DbgEvent, INFINITE) do //等待调试消息`

`begin`

`  DbgParam := DBG_CONTINUE; //默认处理调试消息`

`  case DbgEvent.dwDebugEventCode of`

`    EXCEPTION_DEBUG_EVENT:`

`    begin`

`      if DbgEvent.Exception.ExceptionRecord.ExceptionCode <> EXCEPTION_BREAKPOINT then`

`      //判断是不是 int 3 产生的中断.因为 DebugActiveProcess 挂上程序后会向 debuger 发出一个 EXCEPTION_BREAKPOINT 调试消息,这里忽略它`

`        case DbgEvent.Exception.ExceptionRecord.ExceptionCode of //判断异常类型`

`          EXCEPTION_ACCESS_VIOLATION: {AV 异常}`

`          begin`

`            DbgParam := DBG_EXCEPTION_NOT_HANDLED; //表示由目标程序的 SEH 处理异常`

`            if DbgEvent.Exception.ExceptionRecord.ExceptionInformation[0] = 1 then`

`            begin {是写操作产生的 AV}`

`              WExpAddr := DbgEvent.Exception.ExceptionRecord.ExceptionInformation[1]; {WExpAddr =写操作的目标地址}`

`              if (WExpAddr >= PgMin) and (WExpAddr <= PgMax) then {WExpAddr 在 addr 所在 Page 上}`

`              begin`

`                DbgParam := DBG_CONTINUE;//表示我们来处理异常,让目标程序从发生异常处继续执行.`

`                if(WExpAddr <> $9c66BC) then {WExpAddr 不是写 addr,要允许目标程序写操作}`

`                begin`

`                  VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, OldPrt, @NewPrt);//恢复 addr 所在 page 为原来的属性(可写)`

`                  DbgContext.ContextFlags := CONTEXT_CONTROL;`

`                  hThread := OpenThread(THREAD_ALL_ACCESS, false, DbgEvent.dwThreadId);`

`                  //由 Thread Id 得到 Thread handle`

`                  GetThreadContext(hThread, DbgContext);`

`                  //得到 thread 的 Context,设置单步执行标志,让写操作完成后发生单步中断,以便将 addr 重新设为不可写,拦截下一次写操作.`

`                  DbgContext.EFlags := DbgContext.EFlags or $100; {设单步标志}`

`                  SetThreadContext(hThread, DbgContext);`

`                  //设置 thread 的 Context`

`                  rm := true; {标志，表明是我们处理 AV 而产生的单步中断}`

`                end`

`                else //{WExpAddr 是写 addr,进行 Patch}`

`                begin`

`                  pPatch := DbgEvent.Exception.ExceptionRecord.ExceptionAddress;`

`                  //写地址操作语句地址`

`                  VirtualProtectEx(pi.hProcess, pPatch, 3, PAGE_READWRITE, @NewPrt);`

`                  //设操作语句地址为可写`

`                  WriteProcessMemory(pi.hProcess, pPatch, Nop, 3, nil);`

`                  //nop 掉写地址语句`

`                  VirtualProtectEx(pi.hProcess, pPatch, 3, NewPrt, @NewPrt);`

`                  //恢复语句地址属性`

`                end;`

`              end;`

`            end;`

`          end;{AV 异常处理完毕}`

`          EXCEPTION_SINGLE_STEP: {发生单步中断}`

`          begin`

`            if rm then {由于处理 AV 产生的单步中断.}`

`            begin`

`              rm := false;`

`             //重置标志`

`              VirtualProtectEx(pi.hProcess, Pointer($9c66BC), 1, PAGE_EXECUTE_READ, @NewPrt);`

`             //恢复 Page 为不可写属性,以便拦截下次写操作`

`            end;`

`          end;`

`          else//不是 AV 中断或单步中断,由目标程序的 SEH 处理异常`

`            DbgParam := DBG_EXCEPTION_NOT_HANDLED;`

`        end;`

`    end;`

`    EXIT_PROCESS_DEBUG_EVENT:`

`    begin`

`      ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);`

`      //目标程序推出,loader 使命结束,退出`

`      Break;`

`    end;`

`  end;`

`  ContinueDebugEvent(DbgEvent.dwProcessId, DbgEvent.dwThreadId, DbgParam);`

`end;`

`end;`

`4dd466d0e1e0cc123afbe504da2462ca`
